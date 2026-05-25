# Week 4-6：Nano Agent Harness

> **阶段**：Stage 3 | **产出**：Claude Code-like Nano Agent | **时间**：第 4-6 周

---

## 本周目标

这是整个学习计划最重要的阶段。

从"会用 Agent"到"会设计 Agent 系统"的核心跨越就在这里。
要理解 Harness 的每一层是为什么存在的，而不只是"框架 API 怎么调"。

**完成标准**：一个可调试的 Nano Agent，包含 agent loop、tool registry、permission gate、session store、context compaction，有 README、运行步骤、示例 trace 和失败记录。

---

## 学习任务清单

### Week 4：理解 Harness 设计空间（读论文+读源码）

#### 必读论文（按顺序）

- [ ] **[Dive into Claude Code](https://arxiv.org/abs/2604.14228)**（最重要）
  - 读完后能回答：Claude Code 的 agent loop 是怎么驱动的？
  - 读完后能回答：context compaction 在什么时机触发？触发后做了什么？
  - 读完后能回答：permission gate 有哪几层？每层拦截什么？

- [ ] **[AI Harness Engineering](https://arxiv.org/abs/2605.13357)**
  - 读完后能回答：harness 如何成为 agent 能力的来源？
  - 读完后能回答：tool registry 的设计要考虑哪些维度？

- [ ] **[Configuring Agentic AI Coding Tools](https://arxiv.org/abs/2602.14690)**
  - 对比 Claude Code / Codex / Gemini / Cursor 的配置机制差异

#### 读源码目录结构

- [ ] 精读 [learn-claude-code](https://github.com/shareAI-lab/learn-claude-code)
  - 找出 agent loop 在哪里
  - 找出 tool registry 在哪里
  - 找出 permission gate 在哪里
  - 找出 session store 在哪里
  - 找出 context compaction 在哪里

- [ ] 精读 [claw0](https://github.com/shareAI-lab/claw0)（10 章渐进）
  - 第 1-3 章：agent loop → tool registry → session
  - 第 4-6 章：context compaction → memory → gateway
  - 第 7-10 章：heartbeat → delivery → resilience → concurrency

### Week 5：实现 Nano Agent 核心模块

按以下顺序逐步构建：

#### Module 1：Agent Loop

- [ ] 实现基础循环：接收用户输入 → 调用 LLM → 解析输出 → 执行工具 → 循环
- [ ] 加停止条件：无 tool call / 达到 max_steps / 用户中断
- [ ] 加步骤计数和时间统计

#### Module 2：Tool Registry

- [ ] 设计工具注册表（dict 或 class）
- [ ] 支持：工具名、描述、参数 schema、执行函数、权限级别
- [ ] 实现工具查找和调用
- [ ] 加工具不存在时的错误处理

#### Module 3：Permission Gate

- [ ] 定义权限级别：auto（自动执行）/ confirm（需用户确认）/ deny（禁止）
- [ ] 对文件写入、shell 命令等危险操作要求确认
- [ ] 记录所有权限决策日志

#### Module 4：Session Store

- [ ] 实现 Session 对象：session_id、创建时间、消息历史、工具调用记录
- [ ] 支持 Session 持久化（JSON 文件）
- [ ] 支持 Session 恢复（从文件加载历史）

#### Module 5：Context Compaction

- [ ] 监控 context 长度（token 数）
- [ ] 当接近阈值时触发压缩：把旧消息摘要成一段话，替换历史
- [ ] 保留关键信息（工具调用结果、用户原始意图）

### Week 6：完善与对比实验

- [ ] 跑通最小示例（能完成"列出当前目录文件"这类任务）
- [ ] 加入一个自定义工具（你选择的场景）
- [ ] 观察并记录一次完整 trace（每一步是什么、为什么）
- [ ] **对比实验**：用「裸 loop」和「harness」实现同一任务，记录差异
  - 差异点：错误处理、权限控制、历史管理、可调试性
- [ ] 写失败记录：哪里遇到了问题，怎么解决的

---

## 核心产出

**架构图**：
```
用户输入
    ↓
[Agent Loop]
    ↓
[LLM] ←→ [context compaction（超限时压缩历史）]
    ↓
[Tool Registry（工具注册表）]
    ↓
[Permission Gate（权限检查）]
    ↓
[Tool Execution（执行工具）]
    ↓
[Session Store（持久化状态）]
    ↓
结果返回给用户
```

**文件结构**：
```
week4-6-nano-agent-harness/
├── README.md
├── main.py                    # 入口，启动 agent
├── agent/
│   ├── loop.py                # Agent Loop 核心
│   ├── tool_registry.py       # 工具注册表
│   ├── permission_gate.py     # 权限控制
│   ├── session_store.py       # 会话状态管理
│   └── context_compaction.py  # 上下文压缩
├── tools/
│   ├── shell.py               # Shell 命令工具（需 confirm 权限）
│   ├── file_read.py           # 文件读取工具（auto 权限）
│   ├── file_write.py          # 文件写入工具（需 confirm 权限）
│   └── custom_tool.py         # 你的自定义工具
└── traces/
    └── example_trace.json     # 示例完整 trace
```

---

## 推荐学习资料

### 必读论文（本阶段完成）

| 论文 | 链接 | 优先级 |
|------|------|--------|
| Dive into Claude Code | [arxiv.org/abs/2604.14228](https://arxiv.org/abs/2604.14228) | P0 必读 |
| AI Harness Engineering | [arxiv.org/abs/2605.13357](https://arxiv.org/abs/2605.13357) | P0 必读 |
| Configuring Agentic AI Coding Tools | [arxiv.org/abs/2602.14690](https://arxiv.org/abs/2602.14690) | P1 |
| Your Agent, Their Asset | [arxiv.org/abs/2604.04759](https://arxiv.org/abs/2604.04759) | P1 安全相关 |

### 必读源码

| 项目 | 链接 | 读什么 |
|------|------|--------|
| learn-claude-code | [github.com/shareAI-lab/learn-claude-code](https://github.com/shareAI-lab/learn-claude-code) | Harness 最小实现，逐章跟读 |
| claw0（10章）| [github.com/shareAI-lab/claw0](https://github.com/shareAI-lab/claw0) | Gateway 架构渐进式构建 |
| hello-agents | [github.com/datawhalechina/hello-agents](https://github.com/datawhalechina/hello-agents) | 中文系统教程，补原理 |

### 官方文档

| 资料 | 链接 | 重点 |
|------|------|------|
| Claude Code Overview | [code.claude.com/docs/en/overview](https://code.claude.com/docs/en/overview) | 产品级 coding agent 设计 |
| Claude Code Hooks | [code.claude.com/docs/en/hooks](https://code.claude.com/docs/en/hooks) | 如何拦截和扩展 agent 行为 |
| Claude Code 源码解析 | [claudecoding.dev](https://claudecoding.dev/) | 中文模块级架构解读 |
| Claude Code 源码分析地图 | [code.claudecn.com](https://code.claudecn.com/) | 可视化 agent loop 拆解 |

---

## 面试相关知识点

完成本周后，必须能流畅回答（这是大厂 4 年经验水平的核心考点）：

### Harness 设计题

1. **如何设计一个生产级 Agent Harness？** 回答需覆盖：loop 驱动机制、tool registry 结构、permission gate 层次、session 持久化、context 压缩策略

2. **context compaction 是什么？什么时候触发？** 当 context 接近 model 的 token 上限时，把旧消息摘要压缩；保留用户原始意图和关键工具结果

3. **permission gate 有哪几层？** 工具级别（auto/confirm/deny）→ 操作类型（读/写/执行）→ 资源范围（哪些目录/命令）→ 人工审批（危险操作）

4. **session store 存什么？** session_id、消息历史、工具调用记录、用户偏好、中间状态；支持崩溃恢复

5. **裸 loop 和 harness 有什么区别？** 裸 loop 无权限控制、无状态管理、无错误恢复、无可观测性；harness 把这些系统化

### 对比分析题

6. **Claude Code 和 Codex 的 harness 设计有什么差异？** 参考 arxiv.org/abs/2602.14690 的对比分析

---

## 运行方式

```bash
# 安装依赖
pip install anthropic rich  # rich 用于彩色 trace 输出

# 配置 API Key
export ANTHROPIC_API_KEY=your_key

# 运行
python main.py

# 查看 trace
cat traces/example_trace.json | python -m json.tool
```

---

## 完成记录

- **完成日期**：___________
- **实际用时**：___________
- **遇到的主要难点**：
  - Module 1：
  - Module 2：
  - Module 3：
  - Module 4：
  - Module 5：
- **裸 loop vs harness 对比实验结论**：
- **关键收获**：
