title: SecureCRT python 脚本记录
date: 2013-11-21 13:13:24
toc: true
tags: [python, SecureCRT]
categories: programmer
keywords: [python, secure CRT, ctrl c, ^c, 脚本, script, vb]
description: Secure CRT python 脚本记录
---

SecureCRT 从 7.0 版本开始支持 python。

SecureCRT 脚本发送 ctrl c 或其他特殊字符
----------------------------------------

python 脚本不支持 SendKeys，之前不知道怎么发送 ctrl c，瞎写了一个 vb 脚本用 SendKeys 发送。

```
#$language = "VBScript"
#$interface = "1.0"

Sub Main()
crt.Screen.Clear

While 1
    crt.screen.sendkeys("^C")
    crt.sleep(200)
Wend

End Sub
```

Google 一下，发现 python 本身就可以发送控制码，在 stackoverflow 中，`IIRC, Ctrl-C is etx. Thus send \x03`。再看 ascii 码，所有的这些控制码都有对应的 ascii 码。python 版的一直发 ctrl + c：

```
# $language = "python"
# $interface = "1.0"

while True:
    crt.Screen.Send("\x03")
    crt.Sleep(200)
```

