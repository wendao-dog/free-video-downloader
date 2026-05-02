# GitHub 双账号推送清单（wendao-dog / wendao-coder）

## 1. 目标

在同一台 Windows 机器上同时使用两个 GitHub 账号，并做到：

- 不互相覆盖登录状态
- 每个仓库固定使用指定账号
- 日常推送时不需要反复清理凭据

---

## 2. 一次性全局设置（只做一次）

```bash
git config --global credential.helper manager-core
```

说明：

- 使用 Git Credential Manager 保存凭据
- 后续可以并存多个账号

---

## 3. 仓库级绑定原则（关键）

每个项目目录单独设置以下 3 项：

1. `remote.origin.url`：远程地址里带账号名
2. `credential.username`：当前仓库默认用户名
3. `credential.useHttpPath=true`：按路径区分凭据，避免串号

---

## 4. 绑定到 wendao-dog（示例）

在目标项目目录执行：

```bash
git remote set-url origin https://wendao-dog@github.com/wendao-dog/仓库名.git
git config --local credential.username wendao-dog
git config --local credential.useHttpPath true
git push -u origin main
```

首次推送可能会弹浏览器认证，登录 wendao-dog 即可。

---

## 5. 绑定到 wendao-coder（示例）

在另一个项目目录执行：

```bash
git remote set-url origin https://wendao-coder@github.com/wendao-coder/仓库名.git
git config --local credential.username wendao-coder
git config --local credential.useHttpPath true
git push -u origin main
```

---

## 6. 日常最短提交流程

```bash
git add .
git commit -m "你的提交说明"
git push
```

---

## 7. 快速自检命令（建议每次切仓库后执行）

```bash
git rev-parse --show-toplevel
git remote -v
git config --local -l
```

检查点：

- 当前目录确实是目标项目仓库
- `origin` 指向了正确账号和仓库
- 本地凭据配置与仓库账号一致

---

## 8. 常见报错与处理

### 8.1 `remote origin already exists`

含义：已存在 origin，不能重复 `add`。

处理：

```bash
git remote set-url origin https://账号@github.com/组织或账号/仓库.git
```

### 8.2 `Permission denied ... 403`

含义：当前认证账号对目标仓库无写权限。

处理顺序：

1. 检查远程 URL 是否是目标账号
2. 检查目标账号是否有仓库写权限（owner/collaborator）
3. 必要时清理 github 凭据并重新登录正确账号

清理凭据（仅排障时使用）：

```bash
cmdkey /delete:git:https://github.com
```

### 8.3 `Failed to connect to github.com port 443`

含义：当前网络到 `github.com:443` 不通，HTTPS 推送会失败。

先验证连通性：

```bash
Test-NetConnection github.com -Port 443
Test-NetConnection ssh.github.com -Port 443
```

若 `github.com:443` 不通但 `ssh.github.com:443` 可通，建议切到 SSH over 443：

```bash
git remote set-url origin ssh://git@ssh.github.com:443/账号/仓库.git
```

### 8.4 `Permission denied (publickey)`（切 SSH 后）

含义：SSH key 还没有加到对应 GitHub 账号。

处理步骤：

1. 生成专用密钥（示例：wendao-dog）

```bash
ssh-keygen -t ed25519 -C "wendao-dog@users.noreply.github.com" -f "$HOME/.ssh/id_ed25519_wendao_dog"
```

2. 查看公钥并复制

```bash
Get-Content "$HOME/.ssh/id_ed25519_wendao_dog.pub"
```

3. 打开 GitHub → Settings → SSH and GPG keys → New SSH key，粘贴并保存

4. 当前仓库绑定该 key（避免双账号串用）

```bash
git config --local core.sshCommand "ssh -i C:/Users/你的用户名/.ssh/id_ed25519_wendao_dog -o IdentitiesOnly=yes"
```

注意：Windows 下这里建议使用 `C:/...` 这种正斜杠路径，避免私钥路径解析失败。

5. 测试与推送

```bash
ssh -T -i C:/Users/你的用户名/.ssh/id_ed25519_wendao_dog -o IdentitiesOnly=yes -p 443 git@ssh.github.com
git push -u origin main
```

---

## 9. 可直接复制模板

### 模板 A：推送到 wendao-dog

```bash
git remote set-url origin https://wendao-dog@github.com/wendao-dog/<repo>.git
git config --local credential.username wendao-dog
git config --local credential.useHttpPath true
git push -u origin main
```

### 模板 B：推送到 wendao-coder

```bash
git remote set-url origin https://wendao-coder@github.com/wendao-coder/<repo>.git
git config --local credential.username wendao-coder
git config --local credential.useHttpPath true
git push -u origin main
```

---

## 10. 当前项目结论（free-video-downloader）

该项目已成功推送到：

- `ssh://git@ssh.github.com:443/wendao-dog/free-video-downloader.git`
- 分支：`main`
