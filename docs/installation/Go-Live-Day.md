# 正式上线当天执行顺序（超简版）

适用前提：

- 正式环境已部署完成
- 正式环境使用：`docker-compose.dokploy.production.yml`
- 域名：`idtoken.ai`
- Dokploy、PostgreSQL、Redis 都已可用

## 8 步执行顺序

### 1. 最后确认正式环境状态

- 确认 Dokploy 面板正常
- 确认 `new-api` / `postgres` / `redis` 都是运行状态
- 确认 HTTPS 已正常签发

### 2. 最后确认关键变量

至少确认：

- `SESSION_SECRET`
- `CRYPTO_SECRET`
- `NODE_TYPE=master`
- `NODE_NAME=new-api-1`
- `TRUSTED_REDIRECT_DOMAINS=idtoken.ai`

### 3. 先直接访问正式环境

确认：

- `https://idtoken.ai/`
- `https://idtoken.ai/api/status`

都能正常打开或返回正常结果。

### 4. 用测试 Token 做一次接口验证

至少验证：

- `GET /v1/models`
- `POST /v1/chat/completions`
- 如有流式需求，再做一次 SSE 验证

### 5. 确认回滚条件

上线前再确认一次：

- 测试环境仍可用
- 数据库已有备份
- 知道如何把 DNS 指回旧环境

### 6. 修改 `idtoken.ai` DNS

- 将 `A` 记录切到正式机器 IP
- 如果 TTL 已提前调低到 `300`，等待解析生效

### 7. 切流后立刻回归验证

再次检查：

- `https://idtoken.ai/api/status`
- 首页是否正常
- 后台是否可登录
- `POST /v1/chat/completions` 是否正常

### 8. 观察 5 分钟再宣布完成

重点看：

- CPU
- 内存
- Redis 连接数
- PostgreSQL 连接数
- `5xx` 错误率
- 请求延迟

确认无异常后，再宣布正式上线完成。

## 一句话版本

先验正式环境，再验接口，再确认回滚，再切 DNS，切完立刻回归，最后观察 5 分钟。

