---
layout: "enterprise2"
page_title: "Private Terraform Enterprise Security"
sidebar_current: "docs-enterprise2-private-data-security"
---

# Private Terraform Enterprise - Security

Private Terraform Enterprise (PTFE) takes the security of the data it manages
seriously. This table lists which parts of the PTFE app can contain sensitive data, what storage is used, and what encryption is used.

| Object                               | Storage       | Encrypted                             |
|:-------------------------------------|:--------------|:--------------------------------------|
| Ingressed VCS Data                   | Blob Storage  | Vault Transit Encryption              |
| Terraform Plan Result                | Blob Storage  | Vault Transit Encryption              |
| Terraform State                      | Blob Storage  | Vault Transit Encryption              |
| Terraform Logs                       | Blob Storage  | Vault Transit Encryption              |
| Terraform/Environment Variables      | PostgreSQL    | Vault Transit Encryption              |
| Organization/Workspace/Team Settings | PostgreSQL    | No                                    |
| Account Password                     | PostgreSQL    | bcrypt                                |
| 2FA Recovery Codes                   | PostgreSQL    | Vault Transit Encryption              |
| SSH Keys                             | PostgreSQL    | Vault Transit Encryption              |
| User/Team/Organization Tokens        | PostgreSQL    | HMAC SHA512                           |
| OAuth Client ID + Secret             | PostgreSQL    | Vault Transit Encryption              |
| OAuth User Tokens                    | PostgreSQL    | Vault Transit Encryption              |
| Twilio Account Configuration         | PostgreSQL    | Vault Transit Encryption              |
| SMTP Configuration                   | PostgreSQL    | Vault Transit Encryption              |
| SAML Configuration                   | PostgreSQL    | Vault Transit Encryption              |
| Vault Unseal Key                     | Host          | No                                    |

## Vault Transit Encryption
The [Vault Transit Secret Engine](https://www.vaultproject.io/docs/secrets/transit/index.html) handles encryption for data in-transit and is used when encrypting data from the application to the applicable [storage layer](https://www.terraform.io/docs/enterprise/private/reliability-availability.html#components).