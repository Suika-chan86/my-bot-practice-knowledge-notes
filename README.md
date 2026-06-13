# 我的 WSL + Docker + 角色工程 知识库

这是一份从零搭建 QQ 机器人（AstrBot + NapCat）并赋予其濑津美人格的完整实践记录。

## 声明

这份知识库中的绝大部分内容，是在 AI 辅助下生成的，我只是进行了整理、润色与核实。

我的技术水平非常有限，这份笔记的初衷是**记录自己踩过的坑**，而不是编写一份严谨的教程。如果其中有错误、疏漏或不够准确的地方，欢迎指正。

**这些内容，光看是学不会的。**
我写下的每一个排查步骤、每一条经验教训，背后都是 Docker 崩溃、插件冲突、网络不通、角色出戏的真实经历。只有亲自去配置、去报错、去修、去重启，才能真正理解它们为什么这样设计。

所以，请不要把这份笔记当作“教程”——它更像是一份**私人踩坑记录**，最多算一份参考。真正的知识，在你自己的终端里。

## 这份笔记适合谁？
- 在 Windows 上使用 WSL2 + Docker Desktop 搭建个人服务的新手。
- 想要深度定制 LLM 聊天机器人（人格、知识库、长期记忆）的爱好者。
- 被容器网络、端口映射、风控封号等问题折磨过的人。

## 目录导航

### 00-元信息
- `learning-roadmap.md`：已掌握和待学习的内容清单
- `resource-list.md`：优质的外部教程、工具、社区

### 01-核心概念
- `wsl2-overview.md`：WSL2 架构、文件互通、网络模式（NAT vs Mirrored）
- `docker-overview.md`：镜像、容器、卷、网络、Docker Compose
- `wsl-docker-interaction.md`：WSL2 与 Docker Desktop 的协作原理

### 02-项目架构
- `astrbot-architecture.md`：AstrBot 核心结构、数据流、配置管理
- `napcat-qq-bridge.md`：NapCat 如何打通 QQ 与 OneBot
- `rag-knowledge-base.md`：知识库（RAG）的嵌入、分块、重排序
- `long-term-memory.md`：长期记忆方案对比（Milvus vs Lite vs APLR）

### 03-角色工程
- `prompt-engineering.md`：系统提示词设计原则、防出戏黑名单
- `personality-consistency.md`：如何保持人格稳定，避免角色漂移
- `affection-system.md`：好感度系统的集成与防崩策略
- `knowledge-injection.md`：知识库、长期记忆、Prompt 的三层协同

### 04-运维与排错
- `common-issues.md`：高频报错速查表（Docker / AstrBot / 记忆插件）
- `network-debugging.md`：容器网络排查的“六层金字塔”
- `docker-log-analysis.md`：如何阅读和利用容器日志
- `backup-strategy.md`：数据备份、恢复与跨电脑迁移

### 05-习惯与心法
- `best-practices-wsl-docker.md`：长期运维的黄金法则
- `debugging-mindset.md`：从表象深挖至根因的思维框架
- `character-engineering-philosophy.md`：角色工程的本质思考

## 使用方式
- 从头到尾阅读可以获得完整知识框架。
- 遇到具体问题时，直接跳转到 `04-operations` 查阅速查表。

## 关联页面
- [MCP Server](https://github.com/Suika-chan86/mcp-notes)