<properties linkid="develop-net-tutorials-multi-tier-web-site-2-download-and-run" pageTitle="Azure 云服务教程：ASP.NET MVC Web 角色、辅助角色、Azure 存储表、队列和 Blob" metaKeywords="Azure tutorial, Azure storage tutorial, Azure multi-tier tutorial, MVC Web Role tutorial, Azure worker role tutorial, Azure blobs tutorial, Azure tables tutorial, Azure queues tutorial" description="了解如何使用 ASP.NET MVC 和 Azure 创建多层应用程序。该应用程序运行在云服务中，使用 Web 角色和辅助角色，并使用 Azure 存储表、队列和 Blob。" metaCanonical="" services="cloud-services,storage" documentationCenter=".NET" title="Azure 云服务教程：ASP.NET MVC Web 角色、辅助角色、Azure 存储表、队列和 Blob" authors="riande,tdykstra" solutions="" manager="wpickett" editor="mollybos" />

# 配置和部署 Azure 电子邮件服务应用程序 - 第 2 部分（共 5 部分）

这是分为五部分的一系列教程的第二个教程，这些教程讲述如何生成和部署 Azure 电子邮件服务示例应用程序。有关此应用程序和教程系列的信息，请参阅[本系列第一个教程][本系列第一个教程]。

在本教程中，你将学习：

-   如何通过安装 Azure SDK 来设置你的计算机，以便进行 Azure 开发。
-   如何在你的本地计算机上配置和测试 Azure 电子邮件服务应用程序。
-   如何将应用程序发布到 Azure。
-   如何使用 Visual Studio 或 Azure 存储资源管理器查看和编辑 Azure 表、队列和 Blob。
-   如何配置跟踪和查看跟踪数据。
-   如何通过增加辅助角色实例的数目来扩展应用程序。

### 教程章节

-   [先决条件][先决条件]
-   [设置开发环境][设置开发环境]
-   [创建 Azure 存储帐户][创建 Azure 存储帐户]
-   [安装 Azure 存储资源管理器][安装 Azure 存储资源管理器]
-   [创建云服务][创建云服务]
-   [下载并运行已完成的解决方案][下载并运行已完成的解决方案]
-   [在 Visual Studio 中查看开发人员存储][在 Visual Studio 中查看开发人员存储]
-   [针对 Azure 存储空间配置应用程序][针对 Azure 存储空间配置应用程序]
-   [将应用程序部署到 Azure][将应用程序部署到 Azure]
-   [将应用程序从过渡型升级到生产型][将应用程序从过渡型升级到生产型]
-   [将应用程序配置为使用 SendGrid][将应用程序配置为使用 SendGrid]
-   [配置和查看跟踪数据][配置和查看跟踪数据]
-   [添加另一个辅助角色实例来处理增加的负载][添加另一个辅助角色实例来处理增加的负载]

## 先决条件

本教程演示如何使用下述任何一种产品配置你的计算机来进行 Azure 开发，以及如何将 Microsoft Azure 电子邮件服务应用程序部署到 Microsoft Azure 云服务：

-   Visual Studio 2012
-   Visual Studio 2012 Express for Web
-   Visual Studio 2010
-   Visual Web Developer Express 2010。

> [WACOM.NOTE] 本教程编写完以后，Visual Studio 2013 发布，而 Azure 管理门户和 SDK 则进行了更新。如果你使用的是 Visual Studio 2013 和最新版 SDK，则必须执行不同的操作，在这种情况下，我们添加了类似这样的注释来提醒你。这些注释是在 2014 年 3 月编写的，经过修改的操作过程已使用 SDK 版本 2.3 进行测试。

你可以免费注册一个 Azure 帐户，而且，如果你还没有 Visual Studio 2013，则此 SDK 会自动安装 Visual Studio 2013 for Web Express。因此，你可以不支付任何费用就开始 Azure 开发。
你可以创建一个[免费试用帐户][免费试用帐户]或[激活 MSDN 订户权益][激活 MSDN 订户权益]。

> [WACOM.NOTE] 在下面关于 SDK 安装的部分，正确的链接（如果你使用的是 Visual Studio 2013）为 <http://go.microsoft.com/fwlink/?LinkID=324322>。

[WACOM.INCLUDE [install-sdk-2012-only](../includes/install-sdk-2012-only.md)]

## <a name="createWASA"></a><span class="short-header">创建存储帐户</span>创建 Azure 存储帐户

在 Visual Studio 中运行示例应用程序时，你可以访问云端 Azure 开发存储或 Azure 存储帐户中的表、队列和 Blob。开发存储使用 SQL Server Express LocalDB 数据库来模拟云中 Azure 存储空间的工作方式。在本教程中，你一开始将使用开发存储，然后学习在 Visual Studio 中运行应用程序时如何配置应用程序以使用云存储帐户。在教程的这一部分，你需要创建一个 Azure 存储帐户，以便配置在教程后面要用到的 Visual Studio。

1.  在浏览器中，打开 [Azure 管理门户][Azure 管理门户]。

2.  在 [Azure 管理门户][Azure 管理门户]中，单击“存储”，然后单击“新建”。

![新建存储][新建存储]

1.  单击“快速创建”。

![快速创建][快速创建]

1.  在 URL 输入框中，输入 URL 前缀。

    此前缀加上在框下看到的文本将是你的存储帐户的唯一 URL。如果你输入的前缀已由他人使用，你会看到文本框上方显示“此存储名称已被使用”，此时必须选择其他前缀。

2.  将区域设置为你要在其中部署该应用程序的区域。

