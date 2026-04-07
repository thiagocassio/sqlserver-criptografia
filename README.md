# Criptografias no SQL Server

Documentação prática e objetiva sobre os principais recursos de **criptografia e proteção de dados no SQL Server**, com foco em:

* estudo
* laboratório
* testes reais
* validação técnica
* evidências para auditoria
* uso em produção

O material foi organizado por **tópicos separados em pastas**, facilitando leitura, versionamento e evolução contínua no GitHub.

---

## Estrutura do repositório

```text
01-tde
02-always-encrypted
03-column-level-encryption
04-backup-encryption
05-hashing
06-tls-ssl
07-ekm
```

Cada pasta contém:

* conceito objetivo
* quando usar
* implementação prática
* validação
* teste real
* vantagens
* limitações

---

## Tópicos

### 01 — TDE

Proteção de **dados em repouso**, incluindo MDF, LDF, NDF e backups.

### 02 — Always Encrypted

Proteção de **dados sensíveis até mesmo contra DBA**, com chave no cliente.

### 03 — Criptografia em nível de coluna

Proteção de **colunas específicas** usando chave simétrica e certificado.

### 04 — Backup Encryption

Proteção do arquivo **.bak** para restore seguro e DR externo.

### 05 — Hashing

Integridade, comparação segura e **pseudonimização irreversível**.

### 06 — TLS / SSL

Proteção de **dados em trânsito** entre cliente e SQL Server.

### 07 — EKM

Integração com **Azure Key Vault / HSM**, mantendo a chave fora do SQL Server.

---

## Objetivo do projeto

Este repositório foi criado para servir como:

* material de estudo
* laboratório de testes
* referência profissional para DBA
* documentação para blog técnico
* base para auditoria e compliance
* portfolio de segurança SQL Server

O foco é mostrar **como implementar e validar de verdade**, sem exemplos apenas conceituais.

---

## Melhor visão

| Cenário           | Tecnologia                      |
| ----------------- | ------------------------------- |
| Arquivo           | TDE                             |
| Dado sensível     | Always Encrypted                |
| Coluna específica | Criptografia em nível de coluna |
| Backup / mídia    | Backup Encryption               |
| Integridade       | Hashing                         |
| Rede              | TLS / SSL                       |
| Chave externa     | EKM                             |

---

## Próximas evoluções

* converter todos os tópicos para markdown individual
* adicionar imagens dos fluxos no SSMS
* criar laboratório por versão do SQL Server
* adicionar troubleshooting e erros comuns
* incluir cenários de LGPD e ISO 27001

---

> Repositório focado em **DBA SQL Server | Segurança | Criptografia | Auditoria | Laboratório real**.
