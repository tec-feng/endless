title: Docker基础
author: Sunny
date: 2019-08-20 11:22:15
tags:
---

##	Docker是什么

Docker的主要目标是“Build，Ship and Run Any App,Anywhere”，也就是通过对应用组件的封装、分发、部署、运行等生命周期的管理，使用户的APP（可以是一个WEB应用或数据库应用等等）及其运行环境能够做到“一次封装，到处运行”。

![](https://raw.githubusercontent.com/tec-feng/tec-feng.github.io/hexo/image/docker.jpg)

Linux 容器技术的出现就解决了这样一个问题，而 Docker 就是在它的基础上发展过来的。将应用运行在 Docker 容器上面，而 Docker 容器在任何操作系统上都是一致的，这就实现了跨平台、跨服务器。只需要一次配置好环境，换到别的机子上就可以一键部署好，大大简化了操作。

##	Docker能做什么

- 解决了运行环境和配置问题软件容器，方便做持续集成并有助于整体发布的容器虚拟化技术。
- 更快速的应用交付和部署
- 更便捷的升级和扩缩容
- 更简单的系统运维
- 更高效的计算资源利用

## 安装要求

Docker支持以下的CentOS版本：
CentOS 7 (64-bit)
CentOS 6.5 (64-bit) 或更高的版本

目前，CentOS 仅发行版本中的内核支持 Docker。
Docker 运行在 CentOS 7 上，要求系统为64位、系统内核版本为 3.10 以上。
Docker 运行在 CentOS-6.5 或更高的版本的 CentOS 上，要求系统为64位、系统内核版本为 2.6.32-431 或者更高版本。

**查看自己的内核**

~~~shell
uname -r
或者
cat /etc/redhat-release
~~~

## Docker的安装

~~~	shell
yum install -y epel-release
yum install -y docker-io
# 安装后的配置文件 /etc/sysconfig/docker
service docker start 
docker version
~~~









