---
name: crawl4ai
description: 当你需要爬取网页、提取内容为markdown或抓取多个URL时使用。封装Crawl4AI REST API以绕过MCP SSE问题。
---

# Crawl4AI 技能

## 概述

Crawl4AI 是一个对LLM友好的网页爬虫，可以将网页转换为干净的markdown格式。此技能提供直接的REST API访问，以绕过MCP SSE传输问题。

**核心原则：** REST API直接调用 = 可靠的网页抓取。

**开始时声明：** "我正在使用crawl4ai技能来抓取网页内容。"

## 服务器配置

默认服务器：`http://192.168.50.178:11235`

要更改服务器，请设置环境变量：
```bash
export CRAWL4AI_SERVER="http://your-server:11235"
```

## 可用命令

### 1. `/md` - 提取Markdown

将URL转换为干净的、LLM可用的markdown。

**用法：**
```bash
# 基本用法 - 返回过滤后的markdown
curl -X POST $CRAWL4AI_SERVER/md \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com", "f": "fit"}'

# 原始markdown（无过滤）
curl -X POST $CRAWL4AI_SERVER/md \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com", "f": "raw"}'

# 使用查询进行BM25过滤内容
curl -X POST $CRAWL4AI_SERVER/md \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com", "f": "bm25", "q": "机器学习"}'

# LLM过滤内容
curl -X POST $CRAWL4AI_SERVER/md \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com", "f": "llm", "q": "提取价格信息"}'
```

**过滤器类型：**
- `fit` - 基于启发式的噪声过滤（默认，AI友好）
- `raw` - 完整的markdown，无过滤
- `bm25` - 基于查询的BM25算法
- `llm` - 基于查询的LLM驱动的内容提取

**响应格式：**
```json
{
  "url": "https://example.com",
  "filter": "fit",
  "markdown": "# 干净的markdown内容...",
  "success": true
}
```

### 2. `/crawl` - 批量爬取

并行爬取多个URL。

**用法：**
```bash
# 单个URL
curl -X POST $CRAWL4AI_SERVER/crawl \
  -H "Content-Type: application/json" \
  -d '{"urls": ["https://example.com"]}'

# 多个URL
curl -X POST $CRAWL4AI_SERVER/crawl \
  -H "Content-Type: application/json" \
  -d '{"urls": ["https://example.com", "https://another.com"]}'
```

**响应格式：**
```json
{
  "results": [
    {
      "url": "https://example.com",
      "markdown": "...",
      "success": true
    }
  ]
}
```

## 最佳实践

1. **一般抓取使用 `fit` 过滤器** - 在干净度和完整性之间取得最佳平衡
2. **需要全部内容时使用 `raw`** - 保留所有内容，包括导航/页脚
3. **针对提取使用 `bm25`** - 比LLM更快，适合基于关键词的过滤
4. **在请求之间添加延迟** - 尊重目标服务器
5. **优雅地处理错误** - 检查响应中的 `success` 字段

## 常见用例

**抓取文档：**
```bash
curl -X POST $CRAWL4AI_SERVER/md \
  -H "Content-Type: application/json" \
  -d '{"url": "https://docs.example.com/guide", "f": "fit"}'
```

**提取文章内容：**
```bash
curl -X POST $CRAWL4AI_SERVER/md \
  -H "Content-Type: application/json" \
  -d '{"url": "https://blog.example.com/post", "f": "bm25", "q": "文章主要内容"}'
```

**研究多个来源：**
```bash
curl -X POST $CRAWL4AI_SERVER/crawl \
  -H "Content-Type: application/json" \
  -d '{"urls": ["https://site1.com", "https://site2.com", "https://site3.com"]}'
```

## 错误处理

始终检查响应：
```bash
response=$(curl -s -X POST $CRAWL4AI_SERVER/md \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com", "f": "fit"}')

# 检查成功状态
success=$(echo "$response" | jq -r '.success')

if [ "$success" = "true" ]; then
  echo "$response" | jq -r '.markdown'
else
  echo "错误：无法抓取URL"
  echo "$response" | jq -r '.error // "未知错误"'
fi
```

## 故障排除

**连接被拒绝：**
- 检查Crawl4AI服务器是否正在运行：`curl $CRAWL4AI_SERVER/monitor/health`
- 验证CRAWL4AI_SERVER环境变量中的服务器地址

**markdown为空：**
- 尝试使用 `f: "raw"` 而不是 `fit`
- 检查URL是否需要JavaScript渲染
- 验证URL是否可访问

**速率限制：**
- 在请求之间添加延迟
- 考虑服务器容量
- 检查浏览器池状态：`curl $CRAWL4AI_SERVER/monitor/browsers`
