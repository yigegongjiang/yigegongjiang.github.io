---
title: Git Record
date: 2025-05-13 14:20:11
categories:
  - 技术
tags:
  - 计算机基础
---

> SSH / Personal Access Tokens / GPG Keys / Signing Key / ssh-agent
> 裸中央仓库 / worktree
> ...

<!-- more -->

# Email

通过 ssh key 等方式操作 github 等仓库平台时，只要 ssh 验权通过就可以进行仓库操作。不过对于 git commit 等提交操作，git 会强制要求配置 username 和 email。
不过，email 一定要配置好，和账号的 email 一致。github 虽然不对 email 进行操作验证，但是在显示 verified 等标记的时候，还是会校验 email 的。如果 email 不对，则标记 `Unverified`。

整体来说，虽然不强求，但尽量配置好。

# SSH Key

github 这些 git 平台，除了 https 之外，也支持 git 协议的 ssh 操作。
对于公私钥，本机主要使用私钥，公钥在 github 远程平台上设置。

```
tree ~/.ssh/
/Users/example/.ssh/
├── config
├── id_ed25519_personal
├── id_ed25519_personal.pub
├── id_ed25519_work
├── id_ed25519_work.pub
├── known_hosts
├── company-key
└── company-key.pub
```

当需要在一台主机上管理多个 github 账号的时候，需要在 config 文件中进行如下配置：

```
> cat ~/.ssh/config

Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal

Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work

Host company-github
    HostName github.company.com
    User git
    IdentityFile ~/.ssh/company-key
```

对应的 git clone 地址也需要做改变：

```
git clone git@github.com:octocat/test.git

->

git clone git@github-personal:octocat/test.git
```

## SSH 与 Personal Access Tokens 的关系

> 一般操作，使用 SSH Key 的情况下，不需要设置 PAT。

### 核心区别

- SSH 认证

  - 使用 `git@github.com:user/repo.git` 格式地址
  - 依赖本地 `~/.ssh/` 目录的密钥对
  - 通过 `ssh -T git@github.com` 验证连接

- HTTPS 认证
  - 使用 `https://github.com/user/repo.git` 格式地址
  - 需要配置 PAT 替代密码（GitHub 已禁用密码认证）

### 协议检查方法

```bash
git remote -v
# 显示 git@github.com → 使用 SSH
# 显示 https://github.com → 需要 PAT
```

### 需要 PAT 的场景

1. 调用 GitHub REST API
2. 使用 GitHub CLI (`gh`) 操作敏感资源
3. 访问 GitHub Packages 服务（npm/Docker 等）
4. 操作其他 HTTPS 协议的仓库时
5. 第三方 CI/CD 工具集成
6. 使用 GitHub Actions 需要访问仓库时
7. 使用双重认证(2FA)的账户通过 HTTPS 协议操作仓库

### mac 钥匙串存储 PAT

git 客户端（命令）会在用户输入 PAT 后，将 PAT 存储到 mac 的 keychain 中，并和 github domain 绑定。
这个时候，如果有多个 github 账号，就会发生冲突，导致其中一个 github 账号下面的 pat 验证失败。
所以，需要使用 `https://github.company.com/xxx/xx.git` 或 `https://alias.github.com/xxx/xx.git` 这样的格式，来对不同的 github 账号进行区分。

### URL 使用 PAT

URL 协议本身是支持增加 username 和 password 的，即 `https://username:password@github.com/xxx/xx.git`。

这个时候，可以把 username 换成 github 账号，password 换成 PAT，就可以在通过 github 授权的情况下操作 xx 仓库了。
原理：

1. git 客户端会解析 URL，将 username 和 password 提取成 a:b 的格式，进行 base64 编码，放到 "Authorization: Basic ???" 头中。
2. 服务器端接收到请求后，会进行 base64 解码，获取到 username 和 password，然后进行验证。

> 不推荐，简单实用一下还是可以的。

# Signing Key

github 推出的 commit/tag 等操作的签名认证。被签名的 commit 会在历史记录中显示一个绿色的 `verified` 标记。
GPG Keys 是专门做这个事情的，其中，github 也推出了自己的 Signing Key，可以复用 SSH Keys 的设置（操作入口在同一个地方）。

具体配置如下，通过在 `~/.gitconfig` 中增加 url 分流配置，可以为不同的 host 组设置不同的 signing 规则。

```
// ~/.gitconfig-github-personal

[user]
  signingkey = ~/.ssh/id_ed25519_personal
[commit]
	gpgsign = true
[gpg]
	format = ssh
[tag]
	gpgsign = true
```

```
// ~/.gitconfig-github-work

[user]
  signingkey = ~/.ssh/id_ed25519_work
[commit]
	gpgsign = true
[gpg]
	format = ssh
[tag]
	gpgsign = true
```

```
// ~/.gitconfig

...

[includeIf "hasconfig:remote.*.url:git@github-personal:*/**"]
    path = ~/.gitconfig-github-personal
[includeIf "hasconfig:remote.*.url:git@github-work:*/**"]
    path = ~/.gitconfig-github-work

...
```

# ssh-agent

ssh 和 ssh-agent 都是 openssh 的一部分。对于 ssh 而言，公私钥中，私钥是可以被加密的，加密后，私钥就无法被直接使用，需要使用 passphrase 进行解密后才能使用。
这个时候操作流程就比较复杂，每次使用 ssh 的时候，ssh 命令通过操作系统弹窗，让用户输入 passphrase，然后对私钥原始内容进行解密获取私钥再使用。

