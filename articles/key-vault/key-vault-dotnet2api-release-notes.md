<properties
   pageTitle="密钥保管库 .NET 2.x API 发行说明 | Azure"
   description=".NET 开发人员可使用此 API 来编写 Azure 密钥保管库的代码"
   services="key-vault"
   documentationCenter=""
   authors="BrucePerlerMS"
   manager="mbaldwin"
   editor="bruceper" />  

<tags
   ms.service="key-vault"
   ms.devlang="CSharp"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="10/07/2016"
   ms.author="bruceper" 
   wacn.date="11/22/2016"/>  


# Azure 密钥保管库 .NET 2.0 - 发行说明和迁移指南

以下说明和指导面向使用 Azure 密钥保管库 .NET/C# 库的开发人员。在从版本 1.0 更改为版本 2.0 的过程中做出了大量的更新，开发人员需要在代码中完成一些迁移工作，才能享用功能改进和新增的功能，例如密钥保管库证书支持。

密钥保管库证书支持适用于 x509 证书管理，它提供以下行为：

-   允许证书所有者通过密钥保管库创建过程或通过导入现有证书来创建证书。这包括自签名证书和证书颁发机构生成的证书。

- 允许密钥保管库证书所有者在不与私钥材料交互的情况下实现 X509 证书的安全存储和管理。

-   允许证书所有者创建策略来指示密钥保管库如何管理证书的生命周期。

-   允许证书所有者提供联系信息用于接收有关证书过期和续订生命周期事件的通知。

-   支持在选定的颁发者（密钥保管库合作伙伴 X509 证书提供者/证书颁发机构）处自动续订证书。
    - 注意 - 也允许在未建立合作关系的提供者/颁发机构那里获取证书，但这些机构不支持自动续订功能。


## .NET 支持
- **.NET 4.0** 不受 Azure 密钥保管库 .NET/C# 库 2.0 版的支持
- **.NET Core** 受 Azure 密钥保管库 .NET/C# 库 2.0 版的支持

## 命名空间
- **模型**的命名空间已从 **Microsoft.Azure.KeyVault** 更改为 **Microsoft.Azure.KeyVault.Models**。
- **Microsoft.Azure.KeyVault.Internal** 命名空间被弃用。
- Azure SDK 依赖项命名空间已从 **Hyak.Common** 和 **Hyak.Common.Internals** 更改为 **Microsoft.Rest** 和 **Microsoft.Rest.Serialization**


## 类型更改
- *Secret* 已更改为 *SecretBundle*
- *Dictionary* 已更改为 *IDictionary*
- *List<T>, string * 已更改为 *IList<T>*
- *NextList* 已更改为 *NextPageLink*


## 返回类型
- **KeyList** 和 **SecretList** 将返回 *IPage<T>* 而不是 *ListKeysResponseMessage*
- 生成的 **BackupKeyAsync** 将返回包含*值*（备份 Blob）的 *BackupKeyResult*。以前，该方法经过包装，并且只返回值。

## 异常
- *KeyVaultClientException* 已更改为 *KeyVaultErrorException*
- 服务错误已从 *exception.Error* 更改为 *exception.Body.Error.Message*。
- 从 **[JsonExtensionData]** 的错误消息中删除了附加信息。

## 构造函数
- 构造函数仅接受 *HttpClientHandler* 或 *DelegatingHandler*，而不接受 *HttpClient* 作为构造函数的参数。



## 下载的包  
当客户端处理密钥保管库中的依赖项时，将下载以下包
#### 以前的包列表
- package id="Hyak.Common" version="1.0.2" targetFramework="net45"
- package id="Microsoft.Azure.Common" version="2.0.4" targetFramework="net45"
- package id="Microsoft.Azure.Common.Dependencies" version="1.0.0" targetFramework="net45"
- package id="Microsoft.Azure.KeyVault" version="1.0.0" targetFramework="net45"
- package id="Microsoft.Bcl" version="1.1.9" targetFramework="net45"
- package id="Microsoft.Bcl.Async" version="1.0.168" targetFramework="net45"
- package id="Microsoft.Bcl.Build" version="1.0.14" targetFramework="net45"
- package id="Microsoft.Net.Http" version="2.2.22" targetFramework="net45"

#### 当前的包列表
- package id="Microsoft.Azure.KeyVault" version="2.0.0-preview" targetFramework="net45"
- package id="Microsoft.Rest.ClientRuntime" version="2.2.0" targetFramework="net45"
- package id="Microsoft.Rest.ClientRuntime.Azure" version="3.2.0" targetFramework="net45"


## 类更改

- **UnixEpoch** 类已删除
- **Base64UrlConverter** 类已重命名为 **Base64UrlJsonConverter**

## 其他更改

- 在此版本的 API 中，添加了针对暂时性故障配置 KV 操作重试策略的支持。



## Microsoft.Azure.Management.KeyVault NuGet
- 对于返回*保管库*的操作，返回类型是包含 Vault 属性的类。返回类型现在为 *Vault*。
- *PermissionsToKeys* 和 *PermissionsToSecrets* 现在为 *Permissions.Keys* 和 *Permissions.Secrets*
- 某些返回类型更改同样适用于控制平面。

## Microsoft.Azure.KeyVault.Extensions NuGet
- 该包已分解为用于加密操作的 **Microsoft.Azure.KeyVault.Extensions** 和 **Microsoft.Azure.KeyVault.Cryptography**。

<!---HONumber=Mooncake_1114_2016-->
