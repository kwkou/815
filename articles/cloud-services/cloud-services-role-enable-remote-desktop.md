<properties
    pageTitle="为 Azure 云服务启用远程桌面 | Azure"
    description="如何配置 Azure 云服务应用程序以允许远程桌面连接"
    services="cloud-services"
    documentationcenter=""
    author="thraka"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="d3110ee8-6526-4585-aba5-d0bc9a713e9b"
    ms.service="cloud-services"
    ms.workload="tbd"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/22/2016"
    wacn.date="05/22/2017"
    ms.author="adegeo"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="ccc79b0538accb59d65e689562da0c426fa90bfe"
    ms.lasthandoff="05/12/2017" />

# <a name="enable-remote-desktop-connection-for-a-role-in-azure-cloud-services"></a>为 Azure 云服务中的角色设置远程桌面连接
> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/cloud-services-role-enable-remote-desktop-new-portal/)
- [Azure 经典管理门户](/documentation/articles/cloud-services-role-enable-remote-desktop/)
- [PowerShell](/documentation/articles/cloud-services-role-enable-remote-desktop-powershell/)
- [Visual Studio](/documentation/articles/vs-azure-tools-remote-desktop-roles/)


你可以在开发过程中通过在服务定义中加入远程桌面模块来在角色中启用远程桌面连接，也可以通过远程桌面扩展选择启用远程桌面。首选方法是使用远程桌面扩展，因为即使在部署应用程序后，也能启用远程桌面，而不必重新部署应用程序。

## <a name="configure-remote-desktop-from-the-azure-classic-portal"></a>从 Azure 经典管理门户配置远程桌面
Azure 经典管理门户使用远程桌面扩展方法，即使在部署应用程序之后，也能启用远程桌面。使用云服务的“配置”页，可以启用远程桌面、更改用于连接虚拟机的本地 Administrator 帐户、身份验证使用的证书，以及设置到期日期。

1. 单击“云服务”，单击云服务的名称，然后单击“配置”。
2. 单击底部的“远程”按钮。

    ![云服务远程](./media/cloud-services-role-enable-remote-desktop/CloudServices_Remote.png)  

   > [AZURE.WARNING]
   > 当首次启用远程桌面并单击“确定”（复选标记）时，所有角色实例会重新启动。 为避免重新启动，必须对于此角色安装用于对密码进行加密的证书。 若要避免重新启动，请[上载云服务的证书](/documentation/articles/cloud-services-configure-ssl-certificate/#step-3-upload-a-certificate)，然后返回到此对话框。

3. 在“**角色**”中，选择要更新的角色，或选择“**全部**”以选择所有角色。

4. 进行以下任何更改：
    
    - 若要启用远程桌面，请选中“启用远程桌面”复选框。若要禁用远程桌面，请清除该复选框。
    
    - 创建一个要在角色实例的远程桌面连接中使用的帐户。
    
    - 更新现有帐户的密码。

    - 选择要用于身份验证的已上传证书（使用“证书”页上的“上传”上传证书），或者创建新证书。 

    - 更改远程桌面配置的到期日期。

5. 完成配置更新后，请单击“确定”（复选标记）。

## <a name="remote-into-role-instances"></a>远程到角色实例
对角色启用远程桌面后，可以通过各种工具远程连接到角色实例。

若要从 Azure 经典管理门户连接到角色实例，请执行以下操作：
    
1. 单击“实例”，打开“实例”页。
2. 选择一个配置了远程桌面的角色实例。
3. 单击“连接”，并按照说明打开桌面 。
4. 依次单击“**打开**”和“**连接**”，以启动远程桌面连接。

### <a name="use-visual-studio-to-remote-into-a-role-instance"></a>使用 Visual Studio 远程连接到角色实例

在 Visual Studio 的“服务器资源管理器”中：

1. 展开“Azure” > “云服务” > “[云服务名称]”节点。
2. 展开“过渡”或“生产”。
3. 展开各个角色。
4. 右键单击某一角色实例，单击“使用远程桌面连接...”，然后输入用户名和密码。

![服务器资源管理器远程桌面](./media/cloud-services-role-enable-remote-desktop/ServerExplorer_RemoteDesktop.png)  

### <a name="use-powershell-to-get-the-rdp-file"></a>使用 PowerShell 获取 RDP 文件
可以使用 [Get-AzureRemoteDesktopFile](https://msdn.microsoft.com/zh-cn/library/azure/dn495261.aspx) cmdlet 检索 RDP 文件。 然后可以使用具有远程桌面连接的 RDP 文件来访问云服务。

### <a name="programmatically-download-the-rdp-file-through-the-service-management-rest-api"></a>通过服务管理 REST API 以编程方式下载 RDP 文件
可以使用 [下载 RDP 文件](https://msdn.microsoft.com/zh-cn/library/jj157183.aspx) REST 操作下载 RDP 文件。 

## <a name="to-configure-remote-desktop-in-the-service-definition-file"></a>在服务定义文件中配置远程访问

此方法允许你在开发过程中为应用程序启用远程桌面。 此方法需要将加密的密码存储在服务配置文件中，并且如果对远程桌面配置进行了任何更新，都需要重新部署应用程序。 如果想要避免这些弊端，应使用上面所述的基于远程桌面扩展的方法。  

可以通过服务定义文件方法使用 Visual Studio [启用远程桌面连接](/documentation/articles/vs-azure-tools-remote-desktop-roles/)。  
下面的步骤介绍了启用远程桌面需要对服务模型文件进行的更改。 发布时，Visual Studio 会自动进行这些更改。

### <a name="set-up-the-connection-in-the-service-model"></a>在服务模型中设置连接 
使用 **Imports** 元素将 **RemoteAccess** 模块和 **RemoteForwarder** 模块导入到 [ServiceDefinition.csdef](/documentation/articles/cloud-services-model-and-package/#csdef) 文件中。

服务定义文件应类似于下面的示例，并添加 `<Imports>` 元素。

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
[ServiceConfiguration.cscfg](/documentation/articles/cloud-services-model-and-package/#cscfg) 文件应类似于下面的示例，请注意 `<ConfigurationSettings>` 和 `<Certificates>` 元素。 指定的证书必须[已上传到云服务](/documentation/articles/cloud-services-how-to-create-deploy/#how-to-upload-a-certificate-for-a-cloud-service)。


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

## <a name="additional-resources"></a>其他资源

 - [如何配置云服务](/documentation/articles/cloud-services-how-to-configure/)
 - [云服务常见问题 - 远程桌面](/documentation/articles/cloud-services-faq/#remote-desktop)
