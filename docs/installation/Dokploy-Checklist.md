# Dokploy 部署操作清单

本文档提供一份按页面操作的 Dokploy 部署清单，适合直接照着执行。

适用前提：

- 域名：`idtoken.ai`
- 测试环境：腾讯云 `2 核 CPU / 4 GB 内存`
- 测试环境当前系统：`OpenCloudOS 9.4`
- 测试环境当前公网 IP：`43.156.68.154`
- 测试环境当前 Dokploy 面板：`http://43.156.68.154:3000`
- 正式机器：`4 核 CPU / 8 GB 内存`
- 业务类型：以文本网络请求为主，不涉及图片、音频、视频资源

## 先决定用哪套配置

### 测试环境

使用：

- Compose 文件：`docker-compose.dokploy.test.yml`
- 环境变量模板：`.env.dokploy.example`

适合：

- 域名接入
- 功能验证
- 后台配置
- 联调测试

### 正式机器

使用：

- Compose 文件：`docker-compose.dokploy.production.yml`
- 环境变量模板：`.env.dokploy.example`

适合：

- 正式环境起步
- 文本请求流量承接
- 后续扩容准备

## 第 1 步：准备服务器

在目标服务器确认：

- [ ] 已安装 Docker
- [ ] 已安装 Dokploy
- [ ] 服务器有公网 IP
- [ ] 80 / 443 端口已放行
- [ ] 云厂商安全组已放行 80 / 443

如果是测试环境：

- [ ] 当前机器配置为腾讯云 `2 核 4G`

如果是正式机器：

- [ ] 当前机器配置至少为 `4 核 8G`

## 第 2 步：准备 DNS

在域名服务商控制台添加：

- [ ] `A` 记录：`@ -> Dokploy 所在服务器公网 IP`
- [ ] 可选：`CNAME` 记录：`www -> idtoken.ai`

建议：

- [ ] 将 TTL 先设置为 `300`

## 第 3 步：准备环境变量

以 `.env.dokploy.example` 为基础，至少准备这些值：

- [ ] `POSTGRES_PASSWORD`
- [ ] `REDIS_PASSWORD`
- [ ] `SESSION_SECRET`
- [ ] `CRYPTO_SECRET`
- [ ] `TRUSTED_REDIRECT_DOMAINS=idtoken.ai`

生成随机密钥示例：

```bash
openssl rand -hex 32
openssl rand -hex 32
```

## 第 4 步：在 Dokploy 中创建项目

进入 Dokploy 后：

- [ ] 新建一个 Project
- [ ] 项目名称建议使用：`new-api`

## 第 5 步：创建 Docker Compose 应用

在 Project 下：

- [ ] 点击 `Create Application`
- [ ] 选择 `Docker Compose`
- [ ] 连接 Git 仓库或上传 Compose 内容

如果使用当前仓库：

- [ ] 选择仓库根目录
- [ ] Branch 选择你要部署的分支

Compose 文件选择：

- 测试环境：`docker-compose.dokploy.test.yml`
- 正式环境：`docker-compose.dokploy.production.yml`

## 第 6 步：填写 Variables / Secrets

在 Dokploy 的 Variables 或 Secrets 页面中录入：

### 基础项

- [ ] `TZ=Asia/Shanghai`
- [ ] `POSTGRES_USER=root`
- [ ] `POSTGRES_PASSWORD=...`
- [ ] `POSTGRES_DB=new-api`
- [ ] `POSTGRES_HOST=postgres`
- [ ] `REDIS_PASSWORD=...`
- [ ] `REDIS_HOST=redis`
- [ ] `SESSION_SECRET=...`
- [ ] `CRYPTO_SECRET=...`
- [ ] `TRUSTED_REDIRECT_DOMAINS=idtoken.ai`

### 测试环境建议值

- [ ] `NODE_TYPE=master`
- [ ] `NODE_NAME=new-api-test`
- [ ] `MEMORY_CACHE_ENABLED=true`
- [ ] `BATCH_UPDATE_ENABLED=true`
- [ ] `SYNC_FREQUENCY=60`
- [ ] `SQL_MAX_OPEN_CONNS=20`
- [ ] `SQL_MAX_IDLE_CONNS=8`
- [ ] `SQL_MAX_LIFETIME=300`
- [ ] `REDIS_POOL_SIZE=20`
- [ ] `RELAY_MAX_IDLE_CONNS=100`
- [ ] `RELAY_MAX_IDLE_CONNS_PER_HOST=50`
- [ ] `STREAMING_TIMEOUT=180`
- [ ] `MAX_REQUEST_BODY_MB=4`
- [ ] `MAX_FILE_DOWNLOAD_MB=4`
- [ ] `ERROR_LOG_ENABLED=false`
- [ ] `UPDATE_TASK=false`
- [ ] `CountToken=false`

