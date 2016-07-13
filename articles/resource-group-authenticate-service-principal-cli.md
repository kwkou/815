<properties
   pageTitle="使用 Azure CLI 创建 AD 应用程序 | Azure"
   description="描述如何使用 Azure CLI 创建 Active Directory 应用程序，并通过基于角色的访问控制授予它对资源的访问权限。它演示如何使用密码或证书对应用程序进行身份验证。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="timlt"
   editor="tysonn"/>

<tags
   ms.service="azure-resource-manager"
   ms.date="05/26/2016"
   wacn.date="07/11/2016"/>

# 使用 Azure CLI 创建可访问资源的 Active Directory 应用程序

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/resource-group-authenticate-service-principal)
- [Azure CLI](/documentation/articles/resource-group-authenticate-service-principal-cli)
- [门户](/documentation/articles/resource-group-create-service-principal-portal)

本主题演示如何使用[适用于 Mac、Linux 和 Windows 的 Azure CLI](/documentation/articles/xplat-cli-install) 创建 Active Directory (AD) 应用程序，如可访问你的订阅中的其他资源的自动化进程、应用程序或服务。借助 Azure Resource Manager，可以使用基于角色的访问控制向应用程序授予允许的操作。

在本文中，你将创建两个对象 - AD 应用程序和服务主体。AD 应用程序位于注册了应用的租户中，并定义了要运行的进程。服务主体包含 AD 应用程序的标识，并用于分配权限。从 AD 应用程序可以创建多个服务主体。有关应用程序和服务主体的详细说明，请参阅[应用程序对象和服务主体对象](/documentation/articles/active-directory-application-objects)。有关 Active Directory 身份验证的详细信息，请参阅 [Azure AD 的身份验证方案](/documentation/articles/active-directory-authentication-scenarios)。

有 2 个选项用于验证应用程序：

 - 密码 - 当用户想要在执行期间以交互方式登录时适用
 - 证书 - 适用于必须无需用户交互进行身份验证的无人参与脚本
 
## 创建使用密码的 AD 应用程序

在此部分中，你将执行相应步骤来创建使用密码的 AD 应用程序。

1. 切换到 Azure Resource Manager 模式并登录到你的帐户。

        azure config mode arm
        azure login -e AzureChinaCloud

2. 运行 **azure ad app create** 命令创建新的 AD 应用程序。提供应用程序的显示名称、描述应用程序的页面的 URI（该链接未验证）、标识应用程序的 URI，以及应用程序标识的密码。

        azure ad app create --name "exampleapp" --home-page "https://www.contoso.org" --identifier-uris "https://www.contoso.org/example" --password <Your_Password>
        
    将返回该 AD 应用程序。需要使用 AppId 属性来创建服务主体以及获取访问令牌。

        data:    AppId:          4fd39843-c338-417d-b549-a545f584a745
        data:    ObjectId:       4f8ee977-216a-45c1-9fa3-d023089b2962
        data:    DisplayName:    exampleapp
        ...
        info:    ad app create command OK

### 创建服务主体，并将其分配给角色

1. 创建应用程序的服务主体。提供上一步返回的应用程序 ID。

        azure ad sp create 4fd39843-c338-417d-b549-a545f584a745
        
    随后将返回新的服务主体。授权时需要使用对象 ID。
    
        info:    Executing command ad sp create
        - Creating service principal for application 4fd39843-c338-417d-b549-a545f584a74+
        data:    Object Id:        7dbc8265-51ed-4038-8e13-31948c7f4ce7
        data:    Display Name:     exampleapp
        data:    Service Principal Names:
        data:                      4fd39843-c338-417d-b549-a545f584a745
        data:                      https://www.contoso.org/example
        info:    ad sp create command OK

2. 向服务主体授予对你的订阅的权限。在此示例中，你将要向服务主体授予读取订阅中所有资源的权限。有关基于角色的访问控制的详细信息，请参阅 [Azure 基于角色的访问控制](/documentation/articles/role-based-access-control-configure)。

        azure role assignment create --objectId 7dbc8265-51ed-4038-8e13-31948c7f4ce7 -o Reader -c /subscriptions/{subscriptionId}/

### 通过 Azure CLI 手动提供凭据

你已创建 AD 应用程序并为该应用程序创建服务主体。你已向角色分配服务主体。现在，你需要以应用程序身份登录以执行相应操作。在执行按需脚本或命令时，可以手动为应用程序提供凭据。

1. 确定包含服务主体的订阅的 **TenantId**。如果你要检索当前进行验证的订阅的租户 ID，则不需要提供订阅 ID 作为参数。**-r** 开关检索不带引号的值。

        tenantId=$(azure account show -s <subscriptionId> --json | jq -r '.[0].tenantId')

2. 对于用户名，请使用你在创建服务主体时所用的 **AppId**。如果需要检索应用程序 ID，请使用以下命令。在 **search** 参数中提供 Active Directory 应用程序的名称。

        appId=$(azure ad app show --search exampleapp --json | jq -r '.[0].appId')

3. 以服务主体身份登录。

        azure login -e AzureChinaCloud -u "$appId" --service-principal --tenant "$tenantId"

    系统将提示你输入密码。提供在创建 AD 应用程序时指定的密码。

        info:    Executing command login
        Password: ********

现在，你已作为所创建 AD 应用程序的服务主体进行身份验证。

