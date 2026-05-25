# Week 7-8：Multi-Agent 协调系统

> **阶段**：Stage 4 | **产出**：research → write → review → revise 多 Agent 系统 | **时间**：第 7-8 周

---

## 本周目标

理解多 Agent 系统的本质是**协调问题**，不是"让 Agent 随便聊天"。
掌握如何用 supervisor / graph 管理多个 Agent，定义清晰的职责边界。

**完成标准**：一个 4 角色多 Agent 系统（research → write → review → revise），每个 Agent 有明确的输入输出 schema 和停止条件。

---

## 学习任务清单

### Week 7：理解多 Agent 架构模式

#### 核心概念

- [ ] 理解常见角色分工：
  - **Planner**：分解任务，制定计划
  - **Executor**：执行具体子任务
  - **Reviewer / Critic**：检查结果，提出问题
  - **Router**：根据状态决定下一步走哪个 agent
- [ ] 理解 supervisor 模式：一个 orchestrator agent 管理其他 agent，而不是 agent 之间自由通信
- [ ] 理解 graph 模式（LangGraph）：用状态机定义 agent 之间的转移条件
- [ ] 理解何时单 agent 更好：
  - 任务简单、单步可完成 → 单 agent
  - 角色职责高度重叠 → 单 agent
  - 上下文需要完全共享 → 单 agent

#### 精读官方文档

