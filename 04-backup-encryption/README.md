4) Backup Encryption
Objetivo: criptografar apenas o backup.

- .bak
- cópias enviadas para nuvem
- armazenamento externo
- DR site

Quando usar:
- backup vai para storage externo
- fitas
- Azure Blob / S3
- parceiro terceirizado

Como usar:
- via script (T-SQL)

Etapa 1 — Criar Master Key

USE master;
GO
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'SenhaForte@2026';
GO

Etapa 2 — Criar certificado

CREATE CERTIFICATE Cert_Backup
WITH SUBJECT = 'Certificado Backup Encryption';
GO

Etapa 3 — Gerar backup criptografado

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

Etapa 4 — Restaurar backup

RESTORE VERIFYONLY
FROM DISK = 'C:\Backup\SeuBanco_Enc.bak';
GO

- Via interface (SSMS) → assistente gráfico

Caminho no SSMS:

Banco > Tasks > Back Up

Fluxo:

1- selecionar o banco
2- escolher destino do .bak
3- marcar Encrypt backup
4- escolher algoritmo AES 256
5- selecionar certificado
6- executar

Para laboratório visual, esse caminho é ótimo para evidenciar a opção de criptografia.

Como validar:

Validar cabeçalho do backup

RESTORE HEADERONLY
FROM DISK = 'C:\Backup\SeuBanco_Enc.bak';

Validar restore na mesma instância

RESTORE VERIFYONLY
FROM DISK = 'C:\Backup\SeuBanco_Enc.bak';

Resultado esperado
- backup legível apenas com o certificado disponível
- restore falha em outra instância sem certificado

Como testar de verdade:

O teste ideal é:

1- gerar backup criptografado
2- copiar o .bak para outra instância
3- tentar restore sem o certificado
4- validar a falha
5- importar certificado
6- restaurar novamente com sucesso

Esse laboratório prova a proteção do arquivo físico de backup.

Vantagens:
- simples
- excelente para auditoria
- reduz risco de vazamento de backup

Limitações:
- não protege banco online
- apenas o arquivo de backup
