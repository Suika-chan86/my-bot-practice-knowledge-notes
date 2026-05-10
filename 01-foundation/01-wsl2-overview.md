# WSL2 架构与 Windows 的关系

## 1. WSL2 的本质

- WSL2（Windows Subsystem for Linux 2）是微软提供的 Linux 系统调用翻译层。
- 它运行在一个由微软维护的轻量级 Linux 内核上，不是传统虚拟机。
- 启动速度极快，内存和 CPU 占用远低于 VirtualBox/VMware。

## 2. WSL2 的组成

| 组件 | 说明 |
|------|------|
| Linux 内核 | 微软编译的轻量内核，随 Windows Update 推送更新 |
| 发行版 | Ubuntu-22.04、Debian 等，从 Microsoft Store 安装 |
| 文件系统 | 虚拟 ext4 磁盘，存储在 `%LOCALAPPDATA%\Packages\...` |
| 文件互通 | Windows 可通过 `\\wsl$\发行版名\` 访问 Linux 文件；Linux 可通过 `/mnt/c/` 访问 Windows 文件 |

## 3. WSL2 的网络模式

| 模式 | 说明 | 适用场景 |
|------|------|----------|
| **NAT（默认）** | WSL2 拥有独立虚拟网络，通过 IP 转发上网 | 最稳定，推荐长期使用 |
| **Mirrored（镜像）** | WSL2 共享 Windows 的网络接口和 IP | 方便但兼容性差，已知会导致 Docker 卡死 |

### 实践洞察
- 在 Mirrored 模式下，Docker Desktop 曾反复卡在 “Docker Engine Starting”。
- 修改 `%USERPROFILE%\.wslconfig` 中的 `networkingMode=nat` 后恢复稳定。

## 4. WSL2 配置文件

位置：`%USERPROFILE%\.wslconfig`
```
[wsl2]  
memory=4GB # 限制内存使用  
processors=2 # 限制 CPU 核心数  
networkingMode=nat # 网络模式
```