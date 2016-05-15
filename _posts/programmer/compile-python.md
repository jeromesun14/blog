交叉编译 python
===============

## 下载解压 python 源代码
## 配置交叉编译环境变量
比如 CC、CFLAGS、LDFLAGS 等。

## 配置
配置命令如下：
configure 的 prefix 只支持绝对路径。

```
./configure --host=mips64-octeon-linux-gnu --build=x86_64-linux-gnu prefix=/home/sunyongfeng/python-install --disable-ipv6 ac_cv_file__dev_ptmx=no ac_cv_file__dev_ptc=no ac_cv_have_long_long_format=yes
```

问题：
* `--enable-FEATURE`，不清楚有哪些 features ，怎么配置；
* `--enable-PACKAGE`，不清楚有哪些 package，怎么配置。

配置完了之后，在 Modules 目录会生成 Setup 文件。修改 `Modules/Setup` 文件，定制想编译的内置模块。以下是基础模块，目前还不清楚如果不想内置的话要如何编译。
定制内置模块，参见这篇博文 《[定制 Python 嵌入 C++: (四) 定制 Python 内建模块](http://www.adintr.com/article/blog/249)》，讲述各个内置模块的功能。

```
# Modules that should always be present (non UNIX dependent):                                       
                                                                                                    
array arraymodule.c # array objects                                                                 
cmath cmathmodule.c _math.c # -lm # complex math library functions                                  
math mathmodule.c _math.c # -lm # math library functions, e.g. sin()                                
_struct _struct.c   # binary structure packing/unpacking                                            
time timemodule.c # -lm # time operations and variables                                             
operator operator.c # operator.add() and similar goodies                                            
_testcapi _testcapimodule.c    # Python C API test module                                           
_random _randommodule.c # Random number generator                                                   
_collections _collectionsmodule.c # Container types                                                 
_heapq _heapqmodule.c       # Heapq type                                                            
itertools itertoolsmodule.c # Functions creating iterators for efficient looping                    
strop stropmodule.c     # String manipulations                                                      
_functools _functoolsmodule.c   # Tools for working with functions and callable objects             
_elementtree -I$(srcdir)/Modules/expat -DHAVE_EXPAT_CONFIG_H -DUSE_PYEXPAT_CAPI _elementtree.c  # elementtree accelerator
#_pickle _pickle.c  # pickle accelerator                                                            
datetime datetimemodule.c   # date/time type                                                        
_bisect _bisectmodule.c # Bisection algorithms                                                      
                                                                                                    
unicodedata unicodedata.c    # static Unicode character database
```

## 编译
简单的 `make` 命令即可。

## 安装
命令 `make install -i`，安装 `bin`、`lib`、`share`、`man` 等目录至 `./configure` 中配置的 `prefix` 目录。

```
sunyongfeng@R04220:~/python-install$ ls
bin  include  lib  share
sunyongfeng@R04220:~/python-install$ ls -al *
bin:
总用量 9612
drwxr-xr-x 2 sunyongfeng sunyongfeng    4096  5月 13 16:51 .
drwxr-xr-x 6 sunyongfeng sunyongfeng    4096  5月 15 10:58 ..
-rwxrwxr-x 1 sunyongfeng sunyongfeng     123  5月 13 16:38 2to3
-rwxrwxr-x 1 sunyongfeng sunyongfeng     121  5月 13 16:38 idle
-rwxrwxr-x 1 sunyongfeng sunyongfeng     106  5月 13 16:38 pydoc
lrwxrwxrwx 1 sunyongfeng sunyongfeng       7  5月 13 16:51 python -> python2
lrwxrwxrwx 1 sunyongfeng sunyongfeng       9  5月 13 16:51 python2 -> python2.7
-rwxr-xr-x 1 sunyongfeng sunyongfeng 9793952  5月 13 16:51 python2.7
-rwxr-xr-x 1 sunyongfeng sunyongfeng    1709  5月 13 16:51 python2.7-config
lrwxrwxrwx 1 sunyongfeng sunyongfeng      16  5月 13 16:51 python2-config -> python2.7-config
lrwxrwxrwx 1 sunyongfeng sunyongfeng      14  5月 13 16:51 python-config -> python2-config
-rwxrwxr-x 1 sunyongfeng sunyongfeng   18569  5月 13 16:38 smtpd.py

include:
总用量 12
drwxr-xr-x 3 sunyongfeng sunyongfeng 4096  5月 13 16:51 .
drwxr-xr-x 6 sunyongfeng sunyongfeng 4096  5月 15 10:58 ..
drwxr-xr-x 2 sunyongfeng sunyongfeng 4096  5月 13 16:51 python2.7

lib:
总用量 16312
drwxr-xr-x  4 sunyongfeng sunyongfeng     4096  5月 13 16:51 .
drwxr-xr-x  6 sunyongfeng sunyongfeng     4096  5月 15 10:58 ..
-r-xr-xr-x  1 sunyongfeng sunyongfeng 16670684  5月 13 16:51 libpython2.7.a
drwxr-xr-x  2 sunyongfeng sunyongfeng     4096  5月 13 16:51 pkgconfig
drwxr-xr-x 28 sunyongfeng sunyongfeng    20480  5月 13 16:51 python2.7

share:
总用量 12
drwxr-xr-x 3 sunyongfeng sunyongfeng 4096  5月 13 16:51 .
drwxr-xr-x 6 sunyongfeng sunyongfeng 4096  5月 15 10:58 ..
drwxr-xr-x 3 sunyongfeng sunyongfeng 4096  5月 13 16:51 man
sunyongfeng@R04220:~/python-install$ 
```

打包放到目标机上，配置目标机的 PATH，加上 python 的 bin 目录。

## 问题
### 编译依赖
交叉编译的时候，如果没有配置好 CFLAGS、LDFLAGS 之类的变量，可能找不到 python 编译所依赖的头文件或库文件。最终体现在编译的结果（此处可能因不同的变量配置而不同，比如 sqlite3、readline、_ssl 等）：

```
Python build finished, but the necessary bits to build these modules were not found:
_bsddb             _curses_panel      _tkinter        
bsddb185           bz2                dbm             
dl                 gdbm               imageop         
linuxaudiodev      ossaudiodev        sunaudiodev     
To find the necessary bits, look in setup.py in detect_modules() for the module's name.
```

* sqlite3 依赖配置
修改 python 源码根目录下的 setup.py 文件，在 `detect_modules` 函数下，找到 sqlite3 的头文件配置，添加上交叉编译下的 sqlite3 头文件目录。
```
        sqlite_inc_paths = [ '/usr/include',                                                        
                             '/usr/include/sqlite',                                                 
                             '/usr/include/sqlite3',                                                
                             '/usr/local/include',                                                  
                             '/usr/local/include/sqlite',                                           
                             '/usr/local/include/sqlite3',                                          
                           ]                                                                        
        if cross_compiling:                                                                         
            sqlite_inc_paths = [ '/home/sunyongfeng/workshop/prj/images/header/',
                                 '/home/sunyongfeng/workshop/prj/images/header/sqlite',
                                 '/home/sunyongfeng/workshop/prj/images/header/sqlite3',
                               ] 
```

* ssl 依赖配置
类似 sqlite3，在 setup.py 文件的 `detect_modules` 函数下，找到 ssl 相关的头文件与库文件配置，添加上交叉编译下的 ssl 头文件与库文件目录。

```
        # Detect SSL support for the socket module (via _ssl)                                       
        search_for_ssl_incs_in = [                                                                  
                              '/usr/local/ssl/include',                                             
                              '/usr/contrib/ssl/include/',                                          
                              '/home/sunyongfeng/workshop/prj/images/header/',
                             ]                                                                      
        ssl_incs = find_file('openssl/ssl.h', inc_dirs,                                             
                             search_for_ssl_incs_in                                                 
                             )                                                                      
        if ssl_incs is not None:                                                                    
            krb5_h = find_file('krb5.h', inc_dirs,                                                  
                               ['/usr/kerberos/include'])                                           
            if krb5_h:                                                                              
                ssl_incs += krb5_h                                                                  
        ssl_libs = find_library_file(self.compiler, 'ssl',lib_dirs,                                 
                                     ['/usr/local/ssl/lib',                                         
                                      '/usr/contrib/ssl/lib/',                                      
                                      '/home/sunyongfeng/workshop/prj/images/rootfs/lib64'
                                     ] ) 
```

## 附一：Python 内建模块功能说明
直接引自 [定制 Python 嵌入 C++: (四) 定制 Python 内建模块](http://www.adintr.com/article/blog/249)，内容可能已过时，不过有参考价值。

1. array (Modules/arraymodule.c) (http://docs.python.org/library/array.html) 一个可以存放基本类型的高效数组, 提供了和序列类似的操作. 使用放法类似于 a = array.array('b', [10, 20, 30]), 不常使用, 可以考虑去除. 
2. _ast (Python/Python-ast.c) (http://docs.python.org/library/ast.html) 抽象语法树, 供 Python 程序解析处理 Python 语法相关的库, 这个模块的源代码是由脚本自动生成的. 由于 Python-ast.c 本身还会被解释器的其它地方引用, 不能删除, 所以, 如果是为了压缩解释器大小, 保留这个库也没关系. 如果是为了定制 python 的功能, 也可以屏蔽这个库, 但是源代码需要保留, 不能从工程中删掉. 
3. audioop (Modules/audioop.c) (http://docs.python.org/library/audioop.html) 一个音频处理的库, 仅 Win32 平台有效.
4. binascii (Modules/binascii.c) (http://docs.python.org/library/binascii.html) 提供二进制和 ASCII 码的转换, 会被 uu, base64, binhex 这些库引用. 建议保留.
5. cmath (Modules/cmathmodule.c) (http://docs.python.org/library/cmath.html) 提供复数操作的函数
6. errno (Modules/errnomodule.c) (http://docs.python.org/library/errno.html) 提供标准的错误码定义, 在很多地方中都会使用, 需要保留.
7. future_builtins (Modules/future_builtins.c) (http://docs.python.org/library/future_builtins.html) 对那些在 Python2.x 和 Python3 中都有但是意义不一样的函数提供的包装. 使用这里面的函数可以保证调用了正确的版本的函数.
8. gc (Modules/gcmodule.c) (http://docs.python.org/library/gc.html) Python 的垃圾收集接口. 当然保留.
9. imageop (Modules/imageop.c) (http://docs.python.org/library/imageop.html) 一些图像处理的函数.
10. math (Modules/mathmodule.c) (http://docs.python.org/library/math.html) 提供了 C 标准库中的那些数学函数.
11. _md5 (Modules/md5module.c) 提供了 MD5 算法.
12. nt (Modules/posixmodule.c) 一些操作系统习惯的函数, 比如打开文件等等.
13. operator (Modules/operator.c) (http://docs.python.org/library/operator.html) 提供了操作符的等价函数
14. signal (Modules/signalmodule.c) (http://docs.python.org/library/signal.html) 信号机制, 提供异步事件的回调.
15. _sha, _sha256, _sha512 三种 SHA 的加密算法模块.
16. strop (Modules/stropmodule.c) 提供了一些优化的字符串操作.
17.time (Modules/timemodule.c) (http://docs.python.org/library/time.html) 时间操作库.
18. thread (Modules/threadmodule.c) Python 线程的底层模块, threading 会使用 thread 库.
19. cStringIO (Modules/cStringIO.c) (http://docs.python.org/library/stringio.html) StringIO 的高效版本.
20. cPickle (Modules/cPickle.c) (http://docs.python.org/library/pickle.html) Python 的序列化模块.
21. msvcrt (PC/msvcrtmodule.c) (http://docs.python.org/library/msvcrt.html) VC 运行时库的包装, 包括一些文件和屏幕操作函数.
22. _locale (Modules/_localemodule.c) 提供本地化支持的模块.
23. _subprocess (PC/_subprocess.c) (http://docs.python.org/library/subprocess.html) 操作子进程的库, 平台相关的.
24. _codecs (Modules/_codecsmodule.c)  (http://docs.python.org/library/codecs.html) 定义了 Python 的编码器相关接口.
25. _weakref (Modules/_weakref.c) (http://docs.python.org/library/weakref.html) 创建对象的弱引用.
26. _hotshot (Modules/_hotshot.c) (http://docs.python.org/library/hotshot.html) 类似于 Profiler 模块, 而且将来可能被移除, 现在把它去掉也不错.
27. _random (Modules/_randommodule.c) 随机数模块.
28. _bisect (Modules/_bisectmodule.c) (http://docs.python.org/library/bisect.html)  一个基于二分算法, 可以让插入一个数据岛排序的序列后序列仍然有序的库.
29. _heapq (Modules/_heapqmodule.c) (http://docs.python.org/library/heapq.html) 实现堆栈数据结构算法的库.
30. _lsprof (Modules/_lsprof.c) (http://docs.python.org/library/profile.html) Profiler 模块, 统计程序执行的性能.
31. itertools (Modules/itertoolsmodule.c) (http://docs.python.org/library/itertools.html) 一些迭代器操作的模块.
32. _collections (Modules/_collectionsmodule.c) (http://docs.python.org/library/collections.html) 提供了几个高级的容器类.
33. _symtable (Modules/symtablemodule.c) (http://docs.python.org/library/symtable.html) 符号表管理模块.
34. mmap (Modules/mmapmodule.c) (http://docs.python.org/library/mmap.html) 文件内存映射支持模块.
35. _csv (Modules/_csv.c) (http://docs.python.org/library/csv.html) 为 CSV 模块的内部支持. CSV 模块提供了读写 CSV 文件的功能.
36. _sre (Modules/_sre.c)  正则表达式的匹配引擎.
37. parser (Modules/parsermodule.c) (http://docs.python.org/library/parser.html) 操作 Python 语法树的模块.
38. _winreg (PC/_winreg.c) Windows 注册表操作模块.
39. _struct (Modules/_struct.c) 提供在 Python 和 C 之间转换数据类型的功能.
40. datetime (Modules/datetimemodule.c) (http://docs.python.org/library/datetime.html) 日期时间操作函数.
41. _functools (Modules/_functoolsmodule.c) (http://docs.python.org/library/functools.html) 函数相关操作模块.
42. _json (Modules/_json.c) (http://docs.python.org/library/json.html) JSON 数据格式操作模块.
43. xxsubtype (Modules/xxsubtype.c)  这是一个测试相关的模块. 运行 test_descr.py 时会用到.
44. zipimport (Modules/zipimport.c) 这个模块主要用于从 zip 文件中导入 Python 的模块.
45. zlib (Modules/zlibmodule.c) 这个模块提供了 zip 压缩和解压功能, 基于 GNU zip 实现.
46. _multibytecodec, _codecs_cn, _codecs_hk, _codecs_iso2022, _codecs_jp, _codecs_kr, _codecs_tw (Modules/cjkcodecs/*) 这些模块提供了 CJK(中日韩统一表意文字) 的编码和解码. 去掉这部分可以减小 python 解释器 600 多 K.
47. marshal (Python/marshal.c) (http://docs.python.org/library/marshal.html) 为 Python 对象提供序列化的模块.
48. imp (Python/import.c) (http://docs.python.org/library/imp.html) 这个模块提供了 Python 里的 import 语句的实现.
49. __main__, __builtin__, sys, exceptions, _warnings 这部分模块在 config.c 设置里只是一个名字占位符.
50. _io (Modules/_iomodule.c) (http://docs.python.org/library/io.html) 新版本的 Python 输入输出模块, 在 Python 3 中为默认的输入输出处理方法.

## 附二：Python 最佳编译依赖
直接译自 [Python Deployment](https://gist.github.com/reorx/4067217)。

键入如下命令自动安装一些依赖：
```
$ sudo apt-get build-dep python2.7
```

确认安装如下列下的其他 -dev 包。
* python-dev
* libncurses5-dev
* libsqlite3-dev
* libbz2-dev
* libreadline-dev
* libdb4.8-dev
* tcl8.5-dev,tk8.5-dev

下面这个包在 ubuntu 早期版本（如 10.04）并没有自动安装，需确认一下。
* libssl-dev
* libexpat1-dev
* libreadline6-dev
* libgtk2.0-dev

如果想支持 xml 相关：
* libxml2-dev
*libxslt1-dev

如果想支持 MySQLdb （在 pypi 中实际命令为 MySQL-python）：
* libmysqlclient-dev

最终的 make 结果（编译结果）如能如下：

```
Python build finished, but the necessary bits to build these modules were not found:
_tkinter           bsddb185           dl
gdbm               imageop            sunaudiodev
To find the necessary bits, look in setup.py in detect_modules() for the module's name.
```

这个编译 log 提示哪些模块没有被编译到，注意其中有一些并不是必需的或过时的：
* bsddb185: Older version of Oracle Berkeley DB. Undocumented. Install version 4.8 instead.
* dl: For 32-bit machines. Deprecated. Use ctypes instead.
* imageop: For 32-bit machines. Deprecated. Use PIL instead.
* sunaudiodev: For Sun hardware. Deprecated.
* _tkinter: For tkinter graphy library, unnecessary if you don't develop tkinter programs.

## 参考文献
* [定制 Python 嵌入 C++: (四) 定制 Python 内建模块](http://www.adintr.com/article/blog/249)
* [build python 2.7.11 for mips](http://jyhshin.pixnet.net/blog/post/45780376-build-python-2.7.11-for-mips)
* [Python Deployment](https://gist.github.com/reorx/4067217)
