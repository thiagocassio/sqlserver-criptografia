# 03. Criptografia em nível de coluna

## Objetivo
Proteger **colunas específicas com criptografia reversível e controle manual de chave**, ideal para dados como:

- CPF
- salário
- token
- campos internos sensíveis

Esse modelo é ideal quando você precisa de **controle fino da criptografia no T-SQL**.

---

## Quando usar
- proteger poucas colunas específicas
- quando não pode usar **Always Encrypted**
- controle manual da criptografia
- stored procedures
- regras customizadas no SQL Server
- dados internos sensíveis

---

## Como usar
A criptografia em nível de coluna pode ser implementada de duas formas:

- via **script (T-SQL)**
- via **SSMS como console de execução**

---

### Via script (T-SQL)

#### Etapa 1 — Criar Master Key
```sql
USE SeuBanco;
GO
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'SenhaForte@2026';
GO
```

#### Etapa 2 — Criar certificado
```sql
CREATE CERTIFICATE Cert_Coluna
WITH SUBJECT = 'Certificado Criptografia Coluna';
GO
```

#### Etapa 3 — Criar chave simétrica
```sql
CREATE SYMMETRIC KEY SK_CPF
WITH ALGORITHM = AES_256
ENCRYPTION BY CERTIFICATE Cert_Coluna;
GO
```

#### Etapa 4 — Criptografar a coluna
```sql
OPEN SYMMETRIC KEY SK_CPF
DECRYPTION BY CERTIFICATE Cert_Coluna;
GO

UPDATE Cliente
SET CPF_Enc = EncryptByKey(
    Key_GUID('SK_CPF'),
    CPF
);
GO

CLOSE SYMMETRIC KEY SK_CPF;
GO
```

---

### Via interface (SSMS)
No SSMS **não existe wizard dedicado**, como ocorre no TDE ou Always Encrypted.

O uso via ferramenta normalmente é:

1. abrir **New Query**
2. executar os scripts por etapa
3. validar a tabela visualmente
4. consultar dados criptografados

Ou seja, aqui a ferramenta é o **SSMS como console de execução e validação do T-SQL**.

---

## Como validar

### Validar dado criptografado
```sql
SELECT CPF_Enc
FROM Cliente;
```

### Validar descriptografia
```sql
OPEN SYMMETRIC KEY SK_CPF
DECRYPTION BY CERTIFICATE Cert_Coluna;
GO

SELECT 
    CONVERT(VARCHAR(11), DecryptByKey(CPF_Enc)) AS CPF
FROM Cliente;
GO

CLOSE SYMMETRIC KEY SK_CPF;
```

### Resultado esperado
- coluna armazenada em formato binário
- valor original retornando apenas com a chave aberta

---

## Como testar de verdade
O teste ideal é:

1. criar coluna `CPF_Enc VARBINARY(MAX)`
2. criptografar dados reais da coluna CPF
3. consultar a coluna criptografada
4. validar o binário salvo
5. abrir a chave
6. descriptografar e comparar com o valor original

Esse laboratório prova a criptografia e a reversão controlada.

---

## Vantagens
- flexível
- nativo do SQL Server
- ótimo para stored procedures
- controle fino da chave
- ideal para poucas colunas críticas

---

## Limitações
- precisa abrir a chave na sessão
- a aplicação precisa tratar descriptografia
- maior risco de erro operacional
- depende de disciplina no uso de `OPEN` / `CLOSE KEY`
- menos transparente que TDE

---

## Melhor visão
| Cenário | Tecnologia |
|---|---|
| Coluna específica | Criptografia em nível de coluna |
| Controle manual | Symmetric Key + Certificate |
| Stored procedure | Column-Level Encryption |
| Campo interno sensível | Column-Level Encryption |
