<properties
   pageTitle="导航和选择 Linux VM 映像 | Azure"
   description="了解在使用资源管理器部署模型创建 Azure 虚拟机时如何确定映像的确定发布者、产品和 SKU。"
   services="virtual-machines-linux"
   documentationCenter=""
   authors="squillace"
   manager="timlt"
   editor=""
   tags="azure-resource-manager"
   />

<tags
   ms.service="virtual-machines-linux"
   ms.date="03/14/2016"
   wacn.date="05/24/2016"/>

# 使用 Windows PowerShell 和 Azure CLI 来浏览和选择 Linux 虚拟机映像

[AZURE.INCLUDE [arm-api-version-both](../includes/arm-api-version-both.md)]

> [AZURE.NOTE] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是经典部署模型。

## 常用 Linux 映像表

| PublisherName | 产品 | SKU |
|:---------------------------------|:-------------------------------------------|:---------------------------------|:--------------------|
| SUSE                             | openSUSE                                   | 13.2                             |
| SUSE                             | SLES                                       | 12-SP1                           |
| OpenLogic                        | CentOS                                     | 7.1                              |
| Canonical                        | UbuntuServer                               | 14.04.3-LTS                      |

[AZURE.INCLUDE [virtual-machines-common-cli-ps-findimage](../includes/virtual-machines-common-cli-ps-findimage.md)]

<!---HONumber=Mooncake_0118_2016-->