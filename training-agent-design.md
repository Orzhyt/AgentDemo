# 智能体搭建培训设计方案

> **核心理念：用 AI coding 写 AI。**
> 学员不手写代码，而是通过给 AI coding 工具（WorkBuddy / Cursor / Claude Code 等）写 prompt，
> 让 AI 生成 Agent 代码。培训的核心技能是"如何用 prompt 精确驱动 AI 生成你想要的架构"。
>
> 聚焦 Agent 架构本身，淡化数据库/Web/CRUD 等外围工程。

## 总体结构

```
Block 1（~2 天）              Block 2（~0.5 天）
从 0 搭 Agent（AI coding）     LangGraph 框架对比（AI coding）
├── Phase 1: Baseline         ├── LangGraph 实操
└── Phase 2: 逐层升级          └── DSH 架构赏析
```

---

## AI Coding 使用方法论（培训开头讲，30 分钟）

### 核心心法

1. **Prompt 即设计**：你写给 AI coding 工具的 prompt 就是你的架构设计文档。prompt 越精确，生成的代码越接近你想要的
2. **生成 → 审阅 → 运行 → 理解**：每轮 AI coding 产出后，必须读一遍代码、跑一遍、问 AI 解释不懂的部分
3. **迭代式构建**：不要一次 prompt 生成全部。每个组件单独生成，确认正确后再生成下一个
4. **对比驱动学习**：baseline 和 enhanced 的代码都由 AI 生成，学员通过对比两版代码理解架构演进

### 每轮 AI coding 的工作流

```
1. 粘贴 prompt → AI 生成代码
2. 读代码：能看懂每一行吗？不懂的问 AI 解释
3. 跑代码：python main.py，能跑吗？
4. 跑测试：pytest tests/，测试通过吗？
5. 如果不对：告诉 AI 哪里不对，让它修
6. 如果对了：进入下一轮
```

### Prompt 编写原则

- **说清"是什么"**：要生成什么组件，放在哪个文件
- **说清"为什么"**：这个组件解决什么问题（让 AI 理解意图，不只是翻译需求）
- **说清"约束"**：不用框架 / 不超过 N 行 / 保持某接口不变
- **说清"对比"**：和 baseline 的区别是什么（Phase 2 升级时）

---

## Block 1 — 从 0 搭 Agent

### Phase 1：Baseline Agent（~2 小时）

**目标**：用 AI coding 生成一个最简 Agent，跑起来能对话能调工具。

#### 学员使用的 AI Coding Prompt

```
我要创建一个最简单的 Python AI Agent，要求：

1. 不使用任何框架（不用 langchain、不用 langgraph、不用 crewai），纯 Python + openai SDK
2. 使用 DeepSeek API（兼容 OpenAI 格式），base_url="https://api.deepseek.com/v1"
3. 文件结构：
   - main.py：入口，启动交互式对话循环
   - agent.py：Agent 类，包含 while 循环：接收用户输入 → 调 LLM → 如果 LLM 要调工具就执行工具 → 返回回复
   - llm.py：LLM 封装类，chat(messages, tools) → response，支持 function calling
   - memory.py：Memory 类，用一个 list 存对话历史，有 add() 和获取 messages 的方法
   - tools/__init__.py：工具注册表，一个 dict，key 是工具名，value 是函数
   - tools/calculator.py：一个数学计算工具，用 AST 安全解析数学表达式
   - skills/__init__.py：技能注册表（暂时只放一个默认对话技能）
   - skills/chat.py：默认技能，就是普通对话

4. Agent 的核心循环逻辑：
   - 用户输入 → 加入 memory → 调 LLM（传入 tools schema）
   - 如果 LLM 返回 tool_calls：执行工具 → 结果加入 memory → 再调一次 LLM → 返回回复
   - 如果 LLM 返回普通回复：加入 memory → 打印

5. 总代码量控制在 150 行以内，每个文件都有清晰的注释
6. 在 .env 文件里读 API key，用 python-dotenv

请生成完整代码。
```

#### 审阅检查清单（学员生成代码后对照）

- [ ] `agent.py` 的 while 循环能看出来"循环"在哪
- [ ] `memory.py` 就是一个 list，能看出来"记忆"有多简单
- [ ] `tools/__init__.py` 就是一个 dict，能看出来"工具注册"有多简单
- [ ] 跑起来能对话：`python main.py` → 输入 "你好" → 有回复
- [ ] 能调工具：输入 "帮我算 25 * 17" → Agent 调用 calculate 工具 → 返回 425
- [ ] 能记住上下文：先说"我叫张三"，再问"我叫什么" → 能回答

