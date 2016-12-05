<properties 
	pageTitle="Linux 虚拟机虚拟网卡问题导致无法连接问题" 
	description="Linux 虚拟机虚拟网卡问题导致无法连接问题。" 
	services="virtual machine" 
	documentationCenter="" 
	authors=""
	manager="" 
	editor=""/>
<tags ms.service="virtual-machines-aog" ms.date="" wacn.date="12/05/2016"/>
# Linux 虚拟机虚拟网卡问题导致无法连接问题 #

### 问题描述 ###

当 Linux 虚拟机启动时,通过串口输出或者启动日志, 观察到虚拟网卡启动或者初始化故障, 导致虚拟机无法连接.

### 问题分析 ###

常见的超时报错范例如下:

- **CentOS**
 
		Bringing up loopback interface:  [  OK  ]
		Bringing up interface eth0:  Device eth0 has different MAC address than expected, ignoring.  [FAILED]

- **SUSE**
 
		Setting up (localfs) network interfaces:
		    lo        
		    lo        IP address: 127.0.0.1/8   
		              IP address: 127.0.0.2/8   
		done   	eth4      		            No configuration found for eth4
		unused Waiting for mandatory devices:  eth0 
		29 28 27 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10 9 8 7 6 5 4 3 2 1 0 
		 
		eth0                                No interface found		

### 解决方案 ###

- **CentOS**
 
	上述报错指向了 `/etc/sysconfig/network-scripts/ifcfg-eth0` 配置文件里针对 eth0 错误配置了虚拟网卡的硬件地址.解决方案为:

	1.	删除虚拟机保留磁盘,并将系统盘作为数据盘挂载到临时虚拟机上.
	2.	修改配置文件 `/etc/sysconfig/network-scripts/ifcfg-eth0`
	3.	将虚拟网卡硬件地址改成正确的值,或者删除该行.
	4.	保存并退出.
	5.	分离该磁盘, 并基于该磁盘新建虚拟机.

- **SUSE**
 
	上述报错是由于硬件设备映射规则 `/etc/udev/rules.d` 中对 eth0 重命名为了 eth4.解决方案为:

	1.	删除虚拟机保留磁盘,并将系统盘作为数据盘挂载到临时虚拟机上.
	2.	删除 `/etc/udev/rules.d` 中针对 eth0 的命名规则.
	3.	分离该磁盘, 并基于该磁盘新建虚拟机.