3.  将“复制”下拉框设置为“本地冗余”。

    为存储帐户启用地域复制时，会将存储内容复制到辅助位置，这样就能够在主要位置发生重大灾难时故障转移到该位置。地域复制可能会产生额外的成本。对于测试和开发帐户，你通常不希望因为地域复制而付款。有关详细信息，请参阅[如何管理存储帐户][如何管理存储帐户]。

4.  单击**“创建存储帐户”**。

    在下图中，存储帐户是使用 `aestest3.core.windows.net` 创建的。

    ![使用 URL 前缀创建存储][使用 URL 前缀创建存储]

    此步骤可能需要几分钟才能完成。在你等待时，你可以重复这些步骤，创建生产型存储帐户。为了方便，通常可以将一个测试存储帐户用于本地开发，一个测试存储帐户用于在 Azure 中进行测试，另外再使用一个生产型存储帐户。

5.  单击你在上一步中创建的测试帐户，然后单击“管理访问密钥”图标。

    ![管理密钥][管理密钥]

    ![密钥 GUID][密钥 GUID]

    在本教程中，你将需要**主访问密钥**或**辅助访问密钥**。你可以在存储连接字符串中使用其中一个密钥。

    由于有两个密钥，因此你可以定期更改所使用的密钥，而不会造成实时应用程序出现服务中断的情况。你可以重新生成当前未使用的密钥，然后将应用程序中的连接字符串更改为使用重新生成的密钥。如果只有一个密钥，则当你重新生成密钥时，应用程序会失去与存储帐户的连接。显示在图像中的密钥将不再有效，因为在捕获该图像后，重新生成了这些密钥。

6.  请将其中一个密钥复制到剪贴板中，以便在下一部分使用。

## <a name="installASE"></a><span class="short-header">安装 ASE</span>安装 Azure 存储资源管理器

**Azure 存储资源管理器** (ASE) 是一个工具，可用于查询和更新 Azure 存储表、队列和 Blob。你将在这一系列教程中使用它来验证这些数据是否已正确更新，以及使用它来创建测试数据。

1.  安装 [Azure 存储资源管理器][Azure 存储资源管理器]。

    屏幕快照显示的是 ASE 4，但是你也可以根据喜好安装 ASE 5。

2.  启动 **Azure 存储资源管理器**，然后单击“添加帐户”。

    ![添加 ASE 帐户][添加 ASE 帐户]

3.  输入测试存储帐户的名称，然后粘贴先前复制的密钥。

4.  单击“添加存储帐户”。

    ![添加 ASE 帐户][1]

也可以使用其他适用于 Azure 存储空间的工具。有关详细信息，请参阅 [Microsoft Azure 存储资源管理器 (2014)][Microsoft Azure 存储资源管理器 (2014)]。

## <a name="createcloudsvc"></a><span class="short-header">创建云服务</span>创建云服务

1.  在浏览器中，打开 [Azure 管理门户][Azure 管理门户]。

2.  单击“云服务”，然后单击“新建”图标。

    ![快速云][快速云]

3.  单击“快速创建”。

4.  在 URL 输入框中，输入 URL 前缀。

    和存储 URL一样，此 URL 必须是唯一的，而如果你选择的前缀已被他人使用，你将获得一条错误消息。

5.  将区域设置为你要在其中部署该应用程序的区域。

    应在你创建存储帐户的同一区域创建云服务。当云服务和存储帐户位于不同的数据中心（不同区域）时，延迟将增加，并且你需要为数据中心外的带宽付费。数据中心内的带宽是免费的。

    Azure 地缘组实际上是一种机制，目的是最小化数据中心内不同资源之间的距离，这样可以降低延迟。本教程不使用地缘组。有关详细信息，请参阅[如何在 Azure 中创建地缘组][如何在 Azure 中创建地缘组]。

6.  单击“创建云服务”。

    在下图中，使用 URL aescloud.cloudapp.net 创建了一项云服务。

    ![使用 URL 前缀创建存储][2]

    你可以继续执行下一步而无需等待此步骤完成。

## <a name="downloadcnfg"></a><span class="short-header">下载并运行</span>下载并运行已完成的解决方案

1.  下载并解压缩[已完成的解决方案][已完成的解决方案]。

2.  使用提升的权限启动 Visual Studio。

    允许 Visual Studio 在本地运行 Microsoft Azure 项目的计算模拟器需要提升的权限。

    > [WACOM.NOTE] 使用 SDK 2.3 和更高版本时，默认的 Azure 计算模拟器不需要提升的权限，但此项目仍配置为使用原始计算模拟器，因此它仍然需要提升的权限。

3.  还原 NuGet 包。

    为了降低下载大小，在提供完成的解决方案时没有包含已安装 NuGet 包的程序集或其他内容。当你打开并生成解决方案时，NuGet 自动获取所有包内容。为了使这种方式生效，你必须在 Visual Studio 中启用 NuGet 包还原选项。如果你尚未启用 NuGet 包还原，请执行以下步骤。

    > [WACOM.NOTE] 在 Visual Studio 2013 中，默认设置是启用包自动还原，但由于你打开的是 Visual Studio 2012 项目，还原不会自动发生。按照下面的说明，请一直等到打开解决方案，然后打开“管理 NuGet 包”对话框。你将看到右上角的“还原”按钮：单击该按钮对包进行还原。

    1.  在“工具”菜单中，单击“库程序包管理器”，然后单击“管理解决方案的 NuGet 包”。

    2.  在“管理 NuGet 包”对话框的左下角，单击“设置”。

    3.  在“选项”对话框左窗格的“程序包管理器”下选择“常规”。

    4.  选择“在生成期间允许 NuGet 下载缺失的包”。

    ![启用 NuGet 包还原][启用 NuGet 包还原]

