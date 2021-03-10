---
layout: post
title: npm install安装缓慢或超时
categories: [Node,npm]
description: npm install安装缓慢或超时
keywords: npm,node
---

npm install的时候有时候特别慢，怎么也安装不上。可以试试试用其他的镜像源，比如淘宝的，切换镜像源需要安装nrm命令工具：

```bash
npm i -g nrm
```

查看可用的镜像：

```bash
nrm ls
```

![](/images/posts/node/npm-install1.png)

切换镜像源到toabao:

```bash
nrm use taobao
# 查看当前镜像源
nrm ls
```

![](/images/posts/node/npm-install2.jpg)

再使用npm install安装试试：

```bash
npm install
```

顺利安装！