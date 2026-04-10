# 07. EKM — Extensible Key Management

## Objetivo
Manter as **chaves criptográficas fora da instância SQL Server**, usando:

- Azure Key Vault
- Azure Managed HSM
- HSM de terceiros
- BYOK (Bring Your Own Key)

O objetivo é garantir **separação entre dados e gerenciamento de chaves**, elevando o nível de segurança e segregação operacional.

---

## Quando usar
- ambiente enterprise
- ISO 27001
- PCI DSS
- alta exigência de segurança
- HSM
- BYOK
- segregação entre DBA e time de segurança
- políticas rígidas de rotação de chave

---

## Como usar
O EKM é implementado principalmente por:

- via **plataforma + script (recomendado)**
- via **SSMS como console de execução**

> **Observação:** a parte mais importante acontece fora do SQL Server, no **Azure Key Vault / HSM**.

---

### Via plataforma + script

#### Etapa 1 — Criar o Azure Key Vault
No Azure:

- criar o **Key Vault**
- criar ou importar chave **RSA 2048 / 3072**
- conceder permissões:
  - `get`
  - `list`
  - `wrapKey`
  - `unwrapKey`

Essas permissões são necessárias para a aplicação do **SQL Server Connector**.

#### Etapa 2 — Instalar o SQL Server Connector
Instalar no servidor SQL o:

**SQL Server Connector for Azure Key Vault**

Ele registra a DLL usada pelo provider EKM.

#### Etapa 3 — Habilitar EKM no SQL Server
```sql
USE master;
GO
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
GO
EXEC sp_configure 'EKM provider enabled', 1;
RECONFIGURE;
GO
```

#### Etapa 4 — Registrar o provider
```sql
CREATE CRYPTOGRAPHIC PROVIDER AzureKeyVault_EKM
FROM FILE = 'C:\Program Files\SQL Server Connector for Microsoft Azure Key Vault\Microsoft.AzureKeyVaultService.EKM.dll';
GO
```

#### Etapa 5 — Criar a credential
```sql
CREATE CREDENTIAL sysadmin_ekm_cred
WITH IDENTITY = 'SeuKeyVault.vault.azure.net',
SECRET = '<ClientIDSemHifen><ClientSecret>'
FOR CRYPTOGRAPHIC PROVIDER AzureKeyVault_EKM;
GO
```

Essa credential conecta o SQL Server ao Azure Key Vault.

---

### Via interface (SSMS)
O SSMS é usado como **console para execução dos scripts e testes**.

**Fluxo:**
1. abrir **New Query**
2. habilitar EKM
3. registrar provider
4. criar credential
5. testar uso da chave externa

Não existe wizard dedicado no SSMS para EKM.

---

## Como validar

### Validar provider registrado
```sql
SELECT *
FROM sys.cryptographic_providers;
```

### Validar credentials
```sql
SELECT name, credential_identity
FROM sys.credentials;
```

### Resultado esperado
- provider `AzureKeyVault_EKM` registrado
- credential apontando para o vault

---

## Como testar de verdade
O teste ideal é:

1. configurar o Key Vault
2. registrar provider no SQL Server
3. criar credential
4. criar asymmetric key externa
5. usar a chave no **TDE ou Backup Encryption**
6. validar que a chave **não está armazenada dentro do SQL Server**

Esse laboratório prova a separação entre dado e chave.

---

## Vantagens
- chaves ficam fora do SQL Server
- excelente para HSM e BYOK
- melhora segregação entre DBA e segurança
- ideal para PCI DSS, ISO 27001 e ambientes enterprise
- suporta rotação centralizada de chaves
- ótimo para auditoria e segregação de função

---

## Limitações
- depende de conectividade com o Key Vault / HSM
- perda de acesso ao vault pode impactar restore e abertura de chaves
- configuração inicial é mais complexa
- exige provider, connector e permissões do Entra ID
- troubleshooting costuma envolver SQL + Azure + rede
- aumenta dependência de serviços externos

---

## Melhor visão
| Cenário | Tecnologia |
|---|---|
| Chave externa | EKM |
| Azure Key Vault | EKM |
| HSM | EKM |
| BYOK | EKM |
