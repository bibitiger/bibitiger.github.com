---
layout: post
title: "解决github-pages无法被百度抓取问题（octopress）"
date: 2017-02-23 11:47:23 +0800
comments: true
categories: "Octopress"
---

<br>

---

</br>

网上已经有很多关于这个问题的解决方案,例如CDN到七牛等的镜像缓存、修改ip代理、云服务器等等，例如[知乎上的问答“如何解决百度爬虫无法爬取搭建在Github上的个人博客的问题？”](https://www.zhihu.com/question/30898326/answer/137735246)，在此就不一一复述了。直接说一个方便的解决方案。

<!--more-->

我在[github-page](https://help.github.com/articles/what-is-github-pages/)上放的是个人博客，主要就是国内用用，实在是英文不怎么样。使用[octopress](http://octopress.org/)搭建，其实就是静态网页。在这个前提下，那如果我们找一个支持静态网站的空间同步github，并且我们的域名支持智能DNS，那问题是不是很好解决。原来的gitcafe现在的[coding.net](https://coding.net/gitcafe.html)就能很好的解决这个问题，而且他现在还支持jekyll。正好我的域名是用的[万网](https://wanwang.aliyun.com/?spm=5176.3047821.1.3.aV9BIt)，支持智能DNS。

好了，废话说完，开始干活。

首先注册一个coding.net的账号，老路数了。接着建立一个跟账号用户名同名的仓库，例如 
`https://git.coding.net/username/username.git`

在terminal下打开原有octopress的地址（假定这个地址就是octopress，以后都是用这个地址为根目录），到octopress/_deploy目录下，添加coding.net的远程地址，并且新建分支。

```
cd octopress/_deploy
git remote add coding https://git.coding.net/username/username.git
git checkout -b coding-pages
git checkout master
git branch
>  coding-pages
>  * master
git push coding master/coding-pages
```

在coding.net的仓库里设置pages的分支

![set_coding_pages](http://7xtz1f.com2.z0.glb.clouddn.com/duel-github-cant-for-baiduspiders/11.png-shuiyinBlack)

保存了分支之后，打开`http://username.coding.me/username`就能看到我们的博客了。

接着去万网设置域名解析，将原来的github设置为海外，新加一个CNAME类型的记录为pages.coding.me设为默认。

![set_coding_pages](http://7xtz1f.com2.z0.glb.clouddn.com/duel-github-cant-for-baiduspiders/22.png-shuiyinBlack)

一般十分钟之内生效，生效后回到coding去设置`自定义域名`，输入刚才解析的二级域名，绑定完成。

告一段落，这个时候你ping自己的域名地址，同步到coding已经完成，会发现和原来的ip不一样了。

但是如果我们每次deploy之后，都要去手动再给coding提交一次，很麻烦，所以索性直接写到rakefile里去。

```ruby
multitask :push do
  puts "## Deploying branch to Github Pages "
  puts "## Pulling any updates from Github Pages "
  cd "#{deploy_dir}" do 
    Bundler.with_clean_env { system "git pull origin #{deploy_branch}" }
  end
  (Dir["#{deploy_dir}/*"]).each { |f| rm_rf(f) }
  Rake::Task[:copydot].invoke(public_dir, deploy_dir)
  puts "\n## Copying #{public_dir} to #{deploy_dir}"
  cp_r "#{public_dir}/.", deploy_dir
  cd "#{deploy_dir}" do
    system "git checkout #{deploy_branch}"
    system "git add -A"
    message = "Site updated at #{Time.now.utc}"
    puts "\n## Committing: #{message}"
    system "git commit -m \"#{message}\""
    puts "\n## Pushing generated #{deploy_dir} website"
    Bundler.with_clean_env { system "git push origin #{deploy_branch}" }
    puts "\n## Github Pages deploy complete"
    Bundler.with_clean_env { system "git push coding master:coding-pages" }
    puts "\n## coding.net Pages deploy complete"
  end
end
```

以后运行`rake deploy`时，就会自动同步到coding里去了。

最后再说说百度抓取的事，本来到这里就应该结束了，但是百度好死不死的反应慢，就跟大家说说吧。在[站长工具](http://zhanzhang.baidu.com/crawltools/)的抓取诊断测试一下，是否可以抓取成功，如果没有成功的话查看抓取状态下的抓取失败，如果网站IP和没有修改之前的一样，点击后面的报错，隔上大概半个小时再来试下，如果还是这样的话，在右下角的反馈中心反应一下，我是反应了才通过的。。。

百度链接提交的自动方式有三个：主动推送、自动推送、sitemap。
主动推送：我没怎么用，在工程底下建一个txt，将已有想提交的网页地址逐行写入，然后curl一下就好了，但是有条数限制，我是懒得搞这个了，想起来提一下，主要靠自动推送和sitemap。
自动推送：将百度提供的工具代码，放到一个合适的位置就好，然后每次打开网页的时候都会使用百度的push.js。我是放到了octopress/source/_includes/custom/footer.html里，大家可以参考。
sitemap：由于octopress已经require了 jekyll-sitemap，我们只需要保证_config.yml里的url是我们上面解析的二级域名就好，每次generate的时候会自动生成，提交之后在根目录就有一个sitemap.xml,将这个文件的地址提交给百度就好了，百度会不定时的去更新这个文件。

ok，大功告成

