# WSL2 与 Docker Desktop 协作原理

## 1. 它们是什么关系？

WSL2 和 Docker Desktop 不是“二选一”的替代品，而是**分工协作的上下级**。
```
你的笔记本电脑
|
├── Windows 系统
|   |
|   ├── 你的应用程序 (浏览器、QQ、文件管理器)
|   |
|   ├── Docker Desktop (Windows GUI)
|   |   |
|   |   └── 背后依赖一个小型 Linux 虚拟机 (由 WSL2 提供)
|   |
|   └── WSL2 功能 (子系统)
|       |
|       ├── 轻量级 Linux 内核 (由微软提供，与 Windows 并行运行)
|       |
|       ├── 你可以安装多个 Linux 发行版 (如 Ubuntu-22.04)
|       |
|       └── 在发行版内部，你可以像操作真正的 Linux 一样
|           |
|           ├── 安装 Docker Engine
|           ├── 运行容器
|           └── 使用命令行工具
```

## 2. Docker Desktop 在 WSL2 中的架构

Docker Desktop 安装后，会自动创建两个 WSL2 发行版：

| 发行版 | 作用 | 用户是否可见 |
|--------|------|--------------|
| `docker-desktop` | 运行 Docker Engine（守护进程） | 否（后台运行） |
| `docker-desktop-data` | 存储镜像、容器、卷等数据 | 否（后台运行） |
| 你的发行版（如 Ubuntu-22.04） | 提供 `docker` 命令行工具 | **是**（你操作的终端） |

关键原理：
- Docker 守护进程（dockerd）跑在 `docker-desktop` 这个隐藏的 WSL2 发行版里。
- 你在 Ubuntu-22.04 终端里敲 `docker ps`，实际是通过 `/var/run/docker.sock` 这个 socket 文件与守护进程通信。
- 所有容器本质上都是在 `docker-desktop` 发行版的内核上运行的。

## 3. 文件系统的协作

### 3.1 容器数据持久化
当你执行：
```bash
docker run -v /home/suikachan86/astrbot-data:/AstrBot/data ...
```
- 左边的 `/home/suikachan86/astrbot-data` 是你 Ubuntu-22.04 发行版中的路径。
- 右边的 `/AstrBot/data` 是容器内的路径。
- Docker 会在两者之间建立一个绑定挂载，数据实际存储在 Ubuntu 的 ext4 文件系统中。

## 3.2 Windows 与 WSL2 文件互通

* **Windows → WSL2**：在资源管理器地址栏输入 `\\wsl$\Ubuntu-22.04\home\suikachan86` 即可访问。
* **WSL2 → Windows**：在 Linux 终端中 `/mnt/c/` 对应 Windows 的 C 盘。

## 3.3 性能差异

* 在 WSL2 的 **ext4** 文件系统内读写文件非常快（接近原生 Linux）。
* 通过 `/mnt/c/` 跨文件系统访问 Windows 文件较慢（走网络协议转换）。
* **因此，Docker 的 `-v` 挂载务必使用 WSL2 内部的路径，不要指向 `/mnt/c/`。**

---

## 4. 网络协作

### 4.1 默认 NAT 模式（你的当前配置）

* WSL2 内的所有东西（发行版、容器）共享一个独立的虚拟子网。
* 访问外网时，通过 **NAT**（网络地址转换）把内部 IP 转成 Windows 的 IP 出去。
* 容器间通过自定义网络（如 `my-bridge`）可以互通。

### 4.2 Mirrored 模式（不推荐）

* WSL2 共享 Windows 的网络接口，IP 地址和 Windows 一模一样。
* **好处**：`localhost` 就可以直接访问 WSL2 内的服务。
* **坏处**：稳定性极差，Docker Desktop 经常卡在 “Engine starting”，亲身经历已证实这一点。

### 4.3 容器如何访问宿主机？

* 在 NAT 模式下，`localhost` 不行，因为容器和宿主机不在同一个网络栈。
* Docker Desktop 提供了一个特殊 DNS 名 `host.docker.internal`，指向宿主机。
* **更稳妥的方式**：把需要通信的容器放进同一自定义网络，用**容器名**互访。

---

## 5. 资源分配

WSL2 默认会动态占用宿主机的内存和 CPU，但可通过 `%USERPROFILE%\.wslconfig` 限制：

```text
[wsl2]
memory=4GB
processors=2
networkingMode=nat
```


**实践洞察：**

* **Milvus Standalone** 在 4GB 内存下仍然可能 **OOM**，建议至少分配 6GB 或使用更轻量的记忆插件。
* `--restart=unless-stopped` 保证容器开机自启，无需手动干预。

---

## 6. 常见故障模式

| 现象 | 根因 | 解决方案 |
| --- | --- | --- |
| Docker Desktop 卡在 “Engine starting” | Mirrored 网络模式导致握手失败 | `.wslconfig` 改为 `networkingMode=nat` |
| WSL 与 Docker 集成断开 | WSL 发行版网络栈假死 | 点击 Docker Desktop 弹窗的 “Restart WSL integration” |
| 重启后容器没自动启动 | 曾手动 `docker stop` 过，策略记作“手动停” | `docker start` 重新拉起，或改用 `--restart=always` |
| 容器间 ping 不通 | 不在同一自定义网络 | `docker network connect my-bridge 容器名` |
| `host.docker.internal` 解析失败 | NAT 模式下该 DNS 名不保证生效 | 改用自定义网络 + 容器名互访 |

---

## 7. 一句话总结

WSL2 提供 Linux 内核和文件系统，Docker Desktop 提供容器引擎和图形管理。你把数据通过 `-v` 放在 WSL2 的 **ext4** 文件系统上，容器跑在 WSL2 内核里，通过网络与外部通信。

**两者的协作本质是：WSL2 做苦力，Docker Desktop 做管家，你只需要在 Ubuntu 终端里敲命令。**