<properties
    pageTitle="适用于 Azure Service Fabric 的 Eclipse 插件入门 | Azure"
    description="适用于 Azure Service Fabric 的 Eclipse 插件入门。"
    services="service-fabric"
    documentationcenter="java"
    author="sayantancs"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="bf84458f-4b87-4de1-9844-19909e368deb"
    ms.service="service-fabric"
    ms.devlang="java"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="12/27/2016"
    wacn.date="04/24/2017"
    ms.author="saysa"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="b612259b97bf238bac28cb77bf26f684128dd882"
    ms.lasthandoff="04/14/2017" />

# <a name="getting-started-with-eclipse-plugin-for-service-fabric-java-application-development"></a>适用于 Service Fabric Java 应用程序开发的 Eclipse 插件入门
对于 Java 开发人员来说，Eclipse 是最常用的 IDE 之一。 本文介绍如何设置适用于 Service Fabric 的 Eclipse 开发环境。 本文将帮助你安装插件、创建 Service Fabric 应用程序，以及将 Service Fabric 应用程序部署到本地或远程 Service Fabric 群集。

## <a name="install-or-update-service-fabric-plugin-on-eclipse-neon"></a>在 Eclipse Neon 上安装或更新 Service Fabric 插件
Service Fabric 为**适用于 Java 开发人员的 Eclipse IDE** 提供插件，可简化生成和部署 Java 服务的过程。

1. 请确保已安装最新 Eclipse **Neon** 和最新 Buildship（1.0.17 或更高版本）。 可以通过“帮助”>“安装详细信息”检查已安装组件的版本。 可以使用[此处][buildship-update]的说明更新 Buildship。 若要检查 Eclipse Neon 是否为最新版本并根据情况进行更新，可转到“帮助 => 检查更新”。

2. 若要安装 Service Fabric 插件，请选择“帮助 => 安装新软件...”
  1. 在“使用”文本框中，输入：``http://dl.windowsazure.com/eclipse/servicefabric``
  2. 单击“添加”。

  ![适用于 Service Fabric 的 Eclipse Neon 插件][sf-eclipse-plugin-install]

  3. 选择 Service Fabric 插件，然后单击“下一步”。
  4. 继续执行安装并接受最终用户许可协议。

如果已安装 Service Fabric Eclipse 插件，请确保使用的是最新版本。 若要检查是否可以进一步进行更新，可转到“帮助 => 安装详细信息”。 然后，在已安装插件的列表中搜索 Service Fabric 并单击“更新”。 如果存在任何挂起的更新，则会提取并安装该更新。

> [AZURE.NOTE]
> 如果在 Eclipse 上安装或更新 Service Fabric Eclipse 插件耗时很长，则是因为 Eclipse 每次都会尝试提取发生在所有更新站点（已注册到 Eclipse 实例）上的所有新更改的元数据。 因此，若要加快速度，可以使用这样一种小技巧：转到“可用软件站点”，取消选中所有选项，只保留指向 Service Fabric 插件位置 `http://dl.windowsazure.com/eclipse/servicefabric` 的选项。
>

## <a name="create-service-fabric-application-using-eclipse"></a>使用 Eclipse 创建 Service Fabric 应用程序

1. 转到“文件 => 新建 => 其他”。 选择“Service Fabric 项目”。 单击“下一步”。

    ![Service Fabric 新建项目第 1 页][create-application/p1]

2. 为项目提供名称。 单击“下一步”。

    ![Service Fabric 新建项目第 2 页][create-application/p2]

3. 从可用模板集（执行组件、无状态、容器或来宾可执行文件）中选择“服务模板”。 单击“资源组名称” 的 Azure 数据工厂。

    ![Service Fabric 新建项目第 3 页][create-application/p3]

4. 在此页上输入服务名称和/或相关的服务详细信息，然后单击“完成”。

    ![Service Fabric 新建项目第 4 页][create-application/p4]

5. 创建第一个 Service Fabric 项目时，系统会询问是否需要设置 Service Fabric 透视，请选择“是”继续。

    ![Service Fabric 新建项目第 5 页][create-application/p5]

6. 成功创建以后，项目如下所示：

    ![Service Fabric 新建项目第 6 页][create-application/p6]

## <a name="build-and-deploy-the-service-fabric-application-using-eclipse"></a>使用 Eclipse 生成和部署 Service Fabric 应用程序

