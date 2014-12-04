<properties linkid="dev-net-how-to-service-bus-relay" urlDisplayName="Service Bus Relay" pageTitle="How to use Service Bus relay (.NET) - Azure" metaKeywords="get started azure Service Bus Relay C# " description="Learn how to use the Azure Service Bus relay service to connect two applications hosted in different locations." metaCanonical="" services="service-bus" documentationCenter=".NET" title="How to Use the Service Bus Relay Service" authors="sethm" solutions="" manager="dwrede" editor="mattshel" />

# 如何使用 Service Bus 中继服务

本指南演示如何使用 Service Bus 中继服务。示例是用 C# 编写的，并且使用 Service Bus 程序集中包含的扩展型 Windows Communication Foundation API，该程序集是用于 Azure 的 .NET 库的组成部分。有关 Service Bus 中继的详细信息，请参阅[后续步骤][后续步骤]一节。

[WACOM.INCLUDE [create-account-note](../includes/create-account-note.md)]

## <span class="short-header">什么是 Service Bus 中继</span>什么是 Service Bus 中继

Service Bus **中继**服务使你能构建可在 Azure 数据中心和你自己的本地企业环境中运行的**混合应用程序**。Service Bus 中继可简化这一过程，它允许你安全地向公有云公开位于企业网络内的 Windows Communication Foundation (WCF) 服务，而无需打开防火墙连接，也无需对企业网络基础结构进行彻底的更改。

![中继概念][中继概念]

Service Bus 中继使你能够在现有企业环境中托管 WCF 服务。然后，你可以将侦听传入会话和请求这些 WCF 服务的任务委托给在 Azure 内运行的 Service Bus。这使你能够向 Azure 中运行的应用程序代码或者向移动工作者或 Extranet 合作伙伴环境公开这些服务。Service Bus 允许你精确、安全地控制谁可以访问这些服务。它提供了一种强大且安全的方式，从你的现有企业解决方案公开应用程序功能和数据并从云中利用这些功能和数据。

本指南演示如何使用 Service Bus 中继创建 WCFWeb 服务，并使用 TCP 通道绑定（可实现双方之间安全的对话）公开该服务。

## <span class="short-header">创建服务命名空间</span>创建服务命名空间

若要开始在 Azure 中使用 Service Bus 中继，必须先创建一个服务命名空间。服务命名空间提供了用于对应用程序中的 Service Bus 资源进行寻址的范围容器。

创建服务命名空间：

1.  登录到 [Azure 管理门户][Azure 管理门户]。

2.  在该管理门户的左侧导航窗格中，单击
    **Service Bus**。

3.  在该管理门户的下方窗格中，单击“创建”。

    ![][0]

4.  在“添加新命名空间”对话框中，输入命名空间名称。
    系统会立即检查该名称是否可用。

    ![][1]

5.  在确保命名空间名称可用后，选择应托管你的命名空间的国家或地区（确保使用在其中部署计算资源的同一国家/地区）。

    重要说明：选取要部署应用程序的**相同区域**。这将为你提供最佳性能。

6.  单击复选标记。系统现已创建你的服务命名空间并已将其启用。你可能需要等待几分钟，因为系统将为你的帐户配置资源。

    ![][2]

    你创建的命名空间随后将显示在管理门户中，然后要花费一段时间来激活。请等到状态变为“活动”后再继续。

## <span class="short-header">获取管理凭据</span>获取命名空间的默认管理凭据

若要在新命名空间上执行管理操作（如创建中继连接），你必须获取该命名空间的管理凭据。

1.  在左侧导航窗格中，单击“Service Bus”节点以
    显示可用命名空间的列表：
    ![][0]

2.  从显示的列表中选择刚刚创建的命名空间：
    ![][3]

3.  单击**“连接信息”**。
    ![][4]

4.  在“访问连接信息”对话框中，找到“默认颁发者”和“默认密钥”条目。记下这些值，因为你将在下面使用此信息来对命名空间执行操作。

## <span class="short-header">获取 NuGet 包</span>获取 Service Bus NuGet 包

Service Bus **NuGet** 包是获取 Service Bus API 并为应用程序配置所有 Service Bus 依赖项的最简单的方法。利用 NuGet Visual Studio 扩展，可以轻松地在 Visual Studio 和 Visual Studio Express 2012 for Web 中安装和更新库和工具。Service Bus NuGet 包是获取 Service Bus API 并为应用程序配置所有 Service Bus 依赖项的最简单的方法。

要在你的应用程序中安装 NuGet 包，请执行以下操作：

1.  在“解决方案资源管理器”中，右键单击**“引用”**，然后单击**“管理 NuGet 包”**。
2.  搜索“WindowsAzure”，然后选择“Azure Service Bus”项。单击“安装”以完成安装，然后关闭此对话框。

    ![][5]

## <span class="short-header">公开和使用 SOAP Web 服务</span>如何使用 Service Bus 通过 TCP 公开和使用 SOAP Web 服务

