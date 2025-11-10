# 流程
Triage 分诊器（共用 coordinator）：识别问题类型（治疗/诊断/预后/病因/指南/药物/综述），并判定是否进入 GRADE 适用通道（诊断、筛查、预防、治疗、公共卫生问题）。若不适用，走“非 GRADE 通道”（照样严格溯源、标注研究设计与偏倚）。

PICO Elicitor（由 planner 驱动 + human_feedback 回路兜底）：抽取/补全 PICO；PICO 不全则向用户一次性追问缺口（不阻塞则用可推断的默认假设并标注不确定性降低证据确定性）。

Evidence Retriever（沿用 researcher，强化工具）：优先指南/系统评价/临床路径，必要时原始研究。支持 PubMed/指南官网/学会站点检索与抓取。

Guideline Normalizer（由 coder 承担“PROCESSING”步）：将不同来源（ACC/AHA、ESH、ISH、中华医学会等）的推荐分级体系映射到统一展示层（含原体系与映射后的 GRADE 视图并排显示，保持不合并、不混编）。

GRADE Assessor（PROCESSING 步）：基于 Core GRADE/Handbook 规则，对适配问题给出证据确定性与推荐强度；若来源非 GRADE，走“映射 / 不可映射标注”策略（见下）。

Recommendation Synthesizer（沿用 reporter，改模板）：输出“快速结论 + 分栏表 + 推荐分级 + 适用人群/边界 + 争议/不确定性 + 溯源链接”。


# 绘图
```mermaid
flowchart TD
  %% === Entry & Coordination ===
  U["用户问题<br/>自然语言或PICO不完整"] --> C["Coordinator<br/>分诊·语言·背景检索开关"]
  C -->|可选 背景检索| BI["Background Investigator<br/>快速检索概览"]
  C --> P["Planner<br/>问题类型判定与计划生成"]

  %% === Planner: Triage & PICO ===
  subgraph PLAN[Plan 与分诊]
    P --> T{"问题类型<br/>治疗/诊断/预防/筛查/公共卫生/预后/病因/指南/药物/综述"}
    T -->|GRADE 适用| G1["进入 GRADE 通道"]
    T -->|非适用| NG1["进入 非 GRADE 通道"]
    G1 --> PE{"PICO 是否完整"}
    NG1 --> PE
    PE -->|否 一次性补充| HF["Human Feedback<br/>补全关键信息"]
    HF --> PE
    PE -->|是 或采用默认假设| PS["生成执行计划 Steps"]
    PS --> RT["Research Team 执行计划"]
  end

  %% === Research Team Orchestration ===
  subgraph TEAM[Research Team 编排执行]
    RT --> RS{"首个未完成 Step 类型"}
    RS -->|RESEARCH| R["Researcher 证据检索"]
    RS -->|PROCESSING| K["Coder / Evidence Processor<br/>结构化与分级"]
    R --> RT
    K --> RT
    RT -->|全部完成| REP["Reporter 生成循证报告"]
  end

  %% === Researcher: Evidence Priority ===
  subgraph RZ[Researcher 证据优先级]
    R --- GDL["指南与临床路径<br/>ACC-AHA / ESC-ESH / ISH / 本地区指南"]
    R --- SR["系统评价与Meta / HTA"]
    R --- RS1["原始研究<br/>RCT / 队列 / 病例对照"]
    R --- DRUG["药品说明书与标签"]
    R --- TOOLS["工具<br/>web_search / crawl / local_retriever / PubMed / 指南站点检索"]
  end

  %% === Coder / Evidence Processor ===
  subgraph KP[Processing 结构化与分级]
    K --> NORM["Guideline Normalizer<br/>保留原分级体系"]
    NORM --> MAP{"可映射到 GRADE 吗"}
    MAP -->|可以| GRADE1["GRADE 评价<br/>确定性 High/Mod/Low/Very low<br/>推荐 Strong/Weak"]
    MAP -->|不可以| TAG["标注不可映射<br/>保留原体系与证据链解读"]
    GRADE1 --> TAB["对照表生成<br/>来源·年份·原体系·GRADE视图·适用边界·证据要点"]
    TAG --> TAB
  end

  %% === Reporter & Output ===
  REP --> OUT["输出<br/>快速结论·要点清单·比较表·适用边界<br/>争议与不确定性·监测随访·溯源链接"]
  OUT --> END["完成"]

  %% === Policies & Guards ===
  subgraph POL[策略与边界]
    GNR["不整合跨指南结论<br/>默认并排对照"]
    LOC["本地合规优先<br/>监管·医保·资源·价值观"]
    TIME["显示发布日期与检索日期<br/>过期提醒"]
  end

  TAB -.参考策略.-> GNR
  OUT -.合规提示.-> LOC
  R -.元数据.-> TIME

```