**这是锚点。后面所有升级都在跟它对比。**

---

### Phase 2：逐层升级（~1.5 天）

**核心教学设计**：每次升级，学员给 AI coding 一个新 prompt，生成增强版代码 + 对比测试。跑测试看 baseline 失败、enhanced 成功。

#### 升级 1：工具系统（~2 小时）

**AI Coding Prompt**：
```
我现在有一个 baseline agent 在 baseline-agent/ 目录。
请基于它升级工具系统，代码放在 enhanced-agent/ 目录（复制 baseline 再修改）。

升级目标：解决 baseline 工具系统的三个痛点：
1. 工具是硬编码在 dict 里的，加新工具要手动改 __init__.py
2. 没有参数校验，传错参数程序直接崩
3. 工具执行出错时 Agent 直接停止，没有错误恢复

具体要求：
1. tools/__init__.py：改成装饰器注册，用 @register_tool 自动注册，自动从函数签名和 docstring 生成 OpenAI tool schema
2. 每个工具函数用 Pydantic 定义参数 schema，调用时自动校验
3. 工具执行失败时不崩溃，把错误信息作为 tool result 返回给 LLM，让 LLM 自己决定怎么处理
4. 新增一个 weather.py 工具（模拟天气查询，返回固定数据即可）来验证装饰器注册

同时生成测试文件 tests/test_tools.py：
- test_baseline_bad_params：直接调 baseline 的 calculate 工具传无效参数，展示它崩溃
- test_enhanced_bad_params：调 enhanced 的 calculate 传无效参数，展示它返回错误信息而非崩溃
- test_auto_registration：验证新工具用装饰器后自动出现在工具注册表中

不要引入框架。用 pydantic 做参数校验是可以的。
```

**对比体验**：
- Baseline：输入 "算一下 abc" → 程序崩溃
- Enhanced：输入 "算一下 abc" → Agent 说"无效的数学表达式"

#### 升级 2：循环控制 / ReAct（~3 小时）

**AI Coding Prompt**：
```
基于 enhanced-agent/ 当前代码，升级 Agent 的循环控制。

当前痛点：Agent 是"一问一答"模式，一次 LLM 调用就返回。
如果用户说"先查北京天气，再查上海天气，对比温度差"，Agent 无法分步执行。

升级目标：实现 ReAct 循环（Reasoning + Acting）：
1. 新建 agent_react.py，继承现有 Agent
2. 核心变化：把 while True 循环改成"任务驱动"模式
   - run_task(task)：接收一个任务，Agent 自主循环执行
   - 每轮：调 LLM → 如果有 tool_calls 就执行 → 结果加入 memory → 继续循环
   - 如果 LLM 返回普通回复（没有 tool_calls），认为任务完成，退出循环
   - 设置 max_steps=10 防止无限循环
3. 在每轮循环中打印 Thought / Action / Observation，让学员能看到 Agent 的推理过程

生成测试 tests/test_react.py：
- test_baseline_multi_step：给 baseline agent 同一个复杂任务，展示它一步搞不定
- test_enhanced_multi_step：给 enhanced agent "先查北京天气，再查上海天气，对比温度差"，
  验证它分 3 步完成（两次工具调用 + 一次汇总）
- test_max_steps：给一个不可能完成的任务，验证到达 max_steps 后安全退出

不要引入框架。
```

**对比体验**：
- Baseline："先查北京天气再查上海天气对比温差" → 一次调用，瞎编
- Enhanced：打印出 `Step 1: 调用 get_weather(北京) → 15°C` → `Step 2: 调用 get_weather(上海) → 20°C` → `Step 3: 上海比北京高 5°C`

#### 升级 3：记忆系统（~2 小时）

**AI Coding Prompt**：
```
基于 enhanced-agent/ 当前代码，升级记忆系统。

当前痛点：memory.py 就是一个无限增长的 list。
问题1：对话 30 轮后，messages 太长，API 调用 token 超限报错
问题2：新对话开始后，完全忘记之前的内容

升级目标：实现三层记忆架构：
1. 短期记忆：滑动窗口，保留最近 20 条消息
2. 摘要记忆：超出窗口的旧消息，用 LLM 生成摘要，作为 system message 注入
3. 长期记忆：用 ChromaDB 存所有历史消息的 embedding，每次对话时检索最相关的 3 条历史

新建 memory_enhanced.py：
- EnhancedMemory 类
- add(msg)：加入短期记忆，超限则触发摘要压缩
- get_context(query)：返回 [摘要 system msg] + [短期消息] + [相关长期记忆]
- _summarize_and_compress()：把旧消息让 LLM 生成摘要，同时存入 ChromaDB

修改 agent_react.py 使用 EnhancedMemory 替代原来的 Memory。

生成测试 tests/test_memory.py：
- test_baseline_long_conversation：和 baseline agent 对话 30 轮，
  第 1 轮说"我叫张三"，第 30 轮问"我叫什么"，验证 baseline 忘了或报错
- test_enhanced_long_conversation：同样场景，验证 enhanced 通过摘要或长期记忆还记得"张三"

用 chromadb 和 openai embedding。不用 langchain。
```

