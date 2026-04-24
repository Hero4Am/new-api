# GHCR + Dokploy 标准发布流程

本文档用于把当前 `new-api` 部署方式收敛为一套稳定的标准流程：

- GitHub Actions 负责构建镜像
- GHCR（GitHub Container Registry）负责存放镜像
- Dokploy 负责拉取固定镜像并部署

这样做的目标是：

- 测试环境和正式环境都跑固定版本
- 正式环境不再依赖服务器上手动 `git pull` / 手动构建
- 可以快速回滚到上一个稳定镜像
- 以后正式环境扩容时，所有机器可拉同一个镜像

执行跟进可配合：

- `docs/installation/GHCR-Dokploy-Execution-Checklist.md`

## 0. 私有镜像约定

由于当前公开包 `ghcr.io/hero4am/new-api` 已经设为 `Public`，GitHub Packages 不能直接再改回 `Private`。

因此从现在开始，后续私有发布统一改为新包：

- `ghcr.io/hero4am/new-api-private`

也就是说：

- 旧的 `ghcr.io/hero4am/new-api` 只作为历史公开包保留
- 新的测试/正式发布都使用 `ghcr.io/hero4am/new-api-private`

## 1. 当前采用的发布策略

仓库已新增两条工作流：

- 测试镜像构建：`.github/workflows/ghcr-build-test.yml`
- 正式镜像提升：`.github/workflows/ghcr-promote-production.yml`

推荐发布节奏：

1. 代码合并到 `main`
2. Actions 自动构建测试镜像并推送到 GHCR
3. 测试环境在 Dokploy 部署 `sha-xxxxxxx` 或 `test`
4. 验收通过后，手动执行 Promote Workflow
5. 将同一个镜像提升为正式版本标签
6. 正式环境在 Dokploy 部署该正式标签

## 2. 镜像标签约定

### 测试环境标签

- `ghcr.io/<GitHubOwner>/<Repo>-private:sha-xxxxxxx`
- `ghcr.io/<GitHubOwner>/<Repo>-private:test`

说明：

- `sha-xxxxxxx` 精确对应某次提交，便于追溯和回滚
- `test` 只是测试环境的快捷标签，不建议正式环境使用

### 正式环境标签

- `ghcr.io/<GitHubOwner>/<Repo>-private:v2026.04.23-1`
- `ghcr.io/<GitHubOwner>/<Repo>-private:production`

说明：

- 正式环境建议优先使用明确版本号
- `production` 可作为当前正式版本的快捷别名
- 不建议正式环境长期使用 `latest`

## 3. Dokploy 中怎么接

本仓库的 Compose 已切换为支持镜像变量：

- `docker-compose.yml:10`
- `docker-compose.dokploy.test.yml:17`
- `docker-compose.dokploy.production.yml:16`

现在镜像写法为：

```yaml
image: ${APP_IMAGE:-calciumion/new-api:latest}
```

也就是说：

- 不配 `APP_IMAGE` 时，仍会回退到官方默认镜像
- 配置 `APP_IMAGE` 后，Dokploy 会优先拉你自己的 GHCR 镜像

## 4. Dokploy 环境变量建议

测试环境：

```env
APP_IMAGE=ghcr.io/<你的 GitHub 用户名或组织>/new-api-private:test
```

或者更稳一点：

```env
APP_IMAGE=ghcr.io/<你的 GitHub 用户名或组织>/new-api-private:sha-abc1234
```

正式环境：

```env
APP_IMAGE=ghcr.io/<你的 GitHub 用户名或组织>/new-api-private:v2026.04.23-1
```

`.env.dokploy.example` 已补充 `APP_IMAGE` 示例。

## 5. GitHub Actions 使用方式

### 5.1 测试镜像构建

工作流：`.github/workflows/ghcr-build-test.yml`

触发方式：

- push 到 `main`
- 手动执行 `workflow_dispatch`

产物：

- `ghcr.io/<owner>/<repo>-private:sha-xxxxxxx`
- `ghcr.io/<owner>/<repo>-private:test`

### 5.2 正式镜像提升

工作流：`.github/workflows/ghcr-promote-production.yml`

触发方式：

- 手动执行 `workflow_dispatch`

输入参数：

- `source_tag`：测试镜像标签，例如 `sha-abc1234`
- `release_tag`：正式版本标签，例如 `v2026.04.23-1`
- `update_production_tag`：是否同步更新 `production`

这个 Promote 流程不会重新构建镜像，而是直接把测试通过的镜像重新打正式标签。

这意味着：

- 测试环境和正式环境可以使用同一个镜像内容
- 正式发布不会因为“重新构建”引入额外差异

## 6. GHCR 权限建议

### GitHub Actions 推镜像

当前工作流使用 `GITHUB_TOKEN` 推送 GHCR，一般不需要额外配置 Docker Hub 凭证。

### Dokploy 拉镜像

如果 GHCR 镜像是公开的：

- Dokploy 直接拉取即可

如果 GHCR 镜像是私有的：

- 需要在 Dokploy 配置 GHCR Registry 凭证
- 建议单独创建一个 GitHub PAT
- 最少授予 `read:packages`

参考文档：

- Dokploy GHCR：<https://docs.dokploy.com/docs/core/registry/ghcr>
- Dokploy Variables：<https://docs.dokploy.com/docs/core/variables>
- Dokploy Docker Compose：<https://docs.dokploy.com/docs/core/docker-compose>
- GitHub Container Registry：<https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry>

## 7. 推荐的日常发布动作

### 发布到测试环境

1. 本地改完代码后提交并 push
2. 等 `ghcr-build-test` 工作流完成
3. 在 Actions Summary 里拿到 `sha-xxxxxxx`
4. 到 Dokploy 测试环境把 `APP_IMAGE` 改成该标签
5. 点击 `Deploy`
6. 验证 `https://idtoken.ai/api/status`

### 发布到正式环境

1. 确认测试环境验收通过
2. 手动运行 `ghcr-promote-production`
3. 输入：
   - `source_tag=sha-xxxxxxx`
   - `release_tag=v2026.04.23-1`
4. 到 Dokploy 正式环境把 `APP_IMAGE` 改成正式标签
5. 点击 `Deploy`

## 8. 回滚方式

如果正式环境发布后需要回滚：

1. 找到上一个正式可用标签，例如 `v2026.04.18-1`
2. 在 Dokploy 正式环境把 `APP_IMAGE` 改回：

```env
APP_IMAGE=ghcr.io/<你的 GitHub 用户名或组织>/new-api-private:v2026.04.18-1
```

3. 重新点击 `Deploy`

这样可以做到秒级切回旧版本，不需要重新构建。

## 9. 当前对你这套环境的建议

结合当前环境，建议这样执行：

- 测试环境继续使用腾讯云 `2核4G`
- 测试环境优先部署 `sha-xxxxxxx`
- 正式环境使用 `4核8G` 起步
- 正式环境只部署正式标签，不部署 `test`
- Dokploy 继续负责域名、HTTPS、环境变量和 Compose 部署

这是你当前阶段最稳、后续也最好扩容的一套做法。
