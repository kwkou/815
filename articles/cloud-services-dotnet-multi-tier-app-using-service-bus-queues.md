<properties linkid="dev-net-e2e-multi-tier" urlDisplayName="Multi-Tier Application" pageTitle=".NET 多层应用程序 - Azure 教程" metaKeywords="Azure Service Bus queue tutorial, Azure queue tutorial, Azure worker role tutorial, Azure .NET queue tutorial, Azure C# queue tutorial, Azure C# worker role tutorial" description="本教程可帮助你在 Azure 中开发使用 Service Bus 队列在各层之间进行通信的多层应用程序。相关示例使用 .NET。" metaCanonical="" services="cloud-services,service-bus" documentationCenter=".NET" title="使用 Service Bus 队列创建 .NET 多层应用程序" authors="sethm" solutions="" manager="dwrede" editor="mattshel" />

# 使用 Service Bus 队列创建 .NET 多层应用程序

使用 Visual Studio 2013 和免费的 Azure SDK for .NET，
可以轻松针对 Azure 进行开发。如果你还没有 Visual Studio 2013，
则此 SDK 将自动安装 Visual Studio Express 2013，以便
你能完全免费地开始针对 Azure 进行开发。本指南假定你之前未使用过 Microsoft
Azure。完成本指南之后，
你将拥有使用多项 Azure 资源的应用程序，
该应用程序在本地环境中运行并演示多层应用程序的工作方式。

你将了解到以下内容：

-   如何通过单个下载和安装来使你的计算机能够
    进行 Azure 开发。
-   如何使用 Visual Studio 针对 Azure 进行开发。
-   如何使用 Web 角色和辅助角色在 Azure 中创建
    多层应用程序。
-   如何使用 Service Bus 队列在各层之间进行通信。

[WACOM.INCLUDE [create-account-note](../includes/create-account-note.md)]

在本教程中，你将生成多层应用程序并在 Azure 云服务中运行它。前端将为 ASP.NET MVC Web 角色，后端将为辅助角色。你可以创建与前端相同的多层应用程序，作为将部署到 Azure 网站而不是云服务的 Web 项目。有关如何以不同方式处理 Azure 网站前端的说明，请参阅[后续步骤][后续步骤]部分。

以下是已完成应用程序的屏幕快照：

![][0]

**注意：**Azure 还提供了存储队列功能。有关 Azure 存储队列和 Service Bus 队列的详细信息，请参阅 [Azure 队列和 Azure Service Bus 队列 - 比较与对照][Azure 队列和 Azure Service Bus 队列 - 比较与对照]。

## <span class="short-header">角色间通信</span>方案概述：角色间通信

若要提交处理命令，
以 Web 角色运行的前端 UI 组件需要
与以辅助角色运行的中间层逻辑进行交互。此示例使用 Service Bus 中转消息传送
在各层之间进行通信。

在 Web 层和中间层之间使用代理消息传送
将分离这两个组件。与直接消息传送（即 TCP 或 HTTP）相反，
Web 层不会直接连接到中间层，而是将工作单元
作为消息推送到 Service Bus，Service Bus 将以可靠方式保留这些工作单元，
直到中间层准备好使用和处理它们。

Service Bus 提供了两个实体以支持中转消息传送、
队列和主题。通过队列，发送到队列的每个消息
均由一个接收方使用。主题支持发布/订阅
模式，在该模式中，会为注册到主题中的每个订阅
提供每个已发布消息。每个订阅都会以逻辑
方式保留其自己的消息队列。此外，还可以使用
筛选规则配置订阅，这些规则可将传递给订阅队列的消息
集限制为符合筛选条件的消息集。此示例使用
Service Bus 队列。

![][1]

与直接消息传送相比，此通信机制具有多项优势，
即：

-   **暂时分离。**使用异步消息传送模式，
    生产者和使用者不需要在同一时间联机。Service
    Bus 可靠地存储消息，直到使用方准备好
    接收它们。这将允许分布式应用程序的组件
    断开连接，例如，为进行维护而自动断开，
    或因组件故障断开连接，而不会影响系统的
    整体性能。此外，使用方应用程序可能
    只需在一天的特定时段内联机。

