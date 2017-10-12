title: dokuwiki 邮件发送
date: 2017-04-06 13:51:00
toc: true
tags: [dokuwiki, administrator]
categories: dokuwiki
keywords: [administrator, dokuwiki, plugin, 插件, 发送邮件, sendmail, 失败, fail, smtp]
description: dokuwiki 解决无法发送邮件通知。
---

# 不使用外部邮箱可直接发邮件
## sendmail

* 安装 sendmail，`apt-get install sendmail`，安装时间有点长，约 7 分钟。
* 配置 hosts，详见 [Sendmail very slow - /etc/hosts configuration](https://superuser.com/questions/626205/sendmail-very-slow-etc-hosts-configuration/626219#626219)。
  + `hostname`，获取本机 hostname，本样例值为 ubuntu
  + `vi /etc/hosts`，修改 `127.0.0.1 localhost` 一行为 `127.0.0.1       ubuntu.localdomain ubuntu localhost`。**注意**，`ubuntu.localdomain` 为必须值，而且不可另起一行。

## mail 发送失败问题排查
### 注册用户后，邮件发送失败
提示 regmailfail “发送密码邮件时产生错误。请联系管理员！”，如果语言为英文的话，log 为“Looks like there was an error on sending the password mail. Please contact the admin!”。

### 查看发送失败的原因

查看 `/var/log/lighttpd/error.log`，确认原因为服务器没有安装 sendmail。

```
2017-09-29 11:13:55: (mod_fastcgi.c.2673) FastCGI-stderr: PHP Question2Answer email send error: Could not instantiate mail function.
```

`apt-get install sendmail`，安装时间有点长，约 7 分钟。

### 注册后，在注册页面卡很久

查看 `/var/log/mail.log`，确认发一封邮件约需要 20 分钟。原因为解析不到 hostname，sleep 一段时间后 retry。

```
Sep 29 16:40:27 ubuntu sm-msp-queue[21553]: My unqualified host name (ubuntu) unknown; sleeping for retry
Sep 29 16:42:27 ubuntu sm-msp-queue[21553]: unable to qualify my own domain name (ubuntu) -- using short name
Sep 29 16:57:19 ubuntu sendmail[21561]: My unqualified host name (ubuntu) unknown; sleeping for retry
Sep 29 16:59:19 ubuntu sendmail[21561]: unable to qualify my own domain name (ubuntu) -- using short name
Sep 29 16:59:19 ubuntu sendmail[21561]: v8T8xJkc021561: from=www-data, size=2020, class=0, nrcpts=1, msgid=<201
709290859.v8T8xJkc021561@ubuntu>, relay=www-data@localhost
Sep 29 17:00:19 ubuntu sm-mta[21562]: v8T8xJvo021562: from=<www-data@ubuntu>, size=2191, class=0, nrcpts=1, msg
id=<201709290859.v8T8xJkc021561@ubuntu>, proto=ESMTP, daemon=MTA-v4, relay=localhost [127.0.0.1]
Sep 29 17:00:19 ubuntu sendmail[21561]: v8T8xJkc021561: to=sunnogo1 <sun@xxx.com.cn>, ctladdr=www-da
ta (33/33), delay=00:01:00, xdelay=00:01:00, mailer=relay, pri=32020, relay=[127.0.0.1] [127.0.0.1], dsn=2.0.0,
 stat=Sent (v8T8xJvo021562 Message accepted for delivery)
Sep 29 17:00:40 ubuntu sm-mta[21582]: STARTTLS=client, relay=mx.xxx.com.cn., version=TLSv1/SSLv3, verify=F
AIL, cipher=ECDHE-RSA-AES256-SHA, bits=256/256
Sep 29 17:00:41 ubuntu sm-mta[21582]: v8T8xJvo021562: to=<sun@xxx.com.cn>, ctladdr=<www-data@ubuntu>
 (33/33), delay=00:00:22, xdelay=00:00:22, mailer=esmtp, pri=122191, relay=mx.xxx.com.cn. [172.16.2.173],
dsn=2.0.0, stat=Sent (<201709290859.v8T8xJkc021561@ubuntu> [InternalId=1588969] Queued mail for delivery)
```

### 配置好 hosts 之后，注册还需要等 20 多秒

安装后，发送一封邮件约 20s。从 log 上看，是公司邮件服务器响应太慢。dokuwiki 的邮件发送是同步发送。

```
Sep 29 17:38:12 ubuntu sendmail[1864]: v8T9cCZX001864: to=sunnogo2 <sun@xxx.com.cn>, ctladdr=www-dat
a (33/33), delay=00:00:00, xdelay=00:00:00, mailer=relay, pri=32020, relay=[127.0.0.1] [127.0.0.1], dsn=2.0.0,
stat=Sent (v8T9cCSA001865 Message accepted for delivery)
Sep 29 17:38:33 ubuntu sm-mta[1867]: STARTTLS=client, relay=mx.xxx.com.cn., version=TLSv1/SSLv3, verify=FA
IL, cipher=ECDHE-RSA-AES256-SHA, bits=256/256
Sep 29 17:38:34 ubuntu sm-mta[1867]: v8T9cCSA001865: to=<sun@xxx.com.cn>, ctladdr=<www-data@ubuntu.l
ocaldomain> (33/33), delay=00:00:22, xdelay=00:00:22, mailer=esmtp, pri=122215, relay=mx.xxx.com.cn. [172.
16.2.173], dsn=2.0.0, stat=Sent (<201709290938.v8T9cCZX001864@ubuntu.localdomain> [InternalId=1590347] Queued m
ail for delivery)
```

## 附：SMTP发包方式

详见 [dokuwiki 邮件发送](http://sunyongfeng.com/201704/administrator/dokuwiki/sendmail.html)

## 附：sendmail 安装 log

```
root@ubuntu:/var/www/dokuwiki# apt-get install sendmail
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following extra packages will be installed:
  m4 procmail sendmail-base sendmail-bin sendmail-cf sensible-mda
Suggested packages:
  sendmail-doc rmail logcheck sasl2-bin
Recommended packages:
  default-mta mail-transport-agent fetchmail
The following NEW packages will be installed:
  m4 procmail sendmail sendmail-base sendmail-bin sendmail-cf sensible-mda
0 upgraded, 7 newly installed, 0 to remove and 169 not upgraded.
Need to get 1,038 kB of archives.
After this operation, 4,707 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://cn.archive.ubuntu.com/ubuntu/ trusty/main m4 amd64 1.4.17-2ubuntu1 [195 kB]
Get:2 http://cn.archive.ubuntu.com/ubuntu/ trusty/universe sendmail-base all 8.14.4-4.1ubuntu1 [139 kB]
Get:3 http://cn.archive.ubuntu.com/ubuntu/ trusty/universe sendmail-cf all 8.14.4-4.1ubuntu1 [86.1 kB]
Get:4 http://cn.archive.ubuntu.com/ubuntu/ trusty/universe sendmail-bin amd64 8.14.4-4.1ubuntu1 [469 kB]
Get:5 http://cn.archive.ubuntu.com/ubuntu/ trusty-updates/main procmail amd64 3.22-21ubuntu0.1 [135 kB]
Get:6 http://cn.archive.ubuntu.com/ubuntu/ trusty/universe sensible-mda amd64 8.14.4-4.1ubuntu1 [8,246 B]
Get:7 http://cn.archive.ubuntu.com/ubuntu/ trusty/universe sendmail all 8.14.4-4.1ubuntu1 [6,248 B]
Fetched 1,038 kB in 0s (1,375 kB/s)   
Selecting previously unselected package m4.
(Reading database ... 64411 files and directories currently installed.)
Preparing to unpack .../m4_1.4.17-2ubuntu1_amd64.deb ...
Unpacking m4 (1.4.17-2ubuntu1) ...
Selecting previously unselected package sendmail-base.
Preparing to unpack .../sendmail-base_8.14.4-4.1ubuntu1_all.deb ...
Unpacking sendmail-base (8.14.4-4.1ubuntu1) ...
Selecting previously unselected package sendmail-cf.
Preparing to unpack .../sendmail-cf_8.14.4-4.1ubuntu1_all.deb ...
Unpacking sendmail-cf (8.14.4-4.1ubuntu1) ...
Selecting previously unselected package sendmail-bin.
Preparing to unpack .../sendmail-bin_8.14.4-4.1ubuntu1_amd64.deb ...
Unpacking sendmail-bin (8.14.4-4.1ubuntu1) ...
Selecting previously unselected package procmail.
Preparing to unpack .../procmail_3.22-21ubuntu0.1_amd64.deb ...
Unpacking procmail (3.22-21ubuntu0.1) ...
Selecting previously unselected package sensible-mda.
Preparing to unpack .../sensible-mda_8.14.4-4.1ubuntu1_amd64.deb ...
Unpacking sensible-mda (8.14.4-4.1ubuntu1) ...
Selecting previously unselected package sendmail.
Preparing to unpack .../sendmail_8.14.4-4.1ubuntu1_all.deb ...
Unpacking sendmail (8.14.4-4.1ubuntu1) ...
Processing triggers for man-db (2.6.7.1-1ubuntu1) ...
Processing triggers for install-info (5.2.0.dfsg.1-2) ...
Processing triggers for ureadahead (0.100.0-16) ...
Setting up m4 (1.4.17-2ubuntu1) ...
Setting up sendmail-base (8.14.4-4.1ubuntu1) ...
Saving current /etc/mail/sendmail.mc,cf to /var/backups
Setting up sendmail-cf (8.14.4-4.1ubuntu1) ...
Setting up sendmail-bin (8.14.4-4.1ubuntu1) ...
update-rc.d: warning: default stop runlevel arguments (0 1 6) do not match sendmail Default-Stop values (1)
update-alternatives: using /usr/lib/sm.bin/sendmail to provide /usr/sbin/sendmail-mta (sendmail-mta) in auto mode
update-alternatives: using /usr/lib/sm.bin/sendmail to provide /usr/sbin/sendmail-msp (sendmail-msp) in auto mode
 * Stopping Mail Transport Agent (MTA) sendmail                                                                                                               [ OK ] 
Updating sendmail environment ...
Reading configuration from /etc/mail/sendmail.conf.
Validating configuration.
Writing configuration to /etc/mail/sendmail.conf.
Writing /etc/cron.d/sendmail.
Could not open /etc/mail/databases(No such file or directory), creating it.
Reading configuration from /etc/mail/sendmail.conf.
Validating configuration.
Writing configuration to /etc/mail/sendmail.conf.
Writing /etc/cron.d/sendmail.
Could not open /etc/mail/databases(No such file or directory), creating it.
Reading configuration from /etc/mail/sendmail.conf.
Validating configuration.
Creating /etc/mail/databases...

Checking filesystem, this may take some time - it will not hang!
  ...   Done.
 
Checking for installed MDAs...
Adding link for newly extant program (mail.local)
Adding link for newly extant program (procmail)
sasl2-bin not installed, not configuring sendmail support.

To enable sendmail SASL2 support at a later date, invoke "/usr/share/sendmail/update_auth"

 
Creating/Updating SSL(for TLS) information
Creating /etc/mail/tls/starttls.m4...
Creating SSL certificates for sendmail.
Generating DSA parameters, 2048 bit long prime
This could take some time
........+.+.........+.....................+.+.......+...............+.............+............+++++++++++++++++++++++++++++++++++++++++++++++++++*
.+.................+.......+..............+.+..............................+....+...............+.+..+...+......+.......+..+....................+...................+................+.+..........+....+.+.....+..+..+.................+.............+....+....+.......+.................................+..............+....+..+.+......................+.......+.....+.............+.....+.........+............+......+....................+....+....+..+................................+.......+........+.+...+................+....................+++++++++++++++++++++++++++++++++++++++++++++++++++*
Generating RSA private key, 2048 bit long modulus
...........................................................................+++
..............+++
e is 65537 (0x10001)

*** *** *** WARNING *** WARNING *** WARNING *** WARNING *** *** ***

Everything you need to support STARTTLS (encrypted mail transmission
and user authentication via certificates) is installed and configured
but is *NOT* being used.

To enable sendmail to use STARTTLS, you need to:
1) Add this line to /etc/mail/sendmail.mc and optionally
   to /etc/mail/submit.mc:
  include(`/etc/mail/tls/starttls.m4')dnl
2) Run sendmailconfig
3) Restart sendmail

