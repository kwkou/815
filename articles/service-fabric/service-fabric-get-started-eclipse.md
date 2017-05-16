<properties
    pageTitle="适用于 Eclipse 的 Azure Service Fabric 插件 | Azure"
    description="适用于 Eclipse 的 Service Fabric 插件入门。"
    services="service-fabric"
    documentationcenter="java"
    author="sayantancs"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="bf84458f-4b87-4de1-9844-19909e368deb"
    ms.service="service-fabric"
    ms.devlang="java"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="04/06/2017"
    wacn.date="05/15/2017"
    ms.author="saysa"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="c2c108f1f39cddada9e2afa04e5da73fd713c4c0"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="service-fabric-plug-in-for-eclipse-java-application-development"></a>使用适用于 Eclipse 的 Service Fabric 插件开发 Java 应用程序
Eclipse 是面向 Java 开发人员的最常用集成开发环境 (IDE) 之一。 本文介绍如何设置适用于 Azure Service Fabric 的 Eclipse 开发环境。 了解如何安装 Service Fabric 插件、创建 Service Fabric 应用程序，以及将 Service Fabric 应用程序部署到 Eclipse Neon 中的本地或远程 Service Fabric 群集。

## <a name="install-or-update-the-service-fabric-plug-in-in-eclipse-neon"></a>在 Eclipse Neon 中安装或更新 Service Fabric 插件
可在 Eclipse 中安装 Service Fabric 插件。 该插件可帮助简化生成和部署 Java 服务的过程。

1.  请确保安装最新版本的 Eclipse Neon 和最新版本的 Buildship（1.0.17 或更高版本）：
    -   若要检查已安装组件的版本，请在 Eclipse Neon 转到“帮助” > “安装详细信息”。
    -   若要更新 Buildship，请参阅 [Eclipse Buildship: Eclipse Plug-ins for Gradle][buildship-update]（Eclipse Buildship：适用于 Gradle 的 Eclipse 插件）。
    -   若要检查并安装 Eclipse Neon 的更新，请转到“帮助” > “检查更新”。

2.  若要安装 Service Fabric 插件，请在 Eclipse Neon 中转到“帮助” > “安装新软件”。
  1.    在“使用”框中，输入 **http://dl.windowsazure.com/eclipse/servicefabric**。
  2.    单击 **“添加”**。
    ![适用于 Eclipse Neon 的 Service Fabric 插件][sf-eclipse-plugin-install]
  3.    选择 Service Fabric 插件，然后单击“下一步”。
  4.    完成安装步骤，然后接受 Microsoft 软件许可条款。

如果已安装 Service Fabric 插件，请确保使用最新版本。 若要检查可用的更新，请转到“帮助” > “安装详细信息”。 在已安装的插件列表中选择“Service Fabric”，然后单击“更新”。 随后将安装可用的更新。