-   负载量。在许多应用程序中，系统负载
    随时间而变化，而每个工作单元所需的处理时间
    通常为常量。使用队列在消息创建者与使用者
    之间中继意味着，只需将使用方应用程序（辅助）
    设置为适应平均负载而非最大
    负载。队列深度将随传入负载的变化而加大
    和减小。这将直接根据为应用程序加载提供服务所需的基础结构
    的数目来节省成本。

-   **负载平衡。**随着负载增加，可添加更多的辅助进程
    以从队列中读取。每条消息仅由一个
    辅助进程处理。另外，可通过此基于拉取
    的负载平衡来以最合理的方式使用辅助计算机，即使
    这些辅助计算机具有不同的处理能力（因为它们将以其
    最大速率拉取消息）也是如此。此模式通常称为
    “使用者竞争模式”。

    ![][2]

以下各节讨论了实现此体系结构
的代码。

## <span class="short-header">设置环境</span>设置开发环境

在你可以开始开发 Azure 应用程序之前，你需要
获取相应工具并设置开发环境。

1.  若要安装 Azure SDK for .NET，请单击以下按钮：

    [获取工具和 SDK][获取工具和 SDK]

2.  单击“安装 SDK”。

3.  选择要使用的 Visual Studio 版本的链接。本教程中的步骤使用 Visual Studio 2013：

    ![][3]

4.  当提示你运行或保存安装文件时，
    请单击“运行”：

    ![][4]

5.  在 Web 平台安装程序中，单击“安装”，然后继续安装：

    ![][5]

6.  安装完成后，你便做好了
    开发准备工作。SDK 包含了一些工具，可利用这些工具
    在 Visual Studio 中轻松开发 Azure 应用程序。如果你
    未安装 Visual Studio，它还会安装免费的
    Visual Studio Express for Web。

## <span class="short-header">设置命名空间</span>设置 Service Bus 命名空间

下一步是创建服务命名空间
并获取共享密钥。服务命名空间为通过 Service Bus 公开
的每个应用程序提供一个应用程序边界。
创建服务命名空间时，系统将自动生成共享
密钥。服务命名空间和共享密钥的组合
为 Service Bus 提供了用于验证对应用程序的访问的
凭据。

请注意，你还可以使用 Visual Studio 服务器资源管理器管理命名空间和 Service Bus 消息传送实体，但你只能在门户中创建新的命名空间。

### 使用管理门户设置命名空间

1.  登录到 [Azure 管理门户][Azure 管理门户]。

2.  在该管理门户的左侧导航窗格中，单击
    **Service Bus**。

3.  在该管理门户的下方窗格中，单击“创建”。

    ![][6]

4.  在“添加新命名空间”对话框中，输入命名空间名称。
    系统会立即检查该名称是否可用。
    ![][7]

5.  在确保命名空间名称可用后，选择
    应托管你的命名空间的国家或地区（确保
    使用在其中部署计算资源的同一
    国家/地区）。

    重要说明：选取要部署应用程序的**相同区域**。
    这将为你提供最佳性能。

6.  单击复选标记。系统现在会创建你的服务
    命名空间并启用它。你可能需要等待几分钟，因为
    系统将为你的帐户配置资源。

    ![][8]

7.  在主窗口中，单击你的服务命名空间的名称。

    ![][9]

8.  单击“连接信息”。

    ![][10]

9.  在“访问连接信息”面板中，找到“默认颁发者”和“默认密钥”值。

10. 记下该密钥或将其复制到剪贴板。

### 使用 Visual Studio 服务器资源管理器管理服务命名空间和消息传送实体

若要使用 Visual Studio 而非管理门户来管理命名空间并获取连接信息，请按[此处][此处]所述过程进行操作，详见**从 Visual Studio 连接到 Azure** 这一节。当你登录到 Azure 时，服务器资源管理器中 **Microsoft Azure** 树下的 **Service Bus** 节点中会自动填充你所创建的任何命名空间。右键单击任意命名空间，然后单击“属性”，此时就会看到在 Visual Studio 的“属性”窗格中显示与该命名空间关联的连接字符串和其他元数据。

