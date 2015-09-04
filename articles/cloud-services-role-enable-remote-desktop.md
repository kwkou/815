<properties 
pageTitle="为 Azure 云服务中的角色设置远程桌面连接" 
description="如何配置 Azure 云服务应用程序以允许远程桌面连接" 
services="cloud-services" 
documentationCenter="" 
authors="Thraka" 
manager="timlt" 
editor=""/>
<tags 
ms.service="cloud-services"  
ms.date="07/06/2015" 
wacn.date="08/29/2015"/>

# 为 Azure 中的角色设置远程桌面连接
在创建运行应用程序的云服务后，可以远程访问该应用程序中的角色实例，以配置设置或解决问题。必须已为远程桌面配置云服务。

* 是否需要一个证书来启用远程桌面身份验证？ [创建](/documentation/articles/cloud-services-certs-create)一个证书并在下面进行配置。
* 你的云服务是否已有远程桌面设置？ [连接](#remote-into-role-instances)到云服务。

## 设置 Web 角色或辅助角色的远程桌面连接
要启用 Web 角色或辅助角色的远程桌面连接，则可以在应用程序的服务模型中设置连接，或者可以使用 Azure 管理门户在实例进行运行后设置连接。

### 在服务模型中设置连接
**导入**元素必须添加到将 **RemoteAccess** 模块和 **RemoteForwarder** 模块导入服务模型的服务定义文件。使用 Visual Studio 创建 Azure 项目时，将创建服务模型的文件。

服务模型由一个 [ServiceDefinition.csdef](cloud-services-model-and-package.md#csdef) 文件和一个 [ServiceConfiguration.cscfg](cloud-services-model-and-package.md#cscfg) 文件组成。当准备云服务的应用程序进行部署时，定义文件将与角色二进制文件一起打包。ServiceConfiguration.cscfg 文件与应用程序包一起部署并被 Azure 用于确定应用程序的运行方式。

创建项目后，可以使用 Visual Studio [启用远程桌面连接](https://msdn.microsoft.com/zh-cn/library/gg443832.aspx)。

配置连接后，服务定义文件应类似于下面的示例，并添加 `<Imports>` 元素。

```xml
<ServiceDefinition name="<name-of-cloud-service>" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" schemaVersion="2013-03.2.0">
    <WebRole name="WebRole1" vmsize="Small">
        <Sites>
            <Site name="Web">
                <Bindings>
                    <Binding name="Endpoint1" endpointName="Endpoint1" />
                </Bindings>
            </Site>
        </Sites>
        <Endpoints>
            <InputEndpoint name="Endpoint1" protocol="http" port="80" />
        </Endpoints>
        <Imports>
            <Import moduleName="Diagnostics" />
            <Import moduleName="RemoteAccess" />
            <Import moduleName="RemoteForwarder" />
        </Imports>
    </WebRole>
</ServiceDefinition>
```

服务配置文件应类似于下面的示例，并具有你在设置连接时提供的值，请注意 `<ConfigurationSettings>` 和 `<Certificates>` 元素：

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceConfiguration serviceName="<name-of-cloud-service>" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration" osFamily="3" osVersion="*" schemaVersion="2013-03.2.0">
    <Role name="WebRole1">
        <Instances count="2" />
        <ConfigurationSettings>
            <Setting name="Microsoft.WindowsAzure.Plugins.RemoteAccess.Enabled" value="true" />
            <Setting name="Microsoft.WindowsAzure.Plugins.RemoteAccess.AccountUsername" value="[name-of-user-account]" />
            <Setting name="Microsoft.WindowsAzure.Plugins.RemoteAccess.AccountEncryptedPassword" value="[base-64-encrypted-user-password]" />
            <Setting name="Microsoft.WindowsAzure.Plugins.RemoteAccess.AccountExpiration" value="[certificate-expiration]" />
            <Setting name="Microsoft.WindowsAzure.Plugins.RemoteForwarder.Enabled" value="true" />
        </ConfigurationSettings>
        <Certificates>
            <Certificate name="Microsoft.WindowsAzure.Plugins.RemoteAccess.PasswordEncryption" thumbprint="[certificate-thumbprint]" thumbprintAlgorithm="sha1" />
        </Certificates>
    </Role>
</ServiceConfiguration>
```

当你[打包](cloud-services-model-and-package.md#cspkg)并[发布](/documentation/articles/cloud-services-how-to-create-deploy-portal)应用程序时，必须确保已选中“为所有角色启用远程桌面”复选框。[此](https://msdn.microsoft.com/zh-cn/library/ff683672.aspx)文章提供了与使用 Visual Studio 和云服务相关的常见任务的列表。

### 在正在运行的实例上设置连接
在云服务的配置页上，可以启用或修改远程桌面连接设置。有关详细信息，请参阅[配置角色实例的远程访问](/documentation/articles/cloud-services-how-to-configure)。




## 远程到角色实例
要访问 Web 角色或辅助角色的实例，则必须使用远程桌面协议 (RDP) 文件。可以从管理门户下载该文件，也可以以编程方式检索该文件。

### 通过管理门户下载 RDP 文件

可以使用以下步骤从管理门户检索 RDP 文件，然后使用远程桌面连接，连接到使用该文件的实例：

1.  在实例页上，选择该实例，然后单击命令栏上的“连接”。
2.  单击“保存”将远程桌面协议文件保存到本地计算机。
3.  打开“远程桌面连接”，单击“显示选项”，然后单击“打开”。
4.  浏览保存 RDP 文件的位置，选择该文件，单击“打开”，然后单击“连接”。按照说明操作完成连接。

### 使用 PowerShell 获取 RDP 文件
可以使用 [Get-AzureRemoteDesktopFile](https://msdn.microsoft.com/zh-cn/library/azure/dn495261.aspx) cmdlet 检索 RDP 文件。

### 使用 Visual Studio 下载 RDP 文件
在 Visual Studio 中，可以使用服务器资源管理器来创建远程桌面连接。

1.  在服务器资源管理器中，展开“Azure\\云服务\\[云服务名称]”节点。
2.  展开“暂存”或“生产”。
3.  展开各个角色。
4.  右键单击某一角色实例，单击“使用远程桌面连接...”，然后输入用户名和密码。

### 通过服务管理 REST API 以编程方式下载 RDP 文件
可以使用[下载 RDP 文件](https://msdn.microsoft.com/zh-cn/library/jj157183.aspx) REST 操作下载 RDP 文件。然后可以使用具有远程桌面连接的 RDP 文件来访问云服务。

## 后续步骤
可能需要[打包](/documentation/articles/cloud-services-model-and-package)或[上载（部署）](/documentation/articles/cloud-services-how-to-create-deploy-portal)你的应用程序。

<!---HONumber=67-->