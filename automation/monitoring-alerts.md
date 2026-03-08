# System Monitoring and Alerts

Automated monitoring workflows using Maitask packages for system health, performance tracking, and alerting.

## Overview

Set up comprehensive monitoring that:
- Collects system metrics
- Validates against thresholds
- Sends alerts when issues are detected
- Generates reports and dashboards

## Prerequisites

```bash
# Install required packages
maitask install data-validator
maitask install email-sender
maitask install slack-notifier
maitask install echo
```

## Example 1: Server Health Monitoring

### Create Monitoring Script

```bash
cat > examples/automation/monitor_system.sh << 'EOF'
#!/bin/bash

# Collect system metrics (mock data for example)
cat > /tmp/system_metrics.json << 'METRICS'
{
  "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "server": "production-web-01",
  "metrics": {
    "cpu_usage": 78,
    "memory_usage": 85,
    "disk_usage": 72,
    "load_average": 2.1,
    "active_connections": 342,
    "response_time_ms": 150
  }
}
METRICS

# Validate metrics against thresholds
curl -s -X POST "http://localhost:8080/packages/data-validator/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "data": ['$(cat /tmp/system_metrics.json)']
    },
    "options": {
      "validation_mode": "strict",
      "custom_rules": [
        {"field": "cpu_usage", "rule": "max", "value": 80, "message": "CPU usage exceeded 80%"},
        {"field": "memory_usage", "rule": "max", "value": 90, "message": "Memory usage exceeded 90%"},
        {"field": "disk_usage", "rule": "max", "value": 85, "message": "Disk usage exceeded 85%"},
        {"field": "response_time_ms", "rule": "max", "value": 200, "message": "Response time exceeded 200ms"}
      ]
    }
  }' > /tmp/validation_result.json

# Check for violations
VIOLATIONS=$(cat /tmp/validation_result.json | jq '.data.result.summary.invalid_records')

if [ "$VIOLATIONS" -gt 0 ]; then
  echo "Alert: $VIOLATIONS threshold violations detected"

  # Send Slack alert
  curl -s -X POST "http://localhost:8080/packages/slack-notifier/execute" \
    -H "Content-Type: application/json" \
    -d '{
      "input": '$(cat /tmp/validation_result.json)',
      "options": {
        "webhook_url": "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK",
        "channel": "#alerts",
        "username": "System Monitor",
        "icon_emoji": ":warning:",
        "message": "🚨 System Alert: '$VIOLATIONS' threshold violations on production-web-01",
        "color": "danger",
        "fields": [
          {"title": "Server", "value": "production-web-01", "short": true},
          {"title": "Violations", "value": "'$VIOLATIONS'", "short": true},
          {"title": "Time", "value": "$(date)", "short": false}
        ]
      }
    }'

  # Send email alert
  curl -s -X POST "http://localhost:8080/packages/email-sender/execute" \
    -H "Content-Type: application/json" \
    -d '{
      "input": '$(cat /tmp/validation_result.json)',
      "options": {
        "provider": "sendgrid",
        "api_key": "'$SENDGRID_API_KEY'",
        "from": {"email": "alerts@company.com", "name": "System Monitor"},
        "to": [{"email": "oncall@company.com", "name": "On-Call Team"}],
        "subject": "🚨 URGENT: System Alert - production-web-01",
        "html": "<h2>System Alert</h2><p>Threshold violations detected on production-web-01</p><p>Violations: '$VIOLATIONS'</p><p>Please investigate immediately.</p>"
      }
    }'
else
  echo "All systems normal"
fi
EOF

chmod +x examples/automation/monitor_system.sh
```

## Example 2: Database Performance Monitoring

### Database Metrics Collection

