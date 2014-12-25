<properties linkid="multi-tier-application" urlDisplayName="multi-tier-application" pageTitle="multi-tier-application" metaKeywords="multi-tier-application" description="multi-tier-application" metaCanonical="" services="" documentationCenter="develop"  title="中国 Windows Azure 应用程序开发人员说明" authors="" solutions="" manager="TK" editor="Haifeng Liu" />

<h1>使用 Service Bus 队列创建 .NET 多层应用程序</h1>
<p>使用 Visual Studio 2012 和免费的 Windows Azure SDK for .NET，可以轻松针对 Windows Azure 进行开发。如果您还没有 Visual Studio 2012，则此 SDK 将自动安装 Visual Web Developer Express，以便您能完全免费地开始针对 Windows Azure 进行开发。本指南假定您之前未使用过 Windows Azure。完成本指南之后，您将拥有使用多项 Windows Azure 资源的应用程序，该应用程序在本地环境中运行并演示多层应用程序的工作方式。</p>
<p>您将了解：</p>
<ul>
<li>如何通过单个下载和安装来使您的计算机能够进行 Windows Azure 开发。</li>
<li>如何使用 Visual Studio 针对 Windows Azure 进行开发。</li>
<li>如何使用 Web 角色和辅助角色在 Windows Azure 中创建多层应用程序。</li>
<li>如何使用 Service Bus 队列在各层之间进行通信。</li>
</ul>
<p>您将生成一个前端 ASP.NET MVC Web 角色，该角色使用后端辅助角色来处理长时间运行的作业。您将了解如何创建多角色解决方案以及如何使用 Service Bus 队列实现角色间通信。下面显示了已完成应用程序的屏幕快照：</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-multi-tier-01.png"/></p>
<p><strong>注意</strong>：Windows Azure 还提供了存储队列功能。有关 Windows Azure 存储队列和 Service Bus 队列的详细信息，请参阅 <a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/hh767287.aspx">Windows Azure 队列和 Windows Azure Service Bus 队列 - 比较与对照</a>。</p>
<h2><span class="short-header">角色间通信</span>方案概述 </h2>
<p>若要提交处理命令，以 Web 角色运行的前端 UI 组件需要与以辅助角色运行的中间层逻辑进行交互。此示例使用 Service Bu 中转消息传递在各层之间进行通信。</p>
<p>在 Web 层和中间层之间使用代理消息传递将分离这两个组件。与直接消息传递（即 TCP 或 HTTP）相反，Web 层不会直接连接到中间层，而是将工作单元作为消息推送到 Service Bus，Service Bus 将以可靠方式保留这些工作单元，直到中间层准备好使用和处理它们。</p>
<p>Service Bus 提供了两个实体以支持中转消息传递、队列和主题。通过队列，发送到队列的每个消息均由一个接收方使用。主题支持发布/订阅模式，在该模式中，会为注册到主题中的每个订阅提供每个已发布消息。每个订阅都会以逻辑方式保留其自己的消息队列。此外，还可以使用筛选规则配置订阅，这些规则可将传递给订阅队列的消息集限制为符合筛选条件的消息集。此示例使用 Service Bus 队列。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-multi-tier-100.png"/></p>
<p>与直接消息传递相比，此通信机制具有多项优势，即：</p>
<ul>
<li>
<p><strong>暂时分离。</strong>利用异步消息传递模式，创建者和使用者无需同时联机。Service Bus 会以可靠方式存储消息，直到使用方准备好接收这些消息。这将允许分布式应用程序的组件断开连接，例如，为进行维护而自动断开，或因组件故障断开连接，而不会影响系统的整体性能。此外，使用方应用程序可能只需在一天的特定时段内联机。</p>
</li>
<li>
<p><strong>负荷量</strong>。在许多应用程序中，系统负荷随时间而变化，而每个工作单元所需的处理时间通常为常量。使用队列在消息创建者与使用者之间中继意味着，只需将使用方应用程序（辅助）设置为适应平均负荷而非最大负荷。队列深度将随传入负荷的变化而加大和减小。这将直接根据为应用程序加载提供服务所需的基础结构的数目来节省成本。</p>
</li>
<li>
<p><strong>负载平衡。</strong>随着负载增加，可添加更多的辅助进程以从队列中读取。每条消息仅由一个辅助进程处理。另外，可通过此基于拉取的负载平衡来以最合理的方式使用辅助计算机，即使这些辅助计算机具有不同的处理能力（因为它们将以其最大速率拉取消息）也是如此。此模式通常称为“竞争使用者模式”。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-multi-tier-101.png"/></p>
</li>
</ul>
<p>以下各节讨论了实现此体系结构的代码。</p>
<h2>设置开发环境</h2>
<p>您需要先获得相应工具并设置开发环境，然后才能开始开发 Windows Azure 应用程序。</p>
<ol>
<li>
<p>若要安装 Windows Azure SDK for .NET，请单击以下按钮：</p>
<p><a href="http://azure.microsoft.com/zh-cn/develop/net/">获取工具和 SDK</a></p>
</li>
<li>
<p>单击“安装 SDK”。</p>
</li>
<li>
<p>选择您使用的 Visual Studio 版本相应的链接。本教程中的步骤使用 Visual Studio 2012：</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-41.png"/></p>
</li>
<li>
<p>当提示您运行或保存安装文件时，请单击“运行”：</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-3.png"/></p>
</li>
<li>
<p>在 Web 平台安装程序中，单击“安装”，然后继续安装：</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-4-webpi.png"/></p>
</li>
<li>
<p>安装完成后，您便已做好开发准备工作。SDK 包含了一些工具，可利用这些工具在 Visual Studio 中轻松开发 Windows Azure 应用程序。如果您尚未安装 Visual Studio，则 SDK 也会安装免费的 Visual Web Developer Express。</p>
</li>
</ol>
<h2>设置 Service Bus 命名空间</h2>
<p>下一步是创建服务命名空间并获取共享密钥。服务命名空间为通过 Service Bus 公开的每个应用程序提供一个应用程序边界。 创建服务命名空间时，系统将自动生成共享密钥。服务命名空间和共享密钥的组合为 Service Bus 提供了用于验证对应用程序的访问的凭据。</p>
<ol>
<li>
<p>登录到 <a href="http://manage.windowsazure.cn">Windows Azure 管理门户</a>。</p>
</li>
<li>
<p>在该管理门户的左侧导航窗格中，单击“Service Bus”。</p>
</li>
<li>
<p>在管理门户的下方窗格中，单击“创建”。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/sb-queues-03.png"/></p>
</li>
<li>
<p>在“添加新命名空间”对话框中，输入命名空间名称。系统会立即检查该名称是否可用。<br /><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/sb-queues-04.png"/></p>
</li>
<li>
<p>在确保命名空间名称可用后，选择应承载您的命名空间的国家或地区（确保使用在其中部署计算资源的同一国家/地区）。</p>
<p>重要说明：选取要部署应用程序的<strong>相同区域</strong>。这将为您提供最佳性能。</p>
</li>
<li>
<p>单击复选标记。系统现已创建您的服务命名空间并已将其启用。您可能需要等待几分钟，因为系统将为您的帐户设置资源。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-multi-tier-27.png"/></p>
</li>
<li>
<p>在主窗口中，单击您的服务命名空间的名称。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/sb-queues-09.png"/></p>
</li>
<li>
<p>单击“访问密钥”。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/sb-queues-06.png"/></p>
</li>
<li>
<p>在“连接到您的命名空间”窗格中，找到“默认颁发者”和“默认密钥”条目。</p>
</li>
<li>
<p>记下该密钥或将其复制到剪贴板。</p>
</li>
</ol>
<h2>创建 Web 角色</h2>
<p>在本节中，您将生成应用程序的前端。您首先将创建应用程序显示的各种页面。之后，您将添加代码，这些代码用于将项提交到 Service Bus 队列并显示有关队列的状态信息。</p>
<h3>创建项目</h3>
<ol>
<li>
<p>使用管理员权限启动 Microsoft Visual Studio 2012 或 Microsoft Visual Web Developer Express。若要使用管理员权限启动 Visual Studio，请右键单击“Microsoft Visual Studio 2012”（或“Microsoft Visual Web Developer Express”），然后单击“以管理员身份运行”。Windows Azure 计算仿真程序（本指南后面会讨论）要求使用管理员权限启动 Visual Studio。</p>
<p>在 Visual Studio 中的“文件”菜单中，单击“新建”，然后单击“项目”。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-multi-tier-09.png"/></p>
</li>
<li>
<p>从“已安装的模板”的“Visual C#”下，单击“云”，然后单击“Windows Azure 项目”。将项目命名为“MultiTierApp”。然后单击“确定”。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-multi-tier-10.jpg"/></p>
</li>
<li>
<p>从“.NET Framework 4”角色中，双击“ASP.NET MVC 4 Web 角色”。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-multi-tier-11.png"/></p>
</li>
<li>
<p>将鼠标指针停留在“Windows Azure 解决方案”下的“MvcWebRole1”上，单击铅笔图标，并将 Web 角色重命名为“FrontendWebRole”。然后单击“确定”。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-multi-tier-02.png"/></p>
</li>
<li>
<p>从“选择模板”列表中，单击“Internet 应用程序”，然后单击“确定”。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-multi-tier-12.png"/></p>
</li>
<li>
<p>在“解决方案资源管理器”中，右键单击“引用”，然后单击“管理 NuGet 包...”或“添加库程序包引用”。</p>
</li>
<li>
<p>在该对话框的左侧选择“联机”。搜索“WindowsAzure”，然后选择“Windows Azure Service Bus”项。然后，完成安装过程并关闭此对话框。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-multi-tier-13.png"/></p>
</li>
<li>
<p>请注意，现已引用所需的客户端程序集并已添加部分新代码文件。</p>
</li>
<li>
<p>在“解决方案资源管理器”中，右键单击“模型”，单击“添加”，然后单击“类”。在“名称”框中，键入名称“OnlineOrder.cs”。然后单击“添加”。</p>
</li>
</ol>
<h3>为您的 Web 角色编写代码</h3>
<p>在本节中，您将创建应用程序显示的各种页面。</p>
<ol>
<li>
<p>在 Visual Studio 中，在 <strong>OnlineOrder.cs</strong> 文件中，将现有命名空间定义替换为以下代码：</p>
<pre class="prettyprint">namespace FrontendWebRole.Models
{
    public class OnlineOrder
    {
        public string Customer { get; set; }
        public string Product { get; set; }
    }
}</pre>
</li>
<li>
<p>在“解决方案资源管理器”中，双击 <strong>Controllers\HomeController.cs</strong>。在文件顶部添加以下 <strong>using</strong> 语句以包括针对您刚创建的模型以及 Service Bus 的命名空间：</p>
<pre class="prettyprint">using FrontendWebRole.Models;
using Microsoft.ServiceBus.Messaging;
using Microsoft.ServiceBus;</pre>
</li>
<li>
<p>仍在 Visual Studio 中，在 <strong>HomeController.cs</strong> 文件中，将现有命名空间定义替换为以下代码。此代码包含用于处理将项提交到队列这一任务的方法：</p>
<pre class="prettyprint">namespace FrontendWebRole.Controllers
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
    // Controler method for handling submissions from the submission
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


}</pre>
</li>
<li>
<p>从“构建”菜单中，单击“构建解决方案”。</p>
</li>
<li>
<p>现在，您将为上面创建的 <strong>Submit()</strong> 方法创建视图。在 Submit() 方法内右键单击，然后选择“添加视图”</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-multi-tier-33.png"/></p>
</li>
<li>
<p>这将显示一个用于创建视图的对话框。单击“创建强类型视图”所对应的复选框。此外，在“模型类”下拉列表中选择“OnlineOrder”类，然后在“支架模板”下拉列表下选择“创建”。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-multi-tier-34.png"/></p>
</li>
<li>
<p>单击“添加”。</p>
</li>
<li>
<p>现在，您将更改应用程序的显示名称。在“解决方案资源管理器”中，双击 <strong>Views\Shared\_Layout.cshtml</strong> 文件以在 Visual Studio 编辑器中将其打开。</p>
</li>
<li>
<p>将 <strong>My ASP.NET MVC Application</strong> 的所有实例替换为 <strong>LITWARE'S Awesome Products</strong>。</p>
</li>
<li>
<p>将“your logo here”替换为 <strong>LITWARE'S Awesome Products</strong>：</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-multi-tier-35.png"/></p>
</li>
<li>
<p>删除“Home”、“About”和“Contact”链接。删除突出显示的代码：</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-multi-tier-40.png"/></p>
</li>
<li>
<p>最后，微调提交页以包含有关队列的一些信息。在“解决方案资源管理器”中，双击 <strong>Views\Home\Submit.cshtml</strong> 文件以在 Visual Studio 编辑器中将其打开。在 <strong>&lt;h2&gt;Submit&lt;/h2&gt;</strong> 后面添加以下行。现在，<strong>ViewBag.MessageCount</strong> 为空。稍后您将填充它。</p>
<pre class="prettyprint">&lt;p&gt;Current Number of Orders in Queue Waiting to be Processed: @ViewBag.MessageCount&lt;/p&gt;</pre>
</li>
<li>
<p>现在，您已实现您的 UI。您可以按 <strong>F5</strong> 运行应用程序并确认其按预期方式运行。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-multi-tier-36.png"/></p>
</li>
</ol>
<h3>编写用于将项提交到 Service Bus 队列的代码</h3>
<p>现在，您将添加用于将项提交到队列的代码。您首先将创建一个包含 Service Bus 队列连接信息的类。然后，您将从 <strong>Global.aspx.cs</strong> 初始化您的连接。最后，您将更新您之前在 <strong>HomeController.cs</strong> 中创建的提交代码以便实际将项提交到 Service Bus 队列。</p>
<ol>
<li>
<p>在“解决方案资源管理器”中，右键单击“FrontendWebRole”（右键单击项目而不是角色）。单击“添加”，然后单击“类”。</p>
</li>
<li>
<p>将类命名为 <strong>QueueConnector.cs</strong>。单击“添加”以创建类。</p>
</li>
<li>
<p>现在，您将粘贴代码，该代码用于封装连接信息和包含用于初始化与 Service Bus 队列的连接的方法。在 QueueConnector.cs 中，粘贴以下代码，然后为 <strong>Namespace</strong>、<strong>IssuerName</strong> 和 <strong>IssuerKey</strong> 输入值。您可以在<a href="http://manage.windowsazure.cn">管理门户</a>中找到这些值。</p>
<pre class="prettyprint">using System;
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


}</pre>
</li>
<li>
<p>现在，您将确保您的 <strong>Initialize</strong> 方法会被调用。在“解决方案资源管理器”中，双击 <strong>Global.asax\Global.asax.cs</strong>。</p>
</li>
<li>
<p>将以下行添加到 <strong>Application_Start</strong> 方法的底部：</p>
<pre class="prettyprint">QueueConnector.Initialize();</pre>
</li>
<li>
<p>最后，您将更新之前创建的 Web 代码以便将项提交到队列。在“解决方案资源管理器”中，双击您之前创建的 <strong>Controllers\HomeController.cs</strong>。</p>
</li>
<li>
<p>更新 <strong>Submit()</strong> 方法（如下所示）获取队列的消息计数：</p>
<pre class="prettyprint">public ActionResult Submit()
{            
    // Get a NamespaceManager which allows you to perform management and
    // diagnostic operations on your Service Bus Queues.
    var namespaceManager = QueueConnector.CreateNamespaceManager();


// Get the queue, and obtain the message count.
var queue = namespaceManager.GetQueue(QueueConnector.QueueName);
ViewBag.MessageCount = queue.MessageCount;


return View();


}</pre>
</li>
<li>
<p>更新 <strong>Submit(OnlineOrder order)</strong> 方法（如下所示）将订单信息提交到队列：</p>
<pre class="prettyprint">public ActionResult Submit(OnlineOrder order)
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


}</pre>
</li>
<li>
<p>现在，您可以重新运行您的应用程序。每当您提交订单时，消息计数都会增大。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-multi-tier-37.png"/></p>
</li>
</ol>
<h2>云配置管理器</h2>
<p>Windows Azure 支持一组托管 API，提供了跨 Microsoft 云服务创建 Windows Azure 服务客户端（例如 Service Bus）新实例的一致方法。这些 API 使您能够实例化这些客户端（例如 <strong>CloudBlobClient</strong>、<strong>QueueClient</strong>、<strong>TopicClient</strong>），而无论应用程序托管在何处 -- 在本地、在 Microsoft 云服务中、在网站中或是在永久性虚拟机角色中。您还可以使用这些 API 检索实例化这些客户端所需的配置信息，以及更改配置，而无需重新部署调用应用程序。这些 API 位于 <a href="http://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.cloudconfigurationmanager.aspx">Microsoft.WindowsAzure.Configuration.CloudConfigurationManager</a> 类中。还有一些 API 位于客户端。</p>
<h3>连接字符串</h3>
<p>若要实例化客户端（例如 Service Bus <strong>QueueClient</strong>），可以将配置信息表示为连接字符串。在客户端，有一个通过使用该连接字符串实例化客户端类型的 <strong>CreateFromConnectionString()</strong> 方法。例如，考虑下面的配置部分：</p>
<pre class="prettyprint">&lt;ConfigurationSettings&gt;
â€¦
    &lt;Setting name="Microsoft.ServiceBus.ConnectionString" value="Endpoint=sb://[yourServiceNamespace].servicebus.chinacloudapi.cn/;SharedSecretIssuer=[issuerName];SharedSecretValue=[yourDefaultKey]" /&gt;
