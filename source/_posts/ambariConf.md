---
title: centos7 ambari2.6.1.5+hdp2.6.4.0 大数据集群安装部署
date: 2018-04-25
tags:
  - centos7
  - ambari
copyright: true
---
## 前言
本文是讲如何在centos7（64位） 安装ambari+hdp,如果在装有原生hadoop等集群的机器上安装，需要先将集群服务停掉，然后将不需要的环境变量注释掉即可，如果不注释掉，后面虽然可以安装成功，但是在启动某些服务的时候可能会有异常，比如最后提到的hive启动异常。本文适合系统： RedHat7、CentOS7、Oracle Linux7(都是64位)
注意:centos7中文系统有bug（python脚本中文识别问题）,需要使用英文系统。
本文仅作参考（基本每个配置博客都有局限性和坑~），推荐先参考官方文档：
[https://docs.hortonworks.com/HDPDocuments/Ambari-2.6.1.5/bk_ambari-installation/content/ch_Getting_Ready.html](https://docs.hortonworks.com/HDPDocuments/Ambari-2.6.1.5/bk_ambari-installation/content/ch_Getting_Ready.html)
以下均在root用户下执行。
## 1、满足最低系统要求
### 1.1 浏览器
建议您将浏览器(自己使用的windows既可)更新至最新的稳定版本
### 1.2 软件要求（在每台主机上）
```bash
1.2.1 yum和rpm
1.2.2 scp, curl, unzip, tar、 wget
1.2.3 OpenSSL（v1.01，build 16或更高版本）
1.2.4 python：2.7(注意如果有使用python3.x的需求，不要改变python环境变量，否则3.x会报错)
1.2.5 jdk：1.8
1.2.6 mysql：5.6（官网上写的5.6，不确定更高版本有没有问题，也可以使用其他数据库，根据自己习惯）
1.2.7 内存要求：Ambari主机应该至少有1 GB RAM，500 MB空闲，（但如果使用的话，建议内存8g以上，我自己的虚拟机内存4g搭好后跑起来会很卡，配置低的话警告也会很多）
1.2.8 检查最大打开文件描述符,推荐的最大打开文件描述符数为10000或更多
1.2.9 mysql-connector-java
```
<!-- more -->
以上软件大部分系统自带，其余可参考：[CentOS 初始环境配置](http://dongkelun.com/2018/04/05/centosInitialConf/)
## 2、环境准备（在每台主机上）

### 2.1 ssh 免密
只需master 免密到其他节点（包含自身），不需要互通，参考：[linux ssh 免密登录](http://dongkelun.com/2018/04/05/sshConf/)
### 2.2 启用NTP
``` bash
yum install -y ntp
systemctl enable ntpd
```
### 2.3 编辑主机文件
```bash
vim /etc/hosts
```
本文只是在个人虚拟机上进行安装测试，所以只选择两个节点，在公司真实环境下多个节点安装是一样的，ambari对内存要求较高，如果个人电脑配置不高的话，建议学习一下即可。
```bash
192.168.44.138 ambari.master.com
192.168.44.139 ambari.slave1.com
```
其中后面的如ambari.master.com为完全限定域名（FQDN)（通过符号“.”）,不能简单的设为master等，如果该文件里有其他映射，如上面的配置必须要在最前面（自带的localhost下面一行），否则后面安装会报错。
### 2.4 设置主机名
以ambari.master.com为例
2.4.1
```bash
hostname ambari.master.com
```
2.4.2
``` bash
vim /etc/hostname
```
```
ambari.master.com
```
两步缺一不可，通过命令验证
```bash
hostname
hostname -f
```
两个必须都为ambari.master.com才行




### 2.5 编辑网络配置文件

```bash
vim /etc/sysconfig/network
```
修改HOSTNAME属性为FQDN
```
NETWORKING=yes
HOSTNAME=ambari.master.com
```
### 2.6 禁用iptables


```bash
systemctl disable firewalld
service firewalld stop
```
### 2.7 禁用SELinux
2.7.1 临时禁用
``` bash
setenforce 0
```
2.7.2 永久禁用(重启机器)
```bash
vim /etc/sysconfig/selinux
```
将SELINUX改为disabled
```
SELINUX=disabled
```
这样服务器或虚拟机重启也没有问题。


## 3、制作本地源（仅在master）
因为ambari 和 hdp 安装文件比较大，如果在线安装的话会很慢，所以最好选择本地源。
（可以在集群可以访问的任何机器上制作本地源）
### 3.1 安装制作本地源工具
```bash
yum install yum-utils createrepo
```
### 3.2 创建一个HTTP服务器
```
yum install httpd -y
systemctl enable httpd && systemctl start httpd
```
### 3.3 为Web服务器创建目录
```
mkdir -p /var/www/html/hdp/HDP-UTILS
```
### 3.4 下载系统对应的最新版相关安装包
其中包括Ambari、HDP、HDP-UTILS,由于HDP-GPL较小只有几百k，所以没有配置为本地源。
#### 3.4.1 下载
```bash
wget http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.6.1.5/ambari-2.6.1.5-centos7.tar.gz
wget http://public-repo-1.hortonworks.com/HDP/centos7/2.x/updates/2.6.4.0/HDP-2.6.4.0-centos7-rpm.tar.gz
wget http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.22/repos/centos7/HDP-UTILS-1.1.0.22-centos7.tar.gz
```
下载地址见官方文档：[https://docs.hortonworks.com/HDPDocuments/Ambari-2.6.1.5/bk_ambari-installation/content/ch_obtaining-public-repos.html](https://docs.hortonworks.com/HDPDocuments/Ambari-2.6.1.5/bk_ambari-installation/content/ch_obtaining-public-repos.html)
#### 3.4.2 解压
```
tar -zxvf ambari-2.6.1.5-centos7.tar.gz -C /var/www/html
tar -zxvf HDP-2.6.4.0-centos7-rpm.tar.gz -C /var/www/html/hdp/
tar -zxvf HDP-UTILS-1.1.0.22-centos7.tar.gz  -C /var/www/html/hdp/HDP-UTILS/
```
#### 3.4.3 解决在浏览器访问http://ambari.master.com/hdp/HDP/centos7/2.6.4.0-91 为空白
原因：该目录下index.xml使用了 https://ajax.googleapis.com/ajax/libs/jquery/3.1.1/jquery.min.js  国内访问不了谷歌，将index.xml注释掉即可
```
cd /var/www/html/hdp/HDP/centos7/2.6.4.0-91
mv index.xml index.xml.bak
```
此时应该可以在浏览器访问下面的地址了,可以验证一下
```
http://ambari.master.com/ambari/centos7/2.6.1.5-3/
http://ambari.master.com/hdp/HDP/centos7/2.6.4.0-91
http://ambari.master.com/hdp/HDP-UTILS
```
### 3.5 配置ambari、HDP、HDP-UTILS的本地源

``` bash
cp /var/www/html/ambari/centos7/2.6.1.5-3/ambari.repo /etc/yum.repos.d/
cp /var/www/html/hdp/HDP/centos7/2.6.4.0-91/hdp.repo /etc/yum.repos.d/
```
将每个repo里的baseurl和gpgkey的地址修改为本地的
```
vim /etc/yum.repos.d/ambari.repo
```
```
#VERSION_NUMBER=2.6.1.5-3
[ambari-2.6.1.5]
name=ambari Version - ambari-2.6.1.5
baseurl=http://ambari.master.com/ambari/centos7/2.6.1.5-3
gpgcheck=1
gpgkey=http://ambari.master.com/ambari/centos7/2.6.1.5-3/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1
```
``` bash
vim /etc/yum.repos.d/hdp.repo
```
```
#VERSION_NUMBER=2.6.4.0-91
[HDP-2.6.4.0]
name=HDP Version - HDP-2.6.4.0
baseurl=http://ambari.master.com/hdp/HDP/centos7/2.6.4.0-91
gpgcheck=1
gpgkey=http://ambari.master.com/hdp/HDP/centos7/2.6.4.0-91/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1


[HDP-UTILS-1.1.0.22]
name=HDP-UTILS Version - HDP-UTILS-1.1.0.22
baseurl=http://ambari.master.com/hdp/HDP-UTILS
gpgcheck=1
gpgkey=http://ambari.master.com/hdp/HDP/centos7/2.6.4.0-91/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1

```

```
yum clean all
yum list update
yum makecache
yum repolist
```

### 3.6 （可选）如果您的环境中配置了多个存储库，请在集群中的所有节点上部署以下插件
```bash
yum install yum-plugin-priorities -y
vim /etc/yum/pluginconf.d/priorities.conf
```
```
[main]
enabled = 1
gpgcheck=0                
```
## 4、安装ambari（仅在master）
### 4.1安装ambari-server
``` bash
yum install ambari-server -y
```
### 4.2 设置mysql连接器
``` bash
ambari-server setup --jdbc-db=mysql --jdbc-driver=/usr/share/java/mysql-connector-java.jar
```
（如果使用mysql作为hive的元数据库）
### 4.3 创建相关的mysql数据库
创建ambari数据库及用户，登录root用户执行下面语句：
```bash
mysql -uroot -pRoot-123

```
```
create database ambari character set utf8 ;  
CREATE USER 'ambari'@'%'IDENTIFIED BY 'Ambari-123';
GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'%';
FLUSH PRIVILEGES;
```

如果要安装Hive,再创建Hive数据库和用户,再执行下面的语句：
```
create database hive character set utf8 ;  
CREATE USER 'hive'@'%'IDENTIFIED BY 'Hive-123';
GRANT ALL PRIVILEGES ON *.* TO 'hive'@'%';
FLUSH PRIVILEGES;
```
hive用户可以不用指定全部库的权限。

### 4.4 配置ambari-server
#### 4.4.1 setup
``` bash
ambari-server setup
```

#### 4.4.2 配置流程
以下为全部的配置过程，其中主要是自定义jdk,输入JAVA_HOME路径，自定义数据库选mysql，输入数据库用户名，密码等
```
ambari-server setup
Using python  /usr/bin/python2
Setup ambari-server
Checking SELinux...
SELinux status is 'enabled'
SELinux mode is 'permissive'
WARNING: SELinux is set to 'permissive' mode and temporarily disabled.
OK to continue [y/n] (y)? y
Customize user account for ambari-server daemon [y/n] (n)? y
Enter user account for ambari-server daemon (root):ambari
Adjusting ambari-server permissions and ownership...
Checking firewall status...
Checking JDK...
[1] Oracle JDK 1.8 + Java Cryptography Extension (JCE) Policy Files 8
[2] Oracle JDK 1.7 + Java Cryptography Extension (JCE) Policy Files 7
[3] Custom JDK
==============================================================================
Enter choice (1): 3
WARNING: JDK must be installed on all hosts and JAVA_HOME must be valid on all hosts.
WARNING: JCE Policy files are required for configuring Kerberos security. If you plan to use Kerberos,please make sure JCE Unlimited Strength Jurisdiction Policy Files are valid on all hosts.
Path to JAVA_HOME: /opt/jdk1.8.0_151
Validating JDK on Ambari Server...done.
Checking GPL software agreement...
GPL License for LZO: https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html
Enable Ambari Server to download and install GPL Licensed LZO packages [y/n] (n)? y
Completing setup...
Configuring database...
Enter advanced database configuration [y/n] (n)? y
Configuring database...
==============================================================================
Choose one of the following options:
[1] - PostgreSQL (Embedded)
[2] - Oracle
[3] - MySQL / MariaDB
[4] - PostgreSQL
[5] - Microsoft SQL Server (Tech Preview)
[6] - SQL Anywhere
[7] - BDB
==============================================================================
Enter choice (1): 3
Hostname (localhost): 
Port (3306): 
Database name (ambari): 
Username (ambari): 
Enter Database Password (bigdata): 
Re-enter password: 
Configuring ambari database...
Configuring remote database connection properties...
WARNING: Before starting Ambari Server, you must run the following DDL against the database to create the schema: /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql
Proceed with configuring remote database connection properties [y/n] (y)? y
Extracting system views...
ambari-admin-2.6.1.5.3.jar
...........
Adjusting ambari-server permissions and ownership...
Ambari Server 'setup' completed successfully.
```
#### 4.4.3将Ambari数据库脚本导入到数据库
```bash
mysql -uambari -pAmbari-123
use ambari;
source /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql
```

#### 4.4.4 启动ambari
```bash
ambari-server start
```
#### 4.4.5 启动成功，可以通过如下地址访问：

[http://ambari.master.com:8080](http://ambari.master.com:8080)

用户名，密码为admin admin

## 5、使用ambari浏览器界面安装hadoop,hive等组件
### 5.1 登录到ambari管理界面
[http://ambari.master.com:8080](http://ambari.master.com:8080)

![](http://wx4.sinaimg.cn/large/e44344dcly1fqp20p5jrgj213c0gl0tc.jpg)
### 5.2 安装hdp集群，点击Launch Install Wizard
![](http://wx4.sinaimg.cn/large/e44344dcly1fqp20q647jj216g0msdhi.jpg)
### 5.3，设置集群名称
![](http://wx3.sinaimg.cn/large/e44344dcly1fqp20qou0qj21920jcjsn.jpg)
### 5.4 配置本地源
其中HDP-GPL较小,用默认的即可
![](http://wx2.sinaimg.cn/large/e44344dcly1fqp20r1fmlj21f00sxdit.jpg)
### 5.5 设置host
其中下面的为master上ssh的私钥（~/.ssh/id_rsa）
![](http://wx3.sinaimg.cn/large/e44344dcly1fqp20rkdcrj21av0petau.jpg)

### 5.6 Host确认
如果失败或者卡住不动可根据日志解决，如果warn根据提示信息解决，知道全部为Success才可以进行下一步。
![](http://wx2.sinaimg.cn/large/e44344dcly1fqp20ryrqkj21ap0m8wg8.jpg)
![](http://wx2.sinaimg.cn/large/e44344dcly1fqp20say4uj21bc0naq51.jpg)
### 5.7 选择要安装的服务
![](http://wx3.sinaimg.cn/large/e44344dcly1fqp20t3g81j21bs0svgou.jpg)
![](http://wx1.sinaimg.cn/large/e44344dcly1fqp20tfk42j21am0otjtt.jpg)

如果有依赖其他组件选择ok即可，如安装hive依赖tez，pig等
![](http://wx1.sinaimg.cn/large/e44344dcly1fqp20tyfu7j219s0n0di8.jpg)

### 5.8 设置各个服务Master
![](http://wx2.sinaimg.cn/large/e44344dcly1fqp20ucf6yj21dc0scade.jpg)
### 5.9 设置Slaves 和 Clients
![](http://wx4.sinaimg.cn/large/e44344dcly1fqp20uyqtvj21de0natar.jpg)
### 5.10 自定义配置
其中红色的必须要改，大致是设置路径，密码等，如hive要设置hive元数据的数据库信息，我用的master上的mysql
![](http://wx4.sinaimg.cn/large/e44344dcly1fqp20vd6quj21fk0sgmzs.jpg)
测试一下连接
![](http://wx1.sinaimg.cn/large/e44344dcly1fqp20vqfkwj21fk0s2q60.jpg)

没有了红色的即可进行下一步，如遇到warn，可根据提示信息进行修改配置，也可以忽略警告，等装完以后再改。
### 5.11 review前面的配置
![](http://wx3.sinaimg.cn/large/e44344dcly1fqp20w8jhoj21dp0okdi5.jpg)
### 5.12 安装、启动、测试
这里因为个人电脑配置较低，浏览器有点卡，进度条没有显示出来。
![](http://wx3.sinaimg.cn/large/e44344dcly1fqp20wm48sj21ea0mcwgc.jpg)
### 5.13 安装完成
若最后出现警告，可以装完重启所有服务，再检查看看有没有问题，如有警告或启动失败，可根据日志排查原因，一开始安装的的组件较多的话，出现警告的可能性会大一些，所以可以先装几个必要的组件，之后一个一个组件装。
![](http://wx2.sinaimg.cn/large/e44344dcly1fqp20x14rbj21eu0na769.jpg)
### 5.14 概要
![](http://wx1.sinaimg.cn/large/e44344dcly1fqp20xhixnj21am0i5401.jpg)

### 5.15 hive启动异常
这次安装重启之后发现hive等服务启动不成功，我就把hive等卸载然后重装，本来以为是开始是hive没安装成功，但是重装后hive还是启动不成功，看了一下日志，发现是之前手动安装的原生的hive的环境变量没有注释掉，注释掉，重启ambari之后，再启动所有服务，就成功了（再在hive shell 里建表、插入数据、查询验证一下），所以如果在已经安装好的大数据集群上安装ambari，最好先把之前配的环境变量注释掉。
### 5.16 启动成功
![](http://wx4.sinaimg.cn/large/e44344dcly1fqpc4ytg3ij20vm08haag.jpg)
![](http://wx2.sinaimg.cn/large/e44344dcly1fqpujavdpmj21dy0plq6b.jpg)
