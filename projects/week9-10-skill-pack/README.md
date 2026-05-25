# Week 9-10：Reusable Skill Pack

> **阶段**：Stage 5 | **产出**：可复用 Skill 能力包 | **时间**：第 9-10 周

---

## 本周目标

理解 Skill 的本质：不是工具调用，不是一次性 prompt，而是**可发现、可版本化、可分发的流程知识包**。
掌握 MCP、A2A、ACP 三大协议的边界和用途。

**完成标准**：一个完整的可复用 Skill，包含 SKILL.md、实现脚本、触发条件、验收标准和 smoke test。

---

## 学习任务清单

### Week 9：理解三层边界

#### 概念厘清（必须能独立解释）

- [ ] **Tool vs Skill**
  - Tool：可调用的函数接口（calculator、web_search）
  - Skill：完成一类任务的流程知识（"如何做代码审查"的完整步骤）
  - 区别：tool 是原子操作，skill 是多步流程的编排和知识

- [ ] **Skill vs Prompt**
  - Prompt：一次性指令，随对话消失
  - Skill：有名字、有描述、有版本、可被 agent 主动发现和加载
  - 区别：skill 是可复用的，prompt 是一次性的

- [ ] **Skill vs MCP**
  - MCP（Model Context Protocol）：连接外部工具/数据源（数据库、API、文件系统）
  - Skill：告诉 agent 如何完成一类任务（步骤、模板、验收标准）
  - 区别：MCP 是"能访问什么"，Skill 是"知道怎么做什么"

- [ ] **A2A vs ACP**
  - A2A（Agent2Agent）：不同 agent 之间的发现和通信协议
  - ACP（Agent Client Protocol）：agent 与宿主应用（IDE、终端、浏览器）的接口
  - 区别：A2A 是 agent-agent，ACP 是 agent-host

#### 阅读官方 Skill 规范

