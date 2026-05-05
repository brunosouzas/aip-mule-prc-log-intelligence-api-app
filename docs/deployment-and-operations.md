# Deployment and Operations - AIP Process Log Intelligence API App

## Deployment Scope

This application deploys as a Mule Process API and requires connectivity to:

- `aip-mule-sys-anypoint-logs-api` (System API),
- `aip-mule-sys-observability-logs-api` (System API),
- **`aip-mule-sys-control-plane-api-app`** (System API analysis job persistence) when **`sys.controlPlane.mockMode`** is **`false`**; set **`sys.controlPlane.submitPath`** to **`/analysis-jobs`**, **`sys.controlPlane.host`** to the SYS app hostname, **`sys.controlPlane.protocol`**, **`sys.controlPlane.port`**, **`sys.controlPlane.basePath`** (typically **`/api/v1`**), and outbound **`sys.auth.*`** credentials the SYS API expects.

## Required Runtime Properties

Key properties defined under `src/main/resources/properties/`:

- listener and API metadata (`http.private.port`, `api.name`, `api.spec`),
- source collection controls (`process.logs.defaultLimit`, `process.logs.maxLimit`),
- retry controls (`process.source.*`, `process.dispatch.*`),
- downstream endpoints (`sys.anypoint.*`, `sys.observability.*`, `sys.controlPlane.*`).

## Environment Configuration Notes

- `config.yaml` defines the shared baseline.
- `config-local.yaml` enables local development overrides.
- The runtime selects the environment file via `mule.env` (for example `test`, `uat`, `prod`). If not set, it defaults to `local`.
- Production-like environments should override endpoint hosts and set **`sys.controlPlane.mockMode: "false"`** where live SYS control-plane HTTP dispatch is enabled; the outbound JSON uses **camelCase** matching the SYS RAML (**`sourceType`**, **`requestedBy`**, optional **`priority`**, **`logObjectUri`**).

## Operational Checklist

Before deployment:

1. Confirm Exchange dependency for `aip-mule-prc-log-intelligence-api-spec` is published.
2. Confirm target environment has valid API autodiscovery ID.
3. Confirm downstream hosts and TLS trust configuration are available.
4. Confirm retry and limit values match operational SLO and cost constraints.

After deployment:

1. Verify `/health-check/alive` and `/health-check/ready`.
2. Submit a `POST /analysis-jobs` request with valid idempotency key.
3. Verify `x-correlation-id` propagation and `202` accepted response.
4. Verify downstream control-plane telemetry receives dispatch payload.

## Security and Compliance

- Do not log unmasked payload values that may include secrets.
- Keep credentials in secure properties or secret stores.
- Maintain least-privilege scopes for downstream API credentials.

### Secure properties (CloudHub)

- `sys.auth.clientId` is configured as a standard (non-secure) property for traceability and operational support.
- `sys.auth.clientSecret` is injected during deployment as a CloudHub secure property and accessed in flows via `secure::sys.auth.clientSecret`.

Operational expectations:

- Runtime values are declared in `src/main/resources/cicd/secrets.yaml` as mappings from Mule property key to Azure DevOps variable name.
- Quick reference table: `docs/secrets-inventory.md` (keep aligned when `secrets.yaml` changes).
- The deployment pipeline resolves these mappings per environment (`test`/`uat`/`prod`) and fails fast when any required variable is missing or empty.
- Secure values are stored in Azure DevOps secret variables and sent to CloudHub as secure properties; this app also ships `secure-config-<env>.yaml` so Mule secure keys are explicit and aligned with platform standards — see `AI-Powered-Integration-Platform/docs/03-platform-ops/mule-secrets-and-secure-properties.md`.

## Rollback Strategy

- Roll back to previous known-good application version if dependency failures spike or orchestration regressions are detected.
- Preserve compatibility with the deployed RAML contract version during rollback decisions.
