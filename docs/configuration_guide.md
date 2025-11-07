# 配置指南

## 快速设置

将 `conf.yaml.example` 文件复制为 `conf.yaml`，然后根据你的具体需求修改配置。

```bash
cd deer-flow
cp conf.yaml.example conf.yaml
```

## DeerFlow 支持哪些模型？

在 DeerFlow 中，目前仅支持**非推理（non-reasoning）模型**。
这意味着诸如 OpenAI 的 o1/o3 或 DeepSeek 的 R1 等推理模型暂不支持，但未来会加入支持。
此外，由于缺乏工具调用能力，**Gemma-3 系列模型目前也不受支持**。

### 支持的模型

`doubao-1.5-pro-32k-250115`、`gpt-4o`、`qwen-max-latest`、`qwen3-235b-a22b`、`qwen3-coder`、`gemini-2.0-flash`、`deepseek-v3`，以及理论上任何其他兼容 OpenAI API 规范的非推理聊天模型。

> [!注意]
> 深度研究（Deep Research）流程要求模型具备**更长的上下文窗口**，并非所有模型都支持。
> 临时解决方案：
>
> * 可在网页右上角的设置中，将 “研究计划的最大步骤数（Max steps of a research plan）” 设置为 `2`；
> * 或在调用 API 时，将 `max_step_num` 设置为 `2`。

### 如何切换模型？

