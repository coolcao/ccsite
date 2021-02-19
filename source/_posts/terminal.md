---
title: 打造一款简洁高效且比较有颜的终端
date: 2021-02-19 11:28:10
tags: [shell, zsh, terminal, 终端]
categories:
- 技术博客
- 原创
---

作为程序员，经常要和终端打交道。但是默认的终端bash，不仅丑，而且难用至极。

这里推荐使用zsh，被誉为终极shell，但刚安装的zsh默认状态下，也是丑与难用，需要配置一下，才能打造一款舒适的终端shell。

本文就使用zsh来配置一款简洁高效且颜值还不错的终端。

<!-- more -->

## 安装zsh
首先安装zsh。对于mac系统，最简单的就是使用brew进行安装：

```
brew install zsh
```

如果是linux系统，可使用对应的包管理工具进行安装，zsh是一个比较通用的软件，几乎所有的linux都能安装到。

> 一般默认的shell都是bash，但有一些比较新的系统，默认将使用zsh，比如最新的mac os，arch linux等系统默认已使用zsh，所以就不需要安装了。
> 查看当前系统安装了哪些shell，可以查看 `/etc/shells` 文件：
```
       │ File: /etc/shells
───────┼──────────────────────────────────────────────────────────────────────────
   1   │ # List of acceptable shells for chpass(1).
   2   │ # Ftpd will not allow users to connect who are not using
   3   │ # one of these shells.
   4   │
   5   │ /bin/bash
   6   │ /bin/csh
   7   │ /bin/dash
   8   │ /bin/ksh
   9   │ /bin/sh
  10   │ /bin/tcsh
  11   │ /bin/zsh
  12   │ /opt/homebrew/bin/fish
```

## 切换shell为zsh
```
chsh -s /bin/zsh
```

这样我们就切换到了所谓的终极shell： zsh 了，但如上所说，默认zsh也是非常丑，难用：

![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1613710489_20210219114258051_2082537145.png)

如上图，默认zsh就是这么丑，和bash一样丑，需要去调整。

## 安装插件
我们可以安装一些插件来优化zsh的交互性，这里推荐几个非常有用的插件：

1. zsh-syntax-highlighting              语法高亮
2. zsh-autosuggestion                   自动补全
3. zsh-history-substring-search         history子串查询
4. autojump                             自动跳转目录
5. dircolors-solarized                  终端彩色显示
6. starship                             一款友好的终端提示工具

前4个插件，可以通过homebrew安装，安装后，注意看后面的提示，要在.zshrc文件中添加脚本激活插件。
比如语法高亮插件，要在.zshrc中添加：
```
source /opt/homebrew/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```

zsh-autosuggestion：
```
source /opt/homebrew/share/zsh-autosuggestions/zsh-autosuggestions.zsh
```

zsh-history-substring-search:
```
source /opt/homebrew/share/zsh-history-substring-search/zsh-history-substring-search.zsh
bindkey '^[[A' history-substring-search-up
bindkey '^[[B' history-substring-search-down
bindkey '^P' history-substring-search-up
bindkey '^N' history-substring-search-down

```

下面四个bindkey为绑定快捷键，绑定上下箭头键以及CTRL+P,CTRL+N键为插件的上下翻页快捷键。

autojump：
```
[[ -s /Users/coolcao/.autojump/etc/profile.d/autojump.sh ]] && source /Users/coolcao/.autojump/etc/profile.d/autojump.sh
autoload -U compinit && compinit -u
```

## dircolors-solarized
这个是为解决mac下终端不能彩色显示的问题。
首先安装coreutils：
```
brew install coreutils
```
然后将`https://github.com/seebi/dircolors-solarized.git`项目克隆到本地，将项目其中的一个色彩方案扔到 `~/.dir_colors`，然后使用下面的脚本激活并alias一下ls命令：
```
eval `gdircolors -b $HOME/.dir_colors`
alias ls='gls --color=auto -l'
```

## starship
虽然经过配置上面这几个插件后，zsh的交互性大大提高，但还是不美观，外观样式和上面的没有二致。
starship是一款优化终端提示的工具，可兼容多种shell，这里我们使用其来美化一下终端。
美化zsh也可以上网找一些zsh的脚本，这里我使用starship是因为这个工具可配性更高。

具体的安装方法可以参见starship的[官网](https://starship.rs/zh-cn/guide/)。

安装完成后在.zshrc种添加脚本启用：
```
eval "$(starship init zsh)"
```

下面贴一下我的starship配置：

```toml
[character] # The name of the module we are configuring is "character"
error_symbol = "[](bold red) " 
success_symbol = "[גּ](bold green)" # The "success_symbol" segment is being set to "➜" with the color "bold green"
vicmd_symbol = "[V](bold blue) " 

[battery]
#full_symbol = "🔋"
#charging_symbol = "🔌"
#discharging_symbol = "⚡️️"
charging_symbol = ""
discharging_symbol = ""
full_symbol = ""

[[battery.display]]
style = "bold red"
threshold = 30
[[battery.display]]
style = "bold yellow"
threshold = 60
[[battery.display]]
style = "bold green"
threshold = 100

[cmd_duration]
format = " took:[$duration]($style)"

[git_branch]
symbol = " "
truncation_length = 128
truncation_symbol = ""

[git_commit]
commit_hash_length = 128
tag_symbol = " "

[hostname]
disabled = false
format = "on [$hostname](bold red) "
ssh_only = true
trim_at = ".companyname.com"

[memory_usage]
disabled = false
format = "内存占用:[${ram_pct}]($style)"
style = "bold dimmed green"
symbol = " "
threshold = -1

[nodejs]
symbol = " "

```

这里我配置了成功失败的指示图标，配置了显示电池电量的图标以及内存占用的情况。

最终效果如下：
![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1613710490_20210219124808936_1513317209.png)

如果是git项目，会相应的显示git以及项目的基本信息：
![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1613710491_20210219124922250_1673500242.png)


zsh以及starship的可玩性很高，每个人都可以配置出属于自己独一无二的终端。