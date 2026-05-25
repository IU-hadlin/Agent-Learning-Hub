# Week 1：最小 Agent Loop

> **阶段**：Stage 1 | **产出**：Calculator Agent | **时间**：第 1 周

---

## 本周目标

用原生 LLM API（不借任何框架）手写一个完整的 Agent Loop。
理解 tool call 底层机制，是后续所有阶段的基础。

**完成标准**：50-150 行代码，能选择工具、执行工具、把结果喂回模型、返回最终答案。

---

## 学习任务清单

### 核心概念（先读后写）

- [ ] 理解 Agent 基本循环：observe → think → act → observe
- [ ] 区分 chatbot、workflow、agent 三种形态
- [ ] 理解 tool call 和普通对话的区别（模型不执行工具，只输出调用意图）
- [ ] 读完 [Anthropic: Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)

### 动手实现

- [ ] **Step 1**：用 API 完成普通多轮对话
- [ ] **Step 2**：让模型输出结构化 JSON（不用 tool call，先手动 parse）
- [ ] **Step 3**：定义 `calculator` 工具（加减乘除）并注册 schema
- [ ] **Step 4**：调用 API，解析返回的 tool_use / function_call
- [ ] **Step 5**：执行工具函数，把结果构造成 tool_result 消息喂回模型
  - [ ] **Step 6**：循环直到模型返回最终答案（无 tool call）
- [ ] **Step 7**：加最大步数限制（max_steps=10）
- [ ] **Step 8**：加超时和错误处理（工具执行失败时继续）

### 扩展挑战（可选）

- [ ] 再加一个 `search` 工具（模拟搜索，返回固定结果）
- [ ] 让 agent 可以同时调用多个工具（parallel tool calls）
- [ ] 把对话历史打印成可读的 trace 格式

---

## 核心产出

**文件**：`main.py`（或 `index.ts`）

最终能跑通这样的对话：
```
用户：(23 + 47) × 12 是多少？
Agent：调用 calculator(23 + 47) → 70
       调用 calculator(70 × 12) → 840
Agent：结果是 840。
```

**要求**：
- 代码不超过 150 行
- 有清晰注释说明每一步
- 有 README（本文件）说明怎么运行

---

## 推荐学习资料

### 必读（本周完成）

| 资料 | 链接 | 重点 |
|------|------|------|
| Claude Tool Use 官方文档 | [docs.anthropic.com](https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/overview) | tool_use 消息格式、tool_result 格式 |
| OpenAI Function Calling | [platform.openai.com](https://platform.openai.com/docs/guides/function-calling) | function_call 格式对比 |
| Anthropic: Building effective agents | [anthropic.com](https://www.anthropic.com/engineering/building-effective-agents) | Agent 设计原则 |

### 参考代码

| 资料 | 链接 | 用途 |
|------|------|------|
| Claude Tool Use 示例 | [github.com/anthropics/anthropic-cookbook](https://github.com/anthropics/anthropic-cookbook) | 参考实现，不要直接复制 |
| OpenAI Agents SDK | [platform.openai.com](https://platform.openai.com/docs/guides/agents-sdk/) | 了解框架封装了什么 |

### 延伸阅读

| 资料 | 链接 | 说明 |
|------|------|------|
| ReAct 论文 | [arxiv.org/abs/2210.03629](https://arxiv.org/abs/2210.03629) | Reasoning + Acting 基础范式 |
| Toolformer 论文 | [arxiv.org/abs/2302.04761](https://arxiv.org/abs/2302.04761) | 模型如何学习调用工具 |

---

## 面试相关知识点

完成本周后，应该能流畅回答：

1. **tool call 的完整流程是什么？** 用户输入 → 模型返回 tool_use → 代码执行工具 → tool_result 送回模型 → 模型返回最终答案
2. **为什么要加 max_steps 限制？** 防止模型陷入无限循环，控制成本和延迟
3. **模型"调用工具"的本质是什么？** 模型只是输出结构化 JSON 表达调用意图，实际执行由宿主代码完成
4. **parallel tool calls 是什么？** 模型在一次响应里输出多个 tool_use，宿主并行执行后统一返回结果

---

## 运行方式

```bash
# 安装依赖
pip install anthropic  # 或 openai

# 配置 API Key
export ANTHROPIC_API_KEY=your_key_here

# 运行
python main.py
```

---

## 完成记录

- **完成日期**：___________
- **实际用时**：___________
- **遇到的问题**：
- **关键收获**：
