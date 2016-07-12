
<properties
   pageTitle="Service Fabric 群集安全性：使用 Azure Active Directory 进行客户端身份验证 | Azure"
   description="本文介绍如何创建使用 Azure Active Directory (AAD) 进行客户端身份验证的 Service Fabric 群集"
   services="service-fabric"
   documentationCenter=".net"
   authors="seanmck"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="06/13/2016"
   wacn.date="07/04/2016"/>

# 创建使用 Azure Active Directory 进行客户端身份验证的 Service Fabric 群集

你可以使用 Azure Active Directory (AAD) 来保护对 Service Fabric 群集管理终结点的访问。本文介绍如何创建所需的 AAD 项目、如何在群集创建期间填充这些项目，以及之后如何连接到这些群集。

## 在 AAD 中为 Service Fabric 群集建模

AAD 可让组织（称为租户）管理用户对应用程序的访问，这些应用程序划分为提供基于 Web 的 UI 的应用程序，以及提供本机客户端体验的应用程序。在本文中，我们假设你已创建一个租户。否则，请先阅读 [How to get an Azure Active Directory tenant](/documentation/articles/active-directory-howto-tenant/)（如何获取 Azure Active Directory 租户）。

Service Fabric 群集提供其管理功能的各种入口点（包括基于 Web 的 [Service Fabric Explorer](/documentation/articles/service-fabric-visualizing-your-cluster/) 和 [Visual Studio](/documentation/articles/service-fabric-manage-application-in-visual-studio/)）。因此，你将要创建两个 AAD 应用程序来控制对群集的访问：一个 Web 应用程序和一个本机应用程序。

为了简化涉及到配置 AAD 与 Service Fabric 群集的一些步骤，我们创建了一组 Windows PowerShell 脚本。

>[AZURE.NOTE] 必须在创建群集*之前*执行这些步骤；因此，在脚本需要群集名称和终结点的情况下，这些应该是计划的值，而不是所创建的值。

1. [将脚本下载到][sf-aad-ps-script-download]你的计算机。

2. 右键单击 zip 文件，选择“属性”，然后选中“取消阻止”复选框，并应用。

3. 解压缩 zip 文件。

4. 运行 `SetupApplications.ps1` 并提供 TenantId、ClusterName 和 WebApplicationReplyUrl 作为参数。例如：

    ```powershell
    .\SetupApplications.ps1 -TenantId '690ec069-8200-4068-9d01-5aaf188e557a' -ClusterName 'mycluster' -WebApplicationReplyUrl 'https://mycluster.chinaeast.chinacloudapp.cn:19080/Explorer/index.html'
    ```

    可以通过在 Azure 经典管理门户中查看租户的 URL 来查找 **TenantId**。该 URL 中嵌入的 GUID 就是 TenantId。例如：

    https://<i></i>manage.windowsazure.cn/microsoft.onmicrosoft.com#Workspaces/ActiveDirectoryExtension/Directory/**690ec069-8200-4068-9d01-5aaf188e557a**/users

    脚本创建的 AAD 应用程序的前面将加上 **ClusterName**。它不需要完全符合实际群集名称，因为它只是让你更轻松地将 AAD 项目映射到与其配合使用的 Service Fabric 群集。

    **WebApplicationReplyUrl** 是 AAD 在完成登录过程之后将用户返回到的默认终结点。你应该将此项设置为群集的 Service Fabric Explorer 终结点，默认值为：

    https://&lt;cluster_domain&gt;:19080/Explorer

    系统将提示你登录到具有 AAD 租户管理权限的帐户。在你完成此操作后，脚本将继续创建 Web 和本机应用程序来代表 Service Fabric 群集。如果你在 [Azure 经典管理门户][azure-management-portal]中查看租户的应用程序，应会看到两个新条目：

    - *ClusterName*\_Cluster
    - *ClusterName*\_Client

    该脚本将列显你在下一部分创建群集时 Azure Resource Manager (ARM) 模板所需的 Json，使 PowerShell 窗口保持打开状态。

    ![SetupApplication 脚本输出包括 ARM 模板所需的 Json][setupapp-script-output]

## 创建群集

现在，你已创建 AAD 应用程序，接下来可以创建 Service Fabric 群集。目前，Azure 门户不支持配置 Service Fabric 群集的 AAD 身份验证，因此，你需要在 PowerShell 或 Visual Studio 中使用 ARM 模板完成此操作。

请注意，AAD 仅用于向群集进行客户端身份验证。若要创建安全群集，你还必须提供证书用于保护群集中节点之间的通信，并提供群集管理终结点的服务器身份验证。你可以查找 [Azure 快速入门库中安全群集的 ARM 模板][secure-cluster-arm-template]，也可以遵循 [Visual Studio 中 Service Fabric 资源组项目](/documentation/articles/service-fabric-cluster-creation-via-visual-studio/)的自述文件中提供的说明。

将 `SetupApplication` 脚本的 ARM 模板代码段输出作为对方项添加到 fabricSettings、managementEndpoint 等。如果你关闭了窗口，也会显示如下代码：

