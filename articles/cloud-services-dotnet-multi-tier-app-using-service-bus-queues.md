<properties linkid="dev-net-e2e-multi-tier" urlDisplayName="Multi-Tier Application" pageTitle=".NET 多层应用程序 - Azure 教程" metaKeywords="Azure Service Bus queue tutorial, Azure queue tutorial, Azure worker role tutorial, Azure .NET queue tutorial, Azure C# queue tutorial, Azure C# worker role tutorial" description="本教程可帮助你在 Azure 中开发使用 Service Bus 队列在各层之间进行通信的多层应用程序。相关示例使用 .NET。" metaCanonical="" services="cloud-services,service-bus" documentationCenter=".NET" title=".NET Multi-Tier Application Using Service Bus Queues" authors="sethm" solutions="" manager="dwrede" editor="mattshel" />





# 使用 Service Bus 队列创建 .NET 多层应用程序

使用 Visual Studio 2013 和免费的 Azure SDK for .NET，可以轻松针对 Azure 进行开发。如果你还没有 Visual Studio 2013，则此 SDK 将自动安装 Visual Studio Express 2013，以便你能完全免费地开始针对 Azure 进行开发。本指南假定你之前未使用过 Microsoft Azure。完成本指南之后，你将拥有使用多项 Azure 资源的应用程序，该应用程序在本地环境中运行并演示多层应用程序的工作方式。

你将了解到以下内容：

-   如何通过单个下载和安装来使你的计算机能够进行 Azure 开发。
-   如何使用 Visual Studio 针对 Azure 进行开发。
-   如何使用 Web 角色和辅助角色在 Azure 中创建多层应用程序。
-   如何使用 Service Bus 队列在各层之间进行通信。

[WACOM.INCLUDE [create-account-note](../includes/create-account-note.md)]

