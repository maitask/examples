# Web 数据采集

[English](web-data-collection.md) | [中文](web-data-collection_zh-CN.md)

使用可联网包采集公开 Web 数据，并落地为统一报告。

## 适用范围

- Runtime HTTP API（`:8080`）
- 包：`@maitask/hackernews-crawler`、`@maitask/web-search`
- 输出适配器：`file`

## 前置条件

- Runtime 网络策略需允许外部访问。
- 如有需要，请在 Runtime 配置中设置 `execution.allow_network=true` 与 `execution.allowed_hosts`。

```bash
RUNTIME_URL="http://localhost:8080"
HN_PKG='@maitask/hackernews-crawler'
HN_PKG_URL='@maitask%2Fhackernews-crawler'
SEARCH_PKG='@maitask/web-search'
SEARCH_PKG_URL='@maitask%2Fweb-search'
```

安装包：

```bash
curl -sS -X POST "${RUNTIME_URL}/packages/install" -H 'Content-Type: application/json' -d "{\"package\":\"${HN_PKG}\"}" | jq
curl -sS -X POST "${RUNTIME_URL}/packages/install" -H 'Content-Type: application/json' -d "{\"package\":\"${SEARCH_PKG}\"}" | jq
```

## 步骤

1. 拉取 Hacker News 热门帖子。

```bash
curl -sS -X POST "${RUNTIME_URL}/packages/${HN_PKG_URL}/execute" \
  -H 'Content-Type: application/json' \
  -d '{"input":{"storyType":"top","limit":10,"includeComments":false}}' \
  > /tmp/hn_result.json
```

2. 执行 Web 搜索。

```bash
curl -sS -X POST "${RUNTIME_URL}/packages/${SEARCH_PKG_URL}/execute" \
  -H 'Content-Type: application/json' \
  -d '{"input":{"query":"Maitask runtime API","engine":"duckduckgo","limit":10,"language":"en","region":"us"}}' \
  > /tmp/search_result.json
```

3. 生成合并报告。

```bash
jq -n \
  --arg generated_at "$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
  --argjson hn "$(cat /tmp/hn_result.json)" \
  --argjson search "$(cat /tmp/search_result.json)" \
  '{generated_at:$generated_at,hackernews:$hn,web_search:$search}' \
  > /tmp/web_collection_report.json
```

4. 通过文件适配器持久化。

```bash
curl -sS -X POST "${RUNTIME_URL}/output" \
  -H 'Content-Type: application/json' \
  -d "{\"adapter\":\"file\",\"data\":$(cat /tmp/web_collection_report.json),\"config\":{\"path\":\"/tmp/maitask-web-report.json\",\"format\":\"json_pretty\"}}" \
  | jq
```

## 验证

```bash
jq -e '.success == true' /tmp/hn_result.json
jq -e '.success == true' /tmp/search_result.json
jq -e '.success == true' /tmp/maitask-web-report.json
```

## 边界说明

- 外部数据源行为会随时间变化（结果数量、排序、延迟）。
- 面向用户的执行入口建议统一经过 Plane。
