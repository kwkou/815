<properties
    pageTitle="密钥保管库 .NET 2.x API 发行说明 | Azure"
    description=".NET 开发人员可使用此 API 来编写 Azure 密钥保管库的代码"
    services="key-vault"
    author="BrucePerlerMS"
    manager="mbaldwin"
    editor="bruceper" />
<tags
    ms.assetid="1cccf21b-5be9-4a49-8145-483b695124ba"
    ms.service="key-vault"
    ms.devlang="CSharp"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="05/02/2017"
    wacn.date="06/12/2017"
    ms.author="bruceper"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="4927517ca2b6b944e2c6c8dbf3b39470ff6b57c6"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="azure-key-vault-net-20---release-notes-and-migration-guide"></a>Azure 密钥保管库 .NET 2.0 - 发行说明和迁移指南
以下说明和指南适用于使用 Azure 密钥保管库 .NET/C# 库的开发人员。 在从 1.0 版到 2.0 版的转换中，进行了大量需要代码迁移的更新，使你可以受益于功能的改进和新增，例如 **Key Vault 证书**支持。

## <a name="key-vault-certificates"></a>密钥保管库证书

密钥保管库证书支持适用于 x509 证书管理，它提供以下行为：  

- 允许证书所有者通过密钥保管库创建过程或通过导入现有证书来创建证书。 这包括自签名证书和证书颁发机构生成的证书。
- 允许密钥保管库证书所有者在不与私钥材料交互的情况下实现 X509 证书的安全存储和管理。  
- 允许证书所有者创建策略来指示密钥保管库如何管理证书的生命周期。  
- 允许证书所有者提供联系信息用于接收有关证书过期和续订生命周期事件的通知。  
- 支持在选定的颁发者（密钥保管库合作伙伴 X509 证书提供者/证书颁发机构）处自动续订证书。
  
  - 注意 - 也允许在未建立合作关系的提供者/颁发机构那里获取证书，但这些机构不支持自动续订功能。

## <a name="net-support"></a>.NET 支持

- Azure 密钥保管库 .NET/C# 库 2.0 版不支持 **.NET 4.0**
- Azure 密钥保管库 .NET/C# 库 2.0 版支持 **.NET Core**

## <a name="namespaces"></a>命名空间

- **模型**的命名空间从 **Microsoft.Azure.KeyVault** 更改为 **Microsoft.Azure.KeyVault.Models**。
- **Microsoft.Azure.KeyVault.Internal** 命名空间被弃用。
- Azure SDK 依赖项命名空间从 **Hyak.Common** 和 **Hyak.Common.Internals** 更改为 **Microsoft.Rest** 和 **Microsoft.Rest.Serialization**

## <a name="type-changes"></a>类型更改

- *Secret* 更改为 *SecretBundle*
- *Dictionary* 更改为 *IDictionary*
- *List<T>、string []* 更改为 *IList<T>*
- *NextList* 更改为 *NextPageLink*

## <a name="return-types"></a>返回类型

- **KeyList** 和 **SecretList** 将返回 *IPage<T>* 而不是 *ListKeysResponseMessage*
- 生成的 **BackupKeyAsync** 将返回 *BackupKeyResult*，其中包含*值*（备份 blob）。 以前，该方法经过包装，并且只返回值。

## <a name="exceptions"></a>异常

- *KeyVaultClientException* 更改为 *KeyVaultErrorException*
- 服务错误从 *exception.Error* 更改为 *exception.Body.Error.Message*。
- 从 **[JsonExtensionData]** 的错误消息中删除了其他信息。

## <a name="constructors"></a>构造函数

- 构造函数不接受 *HttpClient* 作为构造函数参数，只接受 *HttpClientHandler* 或 *DelegatingHandler[]*。

## <a name="downloaded-packages"></a>下载的包

当客户端处理密钥保管库的依赖项时，会已下载以下内容

### <a name="previous-package-list"></a>以前的包列表

- package id="Hyak.Common" version="1.0.2" targetFramework="net45"
- package id="Microsoft.Azure.Common" version="2.0.4" targetFramework="net45"
- package id="Microsoft.Azure.Common.Dependencies" version="1.0.0" targetFramework="net45"
- package id="Microsoft.Azure.KeyVault" version="1.0.0" targetFramework="net45"
- package id="Microsoft.Bcl" version="1.1.9" targetFramework="net45"
- package id="Microsoft.Bcl.Async" version="1.0.168" targetFramework="net45"
- package id="Microsoft.Bcl.Build" version="1.0.14" targetFramework="net45"
- package id="Microsoft.Net.Http" version="2.2.22" targetFramework="net45"

### <a name="current-package-list"></a>当前的包列表

- package id="Microsoft.Azure.KeyVault" version="2.0.0-preview" targetFramework="net45"
- package id="Microsoft.Rest.ClientRuntime" version="2.2.0" targetFramework="net45"
- package id="Microsoft.Rest.ClientRuntime.Azure" version="3.2.0" targetFramework="net45"

## <a name="class-changes"></a>类更改

- **UnixEpoch** 类已删除
- **Base64UrlConverter** 类已重命名为 **Base64UrlJsonConverter**

## <a name="other-changes"></a>其他更改

- 在此版本的 API 中，添加了针对暂时性故障配置 KV 操作重试策略的支持。

## <a name="microsoftazuremanagementkeyvault-nuget"></a>Microsoft.Azure.Management.KeyVault NuGet

- 对于返回 *保管库*的操作，返回类型是包含 Vault 属性的类。 返回类型现在为 *Vault*。
- *PermissionsToKeys* 和 *PermissionsToSecrets* 现在是 *Permissions.Keys* 和 *Permissions.Secrets*
- 某些返回类型更改同样适用于控制平面。

## <a name="microsoftazurekeyvaultextensions-nuget"></a>Microsoft.Azure.KeyVault.Extensions NuGet

- 该包已分解为 **Microsoft.Azure.KeyVault.Extensions** 和用于加密操作的 **Microsoft.Azure.KeyVault.Cryptography**。

<!--Update_Description: wording update-->