# 将本地 Git 仓库推送到 GitHub

本文以 Windows 的 Git Bash 为例。命令中的 `用户名`、`仓库名` 和提交说明需要替换为自己的内容。

## 1. 检查和配置 Git

```bash
# 查看 Git 是否已正确安装
git --version

# 配置提交者名称和邮箱；通常只需配置一次
git config --global user.name "你的 GitHub 用户名"
git config --global user.email "你的邮箱"
```

## 2. 创建 GitHub 仓库

登录 GitHub，点击 **New repository** 创建仓库。

- 仓库名建议只使用英文字母、数字、连字符和下划线。
- 如果本地已有 README，不要在新仓库中再次自动创建 README，以免首次推送发生冲突。
- GitHub 仓库必须先存在，修改远端地址不会自动创建仓库。

## 3. 初始化本地仓库

在项目目录中右键选择 **Git Bash Here**：

```bash
# 查看当前目录，避免提交错项目
pwd

# 初始化仓库；已经初始化过可跳过
git init

# 统一将主分支命名为 main
git branch -M main

# 查看当前状态
git status
```

## 4. 配置 `.gitignore`

提交前应排除缓存、日志、安装程序和不需要版本管理的大型结果。例如：

```gitignore
# Python 缓存
__pycache__/
*.pyc

# R 会话文件
.Rhistory
.RData

# 日志和临时文件
*.log
*.tmp

# 安装程序和压缩包
*.exe
*.zip

# 大型分析结果；请根据项目实际情况决定是否忽略
results/
```

检查忽略规则：

```bash
# 显示未提交、未跟踪的文件
git status --short

# 同时显示被忽略的文件
git status --ignored --short

# 解释某个文件为何被忽略
git check-ignore -v 文件路径
```

## 5. 提交本地文件

推荐精确选择要提交的文件：

```bash
# 查看工作区状态
git status

# 添加指定文件或目录
git add README.md script/ plot/ Rscript/

# 核对暂存区中的文件
git diff --cached --name-status

# 创建本地提交
git commit -m "Add project code and documentation"
```

如果确定所有未忽略文件都应提交：

```bash
# 添加所有新增、修改和删除；执行前务必检查 git status
git add -A

# 创建本地提交
git commit -m "Update project files"
```

不要未经检查就执行 `git add -A`，否则可能误提交原始数据、分析结果、密码、密钥或大型安装程序。

## 6. 关联远端仓库

使用 SSH：

```bash
# 添加名为 origin 的远端仓库
git remote add origin git@github.com:用户名/仓库名.git
```

使用 HTTPS：

```bash
# HTTPS 推送需要 Personal Access Token，GitHub 不接受账号密码
git remote add origin https://github.com/用户名/仓库名.git
```

检查远端地址：

```bash
# 显示拉取和推送地址
git remote -v

# 只显示 origin 地址
git remote get-url origin
```

切换到另一个仓库：

```bash
# 修改已有 origin；不会自动创建 GitHub 仓库
git remote set-url origin git@github.com:用户名/新仓库名.git
```

## 7. 配置 SSH 密钥

```bash
# 生成 Ed25519 密钥；邮箱仅作为备注
ssh-keygen -t ed25519 -C "你的 GitHub 邮箱"

# 显示公钥；复制完整的一整行
cat ~/.ssh/id_ed25519.pub
```

公钥格式类似：

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... your_email@example.com
```

注意：`SHA256:xxxx` 是指纹，不是公钥，不能粘贴到 GitHub 的 Key 输入框。

在 GitHub 打开 `Settings -> SSH and GPG keys -> New SSH key`，选择 `Authentication Key`，粘贴完整公钥并保存。然后测试：

```bash
# 测试 GitHub SSH 身份验证
ssh -T git@github.com
```

成功时会显示已通过身份验证，但 GitHub 不提供 Shell 访问；这是正常现象。

## 8. 推送到 GitHub

```bash
# 首次推送 main，并建立上游跟踪关系
git push -u origin main

# 以后已建立上游关系时直接推送
git push
```

## 9. 验证推送结果

```bash
# 查看本地分支与远端分支关系
git status --short --branch

# 获取最新远端信息
git fetch origin

# 查看本地和远端提交哈希
git rev-parse HEAD
git rev-parse origin/main
```

当两个哈希相同，说明当前已提交版本已同步到 GitHub。

`Everything up-to-date` 只表示没有新提交需要推送，不代表本地文件夹中的每个文件都已上传。以下内容不会自动上传：

- 尚未执行 `git add` 和 `git commit` 的修改。
- 未跟踪文件。
- 被 `.gitignore` 忽略的文件。

## 10. 常见错误

### `src refspec master does not match any`

本地分支通常叫 `main`，但命令错误地推送了 `master`：

```bash
# 查看当前分支
git branch

# 当前分支是 main 时应这样推送
git push -u origin main
```

### `remote origin already exists`

```bash
# 查看现有远端
git remote -v

# 修改现有 origin，而不是重复添加
git remote set-url origin git@github.com:用户名/仓库名.git
```

### `Permission denied (publickey)`

GitHub 没有接受当前 SSH 密钥：

```bash
# 查看实际提供了哪把密钥
ssh -vT git@github.com
```

确认 GitHub 中保存的是以 `ssh-ed25519` 开头的完整公钥，而不是 `SHA256` 指纹。

### `Everything up-to-date`

表示没有新提交需要推送。如果文件已经修改但尚未提交：

```bash
# 查看修改
git status

# 添加、提交并推送
git add 文件路径
git commit -m "Describe the changes"
git push
```

### 大文件警告

GitHub 建议普通 Git 文件小于 50 MB，单文件超过 100 MB 会被拒绝。代码仓库通常不应提交安装程序、原始测序数据和大型结果文件。

已经误提交的大文件即使随后删除，仍会保留在 Git 历史中。应使用 `.gitignore` 预防；确需版本管理的大文件可考虑 Git LFS。

## 11. 日常更新流程

```bash
# 1. 确认当前目录和分支
pwd
git branch --show-current

# 2. 查看修改
git status

# 3. 精确添加需要提交的文件
git add 文件1 文件2 目录/

# 4. 核对暂存内容
git diff --cached --name-status

# 5. 创建本地提交
git commit -m "简明描述本次修改"

# 6. 推送到 GitHub
git push

# 7. 确认状态
git status --short --branch
```

核心流程：

```text
修改文件 -> git add -> git commit -> git push
```
