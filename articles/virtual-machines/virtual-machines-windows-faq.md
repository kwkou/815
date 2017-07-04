<properties
    pageTitle="有关 Azure 中 Windows VM 的常见问题解答 | Azure"
    description="解答通过 Resource Manager 模型创建 Windows 虚拟机的一些常见问题。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="cynthn"
    manager="timlt"
    editor=""
    tags="azure-resource-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="757da816-a050-4889-a010-6f75d7978eb7"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/17/2017"
    wacn.date="04/17/2017"
    ms.author="cynthn"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="4dd63f1f4514251f319fb383a23d29d8ba73bf6b"
    ms.lasthandoff="04/06/2017" />

# <a name="frequently-asked-question-about-windows-virtual-machines"></a>有关 Windows 虚拟机的常见问题
本文讨论了在 Azure 中使用 Resource Manager 部署模型创建的 Windows 虚拟机的一些常见问题。 有关本主题的 Linux 版本，请参阅[有关 Linux 虚拟机的常见问题](/documentation/articles/virtual-machines-linux-faq/)

## <a name="what-can-i-run-on-an-azure-vm"></a>可以在 Azure VM 上运行哪些程序？
所有订户都可以在 Azure 虚拟机上运行服务器软件。 有关在 Azure 中运行 Microsoft 服务器软件的支持策略的信息，请参阅 [Microsoft server software support for Azure Virtual Machines](https://support.microsoft.com/zh-cn/kb/2721672)（对 Azure 虚拟机中的 Microsoft 服务器软件的支持）

## <a name="how-much-storage-can-i-use-with-a-virtual-machine"></a>虚拟机可以使用多少存储空间？
每个数据磁盘的容量高达 1 TB。 可以使用的数据磁盘数取决于虚拟机大小。 有关详细信息，请参阅[虚拟机大小](/documentation/articles/virtual-machines-windows-sizes/)。

Azure 存储帐户可为操作系统磁盘和任何数据磁盘提供存储空间。 每个磁盘都是一个 .vhd 文件，以页 blob 形式存储。 有关定价详细信息，请参阅 [Storage Pricing Details](/pricing/details/storage/)（存储定价详细信息）。

## <a name="how-can-i-access-my-virtual-machine"></a>如何访问我的虚拟机？
使用适用于 Windows VM 的远程桌面连接 (RDP) 建立远程连接。 有关说明，请参阅[如何连接并登录到运行 Windows 的 Azure 虚拟机](/documentation/articles/virtual-machines-windows-connect-logon/)。 除非将服务器配置为远程桌面服务会话主机，否则最多支持两个并发连接。  

如果在使用远程桌面时遇到问题，请参阅[排查连接到基于 Windows 的 Azure 虚拟机时的远程桌面连接问题](/documentation/articles/virtual-machines-windows-troubleshoot-rdp-connection/)。 

如果熟悉 Hyper-V，可以查找与 VMConnect 类似的工具。 Azure 不提供类似工具，因为不支持通过控制台访问虚拟机。

## <a name="can-i-use-the-temporary-disk-the-d-drive-by-default-to-store-data"></a>我是否可以使用临时磁盘（默认为 D: 驱动器）存储数据？
不要使用临时磁盘来存储数据。 它只是临时存储空间，因此存在丢失数据且数据不能恢复的风险。 将虚拟机移到另一主机时，可能会发生数据丢失的情况。 调整虚拟机大小、更新主机、主机硬件故障等都是需要移动虚拟机的原因。

如果有应用程序需要使用 D: 驱动器号，可以重新分配驱动器号以便临时磁盘使用除 D: 以外的位置。 有关说明，请参阅[更改 Windows 临时磁盘的驱动器号](/documentation/articles/virtual-machines-windows-classic-change-drive-letter/)。

## <a name="how-can-i-change-the-drive-letter-of-the-temporary-disk"></a>如何更改临时磁盘的驱动器号？
可以通过移动页面文件和重新分配驱动器号来更改驱动器号，但需确保按特定顺序执行这些步骤。 有关说明，请参阅[更改 Windows 临时磁盘的驱动器号](/documentation/articles/virtual-machines-windows-classic-change-drive-letter/)。

## <a name="can-i-add-an-existing-vm-to-an-availability-set"></a>可否将现有 VM 添加到可用性集？
不可以。 如果希望将 VM 作为可用性集的一部分，则需要在该可用性集中创建 VM。 目前，不支持在创建 VM 后将其添加到可用性集。
## <a name="can-i-upload-a-virtual-machine-to-azure"></a>可否将虚拟机上载到 Azure？
可以。 有关说明，请参阅 [Upload a Windows VM image to Azure](/documentation/articles/virtual-machines-windows-upload-image/)（将 Windows VM 映像上载到 Azure）

## <a name="can-i-resize-the-os-disk"></a>可否调整 OS 磁盘的大小？
可以。 有关说明，请参阅 [How to expand the OS drive of a Virtual Machine in an Azure Resource Group](/documentation/articles/virtual-machines-windows-expand-os-disk/)（如何扩展 Azure 资源组中虚拟机的 OS 驱动器）。

## <a name="can-i-copy-or-clone-an-existing-azure-vm"></a>可否复制或克隆现有的 Azure VM？
可以。 有关说明，请参阅 [How to create a copy of a Windows virtual machine in the Resource Manager deployment model](/documentation/articles/virtual-machines-windows-vhd-copy/)（如何在 Resource Manager 部署模型中创建 Windows 虚拟机的副本）。

## <a name="does-azure-support-linux-vms"></a>Azure 可否支持 Linux VM？
可以。 若要快速创建 Linux VM 进行试用，请参阅[使用门户在 Azure 上创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-portal/)。

## <a name="can-i-add-a-nic-to-my-vm-after-its-created"></a>创建 VM 后能否向 VM 添加 NIC？
能，目前可行。 首先需停止解除分配 VM。 然后便可添加或删除 NIC（除非它是 VM 上的最后一个 NIC）。 

## <a name="are-there-any-computer-name-requirements"></a>是否有任何计算机名称要求？
可以。 计算机名称的最大长度为 15 个字符。 有关命名资源的详细信息，请参阅 [Infrastructure naming guidelines](/documentation/articles/virtual-machines-windows-infrastructure-naming-guidelines/)（基础结构命名准则）。

## <a name="what-are-the-username-requirements-when-creating-a-vm"></a> 创建 VM 时，用户名有什么要求？

用户名最长为 20 个字符，不能以句点（“.”）结尾。 

不允许使用以下用户名：
<table>
    <tr>
        <td style="text-align:center">administrator </td><td style="text-align:center"> admin </td><td style="text-align:center"> user </td><td style="text-align:center"> user1</td>
    </tr>
    <tr>
        <td style="text-align:center">test </td><td style="text-align:center"> user2 </td><td style="text-align:center"> test1 </td><td style="text-align:center"> user3</td>
    </tr>    <tr>
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
密码的长度必须为 12 到 123 个字符，并满足以下 4 个复杂性要求中的 3 个要求：

* 具有小写字符
* 具有大写字符
* 具有数字
* 具有特殊字符（正则表达式匹配 [\W_]）

不允许使用以下密码：

<table>
    <tr>
        <td>abc@123 </td>
        <td>P@$$w0rd </td>
        <td>P@ssw0rd </td>
        <td>P@ssword123 </td>
        <td>Pa$$word </td>
    </tr>
    <tr>
        <td>pass@word1 </td>
        <td>Password! </td>
        <td>Password1 </td>
        <td>Password22 </td>
        <td>iloveyou! </td>
    </tr>
</table>

## <a name="-windows-"></a>我的 windows 虚拟机为何会被自动重启？

答：通过 Azure 平台部署的 windows 虚拟机，依照最佳实践方案，默认启用了 windows 自动更新，来保证系统的更新和安全。遇到重大更新的时候，虚拟机会自动重启虚拟机，使之生效。如果你不希望自动更新影响到你的在线运行，可以在部署完毕以后，选择自动下载更新但是手动安装。
<!--Update_Description: wording update-->