<properties 
	pageTitle="云服务和管理证书 | Azure" 
	description="了解如何在 Azure 中创建和使用证书" 
	services="cloud-services" 
	documentationCenter=".net" 
	authors="Thraka" 
	manager="timlt" 
	editor=""/>

<tags 
	ms.service="cloud-services" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="12/20/2016"
	wacn.date="01/25/2017"
	ms.author="adegeo"/>

# Azure 云服务证书概述
在 Azure 中，使用云服务需要证书（[服务证书](#what-are-service-certificates)），通过管理 API 进行身份验证也需要证书（[管理证书](#what-are-management-certificates)，使用 Azure 经典管理门户时需要，使用非经典 Azure 门户时则不需要）。本主题扼要介绍了这两种类型的证书，以及在 Azure 中[创建](#create)和部署证书的方法。

Azure 中使用的证书是 x.509 v3 证书，可自签名或由另一个受信任的证书签名。自签名证书由其创建者签名，因此，默认情况下不受信任。大多数浏览器可以忽略此问题。自签名证书应仅在开发和测试云服务时使用。

Azure 使用的证书可以包含一个私钥或公钥。证书具有指纹，它提供了一种明确识别证书的方法。在 Azure [配置文件](/documentation/articles/cloud-services-configure-ssl-certificate/)中，通过指纹可识别云服务应使用的证书。

## <a name="what-are-service-certificates"></a> 什么是服务证书？
将服务证书添加到云服务后可在服务之间实现安全通信。例如，如果部署了 Web 角色，则需要提供可对公开 HTTPS 终结点进行身份验证的证书。在服务定义中定义的服务证书会自动部署到运行角色实例的虚拟机。

使用 Azure 经典管理门户或经典部署模型可将服务证书上载到 Azure 经典管理门户。服务证书与特定的云服务相关联。它们将分配给服务定义文件中的部署。

服务证书可与服务分开管理，且可由不同的人员管理。例如，开发人员上载的服务包可以使用 IT 管理员以前上载到 Azure 的证书。IT 管理员可以管理和续订该证书（更改服务配置），而无需上载新的服务包。在没有新服务包的情况下更新之所以可能是因为证书的逻辑名称、存储名称和存储位置是在服务定义文件中指定的，而证书指纹则是在服务配置文件中指定的。若要更新证书，只需上载新证书并更改服务配置文件中的指纹值。

>[AZURE.NOTE]
[云服务常见问题解答](/documentation/articles/cloud-services-faq/#certificates)一文提供了一些有关证书的有用信息。

## <a name="what-are-management-certificates"></a> 什么是管理证书？
使用管理证书可以通过经典部署模型进行身份验证。许多程序和工具（如 Visual Studio 或 Azure SDK）使用这些证书自动配置和部署各种 Azure 服务。实际上，这些证书与云服务并无关系。

>[AZURE.WARNING] 请注意！ 这些类型的证书允许使用它们进行身份验证的任何人管理与其相关联的订阅。

### 限制
每个订阅最多只能有 100 个管理证书。特定服务管理员用户 ID 下的所有订阅同样最多只能有 100 个管理证书。如果帐户管理员的用户 ID 下已添加 100 个管理证书，但还需要更多证书，则可以添加共同管理员，从而增加额外的证书。

当添加的证书数量达到 100 个之后，请检查是否可重用现有证书，然后决定是否添加更多证书。添加共同管理员可能导致证书管理流程变得更复杂。


<a name="create"></a>
## 创建新的自签名证书
任何可用工具均可用于创建自签名证书，前提是符合以下设置：

* X.509 证书。
* 包含私钥。
* 为密钥交换（.pfx 文件）创建。
* 使用者名称必须与用于访问云服务的域匹配。
    > 你无法获取 chinacloudapp.cn 域（或与 Azure 相关的任何域）的 SSL 证书；该证书的使用者名称必须与用于访问应用程序的自定义域名匹配。例如，**contoso.net**，而不是 **contoso.chinacloudapp.cn**。
* 至少采用 2048 位加密。
* **仅服务证书**：客户端证书必须驻留在*个人*证书存储区。

在 Windows 上有两种简单方法可以创建证书，分别是使用 `makecert.exe` 实用程序或 IIS。

### Makecert.exe

此实用工具已被弃用，此处不再赘述。有关详细信息，请参阅[此 MSDN 文章](https://msdn.microsoft.com/zh-cn/library/windows/desktop/aa386968)。

### PowerShell


        $cert = New-SelfSignedCertificate -DnsName yourdomain.cloudapp.net -CertStoreLocation "cert:\LocalMachine\My"
        $password = ConvertTo-SecureString -String "your-password" -Force -AsPlainText
        Export-PfxCertificate -Cert $cert -FilePath ".\my-cert-file.pfx" -Password $password

>[AZURE.NOTE] 如果要将证书用于 IP 地址而不是域，请在 -DnsName 参数中使用 IP 地址。

如果要[在经典管理门户中使用此证书](/documentation/articles/azure-api-management-certs/)，请将其导出到 **.cer** 文件：


        Export-Certificate -Type CERT -Cert $cert -FilePath .\my-cert-file.cer


### Internet 信息服务 (IIS)

Internet 上有许多关于如何使用 IIS 实现此操作的信息。[此页面](https://www.sslshopper.com/article-how-to-create-a-self-signed-certificate-in-iis-7.html)就是示例之一，其阐述非常清楚。

### Java
你可以使用 Java [创建证书](/documentation/articles/java-create-azure-website-using-java-sdk/#create-a-certificate)。

### Linux
[本文](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)介绍如何通过 SSH 创建证书。

## 后续步骤

 - [上载服务证书到 Azure 经典管理门户](/documentation/articles/cloud-services-configure-ssl-certificate/)。

 - 将[管理 API 证书](/documentation/articles/azure-api-management-certs/)上载到 Azure 经典管理门户。

>[AZURE.NOTE] Azure 门户不使用管理证书访问 API，而是使用用户帐户。

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description:update wording-->