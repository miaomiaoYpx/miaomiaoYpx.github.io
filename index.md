 # 记录windows到linux交叉编译的研究<br>
 
 第一次写博客，记录一下研究windows到linux交叉编译的过程，发布在网上比写到自己电脑的word上更有意义

 写在前面，这是在windows下完成交叉编译的全过程记录，所以有一些针对windows的独特经验，希望能有所帮助吧

 由于之前完全没有接触过交叉编译，甚至连编译程序的详细过程都完全不清楚，所以这个工作是非常有挑战性的，去网上查资料，找到这篇文章：https://www.crifan.com/files/doc/docbook/cross_compile/release/html/cross_compile.html
 详细讲解了交叉编译的原理，非常好，建议先看这个，按照文章的建议，决定用crosstool-ng这个工具来做交叉编译器

 接下来具体的crosstool-ng的使用到生成交叉编译器的过程就参考https://www.crifan.com/files/doc/docbook/crosstool_ng/release/html/crosstool_ng.html#what_is_crosstool_ng

 声明一下，本文的主要流程基本都可以在上面这篇文章中找到，且写的很详细，我这里只是对上述文章中一些情况做补充，所以可以主要参考上面文章，写的真的很详细了

 ### 先总结一下主要流程：
1、安装cygwin<br>
2、下载crosstool-ng的源码，然后在cygwin中编译该源码，得到crosstool-ng程序<br>
3、使用crosstool-ng程序，根据需要，编译生成对应的交叉编译器<br>
4、使用交叉编译器，对你的目标代码进行编译，生成在对应平台下的可执行程序<br>
 另外补充一句，整个步骤几乎全程都在cygwin环境下进行<br>

1、安装cgywin
----
直接去官网找下载链接就行了，一般默认装最新版吧；<br>
 安装cygwin时需注意，最好将安装组件中devel目录下内容全部安装了，否则后面使用make等指令时容易出错，缺失东西，见图所示：
