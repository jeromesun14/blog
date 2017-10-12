title: dokuwiki 中文支持
date: 2017-04-01 22:25:30
toc: true
tags: [dokuwiki, administrator]
categories: dokuwiki
keywords: [administrator, dokuwiki, 中文乱码, Url rewrite, URL 重写]
description: 记录 dokuwiki 中文相关支持
---

# 中文名 page 被转为 url 保存到磁盘
**注意：** 每次升級 dokuwiki 都必须做“问题解决”一节中第2步操作。

元凶：URL rewrite。

## bug 现象
页面 `public:linux工具箱` 在磁盘中被保存为：

```
dokuwiki/data/pages/public/linux%E5%B7%A5%E5%85%B7%E7%AE%B1/
```

处理后：

```
dokuwiki/data/pages/public/linux工具箱/
```

## 问题解决

参考文献：

* [勿慢牛 blog](https://blog.klniu.com/post/tag/dokuwiki/)
* [页面名和命名空间](https://www.dokuwiki.org/start?id=zh:pagename)

修订记录：

1.修改文件 `dokuwiki/conf/local.php`，在末尾增加

```
$conf['fnencode'] = 'gbk';
```

2.修改文件 `dokuwiki/inc/pageutils.php`，删除掉文件名转 url 的操作。

处理完这一步，可验证一下。添加个中文 page，到 data 目录查看是否保存为中文名。

```
function utf8_encodeFN($file,$safe=true){
    global $conf;
    if($conf['fnencode'] == 'utf-8') return $file;

    if($safe && preg_match('#^[a-zA-Z0-9/_\-\.%]+$#',$file)){
        return $file;
    }

    if($conf['fnencode'] == 'safe'){
        return SafeFN::encode($file);
    }

    // 注释这里
    //$file = urlencode($file);
    //$file = str_replace('%2F','/',$file);
    return $file;
}

/**
 * Decode a filename back to UTF-8
 *
 * Uses the 'fnencode' option to determine encoding
 *
 * @author Andreas Gohr <andi@splitbrain.org>
 * @see    urldecode
 *
 * @param string $file file name
 * @return string
 */
function utf8_decodeFN($file){
    global $conf;
    if($conf['fnencode'] == 'utf-8') return $file;

    if($conf['fnencode'] == 'safe'){
        return SafeFN::decode($file);
    }

    // 注释这里
    // return urldecode($file);
    return $file;
}
```

3.恢复原先以 url 格式保存的文件名为中文名
以 python3 运行如下脚本，注意修改 wikipath 为 dokuwiki 路径。实测在 linux 下可用。

```
"""
dokuwiki转码程序。（请将本程序保存为utf8文本）
作用：将dokuwiki默认的编码方式编码生成的目录名、文件名，统一转换为可识别的中文。
要求python版本大于等于3.4
"""
 
from pathlib import Path
from urllib.parse import unquote
 
wikipath = 'd:/lzweb/wiki' # 请在这里设置好dokuwiki的安装目录
 
rootpath = Path(wikipath)
rootpath = rootpath / 'data'
 
def pathRename(path): # 对path进行转码
    newname = path.parent / unquote(path.name)
    path.rename(newname)
 
def dealpath(path): # 对path下的全部文件和目录进行递归编码转换
    allpath = path.glob('*') # 遍历path下的第一层目录
    for p in allpath:
        if p.is_file(): pathRename(p)
        elif p.is_dir(): # 如果是目录
            dealpath(p) # 先对目录下的所有内容改名
            pathRename(p) # 再对该目录改名
 
print('转码开始，请耐心等候...')
dealpath(rootpath)
print('完成全部转换。')
```

4.解决 sitemap 乱码问题，修改 `dokuwiki/inc/common.php`

```
function wl($id = '', $urlParameters = '', $absolute = false, $separator = '&amp;') {
    global $conf;
    if(is_array($urlParameters)) {
        if(isset($urlParameters['rev']) && !$urlParameters['rev']) unset($urlParameters['rev']);
        if(isset($urlParameters['at']) && $conf['date_at_format']) $urlParameters['at'] = date($conf['date_at_format'],$urlParameters['at']);
        $urlParameters = buildURLparams($urlParameters, $separator);
    } else {
        $urlParameters = str_replace(',', $separator, $urlParameters);
    }
    if($id === '') {
        $id = $conf['start'];
    }
    //注释这里
    //$id = idfilter($id);
    if($absolute) {
        $xlink = DOKU_URL;
    } else {
        $xlink = DOKU_BASE;
    }

    if($conf['userewrite'] == 2) {
        $xlink .= DOKU_SCRIPT.'/'.$id;
        if($urlParameters) $xlink .= '?'.$urlParameters;
    } elseif($conf['userewrite']) {
        $xlink .= $id;
        if($urlParameters) $xlink .= '?'.$urlParameters;
    } elseif($id) {
        $xlink .= DOKU_SCRIPT.'?id='.$id;
        if($urlParameters) $xlink .= $separator.$urlParameters;
    } else {
        $xlink .= DOKU_SCRIPT;
        if($urlParameters) $xlink .= '?'.$urlParameters;
    }

    return $xlink;
}
```

