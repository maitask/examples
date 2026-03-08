# Customer Risk Scoring System

This example demonstrates creating a comprehensive customer risk scoring system using Maitask.

## Overview

Build a complete customer risk assessment pipeline that:
- Processes customer transaction data
- Calculates risk scores using multiple factors
- Categorizes customers by risk level
- Generates reports and alerts

## Prerequisites

```bash
# Install required packages
maitask install csv-parser
maitask install data-validator
maitask install echo
maitask install email-sender
```

## Step 1: Create Custom Scoring Package

```bash
curl -X POST "http://localhost:8080/packages" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "customer-risk-scorer",
    "description": "Calculate customer risk scores based on transaction history and behavior patterns",
    "code": "function execute(input, options, context) { const customers = input.customers || []; const weights = options.weights || { transactionCount: 0.25, totalAmount: 0.30, avgAmount: 0.20, daysSinceLastTransaction: 0.15, accountAge: 0.10 }; const thresholds = options.thresholds || { low: 80, medium: 60, high: 40 }; const scoredCustomers = customers.map(customer => { const txCount = customer.transaction_count || 0; const totalAmount = customer.total_amount || 0; const avgAmount = txCount > 0 ? totalAmount / txCount : 0; const daysSinceLast = customer.days_since_last_transaction || 999; const accountAge = customer.account_age_days || 0; const normalizedTxCount = Math.min(txCount / 100, 1); const normalizedTotalAmount = Math.min(totalAmount / 50000, 1); const normalizedAvgAmount = Math.min(avgAmount / 1000, 1); const normalizedDaysSince = Math.max(0, 1 - daysSinceLast / 365); const normalizedAccountAge = Math.min(accountAge / 1095, 1); const score = (normalizedTxCount * weights.transactionCount + normalizedTotalAmount * weights.totalAmount + normalizedAvgAmount * weights.avgAmount + normalizedDaysSince * weights.daysSinceLastTransaction + normalizedAccountAge * weights.accountAge) * 100; let riskCategory; let riskLevel; if (score >= thresholds.low) { riskCategory = \"Low Risk\"; riskLevel = 1; } else if (score >= thresholds.medium) { riskCategory = \"Medium Risk\"; riskLevel = 2; } else if (score >= thresholds.high) { riskCategory = \"High Risk\"; riskLevel = 3; } else { riskCategory = \"Very High Risk\"; riskLevel = 4; } const riskFactors = []; if (daysSinceLast > 180) riskFactors.push(\"Inactive for 6+ months\"); if (txCount < 5) riskFactors.push(\"Low transaction volume\"); if (avgAmount < 50) riskFactors.push(\"Low average transaction\"); if (accountAge < 90) riskFactors.push(\"New account\"); return { ...customer, risk_score: Math.round(score * 10) / 10, risk_category: riskCategory, risk_level: riskLevel, risk_factors: riskFactors, recommendations: generateRecommendations(riskLevel, riskFactors) }; }); function generateRecommendations(riskLevel, factors) { const recommendations = []; if (riskLevel >= 3) { recommendations.push(\"Consider account review\"); if (factors.includes(\"Inactive for 6+ months\")) { recommendations.push(\"Send reactivation campaign\"); } if (factors.includes(\"Low transaction volume\")) { recommendations.push(\"Offer incentives to increase usage\"); } } else if (riskLevel === 2) { recommendations.push(\"Monitor account activity\"); recommendations.push(\"Consider targeted offers\"); } else { recommendations.push(\"Maintain current service level\"); recommendations.push(\"Consider upselling opportunities\"); } return recommendations; } const summary = { total_customers: scoredCustomers.length, average_score: scoredCustomers.reduce((sum, c) => sum + c.risk_score, 0) / scoredCustomers.length, risk_distribution: { low: scoredCustomers.filter(c => c.risk_level === 1).length, medium: scoredCustomers.filter(c => c.risk_level === 2).length, high: scoredCustomers.filter(c => c.risk_level === 3).length, very_high: scoredCustomers.filter(c => c.risk_level === 4).length }, high_risk_customers: scoredCustomers.filter(c => c.risk_level >= 3).length }; return { success: true, data: { customers: scoredCustomers, summary: summary, metadata: { processed_at: new Date().toISOString(), algorithm_version: \"2.0\", weights_used: weights, thresholds_used: thresholds } } }; } execute;",
    "keywords": ["customer", "risk", "scoring", "analytics", "business"],
    "config_schema": {
      "type": "object",
      "properties": {
        "weights": {
          "type": "object",
          "properties": {
            "transactionCount": {"type": "number", "default": 0.25},
            "totalAmount": {"type": "number", "default": 0.30},
            "avgAmount": {"type": "number", "default": 0.20},
            "daysSinceLastTransaction": {"type": "number", "default": 0.15},
            "accountAge": {"type": "number", "default": 0.10}
          }
        },
        "thresholds": {
          "type": "object",
          "properties": {
            "low": {"type": "number", "default": 80},
            "medium": {"type": "number", "default": 60},
            "high": {"type": "number", "default": 40}
          }
        }
      }
    }
  }'
```

