# ZTE E8820v2云编译库—2

首先感谢siwind的源码库 https://github.com/siwind/openwrt 以及他修改的dts文件

增加了科学feeds源，保证此库可以正确编译（只测试了32+128硬件配置）。

## 进行Aciton准备工作

由于我硬改了了32+128，所以要分别对dts和机型描述进行修改，分别在/target/linux/ramips/dts和/target/linux/ramips/image下的mt7621.mk内。
dts文件中这处字段需要修改：

   ```bash
   partition@50000 {
	   			compatible = "denx,uimage";
		   		label = "firmware";
		   		reg = <0x50000 0xfb0000>;
		   	};

   # 将0xfb0000修改为0x1fb0000
   ```

机型代码修改如下：

   ```bash
   define Device/zte_e8820v2
     $(Device/dsa-migration)
     $(Device/uimage-lzma-loader)
     IMAGE_SIZE := 32448k
     DEVICE_VENDOR := ZTE
     DEVICE_MODEL := E8820V2
     DEVICE_COMPAT_VERSION := 2.0
     DEVICE_PACKAGES := kmod-mt7603e kmod-mt76x2e kmod-usb2 \
   	  kmod-usb-ledtrig-usbport luci-app-mtwifi -wpad-openssl
   endef
   TARGET_DEVICES += zte_e8820v2

   # 将IMAGE_SIZE的值修改为32448k
   ```
——————————————————————————————————————————————————————————————

### 编辑Action库

修改/.github/workflows/build-openwrt.yml文件:

   ```bash
  env:
  REPO_URL: https://github.com/yourname/lede.git
  REPO_BRANCH: master
  FEEDS_CONF: feeds.conf.default
  CONFIG_FILE: .config
  DIY_P1_SH: diy-part1.sh
  DIY_P2_SH: diy-part2.sh
  UPLOAD_BIN_DIR: false
  UPLOAD_FIRMWARE: true
  UPLOAD_COWTRANSFER: false
  UPLOAD_WETRANSFER: false
  UPLOAD_RELEASE: false
  TZ: Asia/Shanghai

  # REPO_URL的值修改为你的源码仓库地址
   ```

### 下面进行固件自定义，不推荐直接对.config进行编译，对小白来说很容易没选对依赖，我感觉没有物理机好用。

1. 首先装好 Linux 系统，推荐 Debian 11 或 Ubuntu LTS

2. 安装编译依赖

   ```bash
   sudo apt update -y
   sudo apt full-upgrade -y
   sudo apt install -y ack antlr3 asciidoc autoconf automake autopoint binutils bison build-essential \
   bzip2 ccache cmake cpio curl device-tree-compiler fastjar flex gawk gettext gcc-multilib g++-multilib \
   git gperf haveged help2man intltool libc6-dev-i386 libelf-dev libglib2.0-dev libgmp3-dev libltdl-dev \
   libmpc-dev libmpfr-dev libncurses5-dev libncursesw5-dev libreadline-dev libssl-dev libtool lrzsz \
   mkisofs msmtp nano ninja-build p7zip p7zip-full patch pkgconf python2.7 python3 python3-pyelftools \
   libpython3-dev qemu-utils rsync scons squashfs-tools subversion swig texinfo uglifyjs upx-ucl unzip \
   vim wget xmlto xxd zlib1g-dev python3-setuptools
   ```

3. 下载你修改后的源代码，更新 feeds 并选择配置

   ```bash
   git clone https://github.com/yourid/lede
   cd lede
   ./scripts/feeds update -a
   ./scripts/feeds install -a
   make menuconfig
   ```

附上一份OpenWrt 编译 LuCI -> Applications 添加插件应用说明，https://www.right.com.cn/forum/thread-344825-1-1.html

4. 由于我们是利用Aciton进行云编译，所以在make menuconfig之后，只要把lede文件夹下的.config文件上传到Actions-OpenWrt上并覆盖原文件就行了，之后就是常规的云编译步骤。