* 右键单击上述刚创建的 Service Fabric 应用程序。 在上下文菜单中选择“Service Fabric”选项。 此时会显示一个带多个选项的子菜单， 如下所示：

    ![Service Fabric 右键菜单][publish/RightClick]

  单击进行生成、重新生成和清除的选项以后，将会执行预期的操作。
  - “生成应用程序”会在不清除的情况下生成应用程序
  - “重新生成应用程序”会执行应用程序的干净生成
  - “清除应用程序”会清除应用程序中的已生成项目


* 也可选择通过此菜单部署、取消部署以及发布应用程序。
  - “部署应用程序”会部署到本地群集
  - “发布应用程序...”会询问你在 ``Local.json`` 和 ``Cloud.json`` 之间选择哪个发布配置文件。 这些 JSON 文件用于存储连接到本地群集或云 (Azure) 群集所需的信息（例如连接终结点和安全信息）。

  ![Service Fabric 右键菜单][publish/Publish]

* 还可以通过另一种方式，使用 Eclipse 运行配置来部署 Service Fabric 应用程序。

  1. 选择“运行”=>“运行配置”。 在“分级项目”下选择“ServiceFabricDeployer”运行配置。
  2. 在右窗格的“参数”选项卡下，指定“本地”或“云”作为“publishProfile”。 默认设置为“本地”。 若要部署到远程/云群集，请选择“云”。
  3. 相应地编辑 `Local.json` 或 `Cloud.json`，根据情况输入终结点详细信息和安全凭据，确保在发布配置文件中填充正确的信息。
  4. 确保右窗格中“分级项目”下的“工作目录”指向要部署的应用程序。 如果否，则请单击“工作区...”按钮，选择所需应用程序。
  5. 单击“应用”，然后单击“运行”。

应用程序将在片刻之后生成和部署。 可以从 Service Fabric Explorer 监视其状态。  

## <a name="add-new-service-fabric-service-to-your-service-fabric-application"></a>向 Service Fabric 应用程序添加新的 Service Fabric 服务

可以通过以下步骤将新的 Service Fabric 服务添加到现有的 Service Fabric 应用程序：

1. 右键单击要向其添加服务的项目，打开上下文菜单，然后选择“Service Fabric”选项。 此时会显示一个带多个选项的子菜单。

    ![Service Fabric 添加服务第 1 页][add-service/p1]

2. 选择“添加 ServiceFabric 服务”选项，系统会引导你完成后续步骤，将服务添加到项目。
3. 选择要添加到项目的“服务”模板，然后单击“下一步”。

    ![Service Fabric 添加服务第 2 页][add-service/p2]

4. 输入服务名称（以及其他必需的详细信息），然后单击下方的“添加服务”按钮。  

    ![Service Fabric 添加服务第 3 页][add-service/p3]

5. 成功添加服务以后，整个项目结构现在将如下所示：

    ![Service Fabric 添加服务第 4 页][add-service/p4]

## <a name="upgrade-your-service-fabric-java-application"></a>升级 Service Fabric Java 应用程序

假设你已经使用 Service Fabric Eclipse 插件创建 ``App1`` 项目，并在部署时使用该插件创建了名称为 ``fabric:/App1Application``、application-type 为 ``App1AppicationType`` 且 application-version 为 1.0 的应用程序。 现在，你需要在不关闭应用程序的情况下将其升级。

对应用程序进行更改，并重新生成已修改的服务。  使用服务（以及代码、配置或数据，具体视需要而定）的更新版本更新已修改服务的清单文件 (``ServiceManifest.xml``)。 此外，使用应用程序的更新版本号和已修改的服务来修改应用程序的清单 (``ApplicationManifest.xml``)。  

若要使用 Eclipse 升级应用程序，可以执行以下步骤，先创建重复的运行配置，然后根据需要用其升级应用程序：
1. 选择“运行”=>“运行配置”。 单击左窗格中“分级项目”左侧的小箭头。
2. 右键单击“ServiceFabricDeployer”，然后选择“复制”。 为此配置提供一个新名称，例如 **ServiceFabricUpgrader**。
3. 在右侧面板的“参数”选项卡下，将“-Pconfig='deploy'”更改为“-Pconfig=upgrade”，然后单击“应用”。
4. 现在，你已创建和保存用于升级应用程序的运行配置，该配置可以按需**运行**。 该配置将用于从 application-manifest 文件中获取最近更新的 application-type 版本。

现在，可以使用 Service Fabric Explorer 监视应用程序升级了。 在几分钟后，应用程序将得到更新。

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