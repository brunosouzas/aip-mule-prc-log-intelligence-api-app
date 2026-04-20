# Error Handling and Retries - AIP Process Log Intelligence API App

## Canonical Error Model

The API runtime maps failures into a canonical response payload:

- `error_code`
- `error_message`
- `error_category`
- `correlation_id`
- `timestamp`
- `retryable`

`x-correlation-id` is always returned in response headers when available.

## Error Categories

- **Validation (`400`)**
  - Missing `idempotencyKey`,
  - invalid filter window or over-limit requests,
  - unsupported source values.
- **Idempotency conflict (`409`)**
  - duplicate idempotency key detected in active retention window.
- **Dependency (`503`)**
  - source System API connectivity or timeout failures,
  - control-plane dispatch dependency failures.
- **Internal (`500`)**
  - unexpected runtime or mapping failures.

## Retry Strategy

Source collection retries:

- Implemented via `until-successful` in `prc-orchestrate-source-collection-sf`.
- Controlled by:
  - `process.source.maxRetries`
  - `process.source.retryMillis`

Control-plane dispatch retries:

- Implemented via `until-successful` in `prc-dispatch-analysis-job-sf`.
- Controlled by:
  - `process.dispatch.maxRetries`
  - `process.dispatch.retryMillis`

## Retry Rules

- Retry only transient dependency failures.
- Do not retry validation or idempotency failures.
- Record source-level failures in `sourceStatuses` and set `partialResults=true` when policy allows continuation.

## Operational Guidance

- Monitor `503` error rates as leading indicators of downstream instability.
- Tune retries conservatively to balance reliability and latency/cost.
- Escalate repeated dependency failures before increasing retry counts.