## 创建使用证书的 AD 应用程序

在本部分中，你将执行以下步骤，为使用证书进行身份验证的 AD 应用程序创建服务主体。若要完成这些步骤，必须已安装 [OpenSSL](http://www.openssl.org/)。

1. 创建自签名证书。

        openssl req -x509 -days 3650 -newkey rsa:2048 -out cert.pem -nodes -subj '/CN=exampleapp'

2. 将公钥和私钥组合在一起。

        cat privkey.pem cert.pem > examplecert.pem

3. 打开 **examplecert.pem** 文件并复制证书数据。查找 **-----BEGIN CERTIFICATE-----** 和 **-----END CERTIFICATE-----** 之间的长字符序列。

4. 通过运行 **azure ad app create** 命令创建新的 Active Directory 应用程序，并提供你在上一步中复制为密钥值的证书数据。

        azure ad app create -n "exampleapp" --home-page "https://www.contoso.org" -i "https://www.contoso.org/example" --key-value <certificate data>

    返回 Active Directory 应用程序。需要使用 AppId 属性来创建服务主体以及获取访问令牌。

        data:    AppId:          4fd39843-c338-417d-b549-a545f584a745
        data:    ObjectId:       4f8ee977-216a-45c1-9fa3-d023089b2962
        data:    DisplayName:    exampleapp
        ...
        info:    ad app create command OK

### 创建服务主体，并将其分配给角色

1. 创建应用程序的服务主体。提供上一步返回的应用程序 ID。

        azure ad sp create 4fd39843-c338-417d-b549-a545f584a745
        
    随后将返回新的服务主体。授权时需要使用对象 ID。
    
        info:    Executing command ad sp create
        - Creating service principal for application 4fd39843-c338-417d-b549-a545f584a74+
        data:    Object Id:        7dbc8265-51ed-4038-8e13-31948c7f4ce7
        data:    Display Name:     exampleapp
        data:    Service Principal Names:
        data:                      4fd39843-c338-417d-b549-a545f584a745
        data:                      https://www.contoso.org/example
        info:    ad sp create command OK
        
2. 向服务主体授予对你的订阅的权限。在此示例中，你将要向服务主体授予读取订阅中所有资源的权限。有关基于角色的访问控制的详细信息，请参阅 [Azure 基于角色的访问控制](/documentation/articles/role-based-access-control-configure)。

        azure role assignment create --objectId 7dbc8265-51ed-4038-8e13-31948c7f4ce7 -o Reader -c /subscriptions/{subscriptionId}/

### 准备脚本的值

在脚本中，将传入以服务主体身份登录所需的三个值。你将需要：

- 应用程序 ID
- 租户 ID 
- 证书指纹

你已在前面的步骤中看到应用程序 ID 和证书指纹。但是，如果需要以后检索这些值，相应命令将与用于获取租户 ID 的命令一起显示在下面。

1. 若要检索证书指纹并删除不需要的字符，请使用：

        openssl x509 -in "C:\certificates\examplecert.pem" -fingerprint -noout | sed 's/SHA1 Fingerprint=//g'  | sed 's/://g'
    
     它返回的指纹值类似于：

        30996D9CE48A0B6E0CD49DBB9A48059BF9355851

2. 若要检索租户 ID，请使用：

        azure account show -s <subscriptionId> --json | jq -r '.[0].tenantId'

3. 若要检索应用程序 ID，请使用：

        azure ad app show --search exampleapp --json | jq -r '.[0].appId'

### 通过自动执行的 Azure CLI 脚本提供证书

你已创建 Active Directory 应用程序并为该应用程序创建服务主体。你已向角色分配服务主体。现在，你需要以服务主体身份登录，以便以服务主体身份执行操作。

若要使用 Azure CLI 进行身份验证，请提供证书指纹、证书文件、应用程序 ID 和租户 ID。

        azure login -e AzureChinaCloud --service-principal --tenant {tenant-id} -u {app-id} --certificate-file C:\certificates\examplecert.pem --thumbprint {thumbprint}

现在，你已作为所创建 Active Directory 应用程序的服务主体进行身份验证。

## 后续步骤
  
- 有关 .NET 身份验证示例，请参阅 [Azure Resource Manager SDK for .NET](/documentation/articles/resource-manager-net-sdk)。
- 有关 Java 身份验证示例，请参阅 [Azure Resource Manager SDK for Java](/documentation/articles/resource-manager-java-sdk)。 
- 有关 Python 身份验证示例，请参阅 [Resource Management Authentication for Python（适用于 Python 的资源管理身份验证）](https://azure-sdk-for-python.readthedocs.io/en/latest/resourcemanagementauthentication.html)。
- 有关 REST 身份验证示例，请参阅 [Resource Manager REST API](/documentation/articles/resource-manager-rest-api)。
- 有关将应用程序集成到 Azure 以管理资源的详细步骤，请参阅[使用 Azure Resource Manager API 进行授权的开发人员指南](/documentation/articles/resource-manager-api-authentication)。
- 若要获取有关使用证书和 Azure CLI 的详细信息，请参阅[从 Linux 命令行对 Azure 服务主体进行基于证书的身份验证](http://blogs.msdn.com/b/arsen/archive/2015/09/18/certificate-based-auth-with-azure-service-principals-from-linux-command-line.aspx)。 

<!---HONumber=Mooncake_0704_2016-->