4.  从**“文件”**菜单中，选择**“打开项目”**，导航到下载解决方案的位置，然后打开解决方案文件。

5.  在**“解决方案资源管理器”**中，请确保选择 **AzureEmailService** 作为启动项目。

6.  按 Ctrl+F5 运行应用程序。

    > [WACOM.NOTE] 为了启用要生成的解决方案，你需要添加对当前 SDK 程序集的引用。在解决方案资源管理器中，右键单击“MvcWebRole”项目，然后单击“添加”--“引用”。在**“程序集”**下单击**“扩展”**。选择 *Microsoft.MicrosoftAzure.Diagnostics* 和 *Microsoft.MicrosoftAzure.ServiceRuntime*，然后单击**“确定”**。为 WorkerRoleA 和 WorkerRoleB 项目执行相同的操作。

    此时浏览器中会显示应用程序主页。

    ![运行应用程序。][运行应用程序。]

7.  单击**“新建”**。

8.  输入一些测试数据，然后单击**“创建”**。

    ![运行应用程序。][3]

9.  再创建几个邮件列表项。

    ![邮件列表索引页][邮件列表索引页]

10. 单击**“订户”**，然后添加一些订户。将**“已验证”**设置为 `true`。

    ![订户索引页][订户索引页]

11. 准备添加邮件，方法是：创建一个 *.txt* 文件，其中包含你想要发送的电子邮件的正文。然后创建一个 *.htm* 文件，其中包含相同的文本但带有一些 HTML（例如将邮件中的某个单词设置为粗体或斜体）。你将在下一步使用这些文件。

12. 单击**“邮件”**，然后添加一些邮件。选择在前一步骤中创建的文件。不要更改计划的日期（默认为将来的某一周）。该应用程序无法发送邮件，除非你配置了 SendGrid。

![邮件创建页][邮件创建页]
 ![邮件索引页][邮件索引页]

你已输入并查看的数据将存储在 Azure 开发存储中。开发存储使用 SQL Server Express LocalDB 数据库来模拟云中 Azure 存储空间的工作方式。应用程序使用开发存储是因为，那是你在下载项目时项目根据配置可以使用的东西。该设置存储在 **AzureEmailService** 项目的 *.cscfg* 文件中。*ServiceConfiguration.Local.cscfg* 文件用于确定当你在 Visual Studio 中本地运行应用程序时应使用什么，而 *ServiceConfiguration.Cloud.cscfg* 文件则用于确定当你将应用程序部署到云中时应使用什么。稍后你将了解如何配置应用程序，以便使用以前创建的 Azure 存储帐户。

## <a name="StorageExpVS"></a><span class="short-header">开发人员存储</span>在 Visual Studio 中查看开发人员存储

**服务器资源管理器**中的 **Azure 存储空间**浏览器提供针对 Azure 存储资源的很方便的只读视图。

1.  从 Visual Studio 中的“视图”菜单中，选择**“服务器资源管理器”**。

2.  展开**“Azure 存储空间”**节点下面的**“(开发)”**节点。

    > [WACOM.NOTE] 在当前的 SDK 中，展开**“Microsoft Azure”**/**“存储”**/**“(开发)”**。

3.  展开**“表”**以查看你在前面的步骤中创建的表。

    ![服务器资源管理器][服务器资源管理器]

4.  双击 **MailingList** 表。

    ![VS 存储资源管理器][VS 存储资源管理器]

    请注意窗口如何显示表中的不同架构。`MailingList` 实体具有 `Description` 和 `FromEmailAddress` 属性，而 `Subscriber` 实体则具有 `Verified` 属性（此外还有 `SubscriberGUID`，该属性并没有显示，因为图像不够宽）。该表包含所有属性的列，如果给定表行是针对某个没有给定属性的实体的，则该单元格为空。

不能使用 Visual Studio 中的存储浏览器来更新或删除 Azure 存储资源。你可以使用 [Azure 存储资源管理器][Azure 存储资源管理器]来更新或删除开发存储资源。（若要将 Azure 存储资源管理器配置为使用开发存储，请单击**“添加存储帐户”**对话框中的**“开发人员存储”**复选框。）

> [WACOM.NOTE] 使用最新的 SDK 时，你可以更新并读取**服务器资源管理器**中的开发存储。

## <a name="conf4azureStorage"></a><span class="short-header">使用你的存储帐户</span>将应用程序配置为使用 Azure 存储帐户

接下来，你将了解如何配置应用程序，以便它在 Visual Studio 中运行时使用你的 Azure 存储帐户而不是开发存储。在 Visual Studio 中，可以采用比这更新的方式来这样做，这种方式是在 SDK 的 1.8 版中引入的；较旧的方式则是从 Azure 管理门户复制并粘贴设置。下面的步骤演示了用于配置存储帐户设置的较新方法。

1.  在**“解决方案资源管理器”**中，右键单击 **AzureEmailService** 项目中**“角色”**下的**“MvcWebRole”**，然后单击**“属性”**。

    ![右键单击属性][右键单击属性]

2.  单击“设置”选项卡。在**“服务配置”**下拉框中，选择**“本地”**。

3.  选择 **StorageConnectionString** 条目，你将看到一个省略号 (**...**) 按钮，位于行的右端。单击省略号按钮以打开**“存储帐户连接字符串”**对话框。

    ![右键单击属性][4]

