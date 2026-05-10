# Docker 日志阅读与分析技巧

## 1. 日志在哪儿？
容器的标准输出和标准错误都会被 Docker 捕获，可通过 `docker logs 容器名` 查看。

## 2. 基本阅读命令

| 目的 | 命令示例 |
|------|----------|
| 看最近 50 行 | `docker logs astrbot --tail 50` |
| 实时跟踪 | `docker logs -f astrbot` |
| 过滤关键词 | `docker logs astrbot | grep -E "ERRO|timeout"` |
| 按时间范围过滤 | 配合 `grep` 加正则匹配时间戳 |
| 导出日志到文件 | `docker logs astrbot > astrbot.log` |

## 3. 日志级别的含义
- `INFO`：正常流程记录，如“准备发送消息”、“收到消息”。
- `WARN`：异常但可能不影响运行，如“重试连接中”。
- `ERRO`：出错，需要重视，如“连接超时”、“Token无效”。
- `DBUG`：调试信息，通常包含详细的变量值或SQL语句。

## 4. 关键事件定位方法
1. **以时间戳锚定**：先找到发生故障的大致时间点，然后查看前后几十行。
2. **沿调用链追溯**：从 `ERRO` 行向上翻找，查看是哪个上游函数调用导致的。
3. **识别模式**：例如 `trying to resume download` 反复出现，说明网络不稳定。
4. **交叉对比**：同时观察 AstrBot 日志和 NapCat 日志，判断是发送端还是接收端的问题。

## 5. 常见错误日志速查
- `APITimeoutError`：LLM接口超时 → 延长超时，检查网络。
- `Connection refused`：目标服务未监听端口 → 检查容器是否启动、端口是否正确。
- `MilvusException: server unavailable`：向量数据库不可用 → 检查 Milvus 容器日志。
- `ModuleNotFoundError`：Python 依赖缺失 → 重建容器或安装缺失包。
- `token is invalid` / `401`：API Key 错误 → 检查 Key 有效性。

## 6. 日志持久化与轮转
- Docker 默认保留容器所有日志，长期运行可能导致磁盘占用过大。
- 可在 Docker 守护进程配置中限制单个容器日志大小和份数。
- 对于 AstrBot，可将重要日志挂载到宿主机目录，以便长期存档。

## 7. 阅读日志的思维习惯
1. 不要只看最后一行错误，要追溯其上下文。
2. 区分“症状”和“根因”。`timeout` 只是症状，可能根因是 API 地址填错。
3. 开机启动时尤其关注“初始化”部分，往往隐藏着插件加载失败的信息。