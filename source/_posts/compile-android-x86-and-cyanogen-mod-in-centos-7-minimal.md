---
title: 解决64位CentOS 7 Minimal环境下的安卓编译问题
tags:
  - Linux
  - 安卓
id: 13
categories:
  - 安卓
date: 2016-01-13 17:14:22
---

前段时间攒了个下载机，装的是64位 CentOS 7（<span class="s1">CentOS Linux release 7.2.1511</span>），Minimal最小化安装。把以前老机器上的 Android-x86 源码和 C<span class="s1">yanogenMod Android 的源码拷了份过来，配置一下编译环境，在此处记录一下。</span>

#### 1 . 编译 android x86

首先按照android官网编译环境的配置，安装一些需要的工具：

注意官方给出的是ubuntu的工具名，我们这里安装64位Centos对应的工具。

<span style="color: #0000ff;">yum install flex.x86_64</span>
<span style="color: #0000ff;">yum install bison.x86_64</span>
<span style="color: #0000ff;">yum install gperf.x86_64</span>

接下来，说一下JDK的问题。我系统用的是 Oracle JDK 8，版本太高，编译过程中提示只能用 JDK6或者7。找了个JDK 7，发现还是报 “<span style="color: #ff0000;">error: ordinal has private access in Enum</span>” 错误，最后换成JDK 6，就可以顺利通过。

编译过程使用与系统环境不同的JDK配置，编译启动脚本如下：
`
#!/usr/bin/env bash
ANDROID_HOME=~/jdks/jdk1.6.0_45
PATH=$ANDROID_HOME/bin:$PATH
nohup make -j4 iso_img TARGET_PRODUCT=android_x86 &gt; build.log 2&gt;&amp;1 &amp;
`
下面针对编译过程中出现的各种问题错误，依次解决如下：

*   <span style="color: #ff0000;">错误：prebuilts/misc/linux-x86/ccache/ccache: /lib/ld-linux.so.2: bad ELF interpreter: No such file or directory</span>
<span style="color: #008000;">解决：yum install glibc.i686  （64位环境中的32位库）</span>

*   <span style="color: #ff0000;">错误：bc: command not found</span>
<span style="color: #008000;">解决：yum install bc.x86_64</span>

*   <span style="color: #ff0000;">错误：error while loading shared libraries: libgcc_s.so.1: cannot open shared object file: No such file or directory</span>
<span style="color: #008000;">解决：yum install libgcc.i686 （64位环境中的32位库）</span>

*   <span style="color: #ff0000;">错误：error while loading shared libraries: libz.so.1: cannot open shared object file: No such file or directory</span>
<span style="color: #008000;">解决：yum install libzip.i686 （64位环境中的32位库）</span>

*   <span style="color: #ff0000;">错误：error while loading shared libraries: libstdc++.so.6: cannot open shared object file: No such file or directory</span>
<span style="color: #008000;">解决：yum install libstdc++.i686 （64位环境中的32位库，这个依赖libgcc.i686，所以装这个就会装上libgcc.i686）</span>

*   <span style="color: #ff0000;">错误：ImportError: No module named mako.template</span>
<span style="color: #008000;">解决：yum install python-mako.noarch</span>

*   <span style="color: #ff0000;">错误：patch: command not found</span>
<span style="color: #008000;">解决：yum install patch.x86_64</span>

*   <span style="color: #ff0000;">错误：bash: mkisofs: command not found</span>
<span style="color: #008000;">解决：yum install genisoimage.x86_64</span>
至此，Android X86已经可以编译通过，生成可安装的 Android x86 ISO。

#### 2 . 编译 CyanogenMod Android

在上面成功编译完 Android x86 配置的基础上，编译 CyanogenMod 的时候，会遇到的问题及解决：

*   <span class="s1" style="color: #ff0000;">错误：build/envsetup.sh: line 1934: schedtool: command not found 或者 </span><span class="s1" style="color: #ff0000;">bash: schedtool: command not found</span>
<span style="color: #008000;">解决：原本想的和上面一样 yum install schedtool轻松解决问题的，结果告诉我 <span class="s1">Warning: No matches found for: schedtool，</span>schedtool在yum库里找不到，只好去找源码安装。</span>
<span style="color: #008000;"> 代码下载页：http://slackbuilds.org/repository/13.37/system/schedtool/</span>
<span style="color: #008000;"> 直接下载地址：http://freequaos.host.sk/schedtool/schedtool-1.3.0.tar.bz2</span>
<span style="color: #008000;"> 校验码 (0d968f05d3ad7675f1f33ef1f6d0a3fb)</span>
<span style="color: #008000;"> 解压缩，tar -xjf schedtool-1.3.0.tar.bz2</span>
<span style="color: #008000;"> （如果像我一样，不默认装进系统目录，而只是安装进开发者账号目录里，则打开Makefile，修改 <span class="s1">DESTPREFIX</span><span class="s2">=/home/xxx/developlocal）</span></span>
<span style="color: #008000;"> 然后执行 make &amp; make install，完成安装。</span>
然后按照正常流程，执行<span class="s1">breakfast mako ，接着 </span><span class="s1">brunch mako，就顺利完成编译，得到对应的系统镜像文件了。</span>

&nbsp;

&nbsp;