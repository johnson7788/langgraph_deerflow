# 后端代码梳理

目录
```
src
├── __init__.py
├── agents
│   ├── __init__.py
│   └── agents.py
├── config
│   ├── __init__.py
│   ├── agents.py
│   ├── configuration.py
│   ├── loader.py
│   ├── questions.py
│   ├── report_style.py
│   └── tools.py
配置系统（src/config）
configuration.py/loader.py：加载 conf.yaml 与 .env，把模型、工具、RAG、搜索引擎、Agent 策略等合并成运行时配置。
agents.py / tools.py / questions.py / report_style.py：把“Agent 套餐”“可用工具集”“内置问题列表”“报告风格模板”等以结构化配置提供给后端与前端。前端页面通常会通过 config_request.py 取这些枚举/模板做 UI。官方 README 也展示了“交互模式内置问题”“报告样式/问题选择”等能力

├── crawler
│   ├── __init__.py
│   ├── article.py
│   ├── crawler.py
│   ├── jina_client.py
│   └── readability_extractor.py
├── graph
│   ├── __init__.py
│   ├── builder.py
│   ├── checkpoint.py
│   ├── nodes.py
│   └── types.py
builder.py：构建基础 StateGraph，注册所有节点与边；LangGraph 允许节点返回 Command(goto=...) 来路由。因此后来在 Issue #669 把对 coordinator 额外 add_conditional_edges 的冗余路由去掉，以免与节点内部路由冲突并影响可视化。
GitHub
nodes.py：每个业务节点的“执行体”。其中 coordinator_node(state, config) -> Command[Literal["planner", "background_investigator", "coordinator", "__end__"]] 的签名在 Issue 讨论里被引用过——说明它在节点内决定下一跳（如需要澄清则回到 coordinator，已清晰则进 planner，或直接结束）。
GitHub
types.py：LangGraph 的 State/事件类型定义。
checkpoint.py：会话级状态存储（最初为内存实现），用于“线程/对话”上下文与中间产物的持久；社区多篇解析都提到它的会话管理与状态回放能力。
CSDN
+1
builder.py 最终返回 CompiledGraph（或等价封装），app.py 中通过 graph.astream() 逐 tick 拉取事件并转成 SSE/分块流式响应。报错栈可见 langgraph.pregel.runner.atick() → _panic_or_proceed() 的调用链。


├── llms
│   ├── __init__.py
│   ├── llm.py
│   └── providers
│       └── dashscope.py
llm.py：统一构造/包装 ChatModel（多为 LangChain ChatModel 的一层），支持从 conf.yaml 读取模型名、温度、额外参数。
providers/dashscope.py：封装达摩院 DashScope（通义千问）接口，以统一 API 适配到 LangChain/LangGraph 管线。多篇文章提及项目基于 LangChain + LangGraph，并兼容多家模型

├── podcast
│   ├── graph
│   │   ├── audio_mixer_node.py
│   │   ├── builder.py
│   │   ├── script_writer_node.py
│   │   ├── state.py
│   │   └── tts_node.py
│   └── types.py
├── ppt
│   └── graph
│       ├── builder.py
│       ├── ppt_composer_node.py
│       ├── ppt_generator_node.py
│       └── state.py
├── prompt_enhancer
│   ├── __init__.py
│   └── graph
│       ├── builder.py
│       ├── enhancer_node.py
│       └── state.py
├── prompts
│   ├── __init__.py
│   ├── coder.md
│   ├── coordinator.md
│   ├── planner.md
│   ├── planner_model.py
│   ├── podcast
│   │   └── podcast_script_writer.md
│   ├── ppt
│   │   └── ppt_composer.md
│   ├── prompt_enhancer
│   │   └── prompt_enhancer.md
│   ├── prose
│   │   ├── prose_continue.md
│   │   ├── prose_fix.md
│   │   ├── prose_improver.md
│   │   ├── prose_longer.md
│   │   ├── prose_shorter.md
│   │   └── prose_zap.md
│   ├── reporter.md
│   ├── researcher.md
│   └── template.py
├── prose
│   └── graph
│       ├── builder.py
│       ├── prose_continue_node.py
│       ├── prose_fix_node.py
│       ├── prose_improve_node.py
│       ├── prose_longer_node.py
│       ├── prose_shorter_node.py
│       ├── prose_zap_node.py
│       └── state.py
├── rag
│   ├── __init__.py
│   ├── builder.py
│   ├── ragflow.py
│   ├── retriever.py
│   └── vikingdb_knowledge_base.py
├── server
│   ├── __init__.py
│   ├── app.py
│   ├── chat_request.py
│   ├── config_request.py
│   ├── mcp_request.py
│   ├── mcp_utils.py
│   └── rag_request.py
app.py：创建 FastAPI 应用、注册路由与 CORS、中间件；核心是流式工作流，会把 ChatRequest 传入图并 astream 迭代 LangGraph 事件，逐步写到 HTTP 响应流里。相关报错栈清晰显示了 app.py 中的 _astream_workflow_generator 正在消费 graph.astream()。
GitHub

chat_request.py：Pydantic 的请求体定义，如 messages、thread_id、auto_accepted_plan、feedback（文档示例里就出现了这几个参数）。
GitHub

rag_request.py：RAG 查询相关的入参/出参模型与路由处理器。

config_request.py：把后端可配置项（如可选问题、报告样式、工具清单、模型列表）以 API 暴露给前端。

mcp_request.py / mcp_utils.py：MCP（Model Context Protocol）工具发现/调度的 REST 外壳，使 Agent 能动态加载外部 MCP Server 暴露的工具。社区文章与掘金解析都提到运行期动态装配工具的模式。


├── tools
│   ├── __init__.py
│   ├── crawl.py
│   ├── decorators.py
│   ├── python_repl.py
│   ├── retriever.py
│   ├── search.py
│   ├── tavily_search
│   │   ├── __init__.py
│   │   ├── tavily_search_api_wrapper.py
│   │   └── tavily_search_results_with_images.py
│   └── tts.py
search.py / tavily_search/*：统一搜索抽象，按 .env 的 SEARCH_API 切换 Tavily、DuckDuckGo、Brave、Arxiv、SearxNG。README 写了如何在 .env 里选择后端与配置 API Key。
GitHub

crawler/*：含 readability_extractor.py（正文抽取）、crawler.py（抓取主流程）、article.py（数据结构）、jina_client.py（Jina 相关的客户端/解析增强）。用于把搜索到的 URL 转成结构化“证据”。（多篇解析都把“联网抓取+可读性处理”列作核心能力。）
CSDN

retriever.py / rag/*：封装私域知识检索，支持 RAGFlow/VikingDB；README 里有 RAGFlow 的环境变量配置示例。
GitHub

python_repl.py：受限的 Python 执行器，用于数据整理/小计算（官方 README 强调生产部署要审查 Python REPL/MCPServer 的安全）。
GitHub

tts.py：调用火山引擎 TTS，服务端暴露 /api/tts，官方文档有 curl 调用示例。
GitHub

decorators.py：常见是给工具添加统一日志、限流、重试、异常包装。


├── utils
│   ├── __init__.py
│   └── json_utils.py
└── workflow.py
```

