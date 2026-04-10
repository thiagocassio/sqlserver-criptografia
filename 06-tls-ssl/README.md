# 06. Criptografia de conexão (TLS / SSL)

## Objetivo
Proteger **dados em trânsito** entre cliente e SQL Server.

Protege:
- conexão aplicação → SQL Server
- SSMS → SQL Server
- linked servers
- replicação
- conexões remotas
- tráfego em VPN / cloud / internet

O foco é impedir **captura de credenciais, sniffing e exposição do tráfego de rede**.

---

## Quando usar
- sempre
- especialmente cloud
- VPN
- internet
- conexões remotas
- linked servers
- ambientes híbridos
- replicação entre sites

---

## Como usar
A criptografia de conexão é implementada principalmente por:

- via **interface / plataforma**
- via **script (somente validação)**

> **Observação:** a habilitação do TLS no SQL Server é feita no **SQL Server Configuration Manager**, não por T-SQL.

---

### Via script (T-SQL)
**Não se aplica para habilitação.**

O T-SQL é usado apenas para **validar se a conexão está criptografada**.

---

### Via interface (Configuration Manager)
A configuração principal é feita no:

**SQL Server Configuration Manager**

#### Etapa 1 — Instalar certificado no Windows
O certificado deve estar no repositório do computador local:

- `Local Computer`
- `Personal`
- com chave privada
- EKU para **Server Authentication**

#### Etapa 2 — Vincular no SQL Server
**Caminho:**

`SQL Server Network Configuration > Protocols for MSSQLSERVER > Certificate`

Selecionar o certificado correto.

#### Etapa 3 — Forçar criptografia
No mesmo local:

`Flags > Force Encryption = Yes`

#### Etapa 4 — Reiniciar serviço
Reiniciar o serviço do SQL Server para aplicar.

---

## Como validar

### Validar conexões criptografadas
```sql
SELECT
    session_id,
    encrypt_option,
    client_net_address,
    auth_scheme
FROM sys.dm_exec_connections
WHERE session_id = @@SPID;
```

### Resultado esperado
- `encrypt_option = TRUE`

---

## Como testar de verdade
O teste ideal é:

1. conectar no SQL Server antes da configuração
2. validar `encrypt_option = FALSE`
3. aplicar certificado
4. ativar **Force Encryption**
5. reiniciar serviço
6. reconectar e validar `TRUE`

Esse laboratório prova a proteção do tráfego de rede.

---

## Vantagens
- impede sniffing
- protege credenciais
- essencial para produção
- excelente para cloud e VPN
- validação simples por DMV
- aderente a LGPD e ISO 27001

---

## Limitações
- não protege dados em repouso
- depende de certificado válido no Windows
- clientes legados podem falhar no handshake TLS
- CN/SAN incorreto quebra confiança do certificado
- não substitui TDE ou Always Encrypted
- problemas no certificado podem derrubar conexões seguras

---

## Melhor visão
| Cenário | Tecnologia |
|---|---|
| Rede | TLS / SSL |
| VPN / cloud | TLS / SSL |
| Credenciais | TLS / SSL |
| Linked Server | TLS / SSL |