4.  在**“创建存储连接字符串”**对话框中，单击**“你的订阅”**，然后单击**“下载发布设置”**。

    > [WACOM.NOTE] 在最新 SDK 中，可以通过此对话框登录到 Azure，然后直接选择要使用的存储帐户。

    ![右键单击属性][5]

    Visual Studio 将通过 Azure 门户下载发布设置页的 URL 启动默认浏览器的新实例。如果你尚未登录到门户中，系统会提示你登录。登录后，浏览器会提示你保存发布设置。记下保存设置的位置。

![发布设置][发布设置]

1.  在**“创建存储连接字符串”**对话框中，单击**“导入”**，然后导航到你在上一步保存的发布设置文件。

2.  选择想要使用的订阅和存储帐户，然后单击**“确定”**。

    ![选择存储帐户][选择存储帐户]

3.  按照你所使用的针对 `StorageConnectionString` 连接字符串的相同过程，设置 `Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString` 连接字符串。

    你无需再次下载发布设置文件。当你单击 `Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString` 连接字符串的省略号时，你会发现，**“创建存储连接字符串”**对话框会记住你的订阅信息。当你单击**“你的订阅”**单选按钮时，你需要做的就是选择你此前已经选择过的同一**订阅**和**帐户名**，然后单击**“确定”**。

4.  按照你用于 MvcWebRole 角色的两个连接字符串的同一过程，设置 WorkerRoleA 角色和 workerRoleB 角色的连接字符串。

### 用于配置存储帐户凭据的手动方法

下面的过程演示了如何使用手动方式来配置存储帐户设置。如果你使用的是前一过程中显示的自动方法，则可以跳过此过程，也可以通读此过程，了解自动方法在后台为你执行的操作。

1.  在浏览器中，打开 [Azure 管理门户][Azure 管理门户]。

2.  单击**“存储”**选项卡，然后单击你在上一步创建的测试帐户，再单击**“管理密钥”**图标。

![管理密钥][管理密钥]

![密钥 GUID][密钥 GUID]

1.  复制主访问密钥或辅助访问密钥。

2.  在**“解决方案资源管理器”**中，右键单击 **AzureEmailService** 项目中**“角色”**下的**“MvcWebRole”**，然后单击**“属性”**。

![右键单击属性][右键单击属性]

1.  单击“设置”选项卡。在**“服务配置”**下拉框中，选择**“本地”**。

2.  选择 **StorageConnectionString** 条目，你将看到一个省略号 (**...**) 按钮，位于行的右端。单击省略号按钮以打开**“存储帐户连接字符串”**对话框。

![右键单击属性][4]

1.  在**“创建存储连接字符串”**对话框中，选择**“手动输入的凭据”**单选按钮。输入你的存储帐户的名称以及从门户中复制的主访问密钥或辅助访问密钥。

2.  单击“确定”。

你可以使用同一过程来配置辅助角色的设置，也可以通过编辑配置文件将 Web 角色设置传播到辅助角色。以下步骤介绍如何编辑配置文件。（这仍然是用于设置存储凭据的手动方法的一部分，不过如果你已使用自动方法将设置传播到辅助角色，则不需执行此操作。）

1.  打开位于 **AzureEmailService** 项目中的 **ServiceConfiguration.Local.cscfg** 文件。

    在 `MvcWebRole` 的 `Role` 元素中，你会看到一个 `ConfigurationSettings` 元素，该元素包含你使用 Visual Studio UI 更新的设置。

          <Role name="MvcWebRole">
            <Instances count="1" />
            <ConfigurationSettings>
              <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="DefaultEndpointsProtocol=https;AccountName=[name];AccountKey=[Key]" />
              <Setting name="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=aestest;AccountKey=[Key]" />
            </ConfigurationSettings>
          </Role>

    在适用于这两个辅助角色的 `Role` 元素中，你将看到同样的两个连接字符串。

2.  从 `WorkerRoleA` 和 `WorkerRoleB` 元素中删除这两个连接字符串的 `Setting` 元素，然后在其位置复制并粘贴来自 `MvcWebRole` 元素的 `Setting` 元素。

有关配置文件的详细信息，请参阅[配置 Azure 项目][配置 Azure 项目]

### 测试已配置为使用你的存储帐户的应用程序

1.  按 Ctrl+F5 运行应用程序。按照你此前在本教程中所做的那样，通过单击**“邮件列表”**、**“订户”**和**“邮件”**链接输入一些数据。

你现在可以使用 **Azure 存储资源管理器**或**服务器资源管理器**来查看该应用程序在 Azure 表中输入的数据。

### 使用 Azure 存储资源管理器来查看输入到你的存储帐户中的数据

1.  打开 **Azure 存储资源管理器**。

2.  选择你此前为其输入了凭据的存储帐户。

3.  在“存储类型”下，单击“表”。

4.  选择 `MailingList` 表，然后单击“查询”以查看你在应用程序的“邮件列表”和“订户”页上输入的数据。

![ASE][ASE]

### 使用服务器资源管理器来查看输入到你的存储帐户中的数据

1.  在**“服务器资源管理器”**（或**“数据库资源管理器”**）中，右键单击**“Azure 存储空间”**，然后单击**“添加新的存储帐户”**。

2.  按照你之前用过的同一过程，设置你的存储帐户凭据。

3.  展开**“Azure 存储空间”**下的新节点，查看在你的 Azure 存储帐户中存储的数据。

    ![ASE][6]