记下 **SharedAccessKey** 值或将其复制到剪贴板：

![][11]

## <span class="short-header">创建 Web 角色</span>创建 Web 角色

在本节中，你将生成应用程序的前端。你
首先将创建应用程序显示的各种页面。
之后，你会添加代码以将项提交到 Service Bus
队列并显示有关队列的状态信息。

### 创建项目

1.  使用管理员权限启动 Microsoft Visual
    Studio 2013 或 Microsoft Visual Studio Express。若要
    使用管理员权限启动 Visual Studio，请右键单击
    “Microsoft Visual Studio 2013”（或“Microsoft Visual Studio Express”），
    然后单击“以管理员身份运行”。Azure 计算模拟器
    （本指南后面会讨论）要求
    使用管理员权限启动 Visual Studio。

    在 Visual Studio 的“文件”菜单中，单击“新建”，然后
    单击“项目”。

    ![][12]

2.  从 **Visual C#** 下的**“已安装模板”**，单击**“云”**，
    然后单击**“Azure 云服务”**。将项目命名为
    “MultiTierApp”。然后，单击“确定”。

    ![][13]

3.  从 **.NET Framework 4.5** 角色中，双击 **ASP.NET Web
    角色**。

    ![][14]

4.  将鼠标指针停留在“Azure 云服务解决方案”下的“WebRole1”上，单击
    铅笔图标，并将 Web 角色重命名为“FrontendWebRole”。然后单击“确定”。（请确保你输入“Frontend”而不是“FrontEnd”，此处为小写“e”。)

    ![][15]

5.  从**“新建 ASP.NET 项目”**对话框的**“选择模板”**列表中，单击**“MVC”**，
    然后单击**“确定”**。

    ![][16]

6.  在“解决方案资源管理器”中，右键单击“引用”，然后单击
    “管理 NuGet 包...”或“添加库程序包引用”。

7.  在该对话框的左侧选择“联机”。搜索
    “MicrosoftAzure”，
    然后选择“Azure Service Bus”项。然后，完成安装过程并关闭此对话框。

    ![][17]

8.  请注意，现已引用所需的客户端程序集并已添加部分
    新代码文件。

9.  在“解决方案资源管理器”中，右键单击“模型”，单击“添加”，
    然后单击“类”。在“名称”框中，键入名称
    “OnlineOrder.cs”。然后单击“添加”。

### 为你的 Web 角色编写代码

在本节中，你将创建应用程序显示的
各种页面。

1.  在 Visual Studio 的 **OnlineOrder.cs** 文件中将现有
    命名空间定义替换为以下代码：

        namespace FrontendWebRole.Models
        {
            public class OnlineOrder
            {
                public string Customer { get; set; }
                public string Product { get; set; }
            }
        }

2.  在**解决方案资源管理器**中，双击
    **Controllers\\HomeController.cs**。在文件顶部添加以下 **using**
     语句以包括针对你刚创建的模型
    以及 Service Bus 的命名空间：

        using FrontendWebRole.Models;
        using Microsoft.ServiceBus.Messaging;
        using Microsoft.ServiceBus;

3.  仍在 Visual Studio 的 **HomeController.cs** 文件中，
    将现有命名空间定义替换为以下代码。此代码
    包含用于处理将项提交到队列这一任务的方法：

        namespace FrontendWebRole.Controllers
        {
            public class HomeController : Controller
            {
                public ActionResult Index()
                {
                    // Simply redirect to Submit, since Submit will serve as the
                    // front page of this application
                    return RedirectToAction("Submit");
                }

                public ActionResult About()
                {
                    return View();
                }

                // GET: /Home/Submit
                // Controller method for a view you will create for the submission
                // form
                public ActionResult Submit()
                {
                    // Will put code for displaying queue message count here.

                    return View();
                }

                // POST: /Home/Submit
                // Controller method for handling submissions from the submission
                // form 
                [HttpPost]
                // Attribute to help prevent cross-site scripting attacks and 
                // cross-site request forgery  
                [ValidateAntiForgeryToken] 
                public ActionResult Submit(OnlineOrder order)
                {
                    if (ModelState.IsValid)
                    {
                        // Will put code for submitting to queue here.

                        return RedirectToAction("Submit");
                    }
                    else
                    {
                        return View(order);
                    }
                }
            }
        }

