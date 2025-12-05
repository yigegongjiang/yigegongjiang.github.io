---
title: Flash Open Terminal
date: 2025-11-13 10:19:43
categories:
  - 技术
tags:
  - Work
---

问题(TLDR)：zsh 已经成为标配，但每次启动它都需要等待 1-2s，很烦。
问题(Detail)：`.zshrc` 配置中有很多耗时插件和功能，如 nvm、jenv 等等，打开终端很慢。所有 lazy load 等方案都会在各个旮旯里影响原有的启动逻辑，且维护成本高。

解决方案：ITerm2 & Tmux。

<!-- more -->

### 场景还原：

1. `.zshrc` 有很多功能需求配置，已经影响到了 terminal 启动速度。
2. 工作期间需要开启 n 个 app 画面，非常乱。而对于 ternimal 总是习惯性使用完 kill。每次打开都会 waiting 很久。
3. 终端使用 Warp 和 Iterm2，app launch 使用 Raycast。

### 解决方案：

1. 使用 iterm2 创建一个 profile，设置为 default(即 app 启动后默认打开当前 profile）。

2. General - Command 选择 `Command`，命令配置：

   ```bash
   /usr/bin/env zsh -c '/opt/homebrew/bin/tmux new -As dev'
   ```

3. tmux 使用较简单，了解下就可以熟练操作。
4. 后续工作中需要快速执行命令，通过 Raycast 打开 iterm2 即可，0ms 延迟启动。
5. 使用完可直接 kill iterm2 窗口，依靠 tmux 做会话保持。
6. 其他场景，按个人喜好使用 Warp 即可。

### 结果：

Iterm2 自身启动耗时 0ms，tmux attach sesstion 0ms。 配合一下，终端秒开，快速执行工作生活中的小命令，再也不用处理 `.zshrc` 多配置启动慢问题。

### 扩展：

#### shell 启动参数

- `-l`：以登录 shell 启动，读取登录相关配置（如 `~/.zprofile`），保证 PATH、环境变量齐全。
- `-i`：以交互式 shell 启动，启用提示符、job control，并读取交互配置（如 `~/.zshrc`）。
- `-c`：执行后面的一条命令字符串并退出，常与 `-l`、`-i` 组合，用来在「完整环境」里执行单条命令。

配置了 `-lic`，就会走完整的模拟器登录交互环境，保证各种环境变量、插件都能正常加载。这也是文档开头说到的问题根源。即 `.zshrc` 等配置启动太耗时了。
配置 `-c`，就在裸环境执行 shell 命令，非常快。但这种场景下是裸环境，各种 shell 配置都没有。只应该在必要的场景使用。

#### 更多 Command 配置示例

- `-lic` 示例：

  `/usr/bin/env zsh -c "/opt/homebrew/bin/tmux has-session -t dev 2>/dev/null && /opt/homebrew/bin/tmux attach -t dev || zsh -lic '/opt/homebrew/bin/tmux new -s dev'"`

  完全等价于:

  `/usr/bin/env zsh -c '/opt/homebrew/bin/tmux new -As dev'`

  解析：tmux Check 和 Attach 的时候使用的 `-c`，new 的时候，tmux 会在 session 内部通过 `-lic` 启动 shell(tmux 默认行为）。

- tmux 跟随待执行命令：

  `/usr/bin/env zsh -c '/opt/homebrew/bin/tmux new -As xxx "zsh -lic \"exec npx xxx@latest\""'`

  解析：tmux new session 的时候，如果跟随了待执行命令，就不在走上面提供的默认行为(`-lic`），即 tmux 默认在裸环境下执行待执行命令。需要手动补齐登录交互环境，即 `zsh -lic "exec npx xxx@latest"`。

---
