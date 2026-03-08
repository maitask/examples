# Maitask Examples

[English](README.md) | [中文](README_zh-CN.md)

Comprehensive examples and templates. Learn how to use Maitask Runtime with official packages, create custom workflows, and integrate with third-party services.

## Directory Structure

```text
examples/
├── README.md                    # This file
├── workflows/                   # Complete workflow examples
├── packages/                    # Package-specific examples
├── data-processing/            # Data analysis and processing examples
├── automation/                 # Automation and monitoring examples
└── data/                       # Sample data files and outputs
```

## Quick Start

1. **Start Maitask Server**

    ```bash
    maitask serve --port 8080
    ```

2. **Install Required Packages**

    ```bash
    # Install official packages (use @maitask/ prefix)
    maitask install @maitask/csv-parser
    maitask install @maitask/file-processor
    maitask install @maitask/text-parser
    ```

3. **Run Your First Example**

    ```bash
    cd examples/workflows
    # Follow the step-by-step guide in csv-processing-pipeline.md
    ```

## Example Categories

### 🔄 Workflows

Complete end-to-end workflows combining multiple packages:

- **[CSV Processing Pipeline](workflows/csv-processing-pipeline.md)** - Complete data processing from CSV parsing to validation and reporting
- **[Web Data Collection](workflows/web-data-collection.md)** - Collect and analyze web data from multiple sources
- **[Simple Data Processing](workflows/simple-data-processing.md)** - Minimal end-to-end example suitable for quick smoke testing

### 📦 Package Examples

Examples focused on specific packages:

- **[Email Notifications](packages/email-notifications.md)** - Comprehensive email sending examples with different providers

### 📊 Data Processing

Advanced data analysis and processing scenarios:

- **[Customer Risk Scoring](data-processing/customer-scoring.md)** - Build a complete customer risk assessment system

### 🤖 Automation

Automation and monitoring workflows:

- **[System Monitoring & Alerts](automation/monitoring-alerts.md)** - Comprehensive monitoring with automated alerting

## Sample Data

The `data/` directory contains sample datasets and output files:

- `sample_data.csv` - Sample CSV data for testing
- Various JSON files created by running examples

## Running Examples

### Prerequisites

1. **Maitask Server Running**

    ```bash
    maitask serve --port 8080
    ```

2. **Required Environment Variables** (for some examples)

    ```bash
    export SENDGRID_API_KEY="your_sendgrid_key"
    export SLACK_WEBHOOK_URL="your_slack_webhook"
    ```

3. **Configuration Setup**

    Maitask uses two types of configuration files:

    #### Application Configuration (`.maitask.json`)

    Purpose: Core application settings and execution policies

    **Global config**: `~/.maitask/config.json` (auto-created on first run)
    **Local dev config**: `.maitask.json` in project directory (overrides global)

    ```bash
    # Use local development config
    cp runtime/examples/maitask.example.json .maitask.json

    # Or use global config
    maitask init
    ```

    **What goes in `.maitask.json`:**
    - Workspace and packages directory paths
    - Execution policies (network access, security settings)
    - Resource limits (timeouts, file sizes)
    - Marketplace configuration

    #### Environment Variables (`.env`)

    Purpose: Deployment-specific settings and sensitive data

    ```bash
    # Create .env file for deployment settings
    cp runtime/examples/env.example .env

    # Common environment variables:
    RUST_LOG=info                    # Logging level
    HTTP_HOST=127.0.0.1              # HTTP server host
    HTTP_PORT=8080                   # HTTP server port
    GRPC_HOST=127.0.0.1              # gRPC server host
    GRPC_PORT=50051                  # gRPC server port
    PLANE_URL=http://localhost:8001  # Plane integration URL
    MAITASK_SERVICE_SECRET=your_secret_key   # Service authentication
    ```

    **What goes in `.env`:**
    - Server host/port configurations
    - Logging levels
    - API keys and secrets
    - Database connection strings
    - Deployment-specific settings

### Example Execution

Most examples can be run step-by-step by copying the commands from the example documentation files.

## Package Installation

