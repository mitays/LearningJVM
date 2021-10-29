﻿[TOC]



# 一、编译环境及背景
本文是参照《深入理解Java虚拟机：JVM高级特性与最佳实践（第3版）》 周志明老师的书来实践学习的，站在巨人的肩膀上！！

以下是书中与实际操作的环境对比：
||实践环境|书中环境|
|--|--|--|
| 系统 | RHEL8|Ubuntu 18.04 LTS|
|JDK|JDK12_06222165c35f |JDK12_06222165c35f |
|本地运行测试代码 JDK|1.8.0_301|
因此，在实际操作中会与书中有差异，但差异代表着我需要对每一步操作有更深的理解，排除出现的问题，fighting！！！
`着重提一点，源代码目录/doc/building.html 文件或者 building.md 文件强烈推荐看一看，对于编译的理解以及处理不同操作系统环境准备非常有帮助`

|软件|版本|
|:--|:--|
|GCC|8.4.1-1.el8|
|FreeType|2.9.1-4.el8_3.1|
|CUPS|1:2.2.6-38.el8|
|X11|libXtst-devel-1.2.3-7.el8.x86_64；libXt-devel-1.1.5-12.el8.x86_64；libXrender-devel-0.9.10-7.el8.x86_64；libXrandr-devel-1.5.2-1.el8.x86_64；libXi-devel-1.7.10-1.el8.x86_64|
|ALSA|1.2.4-5.el8|
|libffi|3.1-22.el8|
|Autoconf|2.69-27.el8|


# 二、上手实操
我这里将编译 JDK 的步骤分为如下：
1. 下载 JDK 源码
2. 安装编译依赖环境与 Boot JDK
3. 编译操作