要公开现有 WCF SOAP Web 服务以供外部使用，你必须更改服务绑定和地址。这可能需要更改你的配置文件或者可能需要更改代码，具体取决于你如何设置和配置 WCF 服务。请注意，WCF 允许你对同一服务使用多个网络终结点，因此你可以在添加 Service Bus 终结点以便进行
外部访问的同时保留现有内部终结点。

在此任务中，你将构建一个简单的 WCF 服务并向其添加 Service Bus 侦听程序。此练习假定你熟悉 Visual Studio 2012，因此不演练创建项目的所有详细信息，而是侧重于代码。

在开始下面的步骤之前，请完成以下过程以设置你的环境：

1.  在 Visual Studio 中，在解决方案内创建一个包含以下两个
    项目的控制台应用程序：“客户端”和“服务”。
2.  将这两个项目的目标框架均设置为 .NET Framework 4。
3.  给两个项目都添加“Azure Service Bus NuGet”包。
    这样就会给项目添加所有必要的程序集引用。

### 如何创建该服务

首先，创建该服务本身。任何 WCF 服务都包含
至少三个不同部分：

-   描述交换哪些信息以及将调用哪些
    操作的协定的定义。
-   上述协定的实施方案。
-   托管该服务并公开多个终结点
    的主机。

本节中的代码示例涵盖了其中每个部分。

协定定义用于添加两个数字并返回相应结果的单个操作 **AddNumbers**。**IProblemSolverChannel** 接口使客户端能够更轻松地管理代理生存期。创建这样一个接口被认为是最佳实践。最好将此协定定义放入单独的文件中，以便可从你的“客户端”和“服务”两个项目中引用该文件，但也可以将代码复制到这两个项目中：

        using System.ServiceModel;
     
        [ServiceContract(Namespace = "urn:ps")]
        interface IProblemSolver
        {
            [OperationContract]
            int AddNumbers(int a, int b);
        }
     
        interface IProblemSolverChannel : IProblemSolver, IClientChannel {}

协定到位后，实施起来就很简单了：

        class ProblemSolver : IProblemSolver
        {
            public int AddNumbers(int a, int b)
            {
                return a + b;
            }
        }

**如何以编程方式配置服务主机**

协定和实施完成后，你现在就可以托管服务了。托管发生在 **System.ServiceModel.ServiceHost** 对象内，该对象
负责管理服务实例并托管侦听消息的终结点。下面的代码使用常规的本地终结点和 Service Bus 终结点来配置服务，以便并列展示内部和外部终结点的外观。将字符串“\*\*namespace\*\*”替换为你的命名空间名称，并将“\*\*key\*\*”替换为你在上面的设置步骤中获取的颁发者密钥。

    ServiceHost sh = new ServiceHost(typeof(ProblemSolver));

    sh.AddServiceEndpoint(
       typeof (IProblemSolver), new NetTcpBinding(), 
       "net.tcp://localhost:9358/solver");

    sh.AddServiceEndpoint(
       typeof(IProblemSolver), new NetTcpRelayBinding(), 
       ServiceBusEnvironment.CreateServiceUri("sb", "**namespace**", "solver"))
        .Behaviors.Add(new TransportClientEndpointBehavior {
              TokenProvider = TokenProvider.CreateSharedSecretTokenProvider( "owner", "**key**")});

    sh.Open();

    Console.WriteLine("Press ENTER to close");
    Console.ReadLine();

    sh.Close();

在本示例中，你将创建两个位于同一协定实施中的终结点。一个是本地的，一个通过 Service Bus 进行投影。两者之间的主要区别是绑定；本地终结点使用**NetTcpBinding**，而 Service Bus 终结点和地址使用**NetTcpRelayBinding**。本地终结点有一个使用不同端口的本地网络地址。Service Bus 终结点有一个由字符串“sb”、你的命名空间名称、路径“solver”组成的终结点地址。这将生成 URI“sb://[serviceNamespace].servicebus.windows.net/solver”，将服务终结点标识为具有完全限定的外部 DNS 名称的 Service Bus TCP 终结点。如果将替换上述占位符的代码放入“服务”应用程序的**Main** 函数中，你将会获得一个可正常运行的服务。如果你希望你的服务专门侦听 Service Bus，请删除本地终结点声明。

**如何在 App.config 文件中配置服务主机**

你还可以使用 App.config 文件配置主机。在此情况下，服务托管代码如下所示：

    ServiceHost sh = new ServiceHost(typeof(ProblemSolver));
    sh.Open();
    Console.WriteLine("Press ENTER to close");
    Console.ReadLine();
    sh.Close();

终结点定义将移到 App.config 文件中。请注意，**NuGet** 包已向 App.config 文件添加一系列定义，这些定义是 Service Bus 必需的配置扩展。以下代码段（与上面列出的代码完全等效）应该直接显示在 **system.serviceModel** 元素下。此代码段假定你的项目 C# 命名空间名为“Service”。
请将占位符替换为你的 Service Bus 服务命名空间和密钥。

    <services>
        <service name="Service.ProblemSolver">
            <endpoint contract="Service.IProblemSolver"
                      binding="netTcpBinding"
                      address="net.tcp://localhost:9358/solver"/>
            <endpoint contract="Service.IProblemSolver"
                      binding="netTcpRelayBinding"
                      address="sb://**namespace**.servicebus.windows.net/solver"
                      behaviorConfiguration="sbTokenProvider"/>
        </service>
    </services>
    <behaviors>
        <endpointBehaviors>
            <behavior name="sbTokenProvider">
                <transportClientEndpointBehavior>
                    <tokenProvider>
                        <sharedSecret issuerName="owner" issuerSecret="**key**" />
                    </tokenProvider>
                </transportClientEndpointBehavior>
            </behavior>
        </endpointBehaviors>
    </behaviors>