# Web 层（FastAPI） -> src/server
src/server 定义 FastAPI 应用与一组 API（如 chat 流式接口、RAG、配置查询、TTS 等）。对外暴露 HTTP，内部则把请求转成工作流输入，并以流式方式把 LangGraph 的事件回推给客户端。官方文档明确 /api/tts 与如何以 Docker/Compose 起后端服务；同时文档与社区总结指出 /api/chat/stream 是多智能体研究的主入口，由 chat_stream() 处理并通过

# 编排层（LangGraph）：src/graph
用 LangGraph 的 StateGraph 把多个节点（coordinator、background_investigator、planner、research_team、reporter…）连成一条多阶段研究流水线；节点函数通过返回 Command(goto=...) 显式控制下一跳，避免把路由写死在图上。

# 智能体与工具层：
智能体：src/agents 封装了研究员/编码等 Agent 的角色与工具绑定策略。
工具：src/tools 提供联网检索（Tavily/DuckDuckGo/Brave/Arxiv/SearxNG）、网页抓取/可读性抽取、私域检索（RAGFlow / VikingDB）、Python REPL 执行、TTS 等，供节点按需调用。README 对搜索引擎与 TTS 的后端集成有专门说明。

# LLM 抽象：
src/llms 统一了模型调用（包含 DashScope Provider），后端能按配置切换 OpenAI、阿里通义、或本地/代理模型，社区文章也多次强调其基于 LangChain + LangGraph 的栈。

# RAG 子系统：src/rag 
集成 RAGFlow 与 VikingDB 的知识库接入、检索与构建，作为工具层的“私域知识”来源。



# 一次典型请求链路（以 /api/chat/stream 为例）
前端把用户消息、thread_id、是否 auto_accepted_plan、可能的 feedback 等打包成 ChatRequest POST 到 /api/chat/stream。文档示例中的字段与服务端请求体名称一致。
GitHub

app.py 校验入参后，构造/获取本次线程的 State（checkpoint 里按 thread_id 取/建），然后拿到 graph 调用 graph.astream(...)。每个 tick 产生一个事件（哪个节点、产出/日志/思考片段），服务端把它按流式分块写回。报错栈也印证了这条执行路径。
GitHub

LangGraph 依次触发节点：

coordinator 判断问题是否需要澄清/预检，选择 goto 下一节点；

background_investigator 做初检索/爬取；

planner 生成研究计划（如果配置“人审”，会等待“[ACCEPTED]/[EDIT PLAN]”反馈）；

research_team/coder 分步执行：按步骤动态装配MCP 工具与默认工具（搜索/抓取/私域检索/Python 执行…），每步把证据与中间结论写回状态；掘金解析中有对**“基于配置动态加载 MCP 工具 + 执行步骤”**的函数说明。

reporter 汇总证据、产出最终报告与附件（如图片、引用）。

结束：节点返回 Command(goto="__end__")，服务端关闭流。本线程的中间状态保留在 checkpoint，以便后续增量追问或生成播客/PPT。

