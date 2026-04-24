# Dokploy 部署说明（含测试环境与正式环境配置）

本文档用于记录当前项目接入 Dokploy 的部署方式，以及测试环境、正式环境两套推荐配置。

当前业务前提：

- 主域名：`idtoken.ai`
- 请求类型：以文本类网络请求为主
- 不涉及图片、音频、视频等大媒体资源处理
- 未来需要支持正式环境扩容

## 适用场景

### 测试环境

- 云厂商：腾讯云
- 配置：`2 核 CPU / 4 GB 内存`
- 磁盘：`60 GB SSD`
- 带宽：`30 Mbps`
- 系统：`OpenCloudOS 9.4`
- 当前公网 IP：`43.156.68.154`
- Dokploy 面板：`http://43.156.68.154:3000`
- 目标：先完成域名接入、功能验证、后台配置、联调测试

### 正式机器

- 推荐起步配置：`4 核 CPU / 8 GB 内存`
- 目标：承载正式文本请求流量，并保留后续扩容空间

## Dokploy 接入方式

当前仓库已经自带一份面向 Dokploy 的 Compose 文件：`docker-compose.yml`。

推荐同时参考：

- Dokploy 环境变量模板：`.env.dokploy.example`
- 测试环境 Compose：`docker-compose.dokploy.test.yml`
- 正式环境 Compose：`docker-compose.dokploy.production.yml`
- GHCR 发布流程：`docs/installation/GHCR-Dokploy-Release.md`
- Dokploy 操作清单：`docs/installation/Dokploy-Checklist.md`
- 正式上线前检查清单：`docs/installation/Go-Live-Checklist.md`
- 上线当天超简版顺序：`docs/installation/Go-Live-Day.md`
- 运维收口清单：`docs/installation/Operations-Closeout-Checklist.md`
- PostgreSQL 备份恢复：`docs/installation/PostgreSQL-Backup-Restore.md`

建议流程：

1. 在 Dokploy 中创建 `Docker Compose` 应用
2. 导入当前仓库
3. 根据机器配置选择根目录下对应的 Compose 文件：
   - 测试环境（腾讯云 `2 核 4G`）：`docker-compose.dokploy.test.yml`
   - 正式环境（`4 核 8G`）：`docker-compose.dokploy.production.yml`
   - 通用版：`docker-compose.yml`
4. 参考根目录下的 `.env.dokploy.example` 整理 Variables / Secrets
5. 在 Dokploy 面板中配置环境变量与密钥
6. 在 Dokploy 的 Domains 页面绑定 `idtoken.ai`
7. 打开 HTTPS / Let's Encrypt

## 域名配置

### DNS

在 DNS 中添加：

- `A` 记录：`@ -> Dokploy 所在服务器公网 IP`
- 可选：`CNAME` 记录：`www -> idtoken.ai`

### Dokploy 域名

Dokploy 中必须将 `idtoken.ai` 配置在 Domains 页面统一管理。

说明：

- Compose 文件中不要手写 Traefik `labels`
- 域名、HTTP/HTTPS 路由、Let's Encrypt 证书都交给 Dokploy Domains 管理
- 如果后续域名变更，在 Dokploy Domains 中修改域名，并同步更新 `TRUSTED_REDIRECT_DOMAINS`
- 已验证：手写 Traefik `labels` 与 Dokploy Domains 同时存在时，可能导致 HTTP 正常但 HTTPS 命中 `TRAEFIK DEFAULT CERT`

## 部署架构建议

### 测试环境（2 核 4G）

建议用途：

- 仅部署单实例
- 以验证环境为主，不建议直接承接正式高并发流量

推荐方案：

- `new-api` 单实例
- `PostgreSQL` 单实例
- `Redis` 单实例
- 全部通过 Dokploy 管理

不建议：

- 使用 SQLite
- 开启多实例
- 在测试环境上同时承担高并发正式流量

### 正式机器（4 核 8G）

建议用途：

- 正式环境起步机型
- 文本请求网关场景
- 后续平滑扩容

推荐方案：

- `new-api` 单实例起步
- `PostgreSQL` 与 `Redis` 可先同机，流量增长后独立拆出
- Dokploy 管理应用、域名、证书、环境变量

## 环境变量建议

### 通用必填项

以下变量建议在 Dokploy 中统一配置：