```bash
cat > examples/data/db_metrics.json << 'EOF'
{
  "database": "production-db",
  "timestamp": "2024-01-20T10:30:00Z",
  "metrics": {
    "active_connections": 45,
    "max_connections": 100,
    "query_time_avg_ms": 125,
    "slow_queries": 3,
    "cpu_usage": 65,
    "memory_usage": 78,
    "disk_io_rate": 850,
    "replication_lag_ms": 200
  }
}
EOF

# Validate database metrics
curl -X POST "http://localhost:8080/packages/data-validator/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "data": ['$(cat examples/data/db_metrics.json)']
    },
    "options": {
      "validation_mode": "report-only",
      "custom_rules": [
        {"field": "active_connections", "rule": "max", "value": 80, "message": "High connection count"},
        {"field": "query_time_avg_ms", "rule": "max", "value": 100, "message": "Slow average query time"},
        {"field": "slow_queries", "rule": "max", "value": 5, "message": "Too many slow queries"},
        {"field": "replication_lag_ms", "rule": "max", "value": 1000, "message": "High replication lag"}
      ]
    }
  }' > examples/data/db_validation.json
```

### Process and Alert

```bash
# Check for database issues
DB_ISSUES=$(cat examples/data/db_validation.json | jq '.data.result.summary.invalid_records')

if [ "$DB_ISSUES" -gt 0 ]; then
  echo "Database performance issues detected: $DB_ISSUES"

  # Send detailed Slack notification
  curl -X POST "http://localhost:8080/packages/slack-notifier/execute" \
    -H "Content-Type: application/json" \
    -d '{
      "input": '$(cat examples/data/db_validation.json)',
      "options": {
        "webhook_url": "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK",
        "channel": "#database-alerts",
        "message": "📊 Database Performance Alert",
        "color": "warning",
        "fields": [
          {"title": "Database", "value": "production-db", "short": true},
          {"title": "Issues Found", "value": "'$DB_ISSUES'", "short": true},
          {"title": "Active Connections", "value": "45/100", "short": true},
          {"title": "Avg Query Time", "value": "125ms", "short": true}
        ]
      }
    }'
fi
```

## Example 3: Application Log Monitoring

### Log Analysis Workflow

```bash
# Create sample log data
cat > examples/data/app_logs.json << 'EOF'
{
  "logs": [
    {"timestamp": "2024-01-20T10:00:00Z", "level": "INFO", "message": "User login successful", "user_id": "user123"},
    {"timestamp": "2024-01-20T10:01:00Z", "level": "ERROR", "message": "Database connection failed", "component": "auth"},
    {"timestamp": "2024-01-20T10:02:00Z", "level": "WARN", "message": "High memory usage detected", "memory_pct": 85},
    {"timestamp": "2024-01-20T10:03:00Z", "level": "ERROR", "message": "Payment processing failed", "transaction_id": "tx456"},
    {"timestamp": "2024-01-20T10:04:00Z", "level": "INFO", "message": "Cache refresh completed", "duration_ms": 1200}
  ]
}
EOF

# Filter and count errors
curl -X POST "http://localhost:8080/packages/echo/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": '$(cat examples/data/app_logs.json)',
    "options": {
      "operation": "filter",
      "filterRules": [
        {"field": "level", "operator": "eq", "value": "ERROR"}
      ]
    }
  }' > examples/data/error_logs.json

# Count errors in last hour
ERROR_COUNT=$(cat examples/data/error_logs.json | jq '.data | length')

if [ "$ERROR_COUNT" -gt 5 ]; then
  echo "High error rate detected: $ERROR_COUNT errors"

  # Send escalated alert
  curl -X POST "http://localhost:8080/packages/email-sender/execute" \
    -H "Content-Type: application/json" \
    -d '{
      "input": '$(cat examples/data/error_logs.json)',
      "options": {
        "provider": "sendgrid",
        "api_key": "'$SENDGRID_API_KEY'",
        "from": {"email": "logs@company.com", "name": "Log Monitor"},
        "to": [
          {"email": "devops@company.com", "name": "DevOps Team"},
          {"email": "engineering@company.com", "name": "Engineering"}
        ],
        "subject": "🔥 CRITICAL: High Error Rate Detected",
        "html": "<h2>Application Error Alert</h2><p><strong>Error Count:</strong> '$ERROR_COUNT' in the last hour</p><p><strong>Threshold:</strong> 5 errors</p><p>Immediate investigation required.</p>"
      }
    }'
fi
```

## Example 4: Multi-Service Health Dashboard

### Collect All Service Metrics

