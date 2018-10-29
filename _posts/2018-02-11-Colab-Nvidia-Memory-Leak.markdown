---
layout: post
author: JachinShen
title:  "Google Colab Nvidia Memory Leak Solution"
subtitle: "感谢 Google !"
date:   2018-02-11  21:37:42 +0800
categories: DeepLearning
tags: 
    - DeepLearning
    - Google
    - Cloud
    - Nvidia
    - Linux
---
# Google Colab

Google 的 Colab 真是良心神器，可以免费使用 Nvidia K80 GPU 进行深度学习的训练。虽然已经是很老的 GPU 了，但是对于一些小项目绰绰有余，比起自己的小笔记本，不知道快到哪里去了。

## 问题

刚开始用得很开心，训练模型再也不用等一晚上了。但是用了没多久，再运行 `session` 的时候，就会爆出 `ResourcesExhausted` 的错误。

一开始以为显存和其他用户共享了，想着也许把网络改简单一点，`batch_size` 改小一点，勉强也可以跑跑。然而依然会报错。

这就感觉有点不太对劲，查看了显卡信息`!/opt/bin/nvidia-smi`，发现了问题：显存占用了 90%，只剩下了 200M 左右，怪不得资源不足。

## 原因分析

### 假设1：显存共享

Colab 毕竟是一个共享的平台，Google 很有可能划分了很多虚拟区域，那显存也可能被其他用户共享了

#### 验证

如果真的是共享的，那显存应该是动态变化的。于是不断地运行`!/opt/bin/nvidia-smi`，观察显存的变化。然而，显存却是一个常值，因此，这个假设排除了。

### 假设2：强行中断

刚开始用 Colab 的时候，经常等不及训练完成就强行中断，而这恰恰是 `session` 在运行、显存被占用的时候。强行终止，没有释放显存的过程，很容易造成显存的泄露。

联想到自己刚开始在笔记本上跑的时候，强行中断的确会造成内存的泄露，只有终止 `jupyter notebook` ，内存才会回来。所以，应该就是这个原因。

## 解决

### 思路

既然问题是显存泄露，杀死对应的主程序，就可以回收对应的显存了。那么是哪个主程序呢？

### 嫌疑程序

运行 `!top` 查看，感觉有关系的只有 `python3, python, jupyter` 这三个了。但这些都是和浏览器页面交互的程序，删了以后不就没法交互了吗？

### 第一次尝试

犹豫了很久，还是下决心，删！

```shell
!kill PID(python3对应的PID)
```

不得不感叹 Google 的周到，杀死之后，页面马上就提示“代理程序崩溃，正在重启”。

页面重启后，运行`!/opt/bin/nvidia-smi`，发现显存的确占用少了，但只有一点点，剩下的也就 250 M，依然不能跑模型啊！

### 第二次尝试

至少验证了之前的想法，的确是有程序泄露了内存，所以只要找到那个程序，就可以回收内存了。

剩下来最有可能的，就是 `python` 了。既然会自动重启，也就无所畏惧了。

```shell
!kill PID(python对应的PID)
```

运行`!/opt/bin/nvidia-smi`，奇迹发生了！显存占用为零！没错！就是`python`泄露了显存！

## 反思

1. TensorFlow Session 运行时不要强行中断，除非你明白你在做什么；
2. Google Colab 很棒，要善于利用；
3. 内存/显存泄露时，删除泄露的程序，一般情况下可以回收。
