# 简易数据处理

[English](simple-data-processing.md) | [中文](simple-data-processing_zh-CN.md)

使用一个解析包和一个输出适配器完成最小 Runtime 冒烟验证。

## 适用范围

- 仅使用 Runtime HTTP API（`:8080`）
- 验证包执行链路与输出适配器写入链路
- 不依赖 Plane 鉴权、计费或前端

## 前置条件

```bash
RUNTIME_URL="http://localhost:8080"
CSV_PKG='@maitask/csv-parser'
CSV_PKG_URL='@maitask%2Fcsv-parser'
```

安装包：

```bash
curl -sS -X POST "${RUNTIME_URL}/packages/install" \
  -H 'Content-Type: application/json' \
  -d "{\"package\":\"${CSV_PKG}\"}" | jq
```

## 步骤

1. 解析内联 CSV。

```bash
cat > /tmp/simple_customers.csv <<'CSV'
name,email,age
Alice,alice@example.com,30
Bob,bob@example.com,27
CSV

curl -sS -X POST "${RUNTIME_URL}/packages/${CSV_PKG_URL}/execute" \
  -H 'Content-Type: application/json' \
  -d "{\"input\":{\"text\":$(jq -Rs . < /tmp/simple_customers.csv)}}" \
  > /tmp/simple_parse_response.json
```

2. 通过输出适配器接口将执行结果写入本地文件。

```bash
curl -sS -X POST "${RUNTIME_URL}/output" \
  -H 'Content-Type: application/json' \
  -d "{\"adapter\":\"file\",\"data\":$(cat /tmp/simple_parse_response.json),\"config\":{\"path\":\"/tmp/maitask-simple-output.json\",\"format\":\"json_pretty\"}}" \
  | jq
```

## 验证

```bash
jq -e '.success == true' /tmp/simple_parse_response.json
jq -e '.success == true' /tmp/maitask-simple-output.json
```

## 边界说明

- 本示例仅覆盖 Runtime 本地能力。
- 生产环境的 API 治理与鉴权请通过 Plane。
