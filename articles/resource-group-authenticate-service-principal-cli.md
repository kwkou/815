<properties
   pageTitle="使用 Azure CLI 创建服务主体 | Azure"
   description="描述如何使用 Azure CLI 创建 Active Directory 应用程序和服务主体，并通过基于角色的访问控制授予其对资源的访问权限。它演示如何使用密码或证书对应用程序进行身份验证。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="timlt"
   editor="tysonn"/>

<tags
   ms.service="azure-resource-manager"
   ms.date="07/19/2016"
   wacn.date="09/28/2016"/>

# 使用 Azure CLI 创建服务主体来访问资源

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/resource-group-authenticate-service-principal/)
- [Azure CLI](/documentation/articles/resource-group-authenticate-service-principal-cli/)
- [门户](/documentation/articles/resource-group-create-service-principal-portal/)

如果应用程序或脚本需要访问资源，则多数情况下不需使用用户的凭据即可运行此过程。可以将该用户的不同权限分配给此过程，但用户的工作职责可能会变化。与上述方法不同，也可以为应用程序创建一个标识，其中包括身份验证凭据和角色分配情况。应用程序在每次运行时以此标识登录。本主题介绍如何通过[适用于 Mac、Linux 和 Windows 的 Azure CLI](/documentation/articles/xplat-cli-install/) 为应用程序进行一切所需设置，使之能够使用自己的凭据和标识运行。

在本文中，用户将创建两个对象 - Active Directory (AD) 应用程序和服务主体。AD 应用程序包含凭据（应用程序 ID 和密码/证书）。服务主体包含角色分配情况。从 AD 应用程序可以创建多个服务主体。本主题重点介绍单租户应用程序，即应用程序只会在一个组织中运行。通常会将单租户应用程序作为在组织中运行的业务线应用程序使用。如果应用程序需在多个组织中运行，也可创建多租户应用程序。通常会将多租户应用程序用作软件即服务 (SaaS) 应用程序。若要设置多租户应用程序，请参阅[使用 Azure Resource Manager API 进行授权的开发人员指南](/documentation/articles/resource-manager-api-authentication/)。

使用 Active Directory 需要了解多个概念。有关应用程序和服务主体的详细说明，请参阅[应用程序对象和服务主体对象](/documentation/articles/active-directory-application-objects/)。有关 Active Directory 身份验证的详细信息，请参阅 [Azure AD 的身份验证方案](/documentation/articles/active-directory-authentication-scenarios/)。

使用 Azure CLI 时，可以通过 2 个选项进行 AD 应用程序身份验证：

 - password
 - 证书

