
## 编译步骤

* 安装依赖软件包，在 Ubuntu 14.04 上的依赖软件包有：
  + screen
  + chrpath
  + device_tree_compiler
  + gawk
  + makeinfo，软件包名字是 `texinfo`
```
sudo apt-get install screen chrpath device_tree_compiler gawk texinfo
```
* 下载 `ops-build`
```
git clone https://git.openswitch.net/openswitch/ops-build [<directory>]
cd <directory>
```
* 配置 openswitch 产品
```
make configure genericx86-64
```
* 编译产品
```
make
```
* 部署产品
XXX: todo