#### 升级 4：知识库 / RAG（~2 小时）

**AI Coding Prompt**：
```
基于 enhanced-agent/ 当前代码，新增知识库（RAG）能力。

当前痛点：Agent 只知道 LLM 训练数据里的内容。
问"我们公司产品 X200 的最大负载是多少"会瞎编。

升级目标：
1. 新建 rag/ 目录：
   - rag/ingest.py：文档导入管道。读取 knowledge_docs/ 下的 .md 文件，
     按段落切分（每块 300-500 字，重叠 50 字），生成 embedding，存入 ChromaDB
   - rag/retriever.py：检索接口。search(query, k=3) → 返回最相关的 3 个文档片段
2. 把 RAG 检索注册为一个工具（search_knowledge），让 Agent 通过 function calling 主动检索
3. 在 system prompt 里告诉 Agent："对于公司业务相关问题，先用 search_knowledge 工具检索知识库"

4. 先在 knowledge_docs/ 目录生成一份示例文档 product_manual.md，内容是虚构的公司产品手册：
   - 产品 X200：最大负载 500kg，续航 8 小时，价格 ¥29,800
   - 产品 X500：最大负载 1200kg，续航 12 小时，价格 ¥58,000
   - 产品 S100：轻量版，最大负载 100kg，续航 24 小时，价格 ¥9,800

生成测试 tests/test_rag.py：
- test_baseline_domain_question：baseline agent 问"X200 最大负载多少"，展示它瞎编
- test_enhanced_domain_question：enhanced agent 问同样问题，验证回答包含"500kg"
- test_rag_no_relevant：问知识库里没有的内容，验证 Agent 说"知识库中没有相关信息"而非瞎编

用 chromadb 做向量存储。embedding 用 sentence-transformers 的 all-MiniLM-L6-v2（本地模型，免费）。
```

#### 升级 5：沙箱 / 代码执行（~1.5 小时）

**AI Coding Prompt**：
```
基于 enhanced-agent/ 当前代码，新增代码执行沙箱。

当前痛点：Agent 不能执行代码。如果用户说"帮我算 1 到 100 的和，用代码验证"，
Agent 只能用 LLM 内部能力算，不能真正写代码、跑代码、看结果。

升级目标：
1. 新建 sandbox.py：
   - Sandbox 类，execute(code, timeout=10) → {stdout, stderr, returncode}
   - 用 subprocess 在临时目录执行 Python 代码
   - 设置超时（默认 10 秒），超时则终止进程
   - 捕获 stdout 和 stderr
2. 把代码执行注册为工具（execute_code），让 Agent 能通过 function calling 执行代码
3. 在 system prompt 里告诉 Agent："需要精确计算或数据处理时，可以写 Python 代码并用 execute_code 工具执行"

生成测试 tests/test_sandbox.py：
- test_baseline_code_execution：baseline agent 无法执行代码
- test_enhanced_code_execution：enhanced agent 收到"计算 1 到 100 的和"时，
  自己写代码 → 调 execute_code → 拿到 5050 → 回答用户
- test_sandbox_timeout：直接调 sandbox.execute("import time; time.sleep(100)", timeout=3)，
  验证 3 秒后返回超时错误
- test_sandbox_stderr：执行有语法错误的代码，验证 stderr 被正确捕获

不要用 Docker，用 subprocess + tempfile 就够了。
```

---

### Phase 2 升级总览

```
Baseline（AI 生成，~150 行）
  │
  ├─ 升级1: 工具系统 → 装饰器注册 + Pydantic 校验 + 错误恢复
  │   AI prompt → 生成 enhanced tools + test_tools.py
  │   对比: 传错参数，baseline 崩溃 vs enhanced 优雅处理
  │
  ├─ 升级2: ReAct 循环 → 多步自主规划
  │   AI prompt → 生成 agent_react.py + test_react.py
  │   对比: 复杂任务，baseline 瞎编 vs enhanced 分步执行
  │
  ├─ 升级3: 记忆系统 → 滑动窗口 + 摘要 + ChromaDB 长期记忆
  │   AI prompt → 生成 memory_enhanced.py + test_memory.py
  │   对比: 30轮对话，baseline 忘了 vs enhanced 记得
  │
  ├─ 升级4: 知识库 → RAG 文档检索
  │   AI prompt → 生成 rag/ + test_rag.py + 示例文档
  │   对比: 领域问题，baseline 瞎编 vs enhanced 基于文档
  │
  └─ 升级5: 沙箱 → subprocess 隔离代码执行
      AI prompt → 生成 sandbox.py + test_sandbox.py
      对比: Agent 写代码，baseline 不能 vs enhanced 能跑
```

