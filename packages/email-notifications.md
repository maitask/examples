# Email Notifications Examples

Examples of using the email-sender package for various notification scenarios.

> **Note**: Email-sender package requires network connectivity and HTTP functionality. Ensure your environment supports outbound HTTP requests and configure proxy settings if needed (see [Package Installation](#proxy-support) in the main README).

## Prerequisites

```bash
# Install email sender package
maitask install email-sender

# Set proxy environment variables if needed
export HTTP_PROXY=http://proxy.company.com:8080
export HTTPS_PROXY=http://proxy.company.com:8080
```

## Example 1: Data Validation Report

### Process Data and Send Report

```bash
# 1. Validate some data first
curl -X POST "http://localhost:8080/packages/data-validator/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "data": [
        {"user_id": "U001", "email": "user1@example.com", "score": 85},
        {"user_id": "U002", "email": "invalid-email", "score": 92},
        {"user_id": "U003", "email": "user3@example.com", "score": 78}
      ]
    },
    "options": {
      "validation_mode": "report-only",
      "custom_rules": [
        {"field": "email", "rule": "email"},
        {"field": "score", "rule": "range", "min": 0, "max": 100}
      ]
    }
  }' > examples/data/validation_report.json

# 2. Send validation report via email
curl -X POST "http://localhost:8080/packages/email-sender/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": '$(cat examples/data/validation_report.json)',
    "options": {
      "provider": "sendgrid",
      "api_key": "YOUR_SENDGRID_API_KEY",
      "from": {
        "email": "reports@company.com",
        "name": "Data Validation System"
      },
      "to": [
        {"email": "admin@company.com", "name": "Admin"},
        {"email": "data-team@company.com", "name": "Data Team"}
      ],
      "subject": "Daily Data Validation Report - '$(date +%Y-%m-%d)'",
      "template": "validation-report",
      "template_data": {
        "date": "'$(date +%Y-%m-%d)'",
        "total_records": 3,
        "valid_records": 2,
        "invalid_records": 1,
        "validation_summary": "2 out of 3 records passed validation"
      }
    }
  }'
```

## Example 2: System Alert Email

### SMTP Configuration

```bash
curl -X POST "http://localhost:8080/packages/email-sender/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "alert_data": {
        "system": "production-server-01",
        "metric": "cpu_usage",
        "value": 95,
        "threshold": 80,
        "timestamp": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"
      }
    },
    "options": {
      "provider": "smtp",
      "smtp_config": {
        "host": "smtp.company.com",
        "port": 587,
        "username": "alerts@company.com",
        "password": "YOUR_SMTP_PASSWORD",
        "secure": true
      },
      "from": {
        "email": "alerts@company.com",
        "name": "System Monitoring"
      },
      "to": [
        {"email": "oncall@company.com", "name": "On-Call Team"}
      ],
      "subject": "🚨 CRITICAL ALERT: High CPU Usage on production-server-01",
      "html": "<h2>System Alert</h2><p><strong>Server:</strong> production-server-01</p><p><strong>Metric:</strong> CPU Usage</p><p><strong>Current Value:</strong> 95%</p><p><strong>Threshold:</strong> 80%</p><p><strong>Time:</strong> '$(date)'</p>",
      "text": "CRITICAL ALERT: production-server-01 CPU usage is at 95%, exceeding the 80% threshold."
    }
  }'
```

## Example 3: Welcome Email Template

### Mailgun Provider

```bash
curl -X POST "http://localhost:8080/packages/email-sender/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "new_user": {
        "name": "John Doe",
        "email": "john.doe@example.com",
        "signup_date": "'$(date +%Y-%m-%d)'",
        "account_type": "premium"
      }
    },
    "options": {
      "provider": "mailgun",
      "api_key": "YOUR_MAILGUN_API_KEY",
      "from": {
        "email": "welcome@company.com",
        "name": "Company Welcome Team"
      },
      "to": [
        {"email": "john.doe@example.com", "name": "John Doe"}
      ],
      "subject": "Welcome to Company - Your Premium Account is Ready!",
      "template": "welcome-premium",
      "template_data": {
        "user_name": "John",
        "account_type": "Premium",
        "signup_date": "'$(date +%B\ %d,\ %Y)'",
        "login_url": "https://app.company.com/login",
        "support_email": "support@company.com"
      }
    }
  }'
```

## Example 4: Batch Email Notifications

### Process Multiple Recipients

```bash
# Create sample customer data
cat > examples/data/customers.json << 'EOF'
{
  "customers": [
    {"name": "Alice Johnson", "email": "alice@example.com", "status": "active", "last_login": "2024-01-15"},
    {"name": "Bob Smith", "email": "bob@example.com", "status": "inactive", "last_login": "2023-12-01"},
    {"name": "Carol White", "email": "carol@example.com", "status": "active", "last_login": "2024-01-18"}
  ]
}
EOF

# Send personalized emails to each customer
curl -X POST "http://localhost:8080/packages/email-sender/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": '$(cat examples/data/customers.json)',
    "options": {
      "provider": "sendgrid",
      "api_key": "YOUR_SENDGRID_API_KEY",
      "from": {
        "email": "marketing@company.com",
        "name": "Company Marketing"
      },
      "batch_mode": true,
      "email_template": {
        "subject": "{{#if (eq status \"inactive\")}}We miss you, {{name}}!{{else}}Thanks for being active, {{name}}!{{/if}}",
        "template": "customer-engagement",
        "personalization": {
          "name": "{{name}}",
          "status": "{{status}}",
          "last_login_formatted": "{{last_login}}",
          "reactivation_offer": "{{#if (eq status \"inactive\")}}Special 20% discount just for you!{{/if}}"
        }
      }
    }
  }'
```

## Complete Email Workflow Script

```bash
#!/bin/bash
# examples/packages/run-email-workflow.sh

echo "=== Email Notification Workflow ==="

# 1. Generate daily report
echo "1. Generating daily report..."
curl -s -X POST "http://localhost:8080/packages/data-validator/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {"data": [{"id": 1, "status": "processed"}, {"id": 2, "status": "failed"}]},
    "options": {"validation_mode": "report-only"}
  }' | jq '.data.result' > examples/data/daily_report.json

# 2. Send report to team
echo "2. Sending report to team..."
curl -s -X POST "http://localhost:8080/packages/email-sender/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": '$(cat examples/data/daily_report.json)',
    "options": {
      "provider": "sendgrid",
      "api_key": "'$SENDGRID_API_KEY'",
      "from": {"email": "reports@company.com", "name": "Daily Reports"},
      "to": [{"email": "team@company.com", "name": "Team"}],
      "subject": "Daily Processing Report - '$(date +%Y-%m-%d)'",
      "html": "<h2>Daily Report</h2><p>Report data processed successfully.</p>"
    }
  }'

# 3. Check for alerts and send if needed
echo "3. Checking for alerts..."
ALERT_COUNT=$(curl -s "http://localhost:8080/packages/system-monitor/execute" | jq '.alerts | length')

if [ "$ALERT_COUNT" -gt 0 ]; then
  echo "Sending alert email..."
  curl -s -X POST "http://localhost:8080/packages/email-sender/execute" \
    -H "Content-Type: application/json" \
    -d '{
      "options": {
        "provider": "smtp",
        "from": {"email": "alerts@company.com"},
        "to": [{"email": "oncall@company.com"}],
        "subject": "System Alerts Detected",
        "text": "System monitoring has detected '$ALERT_COUNT' alerts that require attention."
      }
    }'
fi

echo "=== Email Workflow Complete ==="
```

## Configuration Tips

### Environment Variables

```bash
# Set up environment variables for email credentials
export SENDGRID_API_KEY="your_sendgrid_key"
export MAILGUN_API_KEY="your_mailgun_key"
export SMTP_PASSWORD="your_smtp_password"
```

### Template Management

Store email templates in separate files:

```bash
# examples/data/templates/alert-template.html
cat > examples/data/templates/alert-template.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>System Alert</title>
</head>
<body>
    <h2>🚨 System Alert</h2>
    <p><strong>Server:</strong> {{server_name}}</p>
    <p><strong>Alert:</strong> {{alert_message}}</p>
    <p><strong>Severity:</strong> {{severity}}</p>
    <p><strong>Time:</strong> {{timestamp}}</p>
    <hr>
    <p>Please check the system immediately.</p>
</body>
</html>
EOF
```

## Error Handling

```bash
# Check email sending result
RESULT=$(curl -s -X POST "http://localhost:8080/packages/email-sender/execute" \
  -H "Content-Type: application/json" \
  -d '{"options": {...}}')

if echo "$RESULT" | jq -e '.data.result.success' > /dev/null; then
  echo "Email sent successfully"
else
  echo "Email failed: $(echo "$RESULT" | jq -r '.data.result.error')"
fi
```
