# 部署指南

## 📦 部署到 Cloudflare Workers

### 1. 准备工作

确保你有：
- Cloudflare 账户
- 已创建的 Worker（如 `proxy`）
- 已绑定自定义域名（如 `proxy.xiaonuo.live`）

### 2. 更新配置

编辑 `worker.js` 第 6 行，修改为你的域名：

```javascript
domain: 'proxy.xiaonuo.live',  // 修改为你的域名
```

### 3. 部署步骤

#### 方法1：Cloudflare Dashboard（推荐）

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 进入 `Workers & Pages`
3. 选择你的 Worker（如 `proxy`）
4. 点击 `Edit code`
5. **全选并删除**现有代码
6. 复制 `worker.js` 的**完整内容**
7. 粘贴到编辑器中
8. 点击 `Save and Deploy`

#### 方法2：Wrangler CLI

```bash
# 安装 Wrangler
npm install -g wrangler

# 登录 Cloudflare
wrangler login

# 部署
cd /xiaonuo/workspace/tools/proxy-cf
wrangler deploy worker.js --name proxy --compatibility-date 2024-01-01
```

### 4. 验证部署

部署完成后，测试以下功能：

```bash
# 测试主页
curl https://proxy.xiaonuo.live/

# 测试 GitHub 加速
curl -I https://proxy.xiaonuo.live/https://raw.githubusercontent.com/kubernetes/kubernetes/master/README.md

# 测试 Docker API
curl https://proxy.xiaonuo.live/v2/

# 测试调试端点
curl https://proxy.xiaonuo.live/debug
```

**预期结果**：
- 主页返回 HTML
- GitHub 加速返回 200 或 302
- Docker API 返回 401 或空 JSON
- 调试端点返回 JSON 配置信息

### 5. 常见问题

#### Q1: GitHub 加速返回 404

**原因**：Worker 代码未正确部署或使用了旧版本

**解决**：
1. 确认你复制了**完整的** `worker.js` 内容（1528 行）
2. 在 Dashboard 中**全选删除**旧代码后再粘贴
3. 确保点击了 `Save and Deploy`
4. 等待 1-2 分钟让全球边缘节点同步
5. 清除浏览器缓存或使用无痕模式测试

#### Q2: 域名配置错误

**解决**：检查第 6 行的域名配置是否与你绑定的自定义域名一致

#### Q3: Docker 拉取失败

**解决**：
1. 检查 `/etc/docker/daemon.json` 配置
2. 重启 Docker：`sudo systemctl restart docker`
3. 测试基本连接：`curl https://proxy.xiaonuo.live/v2/`

### 6. 更新检查清单

部署前确认：
- [ ] 已修改第 6 行域名配置
- [ ] 复制了完整的 worker.js 内容
- [ ] 删除了 Dashboard 中的旧代码
- [ ] 点击了 Save and Deploy
- [ ] 等待边缘节点同步（1-2分钟）
- [ ] 测试 GitHub 加速功能
- [ ] 测试 Docker 镜像拉取

### 7. 文件信息

当前版本：
- `worker.js`: 51KB, 1528 行
- `README.md`: 16KB, 570 行
- 最后更新：2024-10-15

### 8. 技术支持

- GitHub 仓库：https://github.com/iflyelf/proxy-cf
- 提交 Issue：https://github.com/iflyelf/proxy-cf/issues
- 在线演示：https://proxy.xiaonuo.live
