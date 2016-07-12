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
	ms.date="04/19/2016"
	wacn.date="05/31/2016"/>

# Azure 云服务证书概述
证书在 Azure 中用于云服务（[服务证书](#what-are-service-certificates)）以及用于通过管理 API 进行身份验证（[管理证书](#what-are-management-certificates)，适用于使用 Azure 管理门户而不是 ARM 的场合）。本主题同时提供了有关这两种证书类型的一般概述、如何[创建](#create)以及将其[部署](#deploy)到 Azure。

在 Azure 中使用的证书是 x.509 v3 证书，且可由另一个受信任的证书进行签名或可进行自签名。自签名的证书由其自己的创建者进行签名，因此，默认情况下不受信任。大多数浏览器可以忽略这一点。自签名的证书仅应由自己在开发和测试云服务时使用。

Azure 使用的证书可以包含一个私钥或公钥。证书具有指纹，它提供了一种可对证书进行明确识别的方法。该指纹用于在 Azure [配置文件](/documentation/articles/cloud-services-configure-ssl-certificate/)中识别云服务应使用的证书。

## 什么是服务证书？
服务证书被附加到云服务，可实现与服务之间的安全通信。例如，如果你部署了 Web 角色，将需要提供可对公开的 HTTPS 终结点进行身份验证的证书。在你的服务定义中定义的服务证书会自动部署到运行你的角色实例的虚拟机。

你可以使用 Azure 管理门户或服务管理 API 将服务证书上载到 Azure 管理门户。服务证书与特定的云服务相关联，并分配到服务定义文件中的部署。

可将服务证书和你的服务分开管理，并可由不同的个人进行管理。例如，一名开发人员可以上载服务包，该服务包引用 IT 管理员以前上载到 Azure 的证书。IT 管理员可以管理并续订更改服务配置的证书而无需上载新的服务包。这样之所以可行，是因为证书的逻辑名称及其存储名称和位置是在服务定义文件中指定的，而证书指纹是在服务配置文件中指定的。若要更新证书，只需上载新证书并更改服务配置文件中的指纹值。

## 什么是管理证书？
管理证书允许你使用 Azure 管理门户提供的服务管理 API 进行身份验证。许多程序和工具（如 Visual Studio 或 Azure SDK）将使用这些证书来自动配置和部署各种 Azure 服务。这些并不真正与云服务相关。

>[AZURE.WARNING] 请小心！ 这些类型的证书允许任何使用它们进行身份验证的人管理与它们相关联的订阅。

### 限制
每个订阅限最多可具有 100 个管理证书。特定服务管理员的用户 ID 下的所有订阅同样最多只能具有 100 个管理证书。如果帐户管理员的用户 ID 已用于添加 100 个管理证书且需要更多证书，可以添加共同管理员以添加额外的证书。

在添加 100 个以上证书之前，请检查你是否可重用现有证书。使用共同管理员将可能会给你的证书管理流程增加不必要的复杂性。


<a name="create"></a>
## 创建新的自签名证书
可以使用任何可用工具创建自签名的证书，只要它们符合这些设置：

* X.509 证书。
* 包含私钥。
* 为密钥交换（.pfx 文件）而创建。
* 使用者名称必须与用于访问云服务的域匹配。
    > 你无法获取 chinacloudapp.cn 域（或与 Azure 相关的任何域）的 SSL 证书；该证书的使用者名称必须与用于访问应用程序的自定义域名匹配。例如，**contoso.net**，而不是 **contoso.chinacloudapp.cn**。
* 至少为 2048 位加密。
* **仅服务证书**：客户端证书必须驻留在个人证书存储区。

有两种简单的方法可在 Windows 上创建证书，即使用 `makecert.exe` 实用程序或 IIS。

### Makecert.exe

此实用程序随 Visual Studio 2013/2015 一并安装。它是一个控制台实用程序，可允许你创建和安装证书。如果你启动在安装 Visual Studio 时创建的 **VS2015 开发人员命令提示符**快捷方式，将出现命令提示符，提示在路径中加入此工具。

    makecert -sky exchange -r -n "CN=[CertificateName]" -pe -a sha1 -len 2048 -ss My -sv [CertificateName].pvk [CertificateName].cer


### Internet 信息服务 (IIS)

在 internet 上有许多页面，包含了有关如何使用 IIS 实现此操作的信息。[此处](https://www.sslshopper.com/article-how-to-create-a-self-signed-certificate-in-iis-7.html)就是一个很棒的页面，我认为其说明很不错。

### Java
你可以使用 Java [创建证书](/documentation/articles/java-create-azure-website-using-java-sdk/#create-a-certificate)。

### Linux
[本文](/documentation/articles/virtual-machines-linux-ssh-from-linux/)介绍如何通过 SSH 创建证书。

## 后续步骤

[上载服务证书到 Azure 管理门户](/documentation/articles/cloud-services-configure-ssl-certificate/)。

将[管理 API 证书](/documentation/articles/azure-api-management-certs/)上载到 Azure 管理门户。

<!---HONumber=Mooncake_0523_2016-->