title: Linux kernel 编译问题记录
date: 2017-01-10 17:31:00
toc: true
tags: [kernel, compile]
categories: kernel
keywords: [Linux, kernel, compile, perl, timeconst.pl, failed]
description: Linux 编译问题汇总记录。
---

## Can't use 'defined(@array)'

Linux kernel 无法编译通过，报了如下错误：

```
Can't use 'defined(@array)' (Maybe you should just omit the defined()?) at kernel/timeconst.pl line 373. 
```

`kernel/timeconst.pl` 中的代码：

```
@val = @{$canned_values{$hz}}; 
if (defined(@val)) { 
@val = compute_values($hz); 
} 
output($hz, @val); 
```

将`if (defined(@val))`改为`if (@val)`，再次编译就 OK。

原因：perl版本升级到 v5.22.1，发现官网因为一个bug，将defined(@array)去掉了。可以直接使用数组判断非空。

来自：http://blog.5ibc.net/p/48570.html
