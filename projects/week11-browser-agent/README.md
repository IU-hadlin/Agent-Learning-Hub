# Week 11：Browser Agent

> **阶段**：Stage 6 | **产出**：公开网页信息提取 Agent | **时间**：第 11 周

---

## 本周目标

理解 Browser Agent 和普通 API Tool 的本质区别：API 工具有结构化接口，浏览器操作面对的是非结构化的视觉界面。
掌握 Playwright / browser-use 操作流程，重点是**失败恢复和安全限制**。

**完成标准**：一个只操作公开网页的 Browser Agent，能打开网页、提取信息、生成摘要；遇到失败时能恢复，有截图日志。

---

## 学习任务清单

### 理解 Browser Agent 的独特挑战

- [ ] 理解 Browser Agent vs API Tool 的区别：
  - API Tool：结构化输入输出，确定性强，失败原因明确
  - Browser Agent：操作视觉界面，元素定位不稳定，页面状态不可预期
- [ ] 理解 Browser Agent 的常见失败模式：
  - 元素找不到（页面结构变了）
  - 弹窗遮挡（Cookie 同意、广告）
  - 动态加载（元素还没渲染就操作）
  - 页面跳转（操作后 URL 变化）
  - 反爬机制（IP 封禁、验证码）

### 动手实现

#### Step 1：学习 Playwright 基础

- [ ] 安装 Playwright：`pip install playwright && playwright install chromium`
- [ ] 实现基础操作：打开网页、等待加载、获取文本、截图
- [ ] 实现元素定位：CSS 选择器、XPath、文本内容定位
- [ ] 实现等待机制：`wait_for_selector`、`wait_for_load_state`、`wait_for_timeout`

#### Step 2：用 browser-use 框架

- [ ] 安装 browser-use：`pip install browser-use`
- [ ] 跑通官方示例
- [ ] 理解 browser-use 如何把页面状态描述给 LLM
- [ ] 理解 browser-use 如何解析 LLM 的动作指令

#### Step 3：实现 Browser Agent

- [ ] **基础功能**：
  - `open_url(url)` → 打开页面，等待 DOM 加载完成
  - `get_text(selector)` → 提取指定元素文本
  - `get_all_text()` → 提取页面全部可读文本
  - `take_screenshot(name)` → 截图并保存到 `logs/screenshots/`
  - `click(selector)` → 点击元素（有安全检查）
  - `extract_links()` → 提取页面所有链接

- [ ] **失败恢复**：
  - 元素找不到：截图记录状态，返回 None，不崩溃
  - 弹窗处理：检测常见弹窗模式（Cookie、订阅弹窗），自动关闭
  - 加载超时：设置 30s 超时，超时后截图并报告状态
  - 页面崩溃：捕获 crash 事件，重启 browser 会话

- [ ] **安全限制**（必须实现）：
  - 域名白名单：只允许访问指定的公开域名
  - 禁止登录操作：检测到 login/password 输入框时拒绝操作
  - 禁止表单提交：除非用户明确授权
  - 禁止下载文件：拦截下载事件
  - 每次操作前检查当前 URL 是否在白名单内

- [ ] **日志系统**：
  - 每个操作记录到 `logs/action_log.json`（时间、操作类型、目标、结果）
  - 关键步骤截图（操作前、操作后）
  - 失败时记录完整的页面 HTML（方便复盘）

#### Step 4：实现提取和摘要

- [ ] 从页面提取结构化信息（标题、正文、发布时间、作者）
- [ ] 过滤无关内容（导航、广告、footer）
- [ ] 调用 LLM 对提取内容生成摘要
- [ ] 输出格式：标题 + 来源 URL + 发布时间 + 200 字摘要

---

## 核心产出

