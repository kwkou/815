<properties
	pageTitle=".NET 多层应用程序 | Azure"
	description="本 .NET 教程可帮助你在 Azure 中开发使用服务总线队列在各层之间进行通信的多层应用。"
	services="service-bus"
	documentationCenter=".net"
	authors="sethmanheim"
	manager="timlt"
	editor=""/>

<tags
	ms.service="service-bus"
	ms.workload="tbd"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="get-started-article"
	ms.date="01/10/2017"
	ms.author="sethm"
	wacn.date="02/20/2017"/>  


# 使用 Azure 服务总线队列创建 .NET 多层应用程序

## 介绍

使用 Visual Studio 和免费的用于 .NET 的 Azure SDK，可以轻松针对 Azure 进行开发。本教程将指导你完成创建使用在本地环境中运行的多个 Azure 资源的应用程序的步骤。这些步骤假设你之前未使用过 Azure。

你将学习以下技能：

-   如何通过单个下载和安装来使你的计算机能够进行 Azure 开发。
-   如何使用 Visual Studio 针对 Azure 进行开发。
-   如何使用 Web 角色和辅助角色在 Azure 中创建多层应用程序。
-   如何使用服务总线队列在各层之间进行通信。

[AZURE.INCLUDE [create-account-note](../../includes/create-account-note.md)]