&lt;/ConfigurationSettings&gt;</pre>
<p>以下代码检索连接字符串，创建队列并初始化与队列的连接：</p>
<pre class="prettyprint">QueueClient Client; 

string connectionString =
 CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

var namespaceManager =
 NamespaceManager.CreateFromConnectionString(connectionString); 

if (!namespaceManager.QueueExists(QueueName))
{
    namespaceManager.CreateQueue(QueueName);
}

// Initialize the connection to Service Bus Queue
Client = QueueClient.CreateFromConnectionString(connectionString, QueueName);</pre>
<p>以下部分中的代码使用这些配置管理 API。</p>
<h2><span class="short-header">创建辅助角色 </span></h2>
<p>现在，您将创建用于处理订单提交的辅助角色。此示例使用“Service Bus 队列的辅助角色”Visual Studio 项目模板。首先，您将使用 Visual Studio 中的“服务器资源管理器”获取所需凭据。</p>
<ol>
<li>
<p>从 Visual Studio 的菜单栏中，选择“视图”，然后单击“服务器资源管理器”。“Windows Azure Service Bus”节点将显示在“服务器资源管理器”层次结构中，如下图所示。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/sbexplorer.png"/></p>
</li>
<li>
<p>在“服务器资源管理器”中，右键单击“Windows Azure Service Bus”，然后单击“添加新连接”。</p>
</li>
<li>
<p>在“添加连接”对话框中，键入服务命名空间的名称、颁发者名称和颁发者密钥。然后单击“确定”进行连接。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/sbexploreraddconnect.png"/></p>
</li>
<li>
<p>在 Visual Studio 的“解决方案资源管理器”中，右键单击“MultiTierApp”项目下的“角色”文件夹。</p>
</li>
<li>
<p>单击“添加”，然后单击“新建辅助角色项目”。这将显示“添加新角色项目”对话框。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/sbnewworkerrole.png"/></p>
</li>
<li>
<p>在“添加新角色项目”对话框中，单击“Service Bus 队列的辅助角色”，如下图所示：</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/sbworkerrole1.png"/></p>
</li>
<li>
<p>在“名称”框中，将项目命名为 <strong>OrderProcessingRole</strong>。然后单击“添加”。</p>
</li>
<li>
<p>在“服务器资源管理器”中，右键单击服务命名空间的名称，然后单击“属性”。在 Visual Studio 的“属性”窗格中，第一个条目包含使用包含所需授权凭据的服务命名空间终结点填充的连接字符串。有关示例，请参见下图。双击“ConnectionString”，然后按 <strong>Ctrl+C</strong> 将此字符串复制到剪贴板中。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/sbexplorerproperties.png"/></p>
</li>
<li>
<p>在“解决方案资源管理器”中，右键单击您在步骤 7 中创建的“OrderProcessingRole”（确保右键单击“角色”下的“OrderProcessingRole”而不是类）。然后单击“属性”。</p>
</li>
<li>
<p>在“属性”对话框的“设置”选项卡中，在“Microsoft.ServiceBus.ConnectionString”的“值”框内单击，然后粘贴您在步骤 8 中复制的终结点值。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/sbworkerroleproperties.png"/></p>
</li>
<li>
<p>当您从队列中处理订单时，创建一个 <strong>OnlineOrder</strong> 类来表示这些订单。您可以重用已创建的类。在“解决方案资源管理器”中，右键单击“OrderProcessingRole”项目（右键单击项目而不是角色）。单击“添加”，然后单击“现有项”。</p>
</li>
<li>
<p>浏览到 <strong>FrontendWebRole\Models</strong> 的子文件夹，然后双击“OnlineOrder.cs”以将其添加到此项目中。</p>
</li>
<li>
<p>将 <strong>WorkerRole.cs</strong> 中 <strong>QueueName</strong> 变量的值 <code>"ProcessingQueue"</code> 替换为 <code>"OrdersQueue"</code>，如以下代码所示：</p>
<pre class="prettyprint">// The name of your queue
const string QueueName = "OrdersQueue";</pre>
</li>
<li>
<p>在 WorkerRole.cs 文件顶部添加以下 using 语句：</p>
<pre class="prettyprint">using FrontendWebRole.Models;</pre>
</li>
<li>
<p>在 <code>Run()</code> 函数中，在 <code>if (receivedMessage != null)</code> 循环内的 <code>Trace</code> 语句下面添加以下代码：</p>
<pre class="prettyprint">if (receivedMessage != null)
{
    // Process the message
    Trace.WriteLine("Processing", receivedMessage.SequenceNumber.ToString());


// Add these two lines of code
// View the message as an OnlineOrder
OnlineOrder order = receivedMessage.GetBody&amp;lt;OnlineOrder&amp;gt;();
Trace.WriteLine(order.Customer + ": " + order.Product, "ProcessingMessage");


receivedMessage.Complete();


}</pre>
</li>
<li>
<p>您已完成此应用程序。可通过按 F5 来测试完整的应用程序，就像您之前所做的那样。请注意，消息计数不会递增，因为辅助角色会处理队列中的项并将其标记为完成。您可以通过查看 Windows Azure 计算仿真程序用户界面来查看辅助角色的跟踪输出。可通过右击任务栏的通知区域中的仿真程序图标并选择<strong>“显示计算仿真程序用户界面”</strong>来执行此操作。</p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-multi-tier-38.png"/></p>
<p><img src="http://wacnstorage.blob.core.chinacloudapi.cn/marketing-resource/media/devcenter/dotnet/getting-started-multi-tier-39.png"/></p>
</li>
</ol>
<h2><a name="nextsteps"></a><span class="short-header">后续步骤 </span></h2>
<p>若要了解有关 Service Bus 的详细信息，请使用以下资源：</p>
<ul>
<li><a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/ee732537.aspx">Windows Azure Service Bus</a></li>
<li><a href="/zh-cn/manage/services/service-bus/">Service Bus 操作方法</a></li>
<li><a href="/zh-cn/develop/net/how-to-guides/service-bus-queues/">如何使用 Service Bus 队列</a></li>
</ul>
<p>若要了解有关多层方案的详细信息，请参阅：</p>
<ul>
<li><a href="/zh-cn/develop/net/tutorials/multi-tier-web-site/1-overview/">使用存储表、队列和 Blob 的 .NET 多层应用程序</a></li>
</ul>
</div>