# 在线稿子源接口模板

用户可选择通过在线接口获取故事脚本，而不是使用本地文件。

## 接口配置格式

在 `config.json` 中配置：

```json
{
  "script_source": {
    "mode": "api",
    "api_url": "https://example.com/api/script",
    "api_key": "user-provided-api-key",
    "method": "GET",
    "params": {
      "id": "script-id"
    },
    "headers": {
      "Authorization": "Bearer <api_key>",
      "Content-Type": "application/json"
    },
    "response_path": "data.content",
    "cache_enabled": true,
    "cache_ttl": 3600
  }
}
```

## 字段说明

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `mode` | string | ✅ | 固定为 `"api"` |
| `api_url` | string | ✅ | 接口完整 URL |
| `api_key` | string | ✅ | 用户提供的 API 密钥 |
| `method` | string | ❌ | HTTP 方法，默认 `"GET"` |
| `params` | object | ❌ | URL 查询参数 |
| `headers` | object | ❌ | 自定义请求头 |
| `response_path` | string | ❌ | 响应 JSON 中提取脚本内容的路径，支持点号分隔，默认 `"data.content"` |
| `cache_enabled` | bool | ❌ | 是否缓存接口结果，默认 `true` |
| `cache_ttl` | int | ❌ | 缓存有效期（秒），默认 `3600` |

## 获取流程

1. 根据 `api_url` + `params` 构造请求
2. 携带 `api_key` 和 `headers` 发起 HTTP 请求
3. 根据 `response_path` 从响应 JSON 中提取脚本内容
4. 如果启用缓存，检查本地缓存是否有效
5. 将获取到的脚本内容写入临时文件，进入标准渲染流程

## 安全要求

- API Key 必须存储在用户本地 `config.json` 中
- 导出项目时必须过滤掉 `api_key` 字段
- 日志中不得打印完整 API Key

## 故障处理

| 错误 | 处理方式 |
|------|---------|
| 401 Unauthorized | 提示 API Key 无效，要求重新配置 |
| 403 Forbidden | 提示权限不足 |
| 404 Not Found | 提示脚本 ID 不存在或接口地址错误 |
| 429 Too Many Requests | 提示请求频率超限，稍后重试 |
| Timeout | 提示请求超时，检查网络连接 |
| Invalid Response | 提示响应格式异常，检查 `response_path` 配置 |

## 示例调用

```bash
# GET 请求示例
curl -X GET "https://example.com/api/script?id=abc123" \
  -H "Authorization: Bearer user-api-key" \
  -H "Content-Type: application/json"

# 预期响应
{
  "data": {
    "content": "# 故事标题\n\n全校觉醒日..."
  }
}
```
