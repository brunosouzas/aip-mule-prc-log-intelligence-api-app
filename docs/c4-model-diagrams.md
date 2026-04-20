# C4 Model Diagrams - AIP Process Log Intelligence API Application

## Purpose

Provide implementation-centred architecture diagrams for this Mule Process API runtime so engineers can understand orchestration boundaries, runtime dependencies, and operational behaviour.

## C4 Level 1 - System Context

```mermaid
flowchart LR
    exp["Experience API / API Consumers"]
    sysAny["aip-mule-sys-anypoint-logs-api-app"]
    sysObs["aip-mule-sys-observability-logs-api-app"]
    control["aip-control-plane-go"]
    ops["Platform Operations"]

    app["aip-mule-prc-log-intelligence-api-app<br/>(Mule Process API Runtime)"]

    exp -->|"POST/GET analysis jobs"| app
    app -->|"Collect source logs"| sysAny
    app -->|"Collect source logs"| sysObs
    app -->|"Dispatch analysis.debug.requested.v1 payload"| control
    ops -->|"Monitor runtime and errors"| app
```

## C4 Level 2 - Container View

```mermaid
flowchart LR
    caller["Experience API / Clients"]
    sysAny["System API: Anypoint Logs"]
    sysObs["System API: Observability Logs"]
    control["Control Plane API"]

    subgraph muleApp["aip-mule-prc-log-intelligence-api-app"]
        listener["HTTP Listener + APIkit Router<br/>(api-main.xml)"]
        orchestration["Orchestration Sub-Flows<br/>(log-intelligence-orchestration-impl.xml)"]
        errorMap["Common Error Handler<br/>(common-error.xml)"]
        config["Global Config and Properties<br/>(global-config.xml + config.yaml)"]
    end

    caller --> listener
    listener --> orchestration
    orchestration --> sysAny
    orchestration --> sysObs
    orchestration --> control
    orchestration --> errorMap
    config --> listener
    config --> orchestration
```

## C4 Level 3 - Component View (Orchestration Flows)

```mermaid
flowchart TD
    submit["post:\\analysis-jobs:apikit_config"]
    validate["prc-validate-analysis-request-sf"]
    idempotency["prc-enforce-idempotency-sf"]
    collect["prc-orchestrate-source-collection-sf"]
    collectSwitch["prc-collect-source-logs-sf"]
    anypoint["prc-collect-from-anypoint-sf"]
    observability["prc-collect-from-observability-sf"]
    normalise["prc-normalise-canonical-logs-sf"]
    dispatch["prc-dispatch-analysis-job-sf"]
    buildReq["prc-build-control-plane-request-sf"]
    sendReq["prc-send-control-plane-request-sf"]
    status["prc-get-analysis-status-sf"]
    getFlow["get:\\analysis-jobs\\(jobid\\):apikit_config"]

    submit --> validate
    validate --> idempotency
    idempotency --> collect
    collect --> collectSwitch
    collectSwitch --> anypoint
    collectSwitch --> observability
    anypoint --> normalise
    observability --> normalise
    normalise --> dispatch
    dispatch --> buildReq
    buildReq --> sendReq

    getFlow --> status
```

## Main Runtime Sequence - Analysis Job Submission

```mermaid
sequenceDiagram
    participant Client as Experience API/Client
    participant ApiMain as APIkit POST /analysis-jobs
    participant Validate as Validate + Idempotency
    participant Sources as Source Collection Orchestration
    participant Canonical as Canonical Mapping
    participant Dispatch as Control-Plane Dispatch
    participant Response as API Response Builder

    Client->>ApiMain: POST /analysis-jobs
    ApiMain->>Validate: Validate payload and idempotencyKey
    Validate->>Sources: Collect selected source systems
    Sources->>Canonical: Normalise and mask sensitive values
    Canonical->>Dispatch: Build analysis.debug.requested.v1 payload
    Dispatch-->>Response: jobId + dispatch metadata
    Response-->>Client: 202 Accepted + correlationId
```

## Update Triggers

Update these diagrams whenever APIkit routes, orchestration sub-flow names, dependency endpoints, retry strategy, or error handling behaviour changes.
