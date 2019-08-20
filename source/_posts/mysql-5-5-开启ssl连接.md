title: mysql 5.5 开启ssl连接
author: Sunny
tags: mysql
date: 2019-08-20 11:26:38
---

​	最近客户要求项目对于数据库的连接使用SSL，这样有利于数据安全的传输，以下从网络整理了一些相关文章并实际操作使用，记录备查。

### 1、查看MySQL是否已经开启了SSL

~~~	sql
首先进入mysql的命令模式：
show variables like '%have%ssl%';
+---------------+----------+
| Variable_name | Value    |
+---------------+----------+
| have_openssl  | DISABLED |
| have_ssl      | DISABLED |
+---------------+----------+
从结果上面来看是没有开启ssl支持的，都是DISABLED的状态。
~~~

### 2、通过openssl制作生成SSL证书

#### 2.1	生成一个CA私钥

```shell

administrator@iZj6c41z3iskfi7x3gumhhZ:~$ openssl genrsa 2048 >ca-key.pem
Generating RSA private key, 2048 bit long modulus
......................................................................................................................................+++
................................................................................................................................................................................................................................+++
e is 65537 (0x10001)

```

<!--more-->

#### 2.2	通过CA私钥成数字证书

~~~ shell
administrator@iZj6c41z3iskfi7x3gumhhZ:~$ openssl req -new -x509 -nodes -days 3600 -key ca-key.pem -out ca.pem
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:
~~~

####	2.3	创建MySQL服务器私钥和请求证书

~~~ shell
administrator@iZj6c41z3iskfi7x3gumhhZ:~$ openssl req -newkey rsa:2048 -days 3600 -nodes -keyout server-key.pem -out server-req.pem
Generating a 2048 bit RSA private key
...........................................................................+++
................................................+++
writing new private key to 'server-key.pem'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
~~~

#### 2.4	将生成的私钥转换为 RSA 私钥文件格式

~~~	shell
administrator@iZj6c41z3iskfi7x3gumhhZ:~$ openssl rsa -in server-key.pem -out server-key.pem
writing RSA key
~~~

####	2.5	用CA 证书来生成一个服务器端的数字证书

~~~ shell
administrator@iZj6c41z3iskfi7x3gumhhZ:~$ openssl x509 -req -in server-req.pem -days 3600 -CA ca.pem -CAkey ca-key.pem -set_serial 01 -out server-cert.pem
Signature ok
subject=/C=AU/ST=Some-State/O=Internet Widgits Pty Ltd
Getting CA Private Key
~~~

####	2.6	创建客户端的 RSA 私钥和数字证书

~~~ shell
administrator@iZj6c41z3iskfi7x3gumhhZ:~$ openssl req -newkey rsa:2048 -days 3600 -nodes -keyout client-key.pem -out client-req.pem
Generating a 2048 bit RSA private key
............+++
.......+++
writing new private key to 'client-key.pem'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
~~~

####	2.7	将生成的私钥转换为 RSA 私钥文件格式

~~~ shell
administrator@iZj6c41z3iskfi7x3gumhhZ:~$ openssl rsa -in client-key.pem -out client-key.pem
writing RSA key
~~~

####	2.8	用CA 证书来生成一个客户端的数字证书

~~~	shell
administrator@iZj6c41z3iskfi7x3gumhhZ:~$ openssl x509 -req -in client-req.pem -days 3600 -CA ca.pem -CAkey ca-key.pem -set_serial 01 -out client-cert.pem
Signature ok
subject=/C=AU/ST=Some-State/O=Internet Widgits Pty Ltd
Getting CA Private Key
~~~

####  2.9	查看所有生成的SSL文件

~~~ shell
administrator@iZj6c41z3iskfi7x3gumhhZ:~$ ll *.pem
-rw-rw-r-- 1 administrator administrator 1679 Aug 20 10:42 ca-key.pem
-rw-rw-r-- 1 administrator administrator 1229 Aug 20 10:44 ca.pem
-rw-rw-r-- 1 administrator administrator 1099 Aug 20 10:51 client-cert.pem
-rw-rw-r-- 1 administrator administrator 1675 Aug 20 10:50 client-key.pem
-rw-rw-r-- 1 administrator administrator  956 Aug 20 10:49 client-req.pem
-rw-rw-r-- 1 administrator administrator 1099 Aug 20 10:48 server-cert.pem
-rw-rw-r-- 1 administrator administrator 1675 Aug 20 10:47 server-key.pem
-rw-rw-r-- 1 administrator administrator  956 Aug 20 10:47 server-req.pem
~~~

### 3、MySQL 配置启动SSL

