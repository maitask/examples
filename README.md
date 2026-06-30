# Maitask Examples

[English](README.md) | [中文](README_zh-CN.md)

Capability-aligned examples for the Maitask platform.

## Scope

These examples are aligned with the current responsibilities of:

- `runtime`: package execution engine (native binary worker, HTTP/gRPC, output adapters)
- `plane`: control plane (auth, package metadata, orchestration, credits/payments)
- `plane-frontend`: console UI on top of Plane/Runtime APIs

Validation baseline: reviewed against `../runtime`, `../plane`, and `../plane-frontend` on 2026-03-08.

## Capability Boundaries

- Runtime package install supports registry package names only in this build.
- Runtime is typically internal in production; Plane is the public API boundary.
- Plane can execute packages statelessly by sending `code` and `version` inline to Runtime.
- Frontend routes and UX are managed in `plane-frontend`; this directory only contains backend/API-oriented examples.

## Environment Baseline

1. Runtime API: `http://localhost:8080`
2. Plane API: `http://localhost:8001`
3. Plane Frontend (optional): `http://localhost:3000`
4. Tools: `curl`, `jq`

Health checks:

```bash
curl -sS http://localhost:8080/health | jq
curl -sS http://localhost:8001/health | jq
```

## Example Index

| Domain | English | 中文 |
|---|---|---|
| Workflow | [Simple Data Processing](workflows/simple-data-processing.md) | [简易数据处理](workflows/simple-data-processing_zh-CN.md) |
| Workflow | [CSV Processing Pipeline](workflows/csv-processing-pipeline.md) | [CSV 处理流水线](workflows/csv-processing-pipeline_zh-CN.md) |
| Workflow | [Web Data Collection](workflows/web-data-collection.md) | [Web 数据采集](workflows/web-data-collection_zh-CN.md) |
| Package | [Email Notifications](packages/email-notifications.md) | [邮件通知](packages/email-notifications_zh-CN.md) |
| Data Processing | [Customer Scoring](data-processing/customer-scoring.md) | [客户评分](data-processing/customer-scoring_zh-CN.md) |
| Automation | [Monitoring and Alerts](automation/monitoring-alerts.md) | [监控与告警](automation/monitoring-alerts_zh-CN.md) |
| Deployment | [Control Plane Compose](docker-compose/communication-modes.yml) | [控制面 Compose](docker-compose/communication-modes.yml) |

## Naming Note for Scoped Packages

Official packages use scoped names such as `@maitask/csv-parser`.

When calling Runtime HTTP routes with package names, URL-encode `/` as `%2F`:

```bash
PKG='@maitask%2Fcsv-parser'
curl -sS -X POST "http://localhost:8080/packages/${PKG}/execute" \
  -H 'Content-Type: application/json' \
  -d '{"input":{"text":"name,age\nAlice,30"}}' | jq
```

## Change Policy

When capabilities change in `plane`, `plane-frontend`, or `runtime`, update this directory in both English and Chinese in the same commit.
