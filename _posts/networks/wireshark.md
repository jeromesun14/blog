title: Wireshark Usage
date: 2015-12-13 15:05:20
toc: true
tags: [wireshark]
categories: networks
keywords: [抓包, scapy, 网络测试, wireshark, linux, root, 用户, 非]
description: Wireshark使用过程中的一些记录。
---

Linux中非ROOT用户使用wireshark抓包
----------------------------------

原理没细看，详见[《Sniffing with Wireshark as a Non-Root User》](http://packetlife.net/blog/2010/mar/19/sniffing-wireshark-non-root-user/)。

步骤：

1. 安装`setcap`
2. 创建wireshark组（可选）
3. 配置抓包能力
4. logout再重新登录生效。

```
1. 
sudo apt-get install libcap2-bin

2. 
sudo groupadd wireshark
sudo usermod -a -G wireshark your_user_name
sudo newgrp wireshark
（会切成root用户）
chgrp wireshark /usr/bin/dumpcap
chmod 750 /usr/bin/dumpcap

3. 
setcap cap_net_raw,cap_net_admin=eip /usr/bin/dumpcap
确认是否配置成功：
# getcap /usr/bin/dumpcap
/usr/bin/dumpcap = cap_net_admin,cap_net_raw+eip
```



