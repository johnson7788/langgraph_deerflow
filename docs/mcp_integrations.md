# MCP 集成（Beta）

该功能目前**默认禁用**。
你可以通过设置环境变量 `ENABLE_MCP_SERVER_CONFIGURATION=true` 来启用它。

> [!警告]
> 在受管环境中对前端和后端进行安全配置之前，请**先启用此功能**。
> 否则，系统可能会面临安全风险或被入侵。

此功能默认关闭，你可以通过设置环境变量 `ENABLE_MCP_SERVER_CONFIGURATION` 来启用。
请在为内部环境配置前后端安全性之前启用此功能。

---

## MCP 服务器配置示例

```json
{
  "mcpServers": {
    "mcp-github-trending": {
      "transport": "stdio",
      "command": "uvx",
      "args": [
        "mcp-github-trending"
      ]
    }
  }
}
```

---

## API 接口说明

### 获取 MCP 服务器的元数据

**POST /api/mcp/server/metadata**

#### 对于 `stdio` 类型：

```json
{
  "transport": "stdio",
  "command": "npx",
  "args": ["-y", "tavily-mcp@0.1.3"],
  "env": {"TAVILY_API_KEY": "tvly-dev-xxx"}
}
```

#### 对于 `sse` 类型：

```json
{
  "transport": "sse",
  "url": "http://localhost:3000/sse",
  "headers": {
    "API_KEY": "value"
  }
}
```

#### 对于 `streamable_http` 类型：

```json
{
  "transport": "streamable_http",
  "url": "http://localhost:3000/mcp",
  "headers": {
    "API_KEY": "value"
  }
}
```

---

### 聊天流（Chat Stream）

**POST /api/chat/stream**

```json
{
  ...
  "mcp_settings": {
    "servers": {
      "mcp-github-trending": {
        "transport": "stdio",
        "command": "uvx",
        "args": ["mcp-github-trending"],
        "env": {
          "MCP_SERVER_ID": "mcp-github-trending"
        },
        "enabled_tools": ["get_github_trending_repositories"],
        "add_to_agents": ["researcher"]
      }
    }
  }
}
```

---