### 禁用 Azure 存储模拟器自动启动所需的可选步骤

如果不使用存储模拟器，则可以通过禁用 Azure 存储模拟器的自动启动来缩短项目启动时间，并使用更少的本地资源。

1.  在**“解决方案资源管理器”**中，右键单击 **AzureEmailService** 云项目，然后选择“属性”。

    ![选择云项目属性][选择云项目属性]

2.  选择“开发”选项卡。

3.  将**“启动 Azure 存储模拟器”**设置为 **False**。

    ![禁用存储模拟器的自动启动][禁用存储模拟器的自动启动]

    **注意**：仅当你不使用存储模拟器时，才应将此设置为 false。

    也可以通过此窗口来更改**服务配置**文件。当你将应用程序的运行方式从**“本地”**更改为**“云”**（从 *ServiceConfiguration.Local.cscfg* 更改为 *ServiceConfiguration.Cloud.cscfg*）时，需要使用该配置文件。

4.  在 Windows 系统任务栏中，右键单击计算模拟器图标，然后单击“关闭存储模拟器”。

    ![ASE][7]

## <a name="sendGrid"></a><span class="short-header">SendGrid</span>将应用程序配置为使用 SendGrid

示例应用程序使用 SendGrid 发送电子邮件。若要使用 SendGrid 发送电子邮件，你需要设置 SendGrid 帐户，然后需要使用 SendGrid 凭据更新配置文件。

<div class="note"><p><strong>注意：</strong>如果你不想使用 SendGrid，或者不能使用 SendGrid，你可以轻松地将其替换自己的电子邮件服务。使用 SendGrid 的代码单独置于辅助角色 B 的两个方法中。[教程 5][tut5] 说明了你必须进行何种更改才能实施发送电子邮件的其他方法。如果你希望这样做，则可跳过此过程，继续学习本教程；除电子邮件的实际发送功能外，本应用程序中的所有其他功能（网页、电子邮件计划等）都可以使用。</p></div>

### 创建 SendGrid 帐户

1.  按照[如何在 Azure 中使用 SendGrid 发送电子邮件][如何在 Azure 中使用 SendGrid 发送电子邮件]中的说明注册一个免费帐户。

### 更新辅助角色属性中的 SendGrid 凭据

在本教程前面当你为 Web 角色和两个辅助角色设置存储帐户凭据时，你可能已注意到辅助角色 B 有三项设置是 Web 角色或辅助角色 A 所没有的。现在，你可以使用同一个 UI 来配置这三项设置（在**“服务配置”**下拉列表中选择**“云”**）。

下面的步骤演示了如何使用另一种方法通过编辑配置文件来设置属性。

1.  编辑 `AzureEmailService` 项目中的 *ServiceConfiguration.Cloud.cscfg* 文件，然后将你在上一步获得的 SendGrid 用户名和密码值输入具有这些设置的 `WorkerRoleB` 元素中。下面的代码演示 WorkerRoleB 元素。

    ![SendGridSettings][SendGridSettings]

2.  此外还有 AzureMailServiceURL 设置。将此值设置为你在创建 Azure 云服务时选择的 URL，例如：“http://aescloud.cloudapp.net”。

通过更新云配置文件，你可以配置应用程序在云中运行时将使用的设置。如果你想要应用程序在本地运行时发送电子邮件，则还需更新 *ServiceConfiguration.Local.cscfg* 文件。

## <a name="deployAz"></a><span class="short-header">部署到 Azure</span>将应用程序部署到 Azure

若要部署应用程序，你可以在 Visual Studio 中创建一个程序包，然后通过 Azure 管理门户将其上载，也可以直接通过 Visual Studio 进行发布。在本教程中，你将使用发布方法。

你需要先将应用程序发布到过渡环境，然后将过渡部署提升为生产部署。

### 实施 IP 限制

部署到过渡环境时，应用程序可供知道 URL 的任何人公开访问。因此，你的第一步是实施 IP 限制，确保未经授权的人无法使用它。在生产应用程序中，你会实施 ASP.NET 成员身份系统这样的身份验证和授权机制，不过在示例应用程序中，我们省略了这些功能，目的是简化设置、部署和测试。

1.  打开 *Web.Release.config* 文件（位于 `MvcWebRole` 项目的根文件夹中），将 **ipAddress** 属性值 127.0.0.1 替换为你的 IP 地址。（若要在解决方案资源管理器中查看 **Web.Release.config** 文件，你必须展开 *Web.config* 文件。）

    你可以在[必应][必应]或其他搜索引擎中通过搜索“查找我的 IP”来查找你的 IP 地址。

    发布应用程序时，将应用在 *Web.release.config* 文件中指定的转换，并更新已部署到云的 *web.config* 文件中的 IP 限制元素。创建程序包后，你可以查看 *AzureEmailService\\MvcWebRole\\obj\\Release\\TransformWebConfig\\transformed* 文件夹中的已转换 *web.config* 文件。

### 配置应用程序，以便应用程序在云中运行时使用你的存储帐户

在本教程前面当你为 Web 角色和两个辅助角色设置存储帐户凭据时，你设置了本地运行应用程序时需要使用的凭据。现在，你需要设置当你在云中运行应用程序时要使用的存储帐户凭据。

就本测试运行来说，你用于云的凭据将与在本地运行时使用的凭据相同。如果你部署的是生产应用程序，则通常会使用与测试时所用帐户不同的帐户进行生产。此外，对于生产来说，最佳做法是对诊断 connectionString 使用一个与存储连接字符串所用帐户不同的帐户，但就本测试运行来说，你将使用同一个帐户。

