# AI Agent 从理论到实操 — 3 天培训课程大纲

> **设计原则**：以《AI Agents in Depth》[1] 为骨架，融合 2026 年最新线上资料，聚焦 Agent 核心架构。
> 淡化模型后训练、Agent 评估、持续进化、多 Agent 协作（标注为课后阅读）。

## 参考资料一览

| 编号 | 来源 | 类型 |
|---|---|---|
| [1] | 《AI Agents in Depth》中文版，10 章 313 页 | 书籍 |
| [2] | Anthropic, "Building Effective Agents" (2024) | 官方指南 |
| [S1] | arahi.ai, "AI Agent Architecture in 2026" | 博客 |
| [S2] | promtable.com, "AI Agent Design Cheatsheet 2026" | 速查表 |
| [S3] | reactify-solutions.com, "Context Engineering for AI Agents 2026" | 博客 |
| [S4] | agentmelt.com, "Context Engineering: The 2026 Stack" | 博客 |
| [S5] | agentixforce.ai, "Three-Memory Architecture" | 博客 |
| [S6] | brainupgrade-in.github.io, "Agent Memory Architectures" | 博客 |
| [S7] | modelcontextprotocol.io, MCP 规范 2026-07-28 | 官方规范 |
| [S8] | increatech.com, "MCP in 2026" | 博客 |
| [S9] | tuningengines.com, "ReAct as Production Infrastructure" | 博客 |
| [S10] | getmind.io, "What Is an AI Agent? (2026)" | 博客 |
| [S11] | securetom.com, "AI Agent Sandboxing 2026" | 安全指南 |
| [S12] | knovo.dev, "Context Windows Explained (2026)" | 博客 |
| [S13] | ayanutomate.com, "Context Engineering Practitioner Guide" | 博客 |
| [S14] | cubed.run, "ReAct Agents: Reasoning + Tool Use" | 博客 |

---

# Day 1：Agent 理论基础

## 1.1 上午：Agent 本质 + 核心循环（9:00 - 12:00）

### 1.1.1 从产品到公式：Agent = LLM + 上下文 + 工具（9:00 - 9:45）

**核心内容**：

- **产品全景**：Cursor、Deep Research、Manus、手机助手 — 不是"一问一答"，而是自主规划执行 [1, p.15]
- **核心公式**：Agent = LLM + 上下文 + 工具 [1, p.15]
  - LLM = 大脑：决策内核（预训练世界知识 + 后训练决策策略）
  - 上下文 = 眼睛：每个决策点的信息表示（观察、记忆、知识、状态）
  - 工具 = 手脚：感知和改变外部世界的接口
- **六层架构**（2026 生产级视角）[S1]：
  - 感知（Perception）→ 推理（Reasoning）→ 规划（Planning）→ 记忆（Memory）→ 工具使用（Tool Use）→ 监督（Oversight）
  - 小 Agent 压缩这些层，复杂 Agent 严格分离
- **Agent vs Chatbot 的本质区别**：Agent 是系统，不是 prompt [S1]

**教学方式**：产品案例拆解 → 抽象出公式 → 六层架构图

**来源**：[1] §1.1；[S1]

---

### 1.1.2 ReAct 循环：Agent 的心跳（9:45 - 10:30）

**核心内容**：

- **ReAct = Reasoning + Acting**，Yao et al. ICLR 2023 [1, p.22] [S14]
- **循环结构**：Observe → Think → Act → Verify → 循环或退出 [S10]
- **2026 年的关键变化** [S9] [S14]：
  - 从文本解析（"Action: search[query]"）→ 原生 Function Calling（结构化 JSON）
  - ReAct + MCP 已成为生产 Agent 的默认架构，约占 80% 部署系统 [S9]
  - 循环本身已是基础设施，复杂性转移到工具质量和治理
- **退出条件** [1, p.32] [S2]：
  - 任务完成 / 无工具调用返回 / 错误次数超限 / 达到最大轮次 / token 预算耗尽
