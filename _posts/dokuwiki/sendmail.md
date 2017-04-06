title: dokuwiki 邮件发送
date: 2017-04-06 13:51:00
toc: true
tags: [dokuwiki]
categories: dokuwiki
keywords: [dokuwiki, plugin, 插件, 发送邮件, sendmail, 失败]
description: dokuwiki 解决无法发送邮件通知。
---

# 问题概述

* 插件 [smtp](https://www.dokuwiki.org/plugin:smtp)
* 问题：配置 163 邮箱后，无法发出邮件
* 原因：发送邮件的 sender 邮箱与配置授权的 163 邮箱不一致
* 解决：`配置设置`->`通知设置`，`mailfrom 自动发送邮件时使用的邮件地址` 填成与授权的 163 邮箱保持一致。

# smtp 插件配置

配置：`管理`->`配置设置`，点击目录，选择 `smtp`。

配置项：
* plugin»smtp»smtp_host，您的 SMTP 发送服务器。比如 `smtp.163.com`。
* plugin»smtp»smtp_port，您的 SMTP 服务器监听端口。通常是 25，对于 SSL常是 465。比如，`25`
* plugin»smtp»smtp_ssl，您的 SMTP 服务器所用的加密类型？有 ssl/tls/无，比如 `无`
* plugin»smtp»auth_user，如果需要认证，在这里输入您的用户名。比如 `xxx@163.com`
* plugin»smtp»auth_pass，对应 auth_user 用户名的密码。**注意**，每次进行 `配置设置`，都要重新输入密码。
* plugin»smtp»debug，在发送失败时显示完整的错误信息？在一切正常时请关闭！调试时勾选。

# smtp 调试与553问题解决

使用 plugin:smtp 自带的测试工具，在 `管理` -> `附加插件`，点击 `@检查 SMTP 配置`，进入 SMTP 测试界面。写目标邮箱地址到 `to` 框中，点 `sendmail` 按钮。

出现错误 Log：

```
 There was an unexpected problem communicating with SMTP: Unexpected return code - Expected: 250, Got: 553 | 553 Mail from must equal authorized user
 SMTP log:
Set: the server
Set: the auth
Set: a message will be sent
Connecting to smtp.163.com at 25
Got: 220 163.com Anti-spam GT for Coremail System (163com[20141201])
Sent: EHLO [172.18.111.192]
Got: 250-mail
Got: 250-PIPELINING
Got: 250-AUTH LOGIN PLAIN
Got: 250-AUTH=LOGIN PLAIN
Got: 250-coremail 1Uxr2xKj7kG0xkI17xGrU7I0s8FY2U3Uj8Cz28x1UUUUU7Ic2I0Y2UFdFM3xUCa0xDrUUUUj
Got: 250-STARTTLS
Got: 250 8BITMIME
Sent: AUTH LOGIN
Got: 334 dXNlcm5hbWU6
Sent: c3Vubm9nb0AxNjMuY29t
Got: 334 UGFzc3dvcmQ6
Sent: QVJzYW5iYnNxbDYwNTk1
Got: 235 Authentication successful
Sent: MAIL FROM:<noreply@163.com>
Got: 553 Mail from must equal authorized user
Above may contain passwords - do not post online!
 Message wasn't sent. SMTP seems not to work properly.
```

提醒 [553 错误](http://www.mail163.cn/fault/analysis/1109.html)，发送者的邮箱地址必须是登陆的邮箱地址。

* 客户端错误代码, smtp sever reply:553 mail from must equal authorized user 
* 形成原因, 网易服务器smtp机器要求身份验证帐号和发信帐号必须一致，如果用户在发送邮件时，身份验证帐号和发件人帐号是不同的，因此拒绝发送。 
* 解决办法/流程, 退信表明发信服务器要求身份验证. 建议：
  * 确认一定要勾选“我的服务器要求身份验证”
  * 请核实客户端中设置的电子邮箱地址是否填写正确，确认是输入完整的正确的邮箱地址。
  * 设置的回复地址是否和邮箱地址是一致的，并且是完整的邮箱地址。 

回到 `管理` -> `配置管理`界面，发现 `通知设置` 中的 `mailfrom 自动发送邮件时使用的邮件地址`，填为 `noreply@163.com`，与授权的邮箱不一致。将此配置改为授权的 xxx@163.com 即可。修改后，重新测试，可正常发送邮件。

log: 

```
 Message was sent. SMTP seems to work.
```

发送的邮件：

```
发件人: xxx@163.com [mailto:xxx@163.com] 
发送时间: 2017年4月6日 14:08
收件人: 孙勇峰 <sunyongfeng@xxx.com>
主题: [Administrator] DokuWiki says hello

Hi sunyongfeng

This is a test from http://xxx.com/dokuwiki/ 
________________________________________
本邮件由 DokuWiki 自动创建
http://xxx.com/dokuwiki/ 
```