```env
TZ=Asia/Shanghai

APP_IMAGE=ghcr.io/你的 GitHub 用户名或组织/new-api:test

POSTGRES_PASSWORD=请替换为强密码
REDIS_PASSWORD=请替换为强密码

SQL_DSN=postgresql://root:${POSTGRES_PASSWORD}@postgres:5432/new-api
REDIS_CONN_STRING=redis://:${REDIS_PASSWORD}@redis:6379

SESSION_SECRET=请替换为随机长字符串
CRYPTO_SECRET=请替换为随机长字符串

TRUSTED_REDIRECT_DOMAINS=idtoken.ai
```

> 如果 PostgreSQL / Redis 使用 Dokploy 的独立数据库服务或外部托管服务，请将 `postgres`、`redis` 替换为实际主机名。
>
> 如果暂时还不切到 GHCR，也可以先不配置 `APP_IMAGE`，Compose 会回退到默认镜像。

### 测试环境配置（腾讯云 2 核 4G）

适合验证环境与低流量使用：

```env
NODE_TYPE=master
NODE_NAME=new-api-test

MEMORY_CACHE_ENABLED=true
BATCH_UPDATE_ENABLED=true
SYNC_FREQUENCY=60

SQL_MAX_OPEN_CONNS=20
SQL_MAX_IDLE_CONNS=8
SQL_MAX_LIFETIME=300

REDIS_POOL_SIZE=20

RELAY_MAX_IDLE_CONNS=100
RELAY_MAX_IDLE_CONNS_PER_HOST=50

STREAMING_TIMEOUT=180
MAX_REQUEST_BODY_MB=4
MAX_FILE_DOWNLOAD_MB=4

ERROR_LOG_ENABLED=false
UPDATE_TASK=false
CountToken=false
```

说明：

- `2 核 4G` 不建议保留默认的大连接池
- 当前业务无大媒体请求，可将请求体限制调低，降低内存压力
- `CountToken=false` 属于并发优化项；如果后续业务依赖本地 token 预估能力，可改回 `true`

### 正式机器配置（4 核 8G）

适合文本请求为主的正式环境：

```env
NODE_TYPE=master
NODE_NAME=new-api-1

MEMORY_CACHE_ENABLED=true
BATCH_UPDATE_ENABLED=true
SYNC_FREQUENCY=30

SQL_MAX_OPEN_CONNS=60
SQL_MAX_IDLE_CONNS=20
SQL_MAX_LIFETIME=300

REDIS_POOL_SIZE=50

RELAY_MAX_IDLE_CONNS=500
RELAY_MAX_IDLE_CONNS_PER_HOST=150

STREAMING_TIMEOUT=180
MAX_REQUEST_BODY_MB=8
MAX_FILE_DOWNLOAD_MB=8

ERROR_LOG_ENABLED=false
UPDATE_TASK=false
CountToken=false
```

补充建议：

- 如果后续需要开启错误日志，建议同时配置独立的 `LOG_SQL_DSN`
- 如未观察到内存压力，可适度将 `MAX_REQUEST_BODY_MB` 调回 `8~16`
- 正式环境建议将数据库和 Redis 监控纳入云监控或 Dokploy 监控体系

## 本项目中需要重点关注的配置项

### 1. 数据库连接池

项目默认值较大：

- `SQL_MAX_IDLE_CONNS` 默认 `100`
- `SQL_MAX_OPEN_CONNS` 默认 `1000`

相关代码位置：

- `model/main.go`
- `common/init.go`

对于 `2 核 4G` 或 `4 核 8G` 机器，建议显式下调，避免数据库先成为瓶颈。

### 2. Redis 连接池

项目默认：

- `REDIS_POOL_SIZE=10`

相关代码位置：

- `common/redis.go`

高并发文本请求下建议显式增大。

### 3. 内存缓存与同步频率

建议：

- `MEMORY_CACHE_ENABLED=true`
- `SYNC_FREQUENCY=30~60`

相关代码位置：

- `model/channel_cache.go`
- `model/option.go`

这样可以为后续多节点扩容提前做好准备。

### 4. 批量更新

建议：

- `BATCH_UPDATE_ENABLED=true`

相关代码位置：

- `model/utils.go`

用于减少额度、计数等高频写数据库带来的压力。

### 5. 错误日志

建议：

- 默认关闭 `ERROR_LOG_ENABLED`

原因：

- 错误日志开启后会额外写数据库
- 测试环境和低配机器不适合直接打开

相关代码位置：

