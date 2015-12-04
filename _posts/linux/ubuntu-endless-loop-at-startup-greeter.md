title: 【解决】Ubuntu 12.04启机到登陆界面后，在登陆界面死循环
date: 2015-07-02 22:08:18
toc: false
tags: [Linux, ubuntu]
categories: linux
keywords: [ubuntu, 登录, 死循环]
---

到登录界面后，即使密码正确输入之后，会直接返回登陆界面让你再输入密码，一直这样死循环，没有进入图形界面。

查看/var/log/upstart/lightdm.log有下面的提示：

```
(process:1231): WARNING: 
Error reading existing Xauthority: Error opening file: Permission denied
Error writing X authority: Error opening file '/home/xx/.Xauthority': Permission denied
```

原因是/home/xx/.Xauthority的owner和group都是root，把.Xauthority的这两个属性改成自己的用户名就可以了。

要养成出问题之后，多看/var下的记录的好习惯！

