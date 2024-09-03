---
layout: post
title: 学习笔记｜服务器使用记录
categories: [Study Notes]
description: 自己租用的以及实验室服务器使用过程中的一些记录
keywords: Linux，Ubuntu
mermaid: false
sequence: false
flow: false
mathjax: true
mindmap: false
mindmap2: false
---

​	自己租用的以及实验室服务器使用过程中的一些记录。

## 正文

1、服务器重装以后，ssh免密登录失败。

​	错误：ECDSA host key "ip地址" for has changed and you have requested strict checking.记录下方便记忆。

​	解决方案：在终端上输入以下命令：

​	ssh-keygen -R "你的远程服务器ip地址"

2、WSL ubuntu忘记root密码和用户密码

​	1）重置root密码

​	①以管理员身份打开 PowerShell

​	②输入命令 wsl.exe --user root

​	③输入命令 passwd root 修改 root 密码

​	④输入新的密码并确认后，root密码即得到重置

2）重置用户密码

​	重置与用户密码与重置root密码类似，

​	①以管理员身份打开 PowerShell

​	②输入命令 wsl.exe --user root

​	③输入命令 passwd username 修改用户密码，username即待重置的用户的名称

3、将Windows系统中文件传入WSL子系统中

​	在ubuntu系统中可以直接访问/mnt目录，就是win的系统文件。然后使用mv或者cp就可以传入了。

4、公钥密钥免密登录服务器

​	①创建公钥

​	ssh-keygen 

​	出现如下内容：

​	Enter file in which to save the key (/home/zhangyu/.ssh/id_rsa):

​	直接使用默认选项，回车即可，出现如下内容：

​	Enter passphrase (empty for no passphrase):

​	直接回车，出现内容：

​	Enter same passphrase again:

​	直接回车，创建完成。

​	②修改本地登录用户的~/.ssh/config文件

​	Host hostname 

​	user username

​	Port 1

​	把username改成指定用户就不需要输入root@tencent了

​	③拷贝公钥

​	 ssh-copy-id tencent

5、修改hosts文件

​	vi /etc/hosts

​	ip 别名

6、放行端口

​	firewall-cmd –state 查看防火墙运行状态

​	systemctl start firewalld.service 启动防火墙

​	firewall-cmd --zone=public --add-port=8888/tcp --permanent 放行端口8888

​	firewall-cmd –reload 重启防火墙

7、centos切换字符集

​	echo $LANG 查看当前字符集

​	yum install kde-l10n-Chinese 安装中文字符集

​	echo "export LANG=zh_CN.UTF-8" >> /etc/profile 修改为中文字符集永久

​	source /etc/profile 刷新

8、基于宝塔面板安装cloudreve

​	 https://blog.csdn.net/qq_49488584/article/details/126884686
