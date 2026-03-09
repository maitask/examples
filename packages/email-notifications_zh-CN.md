# 邮件通知

[English](email-notifications.md) | [中文](email-notifications_zh-CN.md)

面向 `@maitask/email-sender` 的包级示例。

## 适用范围

- Runtime 包执行 API
- 基于邮件服务商的外发链路
- 不依赖工作流调度器

## 前置条件

```bash
RUNTIME_URL="http://localhost:8080"
EMAIL_PKG='@maitask/email-sender'
EMAIL_PKG_URL='@maitask%2Femail-sender'
export SENDGRID_API_KEY="<your-sendgrid-api-key>"
```

安装包：

```bash
curl -sS -X POST "${RUNTIME_URL}/packages/install" \
  -H 'Content-Type: application/json' \
  -d "{\"package\":\"${EMAIL_PKG}\"}" | jq
```

## 步骤

1. 生成请求体。

```bash
jq -n --arg key "$SENDGRID_API_KEY" '{
  input: {
    provider: "sendgrid",
    api_key: $key,
    from: { email: "noreply@example.com", name: "Maitask Example" },
    to: [{ email: "receiver@example.com", name: "Receiver" }],
    subject: "Maitask Email Notification Test",
    text: "This is a test notification from Maitask examples."
  }
}' > /tmp/email_request.json
```

2. 执行包。

```bash
curl -sS -X POST "${RUNTIME_URL}/packages/${EMAIL_PKG_URL}/execute" \
  -H 'Content-Type: application/json' \
  -d @/tmp/email_request.json \
  > /tmp/email_response.json
```

## 验证

```bash
jq -e '.success == true' /tmp/email_response.json
jq '.data.result' /tmp/email_response.json
```

## 边界说明

- 投递成功受服务商凭据、发信域策略与额度影响。
- 生产环境建议将该包纳入 Plane 的 workflow/schedule 策略统一治理。