4.  从“构建”菜单中，单击“构建解决方案”。

5.  现在，你将为上面创建的 **Submit()** 方法
    创建视图。在 Submit() 方法内右键单击，
    然后选择“添加视图”

    ![][18]

6.  这将显示一个用于创建视图的对话框。在**“模型类”**下拉列表中选择
    **OnlineOrder** 类，然后在**“模板”**下拉列表中选择
    **“创建”**。

    ![][19]

7.  单击“添加”。

8.  现在，你将更改应用程序的显示名称。在
    “解决方案资源管理器”中，双击
    \*\*Views\\Shared\\\_Layout.cshtml\*\* 文件以在 Visual Studio 编辑器中
    将其打开。

9.  将每一处 **My ASP.NET MVC Application** 替换
    为 **LITWARE'S Awesome Products**。

10. 将“your logo here”替换为 **LITWARE'S Awesome Products**：

    ![][20]

11. 删除“Home”、“About”和“Contact”链接。删除突出显示的代码：

    ![][21]

12. 最后，修改提交页以包含有关队列的
    一些信息。在“解决方案资源管理器”中，
    双击 **Views\\Home\\Submit.cshtml** 文件
    以在 Visual Studio 编辑器中将其打开。在 **\<h2\>Submit\</h2\>** 后面添加以下行。现在，
    **ViewBag.MessageCount** 为空。稍后你将填充它。

        <p>Current Number of Orders in Queue Waiting to be Processed: @ViewBag.MessageCount</p>

13. 现在，你已实现你的 UI。你可以按 **F5** 运行应用程序
    并确认其按预期方式运行。

    ![][22]

### 编写用于将项提交到 Service Bus 队列的代码

现在，你将添加用于将项提交到队列的代码。你首先将
创建一个包含 Service Bus 队列连接信息
的类。然后，你将从
**Global.aspx.cs** 初始化你的连接。最后，你将更新你之前
在 **HomeController.cs** 中创建的提交代码
以便实际将项提交到 ServiceBus 队列。

1.  在“解决方案资源管理器”中，右键单击“FrontendWebRole”（右键单击项目而不是角色）。单击“添加”，然后单击“类”。

2.  将类命名为 **QueueConnector.cs**。单击“添加”以创建类。

3.  现在，你将粘贴代码，该代码用于封装连接信息
    和包含用于初始化与 Service Bus 队列的
    连接的方法。在 QueueConnector.cs 中，粘贴以下代码，
    然后为 **Namespace**、**IssuerName** 和 **IssuerKey** 输入值。你可以
    从[管理门户][Azure 管理门户]，或从 **Service Bus** 节点下的 Visual Studio 服务器资源管理器获取这些值。

        using System;
        using System.Collections.Generic;
        using System.Linq;
        using System.Web;
        using Microsoft.ServiceBus.Messaging;
        using Microsoft.ServiceBus;

        namespace FrontendWebRole
        {
            public static class QueueConnector
            {
                // Thread-safe. Recommended that you cache rather than recreating it
                // on every request.
                public static QueueClient OrdersQueueClient;

                // Obtain these values from the Management Portal
                public const string Namespace = "your service bus namespace";
                public const string IssuerName = "issuer name";
                public const string IssuerKey = "issuer key";

                // The name of your queue
                public const string QueueName = "OrdersQueue";

                public static NamespaceManager CreateNamespaceManager()
                {
                    // Create the namespace manager which gives you access to
                    // management operations
                    var uri = ServiceBusEnvironment.CreateServiceUri(
                        "sb", Namespace, String.Empty);
                    var tP = TokenProvider.CreateSharedSecretTokenProvider(
                        IssuerName, IssuerKey);
                    return new NamespaceManager(uri, tP);
                }

                public static void Initialize()
                {
                    // Using Http to be friendly with outbound firewalls
                    ServiceBusEnvironment.SystemConnectivity.Mode = 
                        ConnectivityMode.Http;

                    // Create the namespace manager which gives you access to 
                    // management operations
                    var namespaceManager = CreateNamespaceManager();

                    // Create the queue if it does not exist already
                    if (!namespaceManager.QueueExists(QueueName))
                    {
                        namespaceManager.CreateQueue(QueueName);
                    }

                    // Get a client to the queue
                    var messagingFactory = MessagingFactory.Create(
                        namespaceManager.Address, 
                        namespaceManager.Settings.TokenProvider);
                    OrdersQueueClient = messagingFactory.CreateQueueClient(
                        "OrdersQueue");
                }
            }
        }

