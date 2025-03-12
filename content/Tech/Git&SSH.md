---
creation date: 2024-11-27 16:43
modification date: 2025-03-11 19:46
tags:
  - git
  - github
  - ssh
---

# Git
参考：[Git + GitHub 10分钟完全入门 (进阶)_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1hA411v7qX/?spm_id_from=333.788.recommend_more_video.-1&vd_source=dbefe8621d153f1b1aaef0768a993d25)
[Git工作流和核心原理 | GitHub基本操作 | VS Code里使用Git和关联GitHub_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1r3411F7kn/?spm_id_from=333.788.recommend_more_video.1&vd_source=dbefe8621d153f1b1aaef0768a993d25)

基本操作:
```bash
# 配置基本用户信息
# --global为可选，若不写则表示配置信息只在该仓库生效
git config --global user.name "user_name"
git config --global user.email "user_email"

git init # 初始化一个仓库

git status # 查看仓库状态
git log # 查看git历史及git版本号 注意此时会进入vim模式，使用q进行退出即可


git clone <url> # 克隆远程仓库到本地
git pull # 拉取远程仓库

git add <add-filename> # 将文件修改保存到暂存区
git commit -m "commit-msg" # 提交暂存区中的修改提交到本地仓库
git commit -am "commit-msg" # 相当于上述两个操作的合并

git commit --amend -m "new-commit-msg" #修改上一次提交


# 分支
git branch # 查看所有分支
git branch <branch-name> # 新建一个分支
git branch -d <branch-name> # 删除分支
git branch -m <old-branch-name> <new-branch-name>
git checkout <branch-name> # 切换分支


# 变基
# 需要先切换到对应分支再完成变基
git rebase <target-branch>

# 合并分支
git merge <branch-name> # 将别的分支合并到当前分支

# 版本回退
git revert -n <版本号>

git push  # 将本地仓库推送到远程仓库
git push -u origin <branch-name>
# 本地分支的代码会被上传到远程仓库origin <branch-name>分支上，将本地分支与远程分支建立关联关系，使得以后的 push 和 pull 操作都可以自动匹配对应的远程分支，省去了每次 push 和 pull 时需要手动指定远程分支的操作。(第一次执行会自动创建一个同名的远程分支)

# origin 指代远程仓库，免去输入URL的痛苦
git remote add <repo-name> # 添加远程仓库

git remote <branch-name>  get-url # 获取某分支的远程仓库URL
git remote <branch-name> set-url  # 设置某分支的远程仓库URL

git branch -vv # 查看分支关联远程仓库信息
```
# ssh
参考：[生成新的 SSH 密钥并将其添加到 ssh-agent - GitHub 文档](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
基本操作：
```bash
# 第一步：生成ssh key
ssh-keygen -t ed25519 -C "注释信息"
ssh-keygen -t rsa -C "注释信息"
# 然后一路enter即可
# 第二步：将SSH key添加到ssh-agent（win可能需要进入git bash操作）
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
# 第三步: 将SSH publickey添加到github

# 注意第一次clone会提示记得根据指令输入yes不要回车！
```
Linux ssh免密：
```bash
# 第一步：生成ssh key
ssh-keygen -t ed25519 -C "注释信息"
ssh-keygen -t rsa -C "注释信息"
# 然后一路enter即可
# 第二步：将SSH key的公钥添加到远程机器（windows的Openssh可能无法直接执行该命令，使用git bash即可）
ssh-copy-id -i ~/.ssh/id_rsa.pub user@ip

或者用scp先将公钥传过去再用cat命令将公钥放入authorized_keys文件：
scp ~/.ssh/id_rsa.pub user@ip:~/.ssh
touch authorized_keys
chmod 600 authorized_keys
cat id_rsa.pub >> authorized_keys
```
- 免密码登录的处理是用户对用户的，切换其他用户后，仍然需要输入密码
- 公钥传到远程机器并生效的操作，可用其他方式实现，如scp后修改authorized_keys
- 远程机器的 **.ssh目录需要700权限，authorized_keys文件需要600权限**

## config
config文件可以配置用户名、IP以及端口等信息，方便登录，例如：
原来登录：`ssh temp@172.0.0.1`经配置后可以变为`ssh temp`

```bash
Host <your_shortcut>
  Hostname <your_ip>
  User <your_username>

e.g.:
Host temp
  Hostname 172.0.0.1
  User temp
```

## authorized_keys
管理用户公钥认证的一个关键文件，保存client的公钥以启用ssh免密登录，需要600权限`chmod 600 authorized_keys`

## known_hosts
SSH客户端配置文件，用以**认证远程主机**和**防止中间人攻击**
场景：A通过ssh首次连接到B，B会将公钥1（host key）传递给A，A将公钥1存入known_hosts文件中，以后A再连接B时，B依然会传递给A一个公钥2，OpenSSH会核对公钥，通过对比公钥1与公钥2 是否相同来进行简单的验证，如果公钥不同，OpenSSH会发出警告，避免你受到DNS Hijack之类的攻击。

## Jumpserver 处理
按照如下格式配置好 `host和user` 然后分别将 sk 对应的 pk 追加到 Jumpserver 和 TargetServer 的 `~/.ssh/authorized_keys` 文件中即可
```bash
Host Jumpserver
    HostName <your host>
    User <your user>
    IdentityFile <sk path>

Host TargetServer
	HostName <your host>
	User <your user>
    ProxyJump Jumpserver
	IdentityFile <sk path>
```


# scp
`scp`（secure copy）是一个在Linux和Unix系统中用于在本地和远程计算机之间安全地复制文件的命令行工具。它使用SSH（Secure Shell）协议来保证数据传输的安全性。以下是一些常见的`scp`用法：

### 基本用法
1. **从本地复制到远程**
```bash
scp local_file remote_username@remote_host:remote_directory
```
   - `local_file`：要复制的本地文件路径。
   - `remote_username`：远程主机的用户名。
   - `remote_host`：远程主机的IP地址或主机名。
   - `remote_directory`：远程主机上的目标目录路径。

2. **从远程复制到本地**
```bash
scp remote_username@remote_host:remote_file local_directory
```
   - `remote_file`：要复制的远程文件路径。
   - `local_directory`：本地的目标目录路径。

### 高级用法
3. **复制多个文件**
```bash
scp remote_username@remote_host:remote_file1 remote_username@remote_host:remote_file2 local_directory
```
   可以一次性复制多个文件。

4. **使用绝对路径**
```bash
scp /path/to/local_file remote_username@remote_host:/path/to/remote_directory
```
使用绝对路径可以更精确地指定文件和目录的位置。

5. **递归复制整个目录**

```bash
scp -r local_directory remote_username@remote_host:remote_directory
```
   `-r` 选项用于递归复制整个目录及其内容。
6. **压缩文件后再复制**
```bash
scp -C local_file remote_username@remote_host:remote_directory
```
   `-C` 选项会在传输过程中自动压缩文件，有助于节省带宽。

7. **指定SSH端口**

```bash
scp -P 2222 local_file remote_username@remote_host:remote_directory
```
   `-P` 选项用于指定SSH连接的端口号。

8. **使用SSH密钥认证**
 ```bash
scp -i /path/to/private_key local_file remote_username@remote_host:remote_directory
```
   `-i` 选项用于指定私钥文件的路径，以便进行SSH密钥认证。

9. **查看传输进度**
```bash
scp -v local_file remote_username@remote_host:remote_directory
```
   `-v` 选项会显示详细的传输进度和调试信息。

10. **忽略文件权限和所有权**
```bash
scp -p local_file remote_username@remote_host:remote_directory
```
`-p` 选项会保留文件的权限和所有权。