---

## Block 2 — LangGraph 框架对比（~0.5 天）

### Part A：LangGraph 实操（3 小时，同样用 AI coding）

#### 步骤 1：用 AI coding 生成 LangGraph 版 Agent（1 小时）

**AI Coding Prompt**：
```
用 LangGraph 实现一个和 enhanced-agent/ 功能相同的 ReAct Agent。

要求：
1. 安装 langgraph + langchain-openai
2. 新建 langgraph-demo/react_agent.py：
   - 用 langgraph.prebuilt.create_react_agent 创建 agent
   - 把 enhanced-agent 里的工具（calculate, get_weather, search_knowledge, execute_code）适配为 langchain Tool 格式
   - 用 ChatOpenAI(model="deepseek-chat", base_url="...") 作为 LLM
3. 实现一个 run() 函数，接收用户输入，调用 agent，打印结果

4. 新建 langgraph-demo/state_graph.py：
   不用 create_react_agent 预构建，而是手动用 StateGraph 搭建：
   - 定义 AgentState（TypedDict，包含 messages 字段）
   - add_node("call_model", call_model_func)
   - add_node("run_tools", run_tools_func)
   - add_conditional_edges("call_model", should_continue, {"tools": "run_tools", END: END})
   - add_edge("run_tools", "call_model")  # 循环边
   - compile()
   这个版本对应 enhanced-agent/agent_react.py 的手写循环

5. 新建 langgraph-demo/comparison.md，写一个对比表：
   手写版 vs LangGraph 版的代码对照（while循环→循环边，if tool_calls→conditional_edges 等）
```

#### 步骤 2：对比讨论（1 小时）

用 AI coding 生成对比分析：
```
对比 enhanced-agent/agent_react.py 和 langgraph-demo/state_graph.py，
生成一个对照表，逐行对应两个版本的核心逻辑：
- while True 循环 → LangGraph 的哪行代码
- if response.tool_calls 判断 → LangGraph 的哪行代码
- memory.messages → LangGraph 的 AgentState
- 工具执行 → LangGraph 的 run_tools node
- max_steps 限制 → LangGraph 怎么实现

然后分析：
1. LangGraph 多提供了什么能力（持久化、流式、人在回路、调试可视化）
2. LangGraph 引入了什么复杂度（概念多、调试需要理解 graph 内部）
3. 什么场景该手写、什么场景该用 LangGraph
```

#### 步骤 3：用 LangGraph 跑同样的测试（1 小时）

```
把 tests/ 下的测试用例适配到 LangGraph 版 agent，
验证 LangGraph 版也能通过同样的测试（test_react, test_memory, test_rag）。
如果有失败，分析是 LangGraph 的限制还是适配问题。
```

### Part B：DeepSeek Harness 架构赏析（1 小时，不做实操）

**用 AI coding 生成讲解材料**：
```
搜索 DeepSeek Harness 的公开信息，生成一份架构讲解文档 langgraph-demo/dsh_analysis.md：

1. 核心理念："Agent = Model + Harness"，"一切皆插件"
2. Cordis 内核：依赖注入 + 服务发现 + 事件系统
3. 插件类型：models, tools, skills, sessions, sandboxes, storage, loops, scheduling, UI
4. 四种运行模式：Standard / Code / Minimal / Creator
5. 和我们手写 Agent 的对应关系：
   - 我们的 tools/ → DSH 的 tool plugin
   - 我们的 memory.py → DSH 的 storage plugin
   - 我们的 agent.py while 循环 → DSH 的 loop plugin
   - 我们的 sandbox.py → DSH 的 sandbox plugin
6. 讨论：如果让你产品化 enhanced-agent，你会怎么设计插件接口？
```

**讨论环节**：学员分组讨论"如果设计自己的 Agent 框架，插件接口怎么定义"，每组用 AI coding 生成一个插件接口草案。

---

## 时间分配总表

