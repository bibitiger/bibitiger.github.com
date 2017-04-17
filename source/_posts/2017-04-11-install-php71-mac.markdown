---
layout: post
title: "在mac上安装php7.1"
date: 2017-04-11 17:10:47 +0800
comments: true
categories: "PHP"
---

##原文地址：[blog.bibitiger.cn/blog/2017/04/11/install-php71-mac/](http://blog.bibitiger.cn/blog/2017/04/11/install-php71-mac/)

<br/>

---


mac上预装了PHP，但是版本是5.5.x，由于有的时候要用最新的版本，或者项目需要，我们需要不同版本的PHP经行开发。有多种办法可以搞定，比如说如果是团队开发，版本控制的话可以使用***Vagrant***虚拟机，这个之后再说，今天就简单的将mac上的PHP升级，以7.1版本为例，其实很简单。

```
curl -s https://php-osx.liip.ch/install.sh | bash -s 7.1
```

在终端里运行，时间可能会有点长，视个人网络情况而定。

<!--more-->

打开预处理文件***~/.bash_profile***,在最后加入环境变量

```
export PATH=/usr/local/php5/bin:$PATH;
```

让我们的配置生效就可以了

```
source ~/.bash_profile
```

现在我们查看版本就能看到我们的PHP已经升级到了7.1

```
$  php -v
PHP 7.1.1 (cli) (built: Feb 13 2017 10:05:49) ( NTS )
Copyright (c) 1997-2017 The PHP Group
Zend Engine v3.1.0, Copyright (c) 1998-2017 Zend Technologies
    with Zend OPcache v7.1.1, Copyright (c) 1999-2017, by Zend Technologies
    with Xdebug v2.5.0, Copyright (c) 2002-2016, by Derick Rethans
```