- **50 行代码看 Agent 本质** [S10]：
  ```python
  for turn in range(max_steps):
      reply = call_model(msgs, tools=TOOLS)
      msgs.append(reply)
      calls = extract_tool_calls(reply)
      if not calls: break  # 完成
      for c in calls:
          output = run_tool(c["name"], c["input"])
          msgs.append({"role": "tool", "content": output})
  ```
- **何时不用 ReAct** [S14]：确定性任务用 workflow；无工具任务用 CoT；延迟敏感用单次调用

**来源**：[1] §1.1.5；[S9] [S10] [S14]

---

### 1.1.3 Harness 工程：模型之外的竞争力（10:30 - 12:00）

**核心内容**：

- **工程范式演进** [1, p.28] [S4]：
  - 提示工程 → 上下文工程 → Harness 工程 → Loop 工程
  - "2026 年的新技能不是 prompting，是 context engineering" [S4]
- **Harness 五要素** [1, p.27]：
  | 要素 | 职责 | 2026 实践 |
  |---|---|---|
  | Context | 信息供给 | 分层压缩、KV Cache 友好设计 |
  | Tools | 观察与行动 | MCP 标准化、Schema 校验 |
  | Constrain | 行为边界 | 默认关闭、显式开放、budget caps |
  | Verify | 结果验证 | 结构化校验、circuit breaker |
  | Correct | 错误纠正 | 静默重试、回退、熔断 |
- **构建三原则** [1, p.29] [2] [S1]：
  - 保持简单：从最简方案开始，ReAct > Plan-Execute > Multi-Agent 作为起点 [S1]
  - 保持透明：显示 Thought/Action/Observation 轨迹
  - 设计好工具接口（ACI）：从 Agent 视角设计，防呆（Poka-yoke）
- **编排模式** [1, p.30] [S2]：
  - 工作流（固定拓扑）vs 自主 Agent（动态决策）vs 混合
  - 实践建议：先提示词 → 再工作流 → 最后自主 Agent
- **生产级 Trade-off** [S1] [S2]：
  - 延迟 vs 推理深度：小模型跑循环，大模型只做难决策
  - 有状态 vs 无状态：重复交互需要记忆架构
  - 工具数量：>30 个工具时 mis-routing 上升，应分组或拆子 Agent [S2]

**来源**：[1] §1.2；[2]；[S1] [S2] [S4]

---

## 1.2 下午：核心组件深入（13:30 - 17:30）

### 1.2.1 上下文工程（13:30 - 15:00）

**核心内容**：

#### 上下文窗口的构成与七层来源

- **七层来源** [S3] [S4]：
  1. System prompt（角色、能力、格式）— 稳定，适合 prompt cache
  2. Tool definitions（JSON Schema）— 常达 8-15K tokens
  3. Retrieved knowledge（RAG 结果）— 动态
  4. Conversation history — 单调增长
  5. Scratchpad / working memory — Agent 自己写的笔记
  6. Long-term memory — 跨会话事实
  7. Current user input — 当前问题
- **顺序很重要** [S4]：稳定的放前面（cache 友好），动态的放后面靠近当前问题

#### 上下文的四种失败模式 [S3] [S13]

| 失败模式 | 症状 | 原因 |
|---|---|---|
| Context rot | 输出质量随输入长度下降 | 注意力分散，中间内容被低权重 |
| Context collapse | 多次重写后丢失细节 | 每次重写趋向简短，领域细节被丢 |
| Context confusion | 无关工具/文档干扰决策 | 40 个未用工具定义提高错误调用概率 |
| Context clash | 新旧上下文矛盾 | 用户说的和检索到的文档冲突 |

#### 四策略框架：Write, Select, Compress, Isolate [S3] [S4]

| 策略 | 做什么 | 解决什么失败 |
|---|---|---|
| **Write** | 写到窗口外（scratchpad、文件、长期存储） | 超出窗口的任务 |
| **Select** | 只拉当前步需要的（RAG、工具选择、few-shot 选择） | Context confusion |
| **Compress** | 压缩必须留下的（摘要、截断、分层摘要） | Context rot |
| **Isolate** | 分到多个窗口（子 Agent 隔离、namespace） | Context clash |