- `controller/relay.go`

### 6. 后台任务

如果当前环境仅承接文本转发请求，建议：

- `UPDATE_TASK=false`

原因：

- 可以减少不必要的异步任务维护开销

### 7. 超时配置

建议保留：

- `STREAMING_TIMEOUT=180~300`

不建议在当前阶段随意设置：

- `RELAY_TIMEOUT`

原因：

- 文本流式请求使用 SSE 时，`http.Client.Timeout` 容易误伤长连接响应

相关代码位置：

- `service/http_client.go`

## Dokploy 部署操作建议

### 应用服务

`new-api` 服务可按场景选择以下 Compose 文件：

- 通用版：`docker-compose.yml`
- 测试环境版（腾讯云 `2 核 4G`）：`docker-compose.dokploy.test.yml`
- 正式环境版（`4 核 8G`）：`docker-compose.dokploy.production.yml`

如果只想先快速验证：

- 优先使用 `docker-compose.dokploy.test.yml`
- 在 Dokploy 中补充环境变量
- 绑定 `idtoken.ai`

如果准备部署正式环境：

- 优先使用 `docker-compose.dokploy.production.yml`
- 结合 `.env.dokploy.example` 调整变量
- 后续根据流量决定是否拆分 PostgreSQL / Redis

### 数据库与 Redis

可选两种方式：

#### 方式一：直接使用当前 Compose 中的 PostgreSQL / Redis

优点：

- 最快上线
- 适合测试环境

缺点：

- 备份、迁移、独立扩容不如拆分服务方便

#### 方式二：在 Dokploy 中单独创建 PostgreSQL / Redis 服务

优点：

- 更适合正式环境
- 备份和迁移更清晰
- 后期拆分应用节点更容易

更推荐正式环境使用这种方式。

## 从测试环境迁移到正式环境

建议流程：

1. 正式机器部署 Dokploy
2. 在正式机器中按同样方式创建 `new-api`、`PostgreSQL`、`Redis`
3. 复制测试环境中的环境变量
4. 保持 `SESSION_SECRET`、`CRYPTO_SECRET` 一致
5. 导出测试环境数据库并导入正式环境
6. Redis 可直接重建，一般无需迁移历史缓存
7. 将 DNS TTL 先调低到 `300`
8. 切换 `idtoken.ai` 的 `A` 记录到正式机器
9. 验证：
   - `https://idtoken.ai/api/status`
   - 后台登录
   - `/v1` 转发请求

## 正式环境扩容建议

### 第一阶段：单机增强

先从 `4 核 8G` 单实例开始，重点观察：

- CPU 使用率
- Redis 连接数
- PostgreSQL 连接数
- 请求响应时间
- 5xx 错误率

### 第二阶段：拆分数据库与 Redis

当正式流量增长后，优先拆出：

- PostgreSQL
- Redis

应用层仍可先保持单实例。

### 第三阶段：应用多实例

扩容到多实例时：

- 仅保留一个 `master`
- 其他实例设置为 `slave`
- 所有实例共享同一套：
  - `SQL_DSN`
  - `REDIS_CONN_STRING`
  - `SESSION_SECRET`
  - `CRYPTO_SECRET`

示例：

```env
NODE_TYPE=slave
NODE_NAME=new-api-2
MEMORY_CACHE_ENABLED=true
SYNC_FREQUENCY=30
```

## 检查清单

### 测试环境上线前

- [ ] `idtoken.ai` 已解析到当前服务器
- [ ] Dokploy 已绑定域名并签发 HTTPS
- [ ] `SESSION_SECRET` 已设置
- [ ] `CRYPTO_SECRET` 已设置
- [ ] 数据库与 Redis 已正常连接
- [ ] `ERROR_LOG_ENABLED=false`
- [ ] `SQL_MAX_OPEN_CONNS` 已显式设置
- [ ] `REDIS_POOL_SIZE` 已显式设置
- [ ] 使用的 Compose 文件为 `docker-compose.dokploy.test.yml`

### 正式环境切换前

- [ ] 正式机器已部署完成
- [ ] 正式环境已导入数据库
- [ ] 测试与正式环境的关键密钥保持一致
- [ ] `https://idtoken.ai/api/status` 已验证
- [ ] 管理后台已验证可登录
- [ ] `/v1` 转发请求已验证可用
- [ ] 使用的 Compose 文件为 `docker-compose.dokploy.production.yml`
