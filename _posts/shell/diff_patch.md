diff 与 patch
=============

diff -ruN xxx_src xxx_dst > xxx.patch


patch

http://812lcl.com/blog/2014/04/03/linuxming-ling-xue-xi-(1)%3Adiffhe-patch/
http://linux-wiki.cn/wiki/%E8%A1%A5%E4%B8%81(patch)%E7%9A%84%E5%88%B6%E4%BD%9C%E4%B8%8E%E5%BA%94%E7%94%A8
http://www.ruanyifeng.com/blog/2012/08/how_to_read_diff.html

`patch [参数] <patchfile>`

参数：
* -p Num  忽略几层文件夹
* -E      选项说明如果发现了空文件，那么就删除它
* -R      取消打过的补丁。

如果使用参数-p0，表示从当前目录找打补丁的目标文件夹，再对该目录中的文件执行patch操作。 而使用参数-p1，表示忽略第一层目录，从当前目录寻找目标文件夹中的子目录和文件，进行patch操作。

产生补丁
```
diff -uN f1 f2 > file.patch
```
打补丁
```
patch -p0 < file.patch
```
或者
```
patch f1 file.patch
```
取消补丁
```
patch -RE -p0 < file.patch
```
或者
```
patch -RE f1 file.patch
```