- **实践建议** [S4]：新 Agent 从 Select + Compress 开始，任务增长后加 Write + Isolate
- **Compaction 实践** [S3] [S4]：窗口到 50% 时触发摘要；Claude Code 和 Cursor 已内置

#### KV Cache 友好设计 [1, p.50]

- 前缀稳定 → 缓存命中 → 成本降 90% [S3]
- 只增不改（Append-only）作为设计原则 [1, p.36]
- Anthropic prompt cache：write 1.25x，read 0.1x [S3]

#### 提示工程要点 [1, p.59]

- 系统提示词的"人格"（语气）、"格式"（结构化）、"组织方式"（流程驱动 vs 规则堆砌）
- 提示注入：上下文安全核心威胁 [1, p.63] — 工具输出是不可信数据

**来源**：[1] 第 2 章；[S3] [S4] [S12] [S13]

---

### 1.2.2 记忆架构（15:00 - 15:45）

**核心内容**：

#### 三记忆架构（2026 共识）[S5] [S6] [S1]

| 记忆类型 | 内容 | 存储方式 | 访问模式 | 容量 |
|---|---|---|---|---|
| 短期（工作记忆） | 当前对话、活跃工具、最近 5-10 轮 | 上下文窗口内 | 零延迟 | 2K-8K tokens |
| 长期（语义记忆） | 用户偏好、领域事实、稳定知识 | 向量数据库 | 语义检索 | 百万条 |
| 情景记忆 | 过去会话、事件序列、决策记录 | 时间索引 + 向量 | 时间+语义检索 | 千次会话 |

- **认知科学对应** [1, p.86] [S5]：工作记忆 ↔ in-context；语义记忆 ↔ vector DB；情景记忆 ↔ event store
- **第四种：程序记忆** [S1]：学到的模式/技能，存为更新的 prompt 或 few-shot 示例

#### 记忆的实现模式 [S1] [S6]

- **向量记忆**：embed 每条消息/事实，按相似度检索 — 简单但有损
- **结构化记忆**：提取事实到 schema（偏好、项目、日期），存数据库 — 精确但工程量大
- **摘要记忆**：定期压缩对话历史为运行摘要 — 便宜但有损
- **层次记忆**：组合以上 + planner 决定何时检索什么 — 最强但最复杂

#### 记忆的难题 [S1] [S6]

- **记什么 vs 忘什么**：朴素系统什么都记，被噪声淹没
- **记忆过期**：去年的偏好可能已失效，需要更新/失效机制
- **隐私**：长期记忆 = 长期存储用户数据，GDPR/CCPA 合规问题
- **多租户隔离**：一个泄露 = 灾难

#### 2026 生态 [S1]

- OpenAI Memory、Claude Projects（原生）
- Mem0、Letta(原 MemGPT)、Zep（第三方框架）

**来源**：[1] §3.1；[S1] [S5] [S6]

---

### 1.2.3 RAG：从管道到工具（15:45 - 16:15）

**核心内容**：

#### RAG 基础 [1, p.89]

- 文档分块：按语义切分，300-500 字/块，重叠 50 字 [1, p.90]
- 稠密嵌入（Dense）：语义理解 [1, p.90]
- 稀疏嵌入（Sparse/BM25）：精确关键词匹配 [1, p.92]
- **混合检索**：稠密 + 稀疏，2026 生产标配 [1, p.95] [S1]

#### 2026 生产级 RAG [S1]

- 混合检索 + **Cross-Encoder Reranker** 重排序
- 朴素向量检索在生产中失败：用户搜精确零件号/SKU/日期时不行

#### 智能体化 RAG [1, p.102]

- **范式转变**：RAG 不是固定 pipeline，而是一个工具
- Agent 通过 function calling 主动决定何时检索、检索什么
- 上下文感知检索 [1, p.105]：用当前对话上下文改写检索 query

**来源**：[1] 第 3 章 §3.2-3.3；[S1]

---

