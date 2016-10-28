<properties 
	pageTitle="SSH 无法启动的原因分析及解决方法" 
	description="本文介绍 SSH 无法启动的原因分析及解决方法" 
	services="virtual machine" 
	documentationCenter="" 
	authors=""
	manager="" 
	editor=""/>
<tags ms.service="virtual-machine-aog" ms.date="" wacn.date="10/28/2016"/>

# SSH 无法启动的原因分析及解决方法 #

## 简介

Secure Shell（缩写为 SSH），由 IETF 的网络工作小组（Network Working Group）所制定；SSH 为一项创建在应用层和传输层基础上的安全协议，为计算机上的 Shell（壳层）提供安全的传输和使用环境。
  
传统的网络服务程序，如 rsh、FTP、POP 和 Telnet 其本质上都是不安全的；因为它们在网络上用明文传送数据、用户帐号和用户口令，很容易受到中间人（man-in-the-middle）攻击方式的攻击。就是存在另一个人或者一台机器冒充真正的服务器接收用户传给服务器的数据，然后再冒充用户把数据传给真正的服务器。
 
而 SSH 是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。利用 SSH 协议可以有效防止远程管理过程中的信息泄露问题。通过 SSH 可以对所有传输的数据进行加密，也能够防止 DNS 欺骗和 IP 欺骗。

SSH 之另一项优点为其传输的数据可以是经过压缩的，所以可以加快传输的速度。SSH 有很多功能，它既可以代替 Telnet，又可以为 FTP、POP、甚至为 PPP 提供一个安全的“通道”。

关于 SSH,详情参见如下:[https://zh.wikipedia.org/zh-cn/Secure_Shell](https://zh.wikipedia.org/zh-cn/Secure_Shell "https://zh.wikipedia.org/zh-cn/Secure_Shell")

## SSH 无法正常启动的几种报错信息 ##

1. ssh 相关<span style="background:#FFFF00">文件权限有误</span>导致无法启动, 报错参考如下:
    `Starting sshd: /var/empty/sshd must be owned by root and not group or world-writable.`
2. ssh <span style="background:#FFFF00">配置文件内容有误, 或语法有误</span>, 报错参考如下:
	`Starting sshd: [FAILED]`


## 解决方案 ##

**方案一:**

通过 [azure 新 portal](https://portal.azure.cn/) 的重置远程访问

![reset-remote](./media/aog-virtual-machines-troubleshoot-ssh-remote-fail/reset-remote.png "reset-remote")

> <span style="background:#FFFF00">** 注意: ** 如果配置文件被删除, 该功能不会重建, 参考其他方案进行修复. </span> 

**方案二:**

1. 删除虚拟机保留磁盘, 将该系统盘作为数据盘附加到临时虚拟机.
2. 登陆临时虚拟机, 进入到对应的目录, 进行手工修复.
3. 修复完毕以后, 分离该磁盘, 并基于此新建虚拟机.
4. 尝试通过SSH重新连接.