### Local Packages (No Network Required)

```bash
# Core official packages
maitask install @maitask/csv-parser       # CSV document parsing
maitask install @maitask/file-processor   # File processing and transformation
maitask install @maitask/text-parser      # Text and markdown parsing

# Additional data processing packages
maitask install @maitask/data-validator   # Data validation and schema checking
maitask install @maitask/pdf-parser       # PDF document parsing
```

### Network-Enabled Packages

```bash
# Official packages with network capabilities
maitask install @maitask/email-sender          # Email sending with SMTP/API support
maitask install @maitask/slack-notifier        # Slack webhook notifications
maitask install @maitask/web-scraper           # Web scraping and content extraction
maitask install @maitask/web-search            # Web search functionality
maitask install @maitask/hackernews-crawler    # Hacker News API integration
maitask install @maitask/nocodb-importer       # NocoDB database integration
maitask install @maitask/github-integration    # GitHub API integration
```

### External Sources

```bash
# npm packages
maitask install npm:lodash

# Git repositories
maitask install https://github.com/user/custom-package.git

# Direct URLs
maitask install https://cdn.example.com/package.js
```

### API Installation

```bash
curl -X POST "http://localhost:8080/install" \
  -H "Content-Type: application/json" \
  -d '{"package": "csv-parser"}'
```

### Proxy Support

```bash
export HTTP_PROXY=http://proxy.company.com:8080
export HTTPS_PROXY=http://proxy.company.com:8080
maitask serve --port 8080
```

The Maitask Runtime automatically detects and uses proxy settings from environment variables.

## Common Patterns

### 1. Data Processing Pipeline

```text
CSV/JSON Input → Parse → Validate → Transform → Output
```

### 2. Monitoring Workflow

```text
Collect Metrics → Validate Thresholds → Alert → Log → Report
```

### 3. Notification System

```text
Event Trigger → Process Data → Format Message → Send Notification
```

## Best Practices

### Error Handling

```bash
# Check execution results
RESULT=$(curl -s -X POST "http://localhost:8080/packages/csv-parser/execute" ...)
if echo "$RESULT" | jq -e '.data.result.success' > /dev/null; then
  echo "Success"
else
  echo "Failed: $(echo "$RESULT" | jq -r '.error')"
fi
```

### Environment Management

```bash
# Use environment variables for sensitive data
export API_KEY="your_secret_key"

# Reference in API calls
curl -d '{"api_key": "'$API_KEY'"}'
```

### Output Management

```bash
# Save intermediate results
curl ... | jq '.data.result' > intermediate_data.json

# Chain operations
curl -d "$(cat intermediate_data.json)" ...
```

## Advanced Examples

### Custom Package Development

See [Customer Risk Scoring](data-processing/customer-scoring.md) for creating sophisticated custom packages with complex business logic, configurable parameters, and detailed reporting.

### Multi-Service Integration

See [System Monitoring](automation/monitoring-alerts.md) for examples of multi-data source monitoring with conditional logic and automated workflows.

## Troubleshooting

### Common Issues

1. **Server Not Running**

    ```bash
    curl http://localhost:8080/health
    ```

2. **Package Not Found**

    ```bash
    curl http://localhost:8080/packages
    ```

3. **Invalid JSON**

    ```bash
    echo '{"test": "data"}' | jq .
    ```

### Getting Help

1. Check package documentation: `maitask info package-name`
2. View API documentation: [docs/api.md](https://github.com/maitask/docs/blob/main/api.md)
3. Review error logs in Maitask output

## Contributing Examples

To add new examples:

1. Follow the existing directory structure
2. Include complete, runnable examples
3. Provide sample data when needed
4. Document prerequisites and expected outputs
5. Add entry to this README

## Next Steps

1. Explore [workflow examples](workflows/) for complete use cases
2. Check [package examples](packages/) for detailed usage
3. Review [data processing examples](data-processing/) for advanced scenarios
4. Look at [automation examples](automation/) for production monitoring

For more detailed documentation, visit the [documentation index](https://github.com/maitask/docs).
