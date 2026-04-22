# 正式上线前最后 15 分钟检查清单

本文档用于在正式切流前做最后一轮快速确认。

适用前提：

- 正式环境已部署在 Dokploy
- 正式环境使用：`docker-compose.dokploy.production.yml`
- 域名：`idtoken.ai`
- 正式机器配置：`4 核 CPU / 8 GB 内存`

## 0. 先确认切流策略

- [ ] 已确认本次切流窗口
- [ ] 已确认谁负责 DNS 修改
- [ ] 已确认谁负责上线后验证
- [ ] 已确认出现问题时可快速回滚

## 1. 机器与服务状态

- [ ] 正式机器可 SSH 登录
- [ ] Dokploy 面板可正常访问
- [ ] `new-api` 服务状态正常
- [ ] `postgres` 服务状态正常
- [ ] `redis` 服务状态正常

## 2. 域名与网络

- [ ] `idtoken.ai` 的 DNS 已准备好
- [ ] 80 / 443 端口已放行
- [ ] 云安全组已放行 80 / 443
- [ ] 域名 TTL 已调低到 `300`（如可操作）

## 3. HTTPS 与访问入口

- [ ] Dokploy 已绑定 `idtoken.ai`
- [ ] HTTPS 已启用
- [ ] Let’s Encrypt 已签发成功
- [ ] `https://idtoken.ai/` 可打开
- [ ] `https://idtoken.ai/api/status` 返回正常

## 4. 环境变量

重点确认这些值已经配置：

- [ ] `SESSION_SECRET`
- [ ] `CRYPTO_SECRET`
- [ ] `POSTGRES_PASSWORD`
- [ ] `REDIS_PASSWORD`
- [ ] `TRUSTED_REDIRECT_DOMAINS=idtoken.ai`
- [ ] `NODE_TYPE=master`
- [ ] `NODE_NAME=new-api-1`

## 5. 正式环境性能参数

- [ ] `MEMORY_CACHE_ENABLED=true`
- [ ] `BATCH_UPDATE_ENABLED=true`
- [ ] `SYNC_FREQUENCY=30`
- [ ] `SQL_MAX_OPEN_CONNS=60`
- [ ] `SQL_MAX_IDLE_CONNS=20`
- [ ] `SQL_MAX_LIFETIME=300`
- [ ] `REDIS_POOL_SIZE=50`
- [ ] `RELAY_MAX_IDLE_CONNS=500`
- [ ] `RELAY_MAX_IDLE_CONNS_PER_HOST=150`
- [ ] `STREAMING_TIMEOUT=180`
- [ ] `MAX_REQUEST_BODY_MB=8`
- [ ] `MAX_FILE_DOWNLOAD_MB=8`
- [ ] `ERROR_LOG_ENABLED=false`
- [ ] `UPDATE_TASK=false`
- [ ] `CountToken=false`

## 6. 数据与后台

- [ ] 数据库已导入正式环境
- [ ] 管理员账号可登录
- [ ] 关键渠道已存在
- [ ] 测试 Token 可正常使用
- [ ] 必要的站点设置已完成

## 7. 上线前接口验证

至少做一次：

- [ ] `GET /api/status`
- [ ] `GET /v1/models`
- [ ] `POST /v1/chat/completions`
- [ ] 流式 SSE 请求验证

## 8. 回滚准备

- [ ] 测试环境仍保留可用
- [ ] 数据库有备份
- [ ] 如需回滚，知道如何把 DNS 指回旧环境
- [ ] 团队知道回滚联系人

## 9. 切流后 5 分钟内观察项

切流后重点观察：

- [ ] CPU 使用率
- [ ] 内存使用率
- [ ] Redis 连接数
- [ ] PostgreSQL 连接数
- [ ] `5xx` 错误率
- [ ] `/v1/chat/completions` 延迟

## 10. 最终执行顺序

1. 最后确认正式环境状态
2. 修改 `idtoken.ai` DNS
3. 等待解析生效
4. 访问首页
5. 访问 `/api/status`
6. 用测试 Token 发一次 `/v1/chat/completions`
7. 观察 5 分钟
8. 确认无异常后宣布上线完成

