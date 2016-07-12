<properties
   pageTitle="配置受 Service Fabric 群集支持的安全连接 | Microsoft Azure"
   description="了解如何使用 Visual Studio 来配置 Azure Service Fabric 群集支持的安全连接。"
   services="service-fabric"
   documentationCenter="na"
   authors="cawaMS"
   manager="paulyuk"
   editor="tglee" />

<tags
   ms.service="multiple"
   ms.date="10/08/2015"
   wacn.date="07/07/2016"/>

# 从 Visual Studio 配置与 Service Fabric 群集的安全连接

了解如何使用 Visual Studio 安全访问已配置访问控制策略的 Azure Service Fabric 群集。

## 群集连接类型

Azure Service Fabric 群集支持两种连接：**不安全**连接和**基于 x509 证书**的安全连接。（如果托管在本地环境的 Service Fabric 群集，则还支持 **Windows** 和 **dSTS** 身份验证。） 创建群集时，你必须配置群集连接类型。创建后，无法更改连接类型。

Visual Studio Service Fabric 工具支持所有用于连接到群集来进行发布的身份验证类型。

## 在发布配置文件中配置群集连接

如果你从 Visual Studio 发布 Service Fabric 项目，请通过单击“连接终结点”部分中的“选择”按钮，使用“发布 Service Fabric 应用程序”对话框来选择 Azure Service Fabric 群集。你可以登录 Azure 帐户，然后选择你订阅下面的现有群集。

![“发布 Service Fabric 应用程序”对话框用于配置 Service Fabric 连接。][publishdialog]

“选择 Service Fabric 群集”对话框将自动验证群集连接。如果通过了验证，则表示系统已安装正确的证书，可安全连接到群集。否则即表示群集不安全。验证失败的原因可能是网络问题，或者系统尚未正确配置为连接到安全群集。

![在“选择 Service Fabric 群集”对话框中，你可以配置现有的 Service Fabric 群集连接，或创建并配置新的群集连接。][selectsfcluster]

### 连接到安全群集

1.	请确保你可以访问目标群集所信任的某个客户端证书。证书通常以个人信息交换 (.pfx) 文件的形式共享。

2.	安装受信任的证书。为此，请双击 .pfx 文件，或使用 PowerShell 脚本 Import-PfxCertificate 来导入证书。将证书安装到 **Cert:\\LocalMachine\\My**。导入证书时，可以接受所有默认设置。

3.	在项目的快捷菜单上选择“发布...”命令打开“发布 Azure应用程序”对话框，然后选择目标群集。该工具将自动解析连接，并将安全连接参数保存在发布配置文件中。

4.	[可选]：可以编辑发布配置文件以指定安全群集连接。

    由于你正在手动编辑“发布配置文件”XML 文件以指定证书信息，因此请务必记下证书存储名称、存储位置以及证书指纹。稍后需要为证书的存储名称和存储位置提供这些值。有关详细信息，请参阅 [如何：检索证书的指纹](https://msdn.microsoft.com/zh-cn/library/ms734695(v=vs.110).aspx)。

    可以使用 ClusterConnectionParameters 参数指定连接到 Service Fabric 群集时要使用的 PowerShell 参数。有效的参数是 Connect-ServiceFabricCluster cmdlet 所接受的任何参数。有关可用参数的列表，请参阅 [Connect-ServiceFabricCluster](https://msdn.microsoft.com/zh-cn/library/mt125938.aspx)。

    如果你要发布到远程群集，需要指定该特定群集的相应参数。下面是连接到不安全群集的示例：

    `<ClusterConnectionParameters ConnectionEndpoint="mycluster.chinaeast.chinacloudapp.cn:19000" />`

    下面是连接到基于 x509 证书的安全群集的示例：

    ```
    <ClusterConnectionParameters
    ConnectionEndpoint="mycluster.chinaeast.chinacloudapp.cn:19000"
    X509Credential="true"
    ServerCertThumbprint="0123456789012345678901234567890123456789"
    FindType="FindByThumbprint"
    FindValue="9876543210987654321098765432109876543210"
    StoreLocation="CurrentUser"
    StoreName="My" />
    ```

5.	编辑其他任何所需的设置（例如升级参数和应用程序参数文件位置），然后从 Visual Studio 中的“发布 Service Fabric应用程序”对话框发布应用程序。

## 后续步骤
有关访问 Service Fabric 群集的详细信息，请参阅[使用 Service Fabric 资源管理器可视化群集](/documentation/articles/service-fabric-visualizing-your-cluster/)。

<!--Image references-->
[publishdialog]: ./media/service-fabric-visualstudio-configure-secure-connections/publishdialog.png
[selectsfcluster]: ./media/service-fabric-visualstudio-configure-secure-connections/selectsfcluster.png

<!---HONumber=Mooncake_0321_2016-->