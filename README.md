# 多功能加速代理 - Cloudflare Workers

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Cloudflare Workers](https://img.shields.io/badge/Cloudflare-Workers-orange)](https://workers.cloudflare.com/)

一个基于 Cloudflare Workers 的多功能加速代理服务，支持 GitHub、Docker、SourceForge、Google Sheets 等资源的加速访问。

## 🌟 在线演示

- 主页：`https://proxy.xiaonuo.live`
- 项目地址：[https://github.com/iflyelf/proxy-cf](https://github.com/iflyelf/proxy-cf)

## 🚀 功能特性

### ✅ GitHub 加速
- GitHub Raw 文件加速
- GitHub Release 下载加速
- Git Clone 克隆加速
- 完整的 Git 协议支持
- 支持 clone、pull、fetch 等所有 Git 操作
- 完整的 CORS 支持

### ✅ Docker 镜像加速

支持几乎所有主流容器注册表，通过路径前缀访问：

| 注册表 | 路径前缀 | 示例 |
|--------|---------|------|
| Docker Hub | 默认（无前缀） | `docker pull nginx:alpine` |
| GitHub Packages | `/ghcr.io` | `docker pull proxy.xiaonuo.live/ghcr.io/owner/image:tag` |
| Google GCR | `/gcr.io` | `docker pull proxy.xiaonuo.live/gcr.io/project/image:tag` |
| Kubernetes | `/registry.k8s.io` | `docker pull proxy.xiaonuo.live/registry.k8s.io/image:tag` |
| Kubernetes (旧) | `/k8s.gcr.io` | `docker pull proxy.xiaonuo.live/k8s.gcr.io/image:tag` |
| Red Hat Quay | `/quay.io` | `docker pull proxy.xiaonuo.live/quay.io/repo/image:tag` |
| Microsoft MCR | `/mcr.microsoft.com` | `docker pull proxy.xiaonuo.live/mcr.microsoft.com/image:tag` |
| NVIDIA NGC | `/nvcr.io` | `docker pull proxy.xiaonuo.live/nvcr.io/image:tag` |
| AWS ECR | `/public.ecr.aws` | `docker pull proxy.xiaonuo.live/public.ecr.aws/image:tag` |
| Cloudsmith | `/docker.cloudsmith.io` | `docker pull proxy.xiaonuo.live/docker.cloudsmith.io/repo/image:tag` |

**特性：**
- 完整的 Docker Registry API v2 支持
- 自动重写认证头部，避免连接重置
- 支持多架构镜像
- 单域名访问所有注册表（通过路径前缀）

### ✅ 其他功能
- **SourceForge 加速** - 文件下载加速
- **Google Sheets 代理** - 无需 API Key 访问数据
- **直链下载加速** - 任意 HTTP/HTTPS 文件
- **断点续传** - 支持 Range 请求
- **Git 配置** - 通过 URL 重写加速克隆

## 📦 部署

### Cloudflare Dashboard 部署

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 进入 **Workers & Pages** → **Create Application** → **Create Worker**
3. 点击 **Edit Code**，删除默认代码
4. 复制 `worker.js` 的全部内容，粘贴到编辑器
5. 修改第 6 行的域名：
   ```javascript
   domain: 'proxy.xiaonuo.live',  // 改为你的域名
   ```
6. 点击 **Save and Deploy**
7. 配置自定义域名：**Settings** → **Triggers** → **Add Custom Domain**
   - 输入：`proxy.xiaonuo.live`

**注意事项：**
- Worker 代码文件较大（约 54KB），请确保完整复制
- 建议使用 Dashboard 的代码编辑器直接粘贴部署
- 部署后访问自定义域名验证功能是否正常

## 📖 使用说明

### 1. GitHub 加速

支持两种 URL 格式：

#### 方式1：URL 参数格式（推荐，兼容性最好）✅

```bash
# Raw 文件
# 原地址
https://raw.githubusercontent.com/username/repo/main/file.txt

# 加速地址
https://proxy.xiaonuo.live/proxy?url=https://raw.githubusercontent.com/username/repo/main/file.txt

# Release 下载
# 原地址
https://github.com/username/repo/releases/download/v1.0/file.zip

# 加速地址
https://proxy.xiaonuo.live/proxy?url=https://github.com/username/repo/releases/download/v1.0/file.zip
```

#### 方式2：直接拼接格式（某些环境可能被阻止）

```bash
# Raw 文件
https://proxy.xiaonuo.live/https://raw.githubusercontent.com/username/repo/main/file.txt

# Release 下载
https://proxy.xiaonuo.live/https://github.com/username/repo/releases/download/v1.0/file.zip
```

**说明**：由于某些 Cloudflare 路由配置限制，建议优先使用方式1（URL 参数格式）。

### 2. Docker 镜像加速

#### Docker Hub（推荐方式）

编辑 `/etc/docker/daemon.json`：

```json
{
    "registry-mirrors": ["https://proxy.xiaonuo.live"]
}
```

重启 Docker：
```bash
sudo systemctl restart docker
```

然后正常拉取镜像：
```bash
docker pull nginx:alpine
docker pull mysql:8.0
```

#### Containerd 配置（Kubernetes 用户）

**方法1：Containerd v1 配置**

编辑 `/etc/containerd/config.toml`：

```toml
[plugins."io.containerd.grpc.v1.cri".registry]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
      endpoint = ["https://proxy.xiaonuo.live"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."gcr.io"]
      endpoint = ["https://proxy.xiaonuo.live/gcr.io"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."registry.k8s.io"]
      endpoint = ["https://proxy.xiaonuo.live/registry.k8s.io"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."k8s.gcr.io"]
      endpoint = ["https://proxy.xiaonuo.live/k8s.gcr.io"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."quay.io"]
      endpoint = ["https://proxy.xiaonuo.live/quay.io"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."ghcr.io"]
      endpoint = ["https://proxy.xiaonuo.live/ghcr.io"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."mcr.microsoft.com"]
      endpoint = ["https://proxy.xiaonuo.live/mcr.microsoft.com"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."nvcr.io"]
      endpoint = ["https://proxy.xiaonuo.live/nvcr.io"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."public.ecr.aws"]
      endpoint = ["https://proxy.xiaonuo.live/public.ecr.aws"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.cloudsmith.io"]
      endpoint = ["https://proxy.xiaonuo.live/docker.cloudsmith.io"]
```

重启 Containerd：
```bash
sudo systemctl restart containerd
```

**方法2：Containerd v2 配置（推荐）**

1. 在 `/etc/containerd/config.toml` 中启用 certs.d 目录：

```toml
[plugins."io.containerd.cri.v1.images".registry]
  config_path = "/etc/containerd/certs.d"
```

2. 为所有注册表创建配置：

```bash
# Docker Hub
sudo mkdir -p /etc/containerd/certs.d/docker.io
sudo tee /etc/containerd/certs.d/docker.io/hosts.toml << EOF
server = "https://docker.io"

[host."https://proxy.xiaonuo.live"]
  capabilities = ["pull", "resolve"]
  skip_verify = false
EOF

# Google GCR
sudo mkdir -p /etc/containerd/certs.d/gcr.io
sudo tee /etc/containerd/certs.d/gcr.io/hosts.toml << EOF
server = "https://gcr.io"

[host."https://proxy.xiaonuo.live/gcr.io"]
  capabilities = ["pull", "resolve"]
  override_path = true
EOF

# Kubernetes Registry
sudo mkdir -p /etc/containerd/certs.d/registry.k8s.io
sudo tee /etc/containerd/certs.d/registry.k8s.io/hosts.toml << EOF
server = "https://registry.k8s.io"

[host."https://proxy.xiaonuo.live/registry.k8s.io"]
  capabilities = ["pull", "resolve"]
  override_path = true
EOF

# Kubernetes 旧版
sudo mkdir -p /etc/containerd/certs.d/k8s.gcr.io
sudo tee /etc/containerd/certs.d/k8s.gcr.io/hosts.toml << EOF
server = "https://k8s.gcr.io"

[host."https://proxy.xiaonuo.live/k8s.gcr.io"]
  capabilities = ["pull", "resolve"]
  override_path = true
EOF

# Quay.io
sudo mkdir -p /etc/containerd/certs.d/quay.io
sudo tee /etc/containerd/certs.d/quay.io/hosts.toml << EOF
server = "https://quay.io"

[host."https://proxy.xiaonuo.live/quay.io"]
  capabilities = ["pull", "resolve"]
  override_path = true
EOF

# GitHub Packages
sudo mkdir -p /etc/containerd/certs.d/ghcr.io
sudo tee /etc/containerd/certs.d/ghcr.io/hosts.toml << EOF
server = "https://ghcr.io"

[host."https://proxy.xiaonuo.live/ghcr.io"]
  capabilities = ["pull", "resolve"]
  override_path = true
EOF

# Microsoft MCR
sudo mkdir -p /etc/containerd/certs.d/mcr.microsoft.com
sudo tee /etc/containerd/certs.d/mcr.microsoft.com/hosts.toml << EOF
server = "https://mcr.microsoft.com"

[host."https://proxy.xiaonuo.live/mcr.microsoft.com"]
  capabilities = ["pull", "resolve"]
  override_path = true
EOF

# NVIDIA NGC
sudo mkdir -p /etc/containerd/certs.d/nvcr.io
sudo tee /etc/containerd/certs.d/nvcr.io/hosts.toml << EOF
server = "https://nvcr.io"

[host."https://proxy.xiaonuo.live/nvcr.io"]
  capabilities = ["pull", "resolve"]
  override_path = true
EOF

# AWS ECR Public
sudo mkdir -p /etc/containerd/certs.d/public.ecr.aws
sudo tee /etc/containerd/certs.d/public.ecr.aws/hosts.toml << EOF
server = "https://public.ecr.aws"

[host."https://proxy.xiaonuo.live/public.ecr.aws"]
  capabilities = ["pull", "resolve"]
  override_path = true
EOF

# Cloudsmith
sudo mkdir -p /etc/containerd/certs.d/docker.cloudsmith.io
sudo tee /etc/containerd/certs.d/docker.cloudsmith.io/hosts.toml << EOF
server = "https://docker.cloudsmith.io"

[host."https://proxy.xiaonuo.live/docker.cloudsmith.io"]
  capabilities = ["pull", "resolve"]
  override_path = true
EOF
```

3. 重启 Containerd：

```bash
sudo systemctl restart containerd
```

4. 测试（使用 crictl）：

```bash
sudo crictl pull nginx:alpine
sudo crictl pull registry.k8s.io/pause:3.9
```

**配置说明**：
- `server` - 原始注册表地址
- `capabilities` - 支持的操作（pull, resolve, push）
- `override_path` - 保留完整路径（用于路径前缀代理）
- `skip_verify` - 是否跳过 TLS 验证（false 更安全）

#### 其他容器注册表（通过路径前缀）

使用单域名访问所有注册表：

```bash
# GitHub Packages
docker pull proxy.xiaonuo.live/ghcr.io/owner/image:tag

# Google GCR
docker pull proxy.xiaonuo.live/gcr.io/project/image:tag

# Quay.io
docker pull proxy.xiaonuo.live/quay.io/repo/image:tag

# Kubernetes
docker pull proxy.xiaonuo.live/registry.k8s.io/image:tag
docker pull proxy.xiaonuo.live/k8s.gcr.io/image:tag

# Microsoft MCR
docker pull proxy.xiaonuo.live/mcr.microsoft.com/image:tag

# NVIDIA NGC
docker pull proxy.xiaonuo.live/nvcr.io/nvidia/cuda:11.0-base

# AWS ECR
docker pull proxy.xiaonuo.live/public.ecr.aws/image:tag

# Cloudsmith
docker pull proxy.xiaonuo.live/docker.cloudsmith.io/repo/image:tag
```

### 3. SourceForge 加速

```bash
# 原地址
https://sourceforge.net/projects/projectname/files/file.zip

# 方式1：URL 参数格式（推荐）✅
https://proxy.xiaonuo.live/proxy?url=https://sourceforge.net/projects/projectname/files/file.zip

# 方式2：根路径参数
https://proxy.xiaonuo.live/?url=https://sourceforge.net/projects/projectname/files/file.zip

# 方式3：直接拼接
https://proxy.xiaonuo.live/https://sourceforge.net/projects/projectname/files/file.zip
```

### 4. Google Sheets 代理

```bash
# 访问 Google Sheets 数据（JSON 格式）
https://proxy.xiaonuo.live/sheet/YOUR_SHEET_ID

# 示例
https://proxy.xiaonuo.live/sheet/1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms
```

**注意：** Sheet 必须设置为"任何人都可以查看"。

### 5. 直链下载加速

```bash
# 方式1：URL 参数格式（推荐）✅
https://proxy.xiaonuo.live/proxy?url=https://example.com/file.zip

# 方式2：根路径参数
https://proxy.xiaonuo.live/?url=https://example.com/file.zip

# 方式3：直接拼接（某些环境可能被阻止）
https://proxy.xiaonuo.live/https://example.com/file.zip
```

**推荐使用方式1**，兼容性最好，不受路由限制。

### 6. Git Clone 加速

#### 方式1：单次克隆（无需配置）
```bash
# 直接使用代理 URL 克隆
git clone https://proxy.xiaonuo.live/proxy?url=https://github.com/username/repo.git

# 示例
git clone https://proxy.xiaonuo.live/proxy?url=https://github.com/torvalds/linux.git
git clone https://proxy.xiaonuo.live/proxy?url=https://github.com/kubernetes/kubernetes.git
```

#### 方式2：全局配置（推荐 - 所有仓库自动使用）
```bash
# 配置 Git 使用代理
git config --global url."https://proxy.xiaonuo.live/proxy?url=https://github.com/".insteadOf "https://github.com/"

# 验证配置
git config --get-regexp url

# 之后所有 Git 操作自动使用代理
git clone https://github.com/username/repo.git
git pull
git fetch

# 取消配置
git config --global --unset url."https://proxy.xiaonuo.live/proxy?url=https://github.com/".insteadOf
```

#### 方式3：单个仓库配置
```bash
cd your-repo
git config url."https://proxy.xiaonuo.live/proxy?url=https://github.com/".insteadOf "https://github.com/"

# 只在当前仓库生效
git pull
git fetch
```

**功能特点：**
- ✅ 支持 clone、pull、fetch 等所有 Git 操作
- ✅ 完整的 Git 协议支持（/info/refs、/git-upload-pack 等）
- ✅ 支持大型仓库克隆
- ✅ 配置后自动生效，无需修改每个命令

### 7. Git 配置加速（旧方式 - 仅供参考）

### 7. Git 配置加速（旧方式 - 仅供参考）

**注意：** 以下方式可能在某些环境中不可用，推荐使用上面的方式1或方式2。

#### 全局配置（方式1 - 推荐）
```bash
git config --global url."https://proxy.xiaonuo.live/proxy?url=https://github.com/".insteadOf "https://github.com/"
```

#### 全局配置（方式2 - 可能被 Cloudflare 阻止）
```bash
git config --global url."https://proxy.xiaonuo.live/https://github.com/".insteadOf "https://github.com/"
```

#### 针对单个仓库
```bash
cd your-repo
git config url."https://proxy.xiaonuo.live/proxy?url=https://github.com/".insteadOf "https://github.com/"
```

#### 取消配置
```bash
git config --global --unset url."https://proxy.xiaonuo.live/proxy?url=https://github.com/".insteadOf
# 或
git config --global --unset url."https://proxy.xiaonuo.live/https://github.com/".insteadOf
```

## 🔧 配置说明

### 修改域名

编辑 `worker.js` 第 6 行：
```javascript
domain: 'your-domain.com',  // 改为你的域名
```

### 添加或修改容器注册表

编辑 `worker.js` 第 9-20 行的 `dockerRegistries` 对象：
```javascript
dockerRegistries: {
  'docker.io': 'https://your-docker-proxy.com',  // 修改 Docker Hub 上游
  'your-registry.com': 'https://your-registry.com',  // 添加新的注册表
  // ... 其他配置
},
```

## 🐛 故障排查

### Docker 拉取失败

**问题1：unauthorized 错误**

这通常是认证问题。某些容器注册表需要特殊的认证处理：

```bash
# 查看详细错误
docker pull --debug proxy.xiaonuo.live/ghcr.io/owner/image:tag

# 测试注册表连接
curl -v https://proxy.xiaonuo.live/v2/
curl -v https://proxy.xiaonuo.live/v2/ghcr.io/owner/image/manifests/tag
```

**解决方案**：
- GitHub Packages (ghcr.io) 需要 GitHub Token
- 某些私有镜像需要先 `docker login`
- 建议使用公开的镜像进行测试

**问题2：连接被重置**
- ✅ 已解决：使用 docker.1panel.live 作为上游代理
- ✅ 已解决：自动重写认证头部

**问题2：daemon.json 配置不生效**
```bash
# 检查配置
cat /etc/docker/daemon.json

# 检查 JSON 格式
python3 -m json.tool /etc/docker/daemon.json

# 重启 Docker
sudo systemctl restart docker

# 查看日志
sudo journalctl -u docker -f
```

**问题3：路径前缀访问失败**
- 确认路径格式正确：`域名/注册表/镜像名:标签`
- 例如：`proxy.xiaonuo.live/ghcr.io/owner/image:tag`

### GitHub 下载慢

可能原因：
- Cloudflare Workers 到 GitHub 的网络延迟
- GitHub 本身限速
- Workers 免费版请求限额

解决方案：
- 等待后重试
- 升级 Cloudflare Workers 付费版

## 📊 性能与限制

### Cloudflare Workers 限制

| 项目 | 免费版 | 付费版 |
|------|--------|--------|
| 每天请求次数 | 100,000 | 无限制 |
| CPU 时间 | 10ms/请求 | 50ms/请求 |
| 响应大小 | 无限制 | 无限制 |

### 容器注册表认证说明

**公开镜像**（无需认证）：
- ✅ Docker Hub 公开镜像
- ✅ gcr.io 公开镜像
- ✅ registry.k8s.io 公开镜像
- ✅ quay.io 公开镜像
- ✅ mcr.microsoft.com 公开镜像

**需要认证的镜像**：
- ⚠️ GitHub Packages (ghcr.io) - 部分镜像需要 GitHub Token
- ⚠️ 私有仓库 - 需要 docker login

**认证方式**：
```bash
# 如果镜像需要认证，先登录
docker login proxy.xiaonuo.live/ghcr.io
# 然后拉取
docker pull proxy.xiaonuo.live/ghcr.io/owner/private-image:tag
```

**注意**：代理服务会转发认证令牌，但某些注册表可能因为域名不匹配而拒绝认证。建议优先使用公开镜像测试。

### 性能优化建议

1. **使用 CDN 缓存**：Docker 镜像层自动缓存
2. **避免大文件**：超过 100MB 的文件可能较慢
3. **合理使用**：避免短时间大量请求

## 🔒 安全说明

- ✅ 全程 HTTPS 加密
- ✅ 不记录用户日志
- ✅ 不存储任何数据
- ✅ 开源透明
- ✅ CORS 跨域支持

**注意：** 不要使用代理下载敏感数据或传输密码。

## 📄 许可证

MIT License

## 👨‍💻 作者

xiaonuo - https://www.danxiaonuo.me

## 🙏 致谢

- 参考了 gh-proxy.com 的设计理念
- 使用 docker.1panel.live 作为 Docker Hub 上游代理
- 感谢 Cloudflare Workers 提供的优秀平台

## 📞 获取帮助

- **在线帮助**：部署后访问主域名查看使用说明
- **调试信息**：访问 `/debug` 端点
- **项目主页**：https://www.danxiaonuo.me

## ⚡ 快速测试

部署后，运行以下命令测试：

```bash
# 测试主页
curl https://proxy.xiaonuo.live/

# 测试 Docker Hub API
curl https://proxy.xiaonuo.live/v2/

# 测试其他注册表 API
curl https://proxy.xiaonuo.live/v2/ghcr.io/linuxserver/baseimage-alpine/tags/list

# 测试 GitHub Raw 文件加速
curl 'https://proxy.xiaonuo.live/proxy?url=https://raw.githubusercontent.com/kubernetes/kubernetes/master/README.md'

# 测试 GitHub Release 下载
curl -I 'https://proxy.xiaonuo.live/proxy?url=https://github.com/junegunn/fzf/releases/download/v0.54.0/fzf-0.54.0-linux_amd64.tar.gz'

# 测试 Git Clone 加速
git clone https://proxy.xiaonuo.live/proxy?url=https://github.com/octocat/Hello-World.git

# 测试 Google Sheets 代理
curl 'https://proxy.xiaonuo.live/sheet/1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms'

# 查看调试信息
curl https://proxy.xiaonuo.live/debug
```

### 推荐测试的公开镜像

```bash
# Docker Hub（通过 registry-mirrors）
docker pull nginx:alpine
docker pull redis:7-alpine

# Google GCR（公开镜像）
docker pull proxy.xiaonuo.live/gcr.io/google-containers/pause:3.2

# Kubernetes（公开镜像）
docker pull proxy.xiaonuo.live/registry.k8s.io/pause:3.9

# Quay.io（公开镜像）
docker pull proxy.xiaonuo.live/quay.io/prometheus/node-exporter:latest

# Microsoft MCR（公开镜像）
docker pull proxy.xiaonuo.live/mcr.microsoft.com/dotnet/runtime:7.0-alpine
```

**注意**：某些注册表（如 ghcr.io）的部分镜像需要认证，建议先测试公开镜像。

## 🎉 开始使用

1. 复制 `worker.js` 到 Cloudflare Workers
2. 修改域名配置（第 6 行）
3. 部署并配置自定义域名
4. 访问主域名查看使用说明
5. 开始享受加速服务！

---

**如有问题，欢迎反馈！**
