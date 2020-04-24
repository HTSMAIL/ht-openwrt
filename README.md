# ht-openwrt
自己编译给自己用的（内容来自他人博客）

编译方法


升级Ubuntu系统
sudo apt-get update

安装依赖
sudo apt-get -y install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler

编译开始
git clone https://github.com/coolsnowwolf/lede #lede源，个人不喜欢

项目开源地址：
git clone https://github.com/Lienol/openwrt

git clone https://github.com/coolsnowwolf/lede

两者主要区别在于 Lienol 带 passwall 插件，lean 带 ssr-plus，LuCI 里的插件两者大部分是一样的，具体可以看我后面的插件列表

./scripts/feeds update -a #更新

./scripts/feeds install -a #安装更新

make menuconfig #配置文件

make V=s

先编译 Linksys EA6500v2 固件吧。

运行 Ubuntu 子系统，进入 lede 文件夹，feed 更新加安装：

cd lede 或者 cd openwrt

./scripts/feeds update -a 

./scripts/feeds install -a 

LinuxCopy
选择编译项目：

make menuconfig

LinxuCopy
上下键选择项目，左右键选择退出保存等。
输入 Y 选择该项目加入固件，N 不选泽，M 编译但不合入固件。
所有项目选完后保存再退出。保存时可以重命名，但只起保存当前配置的作用，编译有效的配置文件名还是 .config。

最后输入 make -j1 V=s （-jn 后面的 n 是线程数。第一次编译推荐用单线程，国内请尽量全局科学上网或者国内白名单）即可开始编译，也可以直接 make V=s 编译。第一次时间比较久，我的台式机要三四个小时，比编译 Android 固件快一些，如果后面只修改选择插件，再次编译可能只要十几二十分钟。
如果需要再次编译：

cd lede 或者 cd openwrt

git pull 同步更新源码

./scripts/feeds update -a && ./scripts/feeds install -a

rm -rf ./tmp && rm -rf .config 清除编译配置和缓存

make menuconfig

make -j1 V=s n=线程数+1，例如4线程的I5填-j5，开始编译

LinuxCopy

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




 
插件列表及简要说明见这里：
Openwrt 编译 LuCI 插件说明-EA6500v2-Lienol.xlsx 提取码: jr6r

Openwrt 编译 LuCI 插件说明-EA6500v2-lean.xlsx 提取码: ic8t

说一下，选择插件不用一股脑儿全选了，根据自己的需要选，很多硬路由都有空间限制，比如这个 EA6500 v2，我开始编译的固件有 35MB，通过 dd-wrt 过渡固件升级不行，miniweb 上传不行，tftp 也不行。就开始怀疑编译的固件有问题，我用了默认选择的插件编译出来只有 10MB，一刷就启动了。后来搜索到是因为 linksys 有分区限制，大约超过 34MB 就不行了。比如 qBittorrent，ssr-plus 体积都挺大的。
这种情况下 LuCI 插件就不用 Y 选，用 M。编译但不合入固件，刷好小体积固件后再到路由器去上传安装编译好的 ipk 插件。

添加 Passwall 插件
Lienol 源的 Passwall 的插件，这个功能和 lean 源的 ssr-plus 插件差不多，支持多个 SS,SSR,V2 或 Trojan 节点，同时具有分流，故障转移，自动恢复，自带 HaProxy 负载均衡。比较而言我更喜欢这个插件。

在 lean 源添加 Passwall 插件编译的方法：

cd lede
vi feeds.conf.default
LinuxCopy
编辑 feeds.conf.default 文件，在末尾加上一行：src-git lienol https://github.com/Lienol/openwrt-package。当然，在 win10 下可以直接搜索该文档进行编辑。
然后输入命令：

./scripts/feeds clean

./scripts/feeds update -a

rm -rf feeds/lienol/lienol/ipt2socks

rm -rf feeds/lienol/lienol/shadowsocksr-libev

rm -rf feeds/lienol/lienol/pdnsd-alt

rm -rf feeds/lienol/package/verysync

rm -rf feeds/lienol/lienol/luci-app-verysync

rm -rf package/lean/kcptun

rm -rf package/lean/trojan

rm -rf package/lean/v2ray

rm -rf package/lean/luci-app-kodexplorer

rm -rf package/lean/luci-app-pppoe-relay

rm -rf package/lean/luci-app-pptp-server

rm -rf package/lean/luci-app-v2ray-server

./scripts/feeds install -a

LinuxCopy
因为包重复冲突所以要删除才行，然后再 make menuconfig。

刷机
硬路由一般刷机有三种方式：在原始固件里直接升级新固件；复位键 30s 后进入 uboot web 界面上传固件；复位后用 tftp 软件发固件。如果路由器直接启动不了了就得电脑连接路由器的串口，通过串口命令，用 tftp 软件发送固件。
无论哪种办法，最好在刷机的时候打开命令窗口持续观测路由 ping 192.168.1.1 -t，TTL=64 是正常连接的状态，TTL=100 时可以访问 boot web 界面上传或者用 tftp 发送固件。有时候 tftp 客户端会发送失败，那就换一个吧，我觉得 tftpd64 下载 这个发送效果不错。
还有一点要注意，操作系统里的 IP 地址设置，要和路由器在同一个 IP 段，包括子网掩码都得手动填好，不要自动获取。有的固件默认 IP 并不是 192.168.1.1，要先确定好，别设错了 ping 半天都不对。

固件
整个编译过程还是比较简单的，就是耗点时间，我把最后编译的固件刷到了 Linksys EA6500v2，比较遗憾，WiFi 体验很差，只支持 802.11bg，没有 5G。当然，这个不是这个源或编译的问题，我试过 openwrt 官方固件也是一样。
如果当有线路由还可以，科学上网比较稳，速度也快。
