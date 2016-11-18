<properties
   pageTitle="Azure 存储安全概述 | Microsoft Azure"
   description=" Azure 存储空间是依赖于持续性、可用性和可缩放性来满足客户需求的现代应用程序的云存储解决方案。本文提供可与 Azure 存储配合使用的核心 Azure 安全功能概述。"
   services="security"
   documentationCenter="na"
   authors="TerryLanfear"
   manager="MBaldwin"
   editor="TomSh"/>  


<tags
   ms.service="security"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="09/16/2016"
   wacn.date="10/31/2016"
   ms.author="terrylan"/>  


# Azure 存储安全概述

Azure 存储空间是依赖于持续性、可用性和可缩放性来满足客户需求的现代应用程序的云存储解决方案。Azure 存储提供配套的安全功能：

- 存储帐户可以通过基于角色的访问控制和 Azure Active Directory 来保护。
- 在应用程序和 Azure 之间传输数据时，可以使用客户端加密、HTTPS 或 SMB 3.0 来保护数据。
- 使用存储服务加密写入 Azure 存储时，可将数据设置为自动加密。

- 可以使用共享访问签名来授予对 Azure 存储中数据对象的委派访问权限。
- 可以使用存储分析来跟踪某人访问存储时使用的身份验证方法。

有关 Azure 存储中安全性的详细信息，请参阅 [Azure Storage security guide](/documentation/articles/storage-security-guide/)（Azure 存储安全指南）。本指南深入探讨 Azure 存储的安全功能，例如存储帐户密钥、传输中数据的静态加密，以及存储分析。

本文提供可与 Azure 存储配合使用的 Azure 安全功能概述。此外，提供了有关每项功能详细信息的文章链接。

下面是本文介绍的核心功能：

- 基于角色的访问控制
- 存储对象的委托访问权限
- 传输中加密
- 静态加密/存储服务加密
- Azure 磁盘加密
- Azure 密钥保管库

## 基于角色的访问控制 (RBAC)

可以使用基于角色的访问控制 (RBAC) 来保护存储帐户。对于想要实施数据访问安全策略的组织而言，必须根据[需要知道](https://en.wikipedia.org/wiki/Need_to_know)和[最低权限](https://en.wikipedia.org/wiki/Principle_of_least_privilege)安全策略限制访问权限。这些访问权限是通过将相应的 RBAC 角色分配给特定范围内的组和应用程序来授予的。可以使用[内置 RBAC 角色](/documentation/articles/role-based-access-built-in-roles/)（例如存储帐户参与者）将权限分配给用户。

了解详细信息：

- [Azure Active Directory Role-based Access Control（Azure Active Directory 基于角色的访问控制）](/documentation/articles/role-based-access-control-configure/)

## 存储对象的委托访问权限

共享访问签名 (SAS) 用于对存储帐户中的资源进行委托访问。使用 SAS 这意味着可以授权客户端在指定时间段内，以一组指定权限有限地访问你的存储帐户中的对象。可以授予这些有限的权限，而不必共享帐户访问密钥。若要使用 SAS 访问存储资源，客户端只需将 SAS 传入到相应的构造函数或方法。

了解详细信息：

- [了解 SAS 模型](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)
- [创建 SAS 并将其用于 Blob 存储](/documentation/articles/storage-dotnet-shared-access-signature-part-2/)

## 传输中加密
传输中加密是通过网络传输数据时用于保护数据的机制。在 Azure 存储中，可以使用以下功能保护数据：

- [传输级别加密](/documentation/articles/storage-security-guide/#encryption-in-transit)，例如从 Azure 存储传入或传出数据时使用的 HTTPS。
- [线路加密](/documentation/articles/storage-security-guide/)，例如 Azure 文件共享的 SMB 3.0 加密。
- [客户端加密](/documentation/articles/storage-security-guide/)，在将数据传输到存储之前加密数据，以及从存储传出数据后解密数据。

了解有关客户端加密的详细信息：

- [适用于 Microsoft Azure 存储的客户端加密](https://blogs.msdn.microsoft.com/windowsazurestorage/2015/04/28/client-side-encryption-for-microsoft-azure-storage-preview/)
- [云安全控件系列：加密传输中的数据](http://blogs.microsoft.com/cybertrust/2015/08/10/cloud-security-controls-series-encrypting-data-in-transit/)

## 静态加密

对许多组织而言，[静态数据加密](https://blogs.microsoft.com/cybertrust/2015/09/10/cloud-security-controls-series-encrypting-data-at-rest/)是实现数据隐私性、合规性和数据所有权的必要措施。有三项 Azure 功能可提供“静态”数据加密：

- [存储服务加密](/documentation/articles/storage-security-guide/#encryption-at-rest)可以请求存储服务在将数据写入 Azure 存储时自动加密数据。
- [客户端加密](/documentation/articles/storage-security-guide/)也提供静态加密功能。
- [Azure 磁盘加密](/documentation/articles/storage-security-guide/)允许加密 IaaS 虚拟机使用的 OS 磁盘和数据磁盘。

了解有关存储服务加密的详细信息：

- [Azure 存储服务加密](/home/features/storage/)适用于 [Azure Blob 存储](/documentation/articles/storage-introduction/)。有关其他 Azure 存储类型的详细信息，请参阅[文件](/documentation/articles/storage-dotnet-how-to-use-files/)、[表](/documentation/articles/storage-dotnet-how-to-use-files/)和[队列](/documentation/articles/storage-dotnet-how-to-use-queues/)。
- [静态数据的 Azure 存储空间服务加密](/documentation/articles/storage-service-encryption/)

## Azure 磁盘加密

适用于虚拟机 (VM) 的 Azure 磁盘加密通过使用 [Azure 密钥保管库](/home/features/key-vault/)中控制的密钥和策略加密你的 VM 磁盘（包括引导磁盘和数据磁盘），帮助解决企业的安全和合规性要求。

适用于 VM 的磁盘加密可用于 Linux 与 Windows 操作系统。它也使用密钥保管库帮助保护、管理和审核磁盘加密密钥的使用。在虚拟机休息时使用 Azure 存储帐户的行业标准加密技术对 VM 磁盘中的所有数据进行加密。适用于 Windows 的磁盘加密解决方案是基于 [Microsoft BitLocker 驱动器加密](https://technet.microsoft.com/zh-cn/library/cc732774.aspx)技术，Linux 解决方案基于 [dm-crypt](https://en.wikipedia.org/wiki/Dm-crypt)。

了解详细信息：

- [Azure Disk Encryption for Windows and Linux IaaS Virtual Machines（适用于 Windows 和 Linux IaaS 虚拟机的 Azure 磁盘加密）](https://gallery.technet.microsoft.com/Azure-Disk-Encryption-for-a0018eb0)

## Azure 密钥保管库

Azure 磁盘加密使用 [Azure 密钥保管库](/home/features/key-vault/)来帮助控制和管理密钥保管库订阅中的磁盘加密密钥和机密，同时确保虚拟机磁盘中的所有数据可在 Azure 存储中静态加密。应使用密钥保管库来审核密钥和策略用法。

了解详细信息：

- [什么是 Azure 密钥保管库？](/documentation/articles/key-vault-whatis/)
- [Azure 密钥保管库入门](/documentation/articles/key-vault-get-started/)

<!---HONumber=Mooncake_1024_2016-->