# Simple Data Processing Workflow

This workflow demonstrates basic data processing using the built-in sample packages.

## Overview

Process CSV data using the available packages:

1. Parse CSV data using the built-in CSV parser
2. Echo and process the data structure
3. Save results using output adapters

## Prerequisites

```bash
# Start Maitask server
maitask serve --port 8080

# Initialize with sample packages (if not already done)
maitask init
```

## Step 1: Prepare Sample Data

Create sample CSV data for processing:

```bash
# Create sample data file
cat > /tmp/sample_customers.csv << 'EOF'
name,email,age,status
John Doe,john@example.com,30,active
Jane Smith,jane@example.com,25,active
Bob Johnson,bob@example.com,35,inactive
Alice Brown,alice@example.com,28,active
EOF
```

## Step 2: Parse CSV Data

```bash
# Parse CSV file into structured data
curl -X POST "http://localhost:8080/packages/csv-parser/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": "'$(cat /tmp/sample_customers.csv)'"
  }' | jq '.' > /tmp/parsed_data.json

# View parsed result
echo "Parsed data:"
cat /tmp/parsed_data.json | jq '.data.rows | length'
echo "Total rows parsed"
```

## Step 3: Echo and Process Data

```bash
# Use echo package to process the parsed data
curl -X POST "http://localhost:8080/packages/echo/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": '$(cat /tmp/parsed_data.json | jq '.data')',
    "options": {
      "processing_note": "Echoing parsed CSV data"
    }
  }' | jq '.' > /tmp/processed_data.json

# Show processed results
echo "Processed data:"
cat /tmp/processed_data.json | jq '.data.echo.rows | length'
echo "records processed"
```

## Step 4: Save Results to File

```bash
# Save the processed data using file adapter
curl -X POST "http://localhost:8080/output" \
  -H "Content-Type: application/json" \
  -d '{
    "adapter": "file",
    "data": '$(cat /tmp/processed_data.json | jq '.data')',
    "config": {
      "path": "/tmp/final_results.json",
      "format": "json"
    }
  }' | jq '.'

# Verify the saved file
echo "Saved results:"
cat /tmp/final_results.json | jq '.echo.count'
echo "records saved to file"
```

## Step 5: Send Results via HTTP

```bash
# Send data to a webhook endpoint (example)
curl -X POST "http://localhost:8080/output" \
  -H "Content-Type: application/json" \
  -d '{
    "adapter": "http",
    "data": {
      "summary": "CSV processing complete",
      "total_records": '$(cat /tmp/final_results.json | jq '.echo.count')',
      "timestamp": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"
    },
    "config": {
      "url": "https://httpbin.org/post",
      "method": "POST",
      "headers": {
        "Content-Type": "application/json"
      }
    }
  }' | jq '.'
```

## Complete Workflow Script

Create a complete script that runs all steps:

```bash
#!/bin/bash
# simple-data-processing-workflow.sh

set -e  # Exit on any error

echo "=== Simple Data Processing Workflow ==="

# Step 1: Create sample data
echo "1. Creating sample data..."
cat > /tmp/sample_customers.csv << 'EOF'
name,email,age,status
John Doe,john@example.com,30,active
Jane Smith,jane@example.com,25,active
Bob Johnson,bob@example.com,35,inactive
Alice Brown,alice@example.com,28,active
EOF

# Step 2: Parse CSV
echo "2. Parsing CSV data..."
curl -s -X POST "http://localhost:8080/packages/csv-parser/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": "'$(cat /tmp/sample_customers.csv)'"
  }' | jq '.' > /tmp/parsed_data.json

PARSED_COUNT=$(cat /tmp/parsed_data.json | jq '.data.count')
echo "   Parsed $PARSED_COUNT rows"

# Step 3: Process with echo
echo "3. Processing data..."
curl -s -X POST "http://localhost:8080/packages/echo/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": '$(cat /tmp/parsed_data.json | jq '.data')',
    "options": {"processing_note": "Data processing workflow"}
  }' | jq '.' > /tmp/processed_data.json

echo "   Data processed successfully"

# Step 4: Save to file
echo "4. Saving results..."
curl -s -X POST "http://localhost:8080/output" \
  -H "Content-Type: application/json" \
  -d '{
    "adapter": "file",
    "data": '$(cat /tmp/processed_data.json | jq '.data')',
    "config": {
      "path": "/tmp/final_results.json",
      "format": "json"
    }
  }' > /dev/null

echo "   Results saved to /tmp/final_results.json"

echo ""
echo "=== Processing Complete ==="
echo "Summary:"
echo "- Input: $(cat /tmp/sample_customers.csv | wc -l) lines"
echo "- Parsed: $PARSED_COUNT records"
echo "- Output: /tmp/final_results.json"

echo ""
echo "Sample of final results:"
cat /tmp/final_results.json | jq '.echo.rows[0]'
```

## Expected Output

After running the complete workflow, you should see output similar to:

```
=== Simple Data Processing Workflow ===
1. Creating sample data...
2. Parsing CSV data...
   Parsed 4 rows
3. Processing data...
   Data processed successfully
4. Saving results...
   Results saved to /tmp/final_results.json

=== Processing Complete ===
Summary:
- Input: 5 lines
- Parsed: 4 records
- Output: /tmp/final_results.json

Sample of final results:
{
  "name": "John Doe",
  "email": "john@example.com",
  "age": "30",
  "status": "active"
}
```

## Key Features Demonstrated

1. **CSV Parsing**: Converting raw CSV text into structured data
2. **Data Processing**: Using echo package to process and validate data
3. **File Output**: Saving results to local files in JSON format
4. **HTTP Output**: Sending data to external APIs via HTTP requests
5. **Pipeline Workflow**: Chaining multiple operations together

## Next Steps

- Try modifying the CSV data to test different scenarios
- Experiment with different output adapter configurations
- Create your own custom packages for specific processing needs
- Process your own CSV files by replacing the sample data

This workflow uses only the built-in sample packages and demonstrates the core data processing capabilities of Maitask Runtime.
