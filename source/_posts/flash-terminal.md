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
   /usr/bin/env zsh -c "/opt/homebrew/bin/tmux has-session -t dev 2>/dev/null && /opt/homebrew/bin/tmux attach -t dev || zsh -lic '/opt/homebrew/bin/tmux new -s dev \; split-window -h \; split-window -v \;'"
   ```

   **命令细节说明**：
   - 使用 `has-session` 检测会话是否已存在，选择不同的启动方式
   - **会话已存在**：直接 `attach`，外层使用 `zsh -c`（不加载 login shell 配置，0ms 启动）
   - **会话不存在**：使用 `zsh -lic` 创建（加载完整 login shell `.zshrc` 配置，确保 PATH 正确）
   - **为什么创建时必须加载配置**：tmux server 启动时会继承外层 shell 的环境变量（特别是 PATH），`.tmux.conf` 和 tpm 插件在初始化阶段需要正确的 PATH 来执行 `git`、`bash` 等命令，否则会出现 127（command not found）错误

   也可使用简化版（不分割窗口）：

   ```bash
   /usr/bin/env zsh -c "/opt/homebrew/bin/tmux has-session -t dev 2>/dev/null && /opt/homebrew/bin/tmux attach -t dev || zsh -lic '/opt/homebrew/bin/tmux new -s dev'"
   ```

3. tmux 使用非常简单，了解下就可以熟练操作。
4. 后续工作中需要快速执行命令，通过 Raycast 打开 iterm2 即可，0ms 延迟启动。
5. 使用完可直接 kill iterm2 窗口，依靠 tmux 做会话保持。
6. 其他场景，按个人喜好使用 Warp 即可。

### 结果：

Iterm2 自身启动耗时 0ms，tmux attach sesstion 0ms。 配合一下，终端秒开，快速执行工作生活中的小命令，再也不用处理 `.zshrc` 多配置启动慢问题。

---
