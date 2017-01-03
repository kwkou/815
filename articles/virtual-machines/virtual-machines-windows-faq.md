<properties
	pageTitle="Windows VM 的常见问题 | Azure"
	description="解答通过 Resource Manager 模型创建 Windows 虚拟机的一些常见问题。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-resource-management"/>  


<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/16/2016"
	wacn.date="12/20/2016"
	ms.author="cynthn"/>  


# 有关 Windows 虚拟机的常见问题 


本文讨论了在 Azure 中使用 Resource Manager 部署模型创建的 Windows 虚拟机的一些常见问题。有关本主题的 Linux 版本，请参阅[有关 Linux 虚拟机的常见问题](/documentation/articles/virtual-machines-linux-faq/)

## 可以在 Azure VM 上运行哪些程序？

所有订户都可以在 Azure 虚拟机上运行服务器软件。有关在 Azure 中运行 Microsoft 服务器软件的支持策略的信息，请参阅 [Microsoft server software support for Azure Virtual Machines](https://support.microsoft.com/zh-cn/kb/2721672)（对 Azure 虚拟机中的 Microsoft 服务器软件的支持）

某些版本的 Windows 7 和 Windows 8.1 可供 MSDN Azure 权益订户以及 MSDN 开发和测试即用即付订户用于开发和测试任务。有关详细信息（包括说明和限制），请参阅 [Windows Client images for MSDN subscribers（适用于 MSDN 订户的 Windows 客户端映像）](http://azure.microsoft.com/blog/2014/05/29/windows-client-images-on-azure/)。


## 虚拟机可以使用多少存储空间？

每个数据磁盘容量最高可达 1 TB。可以使用的数据磁盘数取决于虚拟机大小。有关详细信息，请参阅[虚拟机大小](/documentation/articles/virtual-machines-windows-sizes/)。

Azure 存储帐户可为操作系统磁盘和任何数据磁盘提供存储空间。每个磁盘都是一个 .vhd 文件，存储作为页 blob。有关定价详细信息，请参阅 [Storage Pricing Details](/pricing/details/storage/)（存储定价详细信息）。


## 如何访问我的虚拟机？

使用适用于 Windows VM 的远程桌面连接 (RDP) 建立远程连接。有关说明，请参阅 [How to connect and log on to an Azure virtual machine running Windows](/documentation/articles/virtual-machines-windows-connect-logon/)（如何连接并登录到运行 Windows 的 Azure 虚拟机）。除非将服务器配置为远程桌面服务会话主机，否则最多支持两个并发连接。


如果在使用远程桌面时遇到问题，请参阅 [Troubleshoot Remote Desktop connections to a Windows-based Azure Virtual Machine](/documentation/articles/virtual-machines-windows-troubleshoot-rdp-connection/)（对与基于 Windows 的 Azure 虚拟机的远程桌面连接进行故障排除）。

如果熟悉 Hyper-V，可以查找与 VMConnect 类似的工具。Azure 不提供类似工具，因为不支持通过控制台访问虚拟机。

## 我是否可以使用临时磁盘（默认为 D: 驱动器）存储数据？

不要使用临时磁盘来存储数据。它只是临时存储空间，因此存在丢失数据且数据不能恢复的风险。将虚拟机移到另一主机时，可能会发生数据丢失的情况。调整虚拟机大小、更新主机、主机硬件故障等都是需要移动虚拟机的原因。

如果有应用程序需要使用 D: 驱动器号，可以重新分配驱动器号以便临时磁盘使用除 D: 以外的位置。有关说明，请参阅 [Change the drive letter of the Windows temporary disk](/documentation/articles/virtual-machines-windows-classic-change-drive-letter/)（更改 Windows 临时磁盘的驱动器号）。

## 如何更改临时磁盘的驱动器号？

可以通过移动页面文件和重新分配驱动器号来更改驱动器号，但需确保按特定顺序执行这些步骤。有关说明，请参阅 [Change the drive letter of the Windows temporary disk](/documentation/articles/virtual-machines-windows-classic-change-drive-letter/)（更改 Windows 临时磁盘的驱动器号）。

## 可否将现有 VM 添加到可用性集？

不可以。如果希望将 VM 作为可用性集的一部分，则需要在该可用性集中创建 VM。目前，不支持在创建 VM 后将其添加到可用性集。

## 可否将虚拟机上载到 Azure？

可以。有关说明，请参阅 [Upload a Windows VM image to Azure ](/documentation/articles/virtual-machines-windows-upload-image/)（将 Windows VM 映像上载到 Azure）

## 可否调整 OS 磁盘的大小？

可以。有关说明，请参阅 [How to expand the OS drive of a Virtual Machine in an Azure Resource Group](/documentation/articles/virtual-machines-windows-expand-os-disk/)（如何扩展 Azure 资源组中虚拟机的 OS 驱动器）。

## 可否复制或克隆现有的 Azure VM？

可以。有关说明，请参阅 [How to create a copy of a Windows virtual machine in the Resource Manager deployment model](/documentation/articles/virtual-machines-windows-vhd-copy/)（如何在 Resource Manager 部署模型中创建 Windows 虚拟机副本）。

## Azure 可否支持 Linux VM？

可以。若要快速创建 Linux VM 进行试用，请参阅 [Create a Linux VM on Azure using the Portal](/documentation/articles/virtual-machines-linux-quick-create-portal/)（使用门户在 Azure 上创建 Linux VM）。

## 创建 VM 后能否向 VM 添加 NIC？

否。添加 NIC 只能在创建时进行。

## 是否有任何计算机名称要求？

是的。计算机名称的最大长度为 15 个字符。有关命名资源的详细信息，请参阅 [Infrastructure naming guidelines](/documentation/articles/virtual-machines-windows-infrastructure-naming-guidelines/)（基础结构命名准则）。

## <a name="what-are-the-username-requirements-when-creating-a-vm"></a> 创建 VM 时，用户名有什么要求？

用户名最长为 20 个字符，不能以句点（“.”）结尾。

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

## <a name="what-are-the-password-requirements-when-creating-a-vm"></a> 创建 VM 时，密码有什么要求？

密码的长度必须为 8 到 123 个字符，并满足以下 4 个复杂性要求中的 3 个要求：

- 具有小写字符
- 具有大写字符
- 具有数字
- 具有特殊字符（正则表达式匹配 [\\W\_]）

不允许使用以下密码：

不允许使用以下密码：
<table>
	<tr>
		<td style="text-align:center">abc@123</td><td style="text-align:center">P@$$w0rd</td><td style="text-align:center">P@ssw0rd</td><td style="text-align:center">P@ssword123</td><td style="text-align:center">Pa$$word</td>
	</tr>
	<tr>
		<td style="text-align:center">pass@word1</td><td style="text-align:center">Password!</td><td style="text-align:center">Password1</td><td style="text-align:center">Password22</td><td style="text-align:center">iloveyou!</td>
	</tr>
</table>

## 我的 windows 虚拟机为何会被自动重启？

答：通过 Azure 平台部署的 windows 虚拟机，依照最佳实践方案，默认启用了 windows 自动更新，来保证系统的更新和安全。遇到重大更新的时候，虚拟机会自动重启虚拟机，使之生效。如果您不希望自动更新影响到您的在线运行，可以在部署完毕以后，选择自动下载更新但是手动安装。

<!---HONumber=Mooncake_Quality_Review_1118_2016-->