### 正式机器建议值

- [ ] `NODE_TYPE=master`
- [ ] `NODE_NAME=new-api-1`
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

## 第 7 步：绑定域名

在 Dokploy 应用的 Domains 页面：

- [ ] 添加 `idtoken.ai`
- [ ] 启用 HTTPS
- [ ] 启用 Let’s Encrypt
- [ ] 确认 Compose 文件中没有手写 Traefik `labels`

如果证书签发失败：

- [ ] 检查 DNS 是否已生效
- [ ] 检查 80 / 443 端口是否放行
- [ ] 检查 HTTPS 是否仍命中 `TRAEFIK DEFAULT CERT`
- [ ] 如命中默认 Traefik 证书，删除 Compose 中的手写 Traefik `labels` 后重新部署
- [ ] 如果使用 Cloudflare，先尝试切为 `DNS only`

## 第 8 步：部署应用

在 Dokploy 中点击 Deploy：

- [ ] 确认 Compose 文件选择正确
- [ ] 确认 Variables / Secrets 已保存
- [ ] 点击 `Deploy`

部署完成后确认：

- [ ] `new-api` 容器正常运行
- [ ] `postgres` 容器正常运行
- [ ] `redis` 容器正常运行
- [ ] 应用健康检查通过

## 第 9 步：首次访问检查

部署成功后检查：

- [ ] `https://idtoken.ai/api/status`
- [ ] `https://idtoken.ai/`
- [ ] 管理后台页面可正常打开

如果是首次初始化：

- [ ] 打开初始化页面
- [ ] 创建 root 管理员
- [ ] 登录后台

## 第 10 步：后台基础配置

进入后台后建议先做：

- [ ] 添加渠道
- [ ] 添加测试 Token
- [ ] 检查模型列表
- [ ] 关闭不需要的功能
- [ ] 检查站点标题、公告、登录方式等基础设置

如果当前只是文本请求场景：

- [ ] 保持 `UPDATE_TASK=false`
- [ ] 保持 `ERROR_LOG_ENABLED=false`
- [ ] 保持 `CountToken=false`

## 第 11 步：联调验证

建议验证以下链路：

- [ ] 管理员登录
- [ ] 用户 Token 创建
- [ ] `/v1/models`
- [ ] `/v1/chat/completions`
- [ ] SSE 流式响应
- [ ] 渠道切换与转发是否正常

## 第 12 步：测试环境迁移到正式环境

正式机器上线前建议：

- [ ] 正式机器已安装 Dokploy
- [ ] 正式机器已准备好 DNS / 安全组 / HTTPS
- [ ] 正式环境使用 `docker-compose.dokploy.production.yml`
- [ ] 正式机器使用与测试环境一致的 `SESSION_SECRET`
- [ ] 正式机器使用与测试环境一致的 `CRYPTO_SECRET`

迁移顺序：

1. 在正式机器部署同样的应用
2. 导出测试环境 PostgreSQL 数据
3. 导入正式环境 PostgreSQL
4. 在正式环境完成联调
5. 将 `idtoken.ai` 的 `A` 记录切换到正式机器
6. 再次验证 `https://idtoken.ai/api/status`

Redis 一般不需要迁移历史缓存。

## 第 13 步：正式环境扩容预案

当访问量增长后，建议按以下顺序扩容：

1. 先观察 `4 核 8G` 单机是否足够
2. 优先拆分 PostgreSQL / Redis
3. 再增加应用实例
4. 多实例时：
   - [ ] 仅一个节点为 `master`
   - [ ] 其他节点设置为 `slave`
   - [ ] 共用同一套数据库、Redis、`SESSION_SECRET`、`CRYPTO_SECRET`

## 最终检查清单

### 测试环境

- [ ] 使用 `docker-compose.dokploy.test.yml`
- [ ] 使用 `.env.dokploy.example`
- [ ] 域名 `idtoken.ai` 已生效
- [ ] HTTPS 正常
- [ ] 后台可登录
- [ ] `/api/status` 正常
- [ ] `/v1/chat/completions` 正常

### 正式环境

- [ ] 使用 `docker-compose.dokploy.production.yml`
- [ ] 正式机器为 `4 核 8G` 或更高
- [ ] PostgreSQL / Redis 已确认可用
- [ ] 关键密钥与测试环境一致
- [ ] 已完成切流前联调验证
