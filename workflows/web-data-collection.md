# Web Data Collection Workflow

This example demonstrates collecting and processing web data using Maitask's web-related packages.

## Overview

- Collect Hacker News stories
- Perform web searches
- Process and analyze results

## Prerequisites

```bash
# Install required packages
maitask install hackernews-crawler
maitask install web-search
maitask install data-validator
maitask install echo
```

## Example 1: Hacker News Analysis

### Collect Top Stories

```bash
curl -X POST "http://localhost:8080/packages/hackernews-crawler/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {},
    "options": {
      "limit": 20,
      "storyType": "top",
      "includeComments": false
    }
  }' > examples/data/hn_stories.json
```

### Process Stories

```bash
curl -X POST "http://localhost:8080/packages/echo/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": '$(cat examples/data/hn_stories.json)',
    "options": {
      "operation": "filter",
      "filterRules": [
        {"field": "score", "operator": "gt", "value": 100},
        {"field": "title", "operator": "contains", "value": "AI"}
      ]
    }
  }' > examples/data/filtered_stories.json
```

## Example 2: Market Research

### Search for Trends

```bash
curl -X POST "http://localhost:8080/packages/web-search/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {},
    "options": {
      "query": "artificial intelligence market trends 2024",
      "runtime": "google",
      "limit": 15,
      "language": "en",
      "region": "us",
      "includeSnippets": true
    }
  }' > examples/data/search_results.json
```

### Analyze Results

```bash
curl -X POST "http://localhost:8080/packages/data-validator/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": '$(cat examples/data/search_results.json)',
    "options": {
      "validation_mode": "report-only",
      "required_fields": ["title", "url"],
      "custom_rules": [
        {"field": "url", "rule": "url"},
        {"field": "title", "rule": "not-empty"}
      ]
    }
  }' > examples/data/validated_results.json
```

## Complete Workflow Script

```bash
#!/bin/bash
# examples/workflows/run-web-collection.sh

echo "=== Web Data Collection Workflow ==="

echo "1. Collecting Hacker News stories..."
curl -s -X POST "http://localhost:8080/packages/hackernews-crawler/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {},
    "options": {"limit": 20, "storyType": "top"}
  }' | jq '.data.result' > examples/data/hn_stories.json

echo "2. Searching for AI trends..."
curl -s -X POST "http://localhost:8080/packages/web-search/execute" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {},
    "options": {
      "query": "AI trends 2024",
      "runtime": "google",
      "limit": 10
    }
  }' | jq '.data.result' > examples/data/search_results.json

echo "3. Combining and processing data..."
jq -s '.[0].stories + .[1].results' \
  examples/data/hn_stories.json \
  examples/data/search_results.json > examples/data/combined_data.json

echo "4. Sending to webhook..."
curl -s -X POST "http://localhost:8080/output" \
  -H "Content-Type: application/json" \
  -d '{
    "adapter": "webhook",
    "data": '$(cat examples/data/combined_data.json)',
    "config": {
      "url": "https://hooks.slack.com/services/YOUR/WEBHOOK"
    }
  }'

echo "=== Collection Complete ==="
```

## Output to Different Destinations

### Save to File

```bash
curl -X POST "http://localhost:8080/output" \
  -H "Content-Type: application/json" \
  -d '{
    "adapter": "file",
    "data": '$(cat examples/data/combined_data.json)',
    "config": {
      "path": "./examples/data/web_research_report.json",
      "format": "json"
    }
  }'
```

### Send to API

```bash
curl -X POST "http://localhost:8080/output" \
  -H "Content-Type: application/json" \
  -d '{
    "adapter": "http",
    "data": '$(cat examples/data/combined_data.json)',
    "config": {
      "url": "https://api.company.com/research",
      "method": "POST",
      "headers": {
        "Authorization": "Bearer YOUR_API_TOKEN",
        "Content-Type": "application/json"
      }
    }
  }'
```
