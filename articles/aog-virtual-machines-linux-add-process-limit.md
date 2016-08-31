<properties 
	pageTitle="修改 Linux VM 中单个用户最大进程数的限制" 
	description="修改 Linux VM 中单个用户最大进程数的限制" 
	services="virtual-machines" 
	documentationCenter="" 
	authors=""
	manager="" 
	editor=""/>
<tags ms.service="virtual-machines-aog" ms.date="" wacn.date="08/31/2016"/>
#修改 Linux VM 中单个用户最大进程数的限制

在部署有并发任务执行的虚机上, 会遇到 SSH 无法访问的问题. 本文将帮助你找出其中一种比较特殊的原因, 并提供解决方案。  
>[AZURE.NOTE] 以下案例分析基于 CentOS 7, 对于其他版本的 Linux 操作系统, 会略有不同, 请注意。

##症状描述

虚机在正常运行过程中，突然发现 SSH 连接失败。重启虚机以后，SSH 连接恢复正常。再运行一段时间之后，又发生同样的问题。

##问题分析

- 经过日志分析，azure 平台和虚机运行均无异常。但是从 /var/log/secure.log 里发现如下信息 “sshd[23106]: error: do_exec_pty: fork: Resource temporarily unavailable”
- 根据了解，该虚机上运行的应用程序属于并发，在同一时间段，同一用户的进程数会超过 10000.已经超过了默认的 4096 最大值。这是导致本次 SSH 登陆失败的原因

##解决方案

- 重启虚拟机并以管理员登陆虚拟机，切换成 root 用户
- 查看文件 /etc/security/limits.d/20-nproc.conf， 默认应该为如下内容

		# Default limit for number of user's processes to prevent
		# accidental fork bombs.
		# See rhbz #432903 for reasoning.	
		*          soft    nproc     4096
		root       soft    nproc     unlimited

- 编辑文件 /etc/security/limits.d/20-nproc.conf，将高亮显示行内的 4096，调整为相应的值，或者改成 unlimited.

更多详细介绍， 请参考[这篇文档](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Migration_Planning_Guide/sect-Red_Hat_Enterprise_Linux-Migration_Planning_Guide-System_Management.html)