你可以在项目根目录下修改 `conf.yaml` 文件中的配置来切换模型，
配置格式遵循 [litellm 格式规范](https://docs.litellm.ai/docs/providers/openai_compatible)。

---

### 如何使用 OpenAI 兼容模型？

DeerFlow 支持与 **OpenAI-Compatible** 模型集成，即实现 OpenAI API 规范的模型。
包括多种开源与商业模型（如通义千问、Doubao、Gemini、DeepSeek 等）。
详细文档可参考 [litellm OpenAI-Compatible](https://docs.litellm.ai/docs/providers/openai_compatible)。

以下是使用 OpenAI-Compatible 模型的 `conf.yaml` 示例：

```yaml
# Doubao 模型（火山引擎）
BASIC_MODEL:
  base_url: "https://ark.cn-beijing.volces.com/api/v3"
  model: "doubao-1.5-pro-32k-250115"
  api_key: YOUR_API_KEY

# 阿里云通义千问模型
BASIC_MODEL:
  base_url: "https://dashscope.aliyuncs.com/compatible-mode/v1"
  model: "qwen-max-latest"
  api_key: YOUR_API_KEY

# DeepSeek 官方模型
BASIC_MODEL:
  base_url: "https://api.deepseek.com"
  model: "deepseek-chat"
  api_key: YOUR_API_KEY

# Google Gemini 模型（兼容 OpenAI 接口）
BASIC_MODEL:
  base_url: "https://generativelanguage.googleapis.com/v1beta/openai/"
  model: "gemini-2.0-flash"
  api_key: YOUR_API_KEY
```

以下是使用最优开源 OpenAI-Compatible 模型的配置示例：

```yaml
# 使用最新 deepseek-v3 处理基础任务（SOTA 模型）
BASIC_MODEL:
  base_url: https://api.deepseek.com
  model: "deepseek-v3"
  api_key: YOUR_API_KEY
  temperature: 0.6
  top_p: 0.90

# 使用 qwen3-235b-a22b 处理推理任务
REASONING_MODEL:
  base_url: https://dashscope.aliyuncs.com/compatible-mode/v1
  model: "qwen3-235b-a22b-thinking-2507"
  api_key: YOUR_API_KEY
  temperature: 0.6
  top_p: 0.90

# 使用 qwen3-coder-480b-a35b-instruct 处理编程任务
CODE_MODEL:
  base_url: https://dashscope.aliyuncs.com/compatible-mode/v1
  model: "qwen3-coder-480b-a35b-instruct"
  api_key: YOUR_API_KEY
  temperature: 0.6
  top_p: 0.90
```

此外，你需要在 `src/config/agents.py` 中配置 `AGENT_LLM_MAP`，
以便为不同的智能体分配正确的模型。例如：

```python
# 定义智能体与模型的映射关系
AGENT_LLM_MAP: dict[str, LLMType] = {
    "coordinator": "reasoning",
    "planner": "reasoning",
    "researcher": "reasoning",
    "coder": "basic",
    "reporter": "basic",
    "podcast_script_writer": "basic",
    "ppt_composer": "basic",
    "prose_writer": "basic",
    "prompt_enhancer": "basic",
}
```

---

### 如何使用自签名 SSL 证书的模型？

如果你的 LLM 服务器使用了自签名 SSL 证书，可以通过在配置中添加
`verify_ssl: false` 来关闭 SSL 证书验证：

```yaml
BASIC_MODEL:
  base_url: "https://your-llm-server.com/api/v1"
  model: "your-model-name"
  api_key: YOUR_API_KEY
  verify_ssl: false  # 关闭 SSL 验证
```

> [!警告]
> 关闭 SSL 验证会降低安全性，仅应在开发环境或信任的服务器中使用。
> 在生产环境中，建议使用正规签发的 SSL 证书。

---

### 如何使用 Ollama 模型？

DeerFlow 支持与 **Ollama** 模型集成。可参考 [litellm Ollama 文档](https://docs.litellm.ai/docs/providers/ollama)。
以下为示例配置（需先运行 `ollama serve`）：

```yaml
BASIC_MODEL:
  model: "model-name"  # 例如 qwen3:8b, mistral-small3.1:24b
  base_url: "http://localhost:11434/v1"
  api_key: "whatever"  # 伪造 API key，可随意填写
```

---

### 如何使用 OpenRouter 模型？

DeerFlow 支持与 **OpenRouter** 模型集成。参考 [litellm OpenRouter](https://docs.litellm.ai/docs/providers/openrouter)。

配置步骤如下：

1. 从 [OpenRouter 官网](https://openrouter.ai/) 获取 `OPENROUTER_API_KEY` 并设置到环境变量。
2. 在模型名前添加前缀 `openrouter/`。
3. 配置正确的 OpenRouter 基础 URL。

示例：

```ini
# 在环境变量或 .env 文件中配置
OPENROUTER_API_KEY=""
```

```yaml
BASIC_MODEL:
  model: "openrouter/google/palm-2-chat-bison"
```

> 注意：可用模型名称可能随时间变化，请在
> [OpenRouter 官方文档](https://openrouter.ai/docs) 中核实最新模型名称。

---

### 如何使用 Azure OpenAI 模型？

DeerFlow 支持与 **Azure OpenAI Chat** 模型集成。
参考 [AzureChatOpenAI 文档](https://python.langchain.com/api_reference/openai/chat_models/langchain_openai.chat_models.azure.AzureChatOpenAI.html)。

示例配置：

```yaml
BASIC_MODEL:
  model: "azure/gpt-4o-2024-08-06"
  azure_endpoint: $AZURE_OPENAI_ENDPOINT
  api_version: $OPENAI_API_VERSION
  api_key: $AZURE_OPENAI_API_KEY
```

---

## 关于搜索引擎

### 如何控制 Tavily 搜索的域名范围？

DeerFlow 允许通过配置文件控制 Tavily 搜索结果的包含与排除域。
这样可以聚焦可信来源，提高搜索质量，减少幻觉。

> 提示：目前仅支持 Tavily。

在 `conf.yaml` 中配置如下：

```yaml
SEARCH_ENGINE:
  engine: tavily
  # 只包含以下域（白名单）
  include_domains:
    - trusted-news.com
    - gov.org
    - reliable-source.edu
  # 排除以下域（黑名单）
  exclude_domains:
    - unreliable-site.com
    - spam-domain.net
```

---
