# AstrBot 系统架构

## 1. 整体架构
```text
QQ 用户 → QQ 服务器 → NapCat（OneBot 客户端）→ WebSocket → AstrBot（核心）
│
├─ 人格系统
├─ 知识库（RAG）
├─ 长期记忆（APLR）
├─ 插件系统（好感度、定时任务）
└─ 外部 LLM API（硅基流动 DeepSeek）
```

## 2. 核心组件

| 组件 | 功能 | 数据位置 |
|------|------|----------|
| NapCat | QQ OneBot 协议客户端，收发 QQ 消息 | `napcat-data/` |
| AstrBot 核心 | 消息路由、上下文管理、插件调度 | `astrbot-data/cmd_config.json` |
| 人格系统 | 管理角色设定（系统提示词） | `astrbot-data/` |
| 知识库（RAG） | 检索增强生成，基于文档回答 | 向量数据库 |
| 长期记忆 | 对话总结 + 语义检索 | APLR: `APLR_DailyReview.db` |
| 插件系统 | 扩展功能（好感度、定时任务、文转图） | 各自插件目录 |

## 3. 数据流向

1. 用户发送 QQ 消息 → NapCat 接收并转换为 OneBot 格式
2. NapCat 通过 WebSocket 推送消息 → AstrBot 接收
3. AstrBot 处理管道：
   - 加载系统提示词（人格）
   - 注入知识库检索结果（RAG）
   - 注入长期记忆（APLR）
   - 调用 LLM API 生成回复
4. LLM 返回文本 → AstrBot 回传 NapCat → NapCat 发回 QQ

## 4. 配置管理

| 配置项 | 位置 | 说明 |
|--------|------|------|
| 人格设定 | Web 后台 → 人格管理 | 系统提示词 + 预设对话 |
| 知识库 | Web 后台 → 知识库 | 上传文档，配置分块/检索参数 |
| 长期记忆 | 插件配置页 | APLR 的总结周期、检索参数 |
| 模型提供商 | 服务提供商 | LLM API Key、Embedding 配置 |

## 5. 插件系统

### 已使用插件
- **astrbot_plugin_local_reminiscence (APLR)**：本地长期记忆，无需外部 API
- **好感度插件**：动态调整角色亲密度
- **定时任务插件**：群晚安/私聊晚安主动消息
- **meme_manager**：表情包管理

### 插件排错
- 若指令无响应，检查 Web 后台 → 配置文件管理 → 工具列表是否包含对应工具
- 若无，尝试卸载插件后重新安装