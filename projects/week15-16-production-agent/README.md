# Week 15-16：Production Agent

> **阶段**：Stage 8 | **产出**：别人能 clone 下来跑的生产级 Agent | **时间**：第 15-16 周

---

## 本周目标

把前 14 周学到的所有技术整合成一个**真实可交付的 Agent 项目**。
这个项目是面试的核心作品集，必须达到"别人能 clone 下来跑"的标准。

**完成标准**：有明确用户和任务 + 有日志/trace/重试/超时/成本控制 + 有权限边界 + 有部署方式 + 有完整 README。

---

## 本周目标（选一个方向）

### 方向 A：Production Harness（推荐，技术深度最高）

一个完整的 Agent 运行时框架，包含：
- 完整的工具注册/权限/session 管理
- Eval 系统 + 回归测试
- Trace 可视化
- CI/CD 集成（GitHub Actions）

适合目标：展示系统设计能力，适合 infrastructure/platform 方向的岗位

### 方向 B：OpenClaw-like Gateway

一个 Agent 消息网关，包含：
- channel（不同消息来源：CLI/HTTP/Slack）
- routing（根据任务类型路由到不同 agent）
- session（会话状态管理）
- memory（跨会话记忆）
- heartbeat（agent 心跳检测）
- delivery（结果投递保证）

适合目标：展示分布式系统和 agent 基础设施设计能力

### 方向 C：Coding Agent（贴近岗位热点）

一个 coding 场景的专用 Agent，包含：
- 读取代码仓库
- 理解 PR diff
- 执行代码审查（代码质量/安全/性能）
- 生成修改建议
- 与 GitHub 集成（评论 PR）

适合目标：展示 coding agent 工程经验，适合 AI 工程化/DevTools 方向

---

## 学习任务清单

### Week 15：设计与实现

#### 系统设计（先设计再动手）

- [ ] 写一页系统设计文档：
  - 目标用户是谁？要解决什么问题？
  - 核心功能有哪些？（不超过 5 个）
  - 技术架构图（用文字/ASCII 描述即可）
  - 成功标准是什么？（可量化的）

- [ ] 定义成本控制策略：
  - 每次任务最多消耗多少 token？
  - 每次运行最多调用多少次 LLM？
  - 超出限制时的降级策略

#### 核心实现

- [ ] **日志系统**：
  - 每次运行生成唯一 run_id
  - 记录：时间戳、输入、每步操作、输出、错误
  - 格式：JSON（便于分析），同时输出人类可读格式

- [ ] **Trace 系统**（复用 Week 12-14 的）：
  - token 消耗统计
  - 工具调用链路
  - 延迟分布

- [ ] **错误重试机制**：
  - LLM 调用失败：指数退避重试，最多 3 次
  - 工具调用失败：返回错误信息给模型，让模型决策
  - 整体任务失败：保存状态，支持从断点继续

- [ ] **超时控制**：
  - 单次 LLM 调用：30s 超时
  - 单个工具调用：10s 超时
  - 整体任务：5 分钟超时

- [ ] **成本上限**：
  - 运行前估算 token 消耗
  - 超出上限时暂停并提示用户

- [ ] **权限边界**：
  - 危险操作（写文件/执行 shell/发请求）需要确认
  - 操作范围白名单（只能操作指定目录/域名）
  - 所有权限决策记录日志

### Week 16：完善与交付

#### 部署

- [ ] **CLI 模式**：`python agent.py "your task"` 直接运行
- [ ] **Web API 模式**（至少一种）：
  - FastAPI 接口：`POST /run { "task": "..." }`
  - 或 Streamlit UI：简单的网页界面
- [ ] 写 Dockerfile（可选，加分项）

#### CI/CD（推荐实现）

- [ ] 添加 GitHub Actions：
  - PR 时自动运行测试
  - 自动运行 eval 回归测试
  - 测试失败时阻止合并

- [ ] `.github/workflows/eval.yml` 示例：
  ```yaml
  name: Eval Regression
  on: [pull_request]
  jobs:
    eval:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v3
        - run: pip install -r requirements.txt
        - run: python run_eval.py --dataset eval_dataset.json
        - run: python regression_checker.py --threshold 0.05
  ```

#### README（最重要）

- [ ] 写完整 README，必须包含：
  - **项目简介**：解决什么问题，有什么特点
  - **快速开始**：3 步以内能跑起来
  - **配置说明**：需要哪些 API Key，怎么配置
  - **功能演示**：示例输入输出（截图或 GIF）
  - **架构说明**：核心模块是什么，如何扩展
  - **已知限制**：什么场景不适合，有什么已知问题
  - **如何扩展工具**：新手能看懂怎么加自己的工具

