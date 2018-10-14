-----
title: 安装 LAMP 环境（CentOS7+PHP7.1.5+MySQL5.7）
id: post-20180827233237
tags:
  - PHP
  - MySQL
  - Linux
categories:
  - PHP
date: 2018-08-27 23:32:37
-----
> “工欲善其事必先利其器”，身为一个开发人员，快速的搭建一个能使用的环境是必备的技能之一。我们来记录一个简单的LA(N)MP环境的搭建的过程，也方便后面的查阅。

### 步骤一：安装 Apache & Nginx

升级一下yum源（不是必须的），升级会花点时间，需要选择的地方都选择都输入“y”即可

```shell
yum update
```

#### 安装 Apache

```shell
yum list |grep httpd // 查看有哪些Apache可以安装
yum install -y httpd // 安装Apache指令
http -v // 安装完成后查看Apache版本指令
// 显示内容如下
Server version: Apache/2.4.6 (CentOS)
Server built: Apr 12 2017 21:03:28
```

在本机的浏览器的地址栏中输入虚拟主机的 IP 地址，但是浏览器中没有显示有效的内容，这种情况基本是Centos自带的防火墙开启造成的

```shell
// 关闭防火墙
[root@localhost /]# systemctl systemctl stop firewalld // 关闭防火墙
[root@localhost /]# systemctl systemctl status firewalld // 查看防火墙状态
[root@localhost /]# systemctl systemctl restart httpd // 重启Apache
```

关闭防火墙后再次刷新浏览器，基本就可以正常访问了，如果不想关闭防火墙,那我们可以在防火墙上允许访问80端口（这是 Apache 默认的端口）

```shell
防火墙上开启80端口
[root@localhost /]# firewall-cmd --permanent --zone=public --add-port=80/tcp // 开启80端口并且永久生效 success
[root@localhost /]# firewall-cmd --reload // 重新载入防火墙配置 success
[root@localhost /]# systemctl status firewalld
firewalld.service - firewalld - dynamic firewall daemon
Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
Active: active (running) since Sun 2017-06-11 13:08:10 CST; 8s ago // 防火墙的状体是开启的
```

刷新浏览器依旧能访问到 Apache 的默认页面

#### 安装 Ngixn

```shell
[root@localhost /]# yum search nginx // 搜索没有nginx这个安装包 很遗憾的是没有
[root@localhost /]# yum install nginx // 尝试安装
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
* base: mirrors.tuna.tsinghua.edu.cn
* extras: mirrors.tuna.tsinghua.edu.cn
* updates: mirrors.tuna.tsinghua.edu.cn
No package nginx available. // 还是没有nginx的安装包
// 将nginx放到yum repro库中
[root@localhost /]# rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
[root@localhost /]# yum search nginx // 再次搜索nginx安装包
========================= N/S matched: nginx =========================
nginx-debug.x86_64 : debug version of nginx
nginx-debuginfo.x86_64 : Debug information for package nginx
nginx-module-geoip.x86_64 : nginx GeoIP dynamic modules
nginx-module-geoip-debuginfo.x86_64 : Debug information for package nginx-module-geoip
nginx-module-image-filter.x86_64 : nginx image filter dynamic module
nginx-module-image-filter-debuginfo.x86_64 : Debug information for package nginx-module-image-filter
nginx-module-njs.x86_64 : nginx nginScript dynamic modules
nginx-module-njs-debuginfo.x86_64 : Debug information for package nginx-module-njs
nginx-module-perl.x86_64 : nginx Perl dynamic module
nginx-module-perl-debuginfo.x86_64 : Debug information for package nginx-module-perl
nginx-module-xslt.x86_64 : nginx xslt dynamic module
nginx-module-xslt-debuginfo.x86_64 : Debug information for package nginx-module-xslt
nginx-nr-agent.noarch : New Relic agent for NGINX and NGINX Plus
nginx-release-centos.noarch : nginx repo configuration and pgp public keys
pcp-pmda-nginx.x86_64 : Performance Co-Pilot (PCP) metrics for the Nginx Webserver
nginx.x86_64 : High performance web server // 是存在nginx安装包的
[root@localhost /]# yum install nginx // 安装nginx
[root@localhost /]# nginx -v // 查看nginx版本
nginx version: nginx/1.12.0
[root@localhost /]# systemctl restart nginx // 重启nginx
```