#### 3.1	复制 CA 证书和服务端SSL文件至MySQL 数据目录

~~~ shell
root@iZj6c41z3iskfi7x3gumhhZ:/var/lib/mysql# cp /home/administrator/ca.pem /home/administrator/server-*.pem . -v
‘/home/administrator/ca.pem’ -> ‘./ca.pem’
‘/home/administrator/server-cert.pem’ -> ‘./server-cert.pem’
‘/home/administrator/server-key.pem’ -> ‘./server-key.pem’
‘/home/administrator/server-req.pem’ -> ‘./server-req.pem’
~~~

#### 3.2	修改 MySQL 数据目录的CA 证书和服务端 SSL 文件所属用户与组

~~~	shell
root@iZj6c41z3iskfi7x3gumhhZ:/var/lib/mysql# chown -v mysql.mysql /var/lib/mysql/{ca,server*}.pem
changed ownership of ‘/var/lib/mysql/ca.pem’ from root:root to mysql:mysql
changed ownership of ‘/var/lib/mysql/server-cert.pem’ from root:root to mysql:mysql
changed ownership of ‘/var/lib/mysql/server-key.pem’ from root:root to mysql:mysql
changed ownership of ‘/var/lib/mysql/server-req.pem’ from root:root to mysql:mysql
~~~

####	3.3	配置 MySQL 服务的配置文件 [/etc/my.cnf]

~~~ shell
[mysqld]
ssl-ca=/var/lib/mysql/ca.pem
ssl-cert=/var/lib/mysql/server-cert.pem
ssl-key=/var/lib/mysql/server-key.pem
~~~

####	3.4	重启MySQL服务

~~~ shell
root@iZj6c41z3iskfi7x3gumhhZ:/var/lib/mysql# service mysql restart
mysql stop/waiting
mysql start/running, process 22554
~~~

####	3.5	登陆查看SSL开启状态

~~~ mysql
mysql> show variables like '%have%ssl%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| have_openssl  | YES   |
| have_ssl      | YES   |
+---------------+-------+
2 rows in set (0.00 sec)
 have_openssl 与 have_ssl 值都为YES表示ssl开启成功
~~~

### 4、SSL链接测试

#### 4.1	创建用户指定SSL链接

~~~	mysql
mysql> grant all on *.* to 'testssl'@'%' identified by '123456' require SSL;
Query OK, 0 rows affected (0.00 sec)
~~~

####	4.2	通过密码连接测试

~~~	shell
root@iZj6c41z3iskfi7x3gumhhZ:/var/lib/mysql# mysql -utestssl -p
Enter password: 
ERROR 1045 (28000): Access denied for user 'testssl'@'localhost' (using password: YES)
使用正确的账号和密码登录，但是提示没有权限，代表ssl生效
~~~

####	4.3	通过客户端秘钥和证书SSL加上密码连接测试

~~~	shell
root@iZj6c41z3iskfi7x3gumhhZ:/var/lib/mysql# mysql -utestssl -p --ssl-cert=/home/administrator/client-cert.pem --ssl-key=/home/administrator/client-key.pem
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 39
Server version: 5.5.57-0ubuntu0.14.04.1 (Ubuntu)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
正常的连接到mysql，代表ssl生效
~~~

### 5、总结

####	5.1	SLL介绍

**SSL**（Secure Socket Layer：安全套接字层）利用数据加密、身份验证和消息完整性验证机制，为基于TCP等可靠连接的应用层协议提供安全性保证。

SSL协议提供的功能主要有：

​           1、 数据传输的机密性：利用对称密钥算法对传输的数据进行加密。
​           2.、身份验证机制：基于证书利用数字签名方法对服务器和客户端进行身份验证，其中客户端的身份验证是可选的。
​           3、 消息完整性验证：消息传输过程中使用MAC算法来检验消息的完整性。

如果用户的传输不是通过SSL的方式，那么其在网络中数据都是以明文进行传输的，而这给别有用心的人带来了可乘之机。所以，现在很多大型网站都开启了SSL功能。同样地，在我们数据库方面，如果客户端连接服务器获取数据不是使用SSL连接，那么在传输过程中，数据就有可能被窃取。

####	5.2	默认机制

​	目前MySQL5.7或更高版本已经默认开启SSL连接，如果强制用户使用SSL连接，应用程序需要指明配置相关SSL参数

####	5.3	性能说明

 	对于非常敏感核心的数据，或者QPS本来就不高的核心数据，可以采用SSL方式保障数据安全性；

​	对于采用短链接、要求高性能的应用，或者不产生核心敏感数据的应用，性能和可用性才是首要，建议不要采用SSL方式；