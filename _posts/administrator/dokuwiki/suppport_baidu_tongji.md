title: dokuwiki 支持百度统计
date: 2017-04-14 14:11:20
toc: true
tags: [dokuwiki, administrator]
categories: administrator
keywords: [administrator, dokuwiki, 百度统计, baidu, google, analytics, bootstrap3]
description: dokuwiki 支持百度统计记录
---

dokuwiki 支持 [plugin:googleanalytics](https://www.dokuwiki.org/plugin:googleanalytics)，template [bootstrap3 模板](https://www.dokuwiki.org/template:bootstrap3) 也支持 google analytics。不过众所周知的原因，天朝内无法使用 google 的服务，因此考虑如何支持百度统计。

百度统计添加新网站后，得到统计代码：

```
<script>
var _hmt = _hmt || [];
(function() {
  var hm = document.createElement("script");
  hm.src = "https://hm.baidu.com/hm.js?xxxxxxxxxxxxxxxxxxxxxxxxxx";
  var s = document.getElementsByTagName("script")[0]; 
  s.parentNode.insertBefore(hm, s);
})();
</script>
```

百度官方建议的代码安装步骤：

```
1. 请将代码添加到网站全部页面的</head>标签前。
2. 建议在header.htm类似的页头模板页面中安装，以达到一处安装，全站皆有的效果。
3. 如需在JS文件中调用统计分析代码，请直接去掉以下代码首尾的，<script type="text/javascript">与</script>后，放入JS文件中即可。
如果代码安装正确，一般20分钟后，可以查看网站分析数据。
```

因此本文主要考虑如何将百度统计代码插入到 `<head>` 标签中。

本文使用 template:bootstrap3 ，该模板的 HTML Hooks 一节提示可通过 `meta.html` 添加内容到 `<head>` 内部。

> Inside the HTML `<head>`, use this to add additional styles or metaheaders

不过经过尝试在 `/var/www/dokuwiki/lib/tpl/bootstrap3/extra/hooks/meta.html` 加入统计代码，点开网页看源码，并未查看到统计代码。
考虑到该模板支持 google analytics，因此直接以 `<head>` 为关键字搜索：

```
sunyongfeng@ubuntu:/var/www/dokuwiki/lib/tpl/bootstrap3$ grep -r "<head>"  
mediamanager.php:<head>
main.php:<head>
detail.php:<head>
```

从文件名可直接排队 mediamanager.php，查看 main.php 和 detail.php 及 dokuwiki 某个页面的源码，发现 main.php 为想要的目标文件。

main.php 部分内容：

```php
/**
 * DokuWiki Bootstrap3 Template
 *
 * @link     http://dokuwiki.org/template:bootstrap3
 * @author   Giuseppe Di Terlizzi <giuseppe.diterlizzi@gmail.com>
 * @license  GPL 2 (http://www.gnu.org/licenses/gpl.html)
 */

if (!defined('DOKU_INC')) die();                        // must be run from within DokuWiki
@require_once(dirname(__FILE__).'/tpl_functions.php');  // include hook for template functions
include_once(dirname(__FILE__).'/tpl_global.php');      // Include template global variables
header('X-UA-Compatible: IE=edge,chrome=1');
?><!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="<?php echo $conf['lang'] ?>"
  lang="<?php echo $conf['lang'] ?>" dir="<?php echo $lang['direction'] ?>" class="no-js">
<head>
  <meta charset="UTF-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge" />
  <title><?php echo bootstrap3_page_browser_title() ?></title>
  <script>(function(H){H.className=H.className.replace(/\bno-js\b/,'js')})(document.documentElement)</script>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <?php echo tpl_favicon(array('favicon', 'mobile')) ?>
  <?php tpl_includeFile('meta.html') ?>
  <?php tpl_metaheaders() ?>
  <?php bootstrap3_google_analytics() ?>
  <!--[if lt IE 9]>
  <script type="text/javascript" src="https://oss.maxcdn.com/html5shiv/3.7.2/html5shiv.min.js"></script>
  <script type="text/javascript" src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
  <![endif]-->
</head>
```

某个 wiki 页面的源码：

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="zh"
  lang="zh" dir="ltr" class="no-js">
<head>
  <meta charset="UTF-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge" />
  <title>wiki:markdown语法 [交换机技术平台部 wiki]</title>
  <script>(function(H){H.className=H.className.replace(/\bno-js\b/,'js')})(document.documentElement)</script>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <link rel="shortcut icon" href="/dokuwiki/lib/tpl/bootstrap3/images/favicon.ico" />
<link rel="apple-touch-icon" href="/dokuwiki/lib/tpl/bootstrap3/images/apple-touch-icon.png" />
    <meta name="generator" content="DokuWiki"/>
<meta name="robots" content="index,follow"/>
<meta name="keywords" content="wiki,markdown语法"/>
<link type="text/css" rel="stylesheet" href="/dokuwiki/lib/tpl/bootstrap3/assets/bootstrap/united/bootstrap.min.css"/>
<link type="text/css" rel="stylesheet" href="/dokuwiki/lib/tpl/bootstrap3/assets/fonts/united.fonts.css"/>
<link rel="search" type="application/opensearchdescription+xml" href="/dokuwiki/lib/exe/opensearch.php" title="交换机技术平台部 wiki"/>
<link rel="start" href="/dokuwiki/"/>
<link rel="contents" href="/dokuwiki/doku.php/wiki:markdown语法?do=index" title="网站地图"/>
<link rel="alternate" type="application/rss+xml" title="最近更改" href="/dokuwiki/feed.php"/>
<link rel="alternate" type="application/rss+xml" title="当前命名空间" href="/dokuwiki/feed.php?mode=list&amp;ns=wiki"/>
<link rel="edit" title="编辑本页" href="/dokuwiki/doku.php/wiki:markdown语法?do=edit"/>
<link rel="alternate" type="text/html" title="纯HTML" href="/dokuwiki/doku.php/wiki:markdown%E8%AF%AD%E6%B3%95?do=export_xhtml"/>
<link rel="alternate" type="text/plain" title="Wiki Markup 语言" href="/dokuwiki/doku.php/wiki:markdown%E8%AF%AD%E6%B3%95?do=export_raw"/>
<link rel="canonical" href="http://172.18.111.192:10090/dokuwiki/doku.php/wiki:markdown语法"/>
<link rel="stylesheet" type="text/css" href="/dokuwiki/lib/exe/css.php?t=bootstrap3&amp;tseed=397ac3e12611e6079f8bfb104ef97c20"/>
<link type="text/css" rel="stylesheet" href="/dokuwiki/lib/tpl/bootstrap3/assets/font-awesome/css/font-awesome.min.css"/>
<!--[if gte IE 9]><!-->
<script type="text/javascript">/*<![CDATA[*/var NS='wiki';var SIG=' --- //[[sunyongfeng@ruijie.com.cn|孙勇峰]] 2017/04/14 22:20//';var JSINFO = {"id":"wiki:markdown\u8bed\u6cd5","namespace":"wiki","isadmin":1,"isauth":1,"bootstrap3":{"mode":"show","config":{"collapsibleSections":0,"sidebarOnNavbar":1,"tagsOnTop":1,"tocAffix":1,"tocCollapseOnScroll":1,"tocCollapsed":0,"showSemanticPopup":0}}};
/*!]]>*/</script>
<script type="text/javascript" charset="utf-8" src="/dokuwiki/lib/exe/jquery.php?tseed=23f888679b4f1dc26eef34902aca964f"></script>
<script type="text/javascript" charset="utf-8" src="/dokuwiki/lib/exe/js.php?t=bootstrap3&amp;tseed=397ac3e12611e6079f8bfb104ef97c20"></script>
<script type="text/javascript" src="/dokuwiki/lib/tpl/bootstrap3/assets/bootstrap/js/bootstrap.min.js"></script>
<script type="text/javascript" src="/dokuwiki/lib/tpl/bootstrap3/assets/anchorjs/anchor.min.js"></script>
<script type="text/javascript" src="/dokuwiki/lib/tpl/bootstrap3/assets/bootstrap/js/bootstrap-hover-dropdown.min.js"></script>
<script type="text/javascript">/*<![CDATA[*/jQuery(document).ready(function() { jQuery('body').scrollspy({ target: '#dw__toc', offset: 30 });jQuery("#dw__toc").affix({ offset: { top: (jQuery("main").position().top), bottom: (jQuery(document).height() - jQuery("main").height()) } });jQuery(document).trigger('bootstrap3:anchorjs'); });
/*!]]>*/</script>
<!--<![endif]-->
<style type="text/css">@media screen { body { padding-top: 20px; } #dw__toc.affix { top: 10px; position: fixed !important; } #dw__toc .nav .nav .nav { display: none; }}</style>
    <!--[if lt IE 9]>
  <script type="text/javascript" src="https://oss.maxcdn.com/html5shiv/3.7.2/html5shiv.min.js"></script>
  <script type="text/javascript" src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
  <![endif]-->
</head>
```

故在 main.php 的 `</head>` 前添加百度代码。重新加载 wiki 页面，发现百度统计代码已添加。过几分钟后，在百度统计网页上，可以看到统计信息。

修改后的 main.php：

```php
/**
 * DokuWiki Bootstrap3 Template
 *
 * @link     http://dokuwiki.org/template:bootstrap3
 * @author   Giuseppe Di Terlizzi <giuseppe.diterlizzi@gmail.com>
 * @license  GPL 2 (http://www.gnu.org/licenses/gpl.html)
 */

if (!defined('DOKU_INC')) die();                        // must be run from within DokuWiki
@require_once(dirname(__FILE__).'/tpl_functions.php');  // include hook for template functions
include_once(dirname(__FILE__).'/tpl_global.php');      // Include template global variables
header('X-UA-Compatible: IE=edge,chrome=1');
?><!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="<?php echo $conf['lang'] ?>"
  lang="<?php echo $conf['lang'] ?>" dir="<?php echo $lang['direction'] ?>" class="no-js">
<head>
  <meta charset="UTF-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge" />
  <title><?php echo bootstrap3_page_browser_title() ?></title>
  <script>(function(H){H.className=H.className.replace(/\bno-js\b/,'js')})(document.documentElement)</script>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <?php echo tpl_favicon(array('favicon', 'mobile')) ?>
  <?php tpl_includeFile('meta.html') ?>
  <?php tpl_metaheaders() ?>
  <?php bootstrap3_google_analytics() ?>
  <!--[if lt IE 9]>
  <script type="text/javascript" src="https://oss.maxcdn.com/html5shiv/3.7.2/html5shiv.min.js"></script>
  <script type="text/javascript" src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
  <![endif]-->
  <script>
    var _hmt = _hmt || [];
    (function() {
      var hm = document.createElement("script");
      hm.src = "https://hm.baidu.com/hm.js?b3ec3db35b5308953eb2e2d1ba02d3c0";
      var s = document.getElementsByTagName("script")[0]; 
      s.parentNode.insertBefore(hm, s);
    })();
  </script>
</head>
```

修改后的 wiki 页面源码：

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="zh"
  lang="zh" dir="ltr" class="no-js">
<head>
  <meta charset="UTF-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge" />
  <title>wiki:markdown语法 [交换机技术平台部 wiki]</title>
  <script>(function(H){H.className=H.className.replace(/\bno-js\b/,'js')})(document.documentElement)</script>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <link rel="shortcut icon" href="/dokuwiki/lib/tpl/bootstrap3/images/favicon.ico" />
<link rel="apple-touch-icon" href="/dokuwiki/lib/tpl/bootstrap3/images/apple-touch-icon.png" />
    <meta name="generator" content="DokuWiki"/>
<meta name="robots" content="index,follow"/>
<meta name="keywords" content="wiki,markdown语法"/>
<link type="text/css" rel="stylesheet" href="/dokuwiki/lib/tpl/bootstrap3/assets/bootstrap/united/bootstrap.min.css"/>
<link type="text/css" rel="stylesheet" href="/dokuwiki/lib/tpl/bootstrap3/assets/fonts/united.fonts.css"/>
<link rel="search" type="application/opensearchdescription+xml" href="/dokuwiki/lib/exe/opensearch.php" title="交换机技术平台部 wiki"/>
<link rel="start" href="/dokuwiki/"/>
<link rel="contents" href="/dokuwiki/doku.php/wiki:markdown语法?do=index" title="网站地图"/>
<link rel="alternate" type="application/rss+xml" title="最近更改" href="/dokuwiki/feed.php"/>
<link rel="alternate" type="application/rss+xml" title="当前命名空间" href="/dokuwiki/feed.php?mode=list&amp;ns=wiki"/>
<link rel="edit" title="编辑本页" href="/dokuwiki/doku.php/wiki:markdown语法?do=edit"/>
<link rel="alternate" type="text/html" title="纯HTML" href="/dokuwiki/doku.php/wiki:markdown%E8%AF%AD%E6%B3%95?do=export_xhtml"/>
<link rel="alternate" type="text/plain" title="Wiki Markup 语言" href="/dokuwiki/doku.php/wiki:markdown%E8%AF%AD%E6%B3%95?do=export_raw"/>
<link rel="canonical" href="http://172.18.111.192:10090/dokuwiki/doku.php/wiki:markdown语法"/>
<link rel="stylesheet" type="text/css" href="/dokuwiki/lib/exe/css.php?t=bootstrap3&amp;tseed=397ac3e12611e6079f8bfb104ef97c20"/>
<link type="text/css" rel="stylesheet" href="/dokuwiki/lib/tpl/bootstrap3/assets/font-awesome/css/font-awesome.min.css"/>
<!--[if gte IE 9]><!-->
<script type="text/javascript">/*<![CDATA[*/var NS='wiki';var SIG=' --- //[[sunyongfeng@ruijie.com.cn|孙勇峰]] 2017/04/14 22:20//';var JSINFO = {"id":"wiki:markdown\u8bed\u6cd5","namespace":"wiki","isadmin":1,"isauth":1,"bootstrap3":{"mode":"show","config":{"collapsibleSections":0,"sidebarOnNavbar":1,"tagsOnTop":1,"tocAffix":1,"tocCollapseOnScroll":1,"tocCollapsed":0,"showSemanticPopup":0}}};
/*!]]>*/</script>
<script type="text/javascript" charset="utf-8" src="/dokuwiki/lib/exe/jquery.php?tseed=23f888679b4f1dc26eef34902aca964f"></script>
<script type="text/javascript" charset="utf-8" src="/dokuwiki/lib/exe/js.php?t=bootstrap3&amp;tseed=397ac3e12611e6079f8bfb104ef97c20"></script>
<script type="text/javascript" src="/dokuwiki/lib/tpl/bootstrap3/assets/bootstrap/js/bootstrap.min.js"></script>
<script type="text/javascript" src="/dokuwiki/lib/tpl/bootstrap3/assets/anchorjs/anchor.min.js"></script>
<script type="text/javascript" src="/dokuwiki/lib/tpl/bootstrap3/assets/bootstrap/js/bootstrap-hover-dropdown.min.js"></script>
<script type="text/javascript">/*<![CDATA[*/jQuery(document).ready(function() { jQuery('body').scrollspy({ target: '#dw__toc', offset: 30 });jQuery("#dw__toc").affix({ offset: { top: (jQuery("main").position().top), bottom: (jQuery(document).height() - jQuery("main").height()) } });jQuery(document).trigger('bootstrap3:anchorjs'); });
/*!]]>*/</script>
<!--<![endif]-->
<style type="text/css">@media screen { body { padding-top: 20px; } #dw__toc.affix { top: 10px; position: fixed !important; } #dw__toc .nav .nav .nav { display: none; }}</style>
    <!--[if lt IE 9]>
  <script type="text/javascript" src="https://oss.maxcdn.com/html5shiv/3.7.2/html5shiv.min.js"></script>
  <script type="text/javascript" src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
  <![endif]-->
  <script>
    var _hmt = _hmt || [];
    (function() {
      var hm = document.createElement("script");
      hm.src = "https://hm.baidu.com/hm.js?b3ec3db35b5308953eb2e2d1ba02d3c0";
      var s = document.getElementsByTagName("script")[0]; 
      s.parentNode.insertBefore(hm, s);
    })();
  </script>
</head>
```
