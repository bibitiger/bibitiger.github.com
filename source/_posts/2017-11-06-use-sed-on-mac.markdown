---
layout: post
title: "sed在mac上的一些问题"
date: 2017-11-06 23:15:49 +0800
comments: true
categories: "shell"
---

##原文地址：[blog.bibitiger.cn/blog/2017/11/06/use-sed-on-mac/](http://blog.bibitiger.cn/blog/2017/11/06/use-sed-on-mac/)

<br/>

---

今天要做一个文本拼接的活，本来很简单的事情，使用 sed /a /s轻轻松松就能搞定的事，结果因为mac的原因搞得简直痛不欲生。

让我们看下面一段代码

```
	$: touch test.json
	$: echo "[{"test":["abc","edf"]}" >> test.json
	$: sed -i '/test/a ,{"test1":["qwe"]}]' test.json
	$: sed -i 's/qwe/uio/g' test.json
```

本来我们应该得出的结果是什么呢？

```
	$: cat test.json
		[{test:[abc,edf]}
		,{"test1":["uio"]}]
```

可是就是这么简单的语句，竟然出了问题，什么鬼呢？
`sed: 1: "/a ,{"test1":["qwe"]}]": command a expects \ followed by text`

<!--more-->

翻看man下的帮助文档，也是再没见过这样的了，说的不清不楚，也没有e.g.，看着网上的解释之类的，尝试了-i后面加“”的，也是不行，接着尝试加换行"\n"，还是不行，各种折腾。硬着头皮又读了一遍帮助，再次摸索，正解应该是：

```
 $: sed -i "" "/test/a\\
 	,{\"test1\":[\"qwe\"]}]" test.json
```

必须要在/a后面加上\\然后换行，而且必须是双引号，这样子我们这句话就加入进去了。

对了，-i后面的 "" 这个也是必须的。。。。 如果不写入内容，就直接修改当前文件，如果写入了的话就会生成缓存文件，比如写入"_bat"，原文件内容导入test.json_bat中，这个文件自动生成。

大家用了之后会发现这样也有一个问题，我们插入的内容没有自带换行，这个时候我们应该这样写以确保换行：

```
$: sed -i "" "/test/a\\
 	,{\"test1\":[\"qwe\"]}] \\
 	" test.json
```

对的，还是双斜杠！！！

这个写法实在是有些别扭，而且非常不利于编写，并且还和其他的unix系列系统不一致，所以解决这个问题的终极解决办法是：
###切换成<u>***gnu_sed***</u>

1.brew install gnu-sed --with-default-names

2.vi ~/.zshrc
export PATH="/usr/local/opt/gnu-sed/libexec/gnubin:$PATH"

3.source ~/.zshrc 或者新开窗口，让设置生效

现在我们再使用sed是不是顺手多了，又回到了我们所熟知的领域，尽情玩耍吧