Checking {sendmail,submit}.mc and related databases...
Reading configuration from /etc/mail/sendmail.conf.
Validating configuration.
Creating /etc/mail/databases...
Reading configuration from /etc/mail/sendmail.conf.
Validating configuration.
Creating /etc/mail/databases...
Reading configuration from /etc/mail/sendmail.conf.
Validating configuration.
Creating /etc/mail/Makefile...
Reading configuration from /etc/mail/sendmail.conf.
Validating configuration.
Writing configuration to /etc/mail/sendmail.conf.
Writing /etc/cron.d/sendmail.
Disabling HOST statistics file(/var/lib/sendmail/host_status).
Creating /etc/mail/sendmail.cf...
Creating /etc/mail/submit.cf...
Informational: confCR_FILE file empty: /etc/mail/relay-domains
Warning: confCT_FILE source file not found: /etc/mail/trusted-users
 it was created
Informational: confCT_FILE file empty: /etc/mail/trusted-users
Updating /etc/mail/access...
Linking /etc/aliases to /etc/mail/aliases
Informational: ALIAS_FILE file empty: /etc/mail/aliases
Updating /etc/mail/aliases...



WARNING: local host name (ubuntu) is not qualified; see cf/README: WHO AM I?
/etc/mail/aliases: 0 aliases, longest 0 bytes, 0 bytes total
 
Warning: 1 database(s) sources
        were not found, (but were created)
▽       please investigate.
 * Starting Mail Transport Agent (MTA) sendmail                                                                                                                      

                                                                                                                                                              [ OK ]
Setting up procmail (3.22-21ubuntu0.1) ...
Setting up sensible-mda (8.14.4-4.1ubuntu1) ...
Setting up sendmail (8.14.4-4.1ubuntu1) ...
```


# CAUTION！
若开户 163 邮箱的“授权码”功能，本文无效。

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
Got: 250-coremail 1Uxr2xKj7kG0xkI17xGrU7I0s8sdfsf808Cz28x1UUUUU7Ic2I0Y2UFdFM3xUCa0xDrUUUUj
Got: 250-STARTTLS
Got: 250 8BITMIME
Sent: AUTH LOGIN
Got: 334 dXdsDFD806
Sent: c3Vu80FSKerjMuY29t
Got: 334 UGFz32380cmQ6
Sent: QVJzYW5iSDF8989bDYwNTk1
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
