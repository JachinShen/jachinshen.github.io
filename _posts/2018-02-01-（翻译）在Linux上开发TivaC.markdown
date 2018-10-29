---
layout: post
author: JachinShen
title:  "（翻译）在 Linux 上开发 TivaC "
subtitle: "不禁想起了高中的时光"
date:   2018-02-01  13:21:00 +0800
categories: Environment
tags: 
    - Environment
    - ARM
    - LaunchPad
    - Linux
    - tools
---

高中时，在 Texas Instrument 和上海交通大学电院举办的单片机比赛中拿了奖，得到了一块 `Tiva C series TM4C123GXL` 系列单片机。

之前用的都是 `MSP430G2` 系列单片机，在 `energia` 里面开发。`Tiva C series TM4C123GXL`虽然也能在`energia`里开发，但是有闪存地址限制。

高中时折腾了半天也没配好环境，只能回去学习了。到了大学寒假，时间充裕些，想想过去了两三年，学习资料应该丰富了很多，又已经习惯了在 Linux 下开发，所以试着在 Linux 下配好 `Tiva C series TM4C123GXL` 的开发环境。

Google 了一下后，马上找到了一篇很好的指导文章，不过是英文的。为了方便后人，在此把它翻译成中文。

[原文在此](http://chrisrm.com/howto-develop-on-the-ti-tiva-launchpad-using-linux/)

---

# 如何在 Linux 下开发 TI Tiva 系列开发板

2014年6月28日，作者Chris Miller

这几年来，我几乎一直在用 Atmel 的 AVR 系列 8 位微处理器做项目。AVR 系列和 Linux 兼容很好，所以我一直这样开发。但是我听说 TI 的板子很便宜、工具链很好用，所以我试着买了一块开发板。

然而事实证明，哎呀！我买错了板子。原本我想买 10 美元的[MSP-EXP430G2](http://www.ti.com/tool/MSP-EXP430G2)开发板，但是我不小心订成了 13 美元的[Tiva C series TM4C123GXL](http://www.ti.com/tool/ek-tm4c123gxl)开发板。钱是小事，问题在于这是 ARM 开发板！

所以要用什么工具链呢？我是不是要下载一下臃肿的 IDE 呢？搜索了一番后，还好，编译、烧录和 debug 都可以用 Linux 的命令行工具！

接下来的安装指南来自 [Tiva Template](https://github.com/uctools/tiva-template) 和 [Recursive Labs Blog](http://recursive-labs.com/blog/2012/10/28/stellaris-launchpad-gnu-linux-getting-started)，以及我的一些笔记和小技巧。我的工作目录在 ~/Embedded，系统是 Ubuntu 13.10。

## 依赖

这个部分只需要执行一次。

1. 创建一个工作目录来放项目和工具链。

    ```shell
    mkdir ~/Embedded
    cd ~/Embedded
    ```

2. 用下面的命令安装一些依赖。现在 32 位和 64 位系统应该都可以进行这步了（感谢 Mayank！）如果你恰好是 Arch Linux 用户，最好根据 Vivek 在注释中的建议检查你的系统，便于 64 位环境的配置。

    ```shell
    # 对于 64 位系统：
    sudo dpkg --add-architecture i386

    # 对于所有人：
    sudo apt-get update
    sudo apt-get install flex bison libgmp3-dev libmpfr-dev \
        libncurses5-dev libmpc-dev autoconf texinfo build-essential \
        libftdi-dev python-yaml zlib1g-dev libtool

    # 再次对于 64 位系统：
    sudo apt-get install libc6:i386 libncurses5:i386 libstdc++6:i386

    # 通过 USB 烧录
    sudo apt-get install libusb-1.0-0 libusb-1.0-0-dev

    ```

3. 进入[GNU Tools for ARM Embedded Processors](https://launchpad.net/gcc-arm-embedded/+download)页面下载最新版的 Linux tar 压缩包。然后解压到 *~/Embedded* （包括顶层目录 *gcc-arm-none-eabi-...* ）。

    ```shell
    tar -xvf gcc-arm-none-eabi-4_8-2014q2-20140609-linux.tar.bz2 -C ~/Embedded
    ```

4. 把解压后目录的 bin 目录添加到你的环境变量 PATH。

    ```shell
    export PATH=$PATH:$HOME/Embedded/gcc-arm-none-eabi-4_8-2014q1/bin
    ```

    这个只在你登陆后有效，而且每次你要开发是都要重新运行。更长久的解决方案如下：
    - 把 *gcc-arm-none-eabi-.../bin* 目录下的东西复制到你的环境变量指向的目录之一（不推荐）
    - 修改`.bashrc`文件，把这句命令加在最后一行，每次打开终端就会自动执行。

5. 从 TI 的[Tiva C Series Software section](http://software-dl.ti.com/tiva-c/SW-TM4C/latest/index_FDS.html)下载`TivaWare for Tiva C Series`包。你需要登陆 TI 帐号来下载。我下载时，最新版本是 *SW-TM4C-2.1.0.12573.exe* 。

6. 使用下面的命令，在 *Embedded* 里创建*tivaware*文件夹，并把 exe 文件解压到新文件夹。

    ```shell
    cd ~/Embedded
    mkdir tivaware/
    cd tivaware/
    # 移动下载好的 exe 文件
    mv ~/Downloads/SW-TM4C-2.1.0.12573.exe . #不要漏了这个点
    unzip SW-TM4C-2.1.0.12573.exe
    ```

7. 使用 `make` 编译

    ```shell
    make
    ```

8. [Tiva Template](https://github.com/uctools/tiva-template)有我们需要用来编译的东西，以及一些样例。用下面的命令，从 GitHub 上拉下来。

    ```shell
    cd ~/Embedded
    git clone git@github.com:uctools/tiva-template
    ```

9. lm4flash 是我们用来烧录板子的通用工具。从 GitHub 上拉下来，编译源码。

    ```shell
    cd ~/Embedded
    git clone git://github.com/utzig/lm4tools.git
    cd lm4tools/lm4flash/
    make
    ```

10. lm4flash 开箱即用，但需要 `sudo` 来通过 USB 和开发板通讯。你可以用下面的命令为这个设备设置一些 udev 规则，并且把你的帐号添加到 *dialout* 群组来修正。 **然后拔下开发板，注销，登陆，再重连开发板**。

    ```shell
    cd /etc/udev/rules.d
    echo &quot;SUBSYSTEM==\&quot;usb\&quot;, ATTRS{idVendor}==\&quot;1cbe\&quot;, ATTRS{idProduct}==\&quot;00fd\&quot;, MODE=\&quot;0660\&quot;&quot; | sudo tee 99-tiva-launchpad.rules
    # 记得拔下接口并重连
    ```

11. 我们甚至可以 debug。OpenOCD，也就是 Open On-Chip Debugger，现在支持这块板子。拉下来编译。

    ```shell
    cd ~/Embedded
    git clone http://openocd.zylin.com/openocd
    cd openocd
    git fetch http://openocd.zylin.com/openocd refs/changes/63/2063/1
    git checkout FETCH_HEAD
    git submodule init
    git submodule update
    ./bootstrap
    ./configure --enable-ti-icdi --prefix=`pwd`/..
    make -j3
    make install
    ```

## 编译固件

我们将用 Tiva Template 里的源代码和 Makefile 编译一个简单的 LED 闪光应用。

1. 到 *Embedded/tiva-template-master/src/main.c* 看看怎么让 LED 闪光的。
2. 像下面这样设置 Makefile：

    ```shell
    #######################################
    # user configuration:
    #######################################
    # TARGET: name of the output file
    TARGET = main
    # MCU: part number to build for
    MCU = TM4C123GH6PM
    # SOURCES: list of input source sources
    SOURCES = main.c startup_gcc.c
    # INCLUDES: list of includes, by default, use Includes directory
    INCLUDES = -IInclude
    # OUTDIR: directory to use for output
    OUTDIR = build
    # TIVAWARE_PATH: path to tivaware folder
    TIVAWARE_PATH = $(HOME)/Embedded/tivaware
    ```

3. 执行`make`来编译。

    ```shell
    cd ~/Embedded/tiva-template-master
    make
    ```

4. 应该编译成功了，编译的文件在 *build* 文件夹。

## 烧录

1. 运行下面的命令，通过 USB 烧录。如果你在依赖的第十步没有设置 udev 和群组，记得用 `sudo` 运行。

    ```
    cd ~/Embedded/lm4tools/lm4flash/
    ./lm4flash ~/Embedded/tiva-template-master/build/main.bin
    ```

2. 哇！ reset 按钮下的 RGB LED 已经如愿所偿地亮起了红色。现在可以在 main.c 里面更改延时和颜色来玩玩了。

## 参考资料

- GitHub – Tiva Template: [A template for building firmware for Texas Stellaris ARM microcontrollers](https://github.com/uctools/tiva-template)
- Recursive Labs Blog: [Programming the Stellaris Launchpad with GNU/Linux](http://recursive-labs.com/blog/2012/10/28/stellaris-launchpad-gnu-linux-getting-started/)
- Stack Overflow: [Sample UDEV file for Stellaris Launchpad](http://stackoverflow.com/questions/14174601/sample-udev-file-for-stellaris-launchpad)
- Reactivated: [Writing udev rules](http://www.reactivated.net/writing_udev_rules.html#ownership)
