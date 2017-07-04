<properties
    pageTitle="如何标记 Azure Linux 虚拟机 | Azure"
    description="了解如何标记使用 Resource Manager 部署模型在 Azure 中创建的 Azure Linux 虚拟机。"
    services="virtual-machines-linux"
    documentationcenter=""
    author="mmccrory"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="ca0e17e5-d78e-42e6-9dad-c1e8f1c58027"
    ms.service="virtual-machines-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure-services"
    ms.date="02/28/2017"
    wacn.date="04/24/2017"
    ms.author="memccror"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="7a436bd332accc3d7a594a7a7e24ef6079a92fdc"
    ms.lasthandoff="04/14/2017" />

# <a name="how-to-tag-a-linux-virtual-machine-in-azure"></a>如何在 Azure 中标记 Linux 虚拟机
本文介绍在 Azure 中通过 Resource Manager 部署模型标记 Linux 虚拟机的不同方式。 标记是用户定义的键/值对，可直接放置在资源或资源组中。 针对每个资源和资源组，Azure 当前支持最多 15 个标记。 标记可以在创建时放置在资源中或添加到现有资源中。 请注意，只有通过 Resource Manager 部署模型创建的资源支持标记。

[AZURE.INCLUDE [virtual-machines-common-tag](../../includes/virtual-machines-common-tag.md)]

## <a name="tagging-with-azure-cli"></a>使用 Azure CLI 进行标记
首先需要安装最新的 [Azure CLI 2.0（预览版）](https://docs.microsoft.com/zh-cn/cli/azure/install-az-cli2)，并使用 [az login](https://docs.microsoft.com/zh-cn/cli/azure/#login) 登录到 Azure 帐户。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

还可以使用 [Azure CLI 1.0](/documentation/articles/virtual-machines-linux-tag-nodejs/) 执行这些步骤。

你可以使用此命令查看给定虚拟机的所有属性，包括标记：

        az vm show --resource-group MyResourceGroup --name MyTestVM

若要通过 Azure CLI 添加新的 VM 标记，可以使用 `azure vm update` 命令以及标记参数 **--set**：

        az vm update --resource-group MyResourceGroup --name MyTestVM --set tags.myNewTagName1=myNewTagValue1 tags.myNewTagName2=myNewTagValue2

若要删除标记，可以在 `azure vm update` 命令中使用 **--remove** 参数。

        az vm update --resource-group MyResourceGroup --name MyTestVM --remove tags.myNewTagName1

既然我们已通过 Azure CLI 和门户将标记应用到资源中，那就让我们看一看使用情况详细信息，以在计费门户中的查看标记。

[AZURE.INCLUDE [virtual-machines-common-tag-usage](../../includes/virtual-machines-common-tag-usage.md)]

## <a name="next-steps"></a>后续步骤
* 若要详细了解如何标记 Azure 资源，请参阅 [Azure Resource Manager 概述][Azure Resource Manager Overview]和[使用标记来组织 Azure 资源][Using Tags to organize your Azure Resources]。
* 若要了解标记如何帮助你管理 Azure 资源的使用，请参阅[了解 Azure 帐单][Understanding your Azure Bill]。

[Azure CLI environment]: /documentation/articles/xplat-cli-azure-resource-manager/
[Azure Resource Manager Overview]: /documentation/articles/resource-group-overview/
[Using Tags to organize your Azure Resources]: /documentation/articles/resource-group-using-tags/
[Understanding your Azure Bill]: /documentation/articles/billing-understand-your-bill/