### 1.2.4 工具系统与 MCP（16:15 - 17:00）

**核心内容**：

#### 工具五分类 [1, p.111]

- 感知工具（web_search, read_file）、执行工具（shell_exec, code_interpreter）、协作工具、事件触发、用户沟通

#### 工具设计三原则 [S1] [S2] [S14]

1. **每个工具只做一件事**：`search_company_news` 不是 `http_get_news_api` [S2]
2. **返回结构化结果**：JSON with success/error，不是自由文本 [S1]
3. **高风险工具需显式确认**：退款、群发、删除不应一步之遥 [S1]

#### 工具描述的艺术 [1, p.114] [S2]

- 命名按意图：`search_company_news` 而非 `http_get` [S2]
- 描述 = 文档：1-2 句，包含**何时不该用** [S2]
- 紧凑 Schema：描述性字段名，required vs optional [S2]
- 可读错误：`"Invalid ticker 'XYZ' — try 4-letter like 'AAPL'"` 而非 `"400 Bad Request"` [S2]
- 小结果：返回摘要而非 10K token JSON [S2]

#### 工具太多怎么办 [1, p.117] [S2]

- **>30 个工具时 mis-routing 上升** [S2] — 分组或拆子 Agent
- 层次化组织 + 按需加载 [1, p.117]
- 主动工具发现 [1, p.118]
- Skills：把工具发现变成"按需查阅" [1, p.122]

#### MCP：工具的 USB-C 接口 [S7] [S8] [S9]

- **MCP（Model Context Protocol）**：Anthropic 2024.11 发布，2026.07 捐赠给 Linux Foundation [S8]
- **三个原语** [S8]：Tools（可调用）、Resources（只读上下文）、Prompts（可复用工作流）
- **2026-07-28 规范** [S7]：无状态核心、Tasks（异步操作）、MCP Apps（交互式 UI）
- **生态** [S8]：10,000+ MCP 服务器，9700 万月 SDK 下载，OpenAI/Google/Microsoft 均已采用
- **对 Agent 开发的影响** [S9]：工具碎片化问题已消解，ReAct + MCP 成为默认架构

**来源**：[1] 第 4 章；[S1] [S2] [S7] [S8] [S9] [S14]

---

### 1.2.5 沙箱与代码执行（17:00 - 17:15）

**核心内容**：

#### 为什么需要沙箱 [S11]

- OWASP Agentic AI Top 10 将"意外代码执行"（ASI05）列为顶级风险 [S11]
- "永远不要在没有严格沙箱的情况下执行 Agent 生成的代码" [S11]

#### 隔离层级 [S11]

| 层级 | 技术 | 隔离强度 | 适用场景 |
|---|---|---|---|
| 容器 | Docker | 弱（共享内核） | 可信代码 |
| 系统调用拦截 | gVisor | 中 | 半可信代码 |
| 微虚拟机 | Firecracker/Kata | 强（硬件虚拟化） | 不可信代码 |

#### 三项 mandatory controls [S11]

1. 网络出口白名单（default-deny）
2. 工作空间写限制（只能写 /workspace）
3. 配置文件保护（hooks、MCP config 不可被 Agent 修改）

#### Coding Agent 的七个核心工具 [1, p.134]

- Code Interpreter（沙箱）、Bash Shell、读/写/编辑文件、Glob、Grep
- 代码作为元能力 [1, p.144]：思考工具、规则约束、自举

**来源**：[1] 第 5 章；[S11]

---

### 1.2.6 全书设计模式 + 进阶方向概述（17:15 - 17:30）

**核心内容**：

#### 五个贯穿全书的设计模式 [1, p.36]

- **提议者—审核者**（Proposer-Reviewer）：产出与评判分离，自审不可靠
- **渐进式披露**（Progressive Disclosure）：先目录后细节，优化上下文预算 + 选择精度
- **只增不改**（Append-only）：可缓存、可重放、可审计
- **边界集 + 保留集**
- **最小 diff + 可回滚**

#### 生产级 Agent 的预算与护栏 [S2]

