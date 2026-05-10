# 常见报错与解决方案速查表

## 1. Docker / WSL 相关

| 报错信息 | 原因 | 解决方案 |
|----------|------|----------|
| `WSL integration unexpectedly stopped` | WSL 和 Docker Desktop 通信中断 | 点击弹窗“Restart WSL integration”；或 `wsl --shutdown` 后重启 Docker |
| `Docker Engine Starting` 卡死 | WSL2 网络模式冲突（Mirrored） | `%USERPROFILE%\.wslconfig` 改为 `networkingMode=nat` |
| `Cannot connect to the Docker daemon` | Docker 服务未运行 | 启动 Docker Desktop；或 `net start com.docker.service`（管理员 PowerShell） |
| `No such container: astrbot` | 容器已被删除 | `docker ps -a` 检查状态；若需要重新 `docker run` |
| `port is already allocated` | 端口被占用 | `netstat -ano` | findstr 端口号` 查看占用进程 |
| `network sandbox not found` | 容器网络状态损坏（常由异常关机导致） | `docker stop 容器名 && docker rm 容器名`，重建时指定网络 |
| `Error response from daemon: network my-bridge not found` | 自定义网络不存在 | `docker network create my-bridge` |
| `Unable to find image` 拉取镜像卡住 | Docker Hub 国内访问慢 | 配置镜像加速器（Docker Desktop → Settings → Docker Engine → registry-mirrors） |
| `Error 5: Access denied` 启动服务时 | 没有管理员权限 | 以管理员身份运行 PowerShell |

## 2. AstrBot / NapCat 相关

| 报错信息 | 原因 | 解决方案 |
|----------|------|----------|
| `APITimeoutError: Request timed out` | LLM API 超时 | 延长超时时间至 120 秒；检查 API Key 是否有效；检查网络连通性 |
| `401 Invalid token` | API Key 无效 | 重新生成 Key 并填入；检查 Key 前后无空格 |
| `403 Forbidden` | API Key 无权限或余额不足 | 检查硅基流动账户余额和 Key 状态 |
| `Provider is disabled, skipping` | 模型提供商未启用 | Web 后台 → 服务提供商 → 勾选启用 |
| WebSocket 连接超时 (`Opening handshake timed out`) | NapCat 与 AstrBot 网络不通 | 检查两者是否在同一自定义网络；确认 URL 和端口正确 |
| WebSocket 连接意外关闭 | Token 不匹配或路径错误 | 清空 Token 测试连通；检查 URL 路径（`/` vs `/ws/` vs `/ws`） |
| WebUI 打不开 `localhost:6185` | 容器未运行或端口映射错误 | `docker ps` | grep astrbot` 检查状态；检查启动命令含 `-p 6185:6185` |
| NapCat WebUI `Network Error` | 容器重建导致 Token 变更 | `docker logs napcat` | grep token` 获取新 Token |
| NapCat 定时掉线 | 风控/心跳/协议问题 | 固定 MAC 地址；调整心跳至 30s；登录后不要在其他设备登录同号 |

## 3. 知识库 / 记忆插件相关

| 报错信息 | 原因 | 解决方案 |
|----------|------|----------|
| `无法连接到 Milvus 服务` | Milvus 容器未运行或网络不通 | 仅使用 APLR 则忽略此报错；或确保 `milvus 数据库地址` 已清空 |
| `MilvusLite` 连接失败 | 路径不安全或未安装依赖 | 改为相对路径 `data/milvus_lite.db`；或改用 APLR 插件 |
| `模型加载失败`（APLR） | 本地 Embedding 模型下载不完整 | 删除 `APLR_ModelCache` 中的 `.locks` 和残缺文件夹，重新下载；或手动下载模型 |
| `The read operation timed out`（下载模型时） | HuggingFace 下载网络不稳 | 在宿主机下载完整模型，挂载进容器 |
| `写入长期记忆失败` | Milvus 服务未就绪 | 检查 `docker logs milvus-standalone` 是否有 `ready`；或切换为 APLR |
| `路径遍历检测` | 插件不允许访问插件目录外的路径 | 改为相对路径（如 `data/milvus_lite.db`），不要用绝对路径到 `/AstrBot/data/memories/` |
| `pkg_resources` 找不到 | Milvus Lite 缺少依赖 | 改用 APLR 插件，避免折腾系统级依赖 |
| `AmbiguousIndexName` 报错 | Mnemosyne 残留调用 | 在插件管理中禁用或卸载 Mnemosyne |