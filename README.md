# 🚀 阿里云镜像同步工具（Aliyun Docker Mirror Sync）

将国外或私有的 Docker 镜像一键同步到阿里云镜像仓库（支持多架构、私有仓库、批量操作）。本仓库灵感来源于 docker_images_pusher 仓库（repect）。

---

## ✨ 功能简介

✅ 一键同步多个镜像到阿里云  
✅ 支持 DockerHub 公共 / 私有镜像  
✅ 自动处理多架构（amd64 / arm64）  
✅ 自动打标签防止命名冲突  
✅ 支持 workflow 手动触发 / 自动运行  

---

## 🧩 仓库结构

.
├── .github/
│   └── workflows/
│       └── docker-sync.yml   # 主工作流文件（GitHub Actions 自动执行）
└── README.md                 # 使用说明文档

---

## ⚙️ 环境准备

### 1. 配置 GitHub Secrets

在仓库中添加以下 Secrets（路径：`Settings → Secrets and variables → Actions`）：

| 名称 | 说明 | 示例 |
|------|------|------|
| `ALIYUN_REGISTRY` | 阿里云镜像仓库地址 | `registry.xxxx.aliyuncs.com` |
| `ALIYUN_NAME_SPACE` | 阿里云命名空间 | `my-namespace` |
| `ALIYUN_REGISTRY_USER` | 阿里云仓库用户名 | `your_aliyun_user` |
| `ALIYUN_REGISTRY_PASSWORD` | 阿里云密码或 Token | `your_password` |
| `DOCKERHUB_USER` | （可选）DockerHub 用户名 | `your_dockerhub_user` |
| `DOCKERHUB_TOKEN` | （可选）DockerHub Access Token | `ghp_xxxxx` |

> 💡 如果你不配置 `DOCKERHUB_USER` / `DOCKERHUB_TOKEN`，将只能拉取公共镜像。

---

## 🚀 使用方法

### ✅ 手动触发同步

1. 打开仓库页面  
2. 点击上方 **Actions** → 选择 **镜像同步到阿里云** → 右上角 “Run workflow”  
3. 输入要同步的镜像（支持多个，用逗号分隔）：nginx,redis:7
4. 点击 “Run workflow”

---

### 🧠 自动同步（push main 自动触发）

当你推送到 `main` 分支时，GitHub Actions 也会自动执行同步任务（可关闭）。

---

## 🔧 工作流说明（docker-sync.yml）

| 步骤 | 功能 |
|------|------|
| 🧹 查看初始磁盘空间 | 打印当前环境磁盘情况 |
| 💾 释放构建空间 | 自动清理 CI 环境空间（最大化可用磁盘） |
| 🔁 重启 Docker 服务 | 重新加载 Docker 守护进程 |
| 📦 检出代码仓库 | 获取当前仓库代码 |
| 🧰 安装 Buildx 环境 | 启用高级 Docker 构建支持 |
| 🔐 登录阿里云镜像仓库 | 登录目标 registry |
| 🔐 登录 Docker Hub | 登录源 registry（可拉私有镜像） |
| 🚀 开始同步镜像到阿里云 | 循环拉取 → 重标记 → 推送 |
| 🧹 清理镜像释放空间 | 删除临时镜像，防止磁盘爆满 |

---

## 🧩 镜像命名规则

同步后推送到阿里云的镜像名格式为：

<阿里云仓库>/<命名空间>/<架构><源命名空间><镜像名>:

例如：

| 源镜像 | 目标镜像 |
|--------|-----------|
| `nginx:latest` | `registry.cn-hangzhou.aliyuncs.com/myspace/amd64_library_nginx:latest` |
| `byquan2004/h5-webui:latest` | `registry.cn-hangzhou.aliyuncs.com/myspace/amd64_byquan2004_h5-webui:latest` |

---

## ⚠️ 注意事项

1. 每次执行会删除临时镜像，请勿用于生产机环境。
2. 私有镜像必须提供有效的 DockerHub 登录凭据。
3. 默认只同步最新版本 tag（如果未指定 tag，会自动加上 `:latest`）。
4. 建议在阿里云仓库命名空间下提前创建对应的仓库（可自动创建也行）。

