<properties
	pageTitle="MyDriving Azure IoT 示例：生成 | Azure"
	description="生成一个应用，全面演示如何通过使用 Azure（包括流分析、机器学习和事件中心）构建 IoT 系统。"
	services=""
 	documentationCenter=".net"
    	suite=""
	authors="harikmenon"
	manager="douge"/>  


<tags
	ms.service="iot-suite"
	ms.date="03/25/2016"
	wacn.date="08/22/2016"/>  



# 生成 MyDriving 解决方案并将其部署到你的环境

MyDriving 是一个物联网 (IoT) 解决方案，它可收集你汽车的数据并通过使用机器学习对其进行处理，然后将数据显示你的移动电话上。后端包含 Azure 提供的多种服务。客户端可以是 Android、iOS 或 Windows 10 手机。

我们创建了 MyDriving 解决方案以帮助你快速开始创建自己的 IoT 系统。你可以从 [MyDriving repository on GitHub（GitHub 上的 MyDriving 存储库）](https://github.com/Azure-Samples/MyDriving)获取 Azure Resource Manager 脚本，以将后端体系结构部署到你自己的 Azure 帐户。然后，你可以重新配置不同的服务，并修改查询，以遍实现适应你自己数据等用途。你可以在MyDriving 存储库中找到这些脚本，以及移动应用和 Azure App Service API 项目等的代码。

如果你还没有尝试使用该应用，请查看[入门指南](/documentation/articles/iot-solution-get-started/)。

[MyDriving Reference Guide（MyDriving 参考指南）](http://aka.ms/mydrivingdocs)中提供了有关体系结构的详细帐户。总之，我们设置了几个部分，你可通过设置这些部分来创建类似的项目：

* 一个**客户端应用**在 Android、iOS 和 Windows 10 手机上运行。我们使用 Xamarin 平台共享大量代码，这些代码存储在 GitHub 上的 `src/MobileApp` 下。该应用实际上将执行两个不同的功能：
 * 它从车载诊断系统 (OBD) 设备和其自己的位置服务向系统的云后端中继遥测。
 * 它是一个用户界面，用户可在其中查询其录制的公路行程。
* **云服务**实时摄取公路旅行数据并进行处理。创建此服务的主要工作是选择、参数化以及连接各种 Azure 服务。某些部分需要脚本来筛选和处理传入的数据。我们使用 Azure Resource Manager 模板来配置所有部分。
* **移动服务应用**是设备应用的用户界面部分后的 Web 服务。它的主要工作是查询由已存储并处理的数据组成的数据库。它的代码在 GitHub 上的 `src/MobileAppService` 下。
* **Visual Studio with Xamarin** 是我们的开发环境。Xamarin 同时作为 Visual Studio 的组件和独立的集成开发环境 (IDE)，可用于构建跨平台设备代码。若要生成 iOS 代码，必须具有在 OS X 计算机上运行的 Xamarin 的实例。如有需要，它可以作为从 Visual Studio 托管的代理运行。
* 设备应用的**单元测试**在 Xamarin 测试云中执行。
* **GitHub** 是一个存储库，我们在其中存储所有代码、脚本和模板。
* **Visual Studio Team Services** 是用于管理持续生成及测试 Web 服务和设备应用的云服务。
* **HockeyApp** 用于分发设备代码的发布。它还将收集故障和使用情况报告及用户反馈。
* **Visual Studio Application Insights** 监视移动 Web 服务。

让我们看如何设置以上所有内容。请注意，很多步骤都是可选的。

## 注册帐户

-   [Visual Studio Dev Essentials](https://www.visualstudio.com/products/visual-studio-dev-essentials-vs.aspx)。此免费计划提供了对许多开发人员工具和服务（包括 Visual Studio、Visual Studio 团队服务和 Azure）的轻松访问。它为你提供在 $25/月的 Azure 信用额度，共计 12 个月。它还包括对 Pluralsight 培训和 Xamarin University 的订阅。你还可以分别注册 [Azure](https://azure.com) 和 [Visual Studio Team Services](https://www.visualstudio.com/products/visual-studio-team-services-vs.aspx) 的免费层，但这些不提供 Azure 信用额度。

-   [HockeyApp](https://rink.hockeyapp.net/)（可选），用于管理移动应用的测试分发和收集遥测。

-   [Xamarin](https://xamarin.com/)（必需），用于构建移动应用，以及在 [Xamarin 测试云](https://xamarin.com/test-cloud)上运行调试运行和测试。

-   [GitHub](https://github.com/Azure-Samples/MyDriving/)（可选），用于为你自己的代码创建免费的公共存储库（专用存储库需要付费）。或者，可以使用 Visual Studio Team Services 中的基本计划来创建专用存储库。

-   [Power BI](https://powerbi.microsoft.com/)（可选），用于创建整个系统内的数据的丰富可视化效果。

> [AZURE.NOTE] 不需要 GitHub 帐户即可在 [GitHub MyDriving 存储库](https://github.com/Azure-Samples/MyDriving)中访问 MyDriving 代码。

## 安装开发工具

以下设置用于使用 Azure 后端开发完整的解决方案：iOS、Android 和 Windows 10 移动版跨平台应用。

或者，如果你未使用 Azure 后端，则也可以在 Mac 或 Windows 上使用 Xamarin Studio 来开发移动应用。

此处提供了[此设置的详细说明](https://msdn.microsoft.com/zh-cn/library/mt613162.aspx)。

### Windows 开发计算机

Windows 上的中心工具是 Visual Studio，可与适用于 Android 和 Windows 的 MyDriving 应用、App Service API 项目和微服务扩展配合使用。

Xamarin、Git、仿真程序和其他有用的组件都与 Visual Studio 集成。

安装：

-   [Visual Studio 2015 with Xamarin](https://www.visualstudio.com/products/visual-studio-community-vs)（任何版本--社区版免费）。

-   [SQLite for Universal Windows Platform（面向通用 Windows 平台的 SQLite）](https://visualstudiogallery.msdn.microsoft.com/4913e7d5-96c9-4dde-a1a1-69820d615936)。需要构建 Windows 10 移动版代码。

-   [用于 Visual Studio 2015 的 Azure SDK](https://go.microsoft.com/fwlink/?linkid=518003&clcid=0x409)。为你提供用于在 Azure 中运行应用的 SDK 以及用于管理 Azure 的命令行工具。

-   [Azure Service Fabric SDK](http://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric)。对生成[微服务](/documentation/articles/service-fabric-get-started/)扩展来说是必需的。

此外，请确保具有正确的 Visual Studio 扩展。在“工具”下查看，你将看到“Android、iOS、Xamarin...”。如果没有，请打开控制面板，然后选择“程序和功能”>“Microsoft”>“Visual Studio 2015”>“修改”。在“跨平台开发”下，选择“C# /.Net (Xamarin)”。此时，请检查已安装 **GitHub**。

### Mac 开发计算机

如果你想要针对 iOS 进行开发，Mac（Yosemite 或更高版本）是必需的。尽管我们在 Windows 上使用 Visual Studio with Xamarin 来开发和管理的所有代码，Xamarin 仍会使用 Mac 上安装的代理，以便生成 iOS 代码并对其签名。

![在 Windows 上开发，在 Mac 上生成](./media/iot-solution-build-system/image1.png)

（或者，你可以直接在 Mac 上使用 Xamarin Studio 来开发跨平台应用。）

如果不想将 iOS 作为目标平台包含在内，则不需要 Mac。

安装：

-   [Xamarin Studio for iOS](https://developer.xamarin.com/guides/ios/getting_started/installation/mac/)。你也可以在运行 Windows 虚拟机的 Mac 上设置 Visual Studio 和 Xamarin。请参阅 MSDN 上的 [Mac 用户的设置、安装、和验证](https://msdn.microsoft.com/zh-cn/library/mt488770.aspx)。

-   [Azure 开发工具](/downloads/)（可选）。

在 Mac 上启用远程登录。打开“系统首选项”>“共享”，然后选择“远程登录”。

当你在 Windows 上的 Visual Studio 中打开一个 iOS 项目时，Xamarin 插件将提示你提供 Mac 的 ID。

## 提取 GitHub 存储库

通过使用 GitHub、Visual Studio 或另一个 Git 客户端中的“下载 ZIP”按钮，提取 [GitHub MyDriving 存储库](https://github.com/Azure-Samples/MyDriving)的本地副本。

将文件解压缩到使用短路径名的文件夹，如 C:\\code。

或者，如果你想要了解我们的最新代码或为其做出贡献，请克隆存储库，如下所示：

**git clone https://github.com/Azure-Samples/MyDriving.git**

## 获取必应地图 API 密钥

[注册必应地图 API 密钥](https://msdn.microsoft.com/zh-cn/library/ff428642.aspx)。

你需要在 `src/MobileApps/MyDriving/MyDriving.Utils/Logger.cs` 中的第 22 行中将其替换。



## 构建演示应用

在 Visual Studio 中打开以下解决方案：

-   src\MobileApps\MyDriving.sln

-   src\MobileAppService\MyDrivingService.sln

-   src\Extensions\ServiceFabric\VINLookUpApplication\VINLookUpApplication.sln

你将收到提示：

-   信任某些可能不受信任的项目。如果想要继续操作，请选择打开它们。

-   如果正在使用新的 Windows 10 计算机，请设置开发人员模式。

-   提供你的 Xamarin 凭据。

-   连接到 Xamarin Mac。如果你没有 Mac，请右键单击 Visual Studio 中的 iOS 项目，然后选择“卸载项目”。

重新生成解决方案。

如果在生成时遇到问题，请尝试我们发现的异常解决方案：

-   未加载 VINLookupApplication 项目：请确保你已安装 [用于 Visual Studio 2015 的 Azure SDK](https://go.microsoft.com/fwlink/?linkid=518003&clcid=0x409)。

-   未生成 Service Fabric 项目：首先生成接口项目，并确保你已安装 Service Fabric SDK。

-   未生成 Android 应用：

    -   打开“工具”>“Android”>“Android SDK 管理器”，并确保安装了 Android 6 (API 23)/SDK 平台。

    -   删除此目录，然后重新生成：<br/>
        `%LocalAppData%\Xamarin\zips`

## 了解代码

在解决方案中，你将发现：

-   Azure 扩展：Service Fabric。

-   Azure HDInsight：用于在 Azure 中处理行程数据的脚本。

-   Mobile Apps：设备应用。

-   MobileAppsService/MyDrivingService：Web 后端。

-   Power BI：报表定义。

-   脚本：

    -   Resource Manager：用于生成 Azure 资源的模板。

    -   PowerShell：运行 Resource Manager 模板的脚本。

    -   Azure SQL 数据库：调试数据库。

-   SQL 数据库：CreateTables：架构定义。

-   Azure 流分析：转换传入数据流的查询。

## 在开发模式下运行应用

基于你正在使用的设备，执行操作以运行应用：

-  后端：将 MyDrivingService 设置为启动项目，并按 F5 来运行后端 Web 服务。它将打开 API 列表的浏览器视图。

-  移动客户端：[mobile apps are developed in Xamarin（在 Xamarin 中开发移动应用）](https://developer.xamarin.com/guides/cross-platform/deployment,_testing,_and_metrics/debugging_with_xamarin/)。
 -  Android：有关详细信息，请参阅 [Debugging in Xamarin（在 Xamarin 中调试）](http://developer.xamarin.com/guides/android/deployment,_testing,_and_metrics/debugging_with_xamarin_android/)。

 -  iOS：有关详细信息，请参阅 [Debugging in iOS（在 iOS 中调试）](http://developer.xamarin.com/guides/ios/deployment,_testing,_and_metrics/debugging_in_xamarin_ios/)。

 -  Windows Phone：有关详细信息，请参阅 [Xamarin + Windows Phone](https://developer.xamarin.com/guides/cross-platform/windows/phone/)。

## 将移动应用上传到 HockeyApp

HockeyApp 管理 Android、iOS 或 Windows 应用的分发，以测试用户并通知用户新的发布。它还会收集有用的故障报告、包含屏幕截图的用户反馈以及使用情况指标。

[首先上传](http://support.hockeyapp.net/kb/app-management-2/how-to-create-a-new-app)你的生成应用。然后从你的开发计算机登录 [HockeyApp](https://rink.hockeyapp.net)。在开发人员仪表板上，单击“新建应用”，然后将生成的文件拖动到窗口中。（以后，你可以使用你的生成服务自动完成此操作。）

现在，你已在应用仪表板中。

![应用仪表板上的概述选项卡](./media/iot-solution-build-system/image2.png)

对运行你的应用的每个平台重复此过程。然后可以执行以下操作：

-  在仪表板中使用[应用 ID](http://support.hockeyapp.net/kb/app-management-2/how-to-find-the-app-id) 来从应用发送故障数据和反馈。在 MyDriving 中，在 src/MobileApps/MyDriving/MyDriving.Utils/Logger.cs 中更新 ID。

-  [邀请测试用户](http://support.hockeyapp.net/kb/app-management-2/how-to-invite-beta-testers)。获取用于招聘测试人员用户的 URL。他们将能够注册你的团队、下载应用以及向你发送反馈。

-  如果你希望 beta 版本更加开放，请将分发设置为公用。单击“管理应用”>“分发”>“下载 = 公用”。现在，任何人都可以下载你的应用和向你发送反馈，并且在你发布新版本时，他们将看到通知。你也可以获取他们发出的故障报告。

    ![仪表板上的团队](./media/iot-solution-build-system/image3.png)

-  [Link crash reports to Visual Studio Team Services（将故障报告链接到 Visual Studio Team Services）](http://support.hockeyapp.net/kb/third-party-bug-trackers-services-and-webhooks/how-to-use-hockeyapp-with-visual-studio-team-services-vsts-or-team-foundation-server-tfs)。单击“管理应用”>“Visual Studio Team Services”。当接收到故障报告或反馈时，HockeyApp 可以在 Team Services 中自动创建工作项。

有关详细信息，请访问 [HockeyApp 站点](https://hockeyapp.net)。

## 在 Xamarin 测试云上测试移动应用

[Xamarin 测试云](https://developer.xamarin.com/guides/testcloud/introduction-to-test-cloud/)将自动在云中进行真实设备上的 UI 测试。通过使用 NUnit 框架，你可以编写通过用户界面运行你的应用的测试。

若要使用 Xamarin，你需要将 [Xamarin.UITests](https://developer.xamarin.com/guides/testcloud/uitest/intro-to-uitest/) SDK 合并到你的应用中（作为 NuGet 包）。你将在演示应用发现它，当你使用 Xamarin 模板创建新测试项目时，它也包含在其中。

![在何处找到接口上的跨平台 SDK](./media/iot-solution-build-system/image4.png)

示例测试项目将包含在存储库中的应用中。在 [MyDriving](https://github.com/Azure-Samples/MyDriving/tree/master/src/MobileAppService) 中，在 [src](https://github.com/Azure-Samples/MyDriving/tree/master/src)/MobileApps/[MyDriving](https://github.com/Azure-Samples/MyDriving/tree/master/src/MobileApps/MyDriving)/MyDriving.UITests/ 下查看。

如果你使用 Visual Studio Team Services 生成，则可以轻松编写 Xamarin UI 单元测试并将它们作为生成的一部分运行。

## 部署 Azure 服务

若要执行 Azure 服务和 Team Services 生成服务的自动部署，请参阅 **scripts/README.md** 中的详细说明。

Azure 提供了大量可用于生成云应用程序的不同服务。尽管许多服务可以独立使用（如 App Service/Web Apps），但当它们互连形成集成系统（如我们在 MyDriving 中使用的系统）时才具有最佳性能。

可以手动创建和互连 Azure 服务，但使用 Azure Resource Manager 模板更加快速可靠。[Resource Manager](/documentation/articles/resource-group-overview/) 将自动部署解决方案的资源，并使它们互连。

你将在 GitHub 存储库中的 [scripts/ARM](https://github.com/Azure-Samples/MyDriving/tree/master/scripts/ARM) 下找到 MyDriving 系统的模板。它可让你全面准确地查看不同服务在体系结构中的互连方式。我们还将在 [MyDriving Reference Guide（MyDriving 参考指南）](http://aka.ms/mydrivingdocs)中对这些内容进行详细介绍，但仅通过通读模板本身便可学习很多相关内容。

> [AZURE.NOTE] 大多数 Azure 服务具有关联的费用，具体取决于定价层。如果你是 Azure 的新用户，则可以[试用](/pricing/1rmb-trial/)。但是，如果你不打算使用 MyDriving 系统中的某些组件，请确保将其删除以避免产生费用。本文后面的“估计运营成本”一节提供了典型服务费用的摘要。

### 编辑模板

若要自定义部署，可能要删除不需要的组件或添加其他组件，首先制作 scenario\_complete.params.json 和 scenario\_complete.json 的副本，并在其中进行更改。

你可以使用 scenario\_complete.params.json 文件来替代各种默认值，如服务 SKU 或存储复制类型，如下表中所述。默认值将选择成本最低的选项。

| **参数** | **说明** | **默认值** |
|--------|---------|-------|
| IoT 中心 SKU | Azure IoT 中心服务层 | F1 |
| 存储帐户类型 | 存储复制类型 | 标准 LRS |
| SQL 服务目标 | 并发槽使用量 | DW100 |
| 托管计划 SKU | App Service 的服务计划 | F1 |

在 scenario\_complete.json 中：

-   搜索“baseName”并将其更改为你喜欢的名称。

-   搜索“Create”。以下各部分均将创建一个资源。

-   将 sqlServerAdminLogin 和 sqlServerAdminPassword 设置为合适的值。

-   删除创建资源的部分之前，通过在文件中的其他位置搜索它的名称来检查它是否具有依赖项。请注意，创建服务的每一部分都包含列出其依赖关系的 dependsOn 部分。

下面是该模板配置的内容。有关详细信息，请参阅[参考指南](http://aka.ms/mydrivingdocs)。

| **服务** | **说明和详细信息**  
|---|----
| 存储帐户 | 该模板将创建三个帐户：                                                                                                                                                                       
|| - 从流分析接收聚合的遥测的 SQL 数据库，充当通过 API 终结点公开此数据的 Azure App Service 表的后备存储。                      
|| - 从另一个流分析作业累积历史数据的 Blob 存储，将由 HDInsight 处理。                                                                                         
|| - SQL 数据库，接收由 HDInsight 处理以供 Power BI 使用的结果。                                                                                                                 
| Azure IoT 中心 | 建立与每个连接的设备的双向连接。在 MyDriving 解决方案中，移动应用充当现场网关，将数据发送到 Azure IoT 中心。然后 Azure IoT 中心将作为流分析的输入。 |
| Azure 事件中心 | 流分析作业的输出，将输出加入使用 Azure Service Fabric 创建的扩展队列。                                                                                               
| Azure SQL 数据仓库 |                                                                                                                                                                                                            
| 流分析作业 | 将输入和输出与查询连接起来，可用于为 App Service API、扩展和 Power BI 聚合实时和历史数据。                               
| Service Fabric 托管计划 | 用于扩展。                                                                                                                                                                                            
| App Service（“移动应用”） | 托管为移动应用提供终结点的 Mobile Apps API 项目。必须从 Visual Studio 向 App Service 部署 API 代码。                                                         
| 警报规则 | 如果应用响应指示失败，则将向你发送电子邮件。                                                                                                                                            
| Application Insights | 用于在 App Service 中监视 API 的性能。必须在 Visual Studio 中配置连接。                                                                                          
| Azure 密钥保管库 | 用于保存 Web 服务群集证书。                                                                                                                                                                

### 运行模板

**scripts/README.md** 中提供了有关运行模板的详细说明。

若要通过使用脚本来在你自己的 Azure 帐户中预配所有这些服务，请执行以下操作之一：

-   使用 PowerShell：

    ```

    cd scripts/PowerShell;
    deploy.ps1 *location* *resourceGroupName*
    ```

 -   location 是指 Azure 位置，如 `China East` 或 `China North`。使用 `Get-AzureLocation` 可查找可用位置的列表。

 -   resourceGroupName 是你想要为组（所有资源将都属于该组）提供的名称。当完成对资源的操作时，可以通过删除此组来一起删除所有这些资源。

-   使用 Bash 运行 DeploymentScripts/Bash/deploy.sh。

-   打开并生成 Visual Studio 解决方案 DeploymentScripts/VS/DeployARM.sln。

请注意，每当运行该模板时，都会创建一组具有新名称的新资源。若要删除这些资源，请转到门户预览并删除该资源组。

如果脚本因任何原因失败，你可以重新运行。

脚本将向你提供在 Visual Studio Team Services 中配置持续集成的选项。如果已设置了 Team Services 项目，你将会有 URL：https://yourAccountName.visualstudio.com。在系统提示时输入完整的 URL。你可以为 Team Services 项目提供一个新名称或已有的名称。

## 在 Visual Studio Team Services 中设置生成和测试定义

我们在此项目中使用 Team Services 主要是为了使用其生成和测试功能。但它还提供卓越的协作支持，如使用看板的任务管理、与任务和源控件集成的代码审阅，以及封闭生成。它可与 GitHub、Xamarin、HockeyApp（当然还有 Visual Studio）等其他工具良好集成。你可以通过 Web 界面或通过 Visual Studio 访问它，具体取决于哪种方式更方便。

生成和发布定义中的步骤使用 Team Services [应用商店](https://marketplace.visualstudio.com/VSTS)中提供的各种插件服务。除了用于运行命令行或复制文件的基本实用程序外，还有一些通过 Xamarin、Android 和其他供应商调用生成并连接到 HockeyApp 的服务。

![Team Services 中的生成选项](./media/iot-solution-build-system/image5.png)

### 生成定义

我们有针对每个主要目标的生成定义。我们还有用于功能和回归测试的变体。这为我们提供：

-   MyDriving.Services（移动应用的后端 Web 应用）

-   MyDriving.Xamarin.Android

    -   MyDriving.Xamarin.Android-Feature

    -   MyDriving.Xamarin.Android-Regression

-   MyDriving.Xamarin.iOS

    -   MyDriving.Xamarin.iOS-Feature

    -   MyDriving.Xamarin.iOS-Regression

-   MyDriving.Xamarin.UWP

    -   MyDriving.Xamarin.UWP-Feature

    -   MyDriving.Xamarin.UWP-Regression

如果想要查看我们的配置的完整详细信息，请参阅 [MyDriving Reference Guide（MyDriving 参考指南）](http://aka.ms/mydrivingdocs)的第 4.7 节，“生成和发布配置”。 他们遵循相同的常规模式。脚本：

1.  还原 NuGet 包。我们不在存储库中保留已编译的代码，因此每个生成的第一步便是还原所需的 NuGet 包。

2.  激活许可证。生成在云中执行，因此，在需要许可证时（尤其是使用 Xamarin 生成服务时），我们需要在当前生成计算机上激活许可证。然后，我们会立即停用该许可证，以遍可以在另一台计算机上使用它。

3.  使用适当的服务进行生成。我们对移动应用使用 Xamarin 生成，对后端 Web 服务使用 Visual Studio 生成。

4.  生成测试。

5.  运行测试。我们在 Xamarin 测试云中运行移动应用测试。

6.  将生成结果发布到放置位置。

主要生成的触发器设置为持续集成。也就是说，每当代码签入主分支时都会运行生成。

![在其中将触发器设置为持续集成的接口](./media/iot-solution-build-system/image6.png)

### 发布定义

发布定义的设置方式大体相同。

对于 Web 服务，我们将部署设置为 Azure Web 应用：

![用于将部署设置为 Azure Web 应用的接口](./media/iot-solution-build-system/image7.png)

并将发布触发器设置为连续部署。也就是说，成功生成后的每次签入都会导致更新 Web 应用。

![用于将发布触发器设置为连续部署的接口](./media/iot-solution-build-system/image8.png)

对于移动应用，我们将部署到 HockeyApp：

![用于将移动应用部署到 HockeyApp 的接口](./media/iot-solution-build-system/image9.png)

## 通过使用 Application Insights 探索遥测

[Application Insights](/documentation/articles/app-insights-overview/) 收集有关 Web 服务的性能和使用情况的遥测。Application Insights SDK 将遥测从服务发送到 Azure 中的 Application Insights 资源。

浏览到模板配置的 Application Insights 资源。你可以在那里浏览你的 [Mobile App Service 项目](https://github.com/Azure-Samples/MyDriving/tree/master/src/MobileAppService)的性能图表。它们会显示服务器请求和响应时间、故障和异常计数。此外，还有一个依赖项响应时间图表，即，对数据库和对 REST API 的调用。如果有任何性能问题，你将能够看到是系统的哪一部分导致了这些问题。

![示例性能图表](./media/iot-solution-build-system/image11.png)

如果你的 Web 服务是手动设置的，则可轻松获得相同的图表。在“Web 服务”边栏选项卡中，单击“工具”>“扩展”>“添加”。选择“Application Insights”。

![用于选择“Application Insights”以获取图表的接口](./media/iot-solution-build-system/image12.png)

此功能是通过使用 Application Insights SDK 检测你的应用程序来实现的。

可以通过在开发时[添加 Application Insights SDK](/documentation/articles/app-insights-asp-net/) 来添加自定义遥测（或检测在 Azure 外部运行的应用程序）。这对于记录依赖于应用程序的指标（如用户的平均行程长度或总里程）非常有用。在 Visual Studio 中，右键单击该项目，然后选择“添加 Application Insights”。

![用于选择“添加 Application Insights”以添加自定义遥测的接口](./media/iot-solution-build-system/image10.png)

如果 Application Insights 看到异常的失败响应数，它将发送警报电子邮件。你还可以按照不同的指标（如响应时间）设置你自己警报。

如果只是要确保你的 Web 服务始终保持正常运行，可以设置[可用性测试](/documentation/articles/app-insights-monitor-web-app-availability/)。这些测试将每隔 15 分钟从世界各地的不同位置对你的站点进行 ping 操作。同样，如果出现问题你将收到一封电子邮件。

## 估计运营成本

小规模运行这样一种应用的成本极低。许多服务具有免费的入门级层级，因此开发和小规模运营所需的成本非常低。当然，你自己的应用无需使用 MyDriving 中演示的所有功能。

下面对设置 MyDriving 的开发配置成本进行了粗略估计。我们还注意到我们以前不使用一些备选方案。此信息在你估计成本时可能有所帮助。

我们假设：

-   一个不超过五人的团队（加上观察的利益干系人）。

-   运行约一个月。

-   每天有 100 用户的四个行程。

>[AZURE.NOTE] 如果你是 Azure 的新用户，可以使用[试用版](/pricing/1rmb-trial/)。

| **服务/组件** | **说明** | **成本/月** |
|--------|--------|----------------|
| [Visual Studio 2015 Community](https://www.visualstudio.com/products/visual-studio-community-vs) with [Xamarin](https://visualstudiogallery.msdn.microsoft.com/dcd5b7bd-48f0-4245-80b6-002d22ea6eee) <br/>跨平台开发环境| Visual Studio Community。（需要 [Visual Studio Professional](https://www.visualstudio.com/vs-2015-product-editions) for [Xamarin.Forms](https://xamarin.com/forms) 从单个代码库设计跨平台。） | $0 |
| [Azure IoT 中心](/pricing/details/iot-hub)<br/>与设备的双向数据连接 | 8,000 条消息 + 0.5 KB/消息免费。 | $0 |
| [流分析](/pricing/details/stream-analytics)<br/>高容量流数据处理 | 启用时，每小时对每个流式处理单位收费 $0.031。选择所需的流式处理单位数；更多情况下会增加。 | $23 |
| [App Service](/pricing/details/app-service) <br/>托管移动后端 | B1 层--生产 Web 应用。 | $56 |
| Visual Studio Team Services <br/>生成、单元测试和发布管理；任务管理 | 私有 Agent，五个用户。| $0 |
| [Application Insights](/pricing/details/application-insights) <br/>监视 Web 服务和网站的性能及使用情况| 免费层。 | $0 |
| [HockeyApp](http://hockeyapp.net/pricing) <br/> 分发 beta 应用以及反馈、使用情况和故障数据的集合 | 为新用户提供两个免费应用。<br/> 之后每月 $30。 | $0 |
| [Xamarin](https://store.xamarin.com)<br/> 针对多个设备的统一平台上的代码 | 试用版。<br/>之后每月 $25。| $0 |
| 面向 Azure App Service 的 [SQL 数据库](/pricing/details/sql-database/)| 基本层；单一数据库模型。 | $5 |
| [Service Fabric](/documentation/services/service-fabric/)（可选） | 运行本地群集。 | $0 |
| [Power BI](https://powerbi.microsoft.com/pricing)<br/> 流式传输数据和静态数据的通用显示和调查| 免费层：1 GB、10,000 行/小时，每天刷新。<br/>对于[更高的限制](https://powerbi.microsoft.com/documentation/powerbi-power-bi-pro-content-what-is-it)、更多连接选项和协作，$10/用户/月。 | $0 |
| [存储](/pricing/details/storage/) | L（本地冗余）&lt; 100 G，$0.024/GB。 | $3 |
| [Data Factory](/pricing/details/data-factory) | 每个活动 $0.60 * (8 - 5 FOC)。| $2 |
| [HDInsight](/pricing/details/hdinsight)<br/> 用于每日重新导流的按需群集 | 三个 A3 节点，$0.32/小时，每天 1 小时 * 31 天。 | $30 |
| [事件中心](/pricing/details/event-hubs) | 每月吞吐量单位 $11 的基本费用 + $0.028 入口费用。 | $11 |
| OBD 硬件保护装置 || $12 |
| **总计**| | **$157** |

有关详细信息，请参阅：

-   [Azure service quotas and limits（Azure 服务配额和限制）](/documentation/articles/azure-subscription-service-limits/#iot-hub-limits)的摘要

-   Azure [价格计算器](/pricing/calculator)

## 向我们发送你的反馈

因为我们已经创建了 MyDriving 来帮助你快速开始创建自己的 IoT 系统，我们非常希望收到你对它的工作情况的反馈。如果出现以下情况，请告知我们：

-  遇到困难或难题。

-  存在更适合你的方案的扩展点。

-  你发现了完成某些需求的更高效方法。

-  你有改进 MyDriving 或此文档的任何其他建议。

若要提供反馈，请 [在 GitHub 上提出问题] 或在下面留言（en-us 版）。

我们期待收到你的反馈！

## 后续步骤

我们建议使用 [MyDriving Reference Guide（MyDriving 参考指南）](http://aka.ms/mydrivingdocs)，它对系统及其组件的设计进行了全面说明。

<!---HONumber=Mooncake_0815_2016-->