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


## screen
### Solve screen error "Cannot open your terminal '/dev/pts/0' - please check"

原因：用 ssh 登一个账户，`su - xxx` 到另一个账户，再用 screen。

* [Solve screen error "Cannot open your terminal '/dev/pts/0' - please check"](https://makandracards.com/makandra/2533-solve-screen-error-cannot-open-your-terminal-dev-pts-0-please-check)
* [解决screen Cannot open your terminal '/dev/pts/1'问题](http://blog.sina.com.cn/s/blog_704836f401010osn.html)，有详细原因分析，ssh userA，系统分配的 tty 给 userA，如果 su - userB，userB 还是用的 userA 的 tty，只可读，不可写。
