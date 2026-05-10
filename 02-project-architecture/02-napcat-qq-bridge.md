# NapCat 与 QQ 的通信原理

## 1. NapCat 的角色
NapCat 是一个基于 NTQQ 协议的 QQ 机器人框架，它将自己伪装成一个真实的 QQ 客户端，通过 hook 技术注入到 QQ 中，从而接收和发送消息。

在与 AstrBot 的协作中，NapCat 充当 **QQ 与 OneBot 之间的翻译官**。

## 2. 整体通信链路
```
QQ 用户 → 腾讯服务器 → NapCat (NTQQ Hook) → WebSocket → AstrBot (OneBot)
```
NapCat 收到 QQ 消息后，会将其转换为 OneBot v11 标准格式的事件，然后通过 WebSocket 推送给 AstrBot。反之，AstrBot 调用 OneBot API 发送的消息，也由 NapCat 代为执行。

## 3. OneBot 协议核心概念
- **事件 (Event)**：收到消息时，NapCat 产生一个事件，如 `message.private.friend`。
- **API 调用**：AstrBot 向 NapCat 发送 action（如 `send_msg`），NapCat 执行后返回结果。
- **WebSocket**：通信双向通道，NapCat 作为服务端或客户端与 AstrBot 建立连接。

## 4. 连接模式
- **正向 WebSocket**：AstrBot 主动连接 NapCat（较少用）。
- **反向 WebSocket**：NapCat 主动连接 AstrBot（目前在用的模式）。
  - NapCat 配置中填写 `ws://astrbot:6199/ws/`（不填或自定义 Token）。
  - AstrBot 监听 `0.0.0.0:6199`，等待 NapCat 连接。

## 5. 登录与身份验证
- NapCat 通过读取本地的 QQ 登录态文件（cookie / token）实现免扫码登录。
- 首次登录需要打开 WebUI (`localhost:6099`) 扫码。
- 开启 QQ 的“登录保护”后，NapCat 被视为独立设备，授权后即可长期在线。
- 固定容器的 MAC 地址可以避免每次重启被识别为新设备。

## 6. 常见问题与优化
- **掉线重连**：NapCat 内部有断线重连机制，心跳间隔设置为 30 秒较安全。
- **风控与封号**：
  - 使用小号，避免主号被波及。
  - 不要同时手机登录同号，避免挤掉线。
  - 控制消息频率，开启健康模式。
- **网络隔离**：必须与 AstrBot 在同一自定义网络（如 `my-bridge`），否则容器名无法解析。

## 7. 实践洞察
- NapCat 的持久化数据只需保留 `napcat-data` 目录，里面包含登录凭证和设备指纹。
- 更换电脑时，拷贝该目录即可大概率免扫码。
- 若需要固定 MAC，重建容器时加上 `--mac-address 02:42:ac:11:**:**` 保持设备指纹一致。