> [AZURE.NOTE]
> 如果安装或更新 Service Fabric 插件时运行缓慢，原因可能是 Eclipse 设置有问题。 Eclipse 将收集有关所有更改的元数据，以更新已注册到 Eclipse 实例的站点。 若要加速 Service Fabric 插件更新的检查和安装过程，请转到“可用软件站点”。 清除所有站点对应的复选框，但指向 Service Fabric 插件位置 (http://dl.windowsazure.com/eclipse/servicefabric) 的站点除外。

## <a name="create-a-service-fabric-application-in-eclipse"></a>在 Eclipse 中创建 Service Fabric 应用程序

1.  在 Eclipse Neon 中，转到“文件” > “新建” > “其他”。 选择“Service Fabric 项目”，然后单击“下一步”。

    ![Service Fabric 新建项目第 1 页][create-application/p1]

2.  输入项目的名称，然后单击“下一步”。

    ![Service Fabric 新建项目第 2 页][create-application/p2]

3.  从模板列表中，选择“服务模板”。 选择服务模板的类型（“执行组件”、“无状态”、“容器”或“来宾二进制”），然后单击“下一步”。

    ![Service Fabric 新建项目第 3 页][create-application/p3]

4.  输入服务名称和服务详细信息，然后单击“完成”。

    ![Service Fabric 新建项目第 4 页][create-application/p4]

5. 创建第一个 Service Fabric 项目时，请在“打开关联的透视图”对话框中单击“是”。

    ![Service Fabric 新建项目第 5 页][create-application/p5]

6.  新项目如下所示：

    ![Service Fabric 新建项目第 6 页][create-application/p6]

## <a name="build-and-deploy-a-service-fabric-application-in-eclipse"></a>在 Eclipse 中生成和部署 Service Fabric 应用程序

1.  右键单击新建的 Service Fabric 应用程序，然后选择“Service Fabric”。

    ![Service Fabric 右键菜单][publish/RightClick]

2. 在子菜单中选择所需的选项：
    -   若要生成应用程序但不清理，请单击“生成应用程序”。
    -   若要生成已清理的应用程序，请单击“重新生成应用程序”。
    -   若要清理已生成项目的应用程序，请单击“清理应用程序”。

3.  还可以通过此菜单部署、取消部署和发布应用程序：
    -   若要部署到本地群集，请单击“部署应用程序”。
    -   在“发布应用程序”对话框中选择发布配置文件：
        -  **Local.json**
        -  **Cloud.json**

     这些 JavaScript 对象表示法 (JSON) 文件存储连接到本地群集或云 (Azure) 群集所需的信息（例如连接终结点和安全信息）。

  ![Service Fabric 发布菜单][publish/Publish]

部署 Service Fabric 应用程序的另一种方式是使用 Eclipse 运行配置。

  1.    转到“运行” > “运行配置”。
  2.    在“Gradle 项目”下面，选择“ServiceFabricDeployer”运行配置。
  3.    在右窗格中的“参数”选项卡上，请为“publishProfile”选择“local”或“cloud”。  默认值为 **local**。 若要部署到远程群集或云群集，请选择“cloud”。
  4.    为了确保在发布配置文件中填充正确的信息，请根据需要编辑 **Local.json** 或 **Cloud.json**。 可以添加或更新终结点详细信息和安全凭据。
  5.    确保“工作目录”指向要部署的应用程序。 若要更改应用程序，请单击“工作区”按钮，然后选择所需的应用程序。
  6.    单击“应用”，然后单击“运行”。

应用程序将在片刻之后生成和部署。 可在 Service Fabric Explorer 中监视部署状态。  

## <a name="add-a-service-fabric-service-to-your-service-fabric-application"></a>将 Service Fabric 服务添加到 Service Fabric 应用程序

若要将 Service Fabric 服务添加到现有的 Service Fabric 应用程序，请执行以下步骤：

1.  右键单击要将该服务添加到的项目，然后单击“Service Fabric”。

    ![Service Fabric 添加服务第 1 页][add-service/p1]

2.  单击“添加 Service Fabric 服务”，完成将服务添加到项目的一系列步骤。
3.  选择要添加到项目的服务模板，然后单击“下一步”。

    ![Service Fabric 添加服务第 2 页][add-service/p2]

4.  输入服务名称（根据需要输入其他详细信息），然后单击“添加服务”按钮。  

    ![Service Fabric 添加服务第 3 页][add-service/p3]

5.  添加该服务后，整个项目结构类似于以下项目：

    ![Service Fabric 添加服务第 4 页][add-service/p4]

## <a name="upgrade-your-service-fabric-java-application"></a>升级 Service Fabric Java 应用程序

假设你在升级方案中使用 Service Fabric 插件在 Eclipse 中创建了 **App1** 项目。 你已使用该插件部署了该项目，以创建名为 **fabric:/App1Application** 的应用程序。 该应用程序的类型为 **App1AppicationType**，应用程序版本为 1.0。 现在，你想要在不影响可用性的情况下升级该应用程序。

首先，请对应用程序进行任何更改，然后重新生成已修改的服务。 使用服务的更新版本（以及相关的代码、配置或数据）更新已修改服务的清单文件 (ServiceManifest.xml)。 此外，使用应用程序和已修改服务的更新版本号修改应用程序的清单 (ApplicationManifest.xml)。  

若要使用 Eclipse Neon 升级应用程序，可以创建重复的运行配置文件。 然后，根据需要使用该文件升级应用程序。

1.  转到“运行” > “运行配置”。 在左窗格中，单击“Gradle 项目”左侧的小箭头。
2.  右键单击“ServiceFabricDeployer”，然后选择“复制”。 输入此配置的新名称，例如 **ServiceFabricUpgrader**。
3.  在右侧面板中的“参数”选项卡上，将 **-Pconfig='deploy'** 更改为 **-Pconfig='upgrade'**，然后单击“应用”。

此过程将创建并保存随时可用于升级应用程序的运行配置配置文件。 此过程还会从应用程序清单文件中获取最近更新的应用程序类型版本。

应用程序升级需要几分钟时间。 可在 Service Fabric Explorer 中监视应用程序升级状态。

<!-- Images -->

[sf-eclipse-plugin-install]: ./media/service-fabric-get-started-mac/sf-eclipse-plugin-install.png

[create-application/p1]:./media/service-fabric-get-started-eclipse/create-application/p1.png
[create-application/p2]:./media/service-fabric-get-started-eclipse/create-application/p2.png
[create-application/p3]:./media/service-fabric-get-started-eclipse/create-application/p3.png
[create-application/p4]:./media/service-fabric-get-started-eclipse/create-application/p4.png
[create-application/p5]:./media/service-fabric-get-started-eclipse/create-application/p5.png
[create-application/p6]:./media/service-fabric-get-started-eclipse/create-application/p6.png

[publish/Publish]: ./media/service-fabric-get-started-eclipse/publish/Publish.png
[publish/RightClick]: ./media/service-fabric-get-started-eclipse/publish/RightClick.png

[add-service/p1]: ./media/service-fabric-get-started-eclipse/add-service/p1.png
[add-service/p2]: ./media/service-fabric-get-started-eclipse/add-service/p2.png
[add-service/p3]: ./media/service-fabric-get-started-eclipse/add-service/p3.png
[add-service/p4]: ./media/service-fabric-get-started-eclipse/add-service/p4.png

<!-- Links -->
[buildship-update]: https://projects.eclipse.org/projects/tools.buildship
<!--Update_Description:update install and configure steps with a new version plugin-->