| 护栏 | 说明 |
|---|---|
| Max-step cap | 循环上限（如 20 步） |
| Token budget | 总 token 预算，超限终止 |
| Wall-clock budget | 总时间限制 |
| No-progress detector | N 步无变化则停止并升级 |
| Refusal whitelist | 明确禁止的动作列表 |

#### 路由模式 [S2]

- 便宜的路由模型（GPT-4o-mini / Claude Haiku）选工具
- 强执行模型（Claude / GPT-4o）跑实际步骤
- 推理模型仅用于规划 + 最难子任务

#### 课后阅读（标注不展开）

- **Agent 评估** [1, 第 7 章]：评估指标、LLM-as-a-Judge、eval harness
- **模型后训练** [1, 第 8 章]：SFT、RL、四阶段全景
- **持续进化** [1, 第 9 章]：从经验中学习、四种进化方法
- **多 Agent 协作** [1, 第 10 章]：协作拓扑、共享/隔离上下文、失败模式

**来源**：[1] §1.3；[S2]

---

# Day 2 ~ Day 3.5：从 0 到 1 搭建 Agent（AI coding 驱动）

## 2.1 AI Coding 方法论（Day 2 上午，30 分钟）

### 2.1.1 "用 AI 写 AI" 的核心心法

- **Prompt 即设计**：写给 AI coding 工具的 prompt 就是架构设计文档
- **生成 → 审阅 → 运行 → 理解**：每轮产出后必须读代码、跑代码、问 AI 解释
- **迭代式构建**：每个组件单独生成，确认后再生成下一个
- **对比驱动学习**：baseline 和 enhanced 都由 AI 生成，对比理解架构演进

### 2.1.2 Prompt 编写原则

- 说清"是什么"：生成什么组件，放哪个文件
- 说清"为什么"：解决什么问题（让 AI 理解意图）
- 说清"约束"：不用框架 / 不超过 N 行 / 保持接口不变
- 说清"对比"：和 baseline 的区别是什么

**来源**：课程设计；[2] Anthropic "保持简单"原则

---

## 2.2 Phase 1：Baseline Agent（Day 2 上午，1.5 小时）

### 2.2.1 目标与结构

- 用 AI coding 生成最简 Agent：while 循环 + list 记忆 + dict 工具 + LLM 对话
- 文件结构：`main.py / agent.py / llm.py / memory.py / tools/ / skills/`
- 总代码量 ≤ 150 行
- LLM 用 DeepSeek（兼容 OpenAI SDK，国内直连）

### 2.2.2 AI Coding Prompt 与审阅

- 给学员 copy-paste 可用的 prompt（详见培训设计文档 training-agent-design.md）
- 生成后审阅检查清单：循环在哪、记忆多简单、工具多简单、能对话、能调工具、能记住上下文

### 2.2.3 锚点定位

- **这是锚点**：后面所有升级都在跟它对比
- Baseline 暴露 5 个痛点：工具硬编码无校验、一问一答无多步、记忆无限增长、无外部知识、不能执行代码

**理论对应**：[1] §1.1.5 ReAct 循环；[S10] 50 行 Agent 本质；[S9] 循环是基础设施

---

## 2.3 Phase 2：逐层升级（Day 2 下午 ~ Day 3.5 上午）

### 2.3.1 升级 1：工具系统（Day 2 下午，1.5 小时）

- **痛点**：工具硬编码、无参数校验、出错即崩
- **AI coding 生成**：装饰器注册 + Pydantic Schema 校验 + 错误回传 LLM + 自动生成 tool schema
- **对比测试**：传错参数 → baseline 崩溃 vs enhanced 优雅处理
- **理论对应**：[1] §4.2 工具设计原则；[S2] 工具设计三原则；[S14] 可读错误

### 2.3.2 升级 2：ReAct 循环（Day 2 下午，2 小时）

