# ht-openwrt编译方法说明
关于虚拟机如何运行
https://blog.csdn.net/ballack_linux/article/details/81331527

编译方法

   make V=99 -j 96（核心数）#一键起飞




详细说明篇章：这里才是正式开始》》》》

升级Ubuntu系统
sudo apt-get update

安装依赖
sudo apt-get -y install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler

编译开始克隆源码
git clone https://github.com/coolsnowwolf/lede #lede源

项目开源地址：
git clone https://github.com/Lienol/openwrt

git clone https://github.com/coolsnowwolf/lede

两者主要区别在于 Lienol 带 passwall 插件，lean 带 ssr-plus，LuCI 里的插件两者大部分是一样的

cd lede 或者 cd openwrt

添加passwell插件 
luci-app-passwall停止开发，当然如果存在BUG，欢迎各位大佬PR。

vi feeds.conf.default

使用方法： 编辑OpenWRT源码根目录 feeds.conf.default 将https://github.com/Lienol/openwrt-package 替换为 https://github.com/HTSMAIL/Lienol-openwrt-packages-backup
然后执行

./scripts/feeds clean

./scripts/feeds update -a

./scripts/feeds install -a

或者你可以把该源码手动下载或Git Clone下载放到OpenWRT源码的Package目录里面，然后编译。 如果你使用的是Luci19，请编译时选上"luci","luci-compat","luci-lib-ipkg"后编译

 #更新 
./scripts/feeds update -a 

./scripts/feeds install -a #安装更新

rm -rf ./tmp && rm -rf .config 清除编译配置和缓存

SSR P+添加

使用方法

    #源码根目录，编辑.gitignore文件
    vi .gitignore
    
    #在文件最后一行，加入
    git rm --cached package/lean/luci-app-ssr-plus/ -r
    
    #保存后，进入lean源码目录
    cd package/lean/
    
    #下载源码
    git clone https://gitee.com/xiugan/luci-app-ssr-plus.git
    
    #回到源码根目录
    cd ../..
    
    #拉取源码
    git pull
    make menuconfig #配置文件

    make -j8 download V=s 下载dl库（国内请尽量全局科学上网）

    输入 make -j1 V=s （-j1 后面是线程数。第一次编译推荐用单线程）即可开始编译你要的固件了。

    或者make V=s

上下键选择项目，左右键选择退出保存等。
输入 Y 选择该项目加入固件，N 不选泽，M 编译但不合入固件。
所有项目选完后保存再退出。保存时可以重命名，但只起保存当前配置的作用，编译有效的配置文件名还是 .config。

最后输入 make -j1 V=s （-jn 后面的 n 是线程数。第一次编译推荐用单线程，国内请尽量全局科学上网或者国内白名单）即可开始编译，也可以直接 make V=s 编译。第一次时间比较久，我的台式机要三四个小时，比编译 Android 固件快一些，如果后面只修改选择插件，再次编译可能只要十几二十分钟。
如果需要再次编译：

如果编译出了问题，还可以执行命令 make clean 来清除之前编译所产生的 object 文件（后缀为“.o”的文件）及可执行文件，再来一遍。


编译完成后固件输出在 /lede/bin/targets 目录下，按 CPU 排列。可以用 everything 软件直接搜索 lede 或 openwrt，找 C 盘 Ubuntu 文件夹下，即可看到编译生成的固件。

如果编译失败，绝大多数情况是网络引起的，文件下载不完整，或者有的链接需要翻墙什么的，编译环境和步骤不错的话多试几次就好了。

选项简要说明

进入 menuconfig 第一眼感觉好复杂，不是专业的根本不知道都是啥，不过我们编译自己的固件不需要知道那么多，大多数默认设置就好了。

    Target System (x86) ---> #设置CPU类型（软路由所以选择x86,硬路由根据型号厂家选择自己的cpu)

    Subtarget (x86_64) ---> #CPU子选项

    Target Profile (Generic) ---> #厂家具体型号

    Target Images ---> #设置编译的格式（squashfs，ext4）

    Global build settings ---> #全局设置

    [ ] Advanced configuration options (for developers) ---- #高级配置选项

    [ ] Build the OpenWrt Image Builder #创建OpenWrt镜像生成器

    [ ] Build the OpenWrt SDK #创建OpenWrt SDK

    [ ] Package the OpenWrt-based Toolchain #打包基于OpenWrt的工具链

    [ ] Image configuration ---> #镜像配置

    Base system ---> #设置基础系统

    Administration ---> #管理

    Boot Loaders ---> #设置启动加载器

    Development --->

    Extra packages ---> #设置额外软件包

    Firmware ---> #设置固件

    Fonts --->　#设置字体

    Kernel modules ---> #设置一些接口模块，如LED，i2c，spi等

    Languages ---> #设置语言，如go，lua，node.js，php，Python等等

    Libraries ---> #设置库

    LuCI ---> #LuCi设置（这里重点开始选择- 3. Applications ->进去编译选择“y”，取消选“n”,说明在下边链接 ）

        1. collections luCI HTTPS支持 

        2. modules 模块，选中 Minify Lua Sources 压缩 Lua 脚本可增大固件中的可用空间

        3. applications 应用

        4. themes 主题

        5. protocols 支持协议

        6. libraries 支持docker json等库

        9. freifunk 社区产品

    Mail ---> mail 相关软件，协议等

    Multimedia ---> #设置多媒体，如FFmpeg

    Network ---> #网络配置，如bittorrent，firewall，download manager，VPN，ssh等等

    Sound ---> #声音配置

    Utilities ---> #设置实用程序

    Xorg ---> #字体配置

  拿 EA6500 v2 路由来做例子：

    Target System --> Broadcom BCM47xx/53xx(ARM)

    Target Profile --> Multiple devices

    Target Devices --> Linksys EA6500 V2

    Target Images --> squashfs


    LuCI --> 3. Applications --> 选择需要的插件，根据路由器的 flash 大小
    --> 4. Themes --> 默认就好，有的主题体积会比较大