- [ ] [Claude Code Subagents](https://code.claude.com/docs/en/sub-agents)（任务拆分、上下文隔离、专用子代理）
- [ ] [Agent2Agent Protocol A2A](https://google-a2a.github.io/A2A/specification/)（agent 间如何发现和通信）
- [ ] [Google Agent Development Kit ADK](https://google.github.io/adk-docs/)（多 agent 框架参考）

#### 学习 LangGraph

- [ ] 理解 StateGraph 的核心概念：State、Node、Edge、Conditional Edge
- [ ] 实现一个简单的双节点图：node_a → node_b
- [ ] 理解 checkpointing（可恢复执行）
- [ ] 读 [Open Deep Research 源码](https://github.com/langchain-ai/open_deep_research)（LangGraph 写的多轮搜索 + 状态管理）

### Week 8：实现 4 角色多 Agent 系统

按阶段构建：

#### Phase 1：定义每个 Agent 的 schema

- [ ] **Research Agent**
  - 输入：`{ topic: str, depth: "quick"|"deep" }`
  - 输出：`{ findings: List[str], sources: List[url], gaps: List[str] }`
  - 停止条件：找到 N 个不同来源，或搜索轮次达上限
  - 工具权限：web_search（auto），read_url（auto）

- [ ] **Writer Agent**
  - 输入：`{ findings: List[str], sources: List[url], format: str }`
  - 输出：`{ draft: str, word_count: int, section_count: int }`
  - 停止条件：draft 满足 format 要求
  - 工具权限：无外部工具，纯生成

- [ ] **Reviewer Agent**
  - 输入：`{ draft: str, sources: List[url] }`
  - 输出：`{ score: 1-10, issues: List[str], approved: bool }`
  - 停止条件：打分完毕
  - 工具权限：可查看 sources 内容验证引用

- [ ] **Reviser Agent**
  - 输入：`{ draft: str, issues: List[str] }`
  - 输出：`{ revised_draft: str, changes: List[str] }`
  - 停止条件：所有 issues 都已处理
  - 工具权限：无

#### Phase 2：实现 Supervisor

- [ ] Supervisor 读取当前状态，决定调用哪个 agent
- [ ] 实现状态机：RESEARCH → WRITE → REVIEW → （score>=7 → DONE，score<7 → REVISE → REVIEW）
- [ ] 处理循环：最多 revise 3 次后强制结束
- [ ] 实现上下文传递：每个 agent 只接收它需要的信息（不共享完整历史）

#### Phase 3：边界情况处理

- [ ] 处理 agent 超时：单个 agent 超时后 supervisor 如何决策
- [ ] 处理任务漂移：agent 输出不符合 schema 时的降级处理
- [ ] 处理上下文膨胀：多轮 revise 后 context 如何压缩
- [ ] 加日志：每次 agent 切换都记录原因

---

## 核心产出

**工作流**：
```
用户输入主题
    ↓
[Supervisor]
    ↓
[Research Agent] → findings + sources + gaps
    ↓
[Writer Agent] → draft
    ↓
[Reviewer Agent] → score + issues
    ↓ score < 7
[Reviser Agent] → revised_draft
    ↓ 最多 3 次
[Supervisor] → 输出最终报告
```

**文件结构**：
```
week7-8-multi-agent/
├── README.md
├── main.py                 # 入口
├── supervisor.py           # Supervisor：状态机 + agent 调度
├── agents/
│   ├── research_agent.py   # Research Agent
│   ├── writer_agent.py     # Writer Agent
│   ├── reviewer_agent.py   # Reviewer Agent
│   └── reviser_agent.py    # Reviser Agent
├── schemas/
│   └── types.py            # 所有 agent 的输入输出 schema（dataclass/pydantic）
└── traces/
    └── example_run.json    # 示例完整运行 trace
```

---

## 推荐学习资料

### 必读文档

| 资料 | 链接 | 重点 |
|------|------|------|
| Claude Code Subagents | [code.claude.com/docs/en/sub-agents](https://code.claude.com/docs/en/sub-agents) | 任务拆分、上下文隔离、专用子代理 |
| Claude Code Hooks | [code.claude.com/docs/en/hooks](https://code.claude.com/docs/en/hooks) | 拦截和扩展 agent 行为 |
| LangGraph 文档 | [langchain-ai.github.io/langgraph](https://langchain-ai.github.io/langgraph/) | StateGraph、checkpoint、可控编排 |
| Google ADK | [google.github.io/adk-docs](https://google.github.io/adk-docs/) | Google 多 agent 框架 |
| Agent2Agent Protocol | [google-a2a.github.io/A2A](https://google-a2a.github.io/A2A/specification/) | agent 间发现和通信协议 |
| Agent Client Protocol | [agentclientprotocol.com](https://agentclientprotocol.com/) | agent 与宿主应用的接口 |

### 精读源码

| 项目 | 链接 | 读什么 |
|------|------|--------|
| Open Deep Research | [github.com/langchain-ai/open_deep_research](https://github.com/langchain-ai/open_deep_research) | LangGraph 多轮搜索状态管理 |
| DeerFlow（字节）| [github.com/bytedance/deer-flow](https://github.com/bytedance/deer-flow) | Long-horizon SuperAgent，多 agent 协作 |
| GenAI Agents | [github.com/NirDiamant/GenAI_Agents](https://github.com/NirDiamant/GenAI_Agents) | 对比 ReAct、Plan-and-Execute、Multi-Agent |

### 延伸阅读

| 论文 | 链接 | 内容 |
|------|------|------|
| AutoGen 论文 | [arxiv.org/abs/2308.08155](https://arxiv.org/abs/2308.08155) | 多 agent 对话框架经典设计 |

---

## 面试相关知识点

完成本周后，应该能流畅回答：

1. **多 agent 系统的核心挑战是什么？** 协调：任务分配、状态同步、循环检测、上下文共享边界、失败传播
2. **supervisor 模式和 peer-to-peer 模式有什么区别？** supervisor 有中心协调者，agent 不直接通信；P2P 灵活但难以追踪状态和防止死锁
3. **如何防止多 agent 系统陷入无限循环？** 定义明确的停止条件、最大轮次限制、状态哈希去重、supervisor 强制终止
4. **每个 agent 应该知道多少上下文？** 最小必要原则：只传入该 agent 需要的信息，不共享其他 agent 的内部状态
5. **何时选择单 agent？** 任务可以在一个 context window 内完成、角色职责高度重叠、协调开销大于收益

---

## 运行方式

```bash
# 安装依赖
pip install anthropic langgraph pydantic tavily-python

# 配置 API Key
export ANTHROPIC_API_KEY=your_key
export TAVILY_API_KEY=your_key

# 运行
python main.py "量子计算对密码学的影响"
```

---

## 完成记录

- **完成日期**：___________
- **实际用时**：___________
- **遇到的问题**：
- **多 agent 系统最大的挑战**：
- **关键收获**：
