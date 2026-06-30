# Maitask 示例

[English](README.md) | [中文](README_zh-CN.md)

面向 Maitask 平台的能力对齐示例。

## 适用范围

本目录示例与以下主项目的当前职责对齐：

- `runtime`：Package 执行引擎（原生二进制 worker、HTTP/gRPC、输出适配器）
- `plane`：控制面（认证、包元数据、编排、credits/payments）
- `plane-frontend`：基于 Plane/Runtime API 的控制台 UI

能力基线：已于 2026-03-08 对照 `../runtime`、`../plane`、`../plane-frontend` 校对。

## 能力边界

- 当前 Runtime 构建仅支持通过注册表包名安装包。
- 生产环境通常将 Runtime 置于内网，Plane 作为对外 API 边界。
- Plane 可在执行请求中内联 `code` 与 `version`，以无状态方式驱动 Runtime。
- 前端路由与交互归属 `plane-frontend`；本目录仅提供后端/API 导向示例。

## 环境基线

1. Runtime API：`http://localhost:8080`
2. Plane API：`http://localhost:8001`
3. Plane Frontend（可选）：`http://localhost:3000`
4. 工具：`curl`、`jq`

健康检查：

```bash
curl -sS http://localhost:8080/health | jq
curl -sS http://localhost:8001/health | jq
```

## 示例索引

| 领域 | English | 中文 |
|---|---|---|
| Workflow | [Simple Data Processing](workflows/simple-data-processing.md) | [简易数据处理](workflows/simple-data-processing_zh-CN.md) |
| Workflow | [CSV Processing Pipeline](workflows/csv-processing-pipeline.md) | [CSV 处理流水线](workflows/csv-processing-pipeline_zh-CN.md) |
| Workflow | [Web Data Collection](workflows/web-data-collection.md) | [Web 数据采集](workflows/web-data-collection_zh-CN.md) |
| Package | [Email Notifications](packages/email-notifications.md) | [邮件通知](packages/email-notifications_zh-CN.md) |
| Data Processing | [Customer Scoring](data-processing/customer-scoring.md) | [客户评分](data-processing/customer-scoring_zh-CN.md) |
| Automation | [Monitoring and Alerts](automation/monitoring-alerts.md) | [监控与告警](automation/monitoring-alerts_zh-CN.md) |
| Deployment | [Control Plane Compose](docker-compose/communication-modes.yml) | [控制面 Compose](docker-compose/communication-modes.yml) |

## Scoped 包名说明

官方包使用 scoped 名称，例如 `@maitask/csv-parser`。

通过 Runtime HTTP 路由调用时，需要将 `/` 编码为 `%2F`：

```bash
PKG='@maitask%2Fcsv-parser'
curl -sS -X POST "http://localhost:8080/packages/${PKG}/execute" \
  -H 'Content-Type: application/json' \
  -d '{"input":{"text":"name,age\nAlice,30"}}' | jq
```

## 变更策略

当 `plane`、`plane-frontend` 或 `runtime` 能力发生变化时，本目录需在同一提交中同步更新中英文文档。