```json
  "azureActiveDirectory": {
    "tenantId": "<your_tenant_id>",
    "clusterApplication": "<your_cluster_application_client_id>",
    "clientApplication": "<your_native_application_client_id>"
  }
```

clusterApplication 表示在上一部分创建的 Web 应用程序。你可以在 SetupApplication 脚本输出中找到其ID（称为 `WebAppId`）。clientApplication 表示本机应用程序，在 SetupApplication 输出中，其客户端 ID 以 NativeClientAppId 的形式提供。

## 将用户分配到角色

创建用于表示群集的应用程序后，需要将用户分配到 Service Fabric 支持的角色：只读和管理员。可以使用 [Azure 经典管理门户][azure-management-portal]来执行此操作。

1. 导航到你的租户，然后选择“应用程序”。
2. 选择名称类似于 `myTestCluster_Cluster` 的 Web 应用程序。
3. 单击“用户”选项卡。
4. 选择要分配的用户，然后单击屏幕底部的“分配”按钮。

    ![将用户分配到角色按钮][assign-users-to-roles-button]

5. 选择要分配给用户的角色。

    ![将用户分配到角色][assign-users-to-roles-dialog]

>[AZURE.NOTE] 有关 Service Fabric 中角色的详细信息，请参阅 [Role-based access control for Service Fabric clients](/documentation/articles/service-fabric-cluster-security-roles/)（适用于 Service Fabric 客户端的基于角色的访问控制）。

## 连接到群集

在启用 AAD 的群集上导航到 Service Fabric Explorer 时，你将自动重定向到安全登录页。

若要从 Windows PowerShell 或 Visual Studio 等本机客户端进行连接，需要将 AAD 指定为登录机制，然后提供服务器证书指纹用于验证终结点的身份。下面显示了这两个入口点的详细信息。

### 从 Visual Studio 连接

在 Visual Studio 中，你可以修改发布配置文件以添加所需的属性，如下所示：

```xml
<ClusterConnectionParameters     
    ConnectionEndpoint="<your_cluster_endpoint>:19000"  
    AzureActiveDirectory="true"
    ServerCertThumbprint="<your_cert_thumbprint>"
    />
```

当你发布到群集时，Visual Studio 将弹出一个可在其中向群集进行身份验证的登录窗口。

![Visual Studio 发布期间的 AAD 登录窗口][vs-publish-aad-login]

### 从 Windows PowerShell 连接

在 PowerShell 中，你可以提供 Connect-ServiceFabricCluster cmdlet 的所需参数，如下所示：

```PowerShell
Connect-ServiceFabricCluster -AzureActiveDirectory -ConnectionEndpoint <cluster_endpoint>:19000 -ServerCertThumbprint <server_cert_thumbprint>
```

与在 Visual Studio 中一样，PowerShell 将显示用于身份验证的安全登录窗口。

>[AZURE.NOTE] 默认情况下，PowerShell 和 Visual Studio 使用的 Service Fabric TCP 网关将侦听端口 19000。如果你配置了其他端口，应在指定连接终结点时改用该端口。

## 已知问题

### 因回复地址不匹配而发生本机客户端身份验证错误

从本机客户端（例如 Visual Studio 或 PowerShell）进行身份验证时，你可能会看到如下所示的错误消息：

“回复地址 http://localhost/ 与针对应用程序 &lt;群集客户端应用程序 GUID&gt; 配置的回复地址不匹配”

若要解决此问题，除了现有的地址“urn:ietf:wg:oauth:2.0:oob”以外，还应将 **http://<i></i>localhost** 作为重定向 URI 添加到 AAD 的群集客户端应用程序定义中。

## 后续步骤

- 阅读有关 [Service Fabric 群集安全性](/documentation/articles/service-fabric-cluster-security/)的详细信息
- 了解如何[使用 Visual Studio 发布到远程群集](/documentation/articles/service-fabric-publish-app-remote-cluster/)

<!-- Links -->
[sf-aad-ps-script-download]: http://servicefabricsdkstorage.blob.core.windows.net/publicrelease/MicrosoftAzureServiceFabric-AADHelpers.zip
[secure-cluster-arm-template]: https://github.com/Azure/azure-quickstart-templates/tree/master/service-fabric-secure-cluster-5-node-1-nodetype-wad
[aad-graph-api-docs]: https://msdn.microsoft.com/zh-cn/library/azure/ad/graph/api/api-catalog
[azure-management-portal]: https://manage.windowsazure.cn

<!-- Images -->
[assign-users-to-roles-button]: ./media/service-fabric-cluster-security-client-auth-with-aad/assign-users-to-roles-button.png
[assign-users-to-roles-dialog]: ./media/service-fabric-cluster-security-client-auth-with-aad/assign-users-to-roles.png
[setupapp-script-output]: ./media/service-fabric-cluster-security-client-auth-with-aad/setupapp-script-arm-json-output.png
[vs-publish-aad-login]: ./media/service-fabric-cluster-security-client-auth-with-aad/vs-login-prompt.png

<!---HONumber=Mooncake_0627_2016-->