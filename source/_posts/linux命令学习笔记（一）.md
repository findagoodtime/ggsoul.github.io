---
title: linux命令学习笔记（一）
date: 2016-06-14 22:28:24
tags: linux
categories: work
---

关于一下工作中用到的 linux 命令，每次遇到点问题都 google 好烦

<!--more-->

## 基本命令

* 创建文件夹 `mkdir file`
* 删除文件 `rm -rf file`
* 文件重命名 `mv file1 file2`
* 复制文件 `cp file1 file2`
* 查找文件 `find . -name 'file'`
* 分析一行 `grep [--color=auto] '查找字符串' filename`
* 打印日志 `tail -f xxxlog`
* 获取指定程序的进程 `ps -ef|grep 指定程序`
* 查看运行程序的状态 `netstat -tunpl`
* 杀死进程 `kill -9 pid`
	
## linux bash shell中获取本机ip地址

* 方法一：
```
/sbin/ifconfig -a|grep inet|grep -v 127.0.0.1|grep -v inet6|awk '{print $2}'|tr -d "addr:"
or
/sbin/ifconfig|sed -n '/inet addr/s/^[^:]*:\([0-9.]\{7,15\}\) .*/\1/p'
```
* 方法二： 
```
local_host="`hostname --fqdn`"
local_ip=`host $local_host 2>/dev/null | awk '{print $NF}'`
echo $local_ip
```
* 方法三：
```
local_host="`hostname --fqdn`"
nslookup -sil $local_host 2>/dev/null | grep Address: | sed '1d' | sed 's/Address://g'
```

## 从一个机器传到另一个机器

本机IP：192.168.138.150

要传送的IP地址为：192.168.138.151

任务：拷贝/etc/ha.d/ldirectord.cf文件到151机器上，地址为：/etc/ha.d

在本机上操作，使用命令scp：

以下操作是从本地拷贝到服务器上

scp /etc/ha.d/ldirectord.cf root@192.168.138.151:/etc/ha.d

## Linux下查看、关闭及开启防火墙命令

* 永久性生效，重启后不会复原 
> 开启： chkconfig iptables on
> 关闭： chkconfig iptables off 
* 即时生效，重启后复原 
> 开启： service iptables start
> 关闭： service iptables stop 
> 需要说明的是对于Linux下的其它服务都可以用以上命令执行开启和关闭操作。
> 在开启了防火墙时，做如下设置，开启相关端口， 修改/etc/sysconfig/iptables 文件，添加以下内容： 
> `-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT `
> `-A RH-Firewall-1-INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT`
* 查看防火墙状态
> chkconfig iptables --list

## memcached启动脚本

`/usr/local/bin/memcached -d -m 6144 -u root -l 192.168.1.151 -p 12000 -c 256 -P /tmp/memcached.pid`

* -d 选择是启动一个守护的进程
* -m 设置内存大小
* -u 是运行memcache的用户
* -l 是监听的服务器Ip地址
* -p 是社会memcache监听的端口号
* -c 选项是最大的并发连接数
* -p 是设置保存Memcache的pid文件

## 切换用户

su root（对应用户）
su 后面不加用户是默认切到 root
su 是不改变当前变量
`su -` 是改变为切换到用户的变量 
也就是说 su 只能获得 root 的执行权限，不能获得环境变量
而 `su -` 是切换到 root 并获得 root 的环境变量及执行权限

## vim设置

设置编码： `set encoding=utf-8`

粘贴代码缩进混乱：取消自动缩进`set paste`，敲代码的时候再设置回来`set nopaste`

> 注：在 normal 模式，输入 `:` 然后设置