## 2.1 下载 JDK 源码
下载之前首先要区分一下：
[https://hg.openjdk.java.net/jdk/jdk12/file/06222165c35f](https://hg.openjdk.java.net/jdk/jdk12/file/06222165c35f) ：二进制源码
[https://jdk.java.net/java-se-ri/12](https://jdk.java.net/java-se-ri/12)：编译好的 JDK

1. 进入源码下载网站[https://hg.openjdk.java.net/](https://hg.openjdk.java.net/)，选择 jdk -> jdk12，点击 browse 查看到的即是源码目录；
2. 点击左侧的 zip/gz 来下载不同版本的源码压缩包；
![在这里插入图片描述](https://img-blog.csdnimg.cn/0f232ef5485d4d83beb1ef33f8112545.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbWl0YXlz,size_20,color_FFFFFF,t_70,g_se,x_16#pic_left)

| 压缩包 | 大小 |
|--|--|
|jdk12-06222165c35f.tar.gz  | 103M |
|jdk12-06222165c35f.zip|171M|

## 2.2 构建编译环境
环境分为 GCC 编译器、OpenJDK 编译依赖库、Bootstrap JDK，这一章节如果书中没有设计到你所用系统，请参阅`源码目录/doc/building.md`
### 2.2.1 GCC编译器
For apt-based distributions (Debian, Ubuntu, etc), try this:
```
sudo apt-get install build-essential
```

For rpm-based distributions (Fedora, Red Hat, etc), try this:
```
sudo yum groupinstall "Development Tools"
```
### 2.2.2 OpenJDK编译依赖库
|工具|库名称|
|:--|:--|
|FreeType|The FreeType Project|
|CUPS|Common UNIX Printing System|
|X11|X Window System|
|ALSA|Advanced Linux Sound Architecture|
|libffi|Portable Foreign Function Interface Library|
|Autoconf|Extensible Package of M4 Macros|
安装命令如下，根据系统版本选择安装


FreeType
> * To install on an apt-based Linux, try running `sudo apt-get install
    libfreetype6-dev`.
  	>* To install on an rpm-based Linux, try running `sudo yum install
    freetype-devel`.
  	>* To install on Solaris, try running `pkg install system/library/freetype-2`.

CUPS
>  * To install on an apt-based Linux, try running `sudo apt-get install
    libcups2-dev`.
>  * To install on an rpm-based Linux, try running `sudo yum install
    cups-devel`.
>* To install on Solaris, try running `pkg install print/cups`.

X11
>* To install on an apt-based Linux, try running `sudo apt-get install
    libx11-dev libxext-dev libxrender-dev libxrandr-dev libxtst-dev libxt-dev`.
>  * To install on an rpm-based Linux, try running `sudo yum install
      libXtst-devel libXt-devel libXrender-devel libXrandr-devel libXi-devel`.
>  * To install on Solaris, try running `pkg install x11/header/x11-protocols
      x11/library/libice x11/library/libpthread-stubs x11/library/libsm
      x11/library/libx11 x11/library/libxau x11/library/libxcb
      x11/library/libxdmcp x11/library/libxevie x11/library/libxext
      x11/library/libxrender x11/library/libxrandr x11/library/libxscrnsaver
      x11/library/libxtst x11/library/toolkit/libxt`.

ALSA
>  * To install on an apt-based Linux, try running `sudo apt-get install
    libasound2-dev`.
>  * To install on an rpm-based Linux, try running `sudo yum install
    alsa-lib-devel`.

libffi
>  * To install on an apt-based Linux, try running `sudo apt-get install
    libffi-dev`.
>  * To install on an rpm-based Linux, try running `sudo yum install
    libffi-devel`.

Autoconf
>  * To install on an apt-based Linux, try running `sudo apt-get install
    autoconf`.
>  * To install on an rpm-based Linux, try running `sudo yum install
    autoconf`.
>  * To install on macOS, try running `brew install autoconf`.
>  * To install on Windows, try running `<path to Cygwin setup>/setup-x86_64 -q
    -P autoconf`.

### 2.2.3 Bootstrap JDK
要编译大版本号为 N 的 JDK ，我们还要另外准备一个大版本号至少为 N-1 的、已经编译好的 JDK，因为 OpenJDK 中许多 Java 语言编写的代码需要在编译器执行，所以需要一个编译器可用的 JDK，官方称为 Bootstrap JDK；这里我们需要安装 JDK11
>  * To install on an apt-based Linux, try running `sudo apt-get install openjdk-11-jdk`.
>  * To install on an rpm-based Linux, try as below.

因为我使用的是 RHEL8 ，所以我采用一下的方式搜索需要安装版本的：

```bash	
[root@linuxprobe ~]# dnf search jdk | grep java-11-*		## 查找对应的软件包名
Last metadata expiration check: 8:14:39 ago on Mon 27 Sep 2021 02:27:51 PM CST.
java-11-openjdk.x86_64 : OpenJDK 11 Runtime Environment
java-11-openjdk-demo.x86_64 : OpenJDK 11 Demos
java-11-openjdk-devel.x86_64 : OpenJDK 11 Development Environment
java-11-openjdk-headless.x86_64 : OpenJDK 11 Headless Runtime Environment
java-11-openjdk-javadoc.x86_64 : OpenJDK 11 API documentation
java-11-openjdk-javadoc-zip.x86_64 : OpenJDK 11 API documentation compressed in a single archive
java-11-openjdk-jmods.x86_64 : JMods for OpenJDK 11
java-11-openjdk-src.x86_64 : OpenJDK 11 Source Bundle
java-11-openjdk-static-libs.x86_64 : OpenJDK 11 libraries for static linking

## 这两个包就是我们需要安装的 Bootstrap JDK
dnf install -y java-11-openjdk
dnf install -y java-11-openjdk-devel
```
## 2.3 编译操作
将 JDK12 包复制到 linux 目录中（/opt/jvm/），并解压缩；编译操作分为两步，一是环境准备，二是编译 JDK；
	configure命令承担了步骤一依赖项检查、参数配置和构建输出目录结构等多项职责，如果编译过程中需要的工具链或者依赖项有缺失，命令执行后将会得到明确的提示，并且给出该依赖的安装命令，属于“友好型”；

>注意，在多次编译，或者目录结构构建成果（configure运行成果）后再次修改配置，必须先使用 `make clean` 和 `make dist-clean` 命令清理目录

在 JDK 目录下运行编译命令：
1. 编译FastDebug版、仅含Server模式的HotSpot虚拟机：`bash configure --disable-warnings-as-errors --enable-debug --with-jvm-variants=server`
2. 编译整个 OpenJDK：`make images`
	```bash
	[root@linuxprobe jdk12-06222165c35f]# pwd
	/opt/jvm/jdk12-06222165c35f
	[root@linuxprobe jdk12-06222165c35f]# bash configure --disable-warnings-as-errors --enable-debug --with-jvm-	variants=server
	[root@linuxprobe jdk12-06222165c35f]# make images
	```
	编译后的文件就在 `JDK目录/build` 下，其文件目录介绍如下：
	![在这里插入图片描述](https://img-blog.csdnimg.cn/a4cbfa4fef2440d797e8f423f6abe7e7.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbWl0YXlz,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
	*上方使用的 `make images` 执行的是整个 OpenJDK 编译，这里 “images” 是 “productimages” 编译目标（Target）的简写别名，这个目标的作用是编译出整个 JDK 镜像，除了 “productimages” 以外；除此外 make 还有以下用法：
>hotspot：只编译HotSpot虚拟机
hotspot-<variant>：只编译特定模式的HotSpot虚拟机
docs-image：产生JDK的文档镜像
test-image：产生JDK的测试镜像
all-images：相当于连续调用product、docs、test三个编译目标
bootcycle-images：编译两次JDK，其中第二次使用第一次的编译结果作为Bootstrap JDK
clean：清理make命令产生的临时文件
dist-clean：清理make和configure命令产生的临时文件

# 三、问题回顾
我所遇到的问题有两个：
1.	因为系统源JDK安装的是 JDK8u302，我直接使用 `dnf remove java` 卸载之前安装的jdk，然后`dnf search jdk` 和 `dnf install java-11-openjdk` 查找和安装 JDK11，然后就开始问题发现之旅：

	a. 查看 java 版本发现仍为 1.8 版本，先对环境变量进行修改：：
	```bash
	449  echo $JAVAHOME		# 查看环境变量，发现环境变量仍然配置的1.8

  	450  echo $PATH			# 查看默认加载java路径，发现路径也未修改，因为引用了JAVA_HOME
  	451  echo $CLASSPATH	
  	
  	# vim /etc/profile 修改环境变量
  	export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.12.0.7-0.el8_4.x86_64
  	export PATH=$PATH:$JAVA_HOME/bin
  	```
  	b. 检查旧版 JDK 安装包是否全部清理，如有残余则全部清理掉：
  	```bash
  	dnf list installed | grep jdk	# 列出所有安装的 jdk 相关软件包
  	dnf remove java-1.8.0-openjdk*	# 清理 java-1.8.0-openjdk 相关的软件包
  	```
  	由此，环境 JDK 已经跟换为 11 版本；
2. 出现的错误如下，本以为是 arguments.cpp 的导致的问题，后经仔细查阅，其实为下方红框中`cclplus: all warnings being treated as errors`，GCC在8.0之后的版本加入了stringop truncation的验证警告，所有这里是因为出现了警告导致编译不通过，由此提醒了我，对于软件版本的监控需要注意，我将会在页首加上各软件版本的信息；
	
	![在这里插入图片描述](https://img-blog.csdnimg.cn/f8c830963b6e40018f4fe0cf6b8bdc7d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAbWl0YXlz,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
	这里将原先的`bash configure --enable-debug --with-jvm-variants=server`
	替换为：`bash configure --disable-warnings-as-errors --enable-debug --with-jvm-variants=server` 即可解决问题

最后，附上一个编译成功后查看编译版本的截图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/8249b937083b4339bd7e03dd836f2e3d.png#pic_center)

