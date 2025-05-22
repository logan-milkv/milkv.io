---
sidebar_label: 'Buildroot SDK'
sidebar_position: 40
---

# 简介

Duo 系列开发板默认的 SDK 是基于 buildroot 构建的，用来生成 Duo 的固件，目前 SDK 有 V1 和 V2 两个版本。

:::tip
- V1 版本 SDK 只支持 RISCV 核。（建议 Duo 64M 使用该版本）
- V2 版本 SDK 既支持 RISCV 核，也支持 ARM 核。(建议 Duo256M 和 DuoS 使用该版本)
:::

*建议 Duo 64M 选用 V1 版本的 SDK，主要是由于其内存较小，V2 版本的 SDK 中相关的 AI 应用目前还无法在 Duo 64M 中正常运行。*

## Buildroot SDK V1

SDK 主要包含如下几个部分：

- u-boot: 2021.10
- linux kernel: 5.10.4
- buildroot: 2021.05
- opensbi: 89182b2

源码地址: [https://github.com/milkv-duo/duo-buildroot-sdk](https://github.com/milkv-duo/duo-buildroot-sdk)

SDK目录结构

```text
├── build                编译目录，存放编译脚本以及各board差异化配置
├── build.sh             Milk-V Duo 一键编译脚本
├── buildroot-2021.05    buildroot 开源工具
├── freertos             freertos 系统
├── fsbl                 fsbl启动固件，prebuilt 形式存在
├── install              执行一次完整编译后，临时存放各 image 路径
├── isp_tuning           图像效果调试参数存放路径
├── linux_5.10           开源 linux 内核
├── middleware           算能多媒体框架，包含 so 与 ko
├── device               存放 Milk-V Duo 相关配置及脚本文件的目录
├── opensbi              开源 opensbi 库
├── out                  Milk-V Duo 最终生成的镜像所在目录
├── ramdisk              存放最小文件系统的 prebuilt 目录
└── u-boot-2021.10       开源 uboot 代码
```

## Buildroot SDK V2

源码地址: [https://github.com/milkv-duo/duo-buildroot-sdk-v2](https://github.com/milkv-duo/duo-buildroot-sdk-v2)

```text
├── build                编译目录，存放编译脚本以及各board差异化配置
├── build.sh             Milk-V Duo 一键编译脚本
├── buildroot            buildroot 开源工具
├── cvi_mpi              算能多媒体框架
├── device               存放 Milk-V Duo 相关配置及脚本文件的目录
├── freertos             freertos 系统
├── fsbl                 fsbl启动固件，prebuilt 形式存在
├── install              执行一次完整编译后，临时存放各 image 路径
├── isp_tuning           图像效果调试参数存放路径
├── linux_5.10           开源 linux 内核
├── osdrv                一些外设驱动源码
├── opensbi              开源 opensbi 库
├── out                  Milk-V Duo 最终生成的镜像所在目录
├── tdl_sdk              算能深度学习算法 SDK
└── u-boot-2021.10       开源 uboot 代码
```

# 编译镜像

准备编译环境，使用本地的 Ubuntu 系统，官方支持的编译环境为 `Ubuntu Jammy 22.04.x amd64`。

如果您使用的是其他的 Linux 发行版，我们强烈建议您使用 Docker 环境来编译，以降低编译出错的概率。

以下分别介绍两种环境下的编译方法。

另外，V1 版本和 V2 版本 SDK 的编译方法基本一致，下面以 V2 版本 SDK 为例，介绍 SDK 的编译步骤。

## 一、使用 Ubuntu 22.04 编译

### 安装编译依赖的工具包

```bash
sudo apt install -y pkg-config build-essential ninja-build automake autoconf libtool wget curl git gcc libssl-dev bc slib squashfs-tools android-sdk-libsparse-utils jq python3-distutils scons parallel tree python3-dev python3-pip device-tree-compiler ssh cpio fakeroot libncurses5 flex bison libncurses5-dev genext2fs rsync unzip dosfstools mtools tcl openssh-client cmake expect python-is-python3
```

对于 [duo-buildroot-sdk-v2](https://github.com/milkv-duo/duo-buildroot-sdk-v2)，还需要安装以下工具包：
```bash
sudo pip install jinja2
```

### 获取 SDK

```bash
git clone https://github.com/milkv-duo/duo-buildroot-sdk-v2.git --depth=1
```

在编译过程中，Buildroot 会自动下载一些源码包。SDK V1 中已经集成了部分常用源码包，可以直接编译。SDK V2 中，为了避免由于网络原因导致下载失败，可以提前下载 `dl.tar` 包并解压到 `buildroot` 目录：
```bash
wget https://github.com/milkv-duo/duo-buildroot-sdk-v2/releases/download/dl/dl.tar
tar xvf ./dl.tar -C ./duo-buildroot-sdk-v2/buildroot/
```
解压后 `duo-buildroot-sdk-v2` 的 `buildroot` 中 `dl` 目录结构如下：
```text
└── buildroot
    └── dl
        ├── acl
        ├── alsa-lib
        ├── alsa-utils
        ├── attr
        ├── ...
```

### 1、一键编译

执行一键编译脚本 `build.sh`：
```bash
cd duo-buildroot-sdk-v2/
./build.sh
```
会看到编译脚本的使用方法提示：
```bash
$ ./build.sh
Usage:
./build.sh              - Show this menu
./build.sh lunch        - Select a board to build
./build.sh [board]      - Build [board] directly, supported boards as follows:
milkv-duo-musl-riscv64-sd
milkv-duo256m-glibc-arm64-sd
milkv-duo256m-musl-riscv64-sd
milkv-duos-glibc-arm64-emmc
milkv-duos-glibc-arm64-sd
milkv-duos-musl-riscv64-emmc
milkv-duos-musl-riscv64-sd
```
最下边列出的是当前支持的目标版本列表。

如提示中所示，有两种方法来编译目录版本。

第一种方法是执行 `./build.sh lunch` 调出交互菜单，选择要编译的版本序号，回车：
```bash
$ ./build.sh lunch
Select a target to build:
1. milkv-duo-musl-riscv64-sd
2. milkv-duo256m-glibc-arm64-sd
3. milkv-duo256m-musl-riscv64-sd
4. milkv-duos-glibc-arm64-emmc
5. milkv-duos-glibc-arm64-sd
6. milkv-duos-musl-riscv64-emmc
7. milkv-duos-musl-riscv64-sd
Which would you like:
```

:::tip
- Duo (64M-DDR) 只有 RISC-V 核，默认为基于 musl libc 编译的 SD 卡镜像。
- Duo256M 支持 RISC-V 和 ARM 核(二选一)，RISC-V 核固件基于 musl libc 编译，ARM 核固件基于 glibc 编译，只配置了 SD 卡镜像。
- DuoS 支持 RISC-V 和 ARM 核(二选一)，RISC-V 核固件基于 musl libc 编译，ARM 核固件基于 glibc 编译，配置了 SD 卡和 eMMC 镜像。
:::

第二种方法是脚本后面带上目标版本的名字，比如要编译 `milkv-duos-musl-riscv64-sd` 的镜像:
```bash
$ ./build.sh milkv-duos-musl-riscv64-sd
```

编译成功后可以在 `out` 目录下看到生成的镜像，其中 `*.img` 后缀的镜像为 SD 卡烧录的镜像，`*.zip` 后缀的镜像为 eMMC 的烧录镜像。

*注: 第一次编译会自动下载所需的工具链，大小为 840M 左右，下载完会自动解压到 SDK 目录下的 `host-tools` 目录，下次编译时检测到已存在 `host-tools` 目录，就不会再次下载了*

### 2、分步编译

如果未执行过一键编译脚本，需要先手动下载工具链 [host-tools](https://github.com/milkv-duo/host-tools.git)，再拷贝或移动到 SDK 根目录：

```bash
git clone https://github.com/milkv-duo/host-tools.git
cp -a host-tools duo-buildroot-sdk-v2/
```

加载环境：
```bash
source build/envsetup_milkv.sh
```
如果是第一次编译，会提示选择要编译的目标：
```bash
Select a target to build:
1. milkv-duo-musl-riscv64-sd
2. milkv-duo256m-glibc-arm64-sd
3. milkv-duo256m-musl-riscv64-sd
4. milkv-duos-glibc-arm64-emmc
5. milkv-duos-glibc-arm64-sd
6. milkv-duos-musl-riscv64-emmc
7. milkv-duos-musl-riscv64-sd
Which would you like:
```

选择对应的数字后，回车，成功加载环境变量后，会显示一些当前目标的信息，比如：
```bash
Target Board: milkv-duos-musl-riscv64-sd
Target Board Storage: sd
Target Board Config: /build/duo-buildroot-sdk-v2/device/target/boardconfig.sh
Target Image Config: /build/duo-buildroot-sdk-v2/device/target/genimage.cfg
Build tdl-sdk: 1
Output dir: /build/duo-buildroot-sdk-v2/install/soc_sg2000_milkv_duos_musl_riscv64_sd
```

环境加载完成后，会在 `device` 目录下创建一个名为 `target` 的链接，链接到编译目标目录，下次再 source 环境时，检测到该 `target` 链接存在时，就不会再提示选择目标了。如果需要更换编译目标，可以加 `lunch` 参数重新调出交互菜单进行选择：
```bash
source build/envsetup_milkv.sh lunch
```

加载环境成功后，再依次输入如下命令完成分步编译：
```bash
clean_all
build_all
```

如果编译的是 SD 卡镜像，还需要再执行如下命令生成 `img` 镜像：
```bash
pack_sd_image
```

比如需要编译 `milkv-duos-musl-riscv64-sd` 的镜像，分步编译命令如下：
```bash
source build/envsetup_milkv.sh milkv-duos-musl-riscv64-sd

clean_all
build_all
pack_sd_image
```

生成的固件位置：
```text
install/soc_sg2000_milkv_duos_musl_riscv64_sd/milkv-duos-musl-riscv64-sd.img
```

注意，SD 卡镜像为 `*.img`，eMMC镜像为 `upgrade.zip`。

:::tip
除了使用 `build_all` 进行完整编译之外，还可以单独编译某个模块，一般来说都是先执行 `clean_xxx` 清理中间文件后，再执行 `build_xxx` 重新编译。支持的模块，可以输入 `clean_` 或者 `build_` 后，双击 `tab` 键查看。以下列出一些常用的模块编译方法：
- fsbl: `clean_fsbl`，`build_fsbl`
- uboot: `clean_uboot`，`build_uboot`
- kernel: `clean_kernel`，`build_kernel`
- osdrv: `clean_osdrv`，`build_osdrv`
- cvi_mpi: `clean_middleware`，`build_middleware`
- tdl-sdk: `clean_tdl_sdk`，`build_tdl_sdk`
:::

## 二、使用 Docker 编译

需要在运行 Linux 系统的主机上支持 Docker。 Docker 的使用方法请参考[官方文档](https://docs.docker.com/)或其他教程。

我们将 SDK 代码放在 Linux 主机系统上，调用 Milk-V 提供的 Docker 镜像环境来编译。

### 在 Linux 主机上拉 SDK 代码

```bash
git clone https://github.com/milkv-duo/duo-buildroot-sdk-v2.git --depth=1
```

在编译过程中，Buildroot 会自动下载一些源码包。SDK V1 中已经集成了部分常用源码包，可以直接编译。SDK V2 中，为了避免由于网络原因导致下载失败，可以提前下载 `dl.tar` 包并解压到 `buildroot` 目录：
```bash
wget https://github.com/milkv-duo/duo-buildroot-sdk-v2/releases/download/dl/dl.tar
tar xvf ./dl.tar -C ./duo-buildroot-sdk-v2/buildroot/
```
解压后 `duo-buildroot-sdk-v2` 的 `buildroot` 中 `dl` 目录结构如下：
```text
└── buildroot
    └── dl
        ├── acl
        ├── alsa-lib
        ├── alsa-utils
        ├── attr
        ├── ...
```

### 进入 SDK 代码目录

```bash
cd duo-buildroot-sdk-v2
```

### 拉取 Docker 镜像并运行

```bash
docker run --privileged -itd --name duodocker -v $(pwd):/home/work milkvtech/milkv-duo:latest /bin/bash
```

命令中部分参数说明:
- `--privileged` 以特权模式启动容器。
- `duodocker` docker 运行时名字，可以使用自己想用的名字。
- `$(pwd)` 当前目录，这里是上一步 cd 到的 duo-buildroot-sdk 目录。
- `-v $(pwd):/home/work`  将当前的代码目录绑定到 Docker 镜像里的 /home/work 目录。
- `milkvtech/milkv-duo:latest` Milk-V 提供的 Docker 镜像，第一次会自动从 hub.docker.com 下载。

Docker 运行成功后，可以用 `docker ps -a` 命令查看运行状态：
```bash
$ docker ps -a
CONTAINER ID   IMAGE                        COMMAND       CREATED       STATUS       PORTS     NAMES
8edea33c2239   milkvtech/milkv-duo:latest   "/bin/bash"   2 hours ago   Up 2 hours             duodocker
```

:::tip
如果 Docker 服务器上的镜像有更新，可以使用 `docker pull milkvtech/milkv-duo:latest` 命令来同步最新的镜像。
:::

### 1. 使用 Docker 一键编译

```bash
docker exec -it duodocker /bin/bash -c "cd /home/work && cat /etc/issue && export FORCE_UNSAFE_CONFIGURE=1 && ./build.sh [board]"
```

注意命令最后的 `./build.sh [board]` 和前面在 Ubuntu 22.04 中一键编译说明中的用法是一样的，直接 `./build.sh` 可以查看命令的使用方法，用 `./build.sh lunch` 可以调出交互选择菜单，用 `./build.sh [board]` 可以直接编译目标版本，`[board]` 可以替换为:
```
milkv-duo-musl-riscv64-sd
milkv-duo256m-glibc-arm64-sd
milkv-duo256m-musl-riscv64-sd
milkv-duos-glibc-arm64-emmc
milkv-duos-glibc-arm64-sd
milkv-duos-musl-riscv64-emmc
milkv-duos-musl-riscv64-sd
```

命令中部分参数说明:
- `duodocker` 运行的 Docker 名字, 与上一步中设置的名字要保持一致
- `"*"` 双引号中是要在 Docker 镜像中运行的 Shell 命令
- `cd /home/work` 是切换到 /home/work 目录，由于运行时已经将该目录绑定到主机的代码目录，所以在 Docker 中 /home/work 目录就是该 SDK 的源码目录
- `cat /etc/issue` 显示 Docker 使用的镜像的版本号，目前是 Ubuntu 22.04.3 LTS，调试用
- `export FORCE_UNSAFE_CONFIGURE=1` Docker 中默认使用 root 用户编译，需要使用该环境变量
- `./build.sh [board]` 执行一键编译脚本

比如需要编译 `milkv-duos-musl-riscv64-sd` 的镜像，编译命令如下:
```bash
docker exec -it duodocker /bin/bash -c "cd /home/work && cat /etc/issue && export FORCE_UNSAFE_CONFIGURE=1 && ./build.sh milkv-duos-musl-riscv64-sd"
```

编译成功后可以在 `out` 目录下看到生成的SD卡烧录镜像 `*.img`

### 2. 使用 Docker 分步编译

如果未执行过一键编译脚本，需要先手动下载工具链 [host-tools](https://github.com/milkv-duo/host-tools.git)，再拷贝或移动到 SDK 根目录：

```bash
git clone https://github.com/milkv-duo/host-tools.git
cp -a host-tools duo-buildroot-sdk-v2/
```

分步编译需要登陆到 Docker 中进行操作，用命令 `docker ps -a` 查看并记录容器的 ID 号，比如 `8edea33c2239`。

如果 `duodocker` 未在列表中，可能是容器已停止，需要先将其重新运行后，再查看窗口的 ID 号：
```bash
cd duo-buildroot-sdk-v2/
docker run --privileged -itd --name duodocker -v $(pwd):/home/work milkvtech/milkv-duo:latest /bin/bash
docker ps -a
```

登陆到 Docker 中:
```bash
docker exec -it 8edea33c2239 /bin/bash
```

进入 Docker 中绑定的代码目录：
```bash
root@8edea33c2239:/# cd /home/work/
```

设置环境变量，Docker 中默认以 root 用户编译，需要配置该环境变量：
```bash
export FORCE_UNSAFE_CONFIGURE=1
```

加载环境：
```bash
source build/envsetup_milkv.sh
```
如果是第一次编译，会提示选择要编译的目标：
```bash
Select a target to build:
1. milkv-duo-musl-riscv64-sd
2. milkv-duo256m-glibc-arm64-sd
3. milkv-duo256m-musl-riscv64-sd
4. milkv-duos-glibc-arm64-emmc
5. milkv-duos-glibc-arm64-sd
6. milkv-duos-musl-riscv64-emmc
7. milkv-duos-musl-riscv64-sd
Which would you like:
```

选择对应的数字后，回车，成功加载环境变量后，会显示一些当前目标的信息，比如：
```bash
Target Board: milkv-duos-musl-riscv64-sd
Target Board Storage: sd
Target Board Config: /build/duo-buildroot-sdk-v2/device/target/boardconfig.sh
Target Image Config: /build/duo-buildroot-sdk-v2/device/target/genimage.cfg
Build tdl-sdk: 1
Output dir: /build/duo-buildroot-sdk-v2/install/soc_sg2000_milkv_duos_musl_riscv64_sd
```

环境加载完成后，会在 `device` 目录下创建一个名为 `target` 的链接，链接到编译目标目录，下次再 source 环境时，检测到该 `target` 链接存在时，就不会再提示选择目标了。如果需要更换编译目标，可以加 `lunch` 参数重新调出交互菜单进行选择：
```bash
source build/envsetup_milkv.sh lunch
```

加载环境成功后，再依次输入如下命令完成分步编译：
```bash
clean_all
build_all
```

如果编译的是 SD 卡镜像，还需要再执行如下命令生成 `img` 镜像：
```bash
pack_sd_image
```

比如需要编译 `milkv-duos-musl-riscv64-sd` 的镜像，分步编译命令如下：
```bash
source build/envsetup_milkv.sh milkv-duos-musl-riscv64-sd

clean_all
build_all
pack_sd_image
```

生成的固件位置：
```
install/soc_sg2000_milkv_duos_musl_riscv64_sd/milkv-duos-musl-riscv64-sd.img
```

注意，SD 卡镜像为 `*.img`，eMMC镜像为 `upgrade.zip`。

:::tip
除了使用 `build_all` 进行完整编译之外，还可以单独编译某个模块，一般来说都是先执行 `clean_xxx` 清理中间文件后，再执行 `build_xxx` 重新编译。支持的模块，可以输入 `clean_` 或者 `build_` 后，双击 `tab` 键查看。以下列出一些常用的模块编译方法：
- fsbl: `clean_fsbl`，`build_fsbl`
- uboot: `clean_uboot`，`build_uboot`
- kernel: `clean_kernel`，`build_kernel`
- osdrv: `clean_osdrv`，`build_osdrv`
- cvi_mpi: `clean_middleware`，`build_middleware`
- tdl-sdk: `clean_tdl_sdk`，`build_tdl_sdk`
:::

编译完成后可以用 `exit` 命令退出 Docker 环境：
```bash
root@8edea33c2239:/home/work# exit
```
在主机代码目录中同样也可以看到生成的固件。

### 停用 Docker

编译完成后，如果不再需要以上的 Docker 运行环境，可先将其停止，再删除:
```bash
docker stop 8edea33c2239
docker rm 8edea33c2239
```

## 三、其他编译注意事项

如果您想尝试在以上两种环境之外的环境下编译本 SDK，下面是可能需要注意的事项，仅供参考。

### cmake 版本号

注意：`cmake` 版本最低要求 `3.16.5`

查看系统中 `cmake` 的版本号

```bash
cmake --version
```

比如在`Ubuntu 20.04`中用 apt 安装的 cmake 版本号为

```
cmake version 3.16.3
```

不满足此SDK最低要求，需要手动安装目前最新的 `3.27.6` 版本

```bash
wget https://github.com/Kitware/CMake/releases/download/v3.27.6/cmake-3.27.6-linux-x86_64.sh
chmod +x cmake-3.27.6-linux-x86_64.sh
sudo sh cmake-3.27.6-linux-x86_64.sh --skip-license --prefix=/usr/local/
```
手动安装的 `cmake` 在 `/usr/local/bin` 中，此时用 `cmake --version` 命令查看其版本号, 应为

```
cmake version 3.27.6
```

### 使用 Windows Linux 子系统 (WSL) 进行编译

如果您希望使用 WSL 执行编译，则构建镜像时会遇到一个小问题，WSL 中的 $PATH 具有 Windows 环境变量，其中路径之间包含一些空格。

要解决此问题，您需要更改 `/etc/wsl.conf` 文件并添加以下行：

```
[interop]
appendWindowsPath = false
```

然后需要使用 `wsl.exe --reboot` 重新启动 WSL。再运行 `./build.sh` 脚本或分步编译命令。

要恢复 `/etc/wsl.conf` 文件中的此更改，请将 `appendWindowsPath` 设置为 `true`。 要重新启动 WSL，您可以使用 Windows PowerShell 命令 `wsl.exe --shutdown`，然后使用`wsl.exe`，之后 Windows 环境变量在 $PATH 中再次可用。

## 四、添加应用包

Buildroot 是一个轻量级的嵌入式 Linux 系统构建框架，其生成的系统没有像 Ubuntu 系统一样的 apt 包管理工具来下载和使用应用包。Duo 默认的 SDK 已经添加了一些常用的工具或命令，如果您需要添加自己的应用，需要对 SDK 做一些修改后重新编译生成所需的系统固件。

以下介绍在 Buildroot 中添加应用包常用的几种方法。

### 开启 Busybox 中的命令

在使用 Buildroot 构建的系统中，部分基础命令由 busybox 提供，您可以查看 busybox 的配置文件中是否有您需要的命令，将其打开后重新编译即可，busybox 的配置文件位置在：

```
buildroot/package/busybox/busybox.config
```

比如 timeout 命令，打开的方法：

```diff {9,10}
diff --git a/buildroot/package/busybox/busybox.config b/buildroot/package/busybox/busybox.config
index d7d58f064..b268cd6f8 100644
--- a/buildroot/package/busybox/busybox.config
+++ b/buildroot/package/busybox/busybox.config
@@ -304,7 +304,7 @@ CONFIG_TEST=y
 CONFIG_TEST1=y
 CONFIG_TEST2=y
 CONFIG_FEATURE_TEST_64=y
-# CONFIG_TIMEOUT is not set
+CONFIG_TIMEOUT=y
 CONFIG_TOUCH=y
 # CONFIG_FEATURE_TOUCH_NODEREF is not set
 CONFIG_FEATURE_TOUCH_SUSV3=y
```

参考该次提交: [busybox: add timeout command](https://github.com/milkv-duo/duo-buildroot-sdk/commit/2833c24f11bb48776094265b7b16cfde87f86083)

### 配置 Buildroot 中预置的应用包

另外，Buildroot 中预置了大量的应用包，通过下载源码编译的方式来生成所需的程序，Buildroot 预置的应用包可以在 `buildroot/package` 目录中查看。

配置使用或者禁用某个应用包，是在目标板的配置文件中实现的，以 `milkv-duos-musl-riscv64-sd` 目标为例，其 buildroot 配置文件是：
```
buildroot/configs/milkv-duos-musl-riscv64-sd_defconfig
```

我们可以在宿主机（比如 Ubuntu）上整体编译过一次 SDK 后，到 Buildroot 编译目录中通过命令行菜单交互的方式来配置相关的应用包。

1. 进入 Buildroot 编译目录

   ```bash
   cd buildroot/output/milkv-duos-musl-riscv64-sd/
   ```

   可以使用 `make show-targets` 命令来查看当前已经使用的应用包：

   ```bash
   $ make show-targets
   busybox coreutils dhcpcd dnsmasq dropbear duo-pinmux e2fsprogs evtest expat fio freetype gdb host-acl host-attr host-autoconf host-automake host-e2fsprogs host-fakeroot host-genimage host-libtool host-libzlib host-m4 host-makedevs host-mkpasswd host-patchelf host-pkgconf host-skeleton host-util-linux host-xz host-zlib htop ifupdown-scripts initscripts iperf3 json-c kmod libevent libffi libnl libopenssl libxml2 libxslt libzlib musl-compat-headers ncurses ntp openssl python-cffi python-evdev python-freetype python-lxml python-modbus-tk python-pillow python-pinpong python-pip python-psutil python-pycparser python-serial python-setuptools python-smbus-cffi python-spidev python3 skeleton skeleton-init-common skeleton-init-sysv spidev_test strace stress-ng tar toolchain toolchain-external toolchain-external-custom urandom-scripts util-linux wpa_supplicant zlib rootfs-ext2 rootfs-tar
   ```

2. 配置 Buildroot

   执行 `make menuconfig` 命令，调出交互菜单：

   <Image src='/docs/duo/buildroot/milkv-duo-buildroot-sdk-01.webp' maxWidth='100%' align='left' />

   在 `Target packages` 中根据分类找到所需的应用包，如果不清楚应用包具体的位置，可以按 `/` 键搜索包名，比如我们要安装 `tar` 命令，由于搜索 `tar` 会出来太多其他无关内容，可以搜索 `package_tar`，可以看到，当前 `=n` 是禁用的状态，其位置为 `Target packages` 的 `System tools` 分类中，可以双击 `ESC` 键退回到主界面再进入到相应的位置，也可以按前面提示的数字直接进入到其所在的位置：

   <Image src='/docs/duo/buildroot/milkv-duo-buildroot-sdk-02.webp' maxWidth='100%' align='left' />

   按空格键选中：

   <Image src='/docs/duo/buildroot/milkv-duo-buildroot-sdk-03.webp' maxWidth='100%' align='left' />

   连续双击 `ESC` 键退出主界面，提示是否保存时，默认是 YES, 直接回车保存退出：

   <Image src='/docs/duo/buildroot/milkv-duo-buildroot-sdk-04.webp' maxWidth='100%' align='left' />

   执行 `make savedefconfig` 命令，将修改的配置保存到原始配置文件中，用 `git status` 命令确认一下，可以看到，原始配置文件已经被修改：

   <Image src='/docs/duo/buildroot/milkv-duo-buildroot-sdk-05.webp' maxWidth='100%' align='left' />

   此时回到 SDK 根目录重新编译即可。

   :::tip
   这里也可以比较一下编译目录中的旧配置文件和新配置文件的差异，把需要更改的部分直接手动修改到原始配置文件 `milkv-duos-musl-riscv64-sd_defconfig` 中：
   ```diff
   diff -u .config.old .config
   ```
   :::

   使用新编译的镜像启动后，在 Duo 设备上测试新加的命令，如果出现 `not found` 错误，可能是该包的 `.mk` 配置文件需要补充一下 gcc 的参数，主要是 `TARGET_CFLAGS` 和 `TARGET_LDFLAGS` 两个参数要加上，可以参考如下几个提交：

   1. [buildroot: enable fio](https://github.com/milkv-duo/duo-buildroot-sdk/commit/6ebbd6e219d3efcd9b95086c75702f4f717e7f03#diff-1a84d28825d604f941100ff9b50ec8d63bf84535a3dd6f7e62c421a657e6556a)
   2. [buildroot: enable spidev_test](https://github.com/milkv-duo/duo-buildroot-sdk/commit/84d6b72eb6c9ff467410376fc982b5ee4f8e3d1f#diff-d0fb4ce2e7555c5cc5399aeaa75e4bf5fd429f034f4d00457f450a994204b97f)
   3. [buildroot: fix build parameter for coremark package](https://github.com/milkv-duo/duo-buildroot-sdk/commit/cdd4fabb6100f87a61ca7560def558cd9e38f91a)

### 添加自己的应用包

编译测试自己的应用，不推荐集成到 Buildroot 工程中的方法。建议使用 [duo-examples](https://github.com/milkv-duo/duo-examples/blob/main/README-zh.md) 的方式。

如果确实需要将自己的应用以 Buildroot package 方式编译，可以参考 Buildroot 中预置包的配置进行添加，以下是几个参考链接：

1. [buildroot: add python-evdev required by the pinpong library](https://github.com/milkv-duo/duo-buildroot-sdk/commit/11caee79c7a48f7f6fd4a5d352ccf8976857d522)
2. [buildroot: add python-freetype required by the pinpong library](https://github.com/milkv-duo/duo-buildroot-sdk/commit/165fc3c4c1adca1d1f635fd2069a5ad67d895c7c)

主要参考 `buildroot-2021.05/package` 中添加的内容。

## 五、删除不需要的应用包

如果你在自行编译固件的过程中，需要删除一些不需要的应用包来加快编译速度，或者禁用一些不需要的包，可以采用上述打开 Buildroot 应用包的 `make menuconfig` 方式，禁用相关的包。

也可以在 Buildroot 的配置文件中将对应的包名删除，以 `milkv-duos-musl-riscv64-sd` 目标为例，比如不需要编译 Python 相关的库，可以做如下修改后，重新编译生成固件即可。

```diff title="buildroot/configs/milkv-duos-musl-riscv64-sd_defconfig"
diff --git a/buildroot/configs/milkv-duos-musl-riscv64-sd_defconfig b/buildroot/configs/milkv-duos-musl-riscv64-sd_defconfig
index 2bc8cd5e3..e78901afb 100644
--- a/buildroot/configs/milkv-duos-musl-riscv64-sd_defconfig
+++ b/buildroot/configs/milkv-duos-musl-riscv64-sd_defconfig
@@ -330,25 +330,6 @@ BR2_PACKAGE_EVTEST=y
 # BR2_PACKAGE_FCONFIG is not set
 BR2_PACKAGE_FLASHROM_ARCH_SUPPORTS=y
 
-BR2_PACKAGE_PYTHON3=y
-BR2_PACKAGE_PYTHON3_PY_PYC=y
-BR2_PACKAGE_PYTHON_LXML=y
-BR2_PACKAGE_PYTHON_PIP=y
-BR2_PACKAGE_PYTHON_SETUPTOOLS=y
-BR2_PACKAGE_PYTHON3_SSL=y
-
-BR2_PACKAGE_PYTHON_SERIAL=y
-BR2_PACKAGE_PYTHON_PILLOW=y
-BR2_PACKAGE_PYTHON_SMBUS_CFFI=y
-BR2_PACKAGE_PYTHON_SPIDEV=y
-BR2_PACKAGE_PYTHON_MODBUS_TK=y
-BR2_PACKAGE_PYTHON_EVDEV=y
-BR2_PACKAGE_PYTHON_FREETYPE=y
-
-BR2_PACKAGE_PYTHON_PINPONG=y
-
-BR2_PACKAGE_PYTHON_PSUTIL=y
-
 #
 # Compression and decompression
 #
```

## 六、常见问题

### Buildroot 编译出错排查方法

SDK 中 Buildroot 默认开启了顶层并行编译以加快编译速度，但是编译出错时，不方便分析出错的日志，所以我们可以先在 config 文件中将其删除，待排解决了问题之后，再将其重新打开。

以 `milkv-duos-musl-riscv64-sd` 目标为例，在其配置文件中将该配置删除后，再删除 `buildroot/output` 目录，重新编译：

```bash title="buildroot/configs/milkv-duos-musl-riscv64-sd_defconfig"
BR2_PER_PACKAGE_DIRECTORIES=y
```

编译出错时除了查看编译终端的出错信息，还可以查看 `build/br.log` 中的完整日志进行排查。

### Buildroot 软件包下载失败

在编译过程中，Buildroot 会自动下载一些源码包。SDK V1 中已经集成了部分常用源码包，可以直接编译。SDK V2 中，为了避免由于网络原因导致下载失败，可以提前下载 `dl.tar` 包并解压到 `buildroot` 目录后，再重新编译：
```bash
rm -rf ./buildroot/output ./buildroot/dl
wget https://github.com/milkv-duo/duo-buildroot-sdk-v2/releases/download/dl/dl.tar
tar xvf ./dl.tar -C ./buildroot/
```

解压后 `buildroot` 中 `dl` 目录结构如下：
```text
└── buildroot
    └── dl
        ├── acl
        ├── alsa-lib
        ├── alsa-utils
        ├── attr
        ├── ...
```