你可以使用同一 UI 来配置连接字符串（只需确保你在“服务配置”下拉列表中选择“云”）。替代方法是编辑配置文件，如以下步骤所述。

1.  打开 **AzureEmailService** 项目中的 *ServiceConfiguration.Local.cscfg* 文件，然后复制 `StorageConnectionString` 和 `Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString` 的 `Setting` 元素。

2.  打开 **AzureEmailService** 项目中的 *ServiceConfiguration.Cloud.cscfg* 文件，然后将复制的元素粘贴到 `MvcWebRole`、`WorkerRoleA` 和 `WorkerRoleB` 的 `Configuration Settings` 元素中，替换已存在那里的 `Setting` 元素。

3.  确保 Web 角色和两个辅助角色元素都定义相同的连接字符串。

### 发布应用程序

1.  以管理员身份启动 Visual Studio 并打开 **AzureEmailService** 解决方案（如果尚未打开）。

2.  右键单击 **AzureEmailService** 云项目并选择**“发布”**。

    ![程序包][程序包]

    将显示“发布 Azure 应用程序”对话框。

    ![云程序包][云程序包]

3.  如果你此前使用了自动方法来导入存储帐户凭据，你的 Azure 订阅将位于下拉列表中，你可以选择它，然后单击“下一步”。否则，请单击“登录以下载凭据”，然后按照[针对 Azure 存储空间配置应用程序][针对 Azure 存储空间配置应用程序]中的说明下载并导入你的发布设置。

4.  在**“通用设置”**选项卡中，验证**“云服务”**下拉列表中的设置。

5.  在**“环境”**下拉列表中，将**“生产”**更改为**“过渡”**。

    ![仪表板][仪表板]

6.  为**“生成配置”**保留默认的**“版本”**设置，为**“服务配置”**保留默认的**“云”**设置。

    本教程可以使用“高级”选项卡中的默认设置。**“高级”**选项卡上是一组用于开发和测试的设置。有关“高级”选项卡的详细信息，请参阅[发布 Azure 应用程序向导][发布 Azure 应用程序向导]。

7.  单击“下一步”。

8.  在向导的“摘要”步骤中，单击“保存”图标（即磁盘图标，显示在“目标”配置文件下拉列表的右侧）以保存发布设置。

    下次发布应用程序时，将使用保存的设置，你不需要再次完成发布向导操作。

9.  复查设置，然后单击“发布”。

    ![发布][发布]

**“Azure 活动日志”**窗口在 Visual Studio 中打开。

1.  单击右箭头图标以展开部署详细信息。

    ![发布][8]
    ![发布][9]

    部署可能需要大约 5 分钟或更长时间才能完成。

2.  当部署状态为完成时，单击**“网站 URL”**以启动应用程序。

    ![仪表板][10]

3.  在**“邮件列表”**、**“订户”**和**“邮件”**网页中输入一些数据，以便测试应用程序。

    **注意**：完成对应用程序的测试后，请删除该应用程序，避免为不使用的资源付费。如果你使用的是[Azure 免费试用帐户][Azure 免费试用帐户]，三个已部署的角色会在几周内用光你的每月配额限制。若要使用 Azure 管理门户删除某个部署，请选择云服务，单击页面底部的“删除”，然后选择生产部署或过渡部署。

    ![发布][11]

4.  在 Visual Studio 的“Azure 活动日志”中，选择“在服务器资源管理器中打开”。

    在“服务器资源管理器”中的**“Azure 计算”**下，你可以监视部署情况。如果你在**“发布 Azure 应用程序”**向导中选择了“为所有角色启用远程桌面”，则可以右键单击某个角色实例，然后选择“使用远程桌面进行连接”。

    ![发布][12]

## <a name="swap"></a><span class="short-header">生产</span>将应用程序从过渡型升级到生产型

1.  在 [Azure 管理门户][Azure 管理门户]中，单击左窗格中的“云服务”图标，然后选择你的云服务。

2.  单击“交换”。

3.  单击**“是”**完成 VIP（虚拟 IP）交换。此步骤可能需要几分钟才能完成。

    ![仪表板][13]

4.  当交换完成后，单击左窗格中的“云服务”图标，然后选择你的云服务。

5.  在“生产”部署的**“仪表板”**选项卡中向下滚动到页面左下角的“速览”部分。请注意，**网站 URL** 已从 GUID 前缀更改为你的云服务的名称。

    ![仪表板][14]

6.  单击**“网站 URL”**下的链接或者将其复制并粘贴到浏览器中，以便对生产型应用程序进行测试。

    如果尚未更改存储帐户设置，则当你在云中运行应用程序时，将显示你在测试过渡版应用程序时输入的数据。

## <a name="trace"></a><span class="short-header">跟踪</span>配置和查看跟踪数据

跟踪是一种很重要的工具，可用于调试云应用程序。在本部分教程中，你将了解如何查看跟踪数据。

1.  确保诊断连接字符串已配置为使用 Azure 存储帐户而不是使用开发存储。

    如果你在操作时遵循了教程前面的说明，它们将是一样的。你可以验证它们是否相同，方法一是使用 Visual Studio UI（角色的“属性”中的“设置”选项卡），方法二是查看 *ServiceConfiguration.\*.cscfg* 文件。

**注意：**最佳做法是跟踪数据和生产数据使用不同的存储帐户，但在本教程中，为了简便起见，你将用于生产的同一个帐户配置用于跟踪。

