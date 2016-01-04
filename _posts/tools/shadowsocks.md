

## shadowsocks in linux

* install, `pip install shadowsocks`
* start shadowsocks, `sslocal -d start -p server_port -k your_password -s server_url(or server_ip) -m aes-256-cfb -l local_port`

## proxychains

* install, `sudo apt-get install proxychains `
* configure, `sudo vi /etc/proxychains.conf`, add line `socks5  127.0.0.1 1080` to tail
* use proxy for app, `sudo proxychains your_app`

