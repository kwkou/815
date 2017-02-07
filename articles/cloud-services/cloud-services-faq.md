<properties
	pageTitle="云服务常见问题 | Azure"
	description="有关云服务的常见问题。"
	services="cloud-services"
	documentationCenter=""
	authors="Thraka"
	manager="timlt"
	editor=""/>  


<tags
	ms.service="cloud-services"
	ms.workload="tbd"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="11/16/2016"
	wacn.date="12/05/2016"
	ms.author="adegeo"/>  


# 云服务常见问题
本文回答了一些关于 Azure 云服务的常见问题。你还可以访问 [Azure 支持 FAQ](http://go.microsoft.com/fwlink/?LinkID=185083) 了解一般的 Azure 定价和支持信息。还可以参阅[云服务 VM 大小页面](/documentation/articles/cloud-services-sizes-specs/)，了解大小信息。

## <a name="certificates"></a> 证书

### 应该在何处安装我的证书？

- **My**  
 具有私钥的应用程序证书（*.pfx、*.p12）。

- **CA**  
所有中间证书都会放入此存储（策略和子 CA）。

- **ROOT**  
根 CA 存储，因此应将主要的根 CA 证书放在此处。

### 无法删除过期的证书

Azure 会阻止删除正在使用的证书。需要删除使用该证书的部署，或使用其他证书或续订证书来更新部署。

### 删除过期的证书

只要未在使用证书，就可以使用 [Remove-AzureCertificate](https://msdn.microsoft.com/zh-cn/library/azure/mt589145.aspx) PowerShell cmdlet 删除该证书。

### 我拥有名为 Windows Azure Service Management for Extensions 的已过期证书

每次将扩展（例如远程桌面扩展）添加到云服务中时，就会创建这些证书。这些证书仅用于加密和解密扩展的专用配置。这些证书是否过期并不重要。系统不会检查过期日期。

### 已删除的证书一直重复出现

这些证书很可能是因为正在使用的某个工具而一直重复出现，例如 Visual Studio。每次重新连接使用证书的工具时，证书将再次上传到 Azure。

### 我的证书一直消失

虚拟机实例回收时，所有本地更改都将丢失。使用[启动任务](/documentation/articles/cloud-services-startup-tasks/)在每次角色启动时将证书安装到虚拟机。

### 我在门户中找不到管理证书

[管理证书](/documentation/articles/azure-api-management-certs/)仅在 Azure 经典经管门户中提供。
### 如何禁用管理证书？

[管理证书](/documentation/articles/azure-api-management-certs/)无法禁用。不想再使用它们时，可以通过 Azure 经典管理门户进行删除。

### 如何为特定 IP 地址创建 SSL 证书？
按照[创建证书教程](/documentation/articles/cloud-services-certs-create/)中的说明操作。使用 IP 地址作为 DNS 名称。

## “安全”
### 禁用 SSL 3.0
若要禁用 SSL 3.0 并使用 TLS 安全性，请创建此[博客](https://azure.microsoft.com/zh-CN/blog/how-to-disable-ssl-3-0-in-azure-websites-roles-and-virtual-machines/)文章中介绍的启动任务。

## 扩展云服务
### 我不能缩放超过 X 个实例
Azure 订阅对可以使用的内核数存在限制。如果已使用所有可用的内核，缩放无法执行。例如，如果有 100 个内核的限制，这意味着云服务可以有 100 个 A1 大小的虚拟机实例或 50 个 A2 大小的虚拟机实例。

## 故障排除

### 无法在多 VIP 云服务中保留 IP

首先，请确保已打开想要为其保留 IP 的虚拟机实例。其次，请确保为过渡和生产部署使用保留 IP。**请勿**在部署升级过程中更改设置。

<!---HONumber=Mooncake_1128_2016-->