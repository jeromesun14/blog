title: git 问题记录
date: 2016-02-18 20:48:18
toc: true
tags: [版本控制, git]
categories: tools
keywords: [git, error, git push]
description: 本文描述SVN的基础使用方法。
---

## `git push` 相关 ##

### error: RPC failed; result=22, HTTP code = 504

> # [Why am I getting a 504 error when accessing Git over HTTP?](http://help.projectlocker.com/knowledge_base/topics/why-am-i-getting-a-504-error-when-accessing-git-over-http)
>
> If you're getting a message of the form:
>
> error: RPC failed; result=22, HTTP code = 504
>
> When attempting to clone your Git repository over HTTP, it probably means that there's a size issue. Git is not designed to perform well over HTTP with repositories larger than about 1 GB. Over HTTP, Git has to be served using FastCGI and so the FastCGI process can sometimes time out before the command can return results. To your team, this looks like intermittent failure and some issue with our servers, but it's actually Git's inability to send the amounts of data in question fast enough.
>
> To work around this, if you need to have a large Git repository, we recommend accessing it via SSH using the instructions at:
>
> [How Do I Connect To My Git Repository?](http://help.projectlocker.com/knowledge_base/topics/how-do-i-connect-to-my-git-repository)
>
> You can email us as well at support [at] to open a support ticket and request that we run garbage collection on your server, which sometimes can free up enough space to make a difference.
>
> Alternatively, we do offer Subversion, which is great for hosting large artifact repositories for your team as well, and because it's not distributed, doesn't have as many issues with copying large amounts of data.


