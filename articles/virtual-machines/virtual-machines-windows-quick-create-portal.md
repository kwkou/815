<properties
    pageTitle="Azure 快速入门 - 创建 Windows VM 门户 | Azure"
    description="Azure 快速入门 - 创建 Windows VM 门户"
    services="virtual-machines-windows"
    documentationcenter="virtual-machines"
    author="neilpeterson"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="infrastructure"
    ms.date="03/20/2017"
    wacn.date="04/17/2017"
    ms.author="nepeters"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="2bd2a39f386626dce06735b540e97809388b7ab7"
    ms.lasthandoff="04/06/2017" />

# <a name="create-a-windows-virtual-machine-with-the-azure-portal-preview"></a>使用 Azure 门户预览创建 Windows 虚拟机

可以通过 Azure 门户预览创建 Azure 虚拟机。 此方法提供一个基于浏览器的用户界面，用于创建和配置 VM 和所有相关的 Azure 资源。

## <a name="log-in-to-azure"></a>登录到 Azure 

通过 http://portal.azure.cn 登录到 Azure 门户预览。

## <a name="create-virtual-machine"></a>创建虚拟机

2. 单击 Azure 门户预览左上角的“新建”按钮。

3. 从“新建”边栏选项卡中选择“计算”，从“计算”边栏选项卡中选择“Windows Server 2016 Datacenter”，然后单击“创建”按钮。

4. 填写虚拟机“基本信息”表单。 在此处输入的用户名和密码用于登录到虚拟机。 对于“资源组”，请创建一个新的资源组。 资源组是在其中创建并集中管理 Azure 资源的逻辑容器。 完成后，单击“确定”。

    ![在门户边栏选项卡中输入 VM 的基本信息](./media/virtual-machine-quick-start/create-windows-vm-portal-basic-blade.png)  

5. 为 VM 选择大小并单击“选择”。 

6. 设置保留默认值，然后单击“确定”。

7. 在摘要页上，单击“确定”以开始虚拟机部署。

8. 若要监视部署状态，请单击虚拟机。 可以在 Azure 门户预览仪表板上或者通过从左侧菜单中选择“虚拟机”来找到该 VM。 创建 VM 后，状态将从“正在部署”更改为“正在运行”。

## <a name="connect-to-virtual-machine"></a>连接到虚拟机

在部署完成后，创建到虚拟机的远程桌面连接。

1. 单击虚拟机边栏选项卡上的“连接”按钮。 此时会创建和下载远程桌面协议文件（.rdp 文件）。

    ![门户 9](./media/virtual-machine-quick-start/portal-quick-start-9.png) 

2. 若要连接到 VM，请打开下载的 RDP 文件。 出现提示时，请单击“连接”。 在 Mac 上，你需要一个 RDP 客户端，例如 Mac 应用商店提供的这个[远程桌面客户端](https://itunes.apple.com/us/app/microsoft-remote-desktop/id715768417?mt=12)。

3. 输入在创建虚拟机时指定的用户名和密码，然后单击“确定”。

4. 你可能会在登录过程中收到证书警告。 单击“是”或“继续”继续进行连接。

## <a name="delete-virtual-machine"></a>删除虚拟机

不再需要资源组、虚拟机和所有相关的资源时，可将其删除。 为此，请从虚拟机边栏选项卡中选择该资源组，然后单击“删除”。

## <a name="next-steps"></a>后续步骤

[安装角色和配置防火墙教程](/documentation/articles/virtual-machines-windows-hero-role/)

[浏览 VM 部署 CLI 示例](/documentation/articles/virtual-machines-windows-cli-samples/)