- [ ] [Claude Code Skills 文档](https://code.claude.com/docs/en/skills)（文件结构、触发机制）
- [ ] [Claude Agent Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills)（skill 在 agent 中如何工作）
- [ ] [Claude Code Agent SDK Skills](https://code.claude.com/docs/en/agent-sdk/skills)（SDK 层面的 skill）
- [ ] [OpenClaw Skills](https://github.com/openclaw/openclaw/blob/main/docs/tools/skills.md)（加载、作用域、安全边界）

#### 读协议规范

- [ ] [Model Context Protocol](https://modelcontextprotocol.io/)（理解 MCP 的工具/资源/提示三层）
- [ ] [Agent2Agent Protocol](https://google-a2a.github.io/A2A/specification/)（agent card、task、push notification）
- [ ] [Agent Client Protocol](https://agentclientprotocol.com/)（宿主与 agent 的消息格式）

#### 读质量评估论文

- [ ] [SWE-Skills-Bench](https://arxiv.org/abs/2603.15401)（skill 是否真的提升成功率）
- [ ] [Agent Skills analysis](https://arxiv.org/abs/2602.08004)（skill 的有效性分析）

### Week 10：实现你的 Skill Pack

#### 选择一个方向（选一个做精）

**选项 A：Code Review Skill**（推荐，面试最常考）
- 输入：git diff 或文件路径
- 步骤：读代码 → 分析风险 → 检查测试 → 检查安全 → 生成报告
- 输出：结构化 review 报告（问题列表 + 严重程度 + 修改建议）

**选项 B：Research Report Skill**
- 输入：研究主题 + 受众类型
- 步骤：搜索 → 筛选 → 分析 → 结构化 → 引用验证 → 生成报告
- 输出：带引用的 Markdown 报告

**选项 C：PDF Extraction Skill**
- 输入：PDF 路径 + 提取目标（摘要/表格/数据）
- 步骤：解析 → 分块 → 定位目标内容 → 结构化输出
- 输出：JSON 格式的提取结果

#### 实现步骤

- [ ] **Step 1：写 SKILL.md**（最重要的文件）
  - name：唯一标识符
  - description：一句话说明何时使用这个 skill
  - trigger：什么情况下 agent 应该加载这个 skill
  - steps：详细的执行步骤（可以有条件分支）
  - tools：需要哪些工具
  - acceptance_criteria：验收标准，agent 如何判断 skill 完成了

- [ ] **Step 2：写实现脚本**
  - 把 skill 的步骤用代码实现
  - 脚本要独立可运行（不依赖特定 agent 框架）

- [ ] **Step 3：写 smoke test**
  - 用 3-5 个测试用例验证 skill 有效
  - 记录：输入 → 期望输出 → 实际输出 → 是否通过
  - 对比：有 skill vs 无 skill 的成功率差异

- [ ] **Step 4：写触发说明**
  - 什么类型的用户请求应该触发这个 skill
  - 什么情况下不应该触发（避免过度触发）

---

## SKILL.md 模板

```markdown
---
name: code-review
description: 对代码变更进行系统性审查，发现风险、改进建议和安全问题
trigger: 用户要求 review 代码、检查 PR、分析变更时
version: 1.0.0
---

## 何时使用

- 用户说"帮我 review 这段代码"
- 用户提供了 git diff 或代码文件
- 用户说"检查这个 PR 有没有问题"

## 何时不使用

- 用户只是想解释代码功能
- 代码只有几行且显然正确

## 执行步骤

1. **读取代码**：获取完整的变更内容（diff 或文件）
2. **理解上下文**：这段代码做什么，在哪里被调用
3. **风险扫描**：
   - 安全漏洞（注入、越权、敏感信息泄露）
   - 错误处理缺失
   - 边界条件未覆盖
4. **测试覆盖**：是否有对应测试，测试是否充分
5. **代码质量**：可读性、重复代码、命名规范
6. **生成报告**：按严重程度分级（Critical/Major/Minor）

## 工具

- read_file：读取代码文件
- shell：运行 `git diff` 获取变更

## 验收标准

- [ ] 报告包含所有发现的问题
- [ ] 每个问题有严重程度标注
- [ ] 每个问题有具体的修改建议
- [ ] Critical 问题必须标注，不能遗漏安全漏洞
```

---

## 核心产出

**文件结构**：
```
week9-10-skill-pack/
├── README.md
├── skills/
│   └── code-review/              # （或你选的方向）
│       ├── SKILL.md              # Skill 描述文件
│       ├── implementation.py     # 实现脚本
│       ├── smoke_test.py         # Smoke test
│       └── examples/
│           ├── input_example.diff
│           └── output_example.md
└── protocols/
    ├── mcp_notes.md              # MCP 学习笔记
    ├── a2a_notes.md              # A2A 学习笔记
    └── acp_notes.md              # ACP 学习笔记
```

---

## 推荐学习资料

### 必读文档

| 资料 | 链接 | 重点 |
|------|------|------|
| Claude Code Skills | [code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills) | Skill 文件结构和触发机制 |
| Claude Agent Skills | [docs.claude.com](https://docs.claude.com/en/docs/agents-and-tools/agent-skills) | Agent SDK 中的 Skill |
| OpenClaw Skills | [github.com/openclaw](https://github.com/openclaw/openclaw/blob/main/docs/tools/skills.md) | Skill 加载、作用域、安全 |
| Model Context Protocol | [modelcontextprotocol.io](https://modelcontextprotocol.io/) | MCP 工具/资源/提示三层 |
| Agent2Agent Protocol | [google-a2a.github.io/A2A](https://google-a2a.github.io/A2A/specification/) | A2A 规范 |
| Agent Client Protocol | [agentclientprotocol.com](https://agentclientprotocol.com/) | ACP 宿主接口 |

### 质量评估论文

| 论文 | 链接 | 用途 |
|------|------|------|
| SWE-Skills-Bench | [arxiv.org/abs/2603.15401](https://arxiv.org/abs/2603.15401) | 评估 skill 是否真的有效 |
| Agent Skills analysis | [arxiv.org/abs/2602.08004](https://arxiv.org/abs/2602.08004) | Skill 有效性分析方法 |

---

## 面试相关知识点

完成本周后，应该能流畅回答：

1. **Skill 和 Tool 的本质区别是什么？** Tool 是原子接口（函数调用），Skill 是流程知识包（多步骤编排 + 上下文 + 验收标准）
2. **MCP 解决了什么问题？** 标准化 agent 连接外部工具和数据源的接口，避免每个 agent 都要写自定义适配器
3. **如何判断一个 skill 是否有效？** 设计对照实验：同一任务有 skill vs 无 skill 的成功率对比；smoke test 覆盖率
4. **A2A 和 ACP 的使用场景分别是什么？** A2A：多个 agent 需要协作时；ACP：agent 需要嵌入 IDE/终端等宿主环境时
5. **skill 的 trigger 条件如何设计？** 要具体（"用户提供了 git diff"），不要模糊（"用户要看代码"）；避免过度触发比漏触发更重要

---

## 运行方式

```bash
# 运行 smoke test
python skills/code-review/smoke_test.py

# 手动测试
python skills/code-review/implementation.py --input examples/input_example.diff
```

---

## 完成记录

- **完成日期**：___________
- **选择的 Skill 方向**：___________
- **Smoke test 结果**：___/___通过
- **有 Skill vs 无 Skill 成功率**：___ vs ___
- **关键收获**：
