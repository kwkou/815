<properties
    pageTitle="在 Azure 中重新部署 Linux 虚拟机 | Azure"
    description="如何通过在 Azure 中重新部署 Linux 虚拟机来解决 SSH 连接问题。"
    services="virtual-machines-linux"
    documentationcenter="virtual-machines"
    author="iainfoulds"
    manager="timlt"
    tags="azure-resource-manager,top-support-issue" />
<tags
    ms.assetid="e9530dd6-f5b0-4160-b36b-d75151d99eb7"
    ms.service="virtual-machines-linux"
    ms.devlang="na"
    ms.topic="support-article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="12/16/2016"
    wacn.date="03/20/2017"
    ms.author="iainfou" />  


# 将 Linux 虚拟机重新部署到新的 Azure 节点
如果在排查 SSH 或应用程序对 Azure 中的 Linux 虚拟机 (VM) 的访问问题时遇到困难，重新部署 VM 可能会有帮助。重新部署 VM 时，将 VM 移到 Azure 基础结构中的新节点，然后重新打开它，同时保留所有配置选项和关联的资源。本文介绍如何使用 Azure CLI 或 Azure 门户预览重新部署 VM。

> [AZURE.NOTE]
重新部署 VM 后，临时磁盘将丢失，与虚拟网络接口关联的动态 IP 地址将更新。

可以使用以下选项之一重新部署 VM。只需选择一个选项即可重新部署 VM：

- [Azure CLI 1.0](#azure-cli-10)
- [Azure 门户预览](#using-azure-portal)

## <a name="azure-cli-10"></a> Azure CLI 1.0
安装[最新版 Azure CLI 1.0](/documentation/articles/xplat-cli-install/)，登录到 Azure 帐户，并确保使用 Resource Manager 模式 (`azure config mode arm`)。

以下示例重新部署名为 `myResourceGroup` 的资源组中名为 `myVM` 的 VM：

    azure vm redeploy --resource-group myResourceGroup --vm-name myVM 

[AZURE.INCLUDE [virtual-machines-common-redeploy-to-new-node](../../includes/virtual-machines-common-redeploy-to-new-node.md)]

## 后续步骤
如果在连接 VM 时遇到问题，可以在[SSH 连接故障排除](/documentation/articles/virtual-machines-linux-troubleshoot-ssh-connection/)或[详细的 SSH 故障排除步骤](/documentation/articles/virtual-machines-linux-detailed-troubleshoot-ssh-connection/)中找到具体的帮助。如果无法访问在 VM 上运行的应用程序，还可以阅读 [应用程序故障排除问题](/documentation/articles/virtual-machines-linux-troubleshoot-app-connection/)。

<!---HONumber=Mooncake_0313_2017-->
<!--Update_Description: wording update-->