4.  现在，你需要确保你的 **Initialize** 方法会被调用。在“解决方案资源管理器”中，双击 **Global.asax\\Global.asax.cs**。

5.  将以下行添加到 **Application\_Start** 方法
    的底部：

        FrontendWebRole.QueueConnector.Initialize();

6.  最后，你将更新之前创建的 Web 代码
    以便将项提交到队列。在**解决方案资源管理器**中，
    双击你此前创建的 **Controllers\\HomeController.cs**
    。

7.  更新 **Submit()** 方法（如下所示）
    获取队列的消息计数：

        public ActionResult Submit()
        {            
            // Get a NamespaceManager which allows you to perform management and
            // diagnostic operations on your Service Bus Queues.
            var namespaceManager = QueueConnector.CreateNamespaceManager();

            // Get the queue, and obtain the message count.
            var queue = namespaceManager.GetQueue(QueueConnector.QueueName);
            ViewBag.MessageCount = queue.MessageCount;

            return View();
        }

8.  更新 **Submit(OnlineOrder order)** 方法（如下所示）
    将订单信息提交到队列：

        public ActionResult Submit(OnlineOrder order)
        {
            if (ModelState.IsValid)
            {
                // Create a message from the order
                var message = new BrokeredMessage(order);

                // Submit the order
                QueueConnector.OrdersQueueClient.Send(message);
                return RedirectToAction("Submit");
            }
            else
            {
                return View(order);
            }
        }

9.  现在，你可以重新运行你的应用程序。每当你提交订单时，
    消息计数都会增大。

    ![][23]

## <span class="short-header">配置管理器</span>云配置管理器

Azure 支持一组托管 API，提供了跨 Microsoft 云服务创建 Azure 服务客户端（例如 Service Bus）新实例的一致方法。这些 API 使你能够实例化这些客户端（例如**CloudBlobClient**、**QueueClient**、**TopicClient**），而无论应用程序托管在何处 --在本地、在 Microsoft 云服务中、在网站中或是在永久性虚拟机角色中。你还可以使用这些 API 检索实例化这些客户端所需的配置信息，以及更改配置，而无需重新部署调用应用程序。这些 API 位于 [Microsoft.MicrosoftAzure.Configuration.CloudConfigurationManager][Microsoft.MicrosoftAzure.Configuration.CloudConfigurationManager] 类中。还有一些 API 位于客户端。

### 连接字符串

若要实例化客户端（例如 Service Bus **QueueClient**），可以将配置信息表示为连接字符串。在客户端，有一个通过使用该连接字符串实例化客户端类型的 **CreateFromConnectionString()** 方法。例如，考虑下面的配置部分：

    <ConfigurationSettings>
    ...
        <Setting name="Microsoft.ServiceBus.ConnectionString" value="Endpoint=sb://[yourServiceNamespace].servicebus.windows.net/;SharedSecretIssuer=[issuerName];SharedSecretValue=[yourDefaultKey]" />
    </ConfigurationSettings>

以下代码检索连接字符串，创建队列并初始化与队列的连接：

    QueueClient Client; 

    string connectionString =
     CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

    var namespaceManager =
     NamespaceManager.CreateFromConnectionString(connectionString); 

    if (!namespaceManager.QueueExists(QueueName))
    {
        namespaceManager.CreateQueue(QueueName);
    }

    // Initialize the connection to Service Bus Queue
    Client = QueueClient.CreateFromConnectionString(connectionString, QueueName);

