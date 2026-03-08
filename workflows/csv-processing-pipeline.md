# CSV Data Processing Pipeline

This example demonstrates a complete CSV data processing workflow using Maitask's built-in packages.

## Overview

Process CSV sales data through validation, transformation, and reporting.

## Prerequisites

```bash
# Start Maitask server
maitask serve --port 8080

# Install required packages
maitask install csv-parser
maitask install data-validator
maitask install echo
```

## Sample Data

Create sample sales data:

```bash
cat > examples/data/sales_data.csv << 'EOF'
name,email,amount,date,region
John Doe,john@example.com,1500,2024-01-15,US
Jane Smith,jane@example.com,2300,2024-01-16,EU
Bob Johnson,bob@example.com,1200,2024-01-17,US
Alice Brown,alice@example.com,1800,2024-01-18,APAC
Charlie Wilson,charlie@invalid-email,500,2024-01-19,US
Diana Ross,diana@example.com,2800,2024-01-20,EU
EOF
```

## Step 1: Parse CSV Data

```bash
curl -X POST "http://localhost:8080/packages/csv-parser/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "text": "'$(cat examples/data/sales_data.csv)'"
    },
    "options": {
      "delimiter": ","
    }
  }' | jq '.data.result' > examples/data/parsed_sales.json
```

**Expected Output:**
```json
{
  "success": true,
  "parser": "csv",
  "columns": ["name", "email", "amount", "date", "region"],
  "rows": [
    {"name": "John Doe", "email": "john@example.com", "amount": "1500", "date": "2024-01-15", "region": "US", "__row": 1},
    {"name": "Jane Smith", "email": "jane@example.com", "amount": "2300", "date": "2024-01-16", "region": "EU", "__row": 2}
  ],
  "totalRows": 6
}
```

## Step 2: Validate Data

```bash
curl -X POST "http://localhost:8080/packages/data-validator/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": '$(cat examples/data/parsed_sales.json)',
    "options": {
      "validation_mode": "strict",
      "required_fields": ["name", "email", "amount"],
      "data_types": {
        "amount": "number",
        "email": "string"
      },
      "custom_rules": [
        {"field": "email", "rule": "email"},
        {"field": "amount", "rule": "positive"}
      ]
    }
  }' | jq '.data.result' > examples/data/validation_result.json
```

**Expected Output:**
```json
{
  "success": true,
  "data": {
    "results": [
      {"valid": true, "errors": [], "data": {"name": "John Doe", "email": "john@example.com", "amount": "1500"}},
      {"valid": false, "errors": ["email must be valid"], "data": {"name": "Charlie Wilson", "email": "charlie@invalid-email", "amount": "500"}}
    ],
    "summary": {
      "total": 6,
      "valid": 5,
      "invalid": 1
    }
  }
}
```

## Step 3: Transform and Aggregate Data

```bash
curl -X POST "http://localhost:8080/packages/echo/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": '$(cat examples/data/validation_result.json)',
    "options": {
      "operation": "transform",
      "transformRules": [
        {"field": "amount", "operation": "number"},
        {"field": "amount_usd", "operation": "calculate", "formula": "amount * 1.0"},
        {"field": "sales_tier", "operation": "conditional", "conditions": [
          {"if": "amount >= 2000", "then": "Premium"},
          {"if": "amount >= 1000", "then": "Standard"},
          {"else": "Basic"}
        ]}
      ],
      "filterRules": [
        {"field": "valid", "operator": "eq", "value": true}
      ]
    }
  }' | jq '.data.result' > examples/data/processed_sales.json
```

## Step 4: Generate Report

```bash
curl -X POST "http://localhost:8080/output" \
  -H "Content-Type: application/json" \
  -d '{
    "adapter": "file",
    "data": '$(cat examples/data/processed_sales.json)',
    "config": {
      "path": "./examples/data/sales_report.json",
      "format": "json"
    }
  }'
```

## Step 5: Send Notification

```bash
curl -X POST "http://localhost:8080/output" \
  -H "Content-Type: application/json" \
  -d '{
    "adapter": "webhook",
    "data": {
      "message": "Sales data processing completed",
      "summary": {
        "total_records": 6,
        "valid_records": 5,
        "invalid_records": 1,
        "total_amount": 10100,
        "processed_at": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"
      }
    },
    "config": {
      "url": "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK",
      "headers": {"Content-Type": "application/json"}
    }
  }'
```

## Complete Workflow Script

```bash
#!/bin/bash
# examples/workflows/run-csv-pipeline.sh

echo "=== CSV Processing Pipeline ==="

echo "1. Parsing CSV data..."
curl -s -X POST "http://localhost:8080/packages/csv-parser/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {"text": "'$(cat examples/data/sales_data.csv)'"},
    "options": {"delimiter": ","}
  }' | jq '.data.result' > examples/data/parsed_sales.json

echo "2. Validating data..."
curl -s -X POST "http://localhost:8080/packages/data-validator/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": '$(cat examples/data/parsed_sales.json)',
    "options": {
      "validation_mode": "strict",
      "required_fields": ["name", "email", "amount"],
      "custom_rules": [
        {"field": "email", "rule": "email"},
        {"field": "amount", "rule": "positive"}
      ]
    }
  }' | jq '.data.result' > examples/data/validation_result.json

echo "3. Processing and transforming..."
curl -s -X POST "http://localhost:8080/packages/echo/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": '$(cat examples/data/validation_result.json)',
    "options": {
      "operation": "transform",
      "transformRules": [
        {"field": "amount", "operation": "number"},
        {"field": "sales_tier", "operation": "conditional", "conditions": [
          {"if": "amount >= 2000", "then": "Premium"},
          {"if": "amount >= 1000", "then": "Standard"},
          {"else": "Basic"}
        ]}
      ]
    }
  }' | jq '.data.result' > examples/data/processed_sales.json

echo "4. Generating report..."
curl -s -X POST "http://localhost:8080/output" \
  -H "Content-Type: application/json" \
  -d '{
    "adapter": "file",
    "data": '$(cat examples/data/processed_sales.json)',
    "config": {"path": "./examples/data/sales_report.json"}
  }'

echo "=== Pipeline Complete ==="
echo "Results saved to examples/data/sales_report.json"
```

## Key Learnings

1. **Package Chaining**: Multiple packages can be chained together using file outputs
2. **Data Validation**: Always validate data before processing
3. **Error Handling**: Invalid records are filtered out gracefully
4. **Output Flexibility**: Results can be saved to files or sent to external systems
5. **Automation**: The entire workflow can be scripted for automation

## Next Steps

- Modify validation rules for your specific data requirements
- Add more transformation steps
- Integrate with your preferred notification system
- Schedule the pipeline to run automatically