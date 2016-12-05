<properties
    pageTitle="在 Azure App Service 中创建 .NET Web 作业 | Azure"
    description="使用 ASP.NET MVC 和 Azure 创建多层应用。前端在 Azure App Service 中的 Web 应用中运行，后端以 Web 作业的形式运行。应用程序使用实体框架、SQL 数据库和 Azure 存储队列和 Blob。"
    services="app-service"
    documentationcenter=".net"
    author="tdykstra"
    manager="wpickett"
    editor="mollybos" />  

<tags
    ms.assetid="99cb9917-483a-45f8-a98d-07d19c68c753"
    ms.service="app-service"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/28/2016"
    wacn.date="12/05/2016"
    ms.author="tdykstra" />

# 在 Azure App Service 中创建 .NET Web 作业
本教程演示如何为使用 [WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk/) 的简单多层 ASP.NET MVC 5 应用程序编写代码。

[WebJobs SDK](/documentation/articles/websites-webjobs-resources/) 可简化针对 Web 作业执行的常见任务（例如，图像处理、队列处理、RSS 聚合、文件维护和发送电子邮件）编写代码。WebJobs SDK 的内置功能使用 Azure 存储空间和 Service Bus，用于计划任务和处理错误，以及用于许多其他常见方案。此外，还可以扩展其设计，且拥有[用于扩展的开源存储库](https://github.com/Azure/azure-webjobs-sdk-extensions/wiki/Binding-Extensions-Overview)。

应用程序示例为广告公告板。用户可以上载广告图像，然后后端进程将图像转换成缩略图。广告列表页显示缩略图，广告详细信息页显示完整大小的图像。下面是屏幕截图：

![广告列表](./media/websites-dotnet-webjobs-sdk-get-started/list.png)

此应用程序示例可处理 [Azure 队列](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/queue-centric-work-pattern) 和 [Azure blob](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/unstructured-blob-storage)。本教程演示如何将应用程序部署到 [Azure App Service](/documentation/articles/app-service-changes-existing-services/) 和 [Azure SQL 数据库](/documentation/articles/sql-database-technical-overview/)。

## <a id="prerequisites"></a>先决条件
本教程假设你知道如何在 Visual Studio 中处理 [ASP.NET MVC 5](http://www.asp.net/mvc/tutorials/mvc-5/introduction/getting-started) 项目。

本教程针对 Visual Studio 2013 编写。如果尚未安装 Visual Studio，则在安装用于 .NET 的 Azure SDK 时会自动安装 Visual Studio。

本教程可以配合 Visual Studio 2015 使用，但在本地运行应用程序之前，必须将 Web.config 和 App.config 文件中 SQL Server LocalDB 连接字符串的 `Data Source` 部分从 `Data Source=(localdb)\v11.0` 更改为 `Data Source=(LocalDb)\MSSQLLocalDB`。

> [AZURE.NOTE] <a name="note"></a>完成本教程需要 Azure 帐户：
><p>
> * 可以[注册一个 Azure 帐户](/pricing/1rmb-trial/?WT.mc_id=A261C142F)：获取用于试用付费版 Azure 服务的信用额度，甚至在用完信用额度后，仍可保留帐户并使用免费 Azure 服务（如网站）。不会收取任何费用，除非明确更改设置并要求收费。
>
>

## <a id="learn"></a>学习内容
本教程介绍如何执行以下任务：

* 在计算机上安装 Azure SDK 进行 Azure 开发。
* 创建控制台应用程序项目，在部署关联的 Web 项目时，该应用程序项目将自动部署为 Azure Web 作业。
* 在开发计算机上本地测试 WebJobs SDK 后端。
* 将包含 Web 作业后端的应用程序发布到应用服务中的 Web 应用。
* 上载文件并将其存储在 Azure Blob 服务中。
* 使用 Azure WebJobs SDK 处理 Azure 存储队列和 Blob。

## <a id="contosoads"></a>应用程序体系结构
应用程序示例使用[以队列为中心的工作模式](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/queue-centric-work-pattern)，减轻后端进程创建缩略图的 CPU 密集型工作。

该应用程序将广告存储在 SQL 数据库中，通过使用 Entity Framework Code First 创建表和访问数据。对于每个广告，数据库存储两个 URL：一个用于全尺寸图像，另一个用于缩略图。

![广告表](./media/websites-dotnet-webjobs-sdk-get-started/adtable.png)

当用户上载一个图像时，Web 应用将在 [Azure Blob](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/unstructured-blob-storage) 中存储图像，并将广告信息存储在带有指向 Blob 的 URL 的数据库中。同时，它将消息写入 Azure 队列。在作为 Azure Web 作业运行的后端进程中，WebJobs SDK 将轮询新消息的队列。显示新消息时，Web 作业将创建该图像的缩略图，并为该广告更新缩略图 URL 数据库字段。下图介绍应用程序各部分之间如何交互：

![Contoso 广告体系结构](./media/websites-dotnet-webjobs-sdk-get-started/apparchitecture.png)

[AZURE.INCLUDE [install-sdk](../../includes/install-sdk-2015-2013.md)]

本教程中的说明不适用于用于 .NET 2.7.1 的 Azure SDK 或更高版本。

## <a id="storage"></a>创建 Azure 存储帐户
Azure 存储帐户可提供在云中存储队列和 Blob 数据的资源。并且，WebJobs SDK 还可用于存储仪表板的日志记录数据。

在实际应用程序中，通常会为应用程序数据与记录数据创建单独帐户，并为测试数据与生产数据创建单独帐户。对于本教程，将只使用一个帐户。

1. 在 Visual Studio 中，打开“服务器资源管理器”窗口。
2. 右键单击“Azure”节点，然后单击“连接到 Azure”。
   ![连接到 Azure](./media/websites-dotnet-webjobs-sdk-get-started/connaz.png)
3. 使用 Azure 凭据登录。
4. 在 Azure 节点下右键单击“存储”，然后单击“创建存储帐户”。
   ![创建存储帐户](./media/websites-dotnet-webjobs-sdk-get-started/createstor.png)
5. 在“创建存储帐户”对话框中，输入存储帐户的名称。

    该名称必须唯一（其他 Azure 存储帐户不能使用该名称）。如果输入的名称已被使用，可以进行更改。

    用于访问存储帐户的 URL 为 *{名称}*.core.chinacloudapi.cn。
6. 将“区域或地缘组”设置为离你最近的区域。

    此设置指定将托管存储帐户的 Azure 数据中心。对于本教程，所做的选择不会带来明显的差异。但是，对于生产 Web 应用，希望 Web 服务器和存储帐户在同一区域，以最大程度减少延迟和数据传出费用。Web 应用（稍后创建）数据中心应尽可能靠近访问 Web 应用的浏览器，以最大程度减少延迟。
7. 将“复制”下拉列表设置为“本地冗余”。

    为存储帐户启用地域复制时，会将存储内容复制到辅助数据中心，这样就能够在主要位置发生重大灾难时将故障转移到该位置。地域复制可能会产生额外的成本。对于测试和开发帐户，你通常不希望因为地域复制而付款。有关详细信息，请参阅[创建、管理或删除存储帐户](/documentation/articles/storage-create-storage-account/)。
8. 单击“创建”。

    ![新建存储帐户](./media/websites-dotnet-webjobs-sdk-get-started/newstorage.png)

## <a id="download"></a>下载应用程序
1. 下载并解压缩[已完成的解决方案](http://code.msdn.microsoft.com/Simple-Azure-Website-with-b4391eeb)。
2. 启动 Visual Studio。
3. 从“文件”菜单中，选择“打开”>“项目/解决方案”，导航到下载解决方案的位置，然后打开解决方案文件。
4. 按 CTRL+SHIFT+B 生成解决方案。

    默认情况下，Visual Studio 自动还原 NuGet 包内容，但不包括在 *.zip* 文件。如果该包无法还原，请转到“管理解决方案的 NuGet 程序包”对话框，并单击右上角的“还原”按钮进行手动安装。
5. 在“解决方案资源管理器”中，请确保选择“ContosoAdsWeb”作为启动项目。

## <a id="configurestorage" name="configure-storage"></a><a name="configure-the-web-app-to-use-your-azure-sql-database-and-storage-account"></a>将应用程序配置为使用存储帐户

1. 打开 ContosoAdsWeb 项目中的应用程序 *Web.config* 文件。

    该文件包含用于处理 Blob 和队列的 SQL 连接字符串和 Azure 存储连接字符串。

    SQL 连接字符串指向 [SQL Server Express LocalDB](http://msdn.microsoft.com/zh-cn/library/hh510202.aspx) 数据库。

    存储连接字符串是一个示例，包含存储帐户名称和访问密钥的占位符。需要将此字符串替换为包含存储帐户的名称和密钥的连接字符串。

    <pre class="prettyprint">&lt;connectionStrings>
      &lt;add name="ContosoAdsContext" connectionString="Data Source=(localdb)\v11.0; Initial Catalog=ContosoAds; Integrated Security=True; MultipleActiveResultSets=True;" providerName="System.Data.SqlClient" />
      &lt;add name="AzureWebJobsStorage" connectionString="DefaultEndpointsProtocol=https;AccountName=<mark>[accountname]</mark>;AccountKey=<mark>[accesskey]</mark>"/>
    &lt;/connectionStrings></pre>

    存储连接字符串名为 AzureWebJobsStorage，因为这是 WebJobs SDK 默认使用的名称。在此处之所以使用同一名称，是因为在 Azure 环境中只能设置一个连接字符串值。
2. 在“服务器资源管理器”中，右键单击存储帐户下的“存储”节点，然后单击“属性”。

    ![单击存储帐户属性](./media/websites-dotnet-webjobs-sdk-get-started/storppty.png)
3. 在“属性”窗口中，单击“存储帐户密钥”，然后单击省略号。

    ![新建存储帐户](./media/websites-dotnet-webjobs-sdk-get-started/newstorage.png)
4. 复制“连接字符串”。

    ![“存储帐户密钥”对话框](./media/websites-dotnet-webjobs-sdk-get-started/cpak.png)
5. 将 *Web.config* 文件中的存储连接字符串替换为刚复制的连接字符串。在粘贴之前，请确保选择引号括住的所有内容，但不包括引号本身。
6. 打开 ContosoAdsWebJob 项目中的 *App.config* 文件。

    此文件包含两个存储连接字符串：一个用于应用程序数据，另一个用于日志记录。可以为应用程序数据和日志记录使用单独的存储帐户，也可以[为数据使用多个存储帐户](https://github.com/Azure/azure-webjobs-sdk/blob/master/test/Microsoft.Azure.WebJobs.Host.EndToEndTests/MultipleStorageAccountsEndToEndTests.cs)。对于本教程，将使用一个存储帐户。连接字符串包含存储帐户密钥的占位符。

      <pre class="prettyprint">&lt;configuration>
      &lt;connectionStrings>
          &lt;add name="AzureWebJobsDashboard" connectionString="DefaultEndpointsProtocol=https;AccountName=<mark>[accountname]</mark>;AccountKey=<mark>[accesskey]</mark>"/>
          &lt;add name="AzureWebJobsStorage" connectionString="DefaultEndpointsProtocol=https;AccountName=<mark>[accountname]</mark>;AccountKey=<mark>[accesskey]</mark>"/>
          &lt;add name="ContosoAdsContext" connectionString="Data Source=(localdb)\v11.0; Initial Catalog=ContosoAds; Integrated Security=True; MultipleActiveResultSets=True;"/>
      &lt;/connectionStrings>
          &lt;startup>
              &lt;supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5" />
      &lt;/startup>
         &lt;/configuration></pre>

    默认情况下，WebJobs SDK 将查找名为 AzureWebJobsStorage 和 AzureWebJobsDashboard 的连接字符串。作为替代方法，可以根据需要[存储该连接字符串，并将其显式传递给 `JobHost` 对象](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/#config)。
7. 将两个存储连接字符串替换为先前复制的连接字符串。
8. 保存所做更改。

## <a id="run"></a>在本地运行应用程序
1. 若要启动应用程序的 Web 前端，请按 CTRL+F5。

    默认浏览器将打开主页。（Web 项目运行，因为已将其设为启动项目。）

    ![Contoso 广告主页](./media/websites-dotnet-webjobs-sdk-get-started/home.png)
2. 若要启动应用程序的 Web 作业后端，请在“解决方案资源管理器”中右键单击 ContosoAdsWebJob 项目，然后单击“调试”>“启动新实例”。

    此时将打开一个控制台应用程序窗口，显示指示 WebJobs SDK JobHost 对象已开始运行的日志记录消息。

    ![显示后端正在运行的控制台应用程序窗口](./media/websites-dotnet-webjobs-sdk-get-started/backendrunning.png)
3. 在浏览器中，单击“创建一个广告”。
4. 输入一些测试数据并选择一个要上载的图像，然后单击“创建”。

    ![创建页面](./media/websites-dotnet-webjobs-sdk-get-started/create.png)

    该应用程序转到索引页，但不显示新广告的缩略图，因为该处理尚未发生。

    在经过片刻等待后，控制台应用程序窗口中的日志记录消息将显示已收到并处理某个队列消息。

    ![显示队列消息已处理的控制台应用程序窗口](./media/websites-dotnet-webjobs-sdk-get-started/backendlogs.png)
5. 在查看控制台应用程序窗口中的日志记录消息后，请刷新“索引”页以查看缩略图。

    ![索引页面](./media/websites-dotnet-webjobs-sdk-get-started/list.png)
6. 单击广告的“详细信息”查看实际尺寸的图像。

    ![详细信息页](./media/websites-dotnet-webjobs-sdk-get-started/details.png)

你已在本地计算机上运行应用程序，该应用程序正在使用计算机上的 SQL Server 数据库，但它在处理云中的队列和 Blob。在下一节中，将使用云数据库以及云 Blob 和队列在云中运行该应用程序。

## <a id="runincloud"></a>在云中运行应用程序
执行以下步骤在云中运行应用程序：

* 部署到 Web 应用。Visual Studio 在应用服务中自动创建新的 Web 应用和 SQL 数据库实例。
* 将 Web 应用配置为使用 Azure SQL 数据库和存储帐户。

在云中运行时创建一些广告后，请查看 WebJobs SDK 仪表板，以了解该仪表板提供的丰富功能。

### 部署到 Web 应用
1. 关闭浏览器和控制台应用程序窗口。
2. 在“解决方案资源管理器”中，右键单击 ContosoAdsWeb 项目，然后单击“发布”。
3. 在“发布 Web”向导的“配置文件”步骤中，单击“Azure Web 应用”。

    ![选择 Azure Web 应用发布目标](./media/websites-dotnet-webjobs-sdk-get-started/pubweb.png)  

4. 登录 Azure（如果尚未登录）。
5. 单击“新建”。

    根据你安装的 Azure SDK for.NET 版本，对话框可能与屏幕截图略有不同。

    ![单击“新建”](./media/websites-dotnet-webjobs-sdk-get-started/clicknew.png)  

6. 在“在 Azure 上创建 Web 应用”对话框框中，在“Web 应用名称”框中输入唯一名称。

    完整的 URL 将包含你在此处输入的内容和 .chinacloudsites.cn（如“Web 应用名称”文本框的旁边所示）。例如，如果 Web 应用名称为 ContosoAds，则 URL 将为 ContosoAds.chinacloudsites.cn。
7. 在“[App Service 计划](/documentation/articles/azure-web-sites-web-hosting-plans-in-depth-overview/)”下拉列表中，选择“创建新的 App Service 计划”。输入 App Service 计划的名称，例如 ContosoAdsPlan。
8. 在“[资源组](/documentation/articles/resource-group-overview/)”下拉列表中，选择“创建新的资源组”。
9. 输入资源组的名称，例如 ContosoAdsGroup。
10. 在“区域”下拉列表中，选择你为存储帐户所选的同一区域。

    此设置指定你的 Web 应用将在哪个 Azure 数据中心运行。在同一数据中心保留 Web 应用和存储帐户可以最大程度地减少延迟和数据传出费用。
11. 在“数据库服务器”下拉列表中选择“创建新的服务器”。
12. 输入数据库服务器的名称，例如 contosoadsserver + 数字或你的姓名，使服务器名称保持唯一。

    服务器名称必须唯一。该名称可以包含小写字母、数字和短划线，但尾部不能包含短划线。

    或者，如果你的订阅已有一台服务器，可从下拉列表中选择该服务器。
13. 输入管理员的“数据库用户名”和“数据库密码”。

    如果你选择了“新建 SQL 数据库服务器”，则在此处不要输入现有名称和密码。你应输入新的名称和密码，你现在定义的名称和密码将在你以后访问数据库时使用。如果你选择之前创建的服务器，系统将提示你已创建的管理用户帐户的密码。
14. 单击“创建”。

    ![在 Azure 对话框中创建 Web 应用](./media/websites-dotnet-webjobs-sdk-get-started/newdb.png)  


    Visual Studio 将创建解决方案、Web 项目、Azure 中的 Web 应用和 Azure SQL 数据库实例。
15. 在“发布 Web”向导的“连接”步骤中，单击“下一步”。

    ![连接步骤](./media/websites-dotnet-webjobs-sdk-get-started/connstep.png)
16. 在“设置”步骤中，清除“在运行时使用此连接字符串”复选框，然后单击“下一步”。

    ![设置步骤](./media/websites-dotnet-webjobs-sdk-get-started/settingsstep.png)

    不需要使用发布对话框设置 SQL 连接字符串，因为稍后会在 Azure 环境中设置该值。

    可以忽略此页面上的警告。

    * 通常，在 Azure 中运行时使用的存储帐户不同于在本地运行时使用的存储帐户，但对于本教程，将在两个环境中使用相同的存储帐户。因此，不需要转换 AzureWebJobsStorage 连接字符串。即使要在云中使用不同的存储帐户，也无需转换连接字符串，因为应用程序在 Azure 中运行时将使用 Azure 环境设置。稍后，将会在教程中遇到这种情况。
    * 对于本教程，不需要更改用于 ContosoAdsContext 数据库的数据模型，因此无需使用 Entity Framework Code First 迁移进行部署。应用程序首次尝试访问 SQL 数据时，Code First 自动创建新的数据库。

    对于本教程，使用“文件发布选项”下的选项默认值。
17. 在“预览”步骤中，单击“开始预览”。

    ![单击“开始预览”](./media/websites-dotnet-webjobs-sdk-get-started/previewstep.png)

    可以忽略有关未发布数据库的警告。Entity Framework Code First 创建数据库；不需要发布该数据库。

    预览窗口显示 Web 作业项目中复制到 Web 应用的 *app\_data\\jobs\\continuous* 文件夹的二进制文件和配置文件。

    ![预览窗口中的 Web 作业文件](./media/websites-dotnet-webjobs-sdk-get-started/previewwjfiles.png)
18. 单击“发布”。

    Visual Studio 部署该应用程序，并在浏览器中打开主页 URL。

    只有在学习下一节在 Azure 环境中设置连接字符串后，才可使用该 Web 应用。将会看到错误页或主页，具体取决于在前面选择的 Web 应用和数据库创建选项。

### 将 Web 应用配置为使用 Azure SQL 数据库和存储帐户。
最佳安全做法是[避免将敏感信息（如连接字符串）放置在源代码存储库中存储的文件内](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/source-control#secrets)。Azure 提供一种方法实现此目的：可在 Azure 环境中设置连接字符串和其他设置值，在 Azure 中运行应用程序时，ASP.NET 配置 API 自动提取这些值。也可以使用**服务器资源管理器**、Azure 门户、Windows PowerShell 或跨平台命令行接口在 Azure 中设置这些值。有关详细信息，请参阅[应用程序字符串和连接字符串的工作原理](https://azure.microsoft.com/blog/2013/07/17/windows-azure-web-sites-how-application-strings-and-connection-strings-work/)。

在本节中，将使用**服务器资源管理器**在 Azure 中设置连接字符串值。

1. 在“服务器资源管理器”中，右键单击“Azure”>“应用服务”>“{你的资源组}”下的 Web 应用，然后单击“查看设置”。

    “Azure Web 应用”窗口将在“配置”选项卡上打开。
2. 将 DefaultConnection 连接字符串的名称更改为 ContosoAdsContext。

    使用关联的数据库创建 Web 应用时，Azure 已自动创建此连接字符串，因此它已具有正确的连接字符串值。只需将名称更改为代码要查找的值。
3. 添加名为 AzureWebJobsStorage 和 AzureWebJobsDashboard 的两个新连接字符串。将类型设置为“自定义”，并将连接字符串值设置为前面针对 *Web.config* 和 *App.config* 文件使用的相同值。（确保包括整个连接字符串而不仅仅是访问密钥，且不包括引号。）

    WebJobs SDK 使用这两个连接字符串：一个用于应用程序数据，另一个用于日志记录。如前面所看到的，Web 前端代码也使用用于应用程序数据的连接字符串。
4. 单击“保存”。

    ![Azure 门户中的连接字符串](./media/websites-dotnet-webjobs-sdk-get-started/azconnstr.png)
5. 在“服务器资源管理器”中右键单击该 Web 应用，然后单击“停止”。
6. Web 应用停止后，再次右键单击该 Web 应用，然后单击“启动”。

   发布时 Web 作业会自动启动，但在更改配置时会停止。若要重新启动它，可以重新启动 Web 应用或者在 [Azure 门户](/documentation/articles/app-service-web-app-azure-portal/)中重新启动 Web 作业。通常建议在更改配置后重新启动 Web 应用。
7. 刷新地址栏中包含 Web 应用 URL 的浏览器窗口。

    此时将显示主页。
8. 就像在本地运行应用程序时一样创建一个广告。

   “索引”页首先不会显示缩略图。
9. 几秒钟后刷新页面，随后会显示缩略图。

   如果未显示缩略图，可能需要等待大约一分钟，让 Web 作业重新启动。如果在刷新页面后仍未显示缩略图，可能是 Web 作业没有自动启动。在此情况下，转到[经典管理门户](https://manage.windowsazure.cn)页中 Web 应用的“Web 作业”选项卡，然后单击“启动”。

### <a name="view-the-webjobs-sdk-dashboard"></a>查看 WebJobs SDK 仪表板
1. 在[经典管理门户](https://manage.windowsazure.cn)中选择 Web 应用。
2. 单击“Web 作业”选项卡。
3. 单击 Web 作业“日志”列中的 URL。

    ![“Web 作业”选项卡](./media/websites-dotnet-webjobs-sdk-get-started/wjtab.png)

    WebJobs SDK 仪表板中将打开一个新的浏览器选项卡。仪表板显示正在运行的 Web 作业，以及 WebJobs SDK 触发的代码中的函数列表。
4. 单击某个函数可以查看有关其执行的详细信息。

    ![WebJobs SDK 仪表板](./media/websites-dotnet-webjobs-sdk-get-started/wjdashboardhome.png)

    ![WebJobs SDK 仪表板](./media/websites-dotnet-webjobs-sdk-get-started/wjfunctiondetails.png)

    单击此页上的“重放函数”会导致 WebJobs SDK 框架再次调用该函数，并且可以首先更改传递给该函数的数据。

> [AZURE.NOTE]
完成测试后，请删除 Web 应用和 SQL 数据库实例。Web 应用免费，但 SQL 数据库实例和存储帐户收费（由于较小，因此费用很低）。此外，如果保持 Web 应用运行，则找到 URL 的任何人都可以创建和查看广告。在经典管理门户中，转到 Web 应用的“仪表板”选项卡，然后单击页面底部的“删除”按钮。然后，可以选中用于同时删除 SQL 数据库实例的复选框。如果只想暂时防止其他人访问 Web 应用，请单击“停止”。在这种情况下，SQL 数据库和存储帐户会继续收费。如果不再需要 SQL 数据库和存储帐户，可以遵循类似过程将其删除。
>
>

## <a id="create"></a>从头开始创建应用程序
在本节中，将执行以下任务：

* 创建包含 Web 项目的 Visual Studio 解决方案。
* 为前端和后端之间共享的数据访问层添加类库项目。
* 启用 Web 作业部署，为后端添加控制台应用程序项目。
* 添加 NuGet 包。
* 设置项目引用。
* 从在学习本教程前面部分时使用的下载应用程序中复制应用程序代码和配置文件。
* 查看用于处理 Azure Blob 和队列及 WebJobs SDK 的代码部分。

### 创建包含 Web 项目的 Visual Studio 解决方案和类库项目
1. 在 Visual Studio 的“文件”菜单中选择“新建”>“项目”。
2. 在“新建项目”对话框中，选择“Visual C#”>“Web”>“ASP.NET Web 应用程序”。
3. 将项目命名为 ContosoAdsWeb，将解决方案命名为 ContosoAdsWebJobsSDK（如果要将解决方案放置在与下载的解决方案相同的文件夹中，请更改此解决方案名称），然后单击“确定”。

    ![新建项目](./media/websites-dotnet-webjobs-sdk-get-started/newproject.png)
4. 在“新建 ASP.NET 项目”对话框中选择 MVC 模板，然后清除“Azure”下的“在云中托管”复选框。

    选中“在云中托管”可让 Visual Studio 自动创建新的 Azure Web 应用和 SQL 数据库。由于前面已创建这些网站和数据库，因此现在创建项目时不需要执行此操作。如果要新建，请选中该复选框。然后，可以按照之前部署应用程序时的方式配置新的 Web 应用和 SQL 数据库。
5. 单击“更改身份验证”。

    ![更改身份验证](./media/websites-dotnet-webjobs-sdk-get-started/chgauth.png)
6. 在“更改身份验证”对话框中，选择“无身份验证”，然后单击“确定”。

    ![无身份验证](./media/websites-dotnet-webjobs-sdk-get-started/noauth.png)
7. 在“新建 ASP.NET 项目”对话框中，单击“确定”。

    Visual Studio 创建解决方案和 Web 项目。
8. 在“解决方案资源管理器”中，右键单击该解决方案（而非项目），然后选择“添加”>“新建项目”。
9. 在“添加新项目”对话框中，选择“Visual C#”>“Windows 桌面”>“类库”模板。
10. 将项目命名为 *ContosoAdsCommon*，然后单击“确定”。

    此项目将包含前端和后端将要使用的实体框架上下文与数据模型。或者，也可以在 Web 项目中定义 EF 相关的类，并从 Web 作业项目引用该项目。但是，这样会使 Web 作业项目引用它不需要的 Web 程序集。

### 在启用 Web 作业部署的情况下添加控制台应用程序项目
1. 右键单击 Web 项目（而非解决方案或类库项目），然后单击“添加”>“新建 Azure Web 作业项目”。

    ![“新建 Azure Web 作业项目”菜单选项](./media/websites-dotnet-webjobs-sdk-get-started/newawjp.png)
2. 在“添加 Azure Web 作业”对话框中，输入 ContosoAdsWebJob 作为“项目名称”和“Web 作业名称”。将“Web 作业运行模式”保留设置为“连续运行”。
3. 单击“确定”。

   Visual Studio 创建控制台应用程序，每当部署 Web 项目时，该应用程序就会部署为 Web 作业。为此，它将在创建项目后执行以下任务：

   * 在 Web 作业项目的 Properties 文件夹中添加 *webjob-publish-settings.json* 文件。
   * 在 Web 项目的 Properties 文件夹中添加 *webjobs-list.json* 文件。
   * 在 Web 作业项目中安装 Microsoft.Web.WebJobs.Publish NuGet 包。

   有关这些更改的详细信息，请参阅[如何使用 Visual Studio 部署 Web 作业](/documentation/articles/websites-dotnet-deploy-webjobs/)。

### 添加 NuGet 包
Web 作业的 new-project 模板自动安装 WebJobs SDK NuGet 包 [Microsoft.Azure.WebJobs](http://www.nuget.org/packages/Microsoft.Azure.WebJobs) 及其依赖项。

在 Web 作业项目中自动安装的其中一个 WebJobs SDK 依赖项是 Azure 存储客户端库 (SCL)。但是，若要处理 Blob 和队列，需要将此依赖项添加到 Web 项目。

1. 打开解决方案的“管理 NuGet 包”对话框。
2. 在左窗格中，选择“已安装的包”。
3. 找到 *Azure 存储空间*包，然后单击“管理”。
4. 在“选择项目”框中，选中“ContosoAdsWeb”复选框，然后单击“确定”。

    所有三个项目都使用实体框架处理 SQL 数据库中的数据。
5. 在左窗格中，选择“联机”。
6. 找到 *EntityFramework* NuGet 包，并将其安装到所有三个项目。

### 设置项目引用
Web 项目和 Web 作业项目都处理 SQL 数据库，因此两者都需要引用 ContosoAdsCommon 项目。

1. 在 ContosoAdsWeb 项目中，设置对 ContosoAdsCommon 项目的引用。（右键单击 ContosoAdsWeb 项目，然后单击“添加”>“引用”。在“引用管理器”对话框中，选择“解决方案”>“项目”>“ContosoAdsCommon”，然后单击“确定”。）
2. 在 ContosoAdsWebJob 项目中，设置对 ContosAdsCommon 项目的引用。

    WebJob 项目需要通过引用来处理图像和访问连接字符串。
3. 在 ContosoAdsWebJob 项目中，设置对 `System.Drawing` 和 `System.Configuration` 的引用。

### 添加代码和配置文件
本教程未说明如何[使用基架创建 MVC 控制器和视图](http://www.asp.net/mvc/tutorials/mvc-5/introduction/getting-started)、如何[编写适用于 SQL Server 数据库的实体框架代码](http://www.asp.net/mvc/tutorials/getting-started-with-ef-using-mvc)，或者[ASP.NET 4.5 的异步编程基础知识](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/web-development-best-practices#async)。因此，其余所有操作是将已下载解决方案中的代码和配置文件复制到新解决方案。完成该操作后，以下部分将演示并说明代码的关键部分。

若要将文件添加到某个项目或文件夹，请右键单击该项目或文件夹，然后单击“添加”>“现有项”。选择所需的文件，然后单击“添加”。如果询问是否想要替换现有文件，请单击“是”。

1. 在 ContosoAdsCommon 项目中，删除 *Class1.cs* 文件，并在其原位置添加已下载项目中的以下文件。

   * *Ad.cs*
   * *ContosoAdscontext.cs*
   * *BlobInformation.cs*<br/><br/>
2. 在 ContosoAdsWeb 项目中，从下载的项目添加以下文件。

   * *Web.config*
   * *Global.asax.cs*
   * 在 *Controllers* 文件夹中：*AdController.cs*
   * 在 *Views/Shared* 文件夹中：*\_Layout.cshtml* 文件
   * 在 *Views/Home* 文件夹中：*Index.cshtml*
   * 在 *Views/Ad* 文件夹（首先创建该文件夹）中：五个 *.cshtml* 文件<br/><br/>
3. 在 ContosoAdsWebJob 项目中，添加已下载项目中的以下文件。

   * *App.config*（将文件类型筛选器更改为“所有文件”）
   * *Program.cs*
   * *Functions.cs*

现在，可以根据本教程前面所述生成、运行和部署应用程序。但是，在执行此操作前，在部署到的第一个 Web 应用中停止正在运行的 Web 作业。否则，Web 作业将处理本地创建的，或新 Web 应用运行的应用程序创建的队列消息，因为所有消息使用相同的存储帐户。

## <a id="code"></a>查看应用程序代码
以下各节解释与处理 WebJobs SDK 和 Azure 存储 Blob 与队列相关的代码。

> [AZURE.NOTE]
对于 WebJobs SDK 的特定代码，请转到 [Program.cs 和 Functions.cs](#programcs) 部分。
>
>

### ContosoAdsCommon - Ad.cs
Ad.cs 文件为 ad 类别定义一个枚举，为 ad 信息定义一个 POCO 实体类。

        public enum Category
        {
            Cars,
            [Display(Name="Real Estate")]
            RealEstate,
            [Display(Name = "Free Stuff")]
            FreeStuff
        }

        public class Ad
        {
            public int AdId { get; set; }

            [StringLength(100)]
            public string Title { get; set; }

            public int Price { get; set; }

            [StringLength(1000)]
            [DataType(DataType.MultilineText)]
            public string Description { get; set; }

            [StringLength(1000)]
            [DisplayName("Full-size Image")]
            public string ImageURL { get; set; }

            [StringLength(1000)]
            [DisplayName("Thumbnail")]
            public string ThumbnailURL { get; set; }

            [DataType(DataType.Date)]
            [DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
            public DateTime PostedDate { get; set; }

            public Category? Category { get; set; }
            [StringLength(12)]
            public string Phone { get; set; }
        }

### ContosoAdsCommon - ContosoAdsContext.cs
ContosoAdsContext 类指定 DbSet 集合中使用的 Ad 类，实体框架将存储在 SQL 数据库中。

        public class ContosoAdsContext : DbContext
        {
            public ContosoAdsContext() : base("name=ContosoAdsContext")
            {
            }
            public ContosoAdsContext(string connString)
                : base(connString)
            {
            }
            public System.Data.Entity.DbSet<Ad> Ads { get; set; }
        }

类具有两个构造函数。第一个由 Web 项目使用，并指定存储在 Web.config 文件或 Azure 运行时环境中的连接字符串的名称。第二个构造函数允许在实际连接字符串中传递。程序需要 Web 作业项目，因为它没有 Web.config 文件。之前看到存储此连接字符串的位置，稍后会看到在实例化 DbContext 类时代码如何检索连接字符串。

### ContosoAdsCommon - BlobInformation.cs
`BlobInformation` 类用于在队列消息中存储有关图像 Blob 的信息。

        public class BlobInformation
        {
            public Uri BlobUri { get; set; }

            public string BlobName
            {
                get
                {
                    return BlobUri.Segments[BlobUri.Segments.Length - 1];
                }
            }
            public string BlobNameWithoutExtension
            {
                get
                {
                    return Path.GetFileNameWithoutExtension(BlobName);
                }
            }
            public int AdId { get; set; }
        }


### ContosoAdsWeb - Global.asax.cs
从 `Application_Start` 方法调用的代码创建*图像* Blob 容器和*图像*队列（如果它们尚不存在）。这确保只要开始使用新的存储帐户，将会自动创建所需的 Blob 容器和队列。

此代码通过使用 *Web.config* 文件或 Azure 运行时环境中的存储连接字符串，获取存储帐户的访问权限。

        var storageAccount = CloudStorageAccount.Parse
            (ConfigurationManager.ConnectionStrings["AzureWebJobsStorage"].ToString());

然后，获取对*图像* Blob 容器的引用，创建尚不存在的容器，并在新容器上设置访问权限。默认情况下，新容器仅允许拥有存储帐户凭据的客户端访问 Blob。Web 应用需要将 Blob 公开，以便它可以使用指向图像 Blob 的 URL 显示图像。

        var blobClient = storageAccount.CreateCloudBlobClient();
        var imagesBlobContainer = blobClient.GetContainerReference("images");
        if (imagesBlobContainer.CreateIfNotExists())
        {
            imagesBlobContainer.SetPermissions(
                new BlobContainerPermissions
                {
                    PublicAccess = BlobContainerPublicAccessType.Blob
                });
        }

类似代码获取对 *thumbnailrequest* 队列的引用并创建一个新队列。在这种情况下，不需要更改权限。

        CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();
        var imagesQueue = queueClient.GetQueueReference("thumbnailrequest");
        imagesQueue.CreateIfNotExists();

### ContosoAdsWeb - \_Layout.cshtml
*\_Layout.cshtml* 文件在页眉和页脚中设置应用名称，并创建“广告”菜单项。

### ContosoAdsWeb - Views\\Home\\Index.cshtml
*Views\\Home\\Index.cshtml* 文件在主页上显示类别链接。链接将查询字符串变量中 `Category` 枚举的整数值传递到“广告索引”页面。

        <li>@Html.ActionLink("Cars", "Index", "Ad", new { category = (int)Category.Cars }, null)</li>
        <li>@Html.ActionLink("Real estate", "Index", "Ad", new { category = (int)Category.RealEstate }, null)</li>
        <li>@Html.ActionLink("Free stuff", "Index", "Ad", new { category = (int)Category.FreeStuff }, null)</li>
        <li>@Html.ActionLink("All", "Index", "Ad", null, null)</li>

### <a name="ResolveBlobName" id="resolveblobname"></a>ContosoAdsWeb - AdController.cs
在 *AdController.cs* 文件中，构造函数调用 `InitializeStorage` 方法创建 Azure 存储客户端库对象，提供用于处理 Blob 和队列的 API。

然后，代码获取对*图像* Blob 容器的引用，正如之前在 *Global.asax.cs* 中所看到的。执行此操作时，它设置适用于 Web 应用的默认[重试策略](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/transient-fault-handling)。对于超过暂时性故障反复重试超过一分钟的 Web 应用，默认指数回退重试策略可能会将其挂起。此处指定的重试策略将在每次尝试后等待 3 秒，最多可尝试 3 次。

        var blobClient = storageAccount.CreateCloudBlobClient();
        blobClient.DefaultRequestOptions.RetryPolicy = new LinearRetry(TimeSpan.FromSeconds(3), 3);
        imagesBlobContainer = blobClient.GetContainerReference("images");

类似代码获取对*图像*队列的引用。

        CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();
        queueClient.DefaultRequestOptions.RetryPolicy = new LinearRetry(TimeSpan.FromSeconds(3), 3);
        imagesQueue = queueClient.GetQueueReference("blobnamerequest");

大部分控制器代码通常用于使用 DbContext 类的实体框架数据模型。例外情况是 HttpPost `Create` 方法，它上载文件并将其保存在 Blob 存储中。模型联编程序为该方法提供一个 [HttpPostedFileBase](http://msdn.microsoft.com/zh-cn/library/system.web.httppostedfilebase.aspx) 对象。

        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<ActionResult> Create(
            [Bind(Include = "Title,Price,Description,Category,Phone")] Ad ad,
            HttpPostedFileBase imageFile)

如果用户选择要上载的文件，则代码上载该文件，将其保存在 Blob 中，并使用指向 Blob 的 URL 更新广告数据库记录。

        if (imageFile != null && imageFile.ContentLength != 0)
        {
            blob = await UploadAndSaveBlobAsync(imageFile);
            ad.ImageURL = blob.Uri.ToString();
        }

执行上载的代码位于 `UploadAndSaveBlobAsync` 方法中。它将创建 Blob 的 GUID 名称，上载和保存该文件，并将引用返回已保存的 Blob。

        private async Task<CloudBlockBlob> UploadAndSaveBlobAsync(HttpPostedFileBase imageFile)
        {
            string blobName = Guid.NewGuid().ToString() + Path.GetExtension(imageFile.FileName);
            CloudBlockBlob imageBlob = imagesBlobContainer.GetBlockBlobReference(blobName);
            using (var fileStream = imageFile.InputStream)
            {
                await imageBlob.UploadFromStreamAsync(fileStream);
            }
            return imageBlob;
        }

HttpPost `Create` 方法上载 Blob 并更新数据库后，将创建队列消息，以通知后端进程图像已准备好转换为缩略图。

        BlobInformation blobInfo = new BlobInformation() { AdId = ad.AdId, BlobUri = new Uri(ad.ImageURL) };
        var queueMessage = new CloudQueueMessage(JsonConvert.SerializeObject(blobInfo));
        await thumbnailRequestQueue.AddMessageAsync(queueMessage);

HttpPost `Edit` 方法的代码类似，不同之处在于如果用户选择新图像文件，则必须删除此广告已存在的任何 Blob。

        if (imageFile != null && imageFile.ContentLength != 0)
        {
            await DeleteAdBlobsAsync(ad);
            imageBlob = await UploadAndSaveBlobAsync(imageFile);
            ad.ImageURL = imageBlob.Uri.ToString();
        }

以下是删除广告时删除 Blob 的代码：

        private async Task DeleteAdBlobsAsync(Ad ad)
        {
            if (!string.IsNullOrWhiteSpace(ad.ImageURL))
            {
                Uri blobUri = new Uri(ad.ImageURL);
                await DeleteAdBlobAsync(blobUri);
            }
            if (!string.IsNullOrWhiteSpace(ad.ThumbnailURL))
            {
                Uri blobUri = new Uri(ad.ThumbnailURL);
                await DeleteAdBlobAsync(blobUri);
            }
        }
        private static async Task DeleteAdBlobAsync(Uri blobUri)
        {
            string blobName = blobUri.Segments[blobUri.Segments.Length - 1];
            CloudBlockBlob blobToDelete = imagesBlobContainer.GetBlockBlobReference(blobName);
            await blobToDelete.DeleteAsync();
        }

### ContosoAdsWeb - Views\\Ad\\Index.cshtml 和 Details.cshtml
*Index.cshtml* 文件显示包含其他广告数据的缩略图：

        <img  src="@Html.Raw(item.ThumbnailURL)" />

*Details.cshtml* 文件显示全尺寸图像：

        <img src="@Html.Raw(Model.ImageURL)" />

### ContosoAdsWeb - Views\\Ad\\Create.cshtml 和 Edit.cshtml
*Create.cshtml* 和 *Edit.cshtml* 文件指定窗体编码，允许控制器获取 `HttpPostedFileBase` 对象。

        @using (Html.BeginForm("Create", "Ad", FormMethod.Post, new { enctype = "multipart/form-data" }))

`<input>` 元素通知浏览器提供文件选择对话框。

        <input type="file" name="imageFile" accept="image/*" class="form-control fileupload" />

### <a id="programcs"></a>ContosoAdsWebJob - Program.cs
当 Web 作业启动时，`Main` 方法将调用 WebJobs SDK `JobHost.RunAndBlock` 方法，以开始执行当前线程上触发的函数。

        static void Main(string[] args)
        {
            JobHost host = new JobHost();
            host.RunAndBlock();
        }

### <a id="generatethumbnail"></a>ContosoAdsWebJob - Functions.cs - GenerateThumbnail 方法
接收队列消息时，WebJobs SDK 将调用此方法。该方法创建缩略图，并将缩略图放在数据库中的 URL。

        public static void GenerateThumbnail(
        [QueueTrigger("thumbnailrequest")] BlobInformation blobInfo,
        [Blob("images/{BlobName}", FileAccess.Read)] Stream input,
        [Blob("images/{BlobNameWithoutExtension}_thumbnail.jpg")] CloudBlockBlob outputBlob)
        {
            using (Stream output = outputBlob.OpenWrite())
            {
                ConvertImageToThumbnailJPG(input, output);
                outputBlob.Properties.ContentType = "image/jpeg";
            }

            // Entity Framework context class is not thread-safe, so it must
            // be instantiated and disposed within the function.
            using (ContosoAdsContext db = new ContosoAdsContext())
            {
                var id = blobInfo.AdId;
                Ad ad = db.Ads.Find(id);
                if (ad == null)
                {
                    throw new Exception(String.Format("AdId {0} not found, can't create thumbnail", id.ToString()));
                }
                ad.ThumbnailURL = outputBlob.Uri.ToString();
                db.SaveChanges();
            }
        }

* `QueueTrigger` 属性指示 WebJobs SDK thumbnailrequest 队列上接收到新消息时调用此方法。

        [QueueTrigger("thumbnailrequest")] BlobInformation blobInfo,

    队列消息中的 `BlobInformation` 对象是自动反序列化为 `blobInfo` 参数。当该方法完成时，将删除队列消息。如果该方法将在完成之前失败，则不会删除队列消息；10 分钟租约过期后，会再次挑选发布和处理消息。如果有消息始终引起异常，则不会无限期地重复此序列。如果尝试处理某条消息 5 次都不成功，会将该消息移到名为 {queuename}-poison 的队列。可以配置最大尝试次数。
* 这两个 `Blob` 属性提供绑定到 Blob 的对象：一个绑定到现有的图像 Blob，另一个绑定到该方法创建的新缩略图 Blob。

        [Blob("images/{BlobName}", FileAccess.Read)] Stream input,
        [Blob("images/{BlobNameWithoutExtension}_thumbnail.jpg")] CloudBlockBlob outputBlob)

    Blob 名称来自队列消息中收到的 `BlobInformation` 对象的属性（`BlobName` 和 `BlobNameWithoutExtension`）。若要获取存储客户端库的完整功能，可以使用兼容 Blob 的 `CloudBlockBlob` 类。如果要重用为使用 `Stream` 对象编写的代码，可以使用 `Stream` 类。

有关如何编写使用 WebJobs SDK 属性的函数的详细信息，请参阅以下资源：

* [如何通过 WebJobs SDK 使用 Azure 队列存储](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/)
* [如何通过 WebJobs SDK 使用 Azure Blob 存储](/documentation/articles/websites-dotnet-webjobs-sdk-storage-blobs-how-to/)
* [如何通过 WebJobs SDK 使用 Azure 表存储](/documentation/articles/websites-dotnet-webjobs-sdk-storage-tables-how-to/)
* [如何通过 WebJobs SDK 使用 Azure Service Bus](/documentation/articles/websites-dotnet-webjobs-sdk-service-bus/)

> [AZURE.NOTE]
> * 如果在多台 VM 上运行 Web 应用，将会同时运行多个 Web 作业；在某些情况下，这可能会导致多次处理相同的数据。如果使用内置队列、Blob 和服务总线触发器，将不会造成问题。SDK 可确保针对每个消息或 Blob 仅处理一次函数。<p>*有关如何实现正常关闭的信息，请参阅[正常关闭](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/#graceful)。<p>*为方便起见，`ConvertImageToThumbnailJPG` 方法中的代码（未显示）使用 `System.Drawing` 命名空间中的类。但是，此命名空间中的类设计用于 Windows 窗体。不支持在 Windows 或 ASP.NET 服务中使用。有关图像处理选项的详细信息，请参阅[动态图像生成](http://www.hanselman.com/blog/BackToBasicsDynamicImageGenerationASPNETControllersRoutingIHttpHandlersAndRunAllManagedModulesForAllRequests.aspx)和[深入学习图像大小调整](http://www.hanselminutes.com/313/deep-inside-image-resizing-and-scaling-with-aspnet-and-iis-with-imageresizingnet-author-na)。
>
>

## 后续步骤
在本教程中，已了解使用 WebJobs SDK 进行后端处理的简单多层应用程序。本部分提供有关进一步了解 ASP.NET 多层应用程序和 Web 作业的一些建议。

### 缺少的功能
该应用程序有意保持入门教程的简单性。在现实的应用程序中，将实施[依赖关系注入](http://www.asp.net/mvc/tutorials/hands-on-labs/aspnet-mvc-4-dependency-injection)或[存储库和单元的工作模式](http://www.asp.net/mvc/tutorials/getting-started-with-ef-using-mvc/advanced-entity-framework-scenarios-for-an-mvc-web-application#repo)，使用[日志记录接口](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/monitoring-and-telemetry#log)，使用 [EF Code First 迁移](http://www.asp.net/mvc/tutorials/getting-started-with-ef-using-mvc/migrations-and-deployment-with-the-entity-framework-in-an-asp-net-mvc-application)管理数据模型更改，并使用 [EF 连接复原](http://www.asp.net/mvc/tutorials/getting-started-with-ef-using-mvc/connection-resiliency-and-command-interception-with-the-entity-framework-in-an-asp-net-mvc-application)管理暂时性的网络错误。

### 缩放 Web 作业
Web 作业在 Web 应用的上下文中运行，并且不可单独缩放。例如，如果你有一个标准 Web 应用实例且只运行后台进程的一个实例，并且该实例使用某些服务器资源（CPU、内存等），从而这些资源也可用于提供 Web 内容。

如果流量因一天中的时间或一周中的某天而变化，并且需要执行的后端处理可以等待，则可以安排 Web 作业在低流量期间运行。如果该解决方案的负载仍然太高，可以在针对该用途专用的 Web 应用中以 Web 作业形式运行后端。然后，可以独立于前端 Web 应用缩放后端 Web 应用。

有关详细信息，请参阅[缩放 Web 作业](/documentation/articles/websites-webjobs-resources/#scale)。

### 避免因 Web 应用超时而导致其关闭
若要确保 Web 作业始终在 Web 应用的所有实例上运行，必须启用 [AlwaysOn](http://weblogs.asp.net/scottgu/archive/2014/01/16/windows-azure-staging-publishing-support-for-web-sites-monitoring-improvements-hyper-v-recovery-manager-ga-and-pci-compliance.aspx) 功能。

### 在 Web 作业的外部使用 WebJobs SDK
使用 WebJobs SDK 的程序无需在 Azure 中的 Web 作业内运行。它可以在本地运行，也可以在其他环境（例如云服务辅助角色或 Windows 服务）中运行。但是，只能通过 Azure Web 应用访问 WebJobs SDK 仪表板。若要使用仪表板，需要通过在经典管理门户的“配置”选项卡上设置 AzureWebJobsDashboard 连接字符串，以将 Web 应用连接到所用的存储帐户。然后，可以使用以下 URL 访问仪表板：

https://{webappname}.scm.chinacloudsites.cn/azurejobs/#/functions

有关详细信息，请参阅[获取仪表板以使用 WebJobs SDK 进行本地开发](http://blogs.msdn.com/b/jmstall/archive/2014/01/27/getting-a-dashboard-for-local-development-with-the-webjobs-sdk.aspx)，但注意显示了旧式连接字符串名称。

### 更多 Web 作业文档
有关详细信息，请参阅 [Azure Web 作业文档资源](/documentation/articles/websites-webjobs-resources/)。

<!---HONumber=Mooncake_1128_2016-->