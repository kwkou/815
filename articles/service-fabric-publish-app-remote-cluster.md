<properties
    pageTitle="使用 Visual Studio 将应用发布到远程群集 | Azure"
    description="了解如何使用 Visual Studio 将应用程序发布到远程 Service Fabric 群集。"
    services="service-fabric"
    documentationCenter="na"
    authors="cawams"
    manager="timlt"
    editor="" />

<tags
    ms.service="multiple"
    ms.date="04/07/2016"
    wacn.date="07/04/2016" />

# 使用 Visual Studio 将应用程序发布到远程群集

适用于 Visual Studio 的 Azure Service Fabric 扩展提供简单、可重复且可编写脚本的方式，用于将应用程序发布到 Service Fabric 群集。

## 发布时所需的项目

### Deploy-FabricApplication.ps1

这是一个 PowerShell 脚本，它使用发布配置文件路径作为参数来发布 Service Fabric 应用程序。由于此脚本是应用程序的一部分，因此，在必要时，你可以根据应用程序对其进行修改。

### 发布配置文件

Service Fabric 应用程序项目中名为 **PublishProfiles** 的文件夹包含 XML 文件，其中存储了用于发布应用程序的基本信息，例如：

- Service Fabric 群集连接参数
- 应用程序参数文件的路径
- 升级设置

默认情况下，应用程序包含两个发布配置文件：Local.xml 和 Cloud.xml。你可以通过复制并粘贴其中一个默认文件，来添加更多的配置文件。

### 应用程序参数文件

的 Service Fabric 应用程序项目中名为 **ApplicationParameters** 的文件夹包含用户指定的应用程序清单参数值 XML 文件。应用程序清单文件可以参数化，以便可以针对部署设置使用不同的值。若要详细了解如何参数化应用程序，请参阅[在 Service Fabric 中管理多个环境](/documentation/articles/service-fabric-manage-multiple-environment-app-configuration/)。

>[AZURE.NOTE] 对于执行组件服务，应该先构建项目，然后尝试使用编辑器或通过“发布”对话框编辑文件。这是因为在构建期间将生成一部分清单文件。

## 使用“发布 Service Fabric 应用程序”对话框发布应用程序

以下步骤演示如何使用 Visual Studio Service Fabric 工具提供的“发布 Service Fabric 应用程序”对话框发布应用程序。

1. 在 Service Fabric 应用程序项目的快捷菜单中，选择“发布...”以查看“发布 Service Fabric 应用程序”对话框。

    ![“发布 Service Fabric 应用程序”对话框][0]

    “目标设置文件”下拉列表中选择的文件是保存除“清单版本”以外的所有设置的位置。你可以重复使用现有配置文件，或通过选择“目标配置文件”下拉列表框中的“<管理配置文件...>”来创建一个新的配置文件。当你选择发布配置文件时，其内容将出现在对话框的相应字段中。若要随时保存更改，请选择“保存配置文件”链接。

2. 在“连接终结点”部分中，指定本地或远程 Service Fabric 群集的发布终结点。若要添加或更改连接终结点，请单击“连接终结点”下拉列表。该列表显示你可以根据 Azure 订阅对其发布的可用 Service Fabric 群集连接终结点。请注意，如果你尚未登录到 Visual Studio，系统将提示你登录。

    使用群集选择对话框从一组可用的订阅和群集中进行选择。

    ![“选择 Service Fabric 群集”对话框][1]

    >[AZURE.NOTE] 如果你要发布到任意终结点（例如合作群集），请参阅下面的“发布到任意群集终结点”部分。

    在你选择终结点后，Visual Studio 将验证与所选 Service Fabric 群集的连接。如果群集未受到保护，Visual Studio 可以立即与其连接。但是，如果群集受保护，则你需要在本地计算机上安装证书才能继续。有关详细信息，请参阅 [How to configure secure connections（如何配置安全连接）](/documentation/articles/service-fabric-visualstudio-configure-secure-connections/)。完成后，选择“确定”按钮。选定的群集将出现在“发布 Service Fabric 应用程序”对话框中。

