# Maitask 示例

[English](README.md) | [中文](README_zh-CN.md)

全面的示例和模板。学习如何使用 Maitask Runtime 与官方包、创建自定义工作流以及与第三方服务集成。

## 目录结构

```text
examples/
├── README.md                    # 本文件
├── workflows/                   # 完整工作流示例
├── packages/                    # 特定包示例
├── data-processing/            # 数据分析和处理示例
├── automation/                 # 自动化和监控示例
└── data/                       # 示例数据文件和输出
```

## 快速开始

1. **启动 Maitask 服务器**

    ```bash
    maitask serve --port 8080
    ```

2. **安装所需包**

    ```bash
    # 安装官方包（使用 @maitask/ 前缀）
    maitask install @maitask/csv-parser
    maitask install @maitask/file-processor
    maitask install @maitask/text-parser
    ```

3. **运行您的第一个示例**

    ```bash
    cd examples/workflows
    # 按照 csv-processing-pipeline.md 中的分步指南操作
    ```

## 示例类别

### 🔄 工作流 (Workflows)

组合多个包的完整端到端工作流：

- **[CSV 处理管道](workflows/csv-processing-pipeline.md)** - 从 CSV 解析到验证和报告的完整数据处理
- **[Web 数据收集](workflows/web-data-collection.md)** - 从多个来源收集和分析 Web 数据
- **[简易数据处理](workflows/simple-data-processing.md)** - 适用于快速冒烟测试的最小端到端示例

### 📦 包示例 (Package Examples)

专注于特定包的示例：

- **[邮件通知](packages/email-notifications.md)** - 使用不同提供商的全面邮件发送示例

### 📊 数据处理 (Data Processing)

高级数据分析和处理场景：

- **[客户风险评分](data-processing/customer-scoring.md)** - 构建完整的客户风险评估系统

### 🤖 自动化 (Automation)

自动化和监控工作流：

- **[系统监控与告警](automation/monitoring-alerts.md)** - 全面的自动告警监控

## 示例数据

`data/` 目录包含示例数据集和输出文件：

- `sample_data.csv` - 用于测试的示例 CSV 数据
- 运行示例创建的各种 JSON 文件

## 运行示例

### 前置要求

1. **Maitask 服务器运行中**

    ```bash
    maitask serve --port 8080
    ```

2. **所需环境变量**（某些示例需要）

    ```bash
    export SENDGRID_API_KEY="your_sendgrid_key"
    export SLACK_WEBHOOK_URL="your_slack_webhook"
    ```

### 示例执行

大多数示例可以通过从示例文档文件复制命令来逐步运行。

## 包安装

### 本地包（无需网络）

```bash
# 核心官方包
maitask install @maitask/csv-parser       # CSV 文档解析
maitask install @maitask/file-processor   # 文件处理和转换
maitask install @maitask/text-parser      # 文本和 Markdown 解析

# 其他数据处理包
maitask install @maitask/data-validator   # 数据验证和模式检查
maitask install @maitask/pdf-parser       # PDF 文档解析
```

### 启用网络的包

```bash
# 具有网络功能的官方包
maitask install @maitask/email-sender          # 支持 SMTP/API 的邮件发送
maitask install @maitask/slack-notifier        # Slack webhook 通知
maitask install @maitask/web-scraper           # 网页抓取和内容提取
maitask install @maitask/web-search            # 网页搜索功能
maitask install @maitask/hackernews-crawler    # Hacker News API 集成
maitask install @maitask/nocodb-importer       # NocoDB 数据库集成
maitask install @maitask/github-integration    # GitHub API 集成
```

### 外部源

```bash
# npm 包
maitask install npm:lodash

# Git 仓库
maitask install https://github.com/user/custom-package.git

# 直接 URL
maitask install https://cdn.example.com/package.js
```

### API 安装

```bash
curl -X POST "http://localhost:8080/install" \
  -H "Content-Type: application/json" \
  -d '{"package": "csv-parser"}'
```

### 代理支持

```bash
export HTTP_PROXY=http://proxy.company.com:8080
export HTTPS_PROXY=http://proxy.company.com:8080
maitask serve --port 8080
```

Maitask Runtime 自动检测并使用环境变量中的代理设置。

## 常见模式

### 1. 数据处理管道

```text
CSV/JSON 输入 → 解析 → 验证 → 转换 → 输出
```

### 2. 监控工作流

```text
收集指标 → 验证阈值 → 告警 → 日志 → 报告
```

### 3. 通知系统

```text
事件触发 → 处理数据 → 格式化消息 → 发送通知
```

## 最佳实践

### 错误处理

```bash
# 检查执行结果
RESULT=$(curl -s -X POST "http://localhost:8080/packages/csv-parser/execute" ...)
if echo "$RESULT" | jq -e '.data.result.success' > /dev/null; then
  echo "成功"
else
  echo "失败：$(echo "$RESULT" | jq -r '.error')"
fi
```

### 环境管理

```bash
# 对敏感数据使用环境变量
export API_KEY="your_secret_key"

# 在 API 调用中引用
curl -d '{"api_key": "'$API_KEY'"}'
```

### 输出管理

```bash
# 保存中间结果
curl ... | jq '.data.result' > intermediate_data.json

# 链式操作
curl -d "$(cat intermediate_data.json)" ...
```

## 高级示例

### 自定义包开发

参见[客户风险评分](data-processing/customer-scoring.md)，了解如何创建具有复杂业务逻辑、可配置参数和详细报告的复杂自定义包。

### 多服务集成

参见[系统监控](automation/monitoring-alerts.md)，了解具有条件逻辑和自动化工作流的多数据源监控示例。

## 故障排除

### 常见问题

1. **服务器未运行**

    ```bash
    curl http://localhost:8080/health
    ```

2. **包未找到**

    ```bash
    curl http://localhost:8080/packages
    ```

3. **无效 JSON**

    ```bash
    echo '{"test": "data"}' | jq .
    ```

### 获取帮助

1. 查看包文档：`maitask info package-name`
2. 查看 API 文档：[docs/api.md](https://github.com/maitask/docs/blob/main/api.md)
3. 查看 Maitask 输出中的错误日志

## 贡献示例

要添加新示例：

1. 遵循现有目录结构
2. 包含完整的可运行示例
3. 在需要时提供示例数据
4. 记录前置要求和预期输出
5. 在此 README 中添加条目

## 下一步

1. 探索[工作流示例](workflows/)了解完整用例
2. 查看[包示例](packages/)了解详细用法
3. 查看[数据处理示例](data-processing/)了解高级场景
4. 查看[自动化示例](automation/)了解生产监控

更多详细资料请访问[文档索引](https://github.com/maitask/docs)。
