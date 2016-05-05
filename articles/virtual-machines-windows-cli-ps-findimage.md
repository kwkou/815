<properties
   pageTitle="导航和选择 Windows VM 映像 | Azure"
   description="了解在使用资源管理器部署模型创建 Windows 虚拟机时如何确定映像的确定发布者、产品和 SKU。"
   services="virtual-machines-windows"
   documentationCenter=""
   authors="squillace"
   manager="timlt"
   editor=""
   tags="azure-resource-manager"
   />

<tags
   ms.service="virtual-machines-windows"
   ms.date="12/08/2015"
   wacn.date="02/17/2016"/>

# 使用 Windows PowerShell 和 Azure CLI 来浏览和选择 Windows 虚拟机映像

> [AZURE.NOTE]当你在本主题中搜索虚拟机映像时，你在 Azure 服务管理器模式下，使用最近安装的适用于 Mac、Linux 和 Windows 的 Azure 命令行接口，或者使用 Windows PowerShell。使用 Azure CLI 时，键入 `azure config mode asm` 即可进入该模式。使用 PowerShell 0.9 或之前版本时，请键入 `Switch-AzureMode AzureServiceManager`。

## 常用映像表


| PublisherName | 产品 | Name |
|:---------------------------------|:-------------------------------------------|:---------------------------------|:--------------------|
| Microsoft SharePoint Group | MicrosoftSharePointServer | 9619bdbee1584b6f80d684565a6eeb74__SharePoint-2013-Trial-3-26-2014 |
| Microsoft SQL Server Group | SQL2014-WS2012R2 | 74bb2f0b8dcc47fbb2914b60ed940c35__SQL-Server-2014-RTM-12.0.2361.0-DataWarehousing-CHS-Win2012R2-cy14su05 |
| Microsoft SQL Server Group | SQL2014-WS2012R2 | 74bb2f0b8dcc47fbb2914b60ed940c35__SQL-Server-2014-RTM-12.0.2361.0-Web-CHS-Win2012R2-cy14su05 |
| Microsoft | WindowsServer | 55bc2b193643443bb879a78bda516fc8__Windows-Server-2012-Datacenter-20151021-zh.cn-127GB.vhd |
| Microsoft | WindowsServer | 55bc2b193643443bb879a78bda516fc8__Windows-Server-2012-R2-20151021-zh.cn-127GB.vhd |
| Microsoft | WindowsServer | 55bc2b193643443bb879a78bda516fc8__Win2K8R2SP1-Datacenter-20151021-zh.cn-127GB.vhd |
| Microsoft | WindowsServer | 55bc2b193643443bb879a78bda516fc8__Windows-Server-Technical-Preview-20151118-en.us-127GB.vhd |
| Microsoft Windows Server Essentials Group | WindowsServerEssentials | 0c5c79005aae478e8883bf950a861ce0__Windows-Server-2012-Essentials-20141204-zhcn |
| Microsoft Windows Server HPC Pack team | WindowsServerHPCPack | 86f94d8ddf9d4aad9b064efb793ff4c2__HPC-Pack-2012R2-Update1-4.3.4660.0-WS2012R2-CHN |


[AZURE.INCLUDE [virtual-machines-common-cli-ps-findimage](../includes/virtual-machines-common-cli-ps-findimage.md)]

<!---HONumber=Mooncake_0118_2016-->