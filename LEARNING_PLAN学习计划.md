# 个人学习计划

**目标**：成为企业级 Agent 工程师，具备大厂（4 年经验级别）面试竞争力
**起点**：有 AI 应用经验（RAG/chatbot），无系统 Agent 工程经验
**投入**：15+ 小时/周
**周期**：约 16 周（4 个月）

---

## 阶段一：补齐基础（第 1-3 周）

### Week 1：最小 Agent Loop（Stage 1）

**原则：不借框架，用原生 API 手写**

- [ ] 用 Claude / OpenAI API 完成普通对话
- [ ] 让模型输出结构化 JSON
- [ ] 定义一个工具函数（calculator）
- [ ] 解析模型的 tool call / function call
- [ ] 执行工具，把结果喂回模型
- [ ] 加最大步数、超时、错误处理

**产出**：50-150 行的 Calculator Agent（项目阶梯 Level 1），放到 `projects/week1-calculator-agent/`

**推荐阅读**：
- [Claude Tool Use](https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/overview)
- [OpenAI Function Calling](https://platform.openai.com/docs/guides/function-calling)

---

### Week 2-3：工具调用、RAG 与记忆（Stage 2）

- [ ] 实现完整 RAG 链路：chunk → embed → retrieve → answer with citations
- [ ] 把搜索、文件、代码执行接成工具
- [ ] 区分短期上下文、会话记忆、长期记忆
- [ ] 处理工具失败、空结果、重复调用
- [ ] 精读 `GPT Researcher` 源码（多轮搜索 + 引用生成）
- [ ] 精读 `mem0` 源码（长期记忆架构）

**产出**：Web Research Agent（项目阶梯 Level 2），放到 `projects/week2-web-research-agent/`

---

## 阶段二：Harness 工程核心（第 4-8 周）⭐

> 这是大厂面试考察的核心，投入最多时间

### Week 4-6：深入 Agent Harness（Stage 3）

**阅读顺序**：
1. [ ] 论文：[Dive into Claude Code](https://arxiv.org/abs/2604.14228)（理解设计空间）
2. [ ] 论文：[AI Harness Engineering](https://arxiv.org/abs/2605.13357)
3. [ ] 跟读：[learn-claude-code](https://github.com/shareAI-lab/learn-claude-code) 从零复刻
4. [ ] 跟读：[claw0](https://github.com/shareAI-lab/claw0) 10 章渐进（gateway 架构）

**必须能独立解释**：
- [ ] agent loop 设计（如何驱动循环、何时停止）
- [ ] tool registry 怎么组织（注册、查找、权限）
- [ ] permission gate 为什么必要（安全边界）
- [ ] context compaction 是什么（为什么上下文会爆，如何压缩）
- [ ] session store 怎么管理状态（跨轮次持久化）
- [ ] 跑通最小示例，加一个自定义工具
- [ ] 观察一次完整 trace，解释每一步原因
- [ ] 用「裸 loop」和「harness」分别实现同一任务，对比差异

**产出**：Claude Code-like Nano Agent（项目阶梯 Level 6），含 README + trace 记录，放到 `projects/week4-nano-agent/`

---

### Week 7-8：多 Agent 协调（Stage 4）

- [ ] 理解 planner / executor / reviewer / critic / router 角色分工
- [ ] 用 LangGraph 实现带状态的多 Agent 图
- [ ] 学习 Claude Code Subagents 模式
- [ ] 定义每个 agent 的输入输出 schema 和停止条件
- [ ] 处理循环、任务漂移、上下文膨胀问题
- [ ] 判断何时单 agent 更好

**推荐阅读**：
- [Claude Code Subagents](https://code.claude.com/docs/en/sub-agents)
- [Agent2Agent Protocol](https://google-a2a.github.io/A2A/specification/)

**产出**：research → write → review → revise 四角色系统（项目阶梯 Level 9），放到 `projects/week7-multi-agent/`

---

## 阶段三：协议与能力打包（第 9-11 周）

### Week 9-10：Skills、MCP、A2A（Stage 5）

- [ ] 掌握三层区别：Tool（调用接口）/ Skill（流程知识包）/ MCP（外部连接）
- [ ] 读 [Claude Code Skills 文档](https://code.claude.com/docs/en/skills)
- [ ] 读 [Configuring Agentic AI Coding Tools](https://arxiv.org/abs/2602.14690)（主流工具配置对比）
- [ ] 写一个最小 SKILL.md（name / description / 触发条件 / 步骤 / 验收标准）
- [ ] 给 skill 加脚本或模板文件
- [ ] 给 skill 写 smoke test

**选一个方向做产出**（Reusable Skill Pack，项目阶梯 Level 8）：
- [ ] `code-review` skill
- [ ] `research-report` skill
- [ ] `pdf-extraction` skill

放到 `projects/week9-skill-pack/`

---

### Week 11：Browser Agent（Stage 6）

- [ ] 理解 browser agent 和普通 API tool 的区别
- [ ] 用 `browser-use` + Playwright 实现网页观察和点击
- [ ] 加安全限制（不登录敏感账号、不越权）
- [ ] 处理页面变化、弹窗、加载失败
- [ ] 记录截图、DOM、动作日志

**产出**：公开网页信息提取 Agent（项目阶梯 Level 5），放到 `projects/week11-browser-agent/`

---

## 阶段四：评测与生产化（第 12-14 周）⭐

> 大厂最常问："你怎么保证 Agent 质量？"

### Week 12-14：Eval、可观测性、安全（Stage 7）

- [ ] 为 agent 准备固定测试集（至少 20 个任务）
- [ ] 记录成功率、失败原因、工具调用次数、成本、延迟
- [ ] 会看 trace，定位失败在 prompt / 工具 / 检索 / 模型 / 状态管理哪层
- [ ] 给危险操作加 human-in-the-loop 确认
- [ ] 掌握 prompt injection / data exfiltration / tool abuse 风险
- [ ] 用回归测试防止改动后能力退化
- [ ] 读 [SWE-bench 论文](https://arxiv.org/abs/2310.06770)（理解生产级评测方法）
- [ ] 配置 LangSmith 做 trace 记录

**产出**：agent eval 表格（20+ 任务 + 期望结果 + 实际结果 + 失败分类），放到 `projects/week12-eval/`

---

## 阶段五：交付真实项目（第 15-16 周）

### Week 15-16：Ship A Real Agent（Stage 8）

选一个方向，做出可展示的面试作品：

**推荐：Production Harness（项目阶梯 Level 11）**
- [ ] 有明确用户、明确任务、明确成功标准
- [ ] 有日志、trace、错误重试、超时、成本上限
- [ ] 有权限边界和人工确认机制
- [ ] 有部署方式（CLI / Web app / GitHub Action）
- [ ] 有完整 README

**产出**：放到 `projects/week15-production-agent/`，别人能 clone 下来跑

---

## 面试准备清单

大厂 Agent 工程师面试（4 年经验级别）必须能流畅回答：

| 题型 | 问题 | 准备状态 |
|------|------|----------|
| 设计题 | 如何从零设计一个可上线的 Agent 系统？ | [ ] |
| 原理题 | context compaction 是什么？为什么需要 session store？ | [ ] |
| 工程题 | 如何防止生产环境的 prompt injection？ | [ ] |
| 对比题 | Skill vs Tool vs MCP 的区别？单/多 agent 如何选择？ | [ ] |
| 项目题 | 讲一个你做过的 Agent 项目，失败在哪，怎么修的 | [ ] |
| 评测题 | 你的 agent 成功率是多少？怎么测量的？ | [ ] |
| 安全题 | 你的 agent 有哪些权限边界？危险操作如何处理？ | [ ] |

---

## 资源优先级

| 优先级 | 资源 | 用途 |
|--------|------|------|
| P0 | [Dive into Claude Code](https://arxiv.org/abs/2604.14228) | 理解 Harness 设计空间 |
| P0 | [learn-claude-code](https://github.com/shareAI-lab/learn-claude-code) | 动手复刻 Harness 机制 |
| P0 | [claw0](https://github.com/shareAI-lab/claw0) | Gateway 架构，10 章渐进 |
| P0 | [AI Harness Engineering](https://arxiv.org/abs/2605.13357) | Harness 作为能力来源的研究 |
| P1 | [hello-agents](https://github.com/datawhalechina/hello-agents) | 中文系统教程，补原理 |
| P1 | LangGraph 官方文档 | 多 Agent 编排 |
| P1 | LangSmith 文档 | 生产级 trace |
| P2 | [SWE-bench](https://arxiv.org/abs/2310.06770) | 理解生产级评测 |
| P2 | [Configuring Agentic AI Coding Tools](https://arxiv.org/abs/2602.14690) | 主流工具对比 |

---

## 学习原则

- **先动手，再读深**：每周必须有可运行代码
- **不借框架手写 Stage 1**：理解底层比会调 API 更重要
- **每个产出写 README**：面试时能直接 demo
- **记录失败**：失败原因和修复过程是面试最好的素材
- **trace 每一次重要运行**：养成可观测习惯

---

## 进度记录

| 阶段 | 完成时间 | 核心产出 | 备注 |
|------|----------|----------|------|
| Week 1 | | Calculator Agent | |
| Week 2-3 | | Web Research Agent | |
| Week 4-6 | | Nano Agent + Harness | |
| Week 7-8 | | Multi-Agent Writer | |
| Week 9-10 | | Skill Pack | |
| Week 11 | | Browser Agent | |
| Week 12-14 | | Eval 表格 | |
| Week 15-16 | | Production Agent | |
