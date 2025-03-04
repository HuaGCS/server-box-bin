# ServerBox AUR 自动更新系统设置指南

本指南详细说明如何设置一个完全自动化的系统，用于监控 ServerBox 的新版本发布并自动更新 AUR 包。

## 1. 准备工作

### 1.1 AUR 账户设置

1. **创建或使用现有 AUR 账户**
   - 访问 https://aur.archlinux.org/ 并注册/登录

2. **生成专用的 SSH 密钥对**
   ```bash
   ssh-keygen -t ed25519 -C "your-email@example.com" -f ~/.ssh/aur_key -N ""
   ```

3. **将公钥添加到 AUR 账户**
   - 复制公钥内容：`cat ~/.ssh/aur_key.pub`
   - 登录 AUR 网站 → "My Account" → 粘贴到 "SSH Public Key"

### 1.2 初始化 AUR 包（如果尚未创建）

1. **创建包仓库**
   ```bash
   ssh aur@aur.archlinux.org setup-repo serverbox-bin
   ```

2. **手动上传初始包**（如果是全新包）
   ```bash
   # 克隆空仓库
   git clone ssh://aur@aur.archlinux.org/serverbox-bin.git
   cd serverbox-bin
   
   # 创建初始文件（可使用之前的脚本生成）
   # 添加 PKGBUILD 和 .SRCINFO
   
   # 提交并推送
   git add .
   git commit -m "Initial commit"
   git push
   ```

## 2. 创建 GitHub 仓库

1. **创建新的 GitHub 仓库**
   - 访问 https://github.com/new
   - 命名为 `serverbox-bin-aur` 或类似名称
   - 设为公开或私有

2. **克隆仓库**
   ```bash
   git clone git@github.com:your-username/serverbox-bin-aur.git
   cd serverbox-bin-aur
   ```

## 3. 准备仓库文件

1. **创建基本目录结构**
   ```bash
   mkdir -p .github/workflows
   ```

2. **添加初始 PKGBUILD**
   - 可以从 AUR 仓库获取当前的 PKGBUILD
   ```bash
   git clone https://aur.archlinux.org/serverbox-bin.git temp
   cp temp/PKGBUILD .
   cp temp/.SRCINFO .
   rm -rf temp
   ```

3. **创建自动化工作流文件**
   ```bash
   # 将前面生成的工作流文件内容保存到这个位置
   nano .github/workflows/auto-update.yml
   ```

4. **添加 README.md**
   - 描述仓库用途和自动化流程

## 4. 配置 GitHub 密钥

1. **添加仓库密钥**
   - 在 GitHub 仓库页面 → "Settings" → "Secrets and variables" → "Actions"
   - 添加以下密钥：
     - `AUR_SSH_PRIVATE_KEY`: 私钥内容 (`cat ~/.ssh/aur_key`)
     - `AUR_USERNAME`: 您的 AUR 用户名或您希望显示的提交者名称
     - `AUR_EMAIL`: 您的电子邮件地址

## 5. 初始提交和测试

1. **提交所有文件**
   ```bash
   git add .
   git commit -m "Set up automatic updates"
   git push
   ```

2. **手动触发工作流测试**
   - GitHub 仓库 → "Actions" → 选择工作流 → "Run workflow"
   - 检查日志确保正常运行

## 6. 工作流程详解

该自动化系统包含以下步骤：

1. **定期检查**：每 12 小时自动检查 ServerBox 的新版本
2. **版本比较**：比较当前 PKGBUILD 中的版本和 GitHub 上的最新版本
3. **下载和校验**：如有新版本，下载并计算校验和
4. **更新 PKGBUILD**：更新版本号、源 URL 和校验和
5. **更新 .SRCINFO**：使用 Docker 生成最新的 .SRCINFO
6. **提交到 GitHub**：将更改提交到您的 GitHub 仓库
7. **推送到 AUR**：将更新后的包文件推送到 AUR

## 7. 自定义和调优

您可以根据需要调整以下参数：

1. **检查频率**：修改 `cron` 表达式（默认每 12 小时）
2. **发布源**：如果 ServerBox 的发布位置变更，更新 API URL
3. **文件模式**：调整 AppImage 文件名匹配模式
4. **通知**：添加失败或成功时的通知（如邮件、Slack）

## 8. 故障排除

1. **工作流失败**
   - 检查 API 访问是否正常
   - 确认 SSH 密钥配置正确
   - 验证包名和路径是否匹配

2. **版本检测问题**
   - 调整版本提取正则表达式
   - 检查 GitHub API 响应格式

3. **AUR 推送错误**
   - 确认 AUR 账户权限
   - 检查是否有冲突更改

## 9. 维护提示

1. **定期查看日志**：确保自动化系统正常运行
2. **手动验证**：偶尔安装包并测试功能
3. **更新工作流**：随着项目变化适时调整工作流

----

完成这些步骤后，您将拥有一个完全自动化的系统，持续监控 ServerBox 的新版本并自动更新 AUR 包。这样，Arch Linux 用户总能及时获取最新版本，而您无需手动干预维护过程。
