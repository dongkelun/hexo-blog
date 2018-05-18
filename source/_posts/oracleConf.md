---
title: centos7 安装oracle11
date: 2018-05-05
tags:
  - centos7
  - oracle
copyright: true
---
## 前言
由于需要学习配置oracle goldengate(ogg),奈何没有oracle环境，所以想自己装一个oracle，搜了一下相关文档，跟着安装了一下，发现oracle安装比mysql安装麻烦多了，而且出现了很多博客上没有提到的错误，所以特此记录一下~
## 1、下载

下载地址：[http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html)，我下载的是Oracle Database 11g Release 2
(11.2.0.1.0) Linux x86-64，注意File1和File2都要下载

## 2、为host添加映射
我的虚拟机之前已经配好
```bash
192.168.44.128 ambari.master.com
```
## 3、安装依赖
### 3.1 先安装pdksh 
centos7没有相关安装包可用，可下载pdksh的rpm包
```bash
wget  http://vault.centos.org/5.11/os/x86_64/CentOS/pdksh-5.2.14-37.el5_8.1.x86_64.rpm
rpm -ivh pdksh-5.2.14-37.el5_8.1.x86_64.rpm
```
### 3.2 安装其他依赖
```
yum -y install binutils compat-libstdc++-33 elfutils-libelf elfutils-libelf-devel expat gcc gcc-c++ glibc glibc-common glibc-devel glibc-headers libaio libaio-devel libgcc libstdc++ libstdc++-devel make pdksh sysstat unixODBC unixODBC-devel
```
### 3.3 检查所有依赖是否安装完整
```
rpm -q binutils compat-libstdc++-33 elfutils-libelf elfutils-libelf-devel expat gcc gcc-c++ glibc glibc-common glibc-devel glibc-headers libaio libaio-devel libgcc libstdc++ libstdc++-devel make pdksh sysstat unixODBC unixODBC-devel | grep "not installed"
```
其中中文系统"not installed" 可能需要替换成中文相关的
<!-- more -->
## 4、添加oracle用户组和用户
```bash
groupadd oinstall
groupadd dba
groupadd asmadmin
groupadd asmdba
useradd -g oinstall -G dba,asmdba oracle -d /home/oracle
```
查看oracle用户
```
id oracle
```
为oracle 用户设置密码
```
passwd oracle
```
## 5、优化系统内核
```bash
vim /etc/sysctl.conf
```
```
fs.aio-max-nr=1048576
fs.file-max=6815744
kernel.shmall=2097152
kernel.shmmni=4096
kernel.shmmax = 2147483648
kernel.sem=250 32000 100 128
net.ipv4.ip_local_port_range=9000 65500
net.core.rmem_default=262144
net.core.rmem_max=4194304
net.core.wmem_default=262144
net.core.wmem_max=1048586
```
其中kernel.shmmax为内存的一半，比如内存为4G，则kernel.shmmax=2*1024*1024*1024=2147483648
使参数生效
```bash
sysctl -p
```
## 6、限制oracle用户的shell权限
```bash
vim /etc/security/limits.conf
```
```
oracle              soft    nproc   2047
oracle              hard    nproc   16384
oracle              soft    nofile  1024
oracle              hard    nofile  65536
```
```bash
vim /etc/pam.d/login
```
```
session  required   /lib64/security/pam_limits.so
session  required   pam_limits.so
```
```bash
vim /etc/profile
```
```
if [ $USER = "oracle" ]; then
if [ $SHELL = "/bin/ksh" ]; then
ulimit -p 16384
ulimit -n 65536
else
ulimit -u 16384 -n 65536
fi
fi
```
## 7、创建oracle相关目录
```bash
mkdir -p /db/app/oracle/product/11.2.0
mkdir /db/app/oracle/oradata
mkdir /db/app/oracle/inventory
mkdir /db/app/oracle/fast_recovery_area
chown -R oracle:oinstall /db/app/oracle
chmod -R 775 /db/app/oracle
mkdir -p /u01/app/oracle/inventory
chown -R oracle:oinstall /u01/app/oracle/inventory
```
## 8、配置oracle用户环境变量
```bash
su oracle
vim .bash_profile
```
```
umask 022
export ORACLE_HOSTNAME=ambari.master.com
export ORACLE_BASE=/db/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/
export ORACLE_SID=ORCL
export PATH=.:$ORACLE_HOME/bin:$ORACLE_HOME/OPatch:$ORACLE_HOME/jdk/bin:$PATH
export LC_ALL="en_US"
export LANG="en_US"
export NLS_LANG="AMERICAN_AMERICA.ZHS16GBK"
export NLS_DATE_FORMAT="YYYY-MM-DD HH24:MI:SS"
```
## 9、解压安装包
如果安装包在root用户下，现切换到root用户
```bash
su
```
```bash
unzip linux.x64_11gR2_database_1of2.zip -d /db
unzip linux.x64_11gR2_database_2of2.zip -d /db
```
然后执行
```bash
mkdir /db/etc/
cp /db/database/response/* /db/etc/
vim /db/etc/db_install.rsp
```
```
oracle.install.option=INSTALL_DB_SWONLY
DECLINE_SECURITY_UPDATES=true
UNIX_GROUP_NAME=oinstall
INVENTORY_LOCATION=/u01/app/oracle/inventory
SELECTED_LANGUAGES=en,zh_CN
ORACLE_HOSTNAME=ambari.master.com
ORACLE_HOME=/db/app/oracle/product/11.2.0
ORACLE_BASE=/db/app/oracle
oracle.install.db.InstallEdition=EE
oracle.install.db.isCustomInstall=true
oracle.install.db.DBA_GROUP=dba
oracle.install.db.OPER_GROUP=dba
```
## 10、安装
先切换到oracle
```
su oracle
cd /db/database/
./runInstaller -silent -ignorePrereq -responseFile /db/etc/response/db_install.rsp
```
![](http://wx1.sinaimg.cn/large/e44344dcly1fr0kjjijlqj210e03y3yr.jpg)
可按他提示的查看日志，新增一个命令窗口，执行
```
tail -f  /u01/app/oracle/inventory/logs/installActions2018-05-04_11-48-18AM.log
```
安装成功：
![](http://wx1.sinaimg.cn/large/e44344dcly1fr0kjjtqyxj20os08rt99.jpg)
根据提示，执行
```bash
su
sh /u01/app/oracle/inventory/orainstRoot.sh
sh /db/app/oracle/product/11.2.0/root.sh
```
## 11配置静默监听
```
su oracle
netca /silent /responsefile /db/etc/netca.rsp
```
查看监听端口
```
netstat -tnulp | grep 1521
```
![](http://wx1.sinaimg.cn/large/e44344dcly1fr0kjk7e4xj20n201umx3.jpg)
## 11、静默创建数据库
```
vim /db/etc/dbca.rsp
GDBNAME = "orcl"
SID = "orcl"
SYSPASSWORD = "oracle"
SYSTEMPASSWORD = "oracle"
SYSMANPASSWORD = "oracle"
DBSNMPPASSWORD = "oracle"
DATAFILEDESTINATION =/db/app/oracle/oradata
RECOVERYAREADESTINATION=/db/app/oracle/fast_recovery_area
CHARACTERSET = "AL32UTF8"
TOTALMEMORY = "3277"
```
其中TOTALMEMORY 设置为总内存的80%（4*1024*0.8）
在root用户下执行(如果没有权限)
```
chown -R oracle:oinstall /db/etc/dbca.rsp   
```


执行静默建库

```bash
dbca -silent -responseFile /db/etc/dbca.rsp
```
![](http://wx2.sinaimg.cn/large/e44344dcly1fr0kjkstktj20pa0a5aag.jpg)
然后查看一下日志看看有没有报错
```bash
vim /db/app/oracle/cfgtoollogs/dbca/orcl/orcl.log
```
如下
```
Copying database files
DBCA_PROGRESS : 1%
DBCA_PROGRESS : 3%
DBCA_PROGRESS : 11%
DBCA_PROGRESS : 18%
DBCA_PROGRESS : 26%
DBCA_PROGRESS : 37%
Creating and starting Oracle instance
DBCA_PROGRESS : 40%
DBCA_PROGRESS : 45%
DBCA_PROGRESS : 50%
DBCA_PROGRESS : 55%
DBCA_PROGRESS : 56%
DBCA_PROGRESS : 60%
DBCA_PROGRESS : 62%
Completing Database Creation
DBCA_PROGRESS : 66%
DBCA_PROGRESS : 70%
DBCA_PROGRESS : 73%
DBCA_PROGRESS : 85%
DBCA_PROGRESS : 96%
DBCA_PROGRESS : 100%
Database creation complete. For details check the logfiles at:
 /db/app/oracle/cfgtoollogs/dbca/orcl.
Database Information:
Global Database Name:orcl
System Identifier(SID):orcl
```
查看oracle实例进程

```
ps -ef | grep ora_ | grep -v grep
```
```
root@ambari:~# ps -ef | grep ora_ | grep -v grep
oracle     3531      1  0 05:48 ?        00:00:00 ora_pmon_orcl
oracle     3533      1 11 05:48 ?        00:00:12 ora_vktm_orcl
oracle     3537      1  0 05:48 ?        00:00:00 ora_gen0_orcl
oracle     3539      1  0 05:48 ?        00:00:00 ora_diag_orcl
oracle     3541      1  0 05:48 ?        00:00:00 ora_dbrm_orcl
oracle     3543      1  0 05:48 ?        00:00:00 ora_psp0_orcl
oracle     3545      1  0 05:48 ?        00:00:00 ora_dia0_orcl
oracle     3547      1 16 05:48 ?        00:00:17 ora_mman_orcl
oracle     3549      1  0 05:48 ?        00:00:00 ora_dbw0_orcl
oracle     3551      1  0 05:48 ?        00:00:00 ora_lgwr_orcl
oracle     3553      1  0 05:48 ?        00:00:00 ora_ckpt_orcl
oracle     3555      1  0 05:48 ?        00:00:00 ora_smon_orcl
oracle     3557      1  0 05:48 ?        00:00:00 ora_reco_orcl
oracle     3559      1  1 05:48 ?        00:00:01 ora_mmon_orcl
oracle     3561      1  0 05:48 ?        00:00:00 ora_mmnl_orcl
oracle     3563      1  0 05:48 ?        00:00:00 ora_d000_orcl
oracle     3565      1  0 05:48 ?        00:00:00 ora_s000_orcl
oracle     3615      1  0 05:48 ?        00:00:00 ora_qmnc_orcl
oracle     4088      1  1 05:48 ?        00:00:00 ora_cjq0_orcl
oracle     4121      1  0 05:48 ?        00:00:00 ora_q000_orcl
oracle     4134      1  0 05:48 ?        00:00:00 ora_q001_orcl

```
查看监听状态

```bash
lsnrctl status
```
![](http://wx4.sinaimg.cn/large/e44344dcly1fr0kjm2nfzj20k10bdgmb.jpg)

## 12、登录到oracle，测试
```bash
sqlplus / as sysdba
select status from v$instance;
```
这是发现oracle执行任何语句报错如图：
![](http://wx2.sinaimg.cn/large/e44344dcly1fr0kjmh4ygj20pb07nq34.jpg)
崩溃~
## 13、各种错误及解决
### 13.1 首先检查前面的步骤有没有错的
如果没有，则执行后面，一开始我发现前面日志异常，第一次装没有经验，试了几下干脆卸载重装。
### 13.2 ORACLE not available
先根据ORACLE not available上网查了一下，解决方法：startup
### 13.3 startup 报错
错误：
```
could not open parameter file '/db/app/oracle/product/11.2.0/dbs/initORCL.ora'
```
### 13.4解决could not open parameter
执行以下命令即可（确保oracle用户对下面的文件夹有权限，前面已经执行过）
```
cp $ORACLE_BASE/admin/orcl/pfile/init.ora.43201822553 $ORACLE_HOME/dbs/initORCL.ora
```
参考：[Linux下无法启动oracle could not open parameter file 解决方法](https://blog.csdn.net/guchuanlong/article/details/7299079)
继续startup,又报错：MEMORY_TARGET not supported on this system
```
SQL> startup
ORA-00845: MEMORY_TARGET not supported on this system
```
### 13.5 解决 MEMORY_TARGET not supported on this system
root 用户下执行
```
mount -t tmpfs shmfs -o size=7g /dev/shm
```
参考：[ORA-00845: MEMORY_TARGET not supported on this system报错解决](https://www.linuxidc.com/Linux/2012-12/76976.htm)
继续startup
```
SQL> startup
ORACLE instance started.

Total System Global Area 1720328192 bytes
Fixed Size		    2214056 bytes
Variable Size		 1006634840 bytes
Database Buffers	  704643072 bytes
Redo Buffers		    6836224 bytes
ORA-01102: cannot mount database in EXCLUSIVE mode
```
如果执行查询会报错：database not mounted，因为上面已经报错ORA-01102: cannot mount database in EXCLUSIVE mode
### 13.6 解决 cannot mount database in EXCLUSIVE mode
先关闭数据库
```
shutdown immediate
```
```
SQL> shutdown immediate
ORA-01507: database not mounted


ORACLE instance shut down.

```
然后在root用户执行
```
fuser -k lkORCL
```
其中lkORCL 为自己设置oracle实例名的大写。
再执行fuser -u lkORCL没有任何输出即可
参考：[ORA-01507: database not mounted （转）](https://blog.csdn.net/qq_27966627/article/details/51062822)

这时再执行startup就可以了
```
SQL> startup
ORACLE instance started.

Total System Global Area 1720328192 bytes
Fixed Size		    2214056 bytes
Variable Size		 1006634840 bytes
Database Buffers	  704643072 bytes
Redo Buffers		    6836224 bytes
Database mounted.
Database opened.
SQL> select * from v$version;

BANNER
--------------------------------------------------------------------------------
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
PL/SQL Release 11.2.0.1.0 - Production
CORE	11.2.0.1.0	Production
TNS for Linux: Version 11.2.0.1.0 - Production
NLSRTL Version 11.2.0.1.0 - Production

SQL> 

```
再执行其他查询语句测试一下即可
## 14、创建用户供远程连接
开放1521端口
```bash
firewall-cmd --zone=public --add-port=1521/tcp --permanent
firewall-cmd --reload
```
```
create user bigdata identified by bigdata;
grant connect, resource to bigdata;
```
利用连接数据库的工具就可以远程连接oracle，如DBeaver,然后建表，插入几条记录，查询测试一下，具体方法不再赘述。

## 参考资料
[CentOS 7静默（无图形化界面）安装Oracle 11g](https://blog.csdn.net/Kenny1993/article/details/75038670)