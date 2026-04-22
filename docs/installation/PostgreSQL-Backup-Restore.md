# PostgreSQL 备份与恢复

本文档记录当前 `new-api` 测试环境在 Dokploy 部署下的 PostgreSQL 手动备份与恢复方式。

适用前提：

- 部署方式：Dokploy `Docker Compose`
- 数据库：`PostgreSQL`
- 数据库名：`new-api`
- 数据库用户：`root`

## 1. 手动备份

先创建备份目录：

```bash
mkdir -p /root/db-backups
```

执行备份：

```bash
docker exec -t $(docker ps --format '{{.Names}}' | grep postgres | head -n 1) \
  pg_dump -U root -d new-api > /root/db-backups/new-api-$(date +%F-%H%M%S).sql
```

查看备份文件：

```bash
ls -lh /root/db-backups
```

## 2. 压缩备份

压缩 SQL 文件：

```bash
gzip /root/db-backups/new-api-*.sql
```

再次查看：

```bash
ls -lh /root/db-backups
```

压缩后通常得到类似文件：

```text
/root/db-backups/new-api-2026-04-22-120000.sql.gz
```

## 3. 恢复备份

恢复命令：

```bash
gunzip -c /root/db-backups/new-api-xxxxxx.sql.gz | \
docker exec -i $(docker ps --format '{{.Names}}' | grep postgres | head -n 1) \
psql -U root -d new-api
```

说明：

- 恢复前建议先确认目标环境数据库状态
- 正式环境恢复前建议先做一次当前库备份
- 恢复到正式环境时，建议先在测试环境验证导入文件可用

## 4. 推荐备份时机

建议至少在以下场景执行手动备份：

- 后台初始化完成后
- 大规模修改渠道前
- 数据清理前
- 切换正式环境前
- 执行数据库迁移前

## 5. 备份文件管理建议

- 备份文件不要只留在服务器本机
- 至少下载一份到本地
- 更推荐额外上传到对象存储或企业网盘
- 保留最近 7 天备份
- 重要上线前的备份建议单独留档

## 6. 当前环境建议

对当前 `idtoken.ai` 测试环境，建议：

- 至少先执行 1 次手动备份
- 记录最近一次备份时间
- 记录备份文件存放位置
- 后续再补定时备份脚本

