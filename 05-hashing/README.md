# 05. Hashing

## Objetivo
Garantir **integridade, comparação segura e pseudonimização irreversível** de dados como:

- CPF
- e-mail
- token
- trilhas de auditoria
- identificadores internos

O objetivo do hashing é gerar uma **impressão digital irreversível do valor**, sem possibilidade de descriptografia.

---

## Quando usar
- senha de aplicação (**preferencialmente fora do SQL Server**)
- trilha de auditoria
- detectar alteração de dados
- LGPD para pseudonimização
- comparação segura
- deduplicação por hash
- validação de integridade

---

## Como usar
O Hashing pode ser implementado de duas formas:

- via **script (T-SQL)**
- via **SSMS como console de execução**

---

### Via script (T-SQL)

#### Etapa 1 — Gerar hash simples
```sql
SELECT HASHBYTES('SHA2_256', '12345678901') AS HashCPF;
```

#### Etapa 2 — Criar coluna para armazenar hash
```sql
ALTER TABLE Cliente
ADD CPF_HASH VARBINARY(32);
GO
```

#### Etapa 3 — Popular a coluna
```sql
UPDATE Cliente
SET CPF_HASH = HASHBYTES('SHA2_256', CPF);
GO
```

#### Etapa 4 — Comparar valor informado
```sql
DECLARE @CPF VARCHAR(11) = '12345678901';

SELECT *
FROM Cliente
WHERE CPF_HASH = HASHBYTES('SHA2_256', @CPF);
GO
```

---

### Via interface (SSMS)
No SSMS **não existe wizard específico para hash**.

O uso via ferramenta normalmente é:

1. abrir **New Query**
2. executar geração do hash
3. comparar valores
4. validar igualdade entre hashes

Ou seja, aqui a ferramenta é o **SSMS como console de execução e evidência do resultado**.

---

## Como validar

### Validar hash gerado
```sql
SELECT CPF, CPF_HASH
FROM Cliente;
```

### Validar comparação segura
```sql
SELECT CASE
    WHEN HASHBYTES('SHA2_256', '12345678901') = CPF_HASH
    THEN 'IGUAL'
    ELSE 'DIFERENTE'
END AS Resultado
FROM Cliente
WHERE Id = 1;
```

### Resultado esperado
- mesmo valor = mesmo hash
- alteração mínima no texto = hash totalmente diferente

---

## Como testar de verdade
O teste ideal é:

1. gerar hash do CPF
2. salvar na tabela
3. alterar um único dígito do CPF
4. gerar novo hash
5. comparar os resultados
6. validar que o hash mudou completamente

Esse laboratório prova **integridade e comparação segura**.

---

## Vantagens
- não permite reversão do valor original
- excelente para comparação segura
- ideal para integridade e auditoria
- simples de implementar com `HASHBYTES`
- ótimo para pseudonimização LGPD
- ótimo para detectar alteração mínima

---

## Limitações
- não recupera o valor original
- não substitui criptografia quando precisa ler o dado
- mesmo hash sem **salt** pode sofrer ataque por dicionário
- escolha ruim do algoritmo compromete segurança
- não serve para dados que precisam ser descriptografados

---

## Melhor visão
| Cenário | Tecnologia |
|---|---|
| Integridade | Hashing |
| Pseudonimização | Hashing |
| Auditoria | Hashing |
| Comparação segura | Hashing |
