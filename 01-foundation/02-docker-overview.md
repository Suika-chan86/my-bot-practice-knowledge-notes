# Docker 核心概念

## 1. Docker 的定位

Docker 是一个容器化平台，用于打包、分发和运行应用程序。

## 2. 核心概念

| 概念 | 本质 | 类比 |
|------|------|------|
| **镜像（Image）** | 只读的模板，包含运行环境和代码 | 安装光盘 |
| **容器（Container）** | 镜像的运行实例，添加可读写层 | 运行中的程序 |
| **数据卷（Volume）** | 持久化存储，独立于容器生命周期 | 外接硬盘 |
| **网络（Network）** | 容器间通信的虚拟网络 | 网线+交换机 |

## 3. 容器生命周期
```
拉取镜像 (pull) → 创建容器 (create) → 启动 (start) → 运行中 (running)  
↓  
暂停 (pause)  
↓  
恢复 (unpause)  
↓  
停止 (stop) -> 删除 (rm)
```

## 4. Docker 网络模式

| 模式 | 特点 | 使用场景 |
|------|------|----------|
| **bridge（默认）** | 隔离网络，通过端口映射与宿主机通信 | 单机单容器 |
| **host** | 与宿主机共享网络栈 | 高性能或简单配置 |
| **自定义网络** | 同一网络内容器可通过名称互访 | 多容器协作（AstrBot + NapCat + Milvus） |

### 实践洞察
- AstrBot、NapCat、Milvus 需放入同一自定义网络（如 `my-bridge`），才能用容器名互访。
- `host.docker.internal` 在某些 WSL2 配置下不可用，改用容器名更稳妥。

## 5. 数据持久化

| 方式 | 命令 | 适用场景 |
|------|------|----------|
| **Bind Mount** | `-v /宿主机路径:/容器内路径` | 配置文件、数据目录 |
| **Volume** | `-v 卷名:/容器内路径` | Docker 管理的存储 |
| **tmpfs** | `--tmpfs /容器内路径` | 临时数据，重启即丢 |

### 实践洞察
- **所有重要数据必须通过 `-v` 映射到宿主机**（如 `astrbot-data`、`napcat-data`）。
- 容器可随意删除重建，只要宿主机目录在，配置和数据就不丢。

## 6. Docker Compose

- 用 YAML 文件定义多个容器的启动配置。
- 一键启动/停止整套服务（如 Milvus 的 etcd + minio + standalone）。
- 命令：`docker-compose up -d` / `docker-compose down`。

### 实践洞察
- Milvus Standalone 通过 Compose 部署时，需确保内存充足。
- 若要切换网络，修改 Compose 文件的 `networks` 部分，指向已存在的 `my-bridge`。