以下部分中的代码使用这些配置管理 API。

## <span class="short-header">创建辅助角色</span>创建辅助角色

现在，你将创建用于处理订单提交
的辅助角色。此示例使用“Service Bus 队列的辅助角色”Visual Studio 项目模板。首先，你将使用 Visual Studio 中的“服务器资源管理器”获取所需凭据。

1.  如果你已将 Visual Studio 连接到你的 Azure 帐户，如**“使用 Visual Studio 服务器资源管理器设置命名空间”**部分所述，则请跳到步骤 5。

2.  从 Visual Studio 的菜单栏中，选择“视图”，然后单击“服务器资源管理器”。“Service Bus”节点将显示在 **Azure** 下的“服务器资源管理器”层次结构中，如下图所示。

    ![][24]

3.  在“服务器资源管理器”中，展开 **Azure**，右键单击“Service Bus”，然后单击“添加新连接”。

4.  在“添加连接”对话框中，键入服务命名空间的名称、颁发者名称和颁发者密钥。然后单击“确定”进行连接。

    ![][25]

5.  在 Visual Studio 的“解决方案资源管理器”中，
    右键单击“MultiTierApp”项目下的“角色”文件夹。

6.  单击“添加”，然后单击“新建辅助角色项目”。这将显示“添加新角色项目”对话框。

    ![][26]

7.  在“添加新角色项目”对话框中，单击“Service Bus 队列的辅助角色”，如下图所示：

    ![][27]

8.  在“名称”框中，将项目命名为 **OrderProcessingRole**。然后单击“添加”。

9.  在“服务器资源管理器”中，右键单击服务命名空间的名称，然后单击“属性”。在 Visual Studio 的“属性”窗格中，第一个条目包含使用包含所需授权凭据的服务命名空间终结点填充的连接字符串。有关示例，请参见下图。双击“ConnectionString”，然后按 **Ctrl+C** 将此字符串复制到剪贴板中。

    ![][28]

10. 在“解决方案资源管理器”中，右键单击你在步骤 7 中创建的“OrderProcessingRole”（确保右键单击“角色”下的“OrderProcessingRole”而不是类）。然后单击“属性”。

11. 在“属性”对话框的“设置”选项卡中，在“Microsoft.ServiceBus.ConnectionString”的“值”框内单击，然后粘贴你在步骤 8 中复制的终结点值。

    ![][29]

12. 当你从队列中处理订单时，创建一个 **OnlineOrder** 类来表示这些订单。你可以重用已创建的类。在“解决方案资源管理器”中，右键单击“OrderProcessingRole”项目（右键单击项目而不是角色）。单击“添加”，然后单击“现有项”。

13. 浏览到 **FrontendWebRole\\Models** 的子文件夹，然后双击“OnlineOrder.cs”以将其添加到此项目中。

14. 在 WorkerRole.cs 中，将 **WorkerRole.cs** 中 **QueueName** 变量的值 `"ProcessingQueue"` 替换为 `"OrdersQueue"`，如以下代码所示：

        // The name of your queue
        const string QueueName = "OrdersQueue";

15. 在 WorkerRole.cs 文件顶部添加以下 using 语句：

        using FrontendWebRole.Models;

16. 在 `Run()` 函数的 `OnMessage` 调用中，添加以下代码到 `try` 子句：

        Trace.WriteLine("Processing", receivedMessage.SequenceNumber.ToString());
        // View the message as an OnlineOrder
        OnlineOrder order = receivedMessage.GetBody<OnlineOrder>();
        Trace.WriteLine(order.Customer + ": " + order.Product, "ProcessingMessage");
        receivedMessage.Complete();

17. 你已完成此应用程序。可通过按 F5 来 测试
    完整的应用程序，就像你之前所做的那样。请注意，消息计数不会递增，因为辅助角色会处理队列中的项并将其标记为完成。你可以通过查看 Azure 计算
    模拟器 UI 来查看辅助角色的跟踪输出。可通过右击任务栏的通知区域中的模拟器图标并选择
    “显示计算模拟器 UI”
    来执行此操作。

    ![][30]

    ![][31]

