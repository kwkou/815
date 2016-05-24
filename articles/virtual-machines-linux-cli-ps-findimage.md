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

> [AZURE.NOTE]当你在本主题中搜索虚拟机映像时，你在 Azure 服务管理器模式下，使用最近安装的适用于 Mac、Linux 和 Windows 的 Azure 命令行接口，或者使用 Windows PowerShell。使用 Azure CLI 时，键入 `azure config mode asm` 即可进入该模式。使用 PowerShell 0.9 或之前版本时，请键入 `Switch-AzureMode AzureServiceManager`。

## 常用 Linux 映像表


| PublisherName | 产品 | Name |
|:---------------------------------|:-------------------------------------------|:---------------------------------|:--------------------|
| OpenLogic | CentOS | f1179221e23b4dbb89e39d70e5bc9e72__OpenLogic-CentOS-70-20150904 |
| OpenLogic | CentOS | f1179221e23b4dbb89e39d70e5bc9e72__OpenLogic-CentOS-71-20150731 |
| Canonical | UbuntuServer | b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-12_04_5-LTS-amd64-server-20150204-en-us-30GB |
| Canonical | UbuntuServer | b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04_3-LTS-amd64-server-20151019-en-us-30GB |


[AZURE.INCLUDE [virtual-machines-common-cli-ps-findimage](../includes/virtual-machines-common-cli-ps-findimage.md)]

<!---HONumber=Mooncake_0118_2016-->