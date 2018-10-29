---
layout: post
author: JachinShen
title:  "Markdown中，Latex嵌入{}注意"
subtitle: "简单记录一下"
date:   2018-03-02  14:06:58 +0800
categories: Tips
tags: 
    - Tips
    - Markdown
    - Latex
---
# Markdown中，Latex嵌入{}注意

由于 `{}` 是 Latex 语法的一部分，并不能直接显示。

比如：

$$\mathbf{A} = {1 ,2 ,3 ,4 ,5}$$

```latex
$$\mathbf{A} = {1 ,2 ,3 ,4 ,5}$$
```

## 解决方法一（推荐）

用`\lbrace`和`\rbrace`代替。

- 优点：
  - 显示正确

- 缺点：
  - 打字麻烦
  - 不直观

$$\mathbf{A} = \lbrace 1 ,2 ,3 ,4 ,5 \rbrace $$

```latex
$$\mathbf{A} = \lbrace 1 ,2 ,3 ,4 ,5 \rbrace $$
```

## 解决方法二

用`\{`和`\{`代替。

- 优点：
  - 直观
  - 打字方便

- 缺点：
  - 本地预览时也许可以，上传到 Github 生成 Pages 时，可能出错。

$$\mathbf{A} = \{ 1 ,2 ,3 ,4 ,5 \} $$

```latex
$$\mathbf{A} = \{ 1 ,2 ,3 ,4 ,5 \} $$
```

## 绝对值符号

在表格中使用时造成困扰：

| $|A|$ | $|B|$ |
|:---:|:---:|
|  1  |  2  |

```latex
| $|A|$ | $|B|$ |
|:---:|:---:|
|  1  |  2  |
```

### 解决办法

用`\lvert`和`\rvert`代替

| $\lvert A \rvert $ | $ \lvert B \rvert $ |
|:---:|:---:|
|  1  |  2  |

```latex
| $\lvert A \rvert $ | $ \lvert B \rvert $ |
|:---:|:---:|
|  1  |  2  |
```
