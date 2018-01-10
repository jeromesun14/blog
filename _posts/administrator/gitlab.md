title: 搭建 gitlab 服务器
date: 2017-12-16 17:19:20
toc: true
tags: [版本控制, git]
categories: tools
keywords: [git, gitlab, server, 服务器, 搭建, 备份, backup]
description: 记录搭建 gitlab 服务器的过程。
---

## apt-get install gitlab-ce 失败

问题 log: gem devise-two-factor 的依赖关系得不到满足。

```

Setting up ruby-paranoia (2.1.3-1) ...
Setting up gitlab (8.5.8+dfsg-5) ...
Creating/updating gitlab user account...
adduser: Warning: The home directory `/var/lib/gitlab' does not belong to the user you are currently creating.
Creating runtime directories for gitlab...
Updating file permissions...
Configuring hostname and email...
Registering /etc/gitlab/gitlab.yml via ucf

Creating config file /etc/gitlab/gitlab.yml with new version
Registering /etc/gitlab/gitlab-debian.conf via ucf

Creating config file /etc/gitlab/gitlab-debian.conf with new version

Creating config file /etc/nginx/sites-available/localhost with new version
Reloading nginx configuration...
Create database if not present
psql: FATAL:  database "gitlab_production" does not exist
psql: FATAL:  role "gitlab" does not exist
Create gitlab user with create database privillege...
CREATE ROLE
Make gitlab user owner of gitlab_production database...
ALTER DATABASE
Grant all privileges to gitlab user...
GRANT
Verifying we have all required libraries...
Could not find gem 'devise-two-factor (~> 2.0.0)' in any of the gem sources
listed in your Gemfile or available on this machine.
dpkg: error processing package gitlab (--configure):
 subprocess installed post-installation script returned error exit status 7
Setting up ruby-debug-inspector (0.0.2-1.1build3) ...
Setting up ruby-binding-of-caller (0.7.2+debian1-3) ...
Setting up ruby-bson (1.10.0-2) ...
Setting up ruby-bson-ext (1.10.0-2build5) ...
Setting up ruby-columnize (0.9.0-1) ...
Setting up ruby-byebug (5.0.0-1build3) ...
Setting up ruby2.3-dev:amd64 (2.3.1-2~16.04.2) ...
Setting up ruby-dev:amd64 (1:2.3.0+1) ...
Setting up ruby-ffi (1.9.10debian-1build2) ...
Setting up ruby-jbuilder (2.3.1-1) ...
Setting up ruby-libvirt (0.5.1-3build5) ...
Setting up ruby-rb-inotify (0.9.7-1) ...
Setting up ruby-listen (3.0.3-3) ...
Setting up ruby-msgpack (0.6.2-1build4) ...
Setting up ruby-rabl (0.11.4-2) ...
Setting up ruby-rabl-rails (0.4.1-1) ...
Setting up ruby-sdoc (0.4.1-1) ...
Setting up ruby-spring (1.3.6-2) ...
Setting up ruby-sqlite3 (1.3.11-2build1) ...
Setting up ruby-web-console (2.2.1-2) ...
Processing triggers for libc-bin (2.23-0ubuntu9) ...
Processing triggers for systemd (229-4ubuntu19) ...
Processing triggers for ureadahead (0.100.0-19) ...
Processing triggers for ufw (0.35-0ubuntu2) ...
Errors were encountered while processing:
 gitlab
E: Sub-process /usr/bin/dpkg returned an error code (1)
netadmin@kmc-b0232:~$ 
```

## 改用官方最新的 deb 包
详见 https://packages.gitlab.com/gitlab/gitlab-ce/install，

* curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
* sudo apt-get install gitlab-ce 

```
netadmin@kmc-b0232:~$ curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
Detected operating system as Ubuntu/xenial.
Checking for curl...
Detected curl...
Running apt-get update... done.
Installing apt-transport-https... done.
Installing /etc/apt/sources.list.d/gitlab_gitlab-ce.list...done.
Importing packagecloud gpg key... done.
Running apt-get update... done.

The repository is setup! You can now install packages.
netadmin@kmc-b0232:~$
```

## 配置
目前还未配置邮件发送。


### nginx
gitlab 默认使用 nginx。安装完后 nginx sites-enabled 有两个，我把 default 直接删了。
然后把另一个的 server_name 改成自己的 ip。修改完记得 `sudo service nginx restart`。

```
netadmin@kmc-b0232:/etc/nginx/sites-available$ ls ../sites-enabled/ -al
total 8
drwxr-xr-x 2 root root 4096 12月 13 19:42 .
drwxr-xr-x 6 root root 4096 12月 13 19:40 ..
lrwxrwxrwx 1 root root   36 12月 13 14:23 gitlab -> /etc/nginx/sites-available/localhost
netadmin@kmc-b0232:/etc/nginx/sites-available$ cat gitlab
## GitLab
##
## Lines starting with two hashes (##) are comments with information.
## Lines starting with one hash (#) are configuration parameters that can be uncommented.
##
##################################
##        CONTRIBUTING          ##
##################################
##
## If you change this file in a Merge Request, please also create
## a Merge Request on https://gitlab.com/gitlab-org/omnibus-gitlab/merge_requests
##
###################################
##         configuration         ##
###################################
##
## See installation.md#using-https for additional HTTPS configuration details.

