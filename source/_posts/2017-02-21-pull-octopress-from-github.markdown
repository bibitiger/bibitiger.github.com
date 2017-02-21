---
layout: post
title: "将已有octopress从github搬移到本地"
date: 2017-02-21 14:22:35 +0800
comments: true
categories: "Octopress"
---


##原文地址：[blog.bibitiger.cn/blog/2017/02/21/pull-octopress-from-github/](http://blog.bibitiger.cn/blog/2017/02/21/pull-octopress-from-github/)

</br>

---

</br>
当我们换了电脑或者遗失了原本项目的时候，我们就需要将原来我们部署在github上的octopress博客重新搬移到本地。这里我们根据之前建立octopress项目的过程，首先清楚两个分支各自的功能：

| source | octopress文件及代码，假如我们的工程根目录为octopress，source对应的就是octopress |
|:---:|:---:|
| master | octopress deploy时生成的缓存文件，可以认为是_deploy文件夹，对应于octopress/_deploy |

<!--more-->

所以我们需要下载的是octopress的内容文件代码：

```
git clone -b source git@github.com:username/username.github.com.git octopress
```

接着下载发布预览内容

```
cd octopress
git clone git@github.com:username/username.github.com.git _deploy
```

安装依赖项

```
gem install bundler # Install dependencies
bundle install
```

由于我们是以前就建立好的工程，所以没有必要去运行`rake install`，如果运行的话，反而会冲掉我们之前设置好的theme。

现在就可以正式使用我们的工程了，可以先预览一下

```
rake generate  //生成
rake preview   //预览
```

这里我遇到了一个问题`=> Creating Categories Tag Cloud
     Build Warning: Layout 'nil' requested in atom.xml does not exist.
                    done.`将octopress/source/atom.xml里的`layout: nil`改为`layout: null`即可。
                    
之后新建博客发布之类的和原来的初建时一样。