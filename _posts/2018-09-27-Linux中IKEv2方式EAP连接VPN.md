---
layout: post
author: JachinShen
title:  "Linux 中使用 IKEv2 方式连接 VPN"
subtitle: "终于配好了环境"
date:   2018-09-26  16:05:00 +0800
categories: Environment
tags: 
    - Environment
    - Linux
    - IKEv2
---

# Linux中IKEv2方式EAP连接VPN

## 什么是IKEv2

根据[维基百科](https://en.wikipedia.org/wiki/Internet_Key_Exchange)介绍，IKEv2全程为Internet Key Exchange第二版，所以本质上是一种协议，只要找到实现这个协议的软件就可以连接了。

## Linux下IKEv2软件

Windows 和 Mac OS 下，系统都自带了IKEv2协议的实现。Linux下则需要通过额外安装的软件实现，其中StrongSwan是比较成熟的一套软件。

## StrongSwan基本使用

在官网[介绍](https://wiki.strongswan.org/projects/strongswan/wiki/IntroductionTostrongSwan)的Invocation and Maintenace章节中，介绍了基本的使用方法：

- `ipsec start` 启动服务
- `ipsec up <name>`启动名为name的vpn
- `ipsec down <name>`关闭名为name的vpn
- `ipsec reload`更改ipsec.conf后，重新加载配置文件，使得配置生效
- `ipsec restart`如果更改了很多地方，`ipsec reload`无效，就直接重启

需要注意的是，这些命令都需要root权限，所以要在前面加上`sudo`。

## 配置文件

在此我需要连接的是上海交通大学提供的访问校园网用VPN，采用IKEv2协议，EAP认证方式，也就是通过帐号密码认证。

`/etc/ipsec.conf`文件配置如下：

```
# ipsec.conf - strongSwan IPsec configuration file

# basic configuration

config setup
	# strictcrlpolicy=yes
	# uniqueids = no

# Add connections here.

conn sjtu
	left=%config
	leftsourceip=%config
	leftauth=eap-gtc
	right=stu.vpn.sjtu.edu.cn
	rightsubnet=0.0.0.0/0
	rightid=@stu.vpn.sjtu.edu.cn
	rightauth=pubkey
	eap_identity=jAccount
	auto=add

```

其中`left`指的是客户端，用`%config`是希望使用分配到的内网ip，认证方式为`eap-gtc`（更新：原本为`eap`，可能会使用默认的认证方式，导致认证失败，需要指定为GTC模式。另外，从源码编译时，需要添加GTC模块`--enable-eap-gtc`）；`right`指的是服务端，认证方式为公钥；eap的帐号就是jAccount的帐号。

`/etc/ipsec.secrets`文件配置如下：

```
jAccount : EAP "Your Password"
```

这个文件是用来储存密码的，将`jAccount`和`Your Password`替换成自己的帐号密码即可。

## 证书拷贝

StrongSwan默认并不会用系统的证书来认证网站，而是用自己目录`/etc/ipsec.d/cacerts`下的证书。方便起见，可以将系统的证书全部复制到该目录下，之后就会自动寻找合适的证书。

Linux下的证书放置的位置不一定相同，下面以常见的`/etc/ssl/certs/`目录为例：

```bash
sudo cp -r /etc/ssl/certs/* /etc/ipsec.d/cacerts
```

## 连接

配置好之后，就可以连接了：

- `sudo ipsec start`
- `sudo ipsec up sjtu`

连接成功的信息如下：

```
parsed IKE_AUTH response 3 [ EAP/REQ/GTC ]
server requested EAP_GTC authentication (id 0x02)
generating IKE_AUTH request 4 [ EAP/RES/GTC ]
sending packet: from xx.xx.xx.xx[4500] to xx.xx.xx.xx[4500] (96 bytes)
received packet: from xx.xx.xx.xx[4500] to xx.xx.xx.xx[4500] (80 bytes)
parsed IKE_AUTH response 4 [ EAP/SUCC ]
EAP method EAP_GTC succeeded, no MSK established
authentication of 'config' (myself) with EAP
generating IKE_AUTH request 5 [ AUTH ]
sending packet: from xx.xx.xx.xx[4500] to xx.xx.xx.xx[4500] (112 bytes)
received packet: from xx.xx.xx.xx[4500] to xx.xx.xx.xx[4500] (272 bytes)
parsed IKE_AUTH response 5 [ AUTH CPRP(ADDR DNS DNS) SA TSi TSr N(AUTH_LFT) N(MOBIKE_SUP) N(NO_ADD_ADDR) ]
authentication of 'stu.vpn.sjtu.edu.cn' with EAP successful
IKE_SA sjtu[5] established between xx.xx.xx.xx[config]...xx.xx.xx.xx[stu.vpn.sjtu.edu.cn]
scheduling reauthentication in 9938s
maximum IKE_SA lifetime 10478s
installing DNS server xx.xx.xx.xx via resolvconf
installing DNS server xx.xx.xx.xx via resolvconf
installing new virtual IP xx.xx.xx.xx
CHILD_SA sjtu{5} established with SPIs xxxxxxxx_x xxxxxxx_x and TS xx.xx.xx.xx/32 === 0.0.0.0/0
received AUTH_LIFETIME of 85520s, reauthentication already scheduled in 9938s
peer supports MOBIKE
connection 'sjtu' established successfully

```

成功之后可以在浏览器中打开net.sjtu.edu.cn，观察页面底部的ip地址，来验证VPN是否真的启用了。

## 调试

如果第一次没有成功，可以通过以下流程调试：

- 看日志，找出错误位置
- 更改配置文件
- `sudo ipsec restart`重启
- `sudo ipsec up sjtu`再次尝试

## PS

- 为什么不用NetworkManager的插件？因为试过了连接不上，应该是支持的EAP认证方式过少的原因。
- 为什么不用简化过的命令行工具`charon-cmd`？因为试过了连不上，也是支持的EAP认证方式过少的原因。