upstream gitlab-workhorse {
  server unix:/run/gitlab/gitlab-workhorse.socket fail_timeout=0;
}

## Normal HTTP host
server {
  ## Either remove "default_server" from the listen line below,
  ## or delete the /etc/nginx/sites-enabled/default file. This will cause gitlab
  ## to be served if you visit any address that your server responds to, eg.
  ## the ip address of the server (http://x.x.x.x/)n 0.0.0.0:80 default_server;
  listen 0.0.0.0:80;
  listen [::]:80;
  server_name your_url_or_your_ip; ## Replace this with something like gitlab.example.com
  server_tokens off; ## Don't show the nginx version number, a security best practice
  root /usr/share/gitlab/public;

  ## See app/controllers/application_controller.rb for headers set

  ## Individual nginx logs for this GitLab vhost
  access_log  /var/log/nginx/gitlab_access.log;
  error_log   /var/log/nginx/gitlab_error.log;

  location / {
    client_max_body_size 0;
    gzip off;

    ## https://github.com/gitlabhq/gitlabhq/issues/694
    ## Some requests take more than 30 seconds.
    proxy_read_timeout      300;
    proxy_connect_timeout   300;
    proxy_redirect          off;

    proxy_http_version 1.1;

    proxy_set_header    Host                $http_host;
    proxy_set_header    X-Real-IP           $remote_addr;
    proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
    proxy_set_header    X-Forwarded-Proto   $scheme;

    proxy_pass http://gitlab-workhorse;
  }
}
netadmin@kmc-b0232:/etc/nginx/sites-available$
```

### gitlab 配置
在 `/etc/gitlab` 目录下，将以下两个文件各一处配置改成你的本机 IP。（我目前只想在局域网内访问 gitlab 服务器）

* gitlab.rb:external_url 'http://your_url_or_your_ip'
* gitlab.yml:    host: your_ip

修改完后 `sudo gitlab-ctl restart` （还不清楚与 sudo gitlab-ctl reconfigure 的差异），即可通过 IP 访问到本地 gitlab 服务器。

## 备份

备份功能官方说明文档，详见 https://docs.gitlab.com/omnibus/settings/backups.html。这里只讲直接运行在 host 主机上的模式，docker 模式见链接里的说明。

* 备份命令，`sudo gitlab-rake gitlab:backup:create`
* 备份路径配置
  + 配置文件路径 `/etc/gitlab/gitlab.rb`
  + 配置内容，修改 `manage_backup_path` 为 true，`backup_path` 为你所需要的路径，默认情况这两项都被注释，默认路径为 `/var/opt/gitlab/backups`
  + 配置生效，需要执行命令 `sudo gitlab-ctl reconfigure`

备份过程：

```
usernamexxx@hostnameyyy:~$ sudo gitlab-rake gitlab:backup:create     
[sudo] password for usernamexxx: 
Dumping database ... 
Dumping PostgreSQL database gitlabhq_production ... [DONE]
done
Dumping repositories ...
 * abc/wtf_proxy ... [DONE]
 * abc/wtf_proxy.wiki ...  [SKIPPED]
done
Dumping uploads ... 
done
Dumping builds ... 
done
Dumping artifacts ... 
done
Dumping pages ... 
done
Dumping lfs objects ... 
done
Dumping container registry images ... 
[DISABLED]
Creating backup archive: 1515588043_2018_01_10_10.2.4_gitlab_backup.tar ... done
Uploading backup archive to remote storage  ... skipped
Deleting tmp directories ... done
done
done
done
done
done
done
done
Deleting old backups ... skipping
usernamexxx@hostnameyyy:~$ 
```

备份后大小：

```
usernamexxx@hostnameyyy:~$ sudo ls -ahl /var/opt/gitlab/backups/
total 938M
drwx------  2 git  root 4.0K 1月  10 20:40 .
drwxr-xr-x 18 root root 4.0K 12月 13 19:37 ..
-rw-------  1 git  git  469M 1月  10 14:46 1515566815_2018_01_10_10.2.4_gitlab_backup.tar
-rw-------  1 git  git  469M 1月  10 20:40 1515588043_2018_01_10_10.2.4_gitlab_backup.tar
```

备份文件路径配置：

```
usernamexxx@hostnameyyy:~$ sudo cat /etc/gitlab/gitlab.rb | grep backup
###! Docs: https://docs.gitlab.com/omnibus/settings/backups.html
# gitlab_rails['manage_backup_path'] = true
# gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"
```

这只是备份到本地，建议把 tar 包备份到其他主机。