如果同时安装 Nginx 和 Apache，并希望通过 80 端口访问的是 Apach 服务器，通过 8080 端口访问的是 Nginx 服务器

```shell
[root@localhost /]# firewall-cmd --permanent --zone=public --add-port=8080/tcp success
[root@localhost /]# firewall-cmd --reload success
[root@localhost /]# firewall-cmd --zone=public --list-ports // 查看所有打开的端口 80/tcp 8080/tcp // 80,8080已经打开
[root@localhost /]# cd /etc/nginx/conf.d
[root@localhost conf.d]# vim default.conf // 将其中的listen 80;修改为 listen 8080;
[root@localhost conf.d]# systemctl restart nginx // 重启nginx
```

通过浏览器访问 xxx.xxx.xxx.xxx:8080 即可访问

### 安装 MySQL

> CetOS7 已使用了 MariaDB 替代了默认的 MySQL，所以我们想使用 MySQL 还得多做点事情

```shell
[root@localhost /]# yum install wget // 安装wget
[root@localhost /]# wget http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm // 下载mysql5.7的rpm文件
[root@localhost /]# yum localinstall mysql57-community-release-el7-8.noarch.rpm // 安装rmp文件
[root@localhost /]#yum install mysql-community-server // 安装MySQL数据库 约190M左右
[root@localhost /]# systemctl start mysqld // 启动数据库
[root@localhost /]# grep 'temporary password' /var/log/mysqld.log // 查看MySQL数据的原始密码文件
2017-06-11T06:18:16.057811Z 1 [Note] A temporary password is generated for root@localhost: uvrpXRuul6/h
[root@localhost /]# mysql -u root -p
// 进入数据库系统
mysql> set password for 'root'@'localhost'=password('Aa@123456'); // 重新为数据库设置密码
Query OK, 0 rows affected, 1 warning (0.01 sec)
// 将MySQL设置为开机启动项
[root@localhost /]# systemctl enable mysqld
[root@localhost /]# systemctl daemon-reload
// 设置数据的默认字符集(非必须的)
[root@localhost ~]# vim /etc/my.cnf
// 将下面两行配置代码加入到my.cnf最后面
character_set_server=utf8
init_connect='SET NAMES utf8'
[root@localhost ~]# systemctl restart mysqld // 重启MySQL
//进入MySQL数据库
mysql> select version();
+-----------+
| version() |
+-----------+
| 5.7.18    |
+-----------+
1 row in set (0.00 sec)
mysql> show variables like '%character%';
+--------------------------+----------------------------+
|       Variable_name      |            Value           |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.01 sec)
```

### 安装 PHP7

```shell
// 安装依赖文件
yum install gcc-c++ libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel
// 安装php组新版本rpm
[root@localhost ~]# rpm -Uvh https://mirror.webtatic.com/yum/el7/epel-release.rpm
Retrieving https://mirror.webtatic.com/yum/el7/epel-release.rpm
warning: /var/tmp/rpm-tmp.F1aZTS: Header V4 RSA/SHA1 Signature, key ID 62e74ca5: NOKEY
Preparing...                           ################################# [100%]
Updating / installing...
1:epel-release-7-5                     ################################# [100%]
[root@localhost ~]# rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
Retrieving https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
warning: /var/tmp/rpm-tmp.rnxYY3: Header V4 RSA/SHA1 Signature, key ID 62e74ca5: NOKEY
Preparing...                           ################################# [100%]
Updating / installing...
1:webtatic-release-7-3                 ################################# [100%]
[root@localhost ~]# yum search php71 // 查看有哪些php最新版的安装包文件
[root@localhost ~]# yum install mod_php71w php71w-mysqlnd php71w-cli php71w-fpm // 安装php7最新版
[root@localhost ~]# php -v // 查看php版本
PHP 7.1.5 (cli) (built: May 12 2017 21:54:58) ( NTS )
Copyright (c) 1997-2017 The PHP Group
Zend Engine v3.1.0, Copyright (c) 1998-2017 Zend Technologies
[root@localhost ~]# systemctl restart httpd // 重启Apache
[root@localhost ~]# vim /var/www/html/index.php
<?php phpinfo(); ?> // 编辑PHP文件内容为这个
```

