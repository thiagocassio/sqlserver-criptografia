# 02. Always Encrypted

## Objetivo
Proteger **dados sensíveis até mesmo do DBA**, mantendo a chave fora do SQL Server.

Exemplos de uso:
- CPF
- cartão
- salário
- dados pessoais
- senhas temporárias
- tokens

A criptografia e descriptografia acontecem no **lado do cliente/aplicação**.

O SQL Server recebe apenas o valor criptografado.

---

## Quando usar
- dados extremamente sensíveis
- cenários **zero trust**
- ambiente onde DBA não pode visualizar dados
- LGPD
- segregação de função
- proteção de PII

---

## Como usar
> **Observação:** a criação do **Column Master Key (CMK)** normalmente depende de um repositório de chaves, como:
> - Windows Certificate Store
> - Azure Key Vault

O caminho mais simples para laboratório é criar pelo **SSMS** e depois versionar o script gerado.

O Always Encrypted pode ser implementado de duas formas:

- via **script (T-SQL)**
- via **interface (SSMS)**

---

### Via script (T-SQL)

#### Etapa 1 — Criar a chave mestra da coluna (CMK)
```sql
CREATE COLUMN MASTER KEY CMK_Lab
WITH (
    KEY_STORE_PROVIDER_NAME = N'MSSQL_CERTIFICATE_STORE',
    KEY_PATH = N'CurrentUser/My/SEU_CERTIFICADO'
);
GO
```

#### Etapa 2 — Criar a chave de criptografia da coluna (CEK)
```sql
CREATE COLUMN ENCRYPTION KEY CEK_Dados
WITH VALUES (
    COLUMN_MASTER_KEY = CMK_Lab,
    ALGORITHM = 'RSA_OAEP',
    ENCRYPTED_VALUE = 0x...
);
GO
```

#### Etapa 3 — Criar tabela com coluna criptografada
```sql
CREATE TABLE ClienteAE (
    Id INT PRIMARY KEY,
    CPF VARCHAR(11)
    COLLATE Latin1_General_BIN2
    ENCRYPTED WITH (
        COLUMN_ENCRYPTION_KEY = CEK_Dados,
        ENCRYPTION_TYPE = DETERMINISTIC,
        ALGORITHM = 'AEAD_AES_256_CBC_HMAC_SHA_256'
    )
);
GO
```

#### Etapa 4 — Inserir dados via cliente compatível
```sql
INSERT INTO ClienteAE (Id, CPF)
VALUES (1, '12345678901');
```

> Para funcionar corretamente, a conexão deve usar:
> `Column Encryption Setting=Enabled`

---

### Via interface (SSMS)
**Caminho:**

`Tabela > Columns > Encrypt Columns`

**Fluxo:**
1. selecionar a tabela
2. escolher a coluna sensível
3. selecionar **Deterministic** ou **Randomized**
4. criar CMK
5. criar CEK
6. aplicar alteração

Para laboratório, essa é a forma mais simples porque o SSMS gera o script automaticamente.

---

## Como validar

### Validar colunas criptografadas
```sql
SELECT 
    c.name AS Coluna,
    cek.name AS CEK
FROM sys.columns c
JOIN sys.column_encryption_keys cek
    ON c.column_encryption_key_id = cek.column_encryption_key_id
WHERE object_id = OBJECT_ID('dbo.ClienteAE');
```

### Validar leitura sem chave
```sql
SELECT * FROM ClienteAE;
```

### Resultado esperado
- sem driver compatível → valor binário ilegível
- com driver + chave → valor descriptografado

---

## Como testar de verdade
O teste ideal é:

1. criar coluna criptografada
2. inserir CPF real
3. consultar no SSMS **sem Always Encrypted habilitado**
4. validar valor criptografado
5. habilitar conexão com `Column Encryption Setting=Enabled`
6. consultar novamente e validar descriptografia

Esse laboratório prova a proteção contra leitura indevida.

---

## Vantagens
- protege contra leitura por administradores
- excelente para LGPD
- ideal para dados altamente sensíveis
- protege mesmo contra acesso direto ao SQL
- forte segregação entre DBA e aplicação

---

## Limitações
- mais complexo
- impacto na aplicação e driver
- algumas consultas perdem compatibilidade
- maior dependência de certificado / key store
- troubleshooting mais sensível

---

## Melhor visão
| Cenário | Tecnologia |
|---|---|
| Dado sensível | Always Encrypted |
| Proteção contra DBA | Always Encrypted |
| Zero trust | Always Encrypted |
| PII / LGPD | Always Encrypted |