| 时间 | 内容 | AI coding 产出 |
|---|---|---|
| **Day 1 上午** | 方法论 + Phase 1: Baseline | baseline-agent/ 全部代码 |
| **Day 1 下午** | 升级 1: 工具系统 + 升级 2: ReAct | enhanced tools/ + agent_react.py + 2 个测试文件 |
| **Day 1 晚上** | 升级 3: 记忆系统 | memory_enhanced.py + test_memory.py |
| **Day 2 上午** | 升级 4: 知识库 RAG | rag/ + 示例文档 + test_rag.py |
| **Day 2 下午** | 升级 5: 沙箱 + 全部测试跑通 | sandbox.py + test_sandbox.py |
| **Day 2 晚上** | 整理对比，准备 Block 2 | 对比演示脚本 |
| **Day 2.5 上午** | Block 2A: LangGraph 实操 | langgraph-demo/ 全部代码 + 对比表 |
| **Day 2.5 下午** | Block 2B: DSH 赏析 + 讨论 | dsh_analysis.md + 插件接口草案 |

## LLM 选型

**推荐 DeepSeek**：
- API 兼容 OpenAI 格式，SDK 直接用 `openai` 库改 base_url
- `deepseek-chat` 支持 function calling
- 成本极低（¥1/1M tokens），培训全流程 < ¥30
- 国内直连，无需代理

```python
from openai import OpenAI
client = OpenAI(api_key="your-key", base_url="https://api.deepseek.com/v1")
```

**Embedding**（RAG 用）：
- 推荐 `sentence-transformers` 的 `all-MiniLM-L6-v2`（本地免费，不依赖外部 API）
- 备选 OpenAI `text-embedding-3-small`

## 项目最终结构

```
training-workshop/
├── baseline-agent/          # Phase 1（AI coding 生成）
│   ├── main.py
│   ├── agent.py             # while 循环
│   ├── llm.py
│   ├── memory.py            # 一个 list
│   ├── tools/
│   │   ├── __init__.py      # 一个 dict
│   │   └── calculator.py
│   └── skills/
│       ├── __init__.py
│       └── chat.py
│
├── enhanced-agent/          # Phase 2（AI coding 逐层生成）
│   ├── main.py
│   ├── agent_react.py       # ReAct 循环
│   ├── llm.py
│   ├── memory_enhanced.py   # 滑动窗口 + 摘要 + 向量
│   ├── tools/               # 装饰器注册 + Pydantic 校验
│   │   ├── __init__.py
│   │   ├── calculator.py
│   │   ├── weather.py
│   │   ├── search_knowledge.py
│   │   └── execute_code.py
│   ├── skills/
│   ├── rag/
│   │   ├── ingest.py
│   │   └── retriever.py
│   └── sandbox.py
│
├── tests/                   # 对比测试（AI coding 生成）
│   ├── test_tools.py
│   ├── test_react.py
│   ├── test_memory.py
│   ├── test_rag.py
│   └── test_sandbox.py
│
├── knowledge_docs/          # RAG 文档（AI coding 生成）
│   └── product_manual.md
│
├── langgraph-demo/          # Block 2（AI coding 生成）
│   ├── react_agent.py       # create_react_agent 一行版
│   ├── state_graph.py       # 手动 StateGraph 版
│   ├── comparison.md        # 代码对照表
│   └── dsh_analysis.md      # DSH 架构赏析
│
├── prompts/                 # 所有 AI coding prompt 存档（教学资产）
│   ├── 01_baseline.md
│   ├── 02_upgrade_tools.md
│   ├── 03_upgrade_react.md
│   ├── 04_upgrade_memory.md
│   ├── 05_upgrade_rag.md
│   ├── 06_upgrade_sandbox.md
│   ├── 07_langgraph.md
│   └── 08_dsh_analysis.md
│
├── .env                     # DEEPSEEK_API_KEY=...
└── README.md
```

## 关键设计决策汇总

1. **全流程 AI coding 驱动**：学员不手写代码，prompt 是课程主体，prompts/ 目录是核心教学资产
2. **对比驱动学习**：每个升级都有 baseline vs enhanced 的行为对比测试，学员亲眼看到差异
3. **不引入框架（Block 1）**：纯 Python + openai SDK + chromadb + pydantic，让学员看到 Agent 的裸骨架
4. **LangGraph 做唯一框架对比（Block 2）**：Python 一致、StateGraph 和手写循环直接对照、稳定可靠
5. **DSH 只赏析不实操**：Node.js 语言不一致 + developer preview，但"一切皆插件"理念值得讨论
6. **淡化数据库/Web**：不搞 CRUD、不搞前端、不搞用户管理，全部聚焦 Agent 内部架构
7. **DeepSeek 做 LLM**：便宜、国内直连、兼容 OpenAI SDK
8. **sentence-transformers 做 embedding**：本地免费，不依赖外部 API，培训环境友好