3. 在“应用程序参数文件”下拉列表框中，导航到应用程序参数文件。应用程序参数文件保留应用程序清单文件中参数的用户指定值。若要添加或更改参数，请选择“编辑”按钮。在“参数”网格中输入或更改参数值。完成后，请选择“保存”按钮。

    ![“编辑参数”对话框][2]

4. 使用“升级应用程序”复选框指定此发布操作是否为升级。升级发布操作不同于一般发布操作。有关差异列表，请参阅 [Service Fabric 应用程序升级](/documentation/articles/service-fabric-application-upgrade/)。若要配置升级设置，请选择“配置升级设置”链接。升级参数编辑器随即会出现。若要详细了解升级参数，请参阅[配置 Service Fabric 应用程序的升级](/documentation/articles/service-fabric-visualstudio-configure-upgrade/)。

5. 选择“清单版本...”按钮以查看“编辑版本”对话框。需要更新应用程序和服务版本才能进行升级。请参阅 [Service Fabric 应用程序升级教程](/documentation/articles/service-fabric-application-upgrade-tutorial/)，以了解应用程序和服务清单版本如何影响升级过程。

    ![“编辑版本”对话框][3]

    如果应用程序和服务版本使用 1.0.0 等语义版本设置，或格式为 1.0.0.0 的数字值，请选择“自动更新应用程序和服务版本”选项。如果你选择此选项，则每当代码、配置或数据包版本更新时，服务和应用程序版本号将自动更新。如果你想要以手动方式编辑版本，请清除该复选框以禁用此功能。

    >[AZURE.NOTE] 对于执行组件项目的所有包条目，请先构建项目以便在服务清单文件中生成这些条目。

6. 指定完所有必要的设置后，选择“发布”按钮以将应用程序发布到选定的 Service Fabric 群集。指定的设置将应用到发布过程。

## 发布到任意群集终结点（包括合作群集）

Visual Studio 发布体验已针对发布到远程群集（与某个 Azure 订阅关联）进行优化。但是，也可以通过直接编辑发布配置文件 XML 发布到任意终结点（例如 Service Fabric 合作群集）。如上所述，默认情况下提供了两个发布配置文件 - **Local.xml** 和 **Cloud.xml** - 但你可以针对不同的环境创建更多的配置文件。例如，你可以创建一个配置文件（例如命名为 **Party.xml**）用于发布到合作群集。

如果要连接到未受保护的群集，只需提供群集连接终结点，例如 `partycluster1.chinaeast.chinacloudapp.cn:19000`。在此情况下，发布配置文件中的连接终结点看起来类似于：

```XML
<ClusterConnectionParameters ConnectionEndpoint="partycluster1.chinaeast.chinacloudapp.cn:19000" />
```

  如果要连接到受保护的群集，则还需要提供本地存储中用于身份验证的客户端证书的详细信息。有关详细信息，请参阅[配置与 Service Fabric 群集的安全连接](/documentation/articles/service-fabric-visualstudio-configure-secure-connections/)。

  设置发布配置文件后，可以在发布对话框中引用它，如下所示。

  ![发布对话框中的新发布配置文件][4]

  请注意，在此情况下，新发布配置文件将指向某个默认的应用程序参数文件。如果你想要将相同的应用程序配置发布到多个环境，这是合理的结果。相反，如果要发布的每个环境有不同的配置，则合理的做法是创建相应的应用程序参数文件。

## 后续步骤

若要了解如何在持续集成环境中自动化发布过程，请参阅[设置 Service Fabric 持续集成](/documentation/articles/service-fabric-set-up-continuous-integration/)。


[0]: ./media/service-fabric-publish-app-remote-cluster/PublishDialog.png
[1]: ./media/service-fabric-publish-app-remote-cluster/SelectCluster.png
[2]: ./media/service-fabric-publish-app-remote-cluster/EditParams.png
[3]: ./media/service-fabric-publish-app-remote-cluster/EditVersions.png
[4]: ./media/service-fabric-publish-app-remote-cluster/publish-to-party-cluster.png

<!---HONumber=Mooncake_0425_2016-->