```bash
cat > examples/automation/collect_all_metrics.sh << 'EOF'
#!/bin/bash

echo "=== Multi-Service Health Check ==="

# Collect metrics from different services
TIMESTAMP=$(date -u +%Y-%m-%dT%H:%M:%SZ)

# Web service metrics
WEB_METRICS='{
  "service": "web-api",
  "timestamp": "'$TIMESTAMP'",
  "status": "healthy",
  "response_time": 89,
  "error_rate": 0.5,
  "requests_per_minute": 1250
}'

# Database metrics
DB_METRICS='{
  "service": "database",
  "timestamp": "'$TIMESTAMP'",
  "status": "healthy",
  "connections": 45,
  "query_time": 125,
  "cpu_usage": 65
}'

# Cache metrics
CACHE_METRICS='{
  "service": "redis-cache",
  "timestamp": "'$TIMESTAMP'",
  "status": "healthy",
  "hit_rate": 94.5,
  "memory_usage": 72,
  "connections": 23
}'

# Combine all metrics
jq -n --argjson web "$WEB_METRICS" --argjson db "$DB_METRICS" --argjson cache "$CACHE_METRICS" \
  '{services: [$web, $db, $cache], timestamp: "'$TIMESTAMP'"}' > examples/data/all_metrics.json

# Validate all services
curl -s -X POST "http://localhost:8080/packages/data-validator/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "data": '$(cat examples/data/all_metrics.json | jq '.services')'
    },
    "options": {
      "validation_mode": "report-only",
      "custom_rules": [
        {"field": "response_time", "rule": "max", "value": 100},
        {"field": "error_rate", "rule": "max", "value": 1.0},
        {"field": "cpu_usage", "rule": "max", "value": 80},
        {"field": "hit_rate", "rule": "min", "value": 90}
      ]
    }
  }' > examples/data/service_health.json

# Generate dashboard data
TOTAL_SERVICES=$(cat examples/data/all_metrics.json | jq '.services | length')
HEALTHY_SERVICES=$(cat examples/data/service_health.json | jq '.data.result.summary.valid_records')
ISSUES=$(cat examples/data/service_health.json | jq '.data.result.summary.invalid_records')

echo "Services: $HEALTHY_SERVICES/$TOTAL_SERVICES healthy"
echo "Issues: $ISSUES"

# Save dashboard data
jq -n \
  --argjson total "$TOTAL_SERVICES" \
  --argjson healthy "$HEALTHY_SERVICES" \
  --argjson issues "$ISSUES" \
  --arg timestamp "$TIMESTAMP" \
  '{
    dashboard: {
      total_services: $total,
      healthy_services: $healthy,
      issues: $issues,
      health_percentage: (($healthy / $total) * 100),
      last_update: $timestamp,
      status: (if $issues == 0 then "all_healthy" else "issues_detected" end)
    }
  }' > examples/data/dashboard.json

# Send daily summary (if it's 9 AM)
HOUR=$(date +%H)
if [ "$HOUR" = "09" ]; then
  curl -s -X POST "http://localhost:8080/packages/slack-notifier/execute" \
    -H "Content-Type: application/json" \
    -d '{
      "options": {
        "webhook_url": "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK",
        "channel": "#daily-reports",
        "message": "📊 Daily Health Summary",
        "color": "good",
        "fields": [
          {"title": "Services", "value": "'$HEALTHY_SERVICES'/'$TOTAL_SERVICES' healthy", "short": true},
          {"title": "Issues", "value": "'$ISSUES'", "short": true},
          {"title": "Overall Health", "value": "'$(cat examples/data/dashboard.json | jq -r '.dashboard.health_percentage')'%", "short": true}
        ]
      }
    }'
fi
EOF

chmod +x examples/automation/collect_all_metrics.sh
```

## Example 5: Automated Incident Response

### Create Incident Workflow

