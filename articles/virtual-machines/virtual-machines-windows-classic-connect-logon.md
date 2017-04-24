<properties
    pageTitle="登录到经典 Azure VM | Azure"
    description="使用 Azure 经典管理门户登录到使用经典部署模型创建的 Windows 虚拟机。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="cynthn"
    manager="timlt"
    editor="tysonn"
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="3c1239ed-07dc-48b8-8b3d-dc8c5f0ab20e"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/17/2017"
    wacn.date="04/24/2017"
    ms.author="cynthn"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="a3b4cdd03d3c9771017087134631eff4f4f41f4c"
    ms.lasthandoff="04/14/2017" />

# <a name="log-on-to-a-windows-virtual-machine-using-the-azure-portal-preview"></a>使用 Azure 门户预览登录到 Windows 虚拟机
在 Azure 门户预览中，使用“连接”按钮启动远程桌面会话，然后登录到 Windows VM。

是否要连接到 Linux VM？ 请参阅[如何登录到运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)。

<!--
Deleting, but not 100% sure
Learn how to [perform these steps using new Azure portal preview](/documentation/articles/virtual-machines-windows-connect-logon/).
-->

> [AZURE.IMPORTANT]
> Azure 提供两个不同的部署模型用于创建和处理资源：[Resource Manager 和经典模型](/documentation/articles/resource-manager-deployment-model/)。 本文介绍如何使用经典部署模型。 Azure 建议大多数新部署使用 Resource Manager 模型。 有关如何使用 Resource Manager 模型登录到 VM 的信息，请参阅[此处](/documentation/articles/virtual-machines-windows-connect-logon/)。

## <a name="connect-to-the-virtual-machine"></a>连接到虚拟机
1. 登录 Azure 门户预览。
2. 单击要访问的虚拟机。 其名称在“所有资源”窗格中列出。

    ![Virtual-machine-locations](./media/virtual-machines-windows-classic-connect-logon/azureportaldashboard.png)

3. 单击虚拟机仪表板顶部的命令栏上的“连接”。

    ![虚拟机的连接图标](./media/virtual-machines-windows-classic-connect-logon/virtualmachine_dashboard_connect.png)

## <a name="log-on-to-the-virtual-machine"></a>登录到虚拟机
[AZURE.INCLUDE [virtual-machines-log-on-win-server](../../includes/virtual-machines-log-on-win-server.md)]

## <a name="next-steps"></a>后续步骤
* 如果“连接”按钮处于非活动状态或者在使用远程桌面连接时遇到其他问题，请尝试重置配置。 在虚拟机仪表板中单击“重置远程访问”。

    ![Reset-remote-access](./media/virtual-machines-windows-classic-connect-logon/virtualmachine_dashboard_reset_remote_access.png)

* 如果密码出现问题，可尝试重置密码。 在虚拟机仪表板左侧，“支持 + 故障排除”下面单击“重置密码”。

    ![Reset-password](./media/virtual-machines-windows-classic-connect-logon/virtualmachine_dashboard_reset_password.png)

如果这些提示不起作用或不是所需的内容，请参阅[对与基于 Windows 的 Azure 虚拟机的远程桌面连接进行故障排除](/documentation/articles/virtual-machines-windows-troubleshoot-rdp-connection/)。 此文将指导你完成诊断和解决常见问题。
<!--Update_Description: change to the new portal-->