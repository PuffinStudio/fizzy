# GitHub Container Registry (ghcr.io) 使用指南

本文档说明如何为PuffinStudio/fizzy fork配置和使用GitHub Container Registry。

## 🚀 快速开始

### 1. 启用GitHub Actions权限

在GitHub仓库设置中启用Actions权限：

1. 访问: `https://github.com/PuffinStudio/fizzy/settings/actions`
2. 在 **Workflow permissions** 部分选择:
   - ✅ **Read and write permissions**
   - ✅ 勾选 **Allow GitHub Actions to create and approve pull requests**
3. 点击 **Save**

### 2. 推送代码触发自动构建

当你推送代码到 `custom/main` 分支时，GitHub Actions会自动构建并推送镜像到ghcr.io：

```bash
git checkout custom/main
# 做一些修改...
git add .
git commit -m "your changes"
git push origin custom/main
```

### 3. 查看构建状态

访问 Actions 页面查看构建状态：
`https://github.com/PuffinStudio/fizzy/actions/workflows/publish-image.yml`

### 4. 手动触发构建

你也可以手动触发构建：

1. 访问: `https://github.com/PuffinStudio/fizzy/actions/workflows/publish-image.yml`
2. 点击 **Run workflow**
3. 选择 `custom/main` 分支
4. 点击 **Run workflow**

## 📦 镜像信息

### 镜像地址

```
ghcr.io/puffinstudio/fizzy:latest
ghcr.io/puffinstudio/fizzy:custom-main
ghcr.io/puffinstudio/fizzy:sha-<commit-sha>
```

### 镜像标签

- `latest` - 最新稳定版本（基于tag）
- `custom-main` - custom/main分支最新构建
- `sha-<short-sha>` - 特定commit的镜像
- `v*` - 版本标签（如v1.0.0）

## 🔐 认证

### 使用GitHub Token拉取镜像

ghcr.io默认需要认证。使用GitHub Personal Access Token (PAT)或`GITHUB_TOKEN`：

```bash
# 登录ghcr.io
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# 或者使用gh CLI
gh auth login
gh auth token | docker login ghcr.io -u USERNAME --password-stdin
```

### 设置包可见性

1. 访问: `https://github.com/PuffinStudio/fizzy/pkgs/container/fizzy`
2. 点击 **Package settings**
3. 在 **Danger Zone** 中设置可见性:
   - **Public** - 任何人都可以拉取
   - **Private** - 只有授权用户可以拉取（需要PAT）
   - **Internal** - 组织成员可以拉取

## 🐳 使用镜像

### 直接运行

```bash
# 拉取镜像
docker pull ghcr.io/puffinstudio/fizzy:custom-main

# 运行容器
docker run -d \
  -p 3000:3000 \
  -e RAILS_MASTER_KEY=your_master_key \
  -e DATABASE_URL=sqlite3:///data/production.sqlite3 \
  -v fizzy_data:/data \
  --name fizzy \
  ghcr.io/puffinstudio/fizzy:custom-main
```

### 使用Docker Compose

```yaml
version: '3.8'

services:
  fizzy:
    image: ghcr.io/puffinstudio/fizzy:custom-main
    ports:
      - "3000:3000"
    environment:
      RAILS_MASTER_KEY: ${RAILS_MASTER_KEY}
      DATABASE_URL: sqlite3:///data/production.sqlite3
    volumes:
      - fizzy_data:/data
    restart: unless-stopped

volumes:
  fizzy_data:
```

### 使用Kamal部署

```yaml
# config/deploy.yml
service: fizzy
image: ghcr.io/puffinstudio/fizzy
servers:
  web:
    - 192.168.1.100
registry:
  server: ghcr.io
  username: PuffinStudio
  password:
    - KAMAL_REGISTRY_PASSWORD
docker:
  image: ghcr.io/puffinstudio/fizzy:custom-main
```

## 🔧 故障排除

### 1. 镜像拉取失败 (401 Unauthorized)

**问题**: 未登录或token无权限

**解决**:
```bash
# 使用GitHub Token登录
echo $GITHUB_TOKEN | docker login ghcr.io -u PuffinStudio --password-stdin
```

### 2. 构建失败

**问题**: GitHub Actions工作流失败

**解决**:
1. 检查仓库Settings -> Actions权限
2. 查看Actions日志: `https://github.com/PuffinStudio/fizzy/actions`
3. 确认Dockerfile无语法错误

### 3. 包不可见

**问题**: 无法访问 `https://github.com/PuffinStudio/fizzy/pkgs/container/fizzy`

**解决**:
1. 首次推送后，包可能需要几分钟才会显示
2. 检查包的可见性设置（可能需要设置为Public）
3. 确保你有仓库的写权限

## 📋 工作流说明

### 自动触发
- **Push to custom/main**: 自动构建并推送 `custom-main` 和 `sha-<commit>` 标签
- **Push tag (v*)**: 自动构建并推送版本标签和 `latest`

### 手动触发
通过workflow_dispatch手动选择分支构建。

### 架构支持
- ✅ linux/amd64
- ✅ linux/arm64

### 签名
镜像使用Cosign进行keyless签名（OIDC）。

## 🔗 相关链接

- [GitHub Container Registry文档](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [GitHub Actions文档](https://docs.github.com/en/actions)
- [仓库Actions页面](https://github.com/PuffinStudio/fizzy/actions)
- [容器包页面](https://github.com/PuffinStudio/fizzy/pkgs/container/fizzy)
