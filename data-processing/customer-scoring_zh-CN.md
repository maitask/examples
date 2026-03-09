# 客户评分

[English](customer-scoring.md) | [中文](customer-scoring_zh-CN.md)

评分流程：先用 Runtime 包校验输入，再生成确定性评分。

## 适用范围

- Runtime 包执行（`@maitask/data-validator`）
- 本地确定性评分转换
- 使用文件适配器落地报告

## 前置条件

```bash
RUNTIME_URL="http://localhost:8080"
VALIDATOR_PKG='@maitask/data-validator'
VALIDATOR_PKG_URL='@maitask%2Fdata-validator'
```

安装包：

```bash
curl -sS -X POST "${RUNTIME_URL}/packages/install" \
  -H 'Content-Type: application/json' \
  -d "{\"package\":\"${VALIDATOR_PKG}\"}" | jq
```

## 步骤

1. 准备客户样本数据。

```bash
cat > /tmp/customers.json <<'JSON'
[
  {"customer_id":"C001","transactions":35,"total_amount":8200,"days_since_last":8},
  {"customer_id":"C002","transactions":6,"total_amount":560,"days_since_last":120},
  {"customer_id":"C003","transactions":18,"total_amount":3100,"days_since_last":42}
]
JSON
```

2. 校验必填字段与基础数值格式。

```bash
curl -sS -X POST "${RUNTIME_URL}/packages/${VALIDATOR_PKG_URL}/execute" \
  -H 'Content-Type: application/json' \
  -d "{\"input\":$(cat /tmp/customers.json),\"options\":{\"validation_mode\":\"strict\",\"required_fields\":[\"customer_id\",\"transactions\",\"total_amount\",\"days_since_last\"],\"custom_rules\":[{\"field\":\"transactions\",\"rule\":\"numeric\"},{\"field\":\"total_amount\",\"rule\":\"numeric\"}]}}" \
  > /tmp/customer_validation.json
```

3. 计算评分与分层。

```bash
jq 'map(. + {
  score: ((.transactions * 1.2) + (.total_amount / 200) - (.days_since_last * 0.3)) | floor,
  tier: (
    ((.transactions * 1.2) + (.total_amount / 200) - (.days_since_last * 0.3)) as $s |
    if $s >= 80 then "A"
    elif $s >= 50 then "B"
    else "C" end
  )
})' /tmp/customers.json > /tmp/customer_scoring_result.json
```

4. 持久化评分输出。

```bash
curl -sS -X POST "${RUNTIME_URL}/output" \
  -H 'Content-Type: application/json' \
  -d "{\"adapter\":\"file\",\"data\":$(cat /tmp/customer_scoring_result.json),\"config\":{\"path\":\"/tmp/customer_scoring_report.json\",\"format\":\"json_pretty\"}}" \
  | jq
```

## 验证

```bash
jq -e '.success == true' /tmp/customer_validation.json
jq -e 'length > 0' /tmp/customer_scoring_result.json
jq -e '.success == true' /tmp/customer_scoring_report.json
```

## 生产说明

若需多租户与可审计评分逻辑，建议将评分包发布到 Plane，并通过 Plane 工作流执行。