1.  在 Visual Studio 的 **WorkerRoleA** 项目中打开 *WorkerRoleA.cs*，搜索 `ConfigureDiagnostics`，然后检查 `ConfigureDiagnostics` 方法。

        private void ConfigureDiagnostics()
        {
            DiagnosticMonitorConfiguration config = DiagnosticMonitor.GetDefaultInitialConfiguration();
            config.ConfigurationChangePollInterval = TimeSpan.FromMinutes(1d);
            config.Logs.BufferQuotaInMB = 500;
            config.Logs.ScheduledTransferLogLevelFilter = LogLevel.Verbose;
            config.Logs.ScheduledTransferPeriod = TimeSpan.FromMinutes(1d);

            DiagnosticMonitor.Start(
                "Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString",
                config);
        }

    在该代码中，`DiagnosticMonitor` 配置为存储多达 500 MB 的跟踪信息（超过 500 MB 以后，将覆盖最旧的数据）和所有跟踪消息 (LogLevel.Verbose)。`ScheduledTransferPeriod` 每分钟都会将跟踪数据传输到存储中。必须设置 `ScheduledTransferPeriod` 才能保存跟踪数据。

    每个辅助角色和 Web 角色中的 `ConfigureDiagnostics` 方法用于配置跟踪侦听器，以便在你调用跟踪 API 时记录数据。有关详细信息，请参阅[在 Microsoft Azure 云应用程序中使用跟踪][在 Microsoft Azure 云应用程序中使用跟踪]

2.  在**服务器资源管理器**中，针对你此前添加的存储帐户双击 **WADLogsTable**（展开“**存储**/**yourstorageaccountname**/**表**”）。你可以输入 [WCF 数据服务筛选器][WCF 数据服务筛选器]以限制显示的实体。在下图中，仅显示警告和错误消息。

    ![仪表板][15]

## <a name="addRole"></a><span class="short-header">添加角色实例</span>添加另一个辅助角色实例来处理增加的负载

有两种方法可用于缩放 Azure 角色中的计算资源，一种方法是指定[虚拟机大小][虚拟机大小]，另一种方法是指定运行的虚拟机的实例计数。

虚拟机 (VM) 大小在 *ServiceDefinition.csdef* 文件的 `WebRole` 或 `WorkerRole` 元素的 `vmsize` 属性中指定。默认设置为 `Small`，可以为你提供一个核心和 1.75 GB 的 RAM。对于那些多线程且使用大量内存、磁盘和带宽的应用程序，你可以通过增加 VM 大小来提高性能。例如，`ExtraLarge` VM 具有 8 个 CPU 核心和 14 GB 的 RAM。在单台计算机上提高内存、cpu 核心数、磁盘空间和带宽称为*向上扩展*。适合进行向上扩展的例子很多，其中包括使用[异步方法][异步方法]的 ASP.NET Web 应用程序。请参阅[虚拟机大小][虚拟机大小]，了解按 VM 大小提供的各个资源的相关说明。

此应用程序中的辅助角色 B 是高负载情况下的限制组件，因为它只会发送电子邮件的工作。（辅助角色 A 只创建队列消息，而这不属资源密集型操作。）由于辅助角色 B 不是多线程，且内存需求量不大，因此不适合进行向上扩展。辅助角色 B 可以通过增加实例数进行线性扩展（也就是说，实例数增加一倍时性能也差不多提高一倍）。增加计算实例数称为“向外扩展”。每个实例都需要成本，因此只应在应用程序需要时向外扩展。

向外扩展 Web 角色或辅助角色时，只需更新 Visual Studio UI 中的设置或直接编辑 *ServiceConfiguration.\*.cscfg* 文件。实例计数可在角色**“属性”**窗口的“配置”选项卡中指定，也可在 *.cscfg* 文件的 `Instances` 元素中指定。当你更新该设置时，你必须部署更新的配置文件，以使更改生效。而对于负载的一过性增加，你还可以在 Azure 管理门户中更改角色实例数。你也可以使用 Azure 管理 API 配置实例数。最后，你可以使用[自动缩放应用程序块][自动缩放应用程序块]来自动向外扩展，以便处理增加的负载。有关自动缩放的详细信息，请参阅[本系列最后一个教程][本系列最后一个教程]结尾处的链接。

在本教程的这一部分，你将使用管理门户向外扩展辅助角色 B，但首先你需要了解如何在 Visual Studio 中实现这一点。

若要在 Visual Studio 中实现这一点，你需要右键单击云项目中“角色”项下的角色，然后选择“属性”。

![右键单击属性][右键单击属性]

然后，你需要选择左侧的“配置”选项卡，再在“服务配置”下拉列表中选择“云”。

![实例计数][实例计数]

请注意，你也可以在此选项卡中配置 VM 大小。

以下步骤说明了如何通过 Azure 管理门户向外扩展。

1.  在 Azure 管理门户中，选择你的云服务，然后单击“缩放”。

2.  增加辅助角色 B 的实例数，然后单击**“保存”**。

    ![增加实例][增加实例]

    设置新的 VM 可能需要数分钟。

3.  选择**“实例”**选项卡，以便查看应用程序中的每个角色实例。

    ![查看实例][查看实例]

## <a name="nextsteps"></a><span class="short-header">后续步骤</span>后续步骤

你现在已经了解如何配置、部署和缩放已完成的应用程序。以下教程介绍了如何从头构建应用程序。在[下一教程][下一教程]中，你将构建 Web 角色。

有关如何使用 Azure 存储表、队列和 Blob 的其他资源的链接，请参阅[本系列最后一个教程][16]。

