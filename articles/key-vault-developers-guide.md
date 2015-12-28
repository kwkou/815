<properties
   pageTitle="密钥保管库开发人员指南 | Windows Azure"
   description="开发人员可以使用 Azure 密钥保管库来管理 Windows Azure 环境中的加密密钥。"
   services="key-vault"
   documentationCenter=""
   authors="BrucePerlerMS"
   manager="mbaldwin"
   editor="bruceper" />
<tags
   ms.service="key-vault"
   ms.date="11/06/2015"
   wacn.date="12/28/2015" />

# Azure 密钥保管库开发人员指南

开发人员可以使用 Azure 密钥保管库来管理 Windows Azure 环境中的加密密钥。密钥保管库支持多种密钥类型和高价值客户密钥算法。此外，你还可以使用密钥保管库安全地存储机密，后者是大小受限且不含任何特定语义的八位字节对象。密钥保管库可以包含混合在一起的密钥和机密。独立管理对象类型的访问控制。

获得成功授权的用户可以执行以下操作：

- 使用 [Create](https://msdn.microsoft.com/zh-cn/library/azure/dn903634.aspx)、[Import](https://msdn.microsoft.com/zh-cn/library/azure/dn903626.aspx)、[Update](https://msdn.microsoft.com/zh-cn/library/azure/dn903616.aspx)、[Delete](https://msdn.microsoft.com/zh-cn/library/azure/dn903611.aspx) 和其他操作管理加密密钥

- 使用 [Get](https://msdn.microsoft.com/zh-cn/library/azure/dn903633.aspx)、[Update](https://msdn.microsoft.com/zh-cn/library/azure/dn986818.aspx、[Delete](https://msdn.microsoft.com/zh-cn/library/azure/dn903613.aspx) 和其他操作管理机密

- 将加密密钥用于 [Sign](https://msdn.microsoft.com/zh-cn/library/azure/dn878096.aspx)/[Verify](https://msdn.microsoft.com/zh-cn/library/azure/dn878082.aspx)、[WrapKey](https://msdn.microsoft.com/zh-cn/library/azure/dn878066.aspx)/[UnwrapKey](https://msdn.microsoft.com/zh-cn/library/azure/dn878079.aspx) 和 [Encrypt](https://msdn.microsoft.com/zh-cn/library/azure/dn878060.aspx)/[Decrypt](https://msdn.microsoft.com/zh-cn/library/azure/dn878097.aspx) 操作

使用 Azure Active Directory 对针对密钥保管库的操作进行身份验证和授权。

## 密钥保管库的编程

程序员的密钥保管库管理系统包含多个接口，并以 REST 作为基础。[密钥保管库 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn903609.aspx)

|[![.NET](./media/key-vault-developers-guide/net.png)](https://msdn.microsoft.com/zh-cn/library/azure/dn903301.aspx)|[![Node.js](./media/key-vault-developers-guide/nodejs.png)](http://azure.github.io/azure-sdk-for-node/azure-arm-keyvault/latest)
|:--:|:--:|
|[.NET](https://msdn.microsoft.com/zh-cn/library/azure/dn903301.aspx)|[Node.js](http://azure.github.io/azure-sdk-for-node/azure-arm-keyvault/latest)

## 管理密钥保管库

可以使用 REST、PowerShell 或 CLI 管理 Azure 密钥保管库容器（保管库），如以下文章中所述：

- [使用 REST 创建和管理密钥保管库](https://msdn.microsoft.com/zh-cn/library/azure/mt620024.aspx)
- [使用 PowerShell 创建和管理密钥保管库](/documentation/articles/key-vault-get-started)
- [使用 CLI 创建和管理密钥保管库](/documentation/articles/key-vault-manage-with-cli)




## 示例

- 此下载包含示例应用程序 HelloKeyVault 和 Azure Web 服务示例。[Azure 密钥保管库代码示例](http://www.microsoft.com/download/details.aspx?id=45343)
- 本教程帮助你了解如何从 Azure 中的 Web 应用程序使用 Azure 密钥保管库。[从 Web 应用程序使用 Azure 密钥保管库](/documentation/articles/key-vault-use-from-web-application)

## 支持库

- [Windows Azure 密钥保管库核心库](http://www.nuget.org/packages/Microsoft.Azure.KeyVault.Core/1.0.0)提供 IKey 和 IKeyResolver 接口，用于通过标识符查找密钥，以及使用密钥执行操作。

- [Windows Azure 密钥保管库扩展](http://www.nuget.org/packages/Microsoft.Azure.KeyVault.Extensions/1.0.0)为 Azure 密钥保管库提供扩展功能。

<!---HONumber=Mooncake_1207_2015-->