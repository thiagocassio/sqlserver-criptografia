# 01. TDE — Transparent Data Encryption

## Objetivo

Proteger **dados em repouso (at rest)** no SQL Server.

Protege:
- arquivo `.mdf`
- arquivo `.ndf`
- arquivo `.ldf`
- backups
- snapshots

O TDE atua em **nível de arquivo físico do banco**.

O SQL Server descriptografa automaticamente ao ler a página em memória.

---

## Quando usar

- LGPD
- proteção contra roubo de backup
- proteção contra cópia indevida dos arquivos
- compliance ISO 27001
- auditoria
- proteção de DR externo

---

## Como usar

O TDE pode ser implementado de duas formas no SQL Server:

- via **script (T-SQL)**
- via **interface (SSMS)**

---

### Via script (T-SQL)

#### Etapa 1 — Criar a Master Key

```sql
USE master;
GO

CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'SenhaForte@2026';
GO
```

#### Etapa 2 — Criar o certificado

```sql
CREATE CERTIFICATE CertTDE
WITH SUBJECT = 'Certificado para TDE';
GO
```

#### Etapa 3 — Criar a chave de criptografia do banco
```sql
USE SeuBanco;
GO

CREATE DATABASE ENCRYPTION KEY
WITH ALGORITHM = AES_256
ENCRYPTION BY SERVER CERTIFICATE CertTDE;
GO
```

#### Etapa 4 — Ativar o TDE
```sql
ALTER DATABASE SeuBanco
SET ENCRYPTION ON;
GO
```

---

### Via interface (SSMS)
**Caminho:**

`Banco > Tasks > Manage Database Encryption`

**Fluxo:**

1. selecionar o banco

2. criar ou escolher certificado

3. escolher algoritmo (**AES 256**)

4. habilitar encryption

5. confirmar

---

## Como validar
```sql
SELECT
DB_NAME(database_id) AS Banco,
encryption_state,
encryptor_type
FROM sys.dm_database_encryption_keys;
```

### Resultado esperado
- `3 = encrypted`

---

## Como testar de verdade

O teste ideal é:

1. ativar o TDE
   
2. gerar backup
   
3. tentar restaurar em outra instância **sem o certificado**
   
4. validar a falha
   
5. importar o certificado

6. restaurar novamente

Esse laboratório prova que o TDE está funcionando corretamente.

---

## Vantagens
- não exige alteração na aplicação
- fácil de administrar
- ótimo para proteção física
- excelente para compliance
- protege backup e arquivos do banco

---

## Limitações
- não protege contra DBA com acesso ao banco online
- dados continuam visíveis em `SELECT`
- depende de certificado para restore
- perda do certificado pode impedir recuperação

---

## Melhor visão

| Cenário | Tecnologia |
|---|---|
| Proteção de MDF / LDF / NDF | TDE |
| Backup criptografado | TDE |
| Proteção física do banco | TDE |
