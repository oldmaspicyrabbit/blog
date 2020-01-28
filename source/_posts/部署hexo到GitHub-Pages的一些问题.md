---
title: 部署hexo到GitHub Pages的一些问题
date: 2020-01-28 22:31:44
comments: true
tags: 
- hexo
- Github pages
categories: 
- hexo 
keywords: 
- github
- hexo
excerpt: 主要记录在借助hexo和GitHub Pages的部署过程中遇到的问题以及解决办法
---

## Overview

第一次使用hexo搭建借助github pages搭建博客网站，如何搭建网上的教程有很多。
简要的描述，主要有三个部分：
*  安装node, hexo, gitbash
*  创建hexo的站点文件，主要是hexo init 
*  将hexo的站点文件host到 github pages  
  第一部分安装相关软件的，我暂时没碰到什么问题，所以直接略过了，下面会介绍后面两个部分中遇到的问题以及我是如何解决的。

## 创建hexo的站点文件--问题以及解决方法
在第二部分，创建hexo站点文件时，执行hexo init出现了几个问题，在网上没有怎么碰到：
### 问题1：git clone 时 error: RPC failed; curl 56 OpenSSL SSL_read: SSL_ERROR_SYSCALL, errno 10054
这部分具体记录已经丢失了，但是出现的问题日志，主要是这一条，导致每次git clone都失败，hexo init直接失败。
WARN git clone failed. Copying data instead  
WARN Failed to install dependencies. Please run 'npm install' manually!  
**解决的办法**：  
在网上搜索了下，虽然给出的直接解决办法是 git config http.sslVerify "false"
但是似乎总是会在download dependencies的时候卡住，还是会失败。
所以后面继续搜索，找到了hexo的github上的issue的[类似问题-hexo init,but no file created](https://github.com/hexojs/hexo/issues/2646)的回复：
Could you please try to clone the repo https://github.com/hexojs/hexo-starter , then run npm install inside the folder?
验证过后，发现问题似乎解决了，站点文件都有，唯一的问题是gitclone的文件夹下会有.git文件，而我们是需要import到自己的github的repository中的，而且如果直接git clone的话，文件夹名仍然是hexo-starter，所以需要用git clone https://github.com/hexojs/hexo-starter  yourFolderName, 文件夹名会是youFolderName。

### 问题2: hexo generate时提示WARN No layout: index.html #1037且hexo s后无法正常显示，而是显示的是空白页面。  
**解决的办法**：在github上找到了类似的[问题-WARN No layout: index.html](https://github.com/hexojs/hexo/issues/1037),解决办法也很简单粗暴：  
Under the folder, enter: git clone https://github.com/hexojs/hexo-theme-landscape.git themes/landscape  
那么很显然，这个问题是因为缺少themes文件造成的html无法正常生成，我在后面用Trivis CI进行build时检查build完后的deploy文件也验证发现了类似的问题，generate出来的index.html文件都是0字节，显示无法正常显示。

## 使用Trivis CI进行build后的branch来host站点--问题以及解决方法  
这里需要首先弄清两个问题，github pages的种类有几种？hexo和Trivis CI和github的workflow到底是怎么样的？

### 问题1：hexo的guidelines上的说明In your GitHub repo’s setting, navigate to “GitHub Pages” section and change Source to gh-pages branch.根本没法实现，因为Source是灰的  
我被这个问题坑了很久，因为前面提到的themes的files缺失导致html文件时空的，我一直以为是这个原因导致的，后来才发现这根本是两个问题，前面的是build就有的问题。而这里这个是deployment的问题，是两回事。
**解决的办法**：在hexo官网上上找到了[解决办法-Project page](https://hexo.io/docs/github-pages)  
If you prefer to have a project page on GitHub:  
	1. Navigate to your repo on GitHub. Go to the Settings tab. Change the Repository name so your blog is available at username.github.io/repository, repository can be any name, like blog or hexo.  
	2. Edit your _config.yml, change the root: value to the /<repository>/ (must starts and ends with a slash, without the brackets).Commit and push.    
所以 My Steps:  
1.Change the repository name into "blog", then the repository change to oldmaspicyrabbit/blog  
2.Change the source from "master" to "gh_pages"  
3.github pages will hint: Your site is published at https://oldmaspicyrabbit.github.io/blog/  
4.open _config.yml file, change "root: /" to "root: /blog/"  

