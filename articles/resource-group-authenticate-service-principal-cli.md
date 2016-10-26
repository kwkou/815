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
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="multiple"
   ms.workload="na"
   ms.date="09/07/2016"
   wacn.date="10/24/2016"/>

# 使用 Azure CLI 创建服务主体来访问资源

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/resource-group-authenticate-service-principal/)
- [Azure CLI](/documentation/articles/resource-group-authenticate-service-principal-cli/)
- [门户](/documentation/articles/resource-group-create-service-principal-portal/)


如果应用程序或脚本需要访问资源，用户在多数情况下不想要使用自己的凭据来运行此过程。用户可能有几种不同的权限可用于应用程序，如果用户职责改变，则不想要应用程序继续使用此凭据。与上述方法不同，也可以为应用程序创建一个标识，其中包括身份验证凭据和角色分配情况。每次应用运行时，将使用这些凭据对其自身进行身份验证。本主题介绍如何通过[适用于 Mac、Linux 和 Windows 的 Azure CLI](/documentation/articles/xplat-cli-install/) 为应用程序进行设置，使之能够使用自己的凭据和标识运行。

使用 Azure CLI 时，可以通过 2 个选项进行 AD 应用程序身份验证：

 - password
 - 证书

本主题演示如何在 Azure CLI 中使用这两个选项。如果想从一个编程框架（例如 Python、Ruby 或 Node.js）登录 Azure，则最好是使用密码身份验证。在确定是使用密码还是证书之前，请参阅[示例应用程序](#sample-applications)部分，获取在不同框架中进行身份验证的示例。

## Active Directory 概念

在本文中，用户将创建两个对象 - Active Directory (AD) 应用程序和服务主体。AD 应用程序是应用程序的全局表示形式。它包含凭据（应用程序 ID 和密码或证书）。服务主体是应用程序在 Active Directory 中的本地表示形式。它包含角色分配。本主题重点介绍单租户应用程序，即应用程序只会在一个组织中运行。通常会将单租户应用程序作为在组织中运行的业务线应用程序使用。在单租户应用程序中，有一个 AD 应用和一个服务主体。

你可能会疑惑 - 为什么需要这两个对象？ 在考虑多租户应用程序时，此方法会更合理。通常对软件即服务 (SaaS) 应用程序使用多租户应用程序，其中应用程序在许多不同订阅中运行。对于多租户应用程序，有一个 AD 应用和多个服务主体（授予应用访问权限的每个 Active Directory 中有一个）。若要设置多租户应用程序，请参阅[使用 Azure Resource Manager API 进行授权的开发人员指南](/documentation/articles/resource-manager-api-authentication/)。

## 所需的权限

若要完成本主题，必须在 Azure Active Directory 和 Azure 订阅中均具有足够的权限。具体而言，必须能够在 Active Directory 中创建应用并向角色分配服务主体。

在 Active Directory 中，你的帐户必须是管理员（如“全局管理员”或“用户管理员”）。如果帐户分配为“用户”角色，则需要有管理员提升你的权限。

在订阅中，帐户必须拥有[所有者](/documentation/articles/role-based-access-built-in-roles/#owner)角色或[用户访问管理员](/documentation/articles/role-based-access-built-in-roles/#user-access-administrator)角色授予的 `Microsoft.Authorization/*/Write` 访问权限。如果你的帐户分配到“参与者”角色，则在尝试向角色分配服务主体时将收到错误。同样，订阅管理员必须向你授予足够的访问权限。

现在转到[密码](#create-service-principal-with-password)或[证书](#create-service-principal-with-certificate)身份验证部分。

## 使用密码创建服务主体

在本部分中，将执行步骤以：

- 使用密码创建 AD 应用程序，并创建服务主体
- 向服务主体分配“读取者”角色

若要快速执行这些步骤，请使用以下命令。

    azure ad sp create -n exampleapp --home-page http://www.contoso.org --identifier-uris https://www.contoso.org/example -p <Your_Password>
    azure role assignment create --objectId ff863613-e5e2-4a6b-af07-fff6f2de3f4e -o Reader -c /subscriptions/{subscriptionId}/

仔细浏览这些步骤以确保理解该过程。

1. 登录到你的帐户。

        azure config mode arm
        azure login -e AzureChinacloud

1. 创建应用程序的服务主体。提供显示名称、描述应用程序的页面的 URI、标识应用程序的 URI，以及应用程序标识的密码。此命令同时创建 AD 应用程序和服务主体。

        azure ad sp create -n exampleapp --home-page http://www.contoso.org --identifier-uris https://www.contoso.org/example -p {your-password}
        
     对于单租户应用程序，不会验证 URI。
     
     如果帐户在 Active Directory 上不具有[所需的权限](#required-permissions)，将看到指示“Authentication\_Unauthorized”或“上下文中找不到订阅”的错误消息。
    
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

2. 向服务主体授予对订阅的权限。在此示例中，向“读取者”角色（授予读取订阅中所有资源的权限）添加服务主体。对于其他角色，请参阅 [RBAC：内置角色](/documentation/articles/role-based-access-built-in-roles/)。对于 **ServicePrincipalName** 参数，请提供创建应用程序时使用的 **ObjectId**。

        azure role assignment create --objectId ff863613-e5e2-4a6b-af07-fff6f2de3f4e -o Reader -c /subscriptions/{subscriptionId}/

     如果帐户没有足够权限来分配角色，将看到一条错误消息。该消息声明你的帐户**无权执行操作 'Microsoft.Authorization/roleAssignments/write' over scope '/subscriptions/{guid}'**。

就这么简单！ AD 应用程序和服务主体设置完毕。下一部分演示如何通过 Azure CLI 使用凭据进行登录。如果想在代码应用程序中使用凭据，则不需要继续了解本主题。可以跳到[示例应用程序](#sample-applications)，获取使用应用程序 ID 和密码登录的示例。

### 通过 Azure CLI 提供凭据

现在，需要以应用程序方式登录以执行相应操作。

1. 以服务主体方式登录时，需提供 AD 应用所在目录的租户 ID。租户是 Active Directory 的实例。若要检索当前已经过身份验证的订阅的租户 ID，请使用：

        azure account show

     将返回：

        info:    Executing command account show
        data:    Name                        : Microsoft Azure MSDN - Visual Studio Ultimate
        data:    ID                          : {guid}
        data:    State                       : Enabled
        data:    Tenant ID                   : {guid}
        data:    Is Default                  : true
        ...

     如果需要获取另一个订阅的租户 ID，请使用以下命令：

        azure account show -s {subscription-id}

3. 以服务主体方式登录。

        azure login -u https://www.contoso.org/example --service-principal --tenant {tenant-id}

    系统将提示输入密码。提供在创建 AD 应用程序时指定的密码。

        info:    Executing command login
        Password: ********

现在，用户已作为所创建服务主体的服务主体进行身份验证。

## 使用证书创建服务主体

在本部分中，将执行步骤以：

- 创建自签名证书
- 使用证书创建 AD 应用程序，并创建服务主体
- 向服务主体分配“读取者”角色

若要完成这些步骤，必须已安装 [OpenSSL](http://www.openssl.org/)。

1. 创建自签名证书。

        openssl req -x509 -days 3650 -newkey rsa:2048 -out cert.pem -nodes -subj '/CN=exampleapp'

2. 将公钥和私钥组合在一起。

        cat privkey.pem cert.pem > examplecert.pem

3. 打开 **examplecert.pem** 文件并查找 **-----BEGIN CERTIFICATE-----** 和 **-----END CERTIFICATE-----** 之间的长字符序列。复制证书数据。创建服务主体时将此数据作为参数传递。

1. 登录到你的帐户。

        azure config mode arm
        azure login -e AzureChinaCloud

1. 创建应用程序的服务主体。提供显示名称、描述应用程序的页面的 URI、标识应用程序的 URI，以及复制的证书数据。此命令同时创建 AD 应用程序和服务主体。

        azure ad sp create -n "exampleapp" --home-page "https://www.contoso.org" -i "https://www.contoso.org/example" --key-value <certificate data>
        
     对于单租户应用程序，不会验证 URI。
     
     如果帐户在 Active Directory 上不具有[所需的权限](#required-permissions)，将看到指示“Authentication\_Unauthorized”或“上下文中找不到订阅”的错误消息。
    
     随后将返回新的服务主体。授权时需要使用对象 ID。
    
        info:    Executing command ad sp create
        - Creating service principal for application 4fd39843-c338-417d-b549-a545f584a74+
        data:    Object Id:        7dbc8265-51ed-4038-8e13-31948c7f4ce7
        data:    Display Name:     exampleapp
        data:    Service Principal Names:
        data:                      4fd39843-c338-417d-b549-a545f584a745
        data:                      https://www.contoso.org/example
        info:    ad sp create command OK
        
2. 向服务主体授予对订阅的权限。在此示例中，向“读取者”角色（授予读取订阅中所有资源的权限）添加服务主体。对于其他角色，请参阅 [RBAC：内置角色](/documentation/articles/role-based-access-built-in-roles/)。对于 **ServicePrincipalName** 参数，请提供创建应用程序时使用的 **ObjectId**。

        azure role assignment create --objectId 7dbc8265-51ed-4038-8e13-31948c7f4ce7 -o Reader -c /subscriptions/{subscriptionId}/

     如果帐户没有足够权限来分配角色，将看到一条错误消息。该消息声明你的帐户**无权执行操作 'Microsoft.Authorization/roleAssignments/write' over scope '/subscriptions/{guid}'**。

### 通过自动执行的 Azure CLI 脚本提供证书

现在，需要以应用程序方式登录以执行相应操作。

1. 以服务主体方式登录时，需提供 AD 应用所在目录的租户 ID。租户是 Active Directory 的实例。若要检索当前已经过身份验证的订阅的租户 ID，请使用：

        azure account show

     将返回：

        info:    Executing command account show
        data:    Name                        : Microsoft Azure MSDN - Visual Studio Ultimate
        data:    ID                          : {guid}
        data:    State                       : Enabled
        data:    Tenant ID                   : {guid}
        data:    Is Default                  : true
        ...

     如果需要获取另一个订阅的租户 ID，请使用以下命令：

        azure account show -s {subscription-id}

1. 若要检索证书指纹并删除不需要的字符，请使用：

        openssl x509 -in "C:\certificates\examplecert.pem" -fingerprint -noout | sed 's/SHA1 Fingerprint=//g'  | sed 's/://g'
    
     它返回的指纹值类似于：

        30996D9CE48A0B6E0CD49DBB9A48059BF9355851

1. 以服务主体方式登录。

        azure login --service-principal --tenant {tenant-id} -u https://www.contoso.org/example --certificate-file C:\certificates\examplecert.pem --thumbprint {thumbprint}

现在，你已作为所创建 Active Directory 应用程序的服务主体进行身份验证。

## 示例应用程序

以下示例应用程序演示如何以服务主体身份登录。

**.NET**

- [在 .NET 中使用模板部署启用 SSH 的 VM](https://azure.microsoft.com/documentation/samples/resource-manager-dotnet-template-deployment/)
- [使用 .NET 管理 Azure 资源和资源组](https://azure.microsoft.com/documentation/samples/resource-manager-dotnet-resources-and-groups/)

**Java**

- [资源入门 - 在 Java 中使用 Azure Resource Manager 模板部署资源](https://azure.microsoft.com/documentation/samples/resources-java-deploy-using-arm-template/)
- [资源入门 - 在 Java 中管理资源组](https://azure.microsoft.com/documentation/samples/resources-java-manage-resource-group//)

**Python**

- [在 Python 中使用模板部署启用 SSH 的 VM](https://azure.microsoft.com/documentation/samples/resource-manager-python-template-deployment/)
- [使用 Python 管理 Azure 资源和资源组](https://azure.microsoft.com/documentation/samples/resource-manager-python-resources-and-groups/)

**Node.js**

- [在 Node.js 中使用模板部署启用 SSH 的 VM](https://azure.microsoft.com/documentation/samples/resource-manager-node-template-deployment/)
- [使用 Node.js 管理 Azure 资源和资源组](https://azure.microsoft.com/documentation/samples/resource-manager-node-resources-and-groups/)

**Ruby**

- [在 Ruby 中使用模板部署启用 SSH 的 VM](https://azure.microsoft.com/documentation/samples/resource-manager-ruby-template-deployment/)
- [使用 Ruby 管理 Azure 资源和资源组](https://azure.microsoft.com/documentation/samples/resource-manager-ruby-resources-and-groups/)


## 后续步骤
  
- 有关将应用程序集成到 Azure 以管理资源的详细步骤，请参阅 [Developer's guide to authorization with the Azure Resource Manager API](/documentation/articles/resource-manager-api-authentication/)（使用 Azure Resource Manager API 进行授权的开发人员指南）。
- 若要获取有关使用证书和 Azure CLI 的详细信息，请参阅[从 Linux 命令行对 Azure 服务主体进行基于证书的身份验证](http://blogs.msdn.com/b/arsen/archive/2015/09/18/certificate-based-auth-with-azure-service-principals-from-linux-command-line.aspx)。

<!---HONumber=Mooncake_1017_2016-->