## Step 2: Prepare Sample Customer Data

```bash
cat > examples/data/customer_data.csv << 'EOF'
customer_id,name,email,transaction_count,total_amount,days_since_last_transaction,account_age_days,customer_type
C001,Alice Johnson,alice@example.com,45,12500,5,730,premium
C002,Bob Smith,bob@example.com,12,3200,30,365,standard
C003,Carol White,carol@example.com,3,450,180,90,basic
C004,David Brown,david@example.com,67,25000,2,1095,premium
C005,Eve Davis,eve@example.com,8,1200,90,180,standard
C006,Frank Wilson,frank@example.com,1,150,300,45,basic
C007,Grace Lee,grace@example.com,89,35000,1,1460,premium
C008,Henry Clark,henry@example.com,23,8900,15,547,standard
EOF
```

## Step 3: Process Customer Data Pipeline

### Parse CSV Data

```bash
curl -X POST "http://localhost:8080/packages/csv-parser/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "text": "'$(cat examples/data/customer_data.csv)'"
    },
    "options": {
      "delimiter": ","
    }
  }' | jq '.data.result' > examples/data/parsed_customers.json
```

### Validate Customer Data

```bash
curl -X POST "http://localhost:8080/packages/data-validator/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": '$(cat examples/data/parsed_customers.json)',
    "options": {
      "validation_mode": "strict",
      "required_fields": ["customer_id", "name", "email"],
      "data_types": {
        "transaction_count": "number",
        "total_amount": "number",
        "days_since_last_transaction": "number",
        "account_age_days": "number"
      },
      "custom_rules": [
        {"field": "email", "rule": "email"},
        {"field": "transaction_count", "rule": "positive"},
        {"field": "total_amount", "rule": "positive"}
      ]
    }
  }' | jq '.data.result' > examples/data/validated_customers.json
```

### Calculate Risk Scores

```bash
curl -X POST "http://localhost:8080/packages/customer-risk-scorer/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "customers": '$(cat examples/data/validated_customers.json | jq '.data.results | map(select(.valid) | .data)')'
    },
    "options": {
      "weights": {
        "transactionCount": 0.30,
        "totalAmount": 0.35,
        "avgAmount": 0.20,
        "daysSinceLastTransaction": 0.10,
        "accountAge": 0.05
      },
      "thresholds": {
        "low": 75,
        "medium": 55,
        "high": 35
      }
    }
  }' | jq '.data.result' > examples/data/scored_customers.json
```

## Step 4: Generate Risk Reports

### High-Risk Customer Report

```bash
curl -X POST "http://localhost:8080/packages/echo/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": '$(cat examples/data/scored_customers.json)',
    "options": {
      "operation": "filter",
      "filterRules": [
        {"field": "risk_level", "operator": "gte", "value": 3}
      ]
    }
  }' | jq '.data.result' > examples/data/high_risk_customers.json
```

### Generate Summary Report

```bash
cat > examples/data/risk_summary.json << EOF
{
  "report_date": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "summary": $(cat examples/data/scored_customers.json | jq '.data.summary'),
  "high_risk_details": $(cat examples/data/high_risk_customers.json | jq '.data'),
  "recommendations": {
    "immediate_action": "Review $(cat examples/data/scored_customers.json | jq '.data.summary.high_risk_customers') high-risk customers",
    "follow_up": "Monitor medium-risk customers weekly",
    "retention": "Engage with inactive customers (180+ days)"
  }
}
EOF
```

## Step 5: Send Automated Alerts

### Email High-Risk Alert

```bash
# Check if there are high-risk customers
HIGH_RISK_COUNT=$(cat examples/data/scored_customers.json | jq '.data.summary.high_risk_customers')

if [ "$HIGH_RISK_COUNT" -gt 0 ]; then
  curl -X POST "http://localhost:8080/packages/email-sender/execute" \
    -H "Content-Type: application/json" \
    -d '{
      "input": '$(cat examples/data/risk_summary.json)',
      "options": {
        "provider": "sendgrid",
        "api_key": "YOUR_SENDGRID_API_KEY",
        "from": {
          "email": "risk-alerts@company.com",
          "name": "Customer Risk Management"
        },
        "to": [
          {"email": "risk-team@company.com", "name": "Risk Management Team"},
          {"email": "customer-success@company.com", "name": "Customer Success"}
        ],
        "subject": "🚨 High-Risk Customer Alert - '$(date +%Y-%m-%d)'",
        "template": "risk-alert",
        "template_data": {
          "date": "'$(date +%Y-%m-%d)'",
          "high_risk_count": '$HIGH_RISK_COUNT',
          "total_customers": '$(cat examples/data/scored_customers.json | jq '.data.summary.total_customers')',
          "average_score": '$(cat examples/data/scored_customers.json | jq '.data.summary.average_score')'
        }
      }
    }'
fi
```

## Step 6: Output to Different Systems

### Save to Database

