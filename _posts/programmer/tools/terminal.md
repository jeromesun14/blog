title: 终端工具使用记录
date: 2018-05-07 16:23:33
toc: true
tags: [terminal]
categories: programmer
keywords: [终端, terminal, tmux, conemu, cygwin, screen, mobaxterm, putty, secureCRT]
description: windows / linux 终端使用记录。
---

## ConEmu

### ssh Linux 系统后，vim 卡住

* 原因：不明
* 规避：在 .bashrc 中 `export TERM=xterm`，默认 TERM 值为 linux

### 文本选取去空格

`Settings` -> `Keys & Macro` -> `Mark/Copy` -> `Text selection`，反选 `Detect line ends`，勾选 `Trim trailing spaces`

## tmux

### 色彩问题

* `tmux -2` 
* 在 .bashrc 中添加 `alias tmux="tmux -2"` 解决

### vim 背景色问题
见 [256 color support for vim background in tmux](https://superuser.com/questions/399296/256-color-support-for-vim-background-in-tmux/399326#399326)。

.bashrc 配置：

```
if [[ -z $TMUX ]]; then
    if [ -e /usr/share/terminfo/x/xterm+256color ]; then # may be xterm-256 depending on your distro
        export TERM='xterm-256color'
    else
        export TERM='xterm'
    fi
else
    if [ -e /usr/share/terminfo/s/screen-256color-s ]; then
        export TERM='screen-256color'
    else
        export TERM='screen'
    fi
fi
```

### 如何翻页看 log （scroll 功能）

见 [How do I scroll in tmux?](https://superuser.com/questions/209437/how-do-i-scroll-in-tmux?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)