#### 最终验收

- [ ] 找一个没看过这个项目的人（朋友/同学），让他/她按 README 跑一遍
- [ ] 记录他/她遇到的所有问题，修复后再验证
- [ ] 把项目推到 GitHub 公开仓库
- [ ] 记录在 LEARNING_PLAN.md 的进度表里

---

## 核心产出

**README 结构**：
```markdown
# Project Name

一句话描述项目做什么。

## 快速开始
pip install -r requirements.txt
export ANTHROPIC_API_KEY=xxx
python agent.py "帮我分析这个代码仓库的架构"

## 功能特性
- ✅ 完整工具注册与权限控制
- ✅ 自动 trace 记录与分析
- ✅ 错误重试与超时保护
- ✅ 成本上限控制
- ✅ 回归测试套件

## 架构
[架构图]

## 配置
| 环境变量 | 说明 | 必填 |
...

## 如何添加新工具
1. 在 tools/ 下创建新文件
2. 实现 execute(input) → output 函数
3. 在 tool_registry.py 注册
4. 指定权限级别

## 已知限制
- 不支持多用户并发
- 长任务（>5 分钟）可能超时
```

**文件结构**：
```
week15-16-production-agent/
├── README.md                 # ← 最重要的文件
├── agent.py                  # 入口
├── requirements.txt
├── Dockerfile                # 可选
├── .github/
│   └── workflows/
│       └── eval.yml          # CI/CD
├── core/
│   ├── loop.py               # Agent Loop
│   ├── tool_registry.py      # 工具注册
│   ├── permission_gate.py    # 权限控制
│   ├── session_store.py      # 状态管理
│   └── context_compaction.py # 上下文压缩
├── tools/                    # 具体工具实现
├── tracer.py                 # Trace 系统
├── logger.py                 # 日志系统
├── eval_dataset.json         # 评测数据集
├── run_eval.py               # 评测入口
└── regression_checker.py     # 回归测试
```

---

## 推荐学习资料

### 系统设计参考

| 资料 | 链接 | 参考什么 |
|------|------|----------|
| Claude Code Overview | [code.claude.com/docs/en/overview](https://code.claude.com/docs/en/overview) | 产品级 coding agent 的完整设计 |
| Claude Code GitHub Actions | [docs.claude.com](https://docs.claude.com/en/docs/claude-code/github-actions) | Coding agent 进入 PR 工作流 |
| OpenClaw | [github.com/openclaw/openclaw](https://github.com/openclaw/openclaw) | 本地优先 agent 架构参考 |
| DeerFlow | [github.com/bytedance/deer-flow](https://github.com/bytedance/deer-flow) | 生产级 long-horizon agent 参考 |

### 工程化参考

| 资料 | 链接 | 参考什么 |
|------|------|----------|
| agents-towards-production | [github.com/NirDiamant/agents-towards-production](https://github.com/NirDiamant/agents-towards-production) | 生产化工程组件 |
| Pydantic AI | [pydantic.dev](https://pydantic.dev/docs/ai/core-concepts/agent/) | 类型安全的生产 Agent |

---

## 面试相关知识点

这个阶段的产出就是面试素材，要能讲清楚：

1. **项目背景**：为什么要做这个 Agent？解决什么真实问题？
2. **架构决策**：为什么选择这个架构？考虑过哪些替代方案？
3. **最大挑战**：遇到的最难的问题是什么？怎么解决的？
4. **失败案例**：哪些尝试失败了？为什么失败？学到了什么？
5. **如何扩展**：如果要支持 10 倍用户量，你会怎么改？
6. **权限设计**：如何防止 Agent 做出不应该做的事？
7. **评测体系**：成功率多少？怎么知道的？怎么保证不退化？

---

## 运行方式

```bash
# 1. 克隆项目
git clone https://github.com/你的用户名/项目名
cd 项目名

# 2. 安装依赖
pip install -r requirements.txt

# 3. 配置 Key
cp .env.example .env
# 编辑 .env 填入 ANTHROPIC_API_KEY

# 4. 运行
python agent.py "你的任务"
```

---

## 完成记录

- **完成日期**：___________
- **GitHub 仓库链接**：___________
- **选择的方向**：___________
- **Eval 成功率**：___________
- **让其他人试用的反馈**：
- **面试时要重点讲的亮点**：
- **关键收获**：