```bash
curl -X POST "http://localhost:8080/output" \
  -H "Content-Type: application/json" \
  -d '{
    "adapter": "database",
    "data": '$(cat examples/data/scored_customers.json)',
    "config": {
      "connection_string": "postgresql://user:pass@localhost/risk_db",
      "table": "customer_risk_scores",
      "mode": "upsert"
    }
  }'
```

### Send to API

```bash
curl -X POST "http://localhost:8080/output" \
  -H "Content-Type: application/json" \
  -d '{
    "adapter": "http",
    "data": '$(cat examples/data/risk_summary.json)',
    "config": {
      "url": "https://api.company.com/risk-management/scores",
      "method": "POST",
      "headers": {
        "Authorization": "Bearer YOUR_API_TOKEN",
        "Content-Type": "application/json"
      }
    }
  }'
```

## Complete Scoring Workflow Script

```bash
#!/bin/bash
# examples/data-processing/run-customer-scoring.sh

echo "=== Customer Risk Scoring Workflow ==="

# 1. Parse customer data
echo "1. Parsing customer data..."
curl -s -X POST "http://localhost:8080/packages/csv-parser/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {"text": "'$(cat examples/data/customer_data.csv)'"},
    "options": {"delimiter": ","}
  }' | jq '.data.result' > examples/data/parsed_customers.json

# 2. Validate data
echo "2. Validating customer data..."
curl -s -X POST "http://localhost:8080/packages/data-validator/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": '$(cat examples/data/parsed_customers.json)',
    "options": {
      "validation_mode": "strict",
      "required_fields": ["customer_id", "email"],
      "custom_rules": [{"field": "email", "rule": "email"}]
    }
  }' | jq '.data.result' > examples/data/validated_customers.json

# 3. Calculate risk scores
echo "3. Calculating risk scores..."
VALID_CUSTOMERS=$(cat examples/data/validated_customers.json | jq '.data.results | map(select(.valid) | .data)')
curl -s -X POST "http://localhost:8080/packages/customer-risk-scorer/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {"customers": '$VALID_CUSTOMERS'},
    "options": {
      "weights": {
        "transactionCount": 0.30,
        "totalAmount": 0.35,
        "avgAmount": 0.20,
        "daysSinceLastTransaction": 0.10,
        "accountAge": 0.05
      }
    }
  }' | jq '.data.result' > examples/data/scored_customers.json

# 4. Generate reports
echo "4. Generating risk reports..."
HIGH_RISK_COUNT=$(cat examples/data/scored_customers.json | jq '.data.summary.high_risk_customers')
TOTAL_CUSTOMERS=$(cat examples/data/scored_customers.json | jq '.data.summary.total_customers')
AVG_SCORE=$(cat examples/data/scored_customers.json | jq '.data.summary.average_score')

echo "   Total Customers: $TOTAL_CUSTOMERS"
echo "   High Risk Customers: $HIGH_RISK_COUNT"
echo "   Average Risk Score: $AVG_SCORE"

# 5. Save results
echo "5. Saving results..."
curl -s -X POST "http://localhost:8080/output" \
  -H "Content-Type: application/json" \
  -d '{
    "adapter": "file",
    "data": '$(cat examples/data/scored_customers.json)',
    "config": {"path": "./examples/data/customer_risk_report.json"}
  }'

# 6. Send alerts if needed
if [ "$HIGH_RISK_COUNT" -gt 2 ]; then
  echo "6. Sending high-risk alert..."
  curl -s -X POST "http://localhost:8080/packages/email-sender/execute" \
    -H "Content-Type: application/json" \
    -d '{
      "options": {
        "provider": "smtp",
        "from": {"email": "alerts@company.com"},
        "to": [{"email": "risk-team@company.com"}],
        "subject": "High-Risk Customer Alert",
        "text": "Warning: '$HIGH_RISK_COUNT' customers are classified as high-risk."
      }
    }'
else
  echo "6. No high-risk alert needed (only $HIGH_RISK_COUNT high-risk customers)"
fi

echo "=== Scoring Complete ==="
echo "Results saved to: examples/data/customer_risk_report.json"
```

## Customization Options

### Adjust Scoring Weights

```bash
# Example: Emphasize recent activity
curl -X POST "http://localhost:8080/packages/customer-risk-scorer/execute" \
  -d '{
    "options": {
      "weights": {
        "daysSinceLastTransaction": 0.40,
        "transactionCount": 0.25,
        "totalAmount": 0.20,
        "avgAmount": 0.10,
        "accountAge": 0.05
      }
    }
  }'
```

### Custom Risk Thresholds

```bash
# Example: More conservative risk assessment
curl -X POST "http://localhost:8080/packages/customer-risk-scorer/execute" \
  -d '{
    "options": {
      "thresholds": {
        "low": 85,
        "medium": 70,
        "high": 50
      }
    }
  }'
```

This comprehensive example demonstrates how to build a sophisticated customer risk scoring system using Maitask's package ecosystem, from data ingestion to automated reporting and alerting.