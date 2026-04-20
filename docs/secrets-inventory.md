# Secrets inventory

Single source for variable names: `src/main/resources/cicd/secrets.yaml`. Update this file when that YAML changes.

## Runtime properties (not secret — also in `config-<env>.yaml`)

| Mule key | ADO variable (TEST) |
|----------|---------------------|
| `sys.auth.clientId` | `AIP_PRC_LOG_INTELLIGENCE_SYS_AUTH_CLIENT_ID_TEST` |

Replace `_TEST` with `_UAT` / `_PROD` for other environments.

## Secure properties

| Mule key | ADO variable (TEST) |
|----------|---------------------|
| `sys.auth.clientSecret` | `AIP_PRC_LOG_INTELLIGENCE_SYS_AUTH_CLIENT_SECRET_TEST` |

Replace `_TEST` with `_UAT` / `_PROD` for other environments.
