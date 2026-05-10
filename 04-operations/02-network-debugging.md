# 容器网络排查方法论

## 1. 排查金字塔（从底层到顶层）
```
第0层：硬件/系统 → 电脑是否联网？Docker Desktop 是否启动？
第1层：容器存活 → docker ps 容器是否 Up？
第2层：网络拓扑 → 容器是否在同一自定义网络？
第3层：DNS 解析 → 容器名是否能被解析为 IP？
第4层：端口连通 → 目标端口是否在监听？能否 TCP 握手？
第5层：应用协议 → HTTP/WebSocket 是否正常响应？
第6层：业务逻辑 → Token 是否正确？路径是否匹配？
```

## 2. 逐层排错命令

| 层级 | 命令 | 目的 |
|------|------|------|
| 1 | `docker ps` | 确认容器存活 |
| 2 | `docker network inspect my-bridge` | 查看容器是否都在同一网络 |
| 3 | `docker exec -it astrbot ping milvus-standalone` | 测试容器名解析 |
| 4 | `docker exec -it astrbot nc -zv milvus-standalone 19530` | 测试端口连通 |
| 5 | `docker exec -it astrbot curl -I http://milvus:19530` | 测试 HTTP 响应 |
| 6 | `docker logs astrbot --tail 50` | 查看应用层报错 |

## 3. 常见网络故障模式

### 3.1 容器名无法解析
- **现象**：`Could not resolve host: milvus`
- **原因**：容器不在同一自定义网络中
- **修复**：`docker network connect my-bridge 容器名`

### 3.2 端口不通
- **现象**：`Connection refused`
- **原因**：目标服务未启动或未监听该端口
- **修复**：检查目标容器日志，确保服务已就绪

### 3.3 WebSocket 连接超时
- **现象**：`Opening handshake timed out`
- **原因**：IP 或端口不通，或 Token 不匹配
- **修复**：
  1. 确认 AstrBot 容器的 6199 端口已映射（`-p 6199:6199`）
  2. 在 NapCat 配置中用容器名代替 IP
  3. 清空 Token 测试，确认连通后再加回

### 3.4 host.docker.internal 不可用
- **现象**：`Could not resolve host: host.docker.internal`
- **原因**：WSL2 某些网络模式下该 DNS 名不可用
- **修复**：改用自定义网络 + 容器名互访

## 4. 最佳实践

1. **优先使用自定义网络**：用 `docker network create` 创建共享网络，所有相关容器加入。
2. **用容器名而不是 IP**：容器重启后 IP 可能变化，容器名不变。
3. **端口映射只给宿主机访问**：容器间通信走 Docker 内部网络，不需要端口映射。
4. **固定 MAC 地址**（NapCat）：防止 QQ 服务器认为换了新设备。