**能力演示**：
```
$ python browser_agent.py "https://arxiv.org/abs/2604.14228"

正在打开页面...
截图已保存：logs/screenshots/step1_open.png

正在提取内容...
标题：Dive into Claude Code
作者：[作者列表]
发布：2026-04

正在生成摘要...
摘要：本文从系统设计角度分析 Claude Code 的 agent harness 架构，
      涵盖工具注册、权限控制、上下文压缩和多代理协作机制...

结果已保存：output/arxiv_2604.14228.json
```

**文件结构**：
```
week11-browser-agent/
├── README.md
├── browser_agent.py         # 入口
├── browser/
│   ├── controller.py        # Playwright 封装
│   ├── safety_guard.py      # 安全检查（域名白名单、禁止操作）
│   ├── popup_handler.py     # 弹窗处理
│   └── extractor.py         # 内容提取
├── logs/
│   ├── action_log.json      # 操作日志
│   └── screenshots/         # 截图
└── output/                  # 提取结果
```

---

## 推荐学习资料

### 必读文档

| 资料 | 链接 | 重点 |
|------|------|------|
| Claude Computer Use | [docs.anthropic.com](https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/computer-use-tool) | Computer Use Agent 官方参考 |
| browser-use 文档 | [github.com/browser-use/browser-use](https://github.com/browser-use/browser-use) | 框架使用方式 |
| Playwright Python 文档 | [playwright.dev/python](https://playwright.dev/python/docs/intro) | 底层操作 API |

### 精读源码

| 项目 | 链接 | 读什么 |
|------|------|--------|
| browser-use | [github.com/browser-use/browser-use](https://github.com/browser-use/browser-use) | 页面状态如何描述给 LLM，动作如何解析 |
| UI-TARS-desktop | [github.com/bytedance/UI-TARS-desktop](https://github.com/bytedance/UI-TARS-desktop) | 多模态桌面操作，视觉 UI 理解 |

### Benchmark 论文（了解评测方法）

| 论文 | 链接 | 内容 |
|------|------|------|
| WebArena | [arxiv.org/abs/2307.13854](https://arxiv.org/abs/2307.13854) | 真实网页环境 Agent Benchmark |
| VisualWebArena | [arxiv.org/abs/2401.13649](https://arxiv.org/abs/2401.13649) | 视觉理解 + 网页操作 |

---

## 安全原则（必须遵守）

1. **只操作公开网页**：不访问需要登录的页面，不处理用户账号
2. **域名白名单制**：明确列出允许访问的域名，其他一律拒绝
3. **不提交表单**：禁止向第三方网站发送任何表单数据
4. **不绕过反爬**：遇到验证码或封禁，停止并报告，不尝试绕过
5. **遵守 robots.txt**：检查目标网站的爬取许可
6. **速率限制**：相同域名两次请求间隔至少 2 秒

---

## 面试相关知识点

完成本周后，应该能流畅回答：

1. **Browser Agent 和 API Tool 最大的区别是什么？** 接口稳定性：API 有明确 schema，浏览器操作面对不稳定的视觉界面；错误处理复杂度完全不同
2. **如何处理页面动态加载？** `wait_for_selector` 等待特定元素出现；`wait_for_load_state('networkidle')` 等待网络空闲；不用固定 sleep
3. **如何防止 Browser Agent 做危险操作？** 域名白名单 + 操作类型黑名单 + 人工确认机制；禁止登录、表单提交、文件下载
4. **截图日志有什么作用？** 调试（复现失败场景）+ 审计（操作可追溯）+ 监控（异常检测）

---

## 运行方式

```bash
# 安装依赖
pip install browser-use playwright anthropic
playwright install chromium

# 配置 API Key
export ANTHROPIC_API_KEY=your_key

# 运行（只允许访问公开网页）
python browser_agent.py "https://example.com"

# 查看日志
cat logs/action_log.json | python -m json.tool
```

---

## 完成记录

- **完成日期**：___________
- **实际用时**：___________
- **遇到的最棘手的失败场景**：
- **安全限制实现情况**：
- **关键收获**：
