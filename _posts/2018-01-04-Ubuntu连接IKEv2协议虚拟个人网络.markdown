---
layout: post
author: JachinShen
title:  "Ubuntu 使用 IKEv2 协议连接虚拟个人网络"
subtitle: "配了半天环境。。。"
date:   2018-01-04  20:55:33 +0800
categories: Environment
tags: 
    - Environment
    - Linux
    - IKEv2
---
# 为什么要配这个环境

上海交通大学网络信息中心，提供了学生虚拟个人网络服务，用于校外访问校园网资源。看介绍不用申请就可以使用，现在配好环境，以后回家也可以用校园网了！

## 我是怎么一步一步配不出环境的

### 学校告诉我怎么配环境

学生虚拟个人网络只支持 IKEv2 协议，学校给出的配置有Windows、有OS X、有Android，可就是没有 Ubuntu！（在此为 Linux 桌面版的存在感默哀）但是身为一个 Linux 用户，怎么能没有折腾的精神呢？所以开始了配置 Ubuntu 下 IKEv2 协议之路。

### 当然是先搜一下啦

刚开始 Google 了一下关键字“Linux IKEv2”，然而得到的搜索结果都是关于如何在 Linux 下配置服务器端，看来没人在 Linux 下用客户端啊。

### 怎么能轻言放弃

但是，还是有蛛丝马迹！

首先是摸到了这条线索[ubuntu 作为 IKEV2 VPN 客户端无法上网](https://segmentfault.com/q/1010000008476517)，看标题很切合需求，但是没有找到配置指南。

继续往下翻，找到了另一条线索[Strongswan ikev2 VPN 配置](https://oogami.name/1467/)，这里有 Ubuntu 客户端的配置。办法就安装是 `strongswan`。在 Ubuntu 下 `apt-cache search strongswan` 搜索相关软件包，发现刚好有，那当然是直接用别人编译好的软件包了。

然而，进一步照教程走下去，发现这是需要和服务端统一的，需要用密钥文件，可学校的虚拟个人网络是用户密码登陆的，感觉这种情况下，图形界面更高效。

### 总算有了一线希望

既然想到了图形界面，就自然而然地想到了 nm-applet，印象里有 pptp 协议，增加一个 IKEv2 协议应该不难吧。

### 开始研究 nm-applet

这里 nm-applet 的界面设计就有点违反直觉了。nm-applet 的下拉菜单里有一个“VPN连接”选项，一般人都以为在这里设置连接，但是点进去，显示灰色的“Add a VPN connection”，点击也没有反应，很不直观。

后来试试各种选项，才发现居然在“编辑连接”里，可以创建连接。不过默认只有 PPTP 协议。好在有提示：“如果您要创建 VPN 连接，但需要的类型不在列表中，您可能没有安装相应的 VPN 插件。”那找找nm-applet的插件应该就可以了。

#### 当然是先找中文资料了

刚开始还是想找中文教程，就以关键字“nm-applet vpn 插件”搜索，在 ArchWiki 里找到了 和networkmanager 相关的线索[NetworkManager](https://wiki.archlinux.org/index.php/NetworkManager_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))，讲得很详细，可是并没有IKEv2的信息。

#### 后来还是找英文资料了

中文教程找不到，只能找英文的了。搜索关键字“nm-applet vpn plugin ikev2”，发现了 networkmanager 的 strongswan 插件[Network-Manager-StrongSwan](https://wiki.strongswan.org/projects/strongswan/wiki/NetworkManager)，最后有安装方法，但是看着一大堆步骤，感觉很麻烦，就先在 Ubuntu 下 `apt-cache search stringswan` ，发现果然有 strongswan-nm，就直接安装了。

#### 但是还是不行

重启 nm-applet ，创建 VPN ，却还是没有这个选项，只能回去再 Google。

#### 万能的 Google！

这时候，Google 的一个推荐关键字引起了我的注意“nm applet vpn grayed out”，意思是那个“ VPN 连接”的地方灰色，点了没反应，和我之前的经历一样，就顺便搜搜看究竟是为什么。

得到的答案是：nm-applet 创建 VPN 的地方不是在“VPN连接”里，而是在“编辑连接”里，选择创建 VPN。

至少解决了一个疑问，但还是没找到可行的IKEv2协议插件。于是又专门搜索了“network-manager-strongswan”，发现许多人遇到了一样的问题[NetworkManager-StrongSwan](https://github.com/trailofbits/algo/issues/263)，原来这是一个旧版本的 bug，但是软件库还没有收录新版本，最后在这个 Ubuntu 的论坛上，找到了终极解决办法[Ubuntu论坛](https://ubuntuforums.org/showthread.php?t=2327303)！那就是：按照官网的办法老老实实编译安装。

#### 又回到了最初的地方

没办法，只能编译安装了。安装过程中会遇到依赖包缺失的问题，`apt-get` 安装一下就好了。需要注意的是，缺失的包名和软件库中的名字很相近，但不完全相同，要先`apt-cache search` 找到对应包再安装。

```shell
sudo apt-get install libnm-util-dev libnm-util2 libnm-glib-dev libnm-glib4 libnm-glib-vpn1 libnm-glib-vpn-dev
sudo apt-get install libnm-gtk0 libnm-gtk-common libnm-gtk-dev
sudo apt-get install libnma0 libnma-common libnma-dev
```

编译安装成功后，打开nm-applet，编辑连接，创建vpn，终于有IKEv2的选项了，然而按照学校在其他平台上的说明，填入用户名密码，还是没有成功。。。

## 反思

- 英文的资料的确比中文资料更加丰富及时，不要畏惧找英文资料，最后会发现直接找英文资料其实更加高效
- 善用搜索引擎，它能解决你遇到的一大半问题
- 不要盲目用软件仓库的东西，必要时还是要自己编译安装
