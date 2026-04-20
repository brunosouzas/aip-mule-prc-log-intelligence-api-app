# Implementation Overview - AIP Process Log Intelligence API App

## Runtime Role

This Mule application implements the Process API runtime that orchestrates log collection and analysis dispatch workflows.

Architecture intent:

- coordinate System API calls without embedding source connector logic,
- normalise all source payloads into canonical process records,
- dispatch analysis jobs to control-plane with idempotency and traceability controls.

## High-Level Flow

1. APIkit route receives `POST /analysis-jobs` or `GET /analysis-jobs/{jobId}`.
2. Runtime sets `correlationId`, derives `idempotencyKey`, and validates request boundaries.
3. Source orchestration dispatches to supported System API channels (`anypoint`, `observability`).
4. Source payloads are normalised into canonical record structure.
5. Analysis dispatch payload is built and sent to control plane with bounded retries.
6. Orchestration summary and correlation headers are returned to the caller.

## Orchestration Components

Implemented in `impl/log-intelligence-orchestration-impl.xml`:

- `prc-validate-analysis-request-sf`: request validation (window, limit, sources, idempotency).
- `prc-enforce-idempotency-sf`: baseline duplicate key detection.
- `prc-orchestrate-source-collection-sf`: source fan-out and bounded retry handling.
- `prc-normalise-canonical-logs-sf`: canonical mapping and sensitive value masking.
- `prc-dispatch-analysis-job-sf`: control-plane event payload build and dispatch.

## Canonical Mapping Model

The process layer maps both source channels into a single canonical shape:

- `logId`, `timestamp`, `sourceSystem`, `serviceName`, `environment`,
- `severity`, `message`, `correlationId`,
- `dataClass`, `redactionApplied`.

This keeps Experience API and control-plane consumers stable when source payload shapes evolve.

## Security and Compliance Controls

Implemented controls:

- correlation IDs are propagated from inbound request to all downstream orchestration steps,
- token/secret/password patterns are masked before analysis dispatch,
- canonical payload excludes credentials and connector-specific sensitive fields.

## Non-Functional Notes

- **Reliability**: bounded `until-successful` retries for source and control-plane dependencies.
- **Observability**: source status summary and correlation headers returned in each orchestration response.
- **Cost**: collection limit guardrails avoid unbounded source pull operations.

## Cross-Repository Alignment

- Contract source of truth remains in `aip-mule-prc-log-intelligence-api-spec`.
- Runtime orchestration must stay aligned with System API contracts and control-plane event schema.