在设置 AD 应用程序以后，如果需要从其他编程框架（例如 Python、Ruby 或 Node.js）登录 Azure，则最好是使用密码身份验证。在确定是使用密码还是证书之前，请参阅[示例应用程序](#sample-applications)部分，获取在不同框架中进行身份验证的示例。

## 获取租户 ID

以服务主体方式登录时，需提供 AD 应用所在目录的租户 ID。租户是 Active Directory 的实例。不管是使用密码还是使用证书进行身份验证，都需要该值，因此现在需获取该值。

1. 登录到你的帐户。

        azure config mode arm
        azure login -e AzureChinaCloud
1. 若要检索当前进行验证的订阅的租户 ID，则不需提供订阅 ID 作为参数。**-r** 开关检索不带引号的值。

        tenantId=$(azure account show -s <subscriptionId> --json | jq -r '.[0].tenantId')

现在转到下面的一个部分，了解如何进行[密码](#create-service-principal-with-password)或[证书](#create-service-principal-with-certificate)身份验证。


## 使用密码创建服务主体

此部分需执行相应步骤，使用密码创建服务主体，然后将其分配给角色。

1. 创建应用程序的服务主体。提供应用程序的显示名称、描述应用程序的页面的 URI（该链接未验证）、标识应用程序的 URI，以及应用程序标识的密码。此命名同时创建 AD 应用程序和服务主体。

        azure ad sp create -n exampleapp --home-page http://www.contoso.org --identifier-uris https://www.contoso.org/example -p <Your_Password>
        
    随后将返回新的服务主体。授权时需要使用对象 ID。登录时需提供服务主体名称。
    
        info:    Executing command ad sp create
        + Creating application exampleapp
        / Creating service principal for application 7132aca4-1bdb-4238-ad81-996ff91d8db+
        data:    Object Id:               ff863613-e5e2-4a6b-af07-fff6f2de3f4e
        data:    Display Name:            exampleapp
        data:    Service Principal Names:
        data:                             7132aca4-1bdb-4238-ad81-996ff91d8db4
        data:                             https://www.contoso.org/example
        info:    ad sp create command OK

2. 向服务主体授予对订阅的权限。在此示例中，需要向服务主体授予读取订阅中所有资源的权限。有关基于角色的访问控制的详细信息，请参阅 [Azure 基于角色的访问控制](/documentation/articles/role-based-access-control-configure/)。若要分配角色，必须拥有[所有者](/documentation/articles/role-based-access-built-in-roles#owner)角色或[用户访问管理员](/documentation/articles/role-based-access-built-in-roles#user-access-administrator)角色授予的 `Microsoft.Authorization/*/Write` 访问权限。

        azure role assignment create --objectId ff863613-e5e2-4a6b-af07-fff6f2de3f4e -o Reader -c /subscriptions/{subscriptionId}/

就这么简单！ AD 应用程序和服务主体设置完毕。下部分说明如何通过 Azure CLI 使用凭据登录；但是，如果要在代码应用程序中使用凭据，则不需继续学习此主题。可以跳到[示例应用程序](#sample-applications)，获取使用应用程序 ID 和密码登录的示例。

### 通过 Azure CLI 提供凭据

1. 至于用户名，可使用创建服务主体时返回的服务主体名称。如果需要检索应用程序 ID，请使用以下命令。在 **search** 参数中提供 Active Directory 应用程序的名称。

        appId=$(azure ad app show --search exampleapp --json | jq -r '.[0].appId')

3. 以服务主体身份登录。

        azure login -e AzureChinaCloud -u "$appId" --service-principal --tenant "$tenantId"

    系统将提示你输入密码。提供在创建 AD 应用程序时指定的密码。

        info:    Executing command login
        Password: ********

现在，用户已作为所创建服务主体的服务主体进行身份验证。

## 使用证书创建服务主体

本部分需执行相关步骤，为使用证书进行身份验证的服务主体创建服务主体。
若要完成这些步骤，必须已安装 [OpenSSL](http://www.openssl.org/)。

1. 创建自签名证书。

        openssl req -x509 -days 3650 -newkey rsa:2048 -out cert.pem -nodes -subj '/CN=exampleapp'

2. 将公钥和私钥组合在一起。

        cat privkey.pem cert.pem > examplecert.pem

3. 打开 **examplecert.pem** 文件并复制证书数据。查找 **-----BEGIN CERTIFICATE-----** 和 **-----END CERTIFICATE-----** 之间的长字符序列。

1. 创建应用程序的服务主体。提供上一步返回的应用程序 ID。

        azure ad sp create -n "exampleapp" --home-page "https://www.contoso.org" -i "https://www.contoso.org/example" --key-value <certificate data>
        
    随后将返回新的服务主体。授权时需要使用对象 ID。
    
        info:    Executing command ad sp create
        - Creating service principal for application 4fd39843-c338-417d-b549-a545f584a74+
        data:    Object Id:        7dbc8265-51ed-4038-8e13-31948c7f4ce7
        data:    Display Name:     exampleapp
        data:    Service Principal Names:
        data:                      4fd39843-c338-417d-b549-a545f584a745
        data:                      https://www.contoso.org/example
        info:    ad sp create command OK
        
2. 向服务主体授予对订阅的权限。在此示例中，需要向服务主体授予读取订阅中所有资源的权限。有关基于角色的访问控制的详细信息，请参阅 [Azure 基于角色的访问控制](/documentation/articles/role-based-access-control-configure/)。若要分配角色，必须拥有[所有者](/documentation/articles/role-based-access-built-in-roles#owner)角色或[用户访问管理员](/documentation/articles/role-based-access-built-in-roles#user-access-administrator)角色授予的 `Microsoft.Authorization/*/Write` 访问权限。

        azure role assignment create --objectId 7dbc8265-51ed-4038-8e13-31948c7f4ce7 -o Reader -c /subscriptions/{subscriptionId}/

### 通过自动执行的 Azure CLI 脚本提供证书

若要使用 Azure CLI 进行身份验证，请提供证书指纹、证书文件、应用程序 ID 和租户 ID。

    azure login --service-principal --tenant 00000 -u 000000 --certificate-file C:\certificates\examplecert.pem --thumbprint 000000

现在，你已作为所创建 Active Directory 应用程序的服务主体进行身份验证。

若需检索证书指纹并删除不需要的字符，请使用：

    openssl x509 -in "C:\certificates\examplecert.pem" -fingerprint -noout | sed 's/SHA1 Fingerprint=//g'  | sed 's/://g'
    
它返回的指纹值类似于：

    30996D9CE48A0B6E0CD49DBB9A48059BF9355851

若需检索应用程序 ID，请使用：

    azure ad app show --search exampleapp --json | jq -r '.[0].appId'

### 通过自动执行的 Azure CLI 脚本提供证书

你已创建 Active Directory 应用程序并为该应用程序创建服务主体。你已向角色分配服务主体。现在，你需要以服务主体身份登录，以便以服务主体身份执行操作。

若要使用 Azure CLI 进行身份验证，请提供证书指纹、证书文件、应用程序 ID 和租户 ID。

        azure login -e AzureChinaCloud --service-principal --tenant {tenant-id} -u {app-id} --certificate-file C:\certificates\examplecert.pem --thumbprint {thumbprint}

现在，你已作为所创建 Active Directory 应用程序的服务主体进行身份验证。

## 后续步骤
  
- 有关将应用程序集成到 Azure 以管理资源的详细步骤，请参阅 [Developer's guide to authorization with the Azure Resource Manager API](/documentation/articles/resource-manager-api-authentication/)（使用 Azure Resource Manager API 进行授权的开发人员指南）。
- 若要获取有关使用证书和 Azure CLI 的详细信息，请参阅[从 Linux 命令行对 Azure 服务主体进行基于证书的身份验证](http://blogs.msdn.com/b/arsen/archive/2015/09/18/certificate-based-auth-with-azure-service-principals-from-linux-command-line.aspx)。

<!---HONumber=Mooncake_0919_2016-->