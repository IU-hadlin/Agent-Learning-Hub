# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 仓库定位

这是一个 **静态内容仓库**，没有构建工具、包管理器或测试框架。核心产物是两个文件：

- `README.md`：内容的权威来源（Markdown 表格、学习清单、资源列表）
- `index.html`：部署在 GitHub Pages 的交互版本（进度追踪、搜索、暗色模式、Markdown 笔记）

运行方式：直接在浏览器打开 `index.html`，无需任何构建步骤。

## 关键架构：内容双写规则

`README.md` 和 `index.html` 的内容**必须保持同步**。这是本仓库最重要的约束。

`index.html` 里有三个 JS 数据结构完整镜像了 README 的内容：

| JS 对象 | 对应 README 内容 |
|---------|----------------|
| `learningData.stages[]` | Stage 0–8 学习清单 + 推荐阅读 + 开源项目 |
| `ladderData[]` | Project Ladder 表格 |
| `resourcesData{}` | Curated Resources 下所有分类表格 |

**修改任意一处内容时，必须同步修改另一处。**

## projects/ 目录结构

`projects/` 下每个子目录（如 `week1-calculator-agent/`）只包含一个 `README.md`，是给学习者的项目脚手架，**不包含代码**。实际项目代码由学习者自己填充。

## index.html 代码组织

`index.html` 是单文件应用，结构分为三部分：

1. **CSS**（`<style>` 块）：CSS 变量驱动的主题系统，支持亮色/暗色切换
2. **数据层**（`const learningData`, `ladderData`, `resourcesData`）：所有展示内容的 JS 数据
3. **逻辑层**：`localStorage` 持久化进度和笔记，`marked.js`（CDN）渲染 Markdown 笔记，`renderAll()` 驱动全量重渲染

状态全部存储在 `localStorage`，key 前缀为 `agent-learning-hub-*`。