在本教程中，你将生成多层应用程序并在 Azure 云服务中运行它。前端将为 ASP.NET MVC Web 角色，后端将为辅助角色。你可以创建与前端相同的多层应用程序，作为将部署到 Azure 网站而不是云服务的 Web 项目。有关如何以不同方式处理 Azure 网站前端的说明，请参阅[后续步骤]。(#nextsteps) 节中添加以下代码。

以下是已完成应用程序的屏幕快照：

![][0]

**注意：**Azure 还提供了存储队列功能。有关 Azure 存储队列和 Service Bus 队列的详细信息，请参阅 [Azure 队列和 Azure Service Bus 队列 - 比较与对照][sbqueuecomparison]。  

##方案概述：角色间的通信##

若要提交处理命令，以 Web 角色运行的前端 UI 组件需要与以辅助角色运行的中间层逻辑进行交互。此示例使用 Service Bus 中转消息传送在各层之间进行通信。

在 Web 层和中间层之间使用代理消息传送将分离这两个组件。与直接消息传送（即 TCP 或 HTTP）相反，Web 层不会直接连接到中间层，而是将工作单元作为消息推送到 Service Bus，Service Bus 将以可靠方式保留这些工作单元，直到中间层准备好使用和处理它们。

Service Bus 提供了两个实体以支持中转消息传送：队列和主题。通过队列，发送到队列的每个消息均由一个接收方使用。主题支持发布/订阅模式，在该模式中，会为注册到主题中的每个订阅提供每个已发布消息。每个订阅都会以逻辑方式保留其自己的消息队列。此外，还可以使用筛选规则配置订阅，这些规则可将传递给订阅队列的消息集限制为符合筛选条件的消息集。此示例使用 Service Bus 队列。

![][1]

与直接消息传送相比，此通信机制具有多项优势，即：

-   **暂时分离。**使用异步消息传送模式，生产者和使用者不必同时处于联机状态。Service Bus 会可靠地存储这些消息,直至使用方可以接收消息。这允许分布式应用程序的组件断开连接。这一断开连接可以是自愿的，例如进行维护时；或者是由于某个组件损坏，但不会影响整个系统。此外，消费应用程序可能只需在一天的某些时间联机。

-   **负载分级**。在许多应用程序中，系统负载随时间而变化，而每个工作单元所需的处理时间通常为常量。使用队列在消息创建者与使用者之间中继意味着，只需将使用方应用程序（辅助）设置为适应平均负载而非最大负载。队列深度将随传入负载的变化而加大和减小。这将直接根据为应用程序加载提供服务所需的基础结构的数目来节省成本。

-   **负载平衡。**随着负载增加，可以添加更多工作进程，以便从工作队列中读取。每条消息仅由其中的一个工作进程处理。此外，这种基于拉取的负载平衡还可以让你以最佳方式使用工作计算机，即使工作计算机的处理能力各不相同，因为工作计算机会以自己的最大速率拉取消息。此模式通常称为"使用者竞争"模式。

    ![][2]

以下各节讨论了实现此体系结构的代码。

##设置开发环境##

在可以开始开发 Azure 应用程序之前，需要下载一些工具并设置开发环境：

1.  若要安装 Azure SDK for .NET，请单击以下按钮：

    [获取工具和 SDK][]

2. 	单击要使用的 Visual Studio 版本的链接。本教程中的步骤使用 Visual Studio 2013：

	![][32]

3. 	当提示你运行或保存安装文件时，请单击"运行"****：

    ![][3]

4.  在 Web 平台安装程序中，单击"安装"****，然后继续安装：

    ![][33]

5.  安装完成后，你便做好了开发准备工作。SDK 包含了一些工具，可利用这些工具在 Visual Studio 中开发 Azure 应用程序。如果你未安装 Visual Studio，它还会安装免费的Visual Studio Express for Web。

##设置 Service Bus 命名空间##

下一步是创建服务命名空间并获取共享访问签名 (SAS) 密钥。服务命名空间为通过 Service Bus 公开的每个应用程序提供一个应用程序边界。创建服务命名空间时，系统将生成 SAS 密钥。服务命名空间和 SAS 密钥的组合为 Service Bus 提供了用于验证对应用程序的访问的凭据。

请注意，你还可以使用 Visual Studio 服务器资源管理器管理命名空间和 Service Bus 消息传送实体，但你只能在门户中创建新的命名空间。 

###使用管理门户设置命名空间

1.  登录到 [Azure 管理门户][]。

2.  在该管理门户的左侧导航窗格中，单击"Service Bus"****。

3.  在管理门户的下方窗格中，单击"创建"****。

    ![][6]

4.  在"添加新命名空间"对话框中，输入命名空间名称****。系统会立即检查该名称是否可用。   
    ![][7]

5.  在确保命名空间名称可用后，选择应承载你的命名空间的国家或地区（确保使用在其中部署计算资源的同一国家/地区）。此外，请确保在命名空间"类型"字段中选择"消息传递"，在"消息层"字段中选择"标准"****************。 

    重要说明：选取要部署应用程序的**相同区域**。这将为你提供最佳性能。

6.  单击复选标记。系统现已创建您的服务命名空间并已将其启用。您可能需要等待几分钟，因为系统将为您的帐户配置资源。

	![][27]

7.  在主窗口中，单击你的服务命名空间的名称。

	![][30]

8. 单击"连接信息"****。

	![][31]

9.  在"访问连接信息"****面板中，找到"默认颁发者"****和"默认密钥"****值。

10.  记下该密钥或将其复制到剪贴板。

###使用 Visual Studio 服务器资源管理器管理服务命名空间和消息传送实体

若要使用 Visual Studio 而非管理门户来管理命名空间并获取连接信息，请按[此处](http://http://msdn.microsoft.com/zh-cn/library/windowsazure/ff687127.aspx)所述过程进行操作，详见**从 Visual Studio 连接到 Azure** 部分。当你登录到 Azure 时，服务器资源管理器中 **Microsoft Azure** 树下的 **Service Bus** 节点中会自动填充你所创建的任何命名空间。右键单击任意命名空间，然后单击"属性"****，此时就会看到在 Visual Studio 的"属性"****窗格中显示与该命名空间关联的连接字符串和其他元数据。 

记下 **SharedAccessKey** 值或将其复制到剪贴板：

![][34]

##创建 Web 角色##

在本部分中，你将生成应用程序的前端。你首先将创建应用程序显示的各种页面。之后，你将添加代码，这些代码用于将项提交到 Service Bus 队列并显示有关队列的状态信息。

### 创建项目

1.  使用管理员权限启动 Microsoft VisualStudio 2013 或 Microsoft Visual Studio Express。若要使用管理员权限启动 Visual Studio，请右键单击"Microsoft Visual Studio 2013"（或"Microsoft Visual Studio Express"），然后单击"以管理员身份运行"。Azure 计算模拟器（本指南后面会讨论）要求使用管理员权限启动 Visual Studio。

    在 Visual Studio 的"文件"****菜单中，单击"新建"****，然后单击"项目"****。

    ![][8]


2.  从"已安装的模板"****的"Visual C#"****下单击"云"****，然后
    单击"Azure 云服务"****。将项目命名为 **MultiTierApp**。然后，单击"确定"****。

    ![][9]

3.  从".NET Framework 4.5"角色中，双击"ASP.NET Web 角色"********。

    ![][10]

4.  将鼠标指针停留在"Azure 云服务解决方案"************下的"WebRole1"上，单击铅笔图标，并将 Web 角色重命名为"FrontendWebRole"。然后，单击"确定"****。（请确保你输入"Frontend"而不是"FrontEnd"，此处为小写"e"。)

    ![][11]

5.  从"新建 ASP.NET 项目"****对话框的"选择模板"****列表中，单击"MVC"****，然后单击"确定"****。

    ![][12]

6.  在"解决方案资源管理器"中，右键单击"引用"，然后单击"管理 NuGet 包..."或"添加库程序包引用"****************。

7.  在该对话框的左侧选择"联机"****。搜索"Service Bus"****，然后选择"Microsoft Azure Service**
    **Bus"项。然后，完成安装过程并关闭此对话框。

    ![][13]

8.  请注意，现已引用所需的客户端程序集并已添加部分新代码文件。

9.  在"解决方案资源管理器"中，右键单击"模型"，单击"添加"，然后单击"类"****************。在"名称"框中，键入名称 **OnlineOrder.cs**。然后单击"添加"****。

### 为你的 Web 角色编写代码

在本部分中，你将创建应用程序显示的各种页面。

1.  在 Visual Studio 的 **OnlineOrder.cs** 文件中将现有命名空间定义替换为以下代码：

        namespace FrontendWebRole.Models
        {
            public class OnlineOrder
            {
                public string Customer { get; set; }
                public string Product { get; set; }
            }
        }

2.  在"解决方案资源管理器"中，双击"Controllers\HomeController.cs"********。在文件顶部添加以下 **using** 语句以包括针对你刚创建的模型以及 Service Bus 的命名空间：

        using FrontendWebRole.Models;
        using Microsoft.ServiceBus.Messaging;
        using Microsoft.ServiceBus;

3.  此外，在 Visual Studio 的 **HomeController.cs** 文件中将现有命名空间定义替换为以下代码。此代码包含用于处理将项提交到队列这一任务的方法：

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

4.  从"生成"菜单中，单击"生成解决方案"********。

5.  现在，你将为上面创建的 **Submit()** 方法创建视图。在 Submit() 方法内右键单击，然后选择"添加视图"****

    ![][14]

6.  这将显示一个用于创建视图的对话框。在"模型类"下拉列表中选择 **OnlineOrder** 类，****然后在"模板"****下拉列表中选择"创建"****。

    ![][15]

7.  单击"添加"****。

8.  现在，你将更改应用程序的显示名称。在"解决方案资源管理器"****中，双击 **Views\Shared\\_Layout.cshtml** 文件以在 Visual Studio 编辑器中将其打开。

9.  将每一处 **My ASP.NET MVC Application** 替换为 **LITWARE'S Awesome Products**。

11. 删除"Home"、"About"和"Contact"链接************。删除突出显示的代码：

	![][28]
  

12. 最后，修改提交页以包含有关队列的一些信息。在"解决方案资源管理器"****中，双击 **Views\Home\Submit.cshtml** 文件以在 Visual Studio 编辑器中将其打开。在 **&lt;h2>Submit&lt;/h2>**. 的后面添加以下行。**ViewBag.MessageCount** 目前是空的。稍后你将填充它。

        <p>队列中当前等待处理的订单数：@ViewBag.MessageCount</p>
             

13. 现在，你已实现你的 UI。你可以按 **F5** 运行应用程序并确认其按预期方式运行。

    ![][17]

### 编写用于将项提交到 Service Bus 队列的代码

现在，你将添加用于将项提交到队列的代码。你首先将创建一个包含 Service Bus 队列连接信息的类。然后，你将从 **Global.aspx.cs** 初始化你的连接。最后，你将更新你之前在 **HomeController.cs** 中创建的提交代码以便实际将项提交到 Service Bus 队列。

1.  在"解决方案资源管理器"中，右键单击 **FrontendWebRole**（右键单击项目而不是角色）。单击"添加"****，然后单击"类"****。

2.  将类命名为 **QueueConnector.cs**。单击"添加"****以创建类。

3.  现在，你将粘贴代码，该代码用于封装连接信息和包含用于初始化与 Service Bus 队列的连接的方法。在 QueueConnector.cs 中，粘贴以下代码，然后为 **Namespace**、**IssuerName** 和 **IssuerKey** 输入值。你可以从[管理门户][Azure Management Portal]，或从 **Service Bus** 节点下的 Visual Studio 服务器资源管理器获取这些值。

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

4.  现在，你需要确保你的 **Initialize** 方法会被调用。在"解决方案资源管理器"****中，双击 **Global.asax\Global.asax.cs**。

5.  将以下行添加到 **Application_Start** 方法的底部：

        FrontendWebRole.QueueConnector.Initialize();

6.  最后，你将更新之前创建的 Web 代码以便将项提交到队列。在"解决方案资源管理器"中，****双击你此前创建的 **Controllers\HomeController.cs**。

7.  更新 **Submit()** 方法（如下所示）获取队列的消息计数：

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

8.  更新 **Submit(OnlineOrder order)** 方法（如下所示）将订单信息提交到队列：

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

9.  现在，你可以重新运行你的应用程序。每当你提交订单时，消息计数都会增大。

    ![][18]

##云配置管理器##

Azure 支持一组托管 API，提供了跨 Microsoft 云服务创建 Azure 服务客户端（例如 Service Bus）新实例的一致方法。这些 API 使你能够实例化这些客户端（例如 **CloudBlobClient**、**QueueClient**、**TopicClient**），而无论应用程序托管在何处 - 本地、在 Microsoft 云服务中、在网站中或是在永久性虚拟机角色中。你还可以使用这些 API 检索实例化这些客户端所需的配置信息，以及更改配置，而无需重新部署调用应用程序。这些 API 位于 [Microsoft.WindowsAzure.Configuration.CloudConfigurationManager][] 类中。还有一些 API 位于客户端。

### 连接字符串

若要实例化客户端（例如 Service Bus **QueueClient**），可以将配置信息表示为连接字符串。在客户端，有一个通过使用该连接字符串实例化客户端类型的 **CreateFromConnectionString()** 方法。例如，考虑下面的配置部分：

	<ConfigurationSettings>
    ...
    	<Setting name="Microsoft.ServiceBus.ConnectionString" value="Endpoint=sb://[yourServiceNamespace].servicebus.chinacloudapi.cn/;SharedSecretIssuer=[issuerName];SharedSecretValue=[yourDefaultKey]" />
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

##创建辅助角色##

现在，你将创建用于处理订单提交的辅助角色。此示例使用"Service Bus 队列的辅助角色"****Visual Studio 项目模板。首先，你将使用 Visual Studio 中的"服务器资源管理器"获取所需凭据。

1. 如果你已将 Visual Studio 连接到你的 Azure 帐户，如**使用 Visual Studio 服务器资源管理器设置命名空间**部分中所述，则请跳到步骤 5。 

2. 从 Visual Studio 的菜单栏中，选择"视图"****，然后单击"服务器资源管理器"****。"Service Bus"****节点将显示在"Azure"****下的"服务器资源管理器"层次结构中，如下图所示。

	![][21]

3. 在"服务器资源管理器"中，展开 Azure，右键单击"Service Bus"****，然后单击"添加新连接"********。

4. 在"添加连接"****对话框中，键入服务命名空间的名称、颁发者名称和颁发者密钥。然后单击"确定"****进行连接。

	![][22]

5.  在 Visual Studio 的"解决方案资源管理器"****中，右键单击
    "MultiTierApp"项目下的****"角色"文件夹****。

6.  单击"添加"****，然后单击"新建辅助角色项目"****。这将显示"添加新角色项目"****对话框。

	![][26]

7.  在"添加新角色项目"****对话框中，单击"Service Bus 队列的辅助角色"****，如下图所示：

	![][23]

8.  在"名称"****框中，将项目命名为 **OrderProcessingRole**。然后单击"添加"****。

9.  在"服务器资源管理器"中，右键单击服务命名空间的名称，然后单击"属性"****。在 Visual Studio 的"属性"****窗格中，第一个条目包含使用包含所需授权凭据的服务命名空间终结点填充的连接字符串。有关示例，请参见下图。双击"ConnectionString"****，然后按 **Ctrl+C** 将此字符串复制到剪贴板中。

	![][24]

10.  在"解决方案资源管理器"中，右键单击你在步骤 7 中创建的"OrderProcessingRole"****（确保右键单击"角色"****下的"OrderProcessingRole"****而不是类）。然后单击"属性"****。

11.  在"属性"对话框的"设置"选项卡中，在"Microsoft.ServiceBus.ConnectionString"的"值"框内单击，然后粘贴你在步骤 8 中复制的终结点值****************。

	![][25]

12.  当你从队列中处理订单时，创建一个 **OnlineOrder** 类来表示这些订单。你可以重用已创建的类。在"解决方案资源管理器"中，右键单击"OrderProcessingRole"****项目（右键单击项目而不是角色）。单击"添加"****，然后单击"现有项"****。

13. 浏览到 **FrontendWebRole\Models** 的子文件夹，然后双击 **OnlineOrder.cs** 以将其添加到此项目中。

14. 在 WorkerRole.cs 中，将 **WorkerRole.cs** 中 **QueueName** 变量的值"ProcessingQueue"替换为"OrdersQueue"如以下代码所示：

		// The name of your queue
		const string QueueName = "OrdersQueue";

15. 在 WorkerRole.cs 文件顶部添加以下 using 语句：

		using FrontendWebRole.Models;

16. 在"Run()"函数中的"OnMessage"调用内，在"try"子句中添加以下代码：

		Trace.WriteLine("Processing", receivedMessage.SequenceNumber.ToString());
		// View the message as an OnlineOrder
		OnlineOrder order = receivedMessage.GetBody<OnlineOrder>();
		Trace.WriteLine(order.Customer + ": " + order.Product, "ProcessingMessage");
		receivedMessage.Complete();

17.  你已完成此应用程序。可通过按 F5 来测试完整的应用程序，就像你之前所做的那样。请注意，消息计数不会递增，因为辅助角色会处理队列中的项并将其标记为完成。你可以通过查看 Azure 计算模拟器 UI 来查看辅助角色的跟踪输出。可通过右击任务栏的通知区域中的模拟器图标并选择"显示计算模拟器 UI"****来执行此操作。

    ![][19]

    ![][20]

##<a name="nextsteps"></a>后续步骤##  

若要了解有关 Service Bus 的详细信息，请参阅以下资源：  
  
* [Azure Service Bus][sbmsdn]
* [Service Bus 操作方法][sbwacom]
* [如何使用 Service Bus 队列][sbwacomqhowto]

若要了解有关多层方案的详细信息，或者要了解如何将应用程序部署到云服务，请参阅：  

* [使用存储表、队列和 Blob 的 .NET 多层应用程序][mutitierstorage]

你可能需要在 Azure 网站而不是 Azure 云服务中实现多层应用程序的前端。若要详细了解网站和云服务之间的差异，请参阅 [Azure 执行模型][executionmodels]。

若要实施在本教程中以标准 Web 项目而不是云服务 Web 角色方式创建的应用程序，请遵循本教程中的步骤，但需注意以下差异：

1. 创建项目时，请选择"Web"类别中的"ASP.NET MVC 4 Web 应用程序"项目模板，而不是"云"类别中的"云服务"模板****************。然后，请遵循创建 MVC 应用程序时遵循的相同指导，直到你转到**云配置管理器**部分。

2. 创建辅助角色时，请在新的独立解决方案中创建它，采用的说明与创建 Web 角色所用的原始说明类似。不过现在，你只是在云服务项目中创建辅助角色。然后，请遵循创建辅助角色所用的相同说明。

3. 你可以分别测试前端和后端，也可以在单独的 Visual Studio 实例中同时运行这二者。

若要了解如何将前端部署到 Azure 网站，请参阅[将 ASP.NET Web 应用程序部署到 Azure 网站](/zh-cn/documentation/articles/web-sites-dotnet-get-started/).若要了解如何将后端部署到 Azure 云服务，请参阅[使用存储表、队列和 Blob 的 .NET 多层应用程序][mutitierstorage]。


  [0]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-01.png
  [1]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-100.png
  [sbqueuecomparison]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh767287.aspx
  [2]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-101.png
  [获取工具和 SDK]: /develop/net/
  [3]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-3.png
  
  
  
  [Azure 管理门户]: http://manage.windowsazure.cn
  [6]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/sb-queues-03.png
  [7]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/sb-queues-04.png
  [8]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-09.png
  [9]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-10.png
  [10]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-11.png
  [11]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-02.png
  [12]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-12.png
  [13]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-13.png
  [14]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-33.png
  [15]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-34.png
  [16]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-35.png
  [17]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-36.png
  [18]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-37.png
  
  [19]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-38.png
  [20]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-39.png
  [21]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/SBExplorer.png
  [22]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/SBExplorerAddConnect.png
  [23]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/SBWorkerRole1.png
  [24]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/SBExplorerProperties.png
  [25]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/SBWorkerRoleProperties.png
  [26]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/SBNewWorkerRole.png
  [27]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-27.png
  [28]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-40.png
  [30]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/sb-queues-09.png
  [31]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/sb-queues-06.png
  [32]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-41.png
  [33]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/getting-started-4-2-WebPI.png
  [34]: ./media/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/VSProperties.png
  [sbmsdn]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee732537.aspx  
  [sbwacom]: /zh-cn/documentation/services/service-bus/  
  [sbwacomqhowto]: /zh-cn/documentation/articles/service-bus-dotnet-how-to-use-queues/  
  [mutitierstorage]: https://code.msdn.microsoft.com/Windows-Azure-Multi-Tier-eadceb36 
  [executionmodels]: /zh-cn/documentation/articles/fundamentals-application-models/
<!--HONumber=39-->