进行这些更改后，该服务将像以前一样启动，但具有两个活动终结点：一个本地的，一个在云中进行侦听。

### 如何创建客户端

**如何以编程方式配置客户端**

若要使用该服务，你可以使用 **ChannelFactory** 对象构造 WCF 客户端。Service Bus 使用通过 Access Control 服务 (ACS) 实现的基于声明的安全模型。**TokenProvider** 类代表具有内置工厂方法的安全令牌提供程序，这些方法可返回一些众所周知的令牌提供程序。下面的示例使用 **SharedSecretTokenProvider** 保存共享机密凭据并从访问控制服务获取相应令牌。名称和密钥是根据上一节所述从门户获取的凭据。

首先，在你的客户端项目中引用服务中的 **IProblemSolver** 协定代码或将其复制到你的客户端项目中。

然后，替换客户端的 **Main** 方法中的代码，再次将占位符文本替换为你的 Service Bus 服务命名空间和密钥：

    var cf = new ChannelFactory<IProblemSolverChannel>(
        new NetTcpRelayBinding(), 
        new EndpointAddress(ServiceBusEnvironment.CreateServiceUri("sb", "**namespace**", "solver")));

    cf.Endpoint.Behaviors.Add(new TransportClientEndpointBehavior
                { TokenProvider = TokenProvider.CreateSharedSecretTokenProvider("owner","**key**") });
     
    using (var ch = cf.CreateChannel())
    {
        Console.WriteLine(ch.AddNumbers(4, 5));
    }

现在你可以编译客户端和服务，运行它们（首先运行服务），客户端将调用该服务并输出“9”。你可以在不同计算机上，甚至跨网络运行客户端和服务器，通信仍将进行。客户端代码还可以在云中或在本地运行。

**如何在 App.config 文件中配置客户端**

你还可以使用 App.config 文件配置客户端。使用这种方法的客户端代码如下所示：

    var cf = new ChannelFactory<IProblemSolverChannel>("solver");
    using (var ch = cf.CreateChannel())
    {
        Console.WriteLine(ch.AddNumbers(4, 5));
    }

终结点定义将移到 App.config 文件中。以下代码段（与上面列出的代码相同）应该直接显示在 **system.serviceModel** 元素下。在此，与之前一样，你必须将占位符替换为你的 Service Bus 服务命名空间和密钥。

    <client>
        <endpoint name="solver" contract="Service.IProblemSolver"
                  binding="netTcpRelayBinding"
                  address="sb://**namespace**.servicebus.windows.net/solver"
                  behaviorConfiguration="sbTokenProvider"/>
    </client>
    <behaviors>
        <endpointBehaviors>
            <behavior name="sbTokenProvider">
                <transportClientEndpointBehavior>
                    <tokenProvider>
                        <sharedSecret issuerName="owner" issuerSecret="**key**" />
                    </tokenProvider>
                </transportClientEndpointBehavior>
            </behavior>
        </endpointBehaviors>
    </behaviors>

## <span class="short-header">后续步骤</span>后续步骤

此时，你已了解 Service Bus **中继**服务的基础知识，请访问以下链接以了解更多信息。

-   构建服务：[为 Service Bus 构建服务][为 Service Bus 构建服务]。
-   构建客户端：[构建 Service Bus 客户端应用程序][构建 Service Bus 客户端应用程序]。
-   Service Bus 示例：从 [Azure 示例][Azure 示例]下载。

  [后续步骤]: #next_steps
  [create-account-note]: ../includes/create-account-note.md
  [中继概念]: ./media/service-bus-dotnet-how-to-use-relay/sb-relay-01.png
  [Azure 管理门户]: http://manage.windowsazure.cn
  [0]: ./media/service-bus-dotnet-how-to-use-relay/sb-queues-13.png
  [1]: ./media/service-bus-dotnet-how-to-use-relay/sb-queues-04.png
  [2]: ./media/service-bus-dotnet-how-to-use-relay/getting-started-multi-tier-27.png
  [3]: ./media/service-bus-dotnet-how-to-use-relay/sb-queues-09.png
  [4]: ./media/service-bus-dotnet-how-to-use-relay/sb-queues-06.png
  [5]: ./media/service-bus-dotnet-how-to-use-relay/getting-started-multi-tier-13.png
  [为 Service Bus 构建服务]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee173564.aspx
  [构建 Service Bus 客户端应用程序]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee173543.aspx
  [Azure 示例]: http://code.msdn.microsoft.com/windowsazure