最后刷新浏览器，就可以看到结果了。

### 备注：常见问题以及解决方法

**问题：MySQL5.7 安装之后，修改简单密码报错，无法设置数据库认为过于简单的密码**

有时候为了测试，密码并不想设置的太复杂，比如“123456”，当执行修改语句 MySQL 会有如下报错

```shell
mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('123456');
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```

原因是 MySQL5.6.6 版本之后增加了密码强度验证插件 validate_password，相关参数设置的较为严格。使用了该插件会检查设置的密码是否符合当前设置的强度规则，若不满足则拒绝设置。

影响的语句和函数有：``create user``，``grant``，``set password``，``password()``，``old password``。

解决方法如下：

1、 查看mysql全局参数配置

该问题其实与 MySQL 的 validate_password_policy 的值有关。

查看一下 MySQL 密码相关的几个全局参数：

```shell
mysql> select @@validate_password_policy;
+----------------------------+
| @@validate_password_policy |
+----------------------------+
| MEDIUM                     |
+----------------------------+
1 row in set (0.00 sec)


mysql> SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password_dictionary_file    |        |
| validate_password_length             | 8      |
| validate_password_mixed_case_count   | 1      |
| validate_password_number_count       | 1      |
| validate_password_policy             | MEDIUM |
| validate_password_special_char_count | 1      |
+--------------------------------------+--------+
6 rows in set (0.08 sec)
```

2、参数解释

validate_password_dictionary_file
插件用于验证密码强度的字典文件路径。

validate_password_length
密码最小长度，参数默认为8，它有最小值的限制，最小值为：validate_password_number_count + validate_password_special_char_count + (2 * validate_password_mixed_case_count)

validate_password_mixed_case_count
密码至少要包含的小写字母个数和大写字母个数。

validate_password_number_count
密码至少要包含的数字个数。

validate_password_policy
密码强度检查等级，0/LOW、1/MEDIUM、2/STRONG。有以下取值：

| Policy | Tests Performed |
| :------------ | :------------ |
| 0 or LOW|Length |
| 1 or MEDIUM | Length; numeric, lowercase/uppercase, and special characters |
| 2 or STRONG | Length; numeric, lowercase/uppercase, and special characters; dictionary file |

默认是1，即MEDIUM，所以刚开始设置的密码必须符合长度，且必须含有数字，小写或大写字母，特殊字符。

validate_password_special_char_count
密码至少要包含的特殊字符数。

3、修改 MySQL 参数配置

```shell
mysql> set global validate_password_policy=0;
Query OK, 0 rows affected (0.05 sec)

mysql> set global validate_password_mixed_case_count=0;
Query OK, 0 rows affected (0.00 sec)

mysql> set global validate_password_number_count=3;
Query OK, 0 rows affected (0.00 sec)

mysql> set global validate_password_special_char_count=0;
Query OK, 0 rows affected (0.00 sec)

mysql> set global validate_password_length=3;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| validate_password_dictionary_file    |       |
| validate_password_length             | 3     |
| validate_password_mixed_case_count   | 0     |
| validate_password_number_count       | 3     |
| validate_password_policy             | LOW   |
| validate_password_special_char_count | 0     |
+--------------------------------------+-------+
6 rows in set (0.00 sec)
```

4、修改简单密码

```shell
mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('123');
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

OK！成功。

> 本文参考：
> https://www.jianshu.com/p/ed2d1820bd3e
> https://blog.csdn.net/kuluzs/article/details/51924374
