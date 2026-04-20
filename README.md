# AIP Mule Process Log Intelligence API - Application

This repository contains the Mule runtime implementation for the `aip-mule-prc-log-intelligence-api` Process API.

## Purpose

Orchestrate canonical log collection through System APIs and dispatch analysis jobs to the AI control plane.

## Runtime Scope

- Implement APIkit routes defined by `aip-mule-prc-log-intelligence-api-spec`.
- Coordinate source collection from Process-owned orchestration flows.
- Normalise payloads into canonical records for control-plane submission.
- Enforce idempotency, correlation propagation, and explicit retry/error behaviour.

## Key Runtime Components

- `src/main/mule/common/global-config.xml`: listener, APIkit, runtime properties, and downstream HTTP configs.
- `src/main/mule/common/common-error.xml`: canonical error mapping with retryability signalling.
- `src/main/mule/api-main.xml`: endpoint flows for orchestration resources and health-checks.
- `src/main/mule/impl/log-intelligence-orchestration-impl.xml`: source dispatch, mapping, idempotency, and control-plane dispatch flow.

## Required Documentation

- `docs/implementation-overview.md`
- `docs/deployment-and-operations.md`
- `docs/error-handling-and-retries.md`
- `docs/c4-model-diagrams.md`
- `docs/secrets-inventory.md` (ADO variable names — update when `cicd/secrets.yaml` changes)

## Cross-Repository Alignment

Contract changes originate in `aip-mule-prc-log-intelligence-api-spec` and must be implemented here without exposing source connector specifics that belong to System APIs.
