---

title: linux ssh 免密登录
date: 2018-04-05
tags:
  - linux
copyright: true
---
以下用三台centos为例，ip分别为192.168.44.138、192.168.44.139、192.168.44.140，分别对应别名master、slave1、slave2
## 1、首先在每个机器上执行
``` bash
ssh-keygen -t rsa
```
一直按回车默认就好
![](http://wx3.sinaimg.cn/large/e44344dcly1fq23r2lbinj20ek0973yr.jpg)
<!-- more -->
## 2、将公钥导入到认证文件中
将三个机器上id_rsa.pub的内容合并起来放在authorized_keys，可以用命令按下面的步骤来，也可以分别将三个机器的公钥拷贝出来放在一个文件里，之后分别复制到各自机器的authorized_keys，我用的是下面的命令
### 2.1 在master上执行
``` bash
cd ~/.ssh
cat id_rsa.pub>>authorized_keys
```
（这时如果配单机的话，就可以免密登录本机了，可以执行ssh localhost 或ssh master验证一下，如下图）
![](http://wx3.sinaimg.cn/large/e44344dcly1fq23r3v47dj21h7035mxd.jpg)

如果不能免密登录，可能是文件权限不对，执行下面的命令，再验证一下
``` bash
chmod 710 authorized_keys
```
然后将master的authorized_keys传到slave1上的.ssh目录下
``` bash
scp -r authorized_keys root@slave1:~/.ssh
```
![](http://wx1.sinaimg.cn/large/e44344dcly1fq23r25etxj21g303iglw.jpg)
### 2.2 在slave1上执行
``` bash
cd ~/.ssh
cat id_rsa.pub>>authorized_keys
scp -r authorized_keys root@slave2:~/.ssh
```
这一步实际是将salve1的id_rsa.pub和master传过来的authorized_keys里的内容合并起来存到authorized_keys，然后将authorized_keys传到slave2机器上
### 2.3 在slave2上执行
``` bash
cd ~/.ssh
cat id_rsa.pub>>authorized_keys
scp -r authorized_keys root@master:~/.ssh
scp -r authorized_keys root@slave1:~/.ssh
```
这一步实际是将salve2的id_rsa.pub和slave1传过来的authorized_keys里的内容合并起来存到authorized_keys，然后将authorized_keys传到master、slave1机器上。
到这里，每台机器上的authorized_keys都含有三台机器的公钥，在每台机器上验证一下是否可以免密ssh登录到三台机器上了。
```bash
ssh master
ssh slave1
ssh slave2
```
如果都不需要输入密码,就代表配置成功！

