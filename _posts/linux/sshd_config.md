title: sshd 配置
date: 2018-07-18 16:30:00
toc: true
tags: [ubuntu, sshd, linux]
categories: linux
keywords: [ubuntu, linux, sshd, sshd_config, TMOUT, maxlogins]
description: sshd 相关配置，no-op 超时断开会话，限制会话数
---

## sshd_config 相关配置

### 配置超时

`timeout = ClientAliveCountMax * ClientAliveInterval`

**ClientAliveCountMax**

> Sets the number of client alive messages which may be sent without sshd(8) receiving any messages back from the client. If this threshold is reached while client alive messages are being sent, sshd will disconnect the client, terminating the session. It is important to note that the use of client alive messages is very different from TCPKeepAlive. The client alive messages are sent through the encrypted channel and therefore will not be spoofable. The TCP keepalive option enabled by TCPKeepAlive is spoofable. The client alive mechanism is valuable when the client or server depend on knowing when a connection has become inactive.

> The default value is 3. If ClientAliveInterval is set to 15, and ClientAliveCountMax is left at the default, unresponsive SSH clients will be disconnected after approximately 45 seconds.

**ClientAliveInterval**

> Sets a timeout interval in seconds after which if no data has been received from the client, **sshd**(8) will send a message through the encrypted channel to request a response from the client. The default is 0, indicating that these messages will not be sent to the client. This option applies to protocol version 2 only. 

### sessions 相关

**MaxSessions**

> Specifies the maximum number of open sessions permitted per network connection. The default is 10.

这个参数为 per network connection 的会话数限制，并不能限制总量。

**MaxStartups**

> Specifies the maximum number of concurrent unauthenticated connections to the SSH daemon. Additional connections will be dropped until authentication succeeds or the **LoginGraceTime** expires for a connection. The default is 10.

> Alternatively, random early drop can be enabled by specifying the three colon separated values ''start:rate:full'' (e.g. "10:30:60"). **sshd**(8) will refuse connection attempts with a probability of ''rate/100'' (30%) if there are currently ''start'' (10) unauthenticated connections. The probability increases linearly and all connection attempts are refused if the number of unauthenticated connections reaches ''full'' (60).

此参数用于控制还未登录成功的会话数。并不能限制登录成功的会话数。

## Linux 相关配置

### NO-OP 超时踢掉会话

TMOUT，用于配置 bash 无操作（no-op）超时时间，即超过 TMOUT 秒未操作会话，则退出登录。只要会话含此变量即可，因此可将该配置放在 `/etc/profile` 或 `/etc/bash.bashrc`。

```bash
TMOUT=180
export TMOUT
```

样例：

```
[144134.041]jeromesun@xxx-12:~> ssh a@16.3.3.31
[144135.789]a@16.3.3.31's password: 
[144135.802]You are on
[144135.803]...
[144135.804]Unauthorized access and/or use are prohibited.
[144135.804]All access and/or use are subject to monitoring.
[144135.804]
[144135.805]Last login: Wed Jul 18 22:37:28 2018 from 192.168.1.92
[144136.252]a@xxx-yyy:~$ 
[144136.437]a@xxx-yyy:~$ 
[144436.444]a@xxx-yyy:~$ timed out waiting for input: auto-logout
[144436.464]Connection to 16.3.3.31 closed.
```

### 限制登录会话数

**maxlogins** / **maxsyslogins**，配置文件在 `/etc/security/limits.conf` 或 `/etc/security/limits.d/*.conf`，配置样例如下。

```
# Some of the lines in this sample might conflict and overwrite each other
# The whole range is included to illustrate the possibilities
#limit users in the users group to two logins each
@users       -       maxlogins     2
#limit all users to three logins each
*            -       maxlogins     3
#limit all users except root to 20 simultaneous logins in total
*            -       maxsyslogins     20
#limit in the users group to 5 logins in total
%users       -       maxlogins     2
#limit all users with a GID >= 1000 to two logins each
1000:        -       maxlogins     2
#limit user johndoe to one login
johndoe      -       maxlogins     2
```

运行样例（超出配置的限制数）：

```
jeromesun@xxx-12:~> ssh a@16.3.3.31
a@16.3.3.31's password: 
You are on
...

Too many logins for 'a'.
Last login: Wed Jul 18 22:37:34 2018 from 192.168.1.92
Connection to 16.3.3.31 closed.
jeromesun@xxx-12:~> 
```



## 参考链接

- https://gitlab.com/gitlab-com/infrastructure/issues/1104
- https://stackoverflow.com/questions/31114690/difference-between-maxstartups-and-maxsessions-in-sshd-config
- http://www.361way.com/ssh-autologout/4679.html
- https://linux.die.net/man/5/sshd_config
