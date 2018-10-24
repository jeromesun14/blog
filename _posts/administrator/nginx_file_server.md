title: nginx 简单文件服务器
date: 2018-10-23 10:44:55
toc: true
tags: [administrator, debian, nginx]
categories: administrator
keywords: [debian, ubuntu, nginx, file server, 文件服务器, 配置]
description: 使用 nginx 作为简单文件服务器。
---

* 安装 nginx，`sudo apt-get install nginx`
* 创建 file server 配置文件 `/etc/nginx/conf.d/file_server.conf`，在这里配置：
  + 监听的端口号
  + 服务器名称，可以是 IP
  + 文件服务器的根目录
* 重启 nginx 服务，`sudo service nginx restart`

/etc/nginx/conf.d/file_server.conf，

```
server {
    listen  80;
    server_name    192.168.1.2;
    charset utf-8;
    root /your/dir/to/share;
    location / {
        autoindex on;
        autoindex_exact_size on;
        autoindex_localtime on;
    }
}
```