## <a name="nextsteps"></a><span class="short-header">后续步骤</span>后续步骤

若要了解有关 Service Bus 的详细信息，请参阅以下资源：

-   [Azure Service Bus][Azure Service Bus]
-   [Service Bus 操作方法][Service Bus 操作方法]
-   [如何使用 Service Bus 队列][如何使用 Service Bus 队列]

若要了解有关多层方案的详细信息，或者要了解如何将应用程序部署到云服务，请参阅：

-   [使用存储表、队列和 Blob 的 .NET 多层应用程序][使用存储表、队列和 Blob 的 .NET 多层应用程序]

你可能需要在 Azure 网站而不是 Azure 云服务中实现多层应用程序的前端。若要详细了解网站和云服务之间的差异，请参阅 [Azure 执行模型][Azure 执行模型]。

若要实施在本教程中以标准 Web 项目而不是云服务 Web 角色方式创建的应用程序，请遵循本教程中的步骤，但需注意以下差异：

1.  创建项目时，请选择 **Web** 类别中的**“ASP.NET MVC 4 Web 应用程序”**项目模板，而不是**“云”**类别中的**“云服务”**模板。然后，请遵循创建 MVC 应用程序时遵循的相同指导，直到你转到**“云配置管理器”**部分。

2.  创建辅助角色时，请在新的独立解决方案中创建它，采用的说明与创建 Web 角色所用的原始说明类似。不过现在，你只是在云服务项目中创建辅助角色。然后，请遵循创建辅助角色所用的相同说明。

3.  你可以分别测试前端和后端，也可以在单独的 Visual Studio 实例中同时运行这二者。

若要了解如何将前端部署到 Azure 网站，请参阅[将 ASP.NET Web 应用程序部署到 Azure 网站][将 ASP.NET Web 应用程序部署到 Azure 网站]。若要了解如何将后端部署到 Azure 云服务，请参阅[使用存储表、队列和 Blob 的 .NET 多层应用程序][使用存储表、队列和 Blob 的 .NET 多层应用程序]。

  [后续步骤]: #nextsteps
  [0]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-01.png
  [Azure 队列和 Azure Service Bus 队列 - 比较与对照]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh767287.aspx
  [1]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-100.png
  [2]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-101.png
  [获取工具和 SDK]: http://go.microsoft.com/fwlink/?LinkId=271920
  [3]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-41.png
  [4]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-3.png
  [5]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-4-2-WebPI.png
  [Azure 管理门户]: http://manage.windowsazure.cn
  [6]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/sb-queues-03.png
  [7]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/sb-queues-04.png
  [8]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-27.png
  [9]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/sb-queues-09.png
  [10]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/sb-queues-06.png
  [此处]: http://http://msdn.microsoft.com/zh-cn/library/windowsazure/ff687127.aspx
  [11]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/VSProperties.png
  [12]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-09.png
  [13]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-10.jpg
  [14]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-11.png
  [15]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-02.png
  [16]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-12.png
  [17]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-13.png
  [18]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-33.png
  [19]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-34.png
  [20]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-35.png
  [21]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-40.png
  [22]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-36.png
  [23]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-37.png
  [24]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/SBExplorer.png
  [25]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/SBExplorerAddConnect.png
  [26]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/SBNewWorkerRole.png
  [27]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/SBWorkerRole1.png
  [28]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/SBExplorerProperties.png
  [29]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/SBWorkerRoleProperties.png
  [30]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-38.png
  [31]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-39.png
  [Azure Service Bus]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee732537.aspx
  [Service Bus 操作方法]: /zh-cn/documentation/services/service-bus/
  [如何使用 Service Bus 队列]: /zh-cn/develop/net/how-to-guides/service-bus-queues/
  [使用存储表、队列和 Blob 的 .NET 多层应用程序]: /zh-cn/develop/net/tutorials/multi-tier-web-site/1-overview/
  [Azure 执行模型]: http://www.windowsazure.com/zh-cn/develop/net/fundamentals/compute/
  [将 ASP.NET Web 应用程序部署到 Azure 网站]: http://www.windowsazure.com/zh-cn/develop/net/tutorials/get-started/
