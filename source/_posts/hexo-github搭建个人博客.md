title: hexo+github搭建个人博客
author: Sunny
date: 2018-09-19 17:39:30
tags:
---
## 前言
多年前踏入技术这个圈子，从事IT行业已经多年，都说了好记性不如烂笔头，网络技术博客的沉淀也算是一种自己的名片。用过csdn来写博客，自己买过服务器，通过WordPress搭建过，也自己手写过一个博客网站，但是最后都以没坚持下来写而放弃维护。偶然在网上查某一个问题，发现了一个很舒服的博客网站，这里我特意用了一个舒服的词来形容它。突然发现博客这个东西还是得坚持下来，也算是给自己一个坚持技术道路的一个理由吧！通过一段时间的接触，聊一下hexo+github搭建出来的博客一些自己的感受：	
*	免费
*	静态页，访问速度快
*	基于github的版本管理和托管
*	通过插件发布博客方便	

<!--more-->

## 准备工作
* 准备一个github的账号
* 本地安装node、npm、git客户端	

## 操作
### github部分
#### 创建仓库
1. 新建一个名字为`用户名.github.io`的仓库，比如我的是`tec-feng.github.io`,创建成功后，那么你的博客网址就是`http://tec-feng.github.io`了。
2. 开启github pages，进入设置，下拉找到github pages，设置你github pages的source和选取一个主题，设置完毕后你就可以通过`http://tec-feng.github.io`访问了，如图所示:
![github pages](http://image.quantaoer.com/github_page.jpg)	

### hexo部分
#### hexo介绍
Hexo是一个快速、简洁且高效的博客框架。	
官方网址：[hexo.io](https://hexo.io/zh-cn/)	
文档网站：[https://hexo.io/zh-cn/docs/](https://hexo.io/zh-cn/docs/)	
#### hexo的安装
```
在电脑的某个地方新建一个hexo的文件夹，比如G:/project/hexo
$ cd G:/project/hexo
$ npm install -g hexo
$ hexo init

```
通过以上的操作hexo就会自动下载一些文件并安装好，目录结构如下图：

![目录结构](http://image.quantaoer.com/hexo_init.jpg)

通过一下命令我们可以启动hexo，执行命令后，hexo会在public文件夹商城对应的静态html文件，这些文件就是我们需要提交到github上面用来显示我们博客的。

```
$ hexo g #生成
$ hexo s #启动本地预览服务，可以浏览http://localhost:4000访问，如果端口占用，可以修改端口_config.yml里面的server：下port:端口

```
#### hexo的主题修改
hexo默认主题比较不符合自己的风格，个人比较喜欢[next](https://github.com/iissnan/hexo-theme-next)的主题,那么我们就动手来安装一下这个主题
```
$ cd G:/project/hexo
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

修改_config.yml中的theme: landscape改为theme: next，然后重新执行hexo g来重新生成。	
如果出现一些莫名其妙的问题，可以先执行hexo clean来清理一下public的内容，然后再来重新生成和发布。

#### 配置上传github
修改`_confg.yml`文件中的deploy的部分：
```
deploy:
  type: git
  repository: git@github.com:tec-feng/tec-feng.github.io.git
  branch: master
```
配置完成以后我们只需要运行`hexo d`就可以将本次所有改动的代码全部提交到github中去。

### 优雅的写博客

当然，我们可以使用hexo最原始的方法去写博客，比如用一下等等命令：
```
hexo new "hello word"
vim hello word.md
...
...
.... 一大堆过程
```
程序员都是比较懒的，有好用的插件当然就要用，那么`hexo-admin`自然就出场了。	`hexo-admin`最主要的功能是提供了在web UI下增删改查博客的功能和简单的管理功能。	
官方网址：[https://jaredforsyth.com/hexo-admin/](https://jaredforsyth.com/hexo-admin/)	
安装`hexo-admin`插件只需要运行如下几行简单的命令：
```
npm install --save hexo-admin
hexo server -d
open http://localhost:4000/admin/  #端口需要根据自己的hexo端口修改
```
以下就是`hexo-admin`的后台界面：
![hexo-admin](http://image.quantaoer.com/hexo_admin.jpg)

剩下的如何在`hexo-admin`上写自己的博客就很简单了，详情可以自己做一些摸索

#### 备份博客源码
当然我们希望我们的博客的所有源代码都由github来给我们管理，那么我们要先对github做一些准备工作。创建两个分支：master和hexo，并将hexo设置为我们的github默认分支，设置如图：	

![默认分支设置](http://image.quantaoer.com/github_set.jpg)

##### 备份
1. 将`hexo`分支clone到本地	
2. 将之前的hexo文件夹中的_config.yml，themes/，source/，scaffolds/，package.json，.gitignore复制至tec-feng.github.io文件夹	
3. 将themes/next/(我用的是NexT主题)中的.git/删除，	
4. 将这些文件提交到hexo分支中,命令如下：
```
git add .
git commit -m "init hexo"
git push origin hexo
```

##### 写博客流程
进入到本地tec-feng.github.io库的本地文件夹中	
![git project](http://image.quantaoer.com/git_project.jpg)
```
运行cmd到该目录下面
$ hexo s #运行本地预览服务
浏览器打开http://localhost:7088/admin，然后在这个里面写博客
$ hexo g #生成静态博客静态文件
$ hexo d #将静态的博客文件部署到github的master分支中
$ git add . 
$ git commit -m "新增了一篇博客"
$ git push origin hexo  #将源代码推入到github的hexo分支中保存
浏览https://tec-feng.github.io/ 就可以额看到你刚刚写的文章发布了


```
