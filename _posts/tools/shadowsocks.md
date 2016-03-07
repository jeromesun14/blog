

## shadowsocks in linux

* install, `pip install shadowsocks`
* start shadowsocks, `sslocal -d start -p server_port -k your_password -s server_url(or server_ip) -m aes-256-cfb -l local_port`

## proxychains

* install, `sudo apt-get install proxychains `
* configure, `sudo vi /etc/proxychains.conf`, add line `socks5  127.0.0.1 1080` to tail
* use proxy for app, `sudo proxychains your_app`

chromium 无法使用 proxychains，有个 [issue](https://github.com/rofl0r/proxychains-ng/issues/45)。

## switchyOmega

可在 Profiles 下的 auto switch 以 rule list 的形式加入自动切换的列表，在 `Rule List Config` 下填入 rule list 的链接即可。
这是一个 [gfwlist](https://github.com/calfzhou/autoproxy-gfwlist)

