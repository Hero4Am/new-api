# GHCR + Dokploy 执行清单（按 9 步跟进）

本文档用于跟进当前 `new-api` 从 GitHub Actions 构建镜像，到 Dokploy 测试/正式发布的执行进度。

适用当前环境：

- GitHub Owner：`hero4am`
- 测试域名：`idtoken.ai`
- 测试环境：腾讯云 `2核 4G`
- 正式环境：计划 `4核 8G`

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

> 说明：上面先按保守方式记录。即使当前测试镜像大概率已经推送成功，也建议等 GitHub Actions 页面最终变绿后再勾选第 3 步。

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
- [ ] 产出标签 `ghcr.io/hero4am/new-api:test`
- [ ] 产出标签 `ghcr.io/hero4am/new-api:sha-xxxxxxx`
- [ ] 在 Summary 中记录本次 `sha` 标签

当前观察到的标签：

- `ghcr.io/hero4am/new-api:test`
- `ghcr.io/hero4am/new-api:sha-dad6dec`

建议：

- 正式用于测试环境部署时，优先使用 `sha-dad6dec` 这种固定标签

### 4. 确认 GHCR 包可被 Dokploy 拉取

如果 GHCR 包准备设为公开：

- [ ] 在 GitHub `Packages` 页面确认容器包存在
- [ ] 包可见性已设为 `public`

如果 GHCR 包准备设为私有：

- [ ] 已创建 GitHub PAT
- [ ] PAT 至少包含 `read:packages`
- [ ] Dokploy 已配置 GHCR Registry 凭证

当前决策：

- [ ] 公开拉取
- [ ] 私有拉取

备注：

- 二选一即可，不需要同时做

### 5. 在 Dokploy 测试环境配置 `APP_IMAGE`

进入：

- `Dokploy -> 测试环境应用 -> Environment / Variables`

填写：

- [ ] 已新增或更新 `APP_IMAGE`

推荐值：

```env
APP_IMAGE=ghcr.io/hero4am/new-api:sha-dad6dec
```

临时快捷值（不建议长期依赖）：

```env
APP_IMAGE=ghcr.io/hero4am/new-api:test
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

- [ ] `source_tag=sha-dad6dec`
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

- [ ] `APP_IMAGE=ghcr.io/hero4am/new-api:v2026.04.24-1`

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
- 测试通过版本：`ghcr.io/hero4am/new-api:sha-dad6dec`

## 建议的实际跟进顺序

今天建议按这个顺序推进：

1. 先等第 3 步 workflow 最终完成
2. 然后立即做第 4 步和第 5 步
3. 再做第 6 步，完成测试环境切换
4. 测试通过后，再做第 7、8、9 步

这样最稳，也最不容易混乱。
