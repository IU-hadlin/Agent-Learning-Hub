# Week 2-3：Web Research Agent

> **阶段**：Stage 2 | **产出**：Web Research Agent | **时间**：第 2-3 周

---

## 本周目标

在 Week 1 手写 Agent Loop 的基础上，实现工具调用、RAG 检索、记忆管理三大核心能力。
理解 Agent 如何从"单次对话"演进为"有记忆、有工具、有引用"的研究助手。

**完成标准**：输入研究主题，自动多轮搜索 → 筛选 → 总结 → 输出带引用链接的报告。

---

## 学习任务清单

### Week 2：工具调用与 RAG（第 1 周）

#### RAG 完整链路

- [ ] 理解 RAG 四步：chunk → embed → retrieve → answer with citations
- [ ] 实现文本分块（chunk_size=500, overlap=50）
- [ ] 调用 embedding API 生成向量
- [ ] 实现余弦相似度检索（可用 numpy，无需向量数据库）
- [ ] 生成带来源引用的回答

#### 工具扩展

- [ ] 实现 `web_search` 工具（用 Tavily API 或 SerpAPI）
- [ ] 实现 `read_url` 工具（抓取网页正文，过滤广告/导航）
- [ ] 实现 `read_file` 工具（读取本地文件内容）
- [ ] 处理工具失败：超时、404、空结果，不崩溃继续执行

### Week 3：记忆管理与质量提升（第 2 周）

#### 记忆系统

- [ ] 理解三层记忆的区别：
  - **短期**：当前对话的 messages 列表（context window 内）
  - **会话**：跨轮次的用户偏好、已知事实（存 dict/sqlite）
  - **长期**：跨会话的用户画像、历史总结（存文件/数据库）
- [ ] 实现会话记忆：把本次研究的关键结论存下来，下次查询可复用
- [ ] 精读 `mem0` 源码，理解记忆层设计

#### 研究质量提升

- [ ] 实现多轮搜索（第一轮搜索 → 分析缺口 → 第二轮补充搜索）
- [ ] 实现去重：相同来源只引用一次
- [ ] 实现引用验证：引用的内容必须在原文中找得到
- [ ] 防止幻觉引用：模型编造不存在的链接

#### 精读开源项目

- [ ] **GPT Researcher**：读懂多轮搜索流程、引用生成逻辑
  - 关注 `gpt_researcher/master_agent.py` 的 research loop
  - 关注报告生成时如何绑定引用来源
- [ ] **mem0**：读懂记忆的存取机制
  - 关注 `mem0/memory/main.py` 的 add/search/update

---

## 核心产出

**能力**：输入主题 → 输出结构化研究报告

```
$ python research_agent.py "大语言模型 Agent 工程化挑战"

正在搜索...（第 1 轮：3 个查询）
正在阅读页面...（共 8 篇）
正在补充搜索...（第 2 轮：2 个查询）
正在生成报告...

=== 研究报告：大语言模型 Agent 工程化挑战 ===

## 核心挑战

1. 上下文管理：Agent 运行时上下文会持续增长... [1]
2. 工具可靠性：工具调用失败率在生产环境约 5-15%... [2]
...

## 参考来源
[1] https://arxiv.org/abs/2604.14228 - Dive into Claude Code
[2] https://arxiv.org/abs/2605.13357 - AI Harness Engineering
```

**文件结构**：
```
week2-3-web-research-agent/
├── README.md
├── main.py              # 入口
├── tools/
│   ├── search.py        # 搜索工具
│   ├── read_url.py      # 网页抓取工具
│   └── memory.py        # 记忆工具
├── rag/
│   ├── chunker.py       # 文本分块
│   ├── embedder.py      # 向量化
│   └── retriever.py     # 检索
└── report_generator.py  # 报告生成
```

---

## 推荐学习资料

### 必读（本阶段完成）

| 资料 | 链接 | 重点 |
|------|------|------|
| LlamaIndex Agents 文档 | [docs.llamaindex.ai](https://docs.llamaindex.ai/en/stable/use_cases/agents/) | RAG + Agent 结合方式 |
| LangChain Docs | [docs.langchain.com](https://docs.langchain.com/) | 工具链设计参考（看架构，不照搬） |
| Model Context Protocol | [modelcontextprotocol.io](https://modelcontextprotocol.io/) | 工具标准化协议，理解设计思路 |
| Gemini Code Execution | [ai.google.dev](https://ai.google.dev/gemini-api/docs/code-execution) | 代码执行工具参考 |

### 精读源码（必须）

| 项目 | 链接 | 读什么 |
|------|------|--------|
| GPT Researcher | [github.com/assafelovic/gpt-researcher](https://github.com/assafelovic/gpt-researcher) | 多轮搜索 + 报告生成 + 引用绑定 |
| mem0 | [github.com/mem0ai/mem0](https://github.com/mem0ai/mem0) | 记忆层 add/search/update 机制 |
| Open Deep Research | [github.com/langchain-ai/open_deep_research](https://github.com/langchain-ai/open_deep_research) | LangGraph 多轮搜索状态管理 |

### 延伸阅读

| 资料 | 链接 | 说明 |
|------|------|------|
| Generative Agents 论文 | [arxiv.org/abs/2304.03442](https://arxiv.org/abs/2304.03442) | 记忆、反思、规划驱动的 Agent |
| Reflexion 论文 | [arxiv.org/abs/2303.11366](https://arxiv.org/abs/2303.11366) | 语言反馈与自我改进 |

### 工具 API

| 工具 | 链接 | 用途 |
|------|------|------|
| Tavily Search API | [tavily.com](https://tavily.com) | 专为 AI 设计的搜索 API，有免费额度 |
| Jina Reader | [jina.ai/reader](https://jina.ai/reader/) | 把 URL 转成干净 Markdown，免费 |

---

## 面试相关知识点

完成本周后，应该能流畅回答：

1. **RAG 的完整链路是什么？** chunk → embed → 存向量库 → retrieve top-k → 拼入 prompt → 生成带引用答案
2. **三种记忆有什么区别？** 短期=context window；会话=跨轮次的 state；长期=跨会话的持久存储
3. **如何防止 Agent 幻觉引用？** 要求模型只引用 retrieve 出的文本，生成后验证引用内容是否在原文中存在
4. **工具失败了怎么办？** try/except 捕获，把错误信息作为 tool_result 返回给模型，让模型决定重试还是换策略
5. **如何处理 context window 超限？** 对历史消息做摘要压缩，或只保留最近 N 轮

---

## 运行方式

```bash
# 安装依赖
pip install anthropic tavily-python numpy requests beautifulsoup4

# 配置 API Key
export ANTHROPIC_API_KEY=your_key
export TAVILY_API_KEY=your_key

# 运行
python main.py "你的研究主题"
```

---

## 完成记录

- **完成日期**：___________
- **实际用时**：___________
- **遇到的问题**：
- **关键收获**：
