---
date: 2014-06-20
title: "[cocos2dx]利用NDK崩溃日志查找BUG"
tags: 
- cocos2dx
-  ndk
-  c++
-  logcat
-  ndk-stack
categories: 
- 技术
---

摘要: 在android上开发c++应用, crash日志都是汇编码, 很难对应到c++代码中去. 通过此文, 你可以定位到程序崩溃时的C++代码, 精确查找问题.

### 背景介绍

1. 本文主要内容: 利用android的crash log来对c++开发的android应用进行错误定位.  

2. 容易稳定复现的BUG, 一般可以通过断点调试来解决. 如果测试人员也无法稳定复现, **log就成了程序吊定位问题的救命稻草**. 

3. 通用操作系统都有自己的日志系统, android也不例外. 救命稻草已经给你了~ ( [怎样查看android的系统日志](#how_to_refer_android_log) )  

4. 但是, android的系统日志在c++代码崩溃时, 打印的都是内存地址和寄存器. 比如, 这样:

```
06-20 15:54:35.331 23889 23889 I DEBUG   : *** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
06-20 15:54:35.331 23889 23889 I DEBUG   : Build fingerprint: 'google/razorg/deb:4.4.2/KOT49H/937116:user/release-keys'
06-20 15:54:35.331 23889 23889 I DEBUG   : Revision: '0'
06-20 15:54:35.331 23889 23889 I DEBUG   : pid: 1981, tid: 2020, name: Thread-3399  >>> com.guangyou.ddgame <<<
06-20 15:54:35.331 23889 23889 I DEBUG   : signal 11 (SIGSEGV), code 1 (SEGV_MAPERR), fault addr 00000028
06-20 15:54:35.431   187   710 D audio_hw_primary: out_set_parameters: enter: usecase(0: deep-buffer-playback) kvpairs: routing=2
06-20 15:54:35.511 23889 23889 I DEBUG   :     r0 76d94458  r1 00000000  r2 00000000  r3 00000000
06-20 15:54:35.511 23889 23889 I DEBUG   :     r4 760c1a48  r5 751e2440  r6 00000001  r7 760c1a48
06-20 15:54:35.511 23889 23889 I DEBUG   :     r8 00000001  r9 76c96f3c  sl 76c861c0  fp 76d94444
06-20 15:54:35.511 23889 23889 I DEBUG   :     ip 00000001  sp 76d94430  lr 75a81bd8  pc 75a81bdc  cpsr 600f0010
06-20 15:54:35.511 23889 23889 I DEBUG   :     d0  746968775f327865  d1  6a6e6169642f675f
06-20 15:54:35.511 23889 23889 I DEBUG   :     d2  5f6f616978757169  d3  676e702e6e776f6d
06-20 15:54:35.511 23889 23889 I DEBUG   :     d4  0000000009000000  d5  0000000000000000
06-20 15:54:35.511 23889 23889 I DEBUG   :     d6  0000000000000000  d7  0000000000000000
```


这密密麻麻的都是些神马, 是人看的么?

饿. 这个麻... 谁让你当程序猿! 让你当! 活该要看天书! 

硬着头皮也要来, 我们就来讲讲怎么消化天书吧~

### 怎样获取android的系统日志<a id="how_to_refer_android_log"></a>

 假设你已经安装了 Android Develop Tools, 可以成功调用`adb`. 并打开android开发用机的调试模式, 连接到电脑.
 
 打开命令行, 在命令行输入: `adb logcat`. 就可以看到满屏幕的日志啦. 
 输入`adb logcat --help`可以看到 `logcat`的用法提示.  
 

 
 这里有两个参数特别提醒一下, 比较常用:
    1. `-v XXXX`: 用来选择log输出样式, 一般建议 `threadtime`, 更加详细.
    2. `-d`: 让log一次性输出后马上完毕. 如果没有此命令, `logcat` 工具会一直输出, 即使更新在界面上. 
    
如果需要保存log到文件, 方便以后查看. 可输入命令:
`adb logcat -v threadtime -d > log.txt`


 
### 理解NDK的crash log
如果你用c++开发的android应用在运行过程中, c++代码发生错误导致程序崩溃, 系统就会记录 crash log到上述的系统日志中. 

下面是我正在开发的游戏一次崩溃后, 截取的日志( 插个广告, 全名斗地主下载地址: http://sj.ddwan.com )

```
06-20 15:54:35.331 23889 23889 I DEBUG   : *** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
06-20 15:54:35.331 23889 23889 I DEBUG   : Build fingerprint: 'google/razorg/deb:4.4.2/KOT49H/937116:user/release-keys'
06-20 15:54:35.331 23889 23889 I DEBUG   : Revision: '0'
06-20 15:54:35.331 23889 23889 I DEBUG   : pid: 1981, tid: 2020, name: Thread-3399  >>> com.guangyou.ddgame <<<
06-20 15:54:35.331 23889 23889 I DEBUG   : signal 11 (SIGSEGV), code 1 (SEGV_MAPERR), fault addr 00000028
06-20 15:54:35.431   187   710 D audio_hw_primary: out_set_parameters: enter: usecase(0: deep-buffer-playback) kvpairs: routing=2
06-20 15:54:35.511 23889 23889 I DEBUG   :     r0 76d94458  r1 00000000  r2 00000000  r3 00000000
06-20 15:54:35.511 23889 23889 I DEBUG   :     r4 760c1a48  r5 751e2440  r6 00000001  r7 760c1a48
06-20 15:54:35.511 23889 23889 I DEBUG   :     r8 00000001  r9 76c96f3c  sl 76c861c0  fp 76d94444
06-20 15:54:35.511 23889 23889 I DEBUG   :     ip 00000001  sp 76d94430  lr 75a81bd8  pc 75a81bdc  cpsr 600f0010
06-20 15:54:35.511 23889 23889 I DEBUG   :     d0  746968775f327865  d1  6a6e6169642f675f
06-20 15:54:35.511 23889 23889 I DEBUG   :     d2  5f6f616978757169  d3  676e702e6e776f6d
06-20 15:54:35.511 23889 23889 I DEBUG   :     d4  0000000009000000  d5  0000000000000000
06-20 15:54:35.511 23889 23889 I DEBUG   :     d6  0000000000000000  d7  0000000000000000
06-20 15:54:35.511 23889 23889 I DEBUG   :     d8  0000000000000000  d9  0000000000000000
06-20 15:54:35.511 23889 23889 I DEBUG   :     d10 0000000000000000  d11 0000000000000000
06-20 15:54:35.511 23889 23889 I DEBUG   :     d12 0000000000000000  d13 0000000000000000
06-20 15:54:35.511 23889 23889 I DEBUG   :     d14 0000000000000000  d15 0000000000000000
06-20 15:54:35.511 23889 23889 I DEBUG   :     d16 c3c3c3c3c3c3c3c3  d17 c3c3c3c3c3c3c3c3
06-20 15:54:35.511 23889 23889 I DEBUG   :     d18 41c7ddc227000000  d19 3ff0000000000000
06-20 15:54:35.511 23889 23889 I DEBUG   :     d20 3f811110896efbb2  d21 3fd7096611460fdb
06-20 15:54:35.511 23889 23889 I DEBUG   :     d22 c0176a8ee0000000  d23 bfc5230c760b0605
06-20 15:54:35.511 23889 23889 I DEBUG   :     d24 0000000000000000  d25 3fc7922925a107e2
06-20 15:54:35.511 23889 23889 I DEBUG   :     d26 3fdaa0f8fab43e33  d27 3fb43ad076b251ab
06-20 15:54:35.511 23889 23889 I DEBUG   :     d28 3fa15cb6bdc3c156  d29 3ec6cd878c3b46a7
06-20 15:54:35.511 23889 23889 I DEBUG   :     d30 3f65f3b6b9b97e01  d31 3ef99342e0ee5069
06-20 15:54:35.511 23889 23889 I DEBUG   :     scr 20000012
06-20 15:54:35.511 23889 23889 I DEBUG   :
06-20 15:54:35.511 23889 23889 I DEBUG   : backtrace:
06-20 15:54:35.511 23889 23889 I DEBUG   :     #00  pc 0089cbdc  /data/app-lib/com.guangyou.ddgame-1/libcocos2dcpp.so (cocos2d::Texture2D::getContentSize() const+32)
06-20 15:54:35.511 23889 23889 I DEBUG   :     #01  pc 0088f8dc  /data/app-lib/com.guangyou.ddgame-1/libcocos2dcpp.so (cocos2d::Sprite::setTexture(std::string const&)+128)
06-20 15:54:35.511 23889 23889 I DEBUG   :     #02  pc 007863dc  /data/app-lib/com.guangyou.ddgame-1/libcocos2dcpp.so (cocos2d::ui::Button::loadTextureDisabled(std::string const&, cocos2d::ui::Widget::TextureResType)+336)
06-20 15:54:35.511 23889 23889 I DEBUG   :
06-20 15:54:35.511 23889 23889 I DEBUG   : stack:
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d943f0  00000001
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d943f4  4006bc0d  /system/lib/libc.so (free+12)
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d943f8  76a72c54
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d943fc  75eca614  /data/app-lib/com.guangyou.ddgame-1/libcocos2dcpp.so
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d94400  751c23c8  [anon:libc_malloc]
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d94404  751c23c8  [anon:libc_malloc]
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d94408  751c23c8  [anon:libc_malloc]
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d9440c  75a749b4  /data/app-lib/com.guangyou.ddgame-1/libcocos2dcpp.so (cocos2d::Sprite::setTexture(cocos2d::Texture2D*)+128)
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d94410  0000003d
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d94414  00e8efc8
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d94418  00000000
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d9441c  00000000
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d94420  00000000
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d94424  76d94458  [stack:2020]
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d94428  00000020
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d9442c  76d94444  [stack:2020]
06-20 15:54:35.511 23889 23889 I DEBUG   :     #00  76d94430  00000000
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d94434  76d94458  [stack:2020]
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d94438  76a66184
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d9443c  760c1a48  /data/app-lib/com.guangyou.ddgame-1/libcocos2dcpp.so
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d94440  76d9447c  [stack:2020]
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d94444  75a748e0  /data/app-lib/com.guangyou.ddgame-1/libcocos2dcpp.so (cocos2d::Sprite::setTexture(std::string const&)+132)
06-20 15:54:35.511 23889 23889 I DEBUG   :     #01  76d94448  76d944ec  [stack:2020]
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d9444c  793ff0e8  [anon:libc_malloc]
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d94450  76a72c54
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d94454  00000000
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d94458  00000000
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d9445c  00000000
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d94460  00000000
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d94464  00000000
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d94468  00000000
06-20 15:54:35.511 23889 23889 I DEBUG   :          76d9446c  00000000
06-20 15:54:35.521 23889 23889 I DEBUG   :          76d94470  7b91dcf8  [anon:libc_malloc]
06-20 15:54:35.521 23889 23889 I DEBUG   :          76d94474  78ce6c50  [anon:libc_malloc]
06-20 15:54:35.521 23889 23889 I DEBUG   :          76d94478  76d944b4  [stack:2020]
06-20 15:54:35.521 23889 23889 I DEBUG   :          76d9447c  7596b3e0  /data/app-lib/com.guangyou.ddgame-1/libcocos2dcpp.so (cocos2d::ui::Button::loadTextureDisabled(std::string const&, cocos2d::ui::Widget::TextureResType)+340)
06-20 15:54:35.521 23889 23889 I DEBUG   :     #02  76d94480  00000001
06-20 15:54:35.521 23889 23889 I DEBUG   :          76d94484  00000000
06-20 15:54:35.521 23889 23889 I DEBUG   :          76d94488  76d944ec  [stack:2020]
06-20 15:54:35.521 23889 23889 I DEBUG   :          76d9448c  793fe780  [anon:libc_malloc]
06-20 15:54:35.521 23889 23889 I DEBUG   :          76d94490  76d944f0  [stack:2020]
06-20 15:54:35.521 23889 23889 I DEBUG   :          76d94494  793ff0e8  [anon:libc_malloc]
06-20 15:54:35.521 23889 23889 I DEBUG   :          76d94498  00000001
06-20 15:54:35.521 23889 23889 I DEBUG   :          76d9449c  4006bc0d  /system/lib/libc.so (free+12)
06-20 15:54:35.521 23889 23889 I DEBUG   :          76d944a0  76a72c54
06-20 15:54:35.521 23889 23889 I DEBUG   :          76d944a4  75eca614  /data/app-lib/com.guangyou.ddgame-1/libcocos2dcpp.so
06-20 15:54:35.521 23889 23889 I DEBUG   :          76d944a8  78ce6c50  [anon:libc_malloc]
06-20 15:54:35.521 23889 23889 I DEBUG   :          76d944ac  78ce6c50  [anon:libc_malloc]
06-20 15:54:35.521 23889 23889 I DEBUG   :          76d944b0  76d9455c  [stack:2020]
06-20 15:54:35.521 23889 23889 I DEBUG   :          76d944b4  75924e54  /data/app-lib/com.guangyou.ddgame-1/libcocos2dcpp.so (cocostudio::ButtonReader::setPropsFromJsonDictionary(cocos2d::ui::Widget*, rapidjson::GenericValue<rapidjson::UTF8<char>, rapidjson::MemoryPoolAllocator<rapidjson::CrtAllocator> > const&)+752)
06-20 15:54:35.521 23889 23889 I DEBUG   :          76d944b8  00000000
06-20 15:54:35.521 23889 23889 I DEBUG   :          76d944bc  78ce6c50  [anon:libc_malloc]
06-20 15:54:35.521 23889 23889 I DEBUG   :
06-20 15:54:35.521 23889 23889 I DEBUG   : memory near r0:
06-20 15:54:35.521 23889 23889 I DEBUG   :     76d94438 76a66184 760c1a48 76d9447c 75a748e0
06-20 15:54:35.521 23889 23889 I DEBUG   :     76d94448 76d944ec 793ff0e8 76a72c54 00000000
...
06-20 15:54:35.521 23889 23889 I DEBUG   :
06-20 15:54:35.521 23889 23889 I DEBUG   : memory near r4:
06-20 15:54:35.521 23889 23889 I DEBUG   :     760c1a28 760811c8 75ee318c 75ee3194 75ee319c
06-20 15:54:35.521 23889 23889 I DEBUG   :     760c1a38 4006d091 75f9a1f4 75f4ee5c 75e8ea0c
...
```

下面来逐行解读:
1. ndk crash log以`*** *** *** *** ***`开始.
2. 第一行`Build fingerprint: 'google/razorg/deb:4.4.2/KOT49H/937116:user/release-keys'` 指明了运行的Android版本, 如果您有多份crash dump的话这个信息就比较有用了.
3. 接着一行显示的是当前的线程id(pid)和进程id(tid). 如果当前崩溃的线程是主线程的话, pid和tid会是一样的~
4. 第四行, 显示的是unix信号. 这里的`signal 11`, 即`SIGSEGV`, 表示段错误, 是最常见的信号.([什么是unix信号](http://zh.wikipedia.org/wiki/%E4%BF%A1%E5%8F%B7_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6)), [什么是`SIGSEGV`](http://zh.wikipedia.org/wiki/SIGSEGV))
5. 接下来的部分是系统寄存器的dump信息.  
|符号|解释|
|:---------------|-----------|
|rX(X=[0~9])| 代表整数寄存器|
|dX(X=[0~31])| 是浮点指针寄存器|
|fp (or r11)  | 指向当前正在执行的函数的堆栈底.|
|ip (or r12)  | 一个寄存器, 我也没弄明白是干啥的.|
|sp (or r13)  | 当前正在执行的函数的堆栈顶.(跟fp相对应)|
|lr (or r14)  | [link register](http://en.wikipedia.org/wiki/Link_register). 简单来说, 当当前指令执行完了, <br/>就会从这个寄存器获取地址, 来知道需要返回<br/>到哪里继续执行.|
|pc (or r15)  | program counter. 存放下一条指令的地址|
|cpsr         | Current Program Status Register. 表示当前<br/>运行环境和状态的一些字节位.|
6. Crash dump还包含PC之前和之后的一些内存字段.
7. 最后, 是崩溃时的调用堆栈. 如果你执行的是debug版本, 还能还原一些c++代码.

### 利用ndk-stack定位崩溃代码
上面的一些信息能简单的帮你定位以下问题. 如果信息量还不够大的话, 那就还有最后一招: 还原历史.

`Android NDK`自从版本R6开始, 提供了一个工具`ndk-stack`( 在目录`{ndk_root}/`中 ). 这个工具能自动分析dump下来的crash log, 将崩溃时的调用内存地址和c++代码一行一行对应起来.

我们先看一下用法, 执行命令`ndk-stack --help`

```
Usage:
   ndk-stack -sym <path> [-dump <path>]

      -sym  Contains full path to the root directory for symbols.
      -dump Contains full path to the file containing the crash dump.
            This is an optional parameter. If ommited, ndk-stack will
            read input data from stdin

```
* -dump参数很容易理解, 即dump下来的log文本文件. `ndk-stack`会分析此文件.
* -sym参数就是你android项目下,编译成功之后,`obj`目录下的文件.

下面我们就来示范一下:

```
$ adb logcat | ndk-stack -sym ./obj/local/armeabi
********** Crash dump: **********
Build fingerprint: 'htc_wwe/htc_bravo/bravo:2.3.3/
GRI40/96875.1:user/release-keys'
pid: 1723, tid: 1743  >>> com.packtpub.droidblaster <<<
signal 11 (SIGSEGV), code 1 (SEGV_MAPERR), fault addr 0000000c
Stack frame #00  pc 00010a2c  /data/data/com.packtpub.droidblaster/lib/libdroidblaster.so: Routine update in /home/packt/Project/Chapter11/DroidBlaster_Part11/jni/TimeService.cpp:25
Stack frame #01  pc 00009fcc  /data/data/com.packtpub.droidblaster/lib/libdroidblaster.so: Routine onStep in /home/packt/Project/Chapter11/DroidBlaster_Part11/jni/DroidBlaster.cpp:53
Stack frame #02  pc 0000a348  /data/data/com.packtpub.droidblaster/lib/libdroidblaster.so: Routine run in /home/packt/Project/Chapter11/DroidBlaster_Part11/jni/EventLoop.cpp:49
Stack frame #03  pc 0000f994  /data/data/com.packtpub.droidblaster/lib/libdroidblaster.so: Routine android_main in /home/packt/Project/Chapter11/DroidBlaster_Part11/jni/Main.cpp:31
...
```

熟悉的代码出现啦~~








