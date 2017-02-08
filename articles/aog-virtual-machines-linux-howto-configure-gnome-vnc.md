<properties
	pageTitle="Linux 虚拟机中配置 GNOME + VNC"
	description="以 CentOS 为例讲述 Linux 虚拟机中配置 GNOME + VNC"
	service=""
	resource="virtualmachines"
	authors=""
	displayOrder=""
	selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Virtual Machines, Linux, CentOS, GNOME, VNC"
    cloudEnvironments="MoonCake" />
<tags
	ms.service="virtual-machines-linux-aog"
	ms.date=""
	wacn.date="1/20/2017" />
# Linux 虚拟机中配置 GNOME + VNC

## 需求描述

在特定的需求下，需要用到 Linux 的图形化界面，但是 Azure 平台提供的虚拟机默认没有开放远程图形化登陆的功能。以下解决方案，提供了市面上非常流行的 GNOME + VNC 的组合来远程图形化管理虚拟机。

>[AZURE.NOTE]以下步骤适用于 CentOS 6.x 版本，其他版本可能略微有区别。

## 解决方案

按照以下步骤完成 GNOME + VNC 的安装 :

1.	下载 GNOME :

	由于 GNOME 组件中包括了 NetworkManager 的软件包，而该软件包已经包含在 WALinuxAgent 的软件包中，为了避免冲突，建议按照如下步骤进行GNOME的安装：

	1.	登陆虚拟机，切换管理员身份。

	2.	编辑 `/etc/yum.conf` 文件，在最后一行加入： `exclude=NetworkManager*`

	3.	保存并退出

	4.	执行命令：`# yum clean all`

	5.	执行命令：`# yum groupinstall basic-desktop desktop-platform x11 fonts`

2.	配置GNOME :

	1.	编辑文件 `~/.xinitrc`（如果不存在，则新建），加入：`exec gnome-session`
	
	2.	保存并退出
	
	3.	编辑文件 `~/.bashrc` (如果非 bash，则修改相对应的文件)，加入以下内容：
	
			if [ $TERM == "xterm" ]; then
			   		export TERM=xterm-color
			fi

3.	将图形化界面设置为默认 :

	1.	编辑文件 `/etc/inittab` 将如下内容：
	
		id:3:initdefault:

	2.	替换成
		
		id:5:initdefault:

	3.	保存并退出

4.	安装 VNC :

	1.	执行命令：`# yum install tigervnc-server`
	
	2.	安装完毕以后，执行命令：`# vncserver`
	
	3.	第一次执行时，需要设置密码，默认端口号为 `5901`，从 `:1`依次加 1

5.	在虚拟机上配置相应终结点，开放 VNC 端口。

6.	通过客户端 VNC Viewer 远程登录虚拟机。 
