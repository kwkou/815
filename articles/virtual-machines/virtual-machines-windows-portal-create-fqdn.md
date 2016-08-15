<!-- Ibiza portal: tested -->

<properties
   pageTitle="在 Azure 门户预览中为 VM 创建 FQDN | Azure"
   description="了解如何在 Azure 门户预览中为基于资源管理器的虚拟机创建完全限定域名或 FQDN。"
   services="virtual-machines-windows"
   documentationCenter=""
   authors="iainfoulds"
   manager="timlt"
   editor="tysonn"
   tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="06/07/2016"
	wacn.date="07/11/2016"/>

# 在 Azure 门户预览中创建完全限定的域名
在 [Azure 门户预览](https://portal.azure.cn)中使用 Resource Manager 部署模型创建虚拟机 (VM) 时，该门户会为虚拟机自动创建一个公共 IP 资源。可以使用此 IP 地址远程访问 VM。虽然默认情况下门户不创建[完全限定域名](https://en.wikipedia.org/wiki/Fully_qualified_domain_name)（简称 FQDN），但在创建 VM 以后，创建完全限定域名相当容易。本文演示了创建 DNS 名称或 FQDN 的步骤。

[AZURE.INCLUDE [virtual-machines-common-portal-create-fqdn](../includes/virtual-machines-common-portal-create-fqdn.md)]

你现在可以使用此 DNS 名称（例如远程桌面协议 (RDP)）远程连接至 VM。

## 后续步骤
你的 VM 已经有公共 IP 和 DNS 名称，现在可以部署通用应用程序框架或服务，例如 IIS、SQL、SharePoint 等等。

你也可以深入了解[使用 Resource Manager](/documentation/articles/resource-group-overview/)，以获取生成 Azure 部署的相关提示。
<!---HONumber=Mooncake_0704_2016-->