<div><a href="/zh-cn/develop/net/tutorials/multi-tier-web-site/3-web-role/" class="site-arrowboxcta download-cta">教程 3</a></div>

  [本系列第一个教程]: /zh-cn/develop/net/tutorials/multi-tier-web-site/1-overview/
  [先决条件]: prerequisites
  [设置开发环境]: #setupdevenv
  [创建 Azure 存储帐户]: #createWASA
  [安装 Azure 存储资源管理器]: #installASE
  [创建云服务]: #createcloudsvc
  [下载并运行已完成的解决方案]: #downloadcnfg
  [在 Visual Studio 中查看开发人员存储]: #StorageExpVS
  [针对 Azure 存储空间配置应用程序]: #conf4azureStorage
  [将应用程序部署到 Azure]: #deployAz
  [将应用程序从过渡型升级到生产型]: #swap
  [将应用程序配置为使用 SendGrid]: #sendGrid
  [配置和查看跟踪数据]: #trace
  [添加另一个辅助角色实例来处理增加的负载]: #addRole
  [免费试用帐户]: /zh-cn/pricing/free-trial/
  [激活 MSDN 订户权益]: /zh-cn/pricing/member-offers/msdn-benefits/
  [Azure 管理门户]: http://manage.windowsazure.cn
  [新建存储]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-portal-new-storage.png
  [快速创建]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-storage-quick.png
  [如何管理存储帐户]: /zh-cn/manage/services/storage/how-to-manage-a-storage-account/
  [使用 URL 前缀创建存储]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-create-storage-url-test.png
  [管理密钥]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-manage-keys.png
  [密钥 GUID]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-guid-keys.PNG
  [Azure 存储资源管理器]: http://azurestorageexplorer.codeplex.com/
  [添加 ASE 帐户]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-ase-add.png
  [1]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-ase-add2.png
  [Microsoft Azure 存储资源管理器 (2014)]: http://blogs.msdn.com/b/windowsazurestorage/archive/2014/03/11/windows-azure-storage-explorers-2014.aspx
  [快速云]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-new-cloud.png
  [如何在 Azure 中创建地缘组]: http://msdn.microsoft.com/zh-cn/library/jj156209.aspx
  [2]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-create-cloud.png
  [已完成的解决方案]: http://code.msdn.microsoft.com/Windows-Azure-Multi-Tier-eadceb36
  [启用 NuGet 包还原]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/NuGetPkgRestore.png
  [运行应用程序。]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-mailinglist1.png
  [3]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-create1.png
  [邮件列表索引页]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-mailing-list-index-page.png
  [订户索引页]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-subscribers-index-page.png
  [邮件创建页]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-message-create-page.png
  [邮件索引页]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-message-index-page.png
  [服务器资源管理器]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-serverExplorer.png
  [VS 存储资源管理器]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-wasVSdata.png
  [右键单击属性]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-rt-prop.png
  [4]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-elip.png
  [5]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-enter.png
  [发布设置]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-3.png
  [选择存储帐户]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-5.png
  [配置 Azure 项目]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee405486.aspx
  [ASE]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-ase1.png
  [6]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-se3.png
  [选择云项目属性]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-aesp.png
  [禁用存储模拟器的自动启动]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-1.png
  [7]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-se4.png
  [如何在 Azure 中使用 SendGrid 发送电子邮件]: http://www.windowsazure.com/zh-cn/develop/net/how-to-guides/sendgrid-email-service/ "SendGrid"
  [SendGridSettings]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-sg.png
  [必应]: http://www.bing.com/search?q=find+my+IP&qs=n&form=QBLH&pq=find+my+ip&sc=8-10&sp=-1&sk= "查找我的 IP"
  [程序包]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-6.png
  [云程序包]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-16.png
  [仪表板]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-7.png
  [发布 Azure 应用程序向导]: http://msdn.microsoft.com/library/windowsazure/hh535756.aspx "发布向导"
  [发布]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-8.png
  [8]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-11.png
  [9]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-9.png
  [10]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-c55.png
  [Azure 免费试用帐户]: http://www.windowsazure.com/zh-cn/pricing/free-trial/ "免费试用帐户"
  [11]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-19.png
  [12]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-12.png
  [13]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-c6.png
  [14]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-c7.png
  [在 Microsoft Azure 云应用程序中使用跟踪]: http://blogs.msdn.com/b/windowsazure/archive/2012/10/24/using-trace-in-windows-azure-cloud-applications-1.aspx "在 Microsoft Azure 中使用跟踪"
  [WCF 数据服务筛选器]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ff683669.aspx "WCF 筛选器"
  [15]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-trc.png
  [虚拟机大小]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee814754.aspx "VM 大小"
  [异步方法]: http://www.asp.net/mvc/tutorials/mvc-4/using-asynchronous-methods-in-aspnet-mvc-4 "异步 MVC"
  [自动缩放应用程序块]: /zh-cn/develop/net/how-to-guides/autoscaling/
  [本系列最后一个教程]: /zh-cn/develop/net/tutorials/multi-tier-web-site/5-worker-role-b/
  [实例计数]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-instanceCnt.png
  [增加实例]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-in3.png
  [查看实例]: ./media/cloud-services-dotnet-multi-tier-app-storage-1-download-run/mtas-in2.png
  [下一教程]: /zh-cn/develop/net/tutorials/multi-tier-web-site/3-web-role/
  [16]: /zh-cn/develop/net/tutorials/multi-tier-web-site/5-worker-role-b/#nextsteps
