# 04. Backup Encryption

## Objetivo
Criptografar **apenas o arquivo de backup**, protegendo a mídia fora da instância SQL Server.

Protege:
- arquivo `.bak`
- cópias enviadas para nuvem
- armazenamento externo
- DR site
- mídia removível
- parceiros terceirizados

Esse recurso é ideal para evitar exposição do backup fora do ambiente produtivo.

---

## Quando usar
- backup vai para storage externo
- fitas
- Azure Blob
- Amazon S3
- parceiro terceirizado
- DR externo
- retenção fora da instância

---

## Como usar
O Backup Encryption pode ser implementado de duas formas:

- via **script (T-SQL)**
- via **interface (SSMS)**

---

### Via script (T-SQL)

#### Etapa 1 — Criar Master Key
```sql
USE master;
GO
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'SenhaForte@2026';
GO
```

#### Etapa 2 — Criar certificado
```sql
CREATE CERTIFICATE Cert_Backup
WITH SUBJECT = 'Certificado Backup Encryption';
GO
```

#### Etapa 3 — Gerar backup criptografado
```sql
BACKUP DATABASE SeuBanco
TO DISK = 'C:\Backup\SeuBanco_Enc.bak'
WITH 
    COMPRESSION,
    ENCRYPTION (
        ALGORITHM = AES_256,
        SERVER CERTIFICATE = Cert_Backup
    ),
    STATS = 10;
GO
```

#### Etapa 4 — Validar o backup
```sql
RESTORE VERIFYONLY
FROM DISK = 'C:\Backup\SeuBanco_Enc.bak';
GO
```

---

### Via interface (SSMS)
**Caminho:**

`Banco > Tasks > Back Up`

**Fluxo:**
1. selecionar o banco
2. escolher destino do `.bak`
3. marcar **Encrypt backup**
4. escolher algoritmo **AES 256**
5. selecionar certificado
6. executar

Para laboratório visual, esse caminho é ótimo para evidenciar a opção de criptografia.

---

## Como validar

### Validar cabeçalho do backup
```sql
RESTORE HEADERONLY
FROM DISK = 'C:\Backup\SeuBanco_Enc.bak';
```

### Validar restore na mesma instância
```sql
RESTORE VERIFYONLY
FROM DISK = 'C:\Backup\SeuBanco_Enc.bak';
```

### Resultado esperado
- backup legível apenas com o certificado disponível
- restore falha em outra instância sem certificado

---

## Como testar de verdade
O teste ideal é:

1. gerar backup criptografado
2. copiar o `.bak` para outra instância
3. tentar restore **sem o certificado**
4. validar a falha
5. importar o certificado
6. restaurar novamente com sucesso

Esse laboratório prova a proteção do arquivo físico do backup.

---

## Vantagens
- simples
- excelente para auditoria
- reduz risco de vazamento de backup
- ótimo para DR externo
- forte aderência à LGPD e ISO 27001

---

## Limitações
- não protege banco online
- protege apenas o arquivo de backup
- depende de certificado para restore
- perda do certificado pode inviabilizar recuperação

---

## Melhor visão
| Cenário | Tecnologia |
|---|---|
| Backup / mídia | Backup Encryption |
| DR externo | Backup Encryption |
| Nuvem / storage | Backup Encryption |
| Parceiro terceirizado | Backup Encryption |