ssh-agent 就是一个内存服务，ssh 拿到解密后的私钥后，直接放到 ssh-agent 中，后续直接从 ssh-agent 中进行读取。

这个时候又会遇到一个问题，就是 macos 系统重启了，ssh-agent 内存数据消失了。所以 ssh 会把 passphrase 存储到 keychain 中，下次启动时，会自动从 keychain 中读取 passphrase。
具体流程是：

1. macos 重启，ssh-agent 数据清空
2. 用户操作了 ssh 命令
3. ssh 客户端，通过 secret api 读取 keychain 中的 passphrase
4. 对密钥原始数据进行解密后，把密钥放到 ssh-agent 中
5. 后续直接从 ssh-agent 中读取

配置 ssh-agent 示例(UseKeychain & AddKeysToAgent)：

```bash
> cat ~/.ssh/config

Host example.com
    HostName example.com
    User git
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_ed25519_personal
    UseKeychain yes
    AddKeysToAgent yes
```

# git 如何存储文件的修改

via [https://swiftrocks.com/what-happens-when-you-move-a-file-in-git](https://swiftrocks.com/what-happens-when-you-move-a-file-in-git)

# 裸仓库

【裸仓库】是一个不包含工作区（working tree）的 Git 仓库，只包含 Git 版本库数据（.git 目录中的内容）。在 github 等平台上，我们通过 push、pull 操作的远程仓库，都是【裸仓库】，因为它们只需要存储版本数据，不需要工作区。

裸仓库 一般使用 xx.git 命名文件夹，内部没有工程目录，完全是 git 操作目录，如下：

```bash
tree
.
├── HEAD
├── config
├── description
├── hooks
│   ├── applypatch-msg.sample
│   ├── commit-msg.sample
│   ├── fsmonitor-watchman.sample
│   ├── post-update.sample
│   ├── pre-applypatch.sample
│   ├── pre-commit.sample
│   ├── pre-merge-commit.sample
│   ├── pre-push.sample
│   ├── pre-rebase.sample
│   ├── pre-receive.sample
│   ├── prepare-commit-msg.sample
│   ├── push-to-checkout.sample
│   └── update.sample
├── info
│   └── exclude
├── objects
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags
```

1. 创建 裸仓库：`git init --bare example.git`
2. clone：`git clone ./path/to/example.git example`

# worktree

```bash
NAME
       git-worktree - 管理多个工作目录

SYNOPSIS
       git worktree add [-f] [--detach] [--checkout] [--lock [--reason <string>]]
                        [--orphan] [(-b | -B) <new-branch>] <path> [<commit-ish>]
       git worktree list [-v | --porcelain [-z]]
       git worktree lock [--reason <string>] <worktree>
       git worktree move <worktree> <new-path>
       git worktree prune [-n] [-v] [--expire <expire>]
       git worktree remove [-f] <worktree>
       git worktree repair [<path>...]
       git worktree unlock <worktree>
```

`git worktree` - 管理多个工作目录

核心功能: 允许你从一个 Git 仓库创建多个独立的工作目录，方便同时处理不同分支或任务。

常用命令:

1. `git worktree add <path> [<branch>]` - 创建新的工作目录
   - 作用: 在 `<path>` 创建新的工作目录，并检出 `[<branch>]` 分支 (默认当前分支)。
   - 常用选项:
     - `f`: 强制创建，即使目录已存在 (小心数据丢失)。
     - `-detach`: 创建分离 HEAD 的工作目录 (不关联分支)。
     - `b <new-branch>`: 创建并检出新分支。
   - 示例:
     - `git worktree add -b feature-branch ../feature-branch`: 创建新分支 `feature-branch` 并检出到新工作目录。
     - `git worktree add ../working-dir feature-branch`: 检出已存在的 `feature-branch` 分支到新工作目录。
     - `git worktree add ../hotfix origin/main`: 创建 `hotfix` 工作目录 (基于 `origin/main`)。
2. `git worktree list` - 列出工作目录
   - 作用: 显示所有已创建的工作目录及其路径和分支信息。
   - 常用选项:
     - `v`: 显示更详细的信息。
3. `git worktree remove <path>` - 删除工作目录
   - 作用: 删除 `<path>` 指定的工作目录。
   - 常用选项:
     - `f`: 强制删除，即使工作目录不干净 (小心数据丢失)。
   - 注意: 仅删除工作目录，不影响仓库本身。
4. `git worktree lock <worktree>` - 锁定工作目录
   - 作用: 锁定 `<worktree>`，防止被 `git worktree remove` 删除。
   - 常用选项:
     - `-reason <string>`: 添加锁定原因。
5. `git worktree unlock <worktree>` - 解锁工作目录
   - 作用: 解锁 `<worktree>`，使其可以被 `git worktree remove` 删除。
6. `git worktree move <worktree> <new-path>` - 移动工作目录
   - 作用: 将 `<worktree>` 移动到 `<new-path>`。
7. `git worktree prune` - 清理无效信息
   - 作用: 清理已手动删除的工作目录在 Git 仓库中残留的记录。
   - 常用选项:
     - `n`: 模拟运行，不实际删除。
8. `git worktree repair` - 修复工作目录
   - 作用: 修复工作目录的元数据 (通常自动维护，极少手动使用)。

# 文件夹大小写

Git 对文件夹和文件的大小写，是不敏感的。有些 IDE 会默认做很多事情，让用户无感知。当不通过 IDE 操作 Git 的时候，开发过程中有可能遇到【修改文件夹/文件名】无效的场景。

---
