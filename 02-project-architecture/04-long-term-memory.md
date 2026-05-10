# 长期记忆插件对比

## 1. 三种记忆方案

| 方案 | 原理 | 优点 | 缺点 |
|------|------|------|------|
| **Milvus Standalone** | 独立向量数据库，通过 Docker Compose 部署 etcd+minio+standalone 三个容器 | 功能完整 | 资源占用高，WSL2 兼容性差，常连不上 |
| **Milvus Lite** | Python 库，内嵌在 AstrBot 容器中，存为本地 .db 文件 | 无需额外容器 | 需额外安装 `milvus-lite` 和 `setuptools`，版本兼容性要求高 |
| **APLR (Local Reminiscence)** | 本地 Embedding 模型 + SQLite + ChromaDB，全本地运行 | 零额外 API 成本，隐私安全，安装即用 | 需下载约 500MB 的本地模型 |

## 2. 实践踩坑记录

### 2.1 Milvus Standalone
- 拉取 `milvusdb/milvus:latest` 直接 `docker run` 无法启动（缺少依赖命令）。
- 通过 `docker-compose` 部署 etcd + minio + standalone 后，`rootcoord` 反复报 `retry func failed`。
- 根本原因：WSL2 内存不足或版本兼容性差。
- 最终放弃。

### 2.2 Milvus Lite
- 在 AstrBot 容器内安装依赖：`pip install pymilvus[milvus_lite]`。
- 仍然报错：`No module named 'pkg_resources'`。
- 需额外安装 `setuptools`。
- 即使安装成功，绝对路径触发了插件安全限制（路径遍历检测）。
- 改为相对路径 `data/milvus_lite.db` 后，因镜像版本问题依然不稳定。

### 2.3 APLR (Local Reminiscence)
- 从插件市场直接安装。
- 需下载 `paraphrase-multilingual-MiniLM-L12-v2` 模型（约 500MB），网络不稳定会导致下载中断。
- 解决方案：在宿主机下载完整模型，挂载到容器的 `/AstrBot/data/local_models/`，插件配置中指定绝对路径。
- 验证：`Loading weights: 100%` → 写入测试记忆 → 检索成功。

## 3. 经验总结

- **不要为了一个功能去维护额外的数据库服务**，除非功能真的需要。
- 对个人项目而言，**本地文件存储（SQLite/ChromaDB）** 比独立数据库稳定几个数量级。
- 网络不稳定时，**尽量在宿主机下载大文件**，再挂载给容器使用。