```bash
cat > examples/automation/incident_response.sh << 'EOF'
#!/bin/bash

# Incident response workflow
INCIDENT_SEVERITY=$1
INCIDENT_MESSAGE=$2

if [ -z "$INCIDENT_SEVERITY" ]; then
  echo "Usage: $0 <severity> <message>"
  echo "Severity: critical, high, medium, low"
  exit 1
fi

echo "=== Incident Response: $INCIDENT_SEVERITY ==="

# Create incident record
INCIDENT_ID="INC-$(date +%Y%m%d-%H%M%S)"
INCIDENT_DATA='{
  "incident_id": "'$INCIDENT_ID'",
  "severity": "'$INCIDENT_SEVERITY'",
  "message": "'$INCIDENT_MESSAGE'",
  "timestamp": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'",
  "status": "open",
  "assigned_team": null
}'

echo "$INCIDENT_DATA" > examples/data/incident_$INCIDENT_ID.json

# Determine response based on severity
case $INCIDENT_SEVERITY in
  "critical")
    # Page on-call team immediately
    curl -s -X POST "http://localhost:8080/packages/email-sender/execute" \
      -H "Content-Type: application/json" \
      -d '{
        "options": {
          "provider": "sendgrid",
          "from": {"email": "incidents@company.com"},
          "to": [
            {"email": "oncall@company.com"},
            {"email": "manager@company.com"}
          ],
          "subject": "🚨 CRITICAL INCIDENT: '$INCIDENT_ID'",
          "text": "CRITICAL INCIDENT DETECTED\n\nID: '$INCIDENT_ID'\nMessage: '$INCIDENT_MESSAGE'\nTime: $(date)\n\nImmediate response required."
        }
      }'

    # Send Slack alert to multiple channels
    for CHANNEL in "#incidents" "#oncall" "#management"; do
      curl -s -X POST "http://localhost:8080/packages/slack-notifier/execute" \
        -H "Content-Type: application/json" \
        -d '{
          "options": {
            "webhook_url": "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK",
            "channel": "'$CHANNEL'",
            "message": "🚨 CRITICAL INCIDENT",
            "color": "danger",
            "fields": [
              {"title": "Incident ID", "value": "'$INCIDENT_ID'", "short": true},
              {"title": "Severity", "value": "CRITICAL", "short": true},
              {"title": "Message", "value": "'$INCIDENT_MESSAGE'", "short": false}
            ]
          }
        }'
    done
    ;;

  "high")
    # Alert team during business hours
    curl -s -X POST "http://localhost:8080/packages/slack-notifier/execute" \
      -H "Content-Type: application/json" \
      -d '{
        "options": {
          "webhook_url": "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK",
          "channel": "#incidents",
          "message": "⚠️ High Priority Incident",
          "color": "warning",
          "fields": [
            {"title": "Incident ID", "value": "'$INCIDENT_ID'", "short": true},
            {"title": "Message", "value": "'$INCIDENT_MESSAGE'", "short": false}
          ]
        }
      }'
    ;;

  "medium"|"low")
    # Create ticket for next business day
    curl -s -X POST "http://localhost:8080/output" \
      -H "Content-Type: application/json" \
      -d '{
        "adapter": "file",
        "data": '$INCIDENT_DATA',
        "config": {"path": "./examples/data/pending_incidents.json", "append": true}
      }'
    ;;
esac

echo "Incident $INCIDENT_ID logged and notifications sent"
EOF

chmod +x examples/automation/incident_response.sh
```

## Cron Job Setup

### Schedule Monitoring Tasks

```bash
# Add to crontab (crontab -e)
cat > examples/automation/monitoring_crontab << 'EOF'
# System monitoring every 5 minutes
*/5 * * * * /path/to/examples/automation/monitor_system.sh

# Database monitoring every 10 minutes
*/10 * * * * /path/to/examples/automation/monitor_database.sh

# Collect all metrics every minute
* * * * * /path/to/examples/automation/collect_all_metrics.sh

# Daily health summary at 9 AM
0 9 * * * /path/to/examples/automation/daily_summary.sh

# Weekly maintenance report on Sundays at 6 AM
0 6 * * 0 /path/to/examples/automation/weekly_report.sh
EOF
```

## Key Benefits

1. **Automated Detection**: Continuous monitoring without manual intervention
2. **Intelligent Alerting**: Context-aware notifications based on severity
3. **Multi-Channel Notifications**: Email, Slack, and other integrations
4. **Audit Trail**: All incidents and responses are logged
5. **Scalable**: Easy to add new services and metrics
6. **Flexible**: Customizable thresholds and response procedures

This monitoring system provides comprehensive coverage for system health, performance tracking, and automated incident response using Maitask's package ecosystem.