在本教程中，你将生成多层应用程序并在 Azure 云服务中运行它。前端是 ASP.NET MVC Web 角色，后端是使用服务总线队列的辅助角色。可创建与前端相同的多层应用程序，作为将部署到 Azure 网站而不是云服务的 Web 项目。有关如何以不同方式处理 Azure 网站前端的说明，请参阅[后续步骤](#nextsteps)部分。你还可以试用 [.NET 本地/云混合应用程序](/documentation/articles/service-bus-dotnet-hybrid-app-using-service-bus-relay/)教程。

以下屏幕截图显示了已完成的应用程序。

![][0]

## 方案概述：角色间通信

若要提交处理命令，以 Web 角色运行的前端 UI 组件必须与以辅助角色运行的中间层逻辑进行交互。此示例使用服务总线中转消息传送在各层之间进行通信。

在 Web 层和中间层之间使用中转消息传送将分离这两个组件。与直接消息传送（即 TCP 或 HTTP）不同，Web 层不会直接连接到中间层，而是将工作单元作为消息推送到服务总线，服务总线将以可靠方式保留这些工作单元，直到中间层准备好使用和处理它们。

服务总线提供了两个实体以支持中转消息传送：队列和主题。通过队列，发送到队列的每个消息均由一个接收方使用。主题支持发布/订阅模式，在该模式下，会为注册到主题中的订阅提供每个已发布消息。每个订阅都会以逻辑方式保留其自己的消息队列。此外，还可以使用筛选规则配置订阅，这些规则可将传递给订阅队列的消息集限制为符合筛选条件的消息集。以下示例使用服务总线队列。

![][1]

与直接消息传送相比，此通信机制具有多项优势：

-   **暂时分离。** 使用异步消息传送模式，生产者和使用者不需要在同一时间联机。服务总线可靠地存储消息，直到使用方准备好接收它们。这将允许分布式应用程序的组件断开连接，例如，为进行维护而自动断开，或因组件故障断开连接，而不会影响系统的整体性能。此外，使用方应用程序可能只需在一天的特定时段内联机。

-   **负载量。** 在许多应用程序中，系统负载随时间而变化，而每个工作单元所需的处理时间通常为常量。使用队列在消息创建者与使用者之间中继意味着，只需将使用方应用程序（辅助）预配为适应平均负载而非最大负载。队列深度将随传入负载的变化而加大和减小。这将直接根据为应用程序加载提供服务所需的基础结构的数目来节省成本。

-   **负载均衡。** 随着负载增加，可添加更多的工作进程以从队列中读取。每条消息仅由一个辅助进程处理。另外，可通过此基于拉取的负载均衡来以最合理的方式使用辅助计算机，即使这些辅助计算机具有不同的处理能力（因为它们将以其最大速率拉取消息）也是如此。此模式通常称为*使用者竞争*模式。

    ![][2]

以下各节讨论了实现此体系结构的代码。

## 设置开发环境

在开始开发 Azure 应用程序之前，获取工具并设置开发环境。

1.  通过[获取工具和 SDK][] 安装用于 .NET 的 Azure SDK。

2. 在 **.NET** 列中，单击要使用的 Visual Studio 的版本。本教程中的步骤使用 Visual Studio 2015。

4.  当提示你是要运行还是保存安装程序时，单击“运行”。

5.  在“Web 平台安装程序”中，单击“安装”，然后继续安装。

6.  安装完成后，你就有了开始开发应用所需的一切。SDK 包含了一些工具，可利用这些工具在 Visual Studio 中轻松开发 Azure 应用程序。如果你未安装 Visual Studio，SDK 还会安装免费的 Visual Studio Express。

## 创建命名空间

下一步是创建服务命名空间并获取共享访问签名 (SAS) 密钥。命名空间为每个通过服务总线公开的应用程序提供应用程序边界。创建命名空间后，系统将生成一个 SAS 密钥。命名空间与 SAS 密钥的组合为服务总线提供了用于验证应用程序访问权限的凭据。

### 使用 Azure 经典管理门户设置命名空间

1.  登录到 [Azure 经典管理门户][]。

2.  在门户的左侧导航窗格中，单击“服务总线”。

3.  在门户的下方窗格中，单击“创建”。

    ![][6]

4.  在“添加新命名空间”页中，输入命名空间名称。系统会立即检查该名称是否可用。

    ![][7]

5.  在确保命名空间名称可用后，选择应承载您的命名空间的国家或地区（确保使用在其中部署计算资源的同一国家/地区）。此外，请确保在命名空间“类型”字段中选择“消息”，在“消息层”字段中选择“标准”。

    > [AZURE.IMPORTANT] 选取要部署应用程序的**相同区域**。这将为你提供最佳性能。

6.  单击“确定”复选标记。系统现已创建您的服务命名空间并已将其启用。您可能需要等待几分钟，因为系统将为您的帐户配置资源。

7.  在主窗口中，单击你的服务命名空间的名称。

8. 单击“连接信息”。

9.  在“访问连接信息”窗格中，找到包含 SAS 密钥和密钥名称的连接字符串。

    ![][35]

10.  记下这些凭据，或将它们复制到剪贴板。

11. 在同一门户页面中，单击页面顶部的“配置”选项卡。

12. 将 **RootManageSharedAccessKey** 策略的主密钥复制到剪贴板，或者将其粘贴到记事本。你将在本教程的后面部分使用此值。

	![][36]

## 创建 Web 角色

在本部分中，你将生成应用程序的前端。首先，你将创建应用程序显示的各种页面。之后，你将添加代码，以便将项目提交到服务总线队列并显示有关队列的状态信息。

### 创建项目

1.  使用管理员特权启动 Microsoft Visual Studio。若要使用管理员特权启动 Visual Studio，请右键单击“Visual Studio”程序图标，然后单击“以管理员身份运行”。Azure 计算模拟器（本文后面会讨论）要求使用管理员权限启动 Visual Studio。

    在 Visual Studio 的“文件”菜单中，单击“新建”，然后单击“项目”。

2.  从“Visual C#”下的“已安装模板”中，单击“云”，然后单击“Azure 云服务”。将项目命名为 **MultiTierApp**。然后，单击“确定”。

    ![][9]

3.  在“.NET Framework 4.5”角色中，双击“ASP.NET Web 角色”。

    ![][10]

4.  将鼠标指针停留在“Azure 云服务解决方案”下的“WebRole1”上，单击铅笔图标，并将 Web 角色重命名为“FrontendWebRole”。然后，单击“确定”。（请确保你输入“Frontend”而不是“FrontEnd”，此处为小写“e”。）

    ![][11]

5.  从“新建 ASP.NET 项目”对话框的“选择模板”列表中，单击“MVC”。

    ![][12]

6. 继续在“新建 ASP.NET 项目”对话框中，单击“更改身份验证”按钮。在“更改身份验证”对话框中，单击“无身份验证”，然后单击“确定”。在本教程中，你将部署无需用户登录名的应用。

	![][16]

7. 返回到“新建 ASP.NET 项目”对话框中，单击“确定”创建项目。

6.  在“解决方案资源管理器”的“FrontendWebRole”项目中，右键单击“引用”，然后单击“管理 NuGet 包”。

7.  单击“浏览”选项卡，然后搜索 `Microsoft Azure Service Bus`。单击“安装”并接受使用条款。

    ![][13]

	请注意，现已引用所需的客户端程序集并已添加部分新代码文件。

9.  在“解决方案资源管理器”中，右键单击“模型”，然后依次单击“添加”和“类”。在“名称”框中，键入名称 **OnlineOrder.cs**。然后单击“添加”。

### 为你的 Web 角色编写代码

在本部分，你将创建应用程序显示的各种页面。

1.  在 Visual Studio 的 OnlineOrder.cs 文件中将现有命名空间定义替换为以下代码：

        namespace FrontendWebRole.Models
        {
            public class OnlineOrder
            {
                public string Customer { get; set; }
                public string Product { get; set; }
            }
        }

2.  在“解决方案资源管理器”中，双击“Controllers\\HomeController.cs”。在文件顶部添加以下 **using** 语句以包括针对你刚创建的模型以及服务总线的命名空间。

        using FrontendWebRole.Models;
        using Microsoft.ServiceBus.Messaging;
        using Microsoft.ServiceBus;

3.  仍在 Visual Studio 的 HomeController.cs 文件中，将现有命名空间定义替换为以下代码。此代码包含用于处理将项提交到队列这一任务的方法。

        namespace FrontendWebRole.Controllers
        {
            public class HomeController : Controller
            {
                public ActionResult Index()
                {
                    // Simply redirect to Submit, since Submit will serve as the
                    // front page of this application.
                    return RedirectToAction("Submit");
                }

                public ActionResult About()
                {
                    return View();
                }

                // GET: /Home/Submit.
                // Controller method for a view you will create for the submission
                // form.
                public ActionResult Submit()
                {
                    // Will put code for displaying queue message count here.

                    return View();
                }

                // POST: /Home/Submit.
                // Controller method for handling submissions from the submission
                // form.
                [HttpPost]
				// Attribute to help prevent cross-site scripting attacks and
				// cross-site request forgery.  
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

4.  在“生成”菜单中，单击“生成解决方案”以测试工作的准确性。

5.  现在，为前面创建的 `Submit()` 方法创建视图。在 `Submit()` 方法（不带任何参数的 `Submit()` 的重载）中右键单击，然后选择“添加视图”。

    ![][14]

6.  此时将显示一个用于创建视图的对话框。在“模板”列表中，选择“创建”。在“模型类”列表中，单击“OnlineOrder”类。

    ![][15]

7.  单击**“添加”**。

8.  现在，请更改应用程序的显示名称。在“解决方案资源管理器”中，双击“Views\\Shared\\_Layout.cshtml”文件，在 Visual Studio 编辑器中打开该文件。

9.  将每一处 **My ASP.NET Application** 替换为 **LITWARE'S Products**。

10. 删除“Home”、“About”和“Contact”链接。删除突出显示的代码：

	![][28]

11. 最后，修改提交页以包含有关队列的一些信息。在“解决方案资源管理器”中，双击“Views\\Home\\Submit.cshtml”文件以在 Visual Studio 编辑器中将其打开。在 `<h2>Submit</h2>` 后面添加以下行。`ViewBag.MessageCount` 当前为空。稍后你将填充它。

	<p>Current number of orders in queue waiting to be processed: @ViewBag.MessageCount</p>

12. 现在，你已实现你的 UI。你可以按 **F5** 运行应用程序并确认其按预期方式运行。

    ![][17]

### 编写用于将项提交到 Service Bus 队列的代码

现在，你将添加用于将项提交到队列的代码。首先，你将创建一个包含服务总线队列连接信息的类。然后，你将从 Global.aspx.cs 初始化你的连接。最后，你将更新你之前在 HomeController.cs 中创建的提交代码以便实际将项提交到服务总线队列。

1.  在“解决方案资源管理器”中，右键单击“FrontendWebRole”（右键单击项目而不是角色）。单击“添加”，然后单击“类”。

2.  将类命名为 **QueueConnector.cs**。单击“添加”以创建类。

3.  现在，将添加可封装连接信息并初始化服务总线队列连接的代码。将 QueueConnector.cs 的全部内容替换为下面的代码，并输入 `your Service Bus namespace`（命名空间名称）和 `yourKey`（即之前在“创建服务总线命名空间”部分的步骤 12 中的 [Azure 经典管理门户][]中获取的**主要密钥**）的值。

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

                // Obtain these values from the portal.
	        public const string Namespace = "your Service Bus namespace";

                // The name of your queue.
                public const string QueueName = "OrdersQueue";

                public static NamespaceManager CreateNamespaceManager()
                {
                    // Create the namespace manager which gives you access to
                    // management operations.
                    var uri = ServiceBusEnvironment.CreateServiceUri(
                        "sb", Namespace, String.Empty);
                    var tP = TokenProvider.CreateSharedAccessSignatureTokenProvider(
                        "RootManageSharedAccessKey", "yourKey");
                    return new NamespaceManager(uri, tP);
                }

                public static void Initialize()
                {
                    // Using Http to be friendly with outbound firewalls.
                    ServiceBusEnvironment.SystemConnectivity.Mode =
                        ConnectivityMode.Http;

                    // Create the namespace manager which gives you access to
                    // management operations.
                    var namespaceManager = CreateNamespaceManager();

                    // Create the queue if it does not exist already.
                    if (!namespaceManager.QueueExists(QueueName))
                    {
                        namespaceManager.CreateQueue(QueueName);
                    }

                    // Get a client to the queue.
                    var messagingFactory = MessagingFactory.Create(
                        namespaceManager.Address,
                        namespaceManager.Settings.TokenProvider);
                    OrdersQueueClient = messagingFactory.CreateQueueClient(
                        "OrdersQueue");
                }
            }
        }


4.  现在，请确保你的 **Initialize** 方法会被调用。在“解决方案资源管理器”中，双击“Global.asax\Global.asax.cs”。

5.  在 **Application\_Start** 方法的末尾添加以下代码行。

        FrontendWebRole.QueueConnector.Initialize();

6.  最后，更新之前创建的 Web 代码以便将项提交到队列。在“解决方案资源管理器”中，双击“Controllers\HomeController.cs”。

7.  更新 `Submit()` 方法（不包含任何参数的重载），如下所示，获取队列的消息计数。

        public ActionResult Submit()
        {
            // Get a NamespaceManager which allows you to perform management and
            // diagnostic operations on your Service Bus queues.
            var namespaceManager = QueueConnector.CreateNamespaceManager();

            // Get the queue, and obtain the message count.
            var queue = namespaceManager.GetQueue(QueueConnector.QueueName);
            ViewBag.MessageCount = queue.MessageCount;

            return View();
        }

8.  更新 `Submit(OnlineOrder order)` 方法（包含一个参数的重载），如下所示，将订单信息提交到队列。

        public ActionResult Submit(OnlineOrder order)
        {
            if (ModelState.IsValid)
            {
                // Create a message from the order.
                var message = new BrokeredMessage(order);

                // Submit the order.
                QueueConnector.OrdersQueueClient.Send(message);
                return RedirectToAction("Submit");
            }
            else
            {
                return View(order);
            }
        }

9.  现在，你可以重新运行应用程序。每当你提交订单时，消息计数都会增大。

    ![][18]

## 创建辅助角色

现在，你将创建用于处理订单提交的辅助角色。此示例使用“服务总线队列的辅助角色”Visual Studio 项目模板。你已从门户中获取所需的凭据。

1. 确保已将 Visual Studio 连接到你的 Azure 帐户。

2.  在 Visual Studio 的“解决方案资源管理器”中，右键单击“MultiTierApp”项目下的“角色”文件夹。

3.  单击“添加”，然后单击“新建辅助角色项目”。此时将显示“添加新角色项目”对话框。

	![][26]

4.  在“添加新角色项目”对话框中，单击“服务总线队列的辅助角色”。

	![][23]

5.  在“名称”框中，将项目命名为 **OrderProcessingRole**。然后单击“添加”。

6.  将在“创建服务总线命名空间”部分的步骤 9 中获取的连接字符串复制到剪贴板。

7.  在“解决方案资源管理器”中，右键单击你在步骤 5 中创建的“OrderProcessingRole”（确保右键单击“角色”下的“OrderProcessingRole”而不是类）。然后单击“属性”。

8.  在“属性”对话框的“设置”选项卡中，在“Microsoft.ServiceBus.ConnectionString”的“值”框内单击，然后粘贴你在步骤 6 中复制的终结点值。

	![][25]

9.  当你从队列中处理订单时，创建一个 **OnlineOrder** 类来表示这些订单。你可以重用已创建的类。在“解决方案资源管理器”中，右键单击“OrderProcessingRole”类（右键单击类图标，而不是角色）。单击“添加”，然后单击“现有项”。

10. 浏览到 **FrontendWebRole\\Models** 的子文件夹，然后双击“OnlineOrder.cs”以将其添加到此项目中。

11. 在 **WorkerRole.cs** 中，将 **QueueName** 变量的值从 `"ProcessingQueue"` 更改为 `"OrdersQueue"`，如以下代码所示。


	// The name of your queue.
	const string QueueName = "OrdersQueue";


12. 在 WorkerRole.cs 文件顶部添加以下 using 语句。


	using FrontendWebRole.Models;


13. 在 `Run()` 函数中，在 `OnMessage()` 调用的内部，将 `try` 子句的内容替换为以下代码。


	Trace.WriteLine("Processing", receivedMessage.SequenceNumber.ToString());
	// View the message as an OnlineOrder.
	OnlineOrder order = receivedMessage.GetBody<OnlineOrder>();
	Trace.WriteLine(order.Customer + ": " + order.Product, "ProcessingMessage");
	receivedMessage.Complete();


14. 你已完成此应用程序。你可以测试整个应用程序，方法是右键单击“解决方案资源管理器”中的 MultiTierApp 项目，选择“设置为启动项目”，然后按 F5。请注意，消息计数不会递增，因为辅助角色会处理队列中的项并将其标记为完成。你可以通过查看 Azure 计算模拟器 UI 来查看辅助角色的跟踪输出。可通过右击任务栏的通知区域中的模拟器图标并选择“显示计算模拟器 UI”来执行此操作。

    ![][19]

    ![][20]

## <a name="next-steps"></a> 后续步骤  

若要了解有关服务总线的详细信息，请参阅以下资源：

* [Azure 服务总线][sbmsdn]  
* [服务总线服务页][sbacom]  
* [如何使用 Service Bus 队列][sbacomqhowto]  

若要了解有关多层方案的详细信息，请参阅：

* [使用存储表、队列和 Blob 的 .NET 多层应用程序][mutitierstorage]  

  [0]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-01.png
  [1]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-100.png
  [2]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-101.png
  [9]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-10.png
  [10]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-11.png
  [11]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-02.png
  [12]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-12.png
  [13]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-13.png
  [14]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-33.png
  [15]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-34.png
  [16]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-14.png
  [17]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-36.png
  [18]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-37.png

  [19]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-38.png
  [20]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-39.png
  [23]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/SBWorkerRole1.png
  [25]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/SBWorkerRoleProperties.png
  [26]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/SBNewWorkerRole.png
  [28]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-40.png

  [sbmsdn]: http://msdn.microsoft.com/zh-cn/library/azure/ee732537.aspx  
  [sbwacom]: /documentation/services/service-bus/  
  [sbwacomqhowto]: /documentation/articles/service-bus-dotnet-get-started-with-queues/  
  [mutitierstorage]: https://code.msdn.microsoft.com/Windows-Azure-Multi-Tier-eadceb36
  

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description:update wording and link references-->