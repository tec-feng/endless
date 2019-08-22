title: Ubuntu14.04 安装Docker
author: Sunny
date: 2019-08-22 14:49:51
tags:
---
最近在一台Ubuntu14.0.4的机器上安装docker，试了很多帖子上面的安装方法，最终都在apt-get update的时候，通不过，导致安装不成功，记录备查。
#### 安装
##### 安装必要的一些系统工具
~~~ shell
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
~~~
#####	安装GPG证书	
~~~ shell
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
~~~

<!--more-->
#####	写入软件源信息
~~~ shell
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
~~~
#####	更新并安装 Docker-CE
~~~ shell
sudo apt-get -y update
sudo apt-get -y install docker-ce
~~~
#####	安装指定版本的Docker-CE:
######	查找Docker-CE的版本:
~~~ shell 
# apt-cache madison docker-ce
#   docker-ce | 17.03.1~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
#   docker-ce | 17.03.0~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
~~~
######	安装指定版本的Docker-CE: 
~~~ shell
# (VERSION 例如上面的 17.03.1~ce-0~ubuntu-xenial)
# sudo apt-get -y install docker-ce=[VERSION]

~~~
####	安装校验
~~~ shell
root@iZj6c41z3iskfi7x3gumhhZ:/etc/apt# docker version
Client:
 Version:           18.06.3-ce
 API version:       1.38
 Go version:        go1.10.3
 Git commit:        d7080c1
 Built:             Wed Feb 20 02:27:13 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server:
 Engine:
  Version:          18.06.3-ce
  API version:      1.38 (minimum version 1.12)
  Go version:       go1.10.3
  Git commit:       d7080c1
  Built:            Wed Feb 20 02:25:38 2019
  OS/Arch:          linux/amd64
  Experimental:     false

~~~