![image](https://user-images.githubusercontent.com/88359975/127953034-02232859-d71b-4e06-be92-85c14fb5e90c.png)

2、安装crosstool-ng
----
去http://crosstool-ng.org/download/crosstool-ng/
 去找最新版下载，需要注意，最新版不一定是最下面的那个，因为该页面的排序有问题，作者在文件列表第一行放了一个提示文件告诉你最新版版本号是多少，
![crosstooldown](https://user-images.githubusercontent.com/88359975/127964162-70c8917f-cac7-4b96-881f-3825b51881da.png)

 编译安装，在cygwin环境下执行以下命令：<br>
 ```./configure --prefix=/opt/crosstool-ng
make
make install
```
 编译过程中，可能会出现错误，大部分情况下是环境中缺失某些工具，因此一开始安装cgywin时选择全部内容的意义就在这里，如果还有缺失，就自己查漏补缺吧，错误信息里会有提示的<br>

 接下来将crosstool-ng添加到环境变量中，这样后面才可以使用 ct-ng 下面的一堆命令<br>
 找到当前用户的.bashrc文件，在windows下可以直接用everything搜索这个文件，然后用notepad++打开直接修改，在最后一行添加:<br>
```PATH=$PATH:/opt/crosstool-ng/bin```<br>
 完事儿测试一下，使用命令:<br>
```ct-ng help```
 测试一下是否安装成功了，或者环境变量是不是配置成功了，未成功的话会提示ct-ng command not found类似的话<br>

3.crosstool-ng的使用
----
 写在前面，要学会使用ct-ng help 来查看crosstool-ng的功能，会列出来所有功能，真的很详细了

使用之前，最好新建一个文件夹，然后在cygwin中 cd 进去，再做所有的编译工作<br>
crosstool-ng内置了一些模板配置，其中很有可能有你需要的，使用命令：<br>
```ct-ng list-samples```
列出所有内置的模板配置，部分列表长这样：<br>

```[G..]   i686-nptl-linux-gnu
[G..]   i686-ubuntu12.04-linux-gnu
[G..]   i686-ubuntu14.04-linux-gnu
[G..]   i686-ubuntu16.04-linux-gnu
[G.X]   i686-w64-mingw32
[G..]   m68k-unknown-elf
[G..]   m68k-unknown-uclinux-uclibc
[G..]   powerpc-unknown-linux-uclibc,m68k-unknown-uclinux-uclibc
[G..]   mips-ar2315-linux-gnu
[G..]   mips-malta-linux-gnu
[G..]   mips-unknown-elf
[G..]   mips-unknown-linux-uclibc
[G..]   mips64el-multilib-linux-uclibc
[G..]   mipsel-multilib-linux-gnu
[G..]   mipsel-sde-elf
[G..]   mipsel-unknown-linux-gnu
[G.X]   i686-w64-mingw32,nios2-spico-elf
[G..]   powerpc-405-linux-gnu
[G..]   powerpc-860-linux-gnu
[G..]   powerpc-e300c3-linux-gnu
[G..]   powerpc-e500v2-linux-gnuspe
[G..]   x86_64-multilib-linux-uclibc,powerpc-unknown-elf
[G..]   powerpc-unknown-linux-gnu
[G..]   powerpc-unknown-linux-uclibc
[G..]   powerpc-unknown_nofpu-linux-gnu
[G..]   powerpc64-multilib-linux-gnu
[G..]   powerpc64-unknown-linux-gnu
[G..]   powerpc64le-unknown-linux-gnu
[G.X]   s390-ibm-linux-gnu
[G..]   s390x-ibm-linux-gnu
[G..]   sh4-multilib-linux-gnu
[G..]   sh4-multilib-linux-uclibc
[G..]   sh4-unknown-linux-gnu
[G..]   sparc-leon-linux-uclibc
[G..]   sparc-unknown-linux-gnu
[G..]   sparc64-multilib-linux-gnu
[G..]   x86_64-centos6-linux-gnu
[G..]   x86_64-centos7-linux-gnu
[G..]   x86_64-multilib-linux-gnu
[G.X]   x86_64-multilib-linux-musl
[G..]   x86_64-multilib-linux-uclibc
[G.X]   x86_64-w64-mingw32,x86_64-pc-linux-gnu
[G..]   x86_64-ubuntu12.04-linux-gnu
[G..]   x86_64-ubuntu14.04-linux-gnu
[G..]   x86_64-ubuntu16.04-linux-gnu
[G..]   x86_64-unknown-linux-gnu
[G..]   x86_64-unknown-linux-uclibc
[G.X]   x86_64-w64-mingw32
[G..]   xtensa-fsf-linux-uclibc
 L (Local)       : sample was found in current directory
 G (Global)      : sample was installed with crosstool-NG
 X (EXPERIMENTAL): sample may use EXPERIMENTAL features
 B (BROKEN)      : sample is currently broken
 ```
 我这里第一次尝试，于是选了x86_64-unknown-linux-gnu这个配置作为基础，然后做一些配置的修改；<br>
 关于这个x86_64-unknown-linux-gnu名称的含义，参考https://www.crifan.com/files/doc/docbook/cross_compile/release/html/cross_compile.html
 文章对交叉编译的解释，写的很详细了，不再赘述<br>
 
  选择好了配置之后，查看配置的可心参数，使用命令：<br>
 ```ct-ng show-<sample> ```<br>
  选定某个配置，使用命令：<br>
  ```ct-ng <sample>```<br>
  这个<sample>就是上面选择的“x86_64-unknown-linux-gnu”这一串字符，<br>
 
 接着，进入当前选择的模板的具体配置，可以查看和修改配置，使用命令：
  ```ct-ng menuconfig```
  会弹出一个配置界面，通过上下来移动，回车来进入选项，空格来打钩或取消打钩<br>
  这里头大部分都不用动，有几项关键的配置要改一下，<br>
  ### 一个配置是<br>
  ```Paths and misc options
    (4) Number of parallel jobs
 ```
  用于设置多线程编译，可以提高编译速度，虽然没有测试过，但我在设置了4个线程的情况下，从头到尾完整编译完一个编译器，基本上也要花数个小时，所以，你懂的<br>
  ### 还有个最关键的配置

  ```Paths and misc options
    [*] Debug crosstool-NG
    [*]   Save intermediate steps
 ```
  `一定要选上，它太重要了`,这个选项的作用是，记录编译步骤，一边你可以从之前出错那一步之前恢复继续编译<br>
 
  你在编译的过程中，一定会遇到各种出错，一旦报错，就得解决问题之后再继续编译，这里如果不勾的话，想想吧，你每次都得从头编译，绝对会裂开的。<br>
  
 这个勾上之后怎么用呢，首先你找到最新的一条执行通过的步骤，举个例子，<br>
  ```\=================================================================
[INFO ]  Installing MPC for host
[EXTRA]    Configuring MPC
[EXTRA]    Building MPC
[EXTRA]    Installing MPC
[INFO ]  Installing MPC for host: done in 182.22s (at 27:10)
[EXTRA]  Saving state to restart at step 'binutils_for_host'...
[INFO ]  =================================================================
[INFO ]  Installing binutils for host
[EXTRA]    Configuring binutils
[ERROR]    configure: error: cannot create configure.lineno; rerun with a POSIX shell
[ERROR]
[ERROR]  >>
[ERROR]  >>  Build failed in step 'Installing binutils for host'
[ERROR]  >>        called in step '(top-level)'
[ERROR]  >>
[ERROR]  >>  Error happened in: CT_DoExecLog[scripts/functions@257]
[ERROR]  >>        called from: do_binutils_backend[scripts/build/binutils/binutils.sh@205]
[ERROR]  >>        called from: do_binutils_for_host[scripts/build/binutils/binutils.sh@92]
[ERROR]  >>        called from: main[scripts/crosstool-NG.sh@632]
 ```
  其中的这句，<br>
```[EXTRA]  Saving state to restart at step 'binutils_for_host'...```<br>
  'binutils_for_host'就是关键字了，下次恢复编译时，使用命令：
  ```ct-ng binutils_for_host+```<br>
  就可以从这一步开始恢复编译，强无敌，可即使这样，仍然很可能要花费很多时间来重复这个编译过程，慢慢来吧<br>
  
  配置好以后，就可以编译了，使用命令：<br>
  ```ct-ng build```<br>
  开始编译，过程中如果出现：<br>
  Extracting 'linux-custom' 的字样，建议终止编译过程，ctrl+c ，然后手动下载需要的软件包源码，否则让程序自动下载的速度可以说是非常慢了，当然，也不一定都很慢，下载链接的话，可以打开该build.log日志文件，在其中找到下载链接，然后去浏览器打开下载就行，通常会快很多，下载下来放到自动生成的tarballs文件夹下就行了，然后在使用恢复的命令或者build命令来恢复构建过程<br>
  等所有的下载都完成了，就可以放任他自己跑了。<br>
 
 ### 编译过程出错的处理
  参考的文章中列出了很多出错和解决办法，这里只列出我遇到的他文章里没有的出错<br>
 #### 1. version mismatch.  This is Automake 1.15.1 
  Automake 版本不对， 使用 autoreconf -ivf 命令强制更新makefile中的配置<br>
 #### 2.'string' in namespace 'std' does not name a type 
  在对应文件中增加#include <string>的引用，是的你没有看错，我们直接改工具源码，不要怕<br>
 #### 3. no usable python found at /usr/bin/python
  打开之前cygwin的安装程序，从中搜索python，在python目录下勾上python对应版本的devel文件，然后安装一下，安装好后从usr/include中找到名为pythonx.x的文件夹，然后放到/usr/bin下面去，如果没有文件夹就创造文件夹<br>
  
  ### 编译成功后：
  最终编译完成，会在x-tools文件夹下找到对应的交叉编译器目录，然后要将编译器添加到环境变量中才能使用，把.bashrc的环境变量改为<br>
```\#PATH=$PATH:/opt/bin
PATH=$PATH:$HOME/.../x-tools/<templename>/bin:/opt/bin
 ```
  <templename>就是你生成的交叉编译器目录名，...是你的路径，按实际情况填写<br>
  再去重启cygwin，然后试试命令：<br>
```<templename> -v```
  正确打印版本号信息说明配置正确，至此，恭喜你，交叉编译器生成成功了！<br>
  
  4.使用交叉编译器编译代码
 ----
  详细步骤可以参考https://www.crifan.com/summary_cross_compile_library_note/ <br>
  这个过程，最重要的一步是，在执行<br>
  ```./configure ... ```
  命令去配置程序时，将以下参数设置好：<br>
  
prefix：指的是，将编译好的库，安装到哪里<br>
build=i686-pc-cygwin：指的是，我当前的pc编译环境是cygwin<br>
target=arm-xscale-linux和–host=arm-xscale-linux：表示我的编译是交叉编译，即编译出来的代码，是运行在xscale上的。<br>
disable-cplusplus：表示是去禁止cpp的编译->不会去编译src/cpp下面的东西->不会出现上面的"XmlRpcCpp.cpp:39: undefined reference to `_xmlrpc_env_init’"错误<br>
CC=arm-xscale-linux-gnueabi-gcc：指定所用的交叉编译器gcc<br>
  
  参数的值根据你的具体情况去填,配置完成后，调用最后的make命令，会调用你的交叉编译器，来编译生成对应平台的可执行文件，然后把该文件拷贝过去试一试吧<br>
  
  ### 补充：一开始只有.c或.cpp的源码，怎么从无到有生成configure和makefile文件<br>
  #### 1.使用autoscan命令，生成configure.scan<br>
  #### 2.将configure.scan该名为configure.ac并修改其内容，可参考https://www.cnblogs.com/fallenmoon/p/7506035.html<br>
  #### 3.执行aclocal和autoconf命令，生成aclocal.m4和configure文件<br>
  #### 4.新建Makefile.am文件，填入以下内容：<br>
AUTOMAKE_OPTIONS=foreign <br>
bin_PROGRAMS=helloworld <br>
helloworld_SOURCES=helloworld.c <br>
  #### 5.执行automake --add-missing生成makefile.in文件<br>
  #### 6.执行configure ... 配置并生成makefile文件，参数填写见上文<br>
  #### 7.执行make<br>
  
  ## 附录
   ### 记录一下在windows上编写脚本来调用cygwin并执行给cygwin执行的脚本的过程
   由于考虑到功能的自动化，所以想编写一个脚本来将使用交叉编译器编译源代码的过程串起来，自动执行。
   
   1.首先编写shell脚本，供cygwin来调用，其中放入了编译源代码的全过程命令，包括autoscan、aclocal等，和上文中一致，脚本写好之后，直接在cygwin控制台中输入./name.sh运行一下脚本试试，成功了再进行下一步<br>
   
   顺便学了一波命令：<br>
   rm 删除文件或文件夹，参数-r 表示递归删除子文件夹中的内容，-f表示强制删除，所以参数这么写:rm -rf 文件夹名称<br>
   cp 复制文件或文件夹，cp -r path1 path2 表示把path1复制到path2下，如果"path1"换成"path1/."就是把path1下的文件复制过去，不包括path1这个文件夹<br>
   mv 修改文件名<br>
   cat > filename << exit  这个指令很棒，完美契合了我需要打开文件并修改其内容的要求，执行指令后会打开文件并进入编辑模式，后续的输入内容直接作为数据写入文件中，然后输入exit表示结束，这样就可以在脚本里实现文件创建及内容编写了，ps:(cat > 表示覆盖从头开始写，cat >> 表示追加在尾部)<br>
   2.接下来就是如何在windows下启动cygwin并让它来调用这个脚本，查了一下资料，直接在cmd控制台里头使用命令：<br>
   ```bash --login``` （前提是bash已经添加了环境变量）<br>
   这样之后，就相当于在cmd中进入了cygwin的环境，接下来可以直接运行第一步的脚本，然而，照这样写到.bat脚本文件中，运行之后发现并不行，它只能运行到bash语句，之后的语句完全不执行了<br>
   可以有两种方法，使得一连串的语句自动执行：<br>
   第一种是，在.bashrc文件末尾，添加你要执行的指令，这样，在cmd调用bash之后，会自动执行这些指令，可以达到效果，但我不喜欢，这样可太怪了，以后要改变的话就得去修改.bashrc文件，这太蠢了；<br>
   第二种是，在cmd执行bash命令的时候，同时告诉他你想做的事，可以是一条指令，也可以是一个文件，但执行一条指令太有限了，很多事情没法做，那么只能是一个文件了，一个脚本文件，里面包含了切换目录，执行第一步的脚本文件等我想要写的命令，然后在cmd中执行：<br>
   ```bash --login -c "filename"```<br>
   filename就是第二个脚本文件咯<br>
   所以最终就是这样，一个编译源码的脚本，一个调用该脚本的脚本，其中的内容基本长这样：<br>
```
cd 某个目录
./build.sh
exit
```
exit用于自动关闭控制台，根据你的需要来调整吧<br>
于是windows上的.bat只有一句话：<br>
   ```bash --login -i -c "test.sh"```<br>
   这样，就基本完成了整个流程的串联<br>
  至此，还遗留了一个问题，configure.ac和Makefile.am中的内容，在脚本中，目前是写死的，没办法根据实际情况去自动调整内容，这个有待后续研究吧

## 记录以下在虚拟机centOS8上安装matlab2021的过程
   
   首先将安装包放到虚拟机上，发现直接拖，拖不过去，试了各种方法后，最后还是用了ftp来传递<br>
   然后将文件中的Matlab910R2021a_Lin64.iso使用mount命令挂载到某个位置；<br>
   接着尝试直接安装，也就是在终端输入命令```./install```<br>
   遇到两种报错：<br>
   1. ```DisplayE​rror' what(): No display available. Aborted (core dumped)```<br>
   这是由于在root权限下执行命令造成的，使用其他用户来执行就好了<br>
   2. ``` what():  Unable to launch the MATLABWindow application```<br>
   查了半天，有个人说<br>
   ```
   You are missing  libselinux.so.1
yay -S libselinux
and retry
   ```
   试了一下说yay命令不存在。。。<br>
   还有一个人说：<br>
   ```
   \cd /home/YourUserName/Downloads/matlab_R2020a_glnxa64/bin/glnxa64
rm libcrypto.so.1.1
sudo su -
cd /home/YourUserName/Downloads/matlab_R2020a_glnxa64
export DISPLAY=':0'
./install
   ```
   这个成功了<br>
   然后就顺利安装啦<br>
