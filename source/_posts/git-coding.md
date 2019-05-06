---
title: 将hexo博客同时托管到github和coding
date: 2016-05-29 13:57:09
tags: git
categories: git
---
# 将hexo博客同时托管到github和coding

发现百度的sitemap.xml抓取不到github上面的，所以今天又弄了一个coding。

在个人设置里面添加好公钥后，本地git登录

然后在站点配置文件`_config.yml`里面配置如下：

<!--more-->
```bash
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  #repository: https://github.com/Rayuu/Rayuu.github.io.git
  #branch: master
  repo:
    github: git@github.com:Rayuu/Rayuu.github.io.git,master
    coding: git@git.coding.net:chay/chay.git,coding-pages

```

接下来`git bash`到站点目录

最后

```bash
hexo d -g

```

没有报错说明完成，添加密钥后也不用每次输入密码了。

这样就完美了哦！

# —————我是分割线—————

今天重装了系统，感觉整个人都好多了。哈哈哈（#滑稽）
重新部署hexo

先安装`hexo`，
再安装`nodejs,git`
接着安装` npm install -g cnpm --registry=https://registry.npm.taobao.org`
```bash
cnpm install -g hexo
```
找个文件夹打开 git bash here
```bash
hexo init
cnpm install hexo-deployer-git --save
cnpm install hexo-generator-sitemap --save
cnpm install hexo-generator-baidu-sitemap --save
cnpm install hexo-generator-search --save
```

把备份的公钥文件夹放到`C:\Users\用户名\.ssh`下面
```bash
ssh -T git@git.coding.net
ssh -T git@github.com
```
让服务器使用`ssh`协议。
成功的话就一切正常。
```bash
hexo d -g
```

上传！！！搞定。。。
