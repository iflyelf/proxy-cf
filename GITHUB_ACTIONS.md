# GitHub Actions 自动部署配置指南

本项目使用 GitHub Actions 自动部署到 Cloudflare Workers。

## 🔧 配置步骤

### 1. 获取 Cloudflare API Token

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 点击右上角头像 → `My Profile` → `API Tokens`
3. 点击 `Create Token`
4. 选择 `Edit Cloudflare Workers` 模板
5. 配置权限：
   - **Account** → `Workers Scripts` → `Edit`
6. 点击 `Continue to summary` → `Create Token`
7. **复制并保存 Token**（只显示一次）

### 2. 获取 Cloudflare Account ID

1. 在 [Cloudflare Dashboard](https://dash.cloudflare.com/) 首页
2. 选择任意一个域名
3. 在右侧栏找到 `Account ID`
4. 点击复制

### 3. 配置 GitHub Secrets

1. 进入 GitHub 仓库：https://github.com/iflyelf/proxy-cf
2. 点击 `Settings` → `Secrets and variables` → `Actions`
3. 点击 `New repository secret`，添加以下 Secrets：

| Secret 名称 | 说明 | 示例 |
|------------|------|------|
| `CLOUDFLARE_API_TOKEN` | Cloudflare API Token | `abcd1234...` |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare Account ID | `1234567890abcdef` |
| `WORKER_URL` | Worker 访问地址（可选） | `https://proxy.xiaonuo.live` |

### 4. 修改 Worker 配置（可选）

如果需要更改 Worker 名称，编辑 `.github/workflows/deploy.yml`：

```yaml
wrangler deploy worker.js \
  --name proxy \  # 改为你的 Worker 名称
  --compatibility-date 2024-01-01
```

## 🚀 部署方式

### 方式 1：自动部署（推荐）

当你推送代码到 `main` 分支时，GitHub Actions 会自动部署：

```bash
git add -A
git commit -m "更新代码"
git push origin main
```

触发条件：
- 修改 `worker.js`
- 修改 `wrangler.toml`
- 修改 `.github/workflows/deploy.yml`

### 方式 2：手动触发

1. 进入 GitHub 仓库
2. 点击 `Actions` 标签
3. 选择 `部署到 Cloudflare Workers`
4. 点击 `Run workflow`
5. 选择部署环境（production/staging）
6. 点击 `Run workflow`

## 📊 部署验证

部署完成后，GitHub Actions 会自动验证以下功能：

- ✅ 主页访问
- ✅ 版本信息
- ✅ GitHub 加速功能
- ✅ Docker Registry API
- ✅ 路由配置

查看部署日志：
1. 进入 GitHub 仓库
2. 点击 `Actions` 标签
3. 选择最新的 workflow 运行记录
4. 查看详细日志

## 🔍 故障排查

### 问题 1：部署失败 - API Token 错误

**错误信息**：
```
Authentication error: Invalid API Token
```

**解决方案**：
1. 检查 `CLOUDFLARE_API_TOKEN` 是否正确
2. 确认 Token 有 `Workers Scripts:Edit` 权限
3. 重新创建 Token 并更新 GitHub Secret

### 问题 2：部署失败 - Account ID 错误

**错误信息**：
```
Account ID is required
```

**解决方案**：
1. 检查 `CLOUDFLARE_ACCOUNT_ID` 是否配置
2. 确认 Account ID 格式正确（16位十六进制）

### 问题 3：部署成功但功能异常

**解决方案**：
1. 等待 2-3 分钟让边缘节点同步
2. 清除浏览器缓存
3. 使用 `curl` 测试：
   ```bash
   curl -s https://proxy.xiaonuo.live/debug | jq
   ```

### 问题 4：GitHub Actions 权限不足

**解决方案**：
1. 进入仓库 `Settings` → `Actions` → `General`
2. 确保 `Workflow permissions` 设置为 `Read and write permissions`
3. 勾选 `Allow GitHub Actions to create and approve pull requests`

## 📝 手动部署（备用方案）

如果 GitHub Actions 无法使用，可以手动部署：

### 方法 1：使用 Wrangler CLI

```bash
# 安装 Wrangler
npm install -g wrangler

# 登录
wrangler login

# 部署
cd /xiaonuo/workspace/tools/proxy-cf
wrangler deploy worker.js \
  --name proxy \
  --compatibility-date 2024-01-01
```

### 方法 2：使用 Cloudflare Dashboard

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 进入 `Workers & Pages`
3. 选择你的 Worker（如 `proxy`）
4. 点击 `Edit code`
5. 复制 `worker.js` 完整内容（52KB，1552行）
6. 粘贴并点击 `Save and Deploy`

## 🔗 相关链接

- GitHub 仓库：https://github.com/iflyelf/proxy-cf
- Cloudflare Workers 文档：https://developers.cloudflare.com/workers/
- Wrangler 文档：https://developers.cloudflare.com/workers/wrangler/
- Actions 运行记录：https://github.com/iflyelf/proxy-cf/actions

## 📞 技术支持

如有问题，请提交 Issue：https://github.com/iflyelf/proxy-cf/issues
