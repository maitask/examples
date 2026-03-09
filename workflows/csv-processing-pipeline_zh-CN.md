# CSV 处理流水线

[English](csv-processing-pipeline.md) | [中文](csv-processing-pipeline_zh-CN.md)

使用 parser + validator + 文件输出的 CSV 流水线示例。

## 适用范围

- Runtime HTTP API（`:8080`）
- 包：`@maitask/csv-parser`、`@maitask/data-validator`
- 输出适配器：`file`

## 前置条件

```bash
RUNTIME_URL="http://localhost:8080"
CSV_PKG='@maitask/csv-parser'
CSV_PKG_URL='@maitask%2Fcsv-parser'
VALIDATOR_PKG='@maitask/data-validator'
VALIDATOR_PKG_URL='@maitask%2Fdata-validator'
```

安装包：

```bash
curl -sS -X POST "${RUNTIME_URL}/packages/install" -H 'Content-Type: application/json' -d "{\"package\":\"${CSV_PKG}\"}" | jq
curl -sS -X POST "${RUNTIME_URL}/packages/install" -H 'Content-Type: application/json' -d "{\"package\":\"${VALIDATOR_PKG}\"}" | jq
```

## 步骤

1. 解析 CSV。

```bash
cat > /tmp/sales.csv <<'CSV'
name,email,amount,region
Alice,alice@example.com,1200,US
Bob,bob@example.com,900,EU
Carol,carol@example.com,1800,APAC
CSV

curl -sS -X POST "${RUNTIME_URL}/packages/${CSV_PKG_URL}/execute" \
  -H 'Content-Type: application/json' \
  -d "{\"input\":{\"text\":$(jq -Rs . < /tmp/sales.csv)}}" \
  > /tmp/csv_parse.json
```

2. 校验解析结果。

```bash
curl -sS -X POST "${RUNTIME_URL}/packages/${VALIDATOR_PKG_URL}/execute" \
  -H 'Content-Type: application/json' \
  -d "{\"input\":$(jq '.data.result.rows' /tmp/csv_parse.json),\"options\":{\"validation_mode\":\"report-only\",\"required_fields\":[\"name\",\"email\",\"amount\"],\"custom_rules\":[{\"field\":\"email\",\"rule\":\"email\"},{\"field\":\"amount\",\"rule\":\"numeric\"}]}}" \
  > /tmp/csv_validate.json
```

3. 持久化校验响应。

```bash
curl -sS -X POST "${RUNTIME_URL}/output" \
  -H 'Content-Type: application/json' \
  -d "{\"adapter\":\"file\",\"data\":$(cat /tmp/csv_validate.json),\"config\":{\"path\":\"/tmp/maitask-csv-validation-report.json\",\"format\":\"json_pretty\"}}" \
  | jq
```

## 验证

```bash
jq -e '.success == true' /tmp/csv_parse.json
jq -e '.success == true' /tmp/csv_validate.json
jq -e '.success == true' /tmp/maitask-csv-validation-report.json
```

## 边界说明

- 本示例主要验证 Runtime 编排与适配器链路。
- 调度、策略与租户治理由 Plane 负责。
