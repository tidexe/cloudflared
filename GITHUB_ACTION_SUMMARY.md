# GitHub Actions 配置总结

本仓库已配置了完整的GitHub Actions工作流，用于自动监控上游仓库、同步fork、构建和发布release。

## 已创建的工作流文件

### 1. 监控上游仓库 (upstream-monitor.yml)
- 位置: `.github/workflows/upstream-monitor.yml`
- 功能: 每小时检查上游仓库的最新release
- 触发: 定时任务(每小时)或手动触发
- 操作: 
  - 检查上游仓库的最新release tag
  - 如果发现新的release，自动在本仓库创建对应的tag
  - 触发repository_dispatch事件通知其他工作流

### 2. 自动构建和发布 (auto-release.yml)
- 位置: `.github/workflows/auto-release.yml`
- 功能: 当上游仓库发布新的release时自动构建和发布
- 触发: repository_dispatch事件或手动触发
- 操作:
  - 构建多种架构的二进制文件(包括MIPS架构的softfloat和hardfloat变体)
  - 创建Debian和RPM包
  - 发布到GitHub Release

### 3. 同步Fork (sync-fork.yml)
- 位置: `.github/workflows/sync-fork.yml`
- 功能: 同步fork仓库与上游仓库
- 触发: repository_dispatch事件或手动触发
- 操作:
  - 使用git命令同步fork仓库与上游仓库的主分支
  - 确保fork仓库与上游仓库保持一致

### 4. 使用API同步Fork (sync-fork-api.yml)
- 位置: `.github/workflows/sync-fork-api.yml`
- 功能: 使用GitHub API同步fork仓库与上游仓库
- 触发: repository_dispatch事件或手动触发
- 操作:
  - 使用GitHub API的merge-upstream端点同步fork
  - 在fork仓库中创建与上游仓库相同的tag

### 5. 推送时构建 (build-on-push.yml)
- 位置: `.github/workflows/build-on-push.yml`
- 功能: 当本仓库有更新时触发自动构建
- 触发: 推送到master分支或手动触发
- 操作:
  - 检查是否有新的tag
  - 构建多种架构的二进制文件(包括MIPS架构)
  - 创建Debian和RPM包
  - 发布到GitHub Release

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

## 修改的文件

### Makefile
- 添加了对MIPS softfloat和hardfloat架构的支持
- 修改了PACKAGE_ARCH变量的设置逻辑

### build-packages.sh
- 添加了对MIPS架构的支持
- 增加了对mipsle-softfloat、mipsle-hardfloat、mips-softfloat、mips-hardfloat、mips64le和mips64架构的处理

### release_pkgs.py
- 更新了默认架构列表，添加了MIPS架构支持

### GITHUB_ACTION_README.md
- 更新了文档，添加了对新增MIPS架构的说明

## 使用说明

### 自动工作流
1. `upstream-monitor.yml`每小时自动检查上游仓库的release
2. 如果发现新的release，会自动触发同步和构建工作流
3. 构建完成后会自动发布到GitHub Release

### 手动触发
可以通过GitHub界面手动触发任何工作流：
1. 进入Actions标签页
2. 选择相应的工作流
3. 点击"Run workflow"按钮
4. 根据需要填写参数(如release tag)

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
