# 数据备份与迁移策略

## 1. 核心原则

> **所有重要数据必须落在宿主机文件系统上，不能只存在于容器内。**

## 2. 关键备份目录

| 目录 | 内容 | 重要性 |
|------|------|--------|
| `/home/suikachan86/astrbot-data/` | AstrBot 全部配置、人格、知识库、插件数据 | ★★★★★ |
| `/home/suikachan86/napcat-data/` | NapCat QQ 登录凭证、设备信息 | ★★★★★ |
| `/home/suikachan86/milvus/` | Milvus Compose 文件和数据卷（如用过） | ★★★ |

## 3. 打包命令

```bash
# 备份 AstrBot
cd /home/suikachan86
tar -czvf astrbot-data-backup.tar.gz astrbot-data/

# 备份 NapCat
tar -czvf napcat-data-backup.tar.gz napcat-data/
```

## 4. 跨电脑迁移步骤
1. 新电脑安装 WSL2 + Docker Desktop。

2. 将 `.tar.gz` 文件复制到新 WSL 的家目录。

3. 解压：
```bash
tar -xzvf astrbot-data-backup.tar.gz -C ~/
tar -xzvf napcat-data-backup.tar.gz -C ~/
```
4. 用相同的 `docker run` 命令重建容器（网络、端口、挂载路径保持一致）。

5. 对 NapCat 加上 `--mac-address` 固定 MAC，减少重新扫码概率。

## 5. 哪些可能“变”

| 项目 | 是否保留 | 备注 |
| :--- | :--- | :--- |
| 人格配置 | ✅ 完全保留 | 存在 `astrbot-data/` |
| 知识库 | ✅ 完全保留 | 文档和向量索引 |
| 长期记忆 | ✅ 保留（APLR 的 SQLite/ChromaDB 文件） | 需一同备份 |
| NapCat 登录状态 | ⚠️ 大概率保留 | 换电脑可能需重新扫码 |
| 容器本身 | ❌ 不保留 | 在新电脑上重建即可 |

## 6. 定期备份建议
- 频率：修改重要配置后立即备份；每周全量备份一次。

- 存储位置：U盘 + 云端双重备份。

- 自动化：编写脚本或用 crontab 定时打包。