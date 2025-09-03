# GitHub Actions 自动发布配置

这个目录包含了用于自动构建和发布cloudflared的GitHub Actions工作流。

## 工作流说明

### 1. 监控上游仓库 (upstream-monitor.yml)

这个工作流会每小时检查一次上游仓库(cloudflare/cloudflared)的最新release：
- 如果发现新的release，会自动在本仓库创建对应的tag
- 触发自动构建和发布工作流

### 2. 自动构建和发布 (auto-release.yml)

这个工作流会在以下情况下触发：
- 当上游仓库发布新的release时（通过upstream-monitor.yml触发）
- 手动触发

工作流会执行以下操作：
1. 构建多种架构的二进制文件
2. 创建Debian和RPM包
3. 发布到GitHub Release

### 3. 同步Fork (sync-fork.yml)

这个工作流会在以下情况下触发：
- 当上游仓库发布新的release时（通过upstream-monitor.yml触发）
- 手动触发

工作流会执行以下操作：
1. 同步fork仓库与上游仓库的主分支
2. 确保fork仓库与上游仓库保持一致

### 4. 使用API同步Fork (sync-fork-api.yml)

这个工作流会在以下情况下触发：
- 当上游仓库发布新的release时（通过upstream-monitor.yml触发）
- 手动触发

工作流会执行以下操作：
1. 使用GitHub API同步fork仓库与上游仓库
2. 这种方法更加可靠，避免了git merge冲突

## 支持的架构

### Linux架构
- amd64 (x86_64)
- 386 (x86)
- arm64
- arm (ARMv5)
- armhf (ARMv7)
- mipsle-softfloat (MIPS little endian softfloat)
- mipsle-hardfloat (MIPS little endian hardfloat)
- mips-softfloat (MIPS big endian softfloat)
- mips-hardfloat (MIPS big endian hardfloat)
- mips64le (MIPS64 little endian)
- mips64 (MIPS64 big endian)
- ppc64le (PowerPC 64-bit little endian)
- s390x (IBM System z)

### Windows架构
- amd64
- 386

### macOS架构
- amd64
- arm64

## 手动触发构建

可以通过GitHub界面手动触发构建：
1. 进入Actions标签页
2. 选择"Auto Release on Upstream Release"工作流
3. 点击"Run workflow"按钮
4. 输入要构建的release tag（例如：2024.8.3）

## 配置说明

### Secrets

工作流需要以下secrets：
- `GITHUB_TOKEN`: GitHub token用于创建release和上传资产

## 故障排除

### 构建失败
如果某个架构的构建失败，可以：
1. 检查构建日志找出失败原因
2. 修复问题后重新触发工作流

### 发布失败
如果发布失败，可以：
1. 手动创建release
2. 重新运行失败的job