- **痛点**：一问一答，无法分步执行复杂任务
- **AI coding 生成**：`agent_react.py`，Thought → Action → Observation 循环，max_steps 限制，打印推理过程
- **对比测试**：复杂多步任务 → baseline 瞎编 vs enhanced 分 3 步完成
- **理论对应**：[1] §1.1.5；[S9] ReAct 生产化；[S14] Function Calling vs 文本解析；[S2] max-step cap

### 2.3.3 升级 3：记忆系统（Day 3 上午，1.5 小时）

- **痛点**：list 无限增长 → token 爆炸；新会话忘记一切
- **AI coding 生成**：滑动窗口 + LLM 摘要 + ChromaDB 长期向量记忆
- **对比测试**：30 轮对话后问第 1 轮内容 → baseline 忘了 vs enhanced 记得
- **理论对应**：[1] §3.1；[S5] 三记忆架构；[S6] 超越上下文窗口；[S3] Compress 策略

### 2.3.4 升级 4：知识库 / RAG（Day 3 上午，1.5 小时）

- **痛点**：LLM 只知道训练数据，问领域知识瞎编
- **AI coding 生成**：文档导入管道 + ChromaDB 向量检索 + search_knowledge 工具 + 示例文档
- **对比测试**：问产品参数 → baseline 瞎编 vs enhanced 基于文档准确回答
- **理论对应**：[1] §3.2-3.3；[S1] 混合检索 + Reranker；[1, p.102] 智能体化 RAG

### 2.3.5 升级 5：沙箱 / 代码执行（Day 3 下午，1 小时）

- **痛点**：Agent 不能写代码、跑代码、看结果
- **AI coding 生成**：subprocess 隔离 + 超时 + 临时目录 + execute_code 工具
- **对比测试**：Agent 自己写代码算 1 到 100 的和 → baseline 不能 vs enhanced 能跑
- **理论对应**：[1] §5.1 Coding Agent；[S11] 沙箱安全；[1, p.144] 代码作为元能力

### 2.3.6 全部测试跑通与对比演示（Day 3 下午，1 小时）

- 跑全部 5 组对比测试：`pytest tests/ -v`
- 整理对比表：每个升级解决了什么痛点、引入了什么新概念、对应 Day 1 哪个理论

---

# Day 3.5：Agent 框架开发体验（0.5 天）

## 3.1 LangGraph 实操（上午，3 小时）

### 3.1.1 用 AI coding 生成 LangGraph 版 Agent（1 小时）

- 生成两个版本：
  - `react_agent.py`：`create_react_agent` 一行构建
  - `state_graph.py`：手动 StateGraph 搭建（对应手写循环）

### 3.1.2 架构对照：手写版 vs LangGraph 版（1 小时）

- **逐行对照表**：
  - 手写 `while True` → LangGraph `add_edge("run_tools", "call_model")`（循环边）
  - 手写 `if response.tool_calls` → LangGraph `add_conditional_edges`
  - 手写 `memory.messages` → LangGraph `AgentState`（类型化状态）
- **LangGraph 额外能力**：持久化、流式、人在回路、调试可视化、并行执行
- **框架选择讨论**：什么时候手写、什么时候用框架

**来源**：[3] LangGraph 官方文档；[1] §1.2.4.4 框架对比

---

## 3.2 DeepSeek Harness 架构赏析（下午，1.5 小时）

### 3.2.1 DSH 核心理念（30 分钟）

- **Agent = Model + Harness** [4]：模型是灵魂，Harness 让 Agent 在真实环境中工作
- **"一切皆插件"** [4]：models, tools, skills, sessions, sandboxes, storage, loops, scheduling, UI
- **Cordis 内核**：依赖注入 + 服务发现 + 事件系统
- **四种运行模式**：Standard / Code / Minimal / Creator

### 3.2.2 DSH 与手写 Agent 的对应 + 讨论（1 小时）

| 手写组件 | DSH 插件 |
|---|---|
| `tools/` | tool plugin |
| `memory.py` | storage plugin |
| `agent.py` while 循环 | loop plugin |
| `sandbox.py` | sandbox plugin |
| `llm.py` | model plugin |

- **讨论**：如果让你产品化 enhanced-agent，你会怎么设计插件接口？
- **分组活动**：用 AI coding 生成一个插件接口草案

