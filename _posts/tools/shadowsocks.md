
title: shadowsocks 记录
date: 2016-02-28 20:48:18
toc: true
tags: [外面的世界, administrator]
categories: tools, administrator
keywords: [shadowsocks, linux, chrome, switchyOmega, proxychains]
description: 到外面的世界看看。
---

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
这是一个 [gfwlist](https://github.com/calfzhou/autoproxy-gfwlist)。

## socks5 转 http 代理
详见 http://xuhehuan.com/2119.html。

使用 `polipo`，用 `apt-get` 可安装，配置如下：
```
# This file only needs to list configuration variables that deviate
# from the default values.  See /usr/share/doc/polipo/examples/config.sample
# and "polipo -v" for variables you can tweak and further information.
logSyslog = false
logFile = "/var/log/polipo/polipo.log"

socksParentProxy = "127.0.0.1:1080"
socksProxyType = socks5

chunkHighMark = 50331648
objectHighMark = 16384

serverMaxSlots = 64
serverSlots = 16
serverSlots1 = 32

proxyAddress = "0.0.0.0"
proxyPort = 8123
```

* 重启 polipo，`sudo service polipo restart`
* 验证 sock5 转 http 是否 ok：

```
export http_proxy="http://127.0.0.1:8123/"
curl www.google.com
```

ok 的话返回网页抓取结果：
```
~$ curl www.google.com.hk                    
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html><head>
<title>Proxy error: 502 Read from server failed: Connection reset by peer.</title>
</head><body>
<h1>502 Read from server failed: Connection reset by peer</h1>
<p>The following error occurred while trying to access <strong>http://www.google.com.hk/</strong>:<br><br>
<strong>502 Read from server failed: Connection reset by peer</strong></p>
<hr>Generated Tue, 08 Nov 2016 15:53:18 CST by Polipo on <em>openswitch-OptiPlex-380:10080</em>.
</body></html>
sunyongfeng@openswitch-OptiPlex-380:~$ 
sunyongfeng@openswitch-OptiPlex-380:~$ curl www.google.com.hk
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html><head>
<title>Proxy error: 502 Read from server failed: Connection reset by peer.</title>
</head><body>
<h1>502 Read from server failed: Connection reset by peer</h1>
<p>The following error occurred while trying to access <strong>http://www.google.com.hk/</strong>:<br><br>
<strong>502 Read from server failed: Connection reset by peer</strong></p>
<hr>Generated Tue, 08 Nov 2016 15:55:03 CST by Polipo on <em>openswitch-OptiPlex-380:10080</em>.
</body></html>
sunyongfeng@openswitch-OptiPlex-380:~$ curl www.google.com.hk
<!doctype html><html itemscope="" itemtype="http://schema.org/WebPage" lang="zh-HK"><head><meta content="text/html; charset=UTF-8" http-equiv="Content-Type"><meta content="/images/branding/googleg/1x/googleg_standard_color_128dp.png" itemprop="image"><title>Google</title><script>(function(){window.google={kEI:'PoUhWJLcI4nW0gLCrrCoCQ',kEXPI:'750722,1351828,1351903,1352148,3700062,3700245,4026241,4028875,4029815,4031109,4032677,4036527,4038012,4039268,4043491,4045841,4048347,4065786,4067860,4068550,4069839,4069841,4071842,4072289,4072364,4072602,4072705,4072775,4073405,4073415,4073486,4073498,4073959,4074684,4076095,4076542,4076931,4076999,4078241,4078407,4078430,4078438,4078454,4078456,4078607,4079105,4079553,4079624,4079870,4080115,4080161,4080165,4080815,4081037,4081038,4081128,4081263,4081278,4081497,4082112,4082219,4082290,4082310,4082441,4082618,4082638,4082778,4083064,4083115,4083281,4083353,4084066,4084258,4084259,4084277,4084325,4084343,4084366,4084402,4084676,4084691,4084956,4085014,4085057,4085066,4085181,4085628,4085917,8300272,8502184,8503585,8504846,8504935,8505150,8505152,8505259,8506255,8506340,8506477,8506615,8507098,8507361,8507381,8507842,8507846,8508176,8508241,8508317,10200083',authuser:0,kscs:'c9c918f0_24'};google.kHL='zh-HK';})();(function(){google.lc=[];google.li=0;google.getEI=function(a){for(var b;a&&(!a.getAttribute||!(b=a.getAttribute("eid")));)a=a.parentNode;return b||google.kEI};google.getLEI=function(a){for(var b=null;a&&(!a.getAttribute||!(b=a.getAttribute("leid")));)a=a.parentNode;return b};google.https=function(){return"https:"==window.location.protocol};google.ml=function(){return null};google.wl=function(a,b){try{google.ml(Error(a),!1,b)}catch(c){}};google.time=function(){return(new Date).getTime()};google.log=function(a,b,c,e,g){a=google.logUrl(a,b,c,e,g);if(""!=a){b=new Image;var d=google.lc,f=google.li;d[f]=b;b.onerror=b.onload=b.onabort=function(){delete d[f]};window.google&&window.google.vel&&window.google.vel.lu&&window.google.vel.lu(a);b.src=a;google.li=f+1}};google.logUrl=function(a,b,c,e,g){var d="",f=google.ls||"";if(!c&&-1==b.search("&ei=")){var h=google.getEI(e),d="&ei="+h;-1==b.search("&lei=")&&((e=google.getLEI(e))?d+="&lei="+e:h!=google.kEI&&(d+="&lei="+google.kEI))}a=c||"/"+(g||"gen_204")+"?atyp=i&ct="+a+"&cad="+b+d+f+"&zx="+google.time();/^http:/i.test(a)&&google.https()&&(google.ml(Error("a"),!1,{src:a,glmm:1}),a="");return a};google.y={};google.x=function(a,b){google.y[a.id]=[a,b];return!1};google.lq=[];google.load=function(a,b,c){google.lq.push([[a],b,c])};google.loadAll=function(a,b){google.lq.push([a,b])};})();var a=window.location,b=a.href.indexOf("#");if(0<=b){var c=a.href.substring(b+1);/(^|&)q=/.test(c)&&-1==c.indexOf("#")&&a.replace("/search?"+c.replace(/(^|&)fp=[^&]*/g,"")+"&cad=h")};</script><style>#gbar,#guser{font-size:13px;padding-top:1px !important;}#gbar{height:22px}#guser{padding-bottom:7px !important;text-align:right}.gbh,.gbd{border-top:1px solid #c9d7f1;font-size:1px}.gbh{height:0;position:absolute;top:24px;width:100%}@media all{.gb1{height:22px;margin-right:.5em;vertical-align:top}#gbar{float:left}}a.gb1,a.gb4{text-decoration:underline !important}a.gb1,a.gb4{color:#00c !important}.gbi .gb4{color:#dd8e27 !important}.gbf .gb4{color:#900 !important}
</style><style>body,td,a,p,.h{font-family:arial,sans-serif}body{margin:0;overflow-y:scroll}#gog{padding:3px 8px 0}td{line-height:.8em}.gac_m td{line-height:17px}form{margin-bottom:20px}.h{color:#36c}.q{color:#00c}.ts td{padding:0}.ts{border-collapse:collapse}em{color:#c03;font-style:normal;font-weight:normal}a em{text-decoration:underline}.lst{height:25px;width:496px}.gsfi,.lst{font:18px arial,sans-serif}.gsfs{font:17px arial,sans-serif}.ds{display:inline-box;display:inline-block;margin:3px 0 4px;margin-left:4px}input{font-family:inherit}a.gb1,a.gb2,a.gb3,a.gb4{color:#11c !important}body{background:#fff;color:black}a{color:#11c;text-decoration:none}a:hover,a:active{text-decoration:underline}.fl a{color:#36c}a:visited{color:#551a8b}a.gb1,a.gb4{text-decoration:underline}a.gb3:hover{text-decoration:none}#ghead a.gb2:hover{color:#fff !important}.sblc{padding-top:5px}.sblc a{display:block;margin:2px 0;margin-left:13px;font-size:11px}.lsbb{background:#eee;border:solid 1px;border-color:#ccc #999 #999 #ccc;height:30px}.lsbb{display:block}.ftl,#fll a{display:inline-block;margin:0 12px}.lsb{background:url(/images/nav_logo229.png) 0 -261px repeat-x;border:none;color:#000;cursor:pointer;height:30px;margin:0;outline:0;font:15px arial,sans-serif;vertical-align:top}.lsb:active{background:#ccc}.lst:focus{outline:none}</style><script></script><link href="/images/branding/product/ico/googleg_lodp.ico" rel="shortcut icon"></head><body bgcolor="#fff"><script>(function(){var src='/images/nav_logo229.png';var iesg=false;document.body.onload = function(){window.n && window.n();if (document.images){new Image().src=src;}
if (!iesg){document.f&&document.f.q.focus();document.gbqf&&document.gbqf.q.focus();}
}
})();</script><div id="mngb"> <div id=gbar><nobr><b class=gb1></b> <a class=gb1 href="http://www.google.com.hk/imghp?hl=zh-TW&tab=wi"></a> <a class=gb1 href="http://maps.google.com.hk/maps?hl=zh-TW&tab=wl"></a> <a class=gb1 href="https://play.google.com/?hl=zh-TW&tab=w8">Play</a> <a class=gb1 href="http://www.youtube.com/?gl=HK&tab=w1">YouTube</a> <a class=gb1 href="http://news.google.com.hk/nwshp?hl=zh-TW&tab=wn"></a> <a class=gb1 href="https://mail.google.com/mail/?tab=wm">Gmail</a> <a class=gb1 href="https://drive.google.com/?tab=wo"></a> <a class=gb1 style="text-decoration:none" href="https://www.google.com.hk/intl/zh-TW/options/"><u></u> &raquo;</a></nobr></div><div id=guser width=100%><nobr><span id=gbn class=gbi></span><span id=gbf class=gbf></span><span id=gbe></span><a href="http://www.google.com.hk/history/optout?hl=zh-TW" class=gb4></a> | <a  href="/preferences?hl=zh-TW" class=gb4></a> | <a target=_top id=gb_70 href="https://accounts.google.com/ServiceLogin?hl=zh-TW&passive=true&continue=http://www.google.com.hk/" class=gb4></a></nobr></div><div class=gbh style=left:0></div><div class=gbh style=right:0></div> </div><center><br clear="all" id="lgpd"><div id="lga"><div style="padding:28px 0 3px"><div style="height:110px;width:276px;background:url(/images/branding/googlelogo/1x/googlelogo_white_background_color_272x92dp.png) no-repeat" title="Google" align="left" id="hplogo" onload="window.lol&&lol()"><div style="color:#777;font-size:16px;font-weight:bold;position:relative;top:70px;left:218px" nowrap=""></div></div></div><br></div><form action="/search" name="f"><table cellpadding="0" cellspacing="0"><tr valign="top"><td width="25%">&nbsp;</td><td align="center" nowrap=""><input name="ie" value="Big5" type="hidden"><input value="zh-HK" name="hl" type="hidden"><input name="source" type="hidden" value="hp"><input name="biw" type="hidden"><input name="bih" type="hidden"><div class="ds" style="height:32px;margin:4px 0"><input style="color:#000;margin:0;padding:5px 8px 0 6px;vertical-align:top" autocomplete="off" class="lst" value="" title="Google " maxlength="2048" name="q" size="57"></div><br style="line-height:0"><span class="ds"><span class="lsbb"><input class="lsb" value="Google " name="btnG" type="submit"></span></span><span class="ds"><span class="lsbb"><input class="lsb" value="name="btnI" onclick="if(this.form.q.value)this.checked=1; else top.location='/doodles/'" type="submit"></span></span></td><td class="fl sblc" align="left" nowrap="" width="25%"><a href="/advanced_search?hl=zh-HK&amp;authuser=0"></a><a href="/language_tools?hl=zh-HK&amp;authuser=0"></a></td></tr></table><input id="gbv" name="gbv" type="hidden" value="1"></form><div id="gac_scont"></div><div style="font-size:83%;min-height:3.5em"><br><div id="als"><style>#als{font-size:small;margin-bottom:24px}#_eEe{display:inline-block;line-height:28px;}#_eEe a{padding:0 3px;}._lEe{display:inline-block;margin:0 2px;white-space:nowrap}._PEe{display:inline-block;margin:0 2px}</style><div id="_eEe">Google.com.hk <a href="http://www.google.com.hk/setprefs?sig=0_rRVcKKfvrZgCB_A43UF1XQ9TPmw%3D&amp;hl=zh-CN&amp;source=homepage" data-ved="0ahUKEwiS85jb15jQAhUJq1QKHUIXDJUQ2ZgBCAU">(&#31616;/a>  <a href="http://www.google.com.hk/setprefs?sig=0_rRVcKKfvrZgCB_A43UF1XQ9TPmw%3D&amp;hl=en&amp;source=homepage" data-ved="0ahUKEwiS85jb15jQAhUJq1QKHUIXDJUQ2ZgBCAY">English</a> </div></div></div><span id="footer"><div style="font-size:10pt"><div style="margin:19px auto;text-align:center" id="fll"><a href="/intl/zh-TW/ads/"></a><a href="/intl/zh-TW/about.html">Google </a><a href="http://www.google.com.hk/setprefdomain?prefdom=US&amp;sig=__A6fXbQceC3SkqMu_FRt85xzWjWI%3D" id="fehl">Google.com</a></div></div><p style="color:#767676;font-size:8pt">&copy; 2016 - <a href="/intl/zh-TW/policies/privacy/">a> - <a href="/intl/zh-TW/policies/terms/"></a></p></span></center><script>(function(){window.google.cdo={height:0,width:0};(function(){var a=window.innerWidth,b=window.innerHeight;if(!a||!b)var c=window.document,d="CSS1Compat"==c.compatMode?c.documentElement:c.body,a=d.clientWidth,b=d.clientHeight;a&&b&&(a!=google.cdo.width||b!=google.cdo.height)&&google.log("","","/client_204?&atyp=i&biw="+a+"&bih="+b+"&ei="+google.kEI);})();})();</script><div id="xjsd"></div><div id="xjsi"><script>(function(){function c(b){window.setTimeout(function(){var a=document.createElement("script");a.src=b;document.getElementById("xjsd").appendChild(a)},0)}google.dljp=function(b,a){google.xjsu=b;c(a)};google.dlj=c;})();(function(){window.google.xjsrm=[];})();if(google.y)google.y.first=[];if(!google.xjs){window._=window._||{};window._._DumpException=function(e){throw e};if(google.timers&&google.timers.load.t){google.timers.load.t.xjsls=new Date().getTime();}google.dljp('/xjs/_/js/k\x3dxjs.hp.en_US.HHTiPh8WO1g.O/m\x3dsb_he,d/rt\x3dj/d\x3d1/t\x3dzcms/rs\x3dACT90oFyHK2p3Cq9BefJNT1WLwsFHb2Nvg','/xjs/_/js/k\x3dxjs.hp.en_US.HHTiPh8WO1g.O/m\x3dsb_he,d/rt\x3dj/d\x3d1/t\x3dzcms/rs\x3dACT90oFyHK2p3Cq9BefJNT1WLwsFHb2Nvg');google.xjs=1;}google.pmc={"sb_he":{"agen":true,"cgen":true,"client":"heirloom-hp","dh":true,"dhqt":true,"ds":"","fl":true,"host":"google.com.hk","isbh":28,"jam":0,"msgs":{"cibl":"","dym":""lcky":""lml":"","oskt":"","psrc":"003Ca href=\"/history\"\u003E\u003C/a\u003E","psrl":"","sbit":"","srch":"Google "},"nds":true,"ovr":{},"pq":"","refpd":true,"refspre":true,"rfs":[],"scd":10,"sce":5,"stok":"ahSc6O6Rc51lCmCOKZYjJdw8hyI"},"d":{}};google.y.first.push(function(){if(google.med){google.med('init');google.initHistory();google.med('history');}});if(google.j&&google.j.en&&google.j.xi){window.setTimeout(google.j.xi,0);}
```

