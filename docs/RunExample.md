# 运行PPT的Agent测试
```
.venv/bin/python -m src.ppt.graph.builder
很简单，就是prompt+模型，输出保存到state中

代码做了什么（执行链路）

状态与图

PPTState(MessagesState) 定义了工作流的可变状态：输入 input、产出 generated_file_path，以及中间产物 ppt_content、ppt_file_path。

build_graph() 用 LangGraph 的 StateGraph 串了三段：START → ppt_composer → ppt_generator → END，compile() 后得到可调用的 workflow。

ppt_composer_node

取出配置里标记为 "ppt_composer" 的模型：get_llm_by_type(AGENT_LLM_MAP["ppt_composer"])。

用两段消息喂给模型：

SystemMessage 来自模版 get_prompt_template("ppt/ppt_composer")（就是 markdown-PPT 的写作指令）。

HumanMessage 是你传进来的 state["input"]（比如一篇报告正文）。

拿到模型输出（一个 ChatMessage），把 content 落到临时文件 ppt_content_*.md，并把：

ppt_content（模型原始返回）

ppt_file_path（临时 md 路径）
回填到状态里（LangGraph 会把返回的 dict merge 进 PPTState）。

ppt_generator_node

读取上一步产出的 ppt_file_path，通过 Marp CLI 执行：

marp <md路径> -o <uuid>.pptx


删除临时 md，返回 generated_file_path（最终的 PPTX）。

脚本直接运行

if __name__ == "__main__" 分支会：

load_dotenv() 读取 .env（模型鉴权等）。

读入 examples/nanjing_tangbao.md 作为输入。

workflow.invoke({"input": report_content}) 同步跑完整个图，得到最终 final_state（里头包含 generated_file_path）。

```