**来源**：[4] DeepSeek Harness 官方文档

---

## 3.3 课程总结（30 分钟）

### 3.3.1 知识闭环

- 回到 Day 1 公式：Agent = LLM + 上下文 + 工具
- 每个升级对应公式的哪个部分
- Harness 五要素在 enhanced-agent 中的体现
- 四策略（Write/Select/Compress/Isolate）在升级中的实践

### 3.3.2 进阶方向

- **课后阅读**：Agent 评估 [1, Ch7]、模型后训练 [1, Ch8]、持续进化 [1, Ch9]、多 Agent 协作 [1, Ch10]
- **实操进阶**：接入 MCP 协议、LangGraph 持久化做长任务、研究 DSH Cordis 源码

---

# 附录

## A. 时间分配总表

| Day | 时段 | 内容 | 产出 |
|---|---|---|---|
| **Day 1** | 上午 | Agent 公式 + ReAct + Harness 工程 | 理论框架 |
| | 下午 | 上下文工程 + 记忆 + RAG + 工具/MCP + 沙箱 + 设计模式 | 组件级理解 |
| **Day 2** | 上午 | 方法论 + Phase 1 Baseline | baseline-agent/ |
| | 下午 | 升级 1 工具 + 升级 2 ReAct | enhanced tools + agent_react |
| **Day 3** | 上午 | 升级 3 记忆 + 升级 4 RAG | memory_enhanced + rag/ |
| | 下午 | 升级 5 沙箱 + 全部测试 | sandbox + 对比演示 |
| **Day 3.5** | 上午 | LangGraph 实操 | langgraph-demo/ |
| | 下午 | DSH 赏析 + 总结 | dsh_analysis + 插件草案 |

## B. 理论到实操对应表

| 理论（Day 1） | 实操（Day 2-3） | 框架（Day 3.5） |
|---|---|---|
| Agent = LLM + 上下文 + 工具 [1, p.15] | baseline 全部代码 | LangGraph AgentState |
| ReAct 循环 [1, p.22] [S9] | 升级 2: agent_react.py | StateGraph 循环边 |
| Harness 五要素 [1, p.27] | 5 个升级各对应一个要素 | DSH 插件化 |
| 四策略: Write/Select/Compress/Isolate [S3] | 升级 3 记忆(Compress) + 升级 4 RAG(Select) | LangGraph checkpointing |
| 三记忆架构 [S5] [S6] | 升级 3: 短期+摘要+长期 | LangGraph state |
| 智能体化 RAG [1, p.102] | 升级 4: search_knowledge 工具 | LangGraph tool node |
| 工具设计三原则 [S2] | 升级 1: 装饰器+Schema+错误处理 | MCP 工具规范 |
| 沙箱安全 [S11] | 升级 5: subprocess 隔离 | DSH sandbox plugin |
| Budget caps + guardrails [S2] | 升级 2: max_steps | LangGraph recursion_limit |
| MCP 协议 [S7] [S8] | 课后进阶 | DSH tool plugin |

## C. 技术选型

| 组件 | 选型 | 理由 |
|---|---|---|
| LLM | DeepSeek (deepseek-chat) | ¥1/1M tokens，国内直连，兼容 OpenAI SDK |
| Embedding | sentence-transformers (all-MiniLM-L6-v2) | 本地免费，不依赖外部 API |
| 向量存储 | ChromaDB | 纯 Python，零配置，持久化到本地 |
| 参数校验 | Pydantic | 自动生成 JSON Schema |
| 框架对比 | LangGraph 1.2+ | Python 一致，StateGraph 对照手写循环 |
| 架构赏析 | DeepSeek Harness | "一切皆插件"理念 |

## D. 学员准备清单

- [ ] Python 3.10+ 安装
- [ ] AI coding 工具（WorkBuddy / Cursor / Claude Code）
- [ ] DeepSeek API key（https://platform.deepseek.com）
- [ ] 课前阅读《AI Agents in Depth》第 1 章