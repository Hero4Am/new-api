# GHCR + Dokploy 执行清单（按 9 步跟进）

本文档用于跟进当前 `new-api` 从 GitHub Actions 构建镜像，到 Dokploy 测试/正式发布的执行进度。

适用当前环境：

- GitHub Owner：`hero4am`
- 测试域名：`idtoken.ai`
- 测试环境：腾讯云 `2核 4G`
- 正式环境：计划 `4核 8G`
- 私有镜像名：`ghcr.io/hero4am/new-api-private`

当前参考文档：

- 标准流程：`docs/installation/GHCR-Dokploy-Release.md`
- Dokploy 部署说明：`docs/installation/Dokploy.md`

## 当前进度（截至 2026-04-24）

- [x] 第 1 步：代码已推送到 GitHub
- [ ] 第 2 步：确认 GitHub Actions 设置
- [ ] 第 3 步：等待测试镜像 workflow 完成并记录标签
- [ ] 第 4 步：确认 GHCR 包可被 Dokploy 拉取
- [ ] 第 5 步：在 Dokploy 测试环境配置 `APP_IMAGE`
- [ ] 第 6 步：发布测试环境并验证
- [ ] 第 7 步：运行正式镜像 Promote workflow
- [ ] 第 8 步：发布正式环境
- [ ] 第 9 步：记录回滚版本

> 说明：原先公开包 `ghcr.io/hero4am/new-api` 已存在；后续私有发布统一改走 `ghcr.io/hero4am/new-api-private`，因此需要再触发一次新的 workflow。

## 9 步执行清单

### 1. 推送代码到 GitHub

- [x] 本地执行 `git push`
- [x] GitHub `Actions` 页面已出现 `Publish GHCR image (test)`
- [x] 当前提交已触发 workflow

当前提交：

- `2eef2c54` `ops: support configurable app image for Dokploy`
- `e242fde2` `ci: add GHCR test build and promote workflows`
- `dad6dec9` `docs: add GHCR and Dokploy release guide`

### 2. 检查 GitHub Actions 设置

进入：

- `GitHub -> Settings -> Actions -> General`

核对：

- [ ] Actions 已启用
- [ ] `Workflow permissions` 为 `Read and write permissions`
- [ ] 当前仓库允许 Actions 使用 `GITHUB_TOKEN`

备注：

- 如果第 3 步能正常把镜像推到 GHCR，通常说明这里已经基本没问题

### 3. 跑测试镜像 workflow

进入：

- `GitHub -> Actions -> Publish GHCR image (test)`

核对：

- [ ] workflow 最终状态为绿色成功
- [ ] 产出标签 `ghcr.io/hero4am/new-api-private:test`
- [ ] 产出标签 `ghcr.io/hero4am/new-api-private:sha-xxxxxxx`
- [ ] 在 Summary 中记录本次 `sha` 标签

历史公开包（仅记录）：

- `ghcr.io/hero4am/new-api:test`
- `ghcr.io/hero4am/new-api:sha-f52049e`

本次需要生成的新私有包标签：

- `ghcr.io/hero4am/new-api-private:test`
- `ghcr.io/hero4am/new-api-private:sha-xxxxxxx`

建议：

- 正式用于测试环境部署时，优先使用 `new-api-private:sha-xxxxxxx` 这种固定标签

### 4. 确认 GHCR 包可被 Dokploy 拉取

- [ ] 已创建 GitHub PAT
- [ ] PAT 至少包含 `read:packages`
- [ ] Dokploy 已配置 GHCR Registry 凭证
- [ ] 已确认 `new-api-private` 保持 `Private`
- [ ] Dokploy 测试拉取权限已验证通过

备注：

- 旧公开包 `new-api` 不再作为后续部署目标

### 5. 在 Dokploy 测试环境配置 `APP_IMAGE`

进入：

- `Dokploy -> 测试环境应用 -> Environment / Variables`

填写：

- [ ] 已新增或更新 `APP_IMAGE`

推荐值：

```env
APP_IMAGE=ghcr.io/hero4am/new-api-private:sha-xxxxxxx
```

临时快捷值（仅测试）：

```env
APP_IMAGE=ghcr.io/hero4am/new-api-private:test
```

### 6. 发布测试环境并验证

进入：

- `Dokploy -> 测试环境应用`

执行：

- [ ] 点击 `Save`
- [ ] 点击 `Deploy`

验证：

- [ ] `https://idtoken.ai/api/status` 返回 `200`
- [ ] 页面/后台可正常访问
- [ ] 关键功能冒烟通过

可记录：

- 测试环境实际发布标签：`________________`
- 发布时间：`________________`
- 验证人：`________________`

### 7. 运行正式镜像 Promote workflow

进入：

- `GitHub -> Actions -> Promote GHCR image (production)`

填写：

- [ ] `source_tag=sha-xxxxxxx`
- [ ] `release_tag=v2026.04.24-1`
- [ ] `update_production_tag=true`

核对：

- [ ] workflow 最终状态为绿色成功
- [ ] 正式标签已生成
- [ ] `production` 标签已更新（如果启用）

### 8. 发布正式环境

进入：

- `Dokploy -> 正式环境应用 -> Environment / Variables`

填写：

- [ ] `APP_IMAGE=ghcr.io/hero4am/new-api-private:v2026.04.24-1`

执行：

- [ ] 点击 `Save`
- [ ] 点击 `Deploy`

验证：

- [ ] 正式域名健康检查通过
- [ ] 管理后台正常
- [ ] 关键 API 正常

可记录：

- 正式环境发布标签：`________________`
- 发布时间：`________________`
- 验证人：`________________`

### 9. 记录回滚版本

上线完成后记录：

- [ ] 当前正式版本已记录
- [ ] 上一个稳定版本已记录
- [ ] 回滚标签已确认可用

建议至少保留：

- 当前正式版本：`________________`
- 上一个稳定版本：`________________`
- 测试通过版本：`ghcr.io/hero4am/new-api-private:sha-xxxxxxx`

## 建议的实际跟进顺序

今天建议按这个顺序推进：

1. 先等第 3 步 workflow 最终完成
2. 然后立即做第 4 步和第 5 步
3. 再做第 6 步，完成测试环境切换
4. 测试通过后，再做第 7、8、9 步

这样最稳，也最不容易混乱。
