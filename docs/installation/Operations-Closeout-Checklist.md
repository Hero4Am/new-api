# 运维收口清单

本文档记录 `idtoken.ai` 测试环境部署完成后的最后收口事项，适用于 Dokploy + Docker Compose 部署方式。

## 当前环境

- 业务域名：`idtoken.ai`
- Dokploy 面板域名：`dokploy.idtoken.ai`
- 测试环境公网 IP：`43.156.68.154`
- 测试环境配置：腾讯云 `2 核 CPU / 4 GB 内存 / 60 GB SSD / 30 Mbps`
- 测试环境系统：`OpenCloudOS 9.4`
- 部署方式：Dokploy `Docker Compose`
- Compose 文件：`docker-compose.dokploy.test.yml`

## 1. Dokploy 面板

- [ ] `https://dokploy.idtoken.ai` 可打开
- [ ] Dokploy 面板 HTTPS 正常
- [ ] `3000` 端口仅允许固定公网 IP 访问
- [ ] `22` 端口仅允许固定公网 IP 访问
- [ ] 不再长期使用 `http://43.156.68.154:3000` 作为日常访问入口

## 2. 业务域名

- [ ] `https://idtoken.ai` 可打开
- [ ] `https://idtoken.ai/api/status` 返回 `200`
- [ ] HTTPS 证书正常，浏览器无安全告警
- [ ] `server_address` 已配置为 `https://idtoken.ai`

验证命令：

```bash
curl -i https://idtoken.ai/api/status
curl -s https://idtoken.ai/api/status | jq '.data.server_address'
```

期望：

```text
HTTP 200
"https://idtoken.ai"
```

## 3. 容器状态

- [ ] `new-api` 容器为 `running`
- [ ] `new-api` 健康检查为 `healthy`
- [ ] `postgres` 容器为 `running`
- [ ] `redis` 容器为 `running`

可在 Dokploy 的 `Containers` 页面检查。

## 4. 后台初始化

- [ ] root 管理员已创建
- [ ] root 管理员可正常登录
- [ ] 系统使用模式为 `Default mode`
- [ ] 后台基础配置已保存

## 5. 业务联调

- [ ] 已添加至少 1 条真实上游渠道
- [ ] 已创建 1 个测试 Token
- [ ] 已验证 `/v1/models`
- [ ] 已验证 `/v1/chat/completions`
- [ ] 已验证 SSE 流式响应

## 6. 备份

- [ ] PostgreSQL 已配置备份方案
- [ ] 至少完成 1 次手动备份
- [ ] 备份文件保存到安全位置
- [ ] 已记录恢复步骤

建议至少保留：

- 每日备份
- 最近 7 天备份
- 重要上线前手动备份

## 7. 密钥归档

以下信息必须保存到安全位置，不要提交到 Git：

- [ ] `POSTGRES_PASSWORD`
- [ ] `REDIS_PASSWORD`
- [ ] `SESSION_SECRET`
- [ ] `CRYPTO_SECRET`
- [ ] root 管理员账号
- [ ] root 管理员密码

说明：

- 测试环境迁移到正式环境时，建议保持 `SESSION_SECRET` 与 `CRYPTO_SECRET` 一致
- 否则可能影响登录会话、加密数据和历史兼容性

## 8. 代码与文档

- [ ] 本地部署配置已提交
- [ ] 文档已提交
- [ ] 变更已推送到远程仓库

当前已整理的关键文档：

- `docs/installation/Dokploy.md`
- `docs/installation/Dokploy-Checklist.md`
- `docs/installation/Go-Live-Checklist.md`
- `docs/installation/Go-Live-Day.md`
- `docs/installation/Operations-Closeout-Checklist.md`
- `docs/installation/PostgreSQL-Backup-Restore.md`

## 9. 正式环境准备

- [ ] 正式机器配置按 `4 核 CPU / 8 GB 内存` 起步
- [ ] 正式环境继续使用 Dokploy `Domains` 管理域名和 HTTPS
- [ ] Compose 文件中不要手写 Traefik `labels`
- [ ] 正式环境使用 `docker-compose.dokploy.production.yml`
- [ ] 正式环境复用测试环境的 `SESSION_SECRET`
- [ ] 正式环境复用测试环境的 `CRYPTO_SECRET`
- [ ] 正式切流前完成数据库备份与恢复验证

## 10. 已验证的重要结论

- `idtoken.ai` 可继续使用 Spaceship DNS，不必立即迁移 Cloudflare
- Spaceship 中 `A @ -> 43.156.68.154` 已可满足当前测试环境访问
- Dokploy `Domains` 已可自动申请 Let's Encrypt 免费证书
- Let's Encrypt 证书由 Dokploy / Traefik 自动续期
- Dokploy `Domains` 与 Compose 中手写 Traefik `labels` 同时存在时，可能导致 HTTPS 命中 `TRAEFIK DEFAULT CERT`
- 当前推荐做法：域名、HTTP/HTTPS 路由、证书统一交给 Dokploy `Domains` 管理
