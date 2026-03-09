# 监控与告警

[English](monitoring-alerts.md) | [中文](monitoring-alerts_zh-CN.md)

监控流水线：阈值校验 + 可选通知。

## 适用范围

- Runtime 包执行
- 通过 `@maitask/data-validator` 做阈值校验
- 可选 Slack/邮件通知
- 文件落地作为审计产物

## 前置条件

```bash
RUNTIME_URL="http://localhost:8080"
VALIDATOR_PKG='@maitask/data-validator'
VALIDATOR_PKG_URL='@maitask%2Fdata-validator'
SLACK_PKG_URL='@maitask%2Fslack-notifier'
EMAIL_PKG_URL='@maitask%2Femail-sender'
```

可选凭据：

```bash
export SLACK_WEBHOOK_URL="https://hooks.slack.com/services/..."
export SENDGRID_API_KEY="<your-sendgrid-api-key>"
```

安装所需包：

```bash
for pkg in '@maitask/data-validator' '@maitask/slack-notifier' '@maitask/email-sender'; do
  curl -sS -X POST "${RUNTIME_URL}/packages/install" \
    -H 'Content-Type: application/json' \
    -d "{\"package\":\"${pkg}\"}" | jq
 done
```

## 步骤

1. 生成指标快照。

```bash
cat > /tmp/system_metrics.json <<'JSON'
{"host":"prod-api-01","cpu_usage":86,"memory_usage":74,"response_time_ms":240}
JSON
```

2. 校验阈值越界。

```bash
curl -sS -X POST "${RUNTIME_URL}/packages/${VALIDATOR_PKG_URL}/execute" \
  -H 'Content-Type: application/json' \
  -d "{\"input\":[$(cat /tmp/system_metrics.json)],\"options\":{\"validation_mode\":\"report-only\",\"custom_rules\":[{\"field\":\"cpu_usage\",\"rule\":\"numeric\"},{\"field\":\"memory_usage\",\"rule\":\"numeric\"},{\"field\":\"response_time_ms\",\"rule\":\"numeric\"}]}}" \
  > /tmp/monitor_validation.json
```

3. 生成标准化告警载荷。

```bash
jq -n \
  --arg ts "$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
  --argjson metrics "$(cat /tmp/system_metrics.json)" \
  --argjson validation "$(cat /tmp/monitor_validation.json)" \
  '{timestamp:$ts,metrics:$metrics,validation:$validation,severity:(if $metrics.cpu_usage>=80 or $metrics.response_time_ms>=200 then "warning" else "ok" end)}' \
  > /tmp/monitor_alert_payload.json
```

4. 持久化告警载荷。

```bash
curl -sS -X POST "${RUNTIME_URL}/output" \
  -H 'Content-Type: application/json' \
  -d "{\"adapter\":\"file\",\"data\":$(cat /tmp/monitor_alert_payload.json),\"config\":{\"path\":\"/tmp/monitor_alert_payload_saved.json\",\"format\":\"json_pretty\"}}" \
  | jq
```

5. 可选：发送 Slack 通知。

```bash
if [ -n "$SLACK_WEBHOOK_URL" ]; then
  curl -sS -X POST "${RUNTIME_URL}/packages/${SLACK_PKG_URL}/execute" \
    -H 'Content-Type: application/json' \
    -d "{\"input\":{\"webhook_url\":\"${SLACK_WEBHOOK_URL}\",\"text\":\"[Maitask] monitor warning: prod-api-01 cpu=86 rt=240ms\"}}" \
    > /tmp/monitor_slack_response.json
fi
```

6. 可选：发送邮件通知。

```bash
if [ -n "$SENDGRID_API_KEY" ]; then
  jq -n --arg key "$SENDGRID_API_KEY" '{
    input: {
      provider: "sendgrid",
      api_key: $key,
      from: { email: "alerts@example.com", name: "Maitask Monitor" },
      to: [{ email: "oncall@example.com", name: "Oncall" }],
      subject: "[Maitask] Monitor warning",
      text: "prod-api-01 exceeded threshold: cpu=86, response_time_ms=240"
    }
  }' > /tmp/monitor_email_request.json

  curl -sS -X POST "${RUNTIME_URL}/packages/${EMAIL_PKG_URL}/execute" \
    -H 'Content-Type: application/json' \
    -d @/tmp/monitor_email_request.json \
    > /tmp/monitor_email_response.json
fi
```

## 验证

```bash
jq -e '.success == true' /tmp/monitor_validation.json
jq -e '.success == true' /tmp/monitor_alert_payload_saved.json
```

## 边界说明

- 本示例采用阈值规则，刻意保持简单。
- 生产告警策略与升级路径应由 Plane 的 workflow/schedule 统一托管。
