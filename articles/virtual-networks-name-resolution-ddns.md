<properties
   pageTitle="使用动态 DNS 注册主机名"
   description="本页面提供有关如何设置动态 DNS 以在你自己的 DNS 服务器中注册主机名的详细信息。"
   services="virtual-network"
   documentationCenter="na"
   authors="GarethBradshawMSFT"
   manager="carmonm"
   editor="tysonn" />
<tags
	ms.service="virtual-network"
	ms.date="04/26/2016"
	wacn.date="05/30/2016"/>


# 使用动态 DNS 在自己的 DNS 服务器中注册主机名
[Azure 为虚拟机 (VM) 和角色实例提供名称解析](/documentation/articles/virtual-networks-name-resolution-for-vms-and-role-instances/)。但是，如果你需要 Azure 所提供的名称解析之外的名称解析，则可以提供自己的 DNS 服务器。这样你可以量身定制你的 DNS 解决方案以满足自己的特定需求。例如，你可能需要通过 Active Directory 域控制器来访问本地资源。

将自定义 DNS 服务器作为 Azure VM 托管时，可将针对同一 VNet 的主机名查询转发到 Azure 以解析主机名。如果不希望使用此路由，可使用动态 DNS 在 DNS 服务器中注册 VM 主机名。Azure 不具备直接在你的 DNS 服务器中创建记录的功能（如凭据），因此通常需要替代方案。以下是一些常见的替代方案。

## Windows 客户端
在启动时或其 IP 地址更改时，未加入域的 Windows 客户端将尝试不安全的动态 DNS (DDNS) 更新。DNS 名称为主机名加上的主 DNS 后缀。Azure 将保留主 DNS 后缀为空，但你可以通过 [UI](https://technet.microsoft.com/zh-cn/library/cc794784.aspx) 或[使用自动化](https://social.technet.microsoft.com/forums/windowsserver/3720415a-6a9a-4bca-aa2a-6df58a1a47d7/change-primary-dns-suffix)在 VM 中对其进行设置。

已加入域的 Windows 客户端通过使用安全的动态 DNS 将其 IP 地址注册到域控制器。域加入过程将在客户端上设置主 DNS 后缀并创建和维护信任关系。

## Linux 客户端
Linux 客户端通常在启动时注册到 DNS 服务器，并假设 DHCP 服务器将执行此操作。Azure 的 DHCP 服务器没有能力也没有凭据在你的 DNS 服务器中注册记录。可以使用名为 nsupdate 的工具发送动态 DNS 更新，该工具包含于绑定包。由于动态 DNS 协议是标准化的，所以即使在 DNS 服务器上未使用绑定时，也可以使用 nsupdate。

可以使用 DHCP 客户端提供的挂钩在 DNS 服务器中创建和维护主机名条目。在 DHCP 周期中，客户端将执行 /etc/dhcp/dhclient-exit-hooks.d/ 中的脚本。通过使用 nsupdate 可将其用于注册新的 IP 地址。例如：

    	#!/bin/sh
    	requireddomain=mydomain.local

    	# only execute on the primary nic
    	if [ "$interface" != "eth0" ]
    	then
    		return
    	fi

		# when we have a new IP, perform nsupdate
		if [ "$reason" = BOUND ] || [ "$reason" = RENEW ] ||
		   [ "$reason" = REBIND ] || [ "$reason" = REBOOT ]
		then
    		host=`hostname`
	    	nsupdatecmds=/var/tmp/nsupdatecmds
  			echo "update delete $host.$requireddomain a" > $nsupdatecmds
  			echo "update add $host.$requireddomain 3600 a $new_ip_address" >> $nsupdatecmds
  			echo "send" >> $nsupdatecmds

  			nsupdate $nsupdatecmds
		fi

		#done
		exit 0;

你还可以使用 nsupdate 命令来执行安全的动态 DNS 更新。例如，在使用 Bind DNS 服务器时，将[生成](http://linux.yyz.us/nsupdate/)公钥-私钥对。已向 DNS 服务器[配置](http://linux.yyz.us/dns/ddns-server.html)密钥的公共部分，所以它可以验证请求中的签名。必须使用 -k 选项将密钥对提供给 nsupdate，以便对动态 DNS 更新请求进行签名。

如果你使用 Windows DNS 服务器，可以使用 nsupdate 中的 -g 参数进行 Kerberos 身份验证（在 nsupdate 的 Windows 版本中不可用）。若要执行此操作，请使用 kinit 加载凭据（例如从 [keytab 文件](http://www.itadmintools.com/2011/07/creating-kerberos-keytab-files.html)进行加载）。然后 nsupdate g 将拾取缓存内的凭据。

如果需要，可以将 DNS 搜索后缀添加到 VM。DNS 后缀在 /etc/resolv.conf 文件中指定。大多数 Linux 发行版自动管理此文件中的内容，因此通常不能对其进行编辑。但是，你可以通过使用 DHCP 客户端的 supersede 命令重写该后缀。若要执行此操作，请在 /etc/dhcp/dhclient.conf 中添加：

		supersede domain-name <required-dns-suffix>;


<!---HONumber=Mooncake_0523_2016-->