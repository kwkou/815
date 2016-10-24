<properties
	pageTitle="Linux VM 的常见问题 | Azure"
	description="回答了通过 Resource Manager 模型创建的 Linux 虚拟机的一些常见问题。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-resource-management"/>  


<tags
	ms.service="virtual-machines-linux"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-linux"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/16/2016"
	wacn.date=""
	ms.author="cynthn"/>  


# 有关 Linux 虚拟机的常见问题 

本文讨论有关在 Azure 中使用 Resource Manager 部署模型创建的 Linux 虚拟机的一些常见问题。有关本主题的 Windows 版本，请参阅[有关 Windows 虚拟机的常见问题](/documentation/articles/virtual-machines-windows-faq/)

## 我可以在 Azure VM 上运行什么程序？

所有订户都可以在 Azure 虚拟机上运行服务器软件。有关详细信息，请参阅 [Azure 认可的分发中的 Linux](/documentation/articles/virtual-machines-linux-endorsed-distros/)


## 使用虚拟机时，我可以使用多少存储？

每个数据磁盘的容量高达 1 TB。你可以使用的数据磁盘的数目取决于虚拟机的大小。有关详细信息，请参阅[虚拟机大小](/documentation/articles/virtual-machines-linux-sizes/)。

Azure 存储帐户提供可用于操作系统磁盘和任意数据磁盘的存储。每个磁盘都是一个 .vhd 文件，以页 blob 形式存储。有关定价详细信息，请参阅 [Storage Pricing Details](/pricing/details/storage/)（存储定价详细信息）。


## 如何访问我的虚拟机？

使用安全外壳 (SSH) 建立远程连接，以登录到虚拟机。请参阅如何[从 Windows](/documentation/articles/virtual-machines-linux-ssh-from-windows/) 或[从 Linux 和 Mac](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/) 进行连接的相关说明。默认情况下，SSH 允许的并发连接最多为 10 个。通过编辑配置文件，可以增大此数目。


如果遇到问题，请查阅[排除安全外壳 (SSH) 连接故障](/documentation/articles/virtual-machines-linux-troubleshoot-ssh-connection/)。


## 我是否可以使用临时磁盘 (/dev/sdb1) 存储数据？

不要使用临时磁盘 (/dev/sdb1) 存储数据。它只是用于临时存储。有丢失无法恢复的数据的风险。


## 我是否可以复制或克隆现有的 Azure VM？

是的。有关说明，请参阅[如何在 Resource Manager 部署模型中创建 Linux 虚拟机的副本](/documentation/articles/virtual-machines-linux-copy-vm/)。

## 创建 VM 后能否向 VM 添加 NIC？

否。添加 NIC 只能在创建时进行。


## 是否有任何计算机名称要求？

是的。计算机名称的最大长度为 64 个字符。有关命名资源的详细信息，请参阅[基础结构命名准则](/documentation/articles/virtual-machines-linux-infrastructure-naming-guidelines/)。


## 创建 VM 时，用户名有什么要求？

用户名的长度必须为 1 到 64 个字符。

不允许使用以下用户名：

<table>
	<tr>
		<td style="text-align:center">administrator </td><td style="text-align:center"> admin </td><td style="text-align:center"> user </td><td style="text-align:center"> user1</td>
	</tr>
	<tr>
		<td style="text-align:center">test </td><td style="text-align:center"> user2 </td><td style="text-align:center"> test1 </td><td style="text-align:center"> user3</td>
	</tr>
	<tr>
		<td style="text-align:center">admin1 </td><td style="text-align:center"> 1 </td><td style="text-align:center"> 123 </td><td style="text-align:center"> a</td>
	</tr>
	<tr>
		<td style="text-align:center">actuser  </td><td style="text-align:center"> adm </td><td style="text-align:center"> admin2 </td><td style="text-align:center"> aspnet</td>
	</tr>
	<tr>
		<td style="text-align:center">backup </td><td style="text-align:center"> console </td><td style="text-align:center"> david </td><td style="text-align:center"> guest</td>
	</tr>
	<tr>
		<td style="text-align:center">john </td><td style="text-align:center"> owner </td><td style="text-align:center"> root </td><td style="text-align:center"> server</td>
	</tr>
	<tr>
		<td style="text-align:center">sql </td><td style="text-align:center"> support </td><td style="text-align:center"> support_388945a0 </td><td style="text-align:center"> sys</td>
	</tr>
	<tr>
		<td style="text-align:center">test2 </td><td style="text-align:center"> test3 </td><td style="text-align:center"> user4 </td><td style="text-align:center"> user5</td>
	</tr>
</table>


## 创建 VM 时，密码有什么要求？

密码的长度必须为 6 到 72 个字符，并满足以下 4 个复杂性要求中的 3 个要求：

- 具有小写字符
- 具有大写字符
- 具有数字
- 具有特殊字符（正则表达式匹配 [\W_]）

不允许使用以下密码：

<table>
	<tr>
		<td style="text-align:center">abc@123</td><td style="text-align:center">P@$$w0rd</td><td style="text-align:center">P@ssw0rd</td><td style="text-align:center">P@ssword123</td><td style="text-align:center">Pa$$word</td>
	</tr>
	<tr>
		<td style="text-align:center">pass@word1</td><td style="text-align:center">Password!</td><td style="text-align:center">Password1</td><td style="text-align:center">Password22</td><td style="text-align:center">iloveyou!</td>
	</tr>
</table>

<!---HONumber=Mooncake_1017_2016-->