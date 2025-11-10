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



# podcast目录
input(str)
   │
   ▼
script_writer_node ──(Script)──► tts_node ──(list[bytes])──► audio_mixer_node ──(bytes)──► output(mp3)


# Agents目录
只有1个agents文件，创建一个 ReAct 风格的智能体（agent），它把模型选择、提示词模板渲染、工具注入等细节封装起来，外部只要给几个字符串与对象就能拿到可用的 agent。

# config目录
加载环境变量、读取/处理 YAML 配置、暴露内置问法、搜索/RAG 选择项、团队成员元数据，以及把这些配置项以统一方式导出给其他模块使用。
作用：作为 config 包的总出口，集中加载 .env，并把常用配置对象/函数 re-export。

# crawler
crawler/* 代码把“网页 URL → 结构化文章 → 适合丢给 LLM 的消息块（文本 + 图片）”这一件事打通了：先用 Jina 抓取 HTML，再用 Readability 提取正文，再把 HTML 转成 Markdown，并把 Markdown 中的图片拆分成独立的“图片消息”。

# Graph
基于 LangGraph 的多智能体研究/写作流水线，带可选的持久化记忆与自定义检查点存储。
这是一个有向状态图（state graph）：节点是“代理角色”（协调者/规划者/研究员/程序员/记者等），边表示节点间的流转逻辑。

工作内存通过 State（继承自 MessagesState）在节点间传递，里面装着语言、主题、计划、观察结果、是否启用背景调查等。

两种“记忆”：

LangGraph 的内置 MemorySaver（易用）

自定义的 ChatStreamManager（可选，用 MongoDB 或 PostgreSQL 把流式消息持久化）

2) graph/types.py

定义了全局状态 State（继承 MessagesState）：

运行期字段（会在节点间传递）：

locale: 语言（默认 en-US）

research_topic: 研究主题

observations: list[str]: 步骤执行产生的“观察/发现”

resources: list[Resource]: RAG 资源（本地检索工具会用到）

plan_iterations: int: 规划迭代计数

current_plan: Plan | str: 当前计划（结构化 Plan 或原始 JSON 字符串）

final_report: str: 最终报告

auto_accepted_plan: bool: 是否自动接受计划（绕过人工反馈）

enable_background_investigation: bool: 是否在规划前做背景检索

background_investigation_results: str: 背景检索结果（字符串化）

Plan、Resource 来自项目内 src.*。


3) graph/builder.py

负责装配状态图和条件流转：

continue_to_running_research_team(state: State)

在“研究小组”执行循环中，决定下一跳：

若 current_plan 不存在或无步骤 → 回到 planner

若所有步骤都有 execution_res → 回到 planner（表明需要新计划/总结）

否则取第一个未完成的步骤：

StepType.RESEARCH → 去 researcher

StepType.PROCESSING → 去 coder

其他类型 → 兜底到 planner

_build_base_graph()

节点：

coordinator（协调对话/收集需求/触发 handoff）

background_investigator（可选的预检索）

planner（产出完整计划）

human_feedback（可选的人类审阅/修改计划）

research_team（调度循环）

researcher（上网/本地检索、写发现）

coder（代码/数据处理）

reporter（按规范生成最终报告）

边：

START → coordinator

background_investigator → planner

research_team --(conditional)--> planner|researcher|coder（用上面的 continue_to_running_research_team）

reporter → END

build_graph_with_memory() / build_graph()

前者使用 MemorySaver 做 LangGraph 的 checkpoint（记录对话/状态，便于恢复）

后者是纯内存态（进程内）

运行主线（典型流程）

START → coordinator：收集主题、语言；如果协调者调用了 handoff_to_planner：

若 enable_background_investigation=True：→ background_investigator → planner，否则直接 → planner

planner 生成计划

若“不足上下文” → → human_feedback（人审/编辑/接受）→ → research_team

若“已足够” → → reporter（直接产出报告）→ END

research_team 根据 continue_to_running_research_team(...)：

有未完成步骤且类型为 RESEARCH → researcher 执行 → 回 research_team

类型为 PROCESSING → coder 执行 → 回 research_team

步骤都完成 → planner（可生成新计划或触发 reporter）

reporter 汇总 observations+计划信息，按格式生成最终报告 → END

# llms
统一从 conf.yaml 与 环境变量 读取各类 LLM 的连接配置；

依据类型（reasoning/basic/vision/code）实例化对应的 LangChain ChatModel；

支持 Azure OpenAI、OpenAI 兼容接口、DeepSeek、阿里 DashScope（经自定义适配器）；

实例缓存，避免重复创建连接对象。
保留 reasoning_content
一些“推理类”模型会在响应里附带 reasoning_content（思维链/草稿）。类中覆写：

_create_chat_result(...)：若响应对象是 openai.BaseModel 并含 choices[0].message.reasoning_content，则把它塞进 generations[0].message.additional_kwargs["reasoning_content"]，供上层取用。

更健壮的流式处理
DashScope 可能提供两种风格的流：

标准 chat.completions 的 streaming；

beta.chat.completions.stream 的分块（type=...、content.delta 等结构）。

本文件提供了两组辅助方法，把不同格式的 chunk 统一转换为 LangChain 的 ChatGenerationChunk：

_convert_delta_to_message_chunk(...)
把单个 delta 转为对应的 *MessageChunk，支持：

role：user/assistant/system/developer/function/tool

函数调用：function_call

工具调用：tool_calls → tool_call_chunk(...)

reasoning_content：若是 assistant 且包含则挂到 additional_kwargs

用 message_id、tool_call_id 等保持一致性

_convert_chunk_to_generation_chunk(...)

处理 usage（转为 UsageMetadata）

处理 choices[0].delta、finish_reason、model、system_fingerprint、logprobs

跳过 content.delta 这类“纯通知块”

产出 ChatGenerationChunk(message=..., generation_info=...)

# prompt_enhanccer
这个模块用 LangGraph 搭建了一个只有单个节点的工作流：

状态类型定义在 state.py。

真正执行“提示增强”的逻辑在 enhancer_node.py。

图（workflow）的构建与编译在 builder.py。

工作流的输入是待增强的 prompt（可选 context 与 report_style），输出是模型生成并解析后的 output（增强后的提示词）。

builder = StateGraph(PromptEnhancerState)   # 指定状态类型
builder.add_node("enhancer", prompt_enhancer_node)  # 加入唯一节点
builder.set_entry_point("enhancer")         # 入口就是它
builder.set_finish_point("enhancer")        # 结束也在它
return builder.compile()                     # 编译得到可运行图对象


# prompts 语法
{% if report_style == "academic" %}
这是学术风格的内容。
{% elif report_style == "popular_science" %}
这是科普风格的内容。
{% else %}
这是默认内容。
{% endif %}

二、语法说明（Jinja / Liquid 通用）
模板语法	含义
{% if condition %}	如果条件为真，渲染下面的内容
{% elif condition %}	否则如果另一条件为真
{% else %}	否则执行这部分
{% endif %}	结束 if 语句块
{{ variable }}	插入变量的值（例如 {{ CURRENT_TIME }} 会被替换为当前时间）

# Prose 文章撰写
“散文/文案写作工作流”，用 LangGraph 把多个“写作节点”（继续写、润色、变短、变长、纠错、自定义指令等）串成一张有条件分支的图，根据输入里的 option 自动走到对应节点，由节点调用 LLM 完成任务并把结果写回状态。

    builder.add_conditional_edges(
        START,
        optional_node,
        {
            "continue": "prose_continue",
            "improve": "prose_improve",
            "shorter": "prose_shorter",
            "longer": "prose_longer",
            "fix": "prose_fix",
            "zap": "prose_zap",
        },
        END,
    )

表示：图从 START 出发，先执行 optional_node(state)，得到一个字符串。

如果返回值匹配到上面字典的某个 key，比如 "shorter"，就跳到对应的节点（"prose_shorter"）。

最后一个参数 END 是默认分支：当返回值不在字典里（例如拼错、为空）时，直接到 END，图不执行任何节点就结束。

# rag
实现了一个“可插拔”的 RAG 检索层：通过统一的抽象 Retriever 接口，对接不同的后端（RAGFlow 与 VikingDB 知识库）。应用侧只关心“列资源、按资源检索文档”，至于底层调用哪个服务、如何鉴权与签名，都被封装起来了。

# server

技术栈：FastAPI + LangGraph/LangChain（消息流与工具调用）+ Pydantic v2（请求/响应模型）+ SSE 流式输出 +（可选）MongoDB/Postgres 做 LangGraph checkpoint。

主要功能：

聊天/智能体工作流的流式输出（/api/chat/stream）

文字转语音（/api/tts，火山引擎）

文稿生成为播客音频/PPT/散文（/api/podcast/generate、/api/ppt/generate、/api/prose/generate）

Prompt 增强（/api/prompt/enhance）

MCP Server 元数据探测（/api/mcp/server/metadata）

RAG 配置与资源检索（/api/rag/config、/api/rag/resources）

全局配置（/api/config）


# enable_background_investigation 
enable_background_investigation 控制“在正式规划(plan)之前，是否先做一次自动的背景检索”。开了它，就会在 coordinator → background_investigator → planner 的路径上先跑一遍搜索，把检索结果喂给规划模型，提升首轮 Plan 的上下文充分度；
文件：graph/nodes.py → coordinator_node

协调器和用户对话后，会根据工具调用决定是否进入 planner：

goto = "planner"
if state.get("enable_background_investigation"):
    goto = "background_investigator"


也就是说：

开关为 True ⇒ 下一跳是 "background_investigator"

开关为 False ⇒ 下一跳直接是 "planner"

builder 里已经把流程线连好了：
START → coordinator，并在 _build_base_graph() 中注册了 background_investigator 节点，且 background_investigator → planner 的边也建好了。