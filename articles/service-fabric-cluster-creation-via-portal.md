
<!--preview portal-->
<properties
   pageTitle="从 Azure 门户预览创建 Service Fabric 群集 | Azure"
   description="从 Azure 门户预览创建 Service Fabric 群集。"
   services="service-fabric"
   documentationCenter=".net"
   authors="ChackDan"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="05/02/2016"
   wacn.date="07/04/2016"/>


# 从 Azure 门户预览创建 Service Fabric 群集

本页面帮助你设置 Azure Service Fabric 群集。订阅必须有足够的核心用于部署将构成此群集的 IaaS VM。


## 搜索 Service Fabric 群集资源

1. 登录到 [Azure 门户预览](https://portal.azure.cn/)。

2. 单击“+ 新建”添加新的资源模板。在“全部”下面的“应用商店”中搜索你的模板，该模板名为“Service Fabric 群集”。

    a.单击顶层的“应用商店”。

    b.在“全部内容”下面输入“Fabric”并按 Enter。有时自动筛选器无法工作，因此请务必按 Enter。![在 Azure 门户预览上搜索 Service Fabric 群集模板的屏幕截图。][SearchforServiceFabricClusterTemplate]

3. 从列表中选择“Service Fabric 群集”。

4. 导航到“Service Fabric 群集”边栏选项卡，然后单击“创建”。

5. 现在你将看到“创建 Service Fabric 群集”边栏选项卡列出了 4 个步骤。

## 步骤 1 - 基本信息

在“基本信息”边栏选项卡中，需要提供群集的基本详细信息。

1. 输入群集的名称。

2. 选择 VM 远程桌面的“用户名”和“密码”。

3. 务必选择要将群集部署到的“订阅”，尤其是在拥有多个订阅时。

4. 创建“新的资源组”，最好让它与群集同名，这样稍后就可以轻松找到它们，在尝试更改部署或删除群集时非常有用。

    >[AZURE.NOTE] 尽管你可以决定使用现有资源组，但最好还是创建新的资源组。这样可以轻松删除不需要的群集。

 	![创建新资源组的屏幕截图。][CreateRG]


5. 从下拉列表中选择一个**位置**。默认值为“中国东部”。按“确定”。

## 步骤 2 - 配置群集

10. 首先让我介绍**节点类型**是什么。节点类型相当于云服务中的角色。节点类型定义 VM 大小、VM 数目及其属性。群集可以有不只一个节点类型，但主节点类型（在门户定义的第一个节点类型）必须至少有 5 个 VM。这是 Service Fabric 系统服务放置到的节点类型。确定是否需要多个节点类型时，请考虑以下因素。

	* 你要部署的应用程序包含前端服务和后端服务。想要将前端服务放在较小型的 VM 上（D2 等 VM 大小）上，并且此 VM 的端口向 Internet 开放，同时你想要将计算密集型后端服务放在非面向 Internet 的较大型 VM 上（D4、D6、D15 等 VM 大小）。

	* 尽管可以将这两种服务都放同一个节点类型上，但我们建议将它们分别放在群集中的两个节点类型上。每个节点类型都可以有不同的属性，例如 Internet 连接、VM 大小，以及能单独调整的 VM 数目。

	* 先定义一个至少包含 5 个 VM 的节点类型。其他节点类型可以至少包含 1 个 VM。

13.  配置节点类型：

	a.选择节点类型的名称（1 到 12 个字符，只能包含字母和数字）。

	b.主节点类型的 VM 大小下限取决于你为群集选择的持久性层。持久性层的默认值为 Bronze。请仔细阅读有关如何[选择 Service Fabric 群集可靠性和持久性](/documentation/articles/service-fabric-cluster-capacity)的文档。

	b.选择 VM 大小/定价层。默认值为“D4 标准”，但如果只是要用此群集来测试自己的应用程序，也可以选择“D2”或任何较小的 VM。

	c.主节点类型的 VM 数目下限取决于选择的可靠性层。可靠性层的默认值为 Silver。请仔细阅读有关如何[选择 Service Fabric 群集可靠性和持久性](/documentation/articles/service-fabric-cluster-capacity)的文档。

	c.选择节点类型的 VM 数目。可在以后扩展或缩减节点类型中的 VM 数目，但数目下限取决于你选择的可靠性层。其他节点类型的下限可以是 1 个 VM。


  	![创建节点类型的屏幕截图。][CreateNodeType]

9. 如果你打算立刻将应用程序部署到群集，请在“应用程序端口”节点类型（或创建的节点类型）上添加要为应用程序开放的端口。稍后可以再为节点类型添加端口，方法是修改与该此节点类型关联的负载平衡器。（添加探测，然后将探测添加到负载平衡器规则。） 立即执行此操作相对来说比较简单，因为门户自动化功能会将所需的探测和规则添加到负载平衡器。

	a.可以在服务清单中查找属于应用程序包的应用程序端口。转到每个应用程序，打开服务清单，然后记下应用程序与外界通信时所需的输入终结点。

	b.在“应用程序输入终结点”字段中添加所有端口（以逗号分隔）。TCP 客户端连接终结点的默认值为 19000，因此不需要指定该值。例如，示例应用程序 WordCount 需要打开端口 83。可以在应用程序包的 servicemanifest.xml 文件中找到该示例应用程序。（其中可能包含多个 servicemanifest.xml 文件。）

    c.大多数示例应用程序都使用端口 80 和 8081，因此如果你打算将示例部署到此群集，请添加这两个端口。![端口][Ports]

10. 不需要配置“放置属性”，因为系统添加了“NodeTypeName”的默认放置属性。如果应用程序有需要，可以添加更多属性。

11. 你不需要配置“容量属性”，不过我们建议你配置，因为你可以在应用程序中使用它将负载报告给系统，从而影响系统在 Service Fabric 群集中所做的位置和资源平衡决策。请先阅读[此文档](/documentation/articles/service-fabric-cluster-resource-manager-architecture)，以了解有关 Service Fabric 资源平衡的详细信息。

12. 针对所有节点类型继续执行上述步骤。

14. 配置群集**诊断**。默认情况下，已在群集上启用诊断，以帮助排查问题。如果你要禁用诊断，请将其“状态”切换为“关闭”。**不**建议关闭诊断。

15. 可选：设置“Service Fabric 群集设置”。你可以使用此高级选项来更改 Service Fabric 群集的默认设置。建议不要更改默认配置，除非你确定自己的应用程序或群集有此需要。



## 步骤 3 - 配置安全性

目前，Service Fabric 只支持通过 X509 证书保护群集。在开始此过程之前，必须先将证书上载到密钥保管库。有关如何执行此操作的详细信息，请参阅 [Service Fabric cluster security（Service Fabric 群集安全性）](/documentation/articles/service-fabric-cluster-security)。

保护群集的选项是可选的，但强烈建议这样做。如果选择不保护群集，请将“安全模式”切换为“不安全”。请注意 - 以后你**无法**将不安全的群集更新为安全群集。

有关安全注意事项和说明的文档，请参阅 [Service Fabric cluster security（Service Fabric 群集安全性）](/documentation/articles/service-fabric-cluster-security)。

![Azure 门户预览上安全配置的屏幕截图。][SecurityConfigs]


## 步骤 4 - 完成群集创建

如要完成群集创建过程，请单击“摘要”查看你提供的配置，或下载将用于部署群集的 Azure Resource Manager 模板。在提供所有必需的设置后，“确定”按钮将会启用，你只需单击它就能启动群集创建过程。

可以在通知栏中查看群集创建进度。（单击屏幕右上角状态栏附近的铃铛图标）。 如果在创建群集时曾经单击“固定到启动板”，现在会看到“部署 Service Fabric 群集”已固定到**开始**面板。

![显示“正在部署 Service Fabric 群集”的开始面板屏幕截图。][Notifications]

## 查看群集状态

创建群集后，你可以在门户检查群集：

1. 转到“浏览”，然后单击“Service Fabric 群集”。

2. 查找你的群集并单击它。![在门户中查找群集的屏幕截图。][BrowseCluster]

3. 现在仪表板将显示群集的详细信息，包括群集的公共 IP 地址。将鼠标光标移到“群集公共 IP 地址”上会显示剪贴板，单击它可以复制该 IP 地址。![仪表板中群集详细信息的屏幕截图。][ClusterDashboard]

  群集仪表板边栏选项卡上的“节点监视器”部分显示运行正常和不正常的 VM 的数目。若要了解有关群集运行状况的详细信息，请参阅 [Service Fabric health model introduction（Service Fabric 运行状况模型简介）](/documentation/articles/service-fabric-health-introduction)。

>[AZURE.NOTE] Service Fabric 群集需要有一定数量的节点可随时启动，以保持可用性和状态 - 称为“维持仲裁”。因此，除非你已事先执行[状态的完整备份](/documentation/articles/service-fabric-reliable-services-backup-restore)，否则关闭群集中的所有计算机通常是不安全的做法。

## 连接到群集并部署应用程序

创建群集后，可以连接到群集，并开始部署应用程序。首先，请在已安装 Service Fabric SDK 的计算机上启动 Windows PowerShell。然后，若要连接到群集，请根据创建的是安全群集还是不安全群集，运行以下 PowerShell 命令集之一：

### 连接到不安全的群集

```powershell
Connect-serviceFabricCluster -ConnectionEndpoint <Cluster FQDN>:19000 -KeepAliveIntervalInSec 10
```

### 连接到安全群集

1. 运行以下命令，以便在即将用于运行“Connect-serviceFabricCluster”PowerShell 命令的计算机上设置证书。

    ```powershell
    Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\My `
            -FilePath C:\docDemo\certs\DocDemoClusterCert.pfx `
            -Password (ConvertTo-SecureString -String test -AsPlainText -Force)
    ```

2. 运行以下 PowerShell 命令连接到安全群集。证书的详细信息与门户上提供的信息相同。

    ```powershell
    Connect-serviceFabricCluster -ConnectionEndpoint <Cluster FQDN>:19000 `
              -KeepAliveIntervalInSec 10 `
              -X509Credential -ServerCertThumbprint <Certificate Thumbprint> `
              -FindType FindByThumbprint -FindValue <Certificate Thumbprint> `
              -StoreLocation CurrentUser -StoreName My
    ```

    例如，上述 PowerShell 命令应该类似于以下内容：

    ```powershell
    Connect-serviceFabricCluster -ConnectionEndpoint sfcluster4doc.chinaeast.chinacloudapp.cn:19000 `
              -KeepAliveIntervalInSec 10 `
              -X509Credential -ServerCertThumbprint C179E609BBF0B227844342535142306F3913D6ED `
              -FindType FindByThumbprint -FindValue C179E609BBF0B227844342535142306F3913D6ED `
              -StoreLocation CurrentUser -StoreName My
    ```

### 部署应用
现在你已创建连接，请运行以下命令来部署应用程序，但别忘了要先用计算机上适当的路径来替换以下所示的路径。以下示例将部署单词计数示例应用程序：

1. 将包复制到前面连接到的群集。

    ```powershell
    $applicationPath = "C:\VS2015\WordCount\WordCount\pkg\Debug"
    ```

    ```powershell
    Copy-ServiceFabricApplicationPackage -ApplicationPackagePath $applicationPath -ApplicationPackagePathInImageStore "WordCount" -ImageStoreConnectionString fabric:ImageStore
    ```
2. 向 Service Fabric 注册应用程序类型。

    ```powershell
    Register-ServiceFabricApplicationType -ApplicationPathInImageStore "WordCount"
    ```

3. 针对刚刚注册的应用程序类型创建新实例。

    ```powershell
    New-ServiceFabricApplication -ApplicationName fabric:/WordCount -ApplicationTypeName WordCount -ApplicationTypeVersion 1.0.0.0
    ```

4. 现在，请打开所选的浏览器，然后连接到应用程序正在侦听的终结点。WordCount 示例应用程序的 URL 如下所示：

    http://sfcluster4doc.westus.chinaeast.chinacloudapp.cn:31000

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->

## 远程连接到虚拟机缩放集实例或群集节点

每次在群集中指定 NodeTypes，都会设置 VM 缩放集。有关详细信息，请参阅[远程连接到 VM 缩放集实例](/documentation/articles/service-fabric-cluster-nodetypes#remote-connect-to-a-vm-scale-set-instance-or-a-cluster-node)。

## 后续步骤

创建群集后，请了解有关保护群集和部署应用的详细信息：

- [在 Visual Studio 中管理 Service Fabric 应用程序](/documentation/articles/service-fabric-manage-application-in-visual-studio)
- [Service Fabric 群集安全性](/documentation/articles/service-fabric-cluster-security)
- [Service Fabric 运行状况模型简介](/documentation/articles/service-fabric-health-introduction)


<!--Image references-->
[SearchforServiceFabricClusterTemplate]: ./media/service-fabric-cluster-creation-via-portal/SearchforServiceFabricClusterTemplate.png
[CreateRG]: ./media/service-fabric-cluster-creation-via-portal/CreateRG.png
[CreateNodeType]: ./media/service-fabric-cluster-creation-via-portal/NodeType.png
[Ports]: ./media/service-fabric-cluster-creation-via-portal/ports.png
[SFConfigurations]: ./media/service-fabric-cluster-creation-via-portal/SFConfigurations.png
[SecurityConfigs]: ./media/service-fabric-cluster-creation-via-portal/SecurityConfigs.png
[Notifications]: ./media/service-fabric-cluster-creation-via-portal/notifications.png
[BrowseCluster]: ./media/service-fabric-cluster-creation-via-portal/browse.png
[ClusterDashboard]: ./media/service-fabric-cluster-creation-via-portal/ClusterDashboard.png
[SecureConnection]: ./media/service-fabric-cluster-creation-via-portal/SecureConnection.png

<!---HONumber=Mooncake_0523_2016-->