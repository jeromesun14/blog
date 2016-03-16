
# 安装 `hexo`

* 安装 `git`
* 安装 `nodejs`，debian / ubuntu 类的环境：`sudo apt-get install npm nodejs-legacy`
如果安装的是 `node` 包，则 `sudo npm install -g hexo-cli` 时可能出现 error ：

```
npm http GET https://registry.npm.taobao.org/inherits

> spawn-sync@1.0.15 postinstall /usr/local/lib/node_modules/hexo-cli/node_modules/hexo-util/node_modules/cross-spawn/node_modules/spawn-sync
> node postinstall

npm WARN This failure might be due to the use of legacy binary "node"
npm WARN For further explanations, please read
/usr/share/doc/nodejs/README.Debian

npm ERR! weird error 1
```

