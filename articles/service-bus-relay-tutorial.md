<properties 
   pageTitle="服务总线中继消息传送教程 | Microsoft Azure"
   description="使用服务总线中继消息传送继构建服务总线客户端应用程序。"
   services="service-bus"
   documentationCenter="na"
   authors="sethmanheim"
   manager="timlt"
   editor="tysonn" />
<tags 
   ms.service="service-bus"
   ms.date="01/26/2016"
   wacn.date="04/01/2016" />

# 服务总线中继消息传送教程

本教程介绍了如何使用服务总线“中继”功能，构建简单的服务总线客户端应用程序和服务。有关使用服务总线[中转消息传送](/documentation/articles/service-bus-messaging-overview/#Brokered-messaging)的相应教程，请参阅[服务总线中转消息传送 .NET 教程](/documentation/articles/service-bus-brokered-tutorial-dotnet/)。

通过此教程，你可以了解创建服务总线客户端和服务应用程序所需的步骤。正如其 WCF 对应项，服务是公开一个或多个终结点的构造，其中每个终结点都公开一个或多个服务操作。服务的终结点用于指定可在其中找到服务的地址、包含客户端必须与服务进行通信的信息的绑定，以及定义服务向其客户端提供的功能的协定。WCF 和服务总线服务之间的主要区别在于：终结点在云中公开，而不是在本地计算机中公开。

完成本教程中的一系列主题后，你将具有一项正在运行的服务和可以调用服务操作的客户端。第一个主题描述了如何设置帐户。接下来的步骤描述了如何定义使用协定的服务、如何实现服务，以及如何使用代码配置该服务。这些主题还描述了如何托管和运行该服务。创建的服务是自托管的，并且客户端和服务在同一台计算机上运行。你可以通过使用代码或配置文件配置服务。

最后三个步骤介绍如何创建客户端应用程序、如何配置客户端应用程序，以及如何创建和使用可以访问主机功能的客户端。

本部分中的所有主题均假定使用 Visual Studio 作为开发环境。

## 注册帐户

第一步是创建服务总线服务命名空间并获取共享访问签名 (SAS) 密钥。服务命名空间为每个通过服务总线公开的应用程序提供应用程序边界。服务命名空间与 SAS 密钥的组合为服务总线提供了一个用于验证应用程序访问权限的凭据。

1. 若要创建服务命名空间，请访问 [Azure 管理门户][]。单击左侧的“服务总线”，然后单击“创建”。为你的命名空间键入一个名称，然后单击复选标记。

	>[AZURE.NOTE] 无需针对客户端和服务应用程序使用相同的命名空间。

1. 在门户的主窗口中，单击在上一步中创建的命名空间的名称。

2. 单击“配置”以查看命名空间的默认共享访问策略。

3. 记下 **RootManageSharedAccessKey** 策略的主键，或将其复制到剪贴板上。你将在本教程的后面部分使用此值。

## 定义 WCF 服务协定以用于服务总线

服务协定用于指定服务支持的操作类型（方法或函数的 Web 服务术语）。约定通过定义 C++、C# 或 Visual Basic 接口来创建。接口中的每个方法都对应一个特定的服务操作。必须将 [ServiceContractAttribute](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.servicecontractattribute.aspx) 属性应用于每个接口，并且必须将 [OperationContractAttribute](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.operationcontractattribute.aspx) 属性应用于每个操作。如果具有 [ServiceContractAttribute](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.servicecontractattribute.aspx) 属性的接口中的方法没有 [OperationContractAttribute](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.operationcontractattribute.aspx) 属性，则该方法是不公开的。该过程后面的示例中提供了这些任务的代码。有关协定和服务的更多讨论，请参阅 WCF 文档中的[设计和实现服务](https://msdn.microsoft.com/zh-cn/library/ms729746.aspx)。

### 使用接口创建服务总线约定

1. 在“开始”菜单中右键单击 Visual Studio，以便以管理员身份启动该程序，然后选择“以管理员身份运行”。

2. 创建新的控制台应用程序项目。单击“文件”菜单并选择“新建”，然后单击“项目”。在“新建项目”对话框中，单击“Visual C#”（如果“Visual C#”未出现，则在“其他语言”下方查看）。单击“控制台应用程序”模板，并将其命名为 **EchoService**。使用默认“位置”。单击“确定”以创建该项目。

3. 在项目中添加对 `System.ServiceModel.dll` 的引用：在“解决方案资源管理器”中，右键单击项目文件夹下的“引用”文件夹，然后单击“添加引用”。在“添加引用”对话框中选择“.NET”选项卡并向下滚动，直到看到“System.ServiceModel”。选中后单击“确定”。

4. 在解决方案资源管理中，双击 Program.cs 文件以在编辑器中将其打开。

5. 为 System.ServiceModel 命名空间添加 `using` 语句。

	```
	using System.ServiceModel;
	```

	[System.ServiceModel](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.aspx) 是可以以编程方式访问 WCF 基本功能的命名空间。服务总线使用 WCF 的许多对象和属性来定义服务约定。

6. 将命名空间的默认名称 **EchoService** 更改为 **Microsoft.ServiceBus.Samples**。

	>[AZURE.IMPORTANT] 本教程使用 C# 命名空间 **Microsoft.ServiceBus.Samples**，它是协定管理类型的命名空间，此类型用于[配置 WCF 客户端中](#configure-the-wcf-client)步骤中的配置文件。在构建此示例时，你可以指定任何想要的命名空间，当你在配置文件中修改了协定以及相应服务的命名空间后，本教程才会生效。在 App.config 文件中指定的命名空间必须与在 C# 文件中指定的命名空间相同。

7. 直接在完成 `Microsoft.ServiceBus.Samples` 命名空间声明后，在命名空间内定义一个名为 `IEchoContract` 的新接口，然后将 `ServiceContractAttribute` 属性应用于该接口，其值为 **http://samples.microsoft.com/ServiceModel/Relay/**。该命名空间值不同于你在整个代码范围内使用的命名空间。相反，该命名空间值将用作此协定的唯一标识符。显式指定命名空间可防止将默认的命名空间值添加到约定名称中。

	```
	[ServiceContract(Name = "IEchoContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
	publicinterface IEchoContract
	{
	}
	```

	>[AZURE.NOTE]通常情况下，服务协定命名空间包含一个包括版本信息的命名方案。服务协定命名空间中包括的版本信息可以使服务通过将新服务协定定义为新命名空间并将其公开到新的终结点上，来隔离重大更改。以这种方式，客户端可以继续使用旧的服务协定，而无需进行更新。版本信息可能包含日期或内部版本号。有关详细信息，请参阅[服务版本控制](http://go.microsoft.com/fwlink/?LinkID=180498)。鉴于此教程的目的，服务协定命名空间的命名方案不包含版本信息。

1. 在 `IEchoContract` 接口中，为 `IEchoContract` 约定在接口中公开的单个操作声明一个方法，然后将 `OperationContractAttribute` 属性应用到你希望将其作为公共服务总线约定的一部分进行公开的方法中。

	```
	[OperationContract]
	string Echo(string text);
	```

9. 在该协定外，声明从 `IEchoChannel` 中继承，并可传承到 `IClientChannel` 接口的通道，如下所示：

	```
    [ServiceContract(Name = "IEchoContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    publicinterface IEchoContract
    {
        [OperationContract]
        String Echo(string text);
    }

    publicinterface IEchoChannel : IEchoContract, IClientChannel { }
	```

	通道是主机和客户端用来互相传递信息的 WCF 对象。随后，你将针对通道编写代码，以在两个应用程序之间回显信息。

1. 在“生成”菜单中，单击“生成解决方案”或按 F6 以确认到目前为止操作的准确性。

### 示例

以下代码显示了一个用于定义服务总线协定的基本接口。

```
using System;
using System.ServiceModel;

namespace Microsoft.ServiceBus.Samples
{
    [ServiceContract(Name = "IEchoContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
        [OperationContract]
        String Echo(string text);
    }

    public interface IEchoChannel : IEchoContract, IClientChannel { }

    class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

既然已创建接口，你可以实现该接口。

## 实现 WCF 协定以使用服务总线

创建服务总线服务首先需要你创建使用接口定义的协定。有关创建接口的详细信息，请参阅上一步。下一步是实现该接口。此步骤包括创建名为 `EchoService` 的类，用于实现用户定义的 `IEchoContract` 接口。实现接口后，即可使用 App.config 配置文件配置接口。该配置文件包含应用程序所需的信息，如服务的名称、约定的名称，以及用来与服务总线通信的协议类型。该过程后面的示例中提供了这些任务所用的代码。有关如何实现服务协定的更多常规讨论，请参阅 Windows Communication Foundation (WCF) 文档中的[实现服务协定](https://msdn.microsoft.com/zh-cn/library/ms733764.aspx)。

1. 在 `IEchoContract` 接口定义的正下方创建名为 `EchoService` 的新类。`EchoService` 类实现 `IEchoContract` 接口。 

	```
	class EchoService : IEchoContract
	{
	}
	```
	
	与其他接口实现类似，你可以在另一个文件中实现定义。但是，在本教程中，实现所在的文件与接口定义和 `Main` 方法所在的文件相同。

2. 应用指示服务名称和命名空间属性的 [ServiceBehaviorAttribute](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.servicebehaviorattribute.aspx) 属性.

	```[ServiceBehavior(Name = "EchoService", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
	class EchoService : IEchoContract
	{
	}
	```

3. 在 `EchoService` 类中，实现 `IEchoContract` 接口中定义的 `Echo` 方法。

	```
	public string Echo(string text)
	{
    	Console.WriteLine("Echoing: {0}", text);
    	return text;
	}
	```

4. 单击“生成”，然后单击“生成解决方案”以确认工作的准确性。

### 定义服务主机的配置

1. 配置文件非常类似于 WCF 配置文件。该配置文件包括服务名称、终结点（即，服务总线公开的、让客户端和主机相互通信的位置）和绑定（用于通信的协议类型）。此处的主要差别在于，配置的服务终结点是指 [netTcpRelayBinding](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.nettcprelaybinding.aspx)，它不是 .NET Framework 的一部分。[NetTcpRelayBinding](https://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.nettcprelaybinding.aspx) 是通过服务总线定义的绑定之一。

2. 在“解决方案资源管理器”中单击 App.config 文件，该文件当前包含以下 XML 元素：

	```
	<?xmlversion="1.0"encoding="utf-8"?>
	<configuration>
	</configuration>
	```

3. 在 App.config 文件中添加一个 `<system.serviceModel>` XML 元素。该元素是一个 WCF 元素，用于定义一个或多个服务。在本示例中，它用于定义服务名称和终结点。

	```
	<?xmlversion="1.0"encoding="utf-8"?>
	<configuration>
	  <system.serviceModel>
  
	  </system.serviceModel>
	</configuration>
	```

4. 在 `<system.serviceModel>` 标记中，添加 `<services>` 元素。可以在单个配置文件中定义多个服务总线应用程序。但是，本教程只定义一个。
 
		```
		<?xmlversion="1.0"encoding="utf-8"?>
		<configuration>
		  <system.serviceModel>
		    <services>
	
		    </services>
		  </system.serviceModel>
		</configuration>
		```

5. 在 `<services>` 元素中，添加 `<service>` 元素来定义服务名称。

	```
	<servicename="Microsoft.ServiceBus.Samples.EchoService">
	</service>
	```

6. 在 `<service>` 元素中，定义终结点协定的位置，以及终结点绑定的类型。

	```
	<endpointcontract="Microsoft.ServiceBus.Samples.IEchoContract"binding="netTcpRelayBinding"/>
	```

	终结点用于定义客户端将在何处查找主机应用程序。接下来，本教程将使用此步骤来创建一个通过服务总线完全公开主机的 URI。绑定声明我们正在将 TCP 用作协议，以与服务总线进行通信。


7. 直接在 `<services>` 元素的后面，添加以下绑定扩展。
 
	```
	<extensions>
	  <bindingExtensions>
	    <addname="netTcpRelayBinding"type="Microsoft.ServiceBus.Configuration.NetTcpRelayBindingCollectionElement, Microsoft.ServiceBus, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
	  </bindingExtensions>
	</extensions>
	```

8. 在“生成”菜单中，单击“生成解决方案”以确认工作的准确性。

### 示例

下面的代码显示服务协定的实现。

```
[ServiceBehavior(Name = "EchoService", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]

    class EchoService : IEchoContract
    {
        public string Echo(string text)
        {
            Console.WriteLine("Echoing: {0}", text);
            return text;
        }
    }
```

以下代码显示了与该服务主机关联的 App.config 文件的基本格式。

```
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <system.serviceModel>
    <services>
      <service name="Microsoft.ServiceBus.Samples.EchoService">
        <endpoint contract="Microsoft.ServiceBus.Samples.IEchoContract" binding="netTcpRelayBinding" />
      </service>
    </services>
    <extensions>
      <bindingExtensions>
        <add name="netTcpRelayBinding" type="Microsoft.ServiceBus.Configuration.NetTcpRelayBindingCollectionElement, Microsoft.ServiceBus, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" />
      </bindingExtensions>
    </extensions>
  </system.serviceModel>
</configuration>
```

## 托管并运行基本 Web 服务以向服务总线注册

此步骤介绍如何运行基本服务总线服务。

### 创建服务总线凭据

1. 安装[服务总线 NuGet 包](https://www.nuget.org/packages/WindowsAzure.ServiceBus)

1. 在 `Main()` 中，创建两个变量，将命名空间和从控制台窗口中读取的 SAS 密钥存储在其中。

	```
	Console.Write("Your Service Namespace: ");
	string serviceNamespace = Console.ReadLine();
	Console.Write("Your SAS key: ");
	string sasKey = Console.ReadLine();
	```

	随后将使用 SAS 密钥来访问你的服务总线项目。命名空间作为参数传递给 `CreateServiceUri` 以创建服务 URI。

4. 使用 [TransportClientEndpointBehavior](https://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.transportclientendpointbehavior.aspx) 对象声明你将使用 SAS 密钥作为凭据类型。在最后一步中添加的代码后直接添加以下代码。

	```
	TransportClientEndpointBehavior sasCredential = new TransportClientEndpointBehavior();
	sasCredential.TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider("RootManageSharedAccessKey", sasKey);
	```

### 为服务创建基本地址

1. 在上一个步骤添加的代码后，为服务的基址创建 `Uri` 实例。此 URI 指定服务总线方案、命名空间，以及服务接口的路径。

	```
	Uri address = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, "EchoService");
	```

	"sb" 是服务总线方案的缩写，并指示我们正在使用 TCP 作为协议。先前当 [NetTcpRelayBinding](https://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.nettcprelaybinding.aspx) 被指定为绑定时，在配置文件中也指示了这一点。
	
	对于本教程中，URI 是 `sb://putServiceNamespaceHere.chinacloudapi.cn/EchoService`。

### 创建并配置服务主机

1. 将连接模式设置为 `AutoDetect`

	```
	ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.AutoDetect;
	```

	连接模式描述服务用于与服务总线进行通信的协议；连接模式为 HTTP 或 TCP。使用默认设置 `AutoDetect`，服务尝试通过 TCP（如果可用）或 HTTP（如果 TCP 不可用）连接到服务总线。请注意这与服务为客户端通信指定的协议不同。为客户端通信指定的协议由所使用的绑定所决定。例如，服务可以使用指定其终结点（公开在服务总线上）的 [BasicHttpRelayBinding](https://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.basichttprelaybinding.aspx) 绑定通过 HTTP 与客户端通信。同一个服务可以指定 **ConnectivityMode.AutoDetect**，以便服务通过 TCP 与服务总线通信。

2. 使用之前在本部分中创建的 URI 创建服务主机。

	```
	ServiceHost host = new ServiceHost(typeof(EchoService), address);
	```

	该服务主机是可实例化服务的 WCF 对象。在这里你将传递想要创建的服务类型（`EchoService` 类型），以及想要公开服务的地址。

3. 在 Program.cs 文件的顶部，添加对 [System.ServiceModel.Description](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.description.aspx) 和 [Microsoft.ServiceBus.Description](https://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.description.aspx) 的引用。

	```
	using System.ServiceModel.Description;
	using Microsoft.ServiceBus.Description;
	```

4. 返回到 `Main()`，配置终结点以启用公开访问。

	```
	IEndpointBehavior serviceRegistrySettings = new ServiceRegistrySettings(DiscoveryType.Public);
	```

	此步骤告知服务总线可以通过检查项目的服务总线 ATOM 源公开找到你的应用程序。如果你将 **DiscoveryType** 设置为 **private**，客户端将仍将能够访问该服务。但是，当搜索服务总线命名空间时不会显示该服务。相反，客户端必须事先知道终结点路径。

5. 将服务凭据应用到 App.config 文件中定义的服务终结点：

	```
	foreach (ServiceEndpoint endpoint in host.Description.Endpoints)
	{
	    endpoint.Behaviors.Add(serviceRegistrySettings);
	    endpoint.Behaviors.Add(sasCredential);
	}
	```

	如在上一步中所述，你可能已经在配置文件中声明多个服务和终结点。如果你已配置，此代码将遍历配置文件并且搜索可能应用了凭据的每个终结点。但是，对于本教程中，配置文件只有一个终结点。

### 打开服务主机

1. 打开服务。

	```
	host.Open();
	```

2. 通知用户该服务正在运行，并说明如何关闭服务。

	```
	Console.WriteLine("Service address: " + address);
	Console.WriteLine("Press [Enter] to exit");
	Console.ReadLine();
	```

3. 完成后，关闭服务主机。

	```
	host.Close();
	```

4. 按“F6”生成项目。

### 示例

下例包括本教程中前面步骤中使用的服务协定和实现，并将服务托管在控制台应用程序中。将以下编译到名为 EchoService.exe 的可执行文件。

```
using System;
using System.ServiceModel;
using System.ServiceModel.Description;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Description;

namespace Microsoft.ServiceBus.Samples
{
    [ServiceContract(Name = "IEchoContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]

    public interface IEchoContract
    {
        [OperationContract]
        String Echo(string text);
    }

    public interface IEchoChannel : IEchoContract, IClientChannel { };

    [ServiceBehavior(Name = "EchoService", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]

    class EchoService : IEchoContract
    {
        public string Echo(string text)
        {
            Console.WriteLine("Echoing: {0}", text);
            return text;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.AutoDetect;         
          
            Console.Write("Your Service Namespace: ");
            string serviceNamespace = Console.ReadLine();
            Console.Write("Your SAS key: ");
            string sasKey = Console.ReadLine();

           // Create the credentials object for the endpoint.
            TransportClientEndpointBehavior sasCredential = new TransportClientEndpointBehavior();
            sasCredential.TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider("RootManageSharedAccessKey", sasKey);

            // Create the service URI based on the service namespace.
            Uri address = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, "EchoService");

            // Create the service host reading the configuration.
            ServiceHost host = new ServiceHost(typeof(EchoService), address);

            // Create the ServiceRegistrySettings behavior for the endpoint.
            IEndpointBehavior serviceRegistrySettings = new ServiceRegistrySettings(DiscoveryType.Public);

            // Add the Service Bus credentials to all endpoints specified in configuration.
            foreach (ServiceEndpoint endpoint in host.Description.Endpoints)
            {
                endpoint.Behaviors.Add(serviceRegistrySettings);
                endpoint.Behaviors.Add(sasCredential);
            }
            
            // Open the service.
            host.Open();

            Console.WriteLine("Service address: " + address);
            Console.WriteLine("Press [Enter] to exit");
            Console.ReadLine();

            // Close the service.
            host.Close();
        }
    }
}
```

## 创建服务协定的 WCF 客户端

下一步将创建基本服务总线客户端，并定义将在后续步骤中实现的服务协定。请注意，许多这样的步骤类似于用于创建服务的步骤：定义协定、编辑 App.config 文件、使用凭据来连接到服务总线等。该过程后面的示例中提供了这些任务所用的代码。

1. 通过执行以下操作为客户端通在当前 Visual Studio 解决方案中创建一个新的项目：
	1. 在解决方案资源管理器中，在包含该服务的同一解决方案中，右键单击当前解决方案（不是项目），然后单击“添加”。然后单击“新建项目”。
	2. 在“添加新项目”对话框中，单击“Visual C#”（如果未显示“Visual C#”，则在“其他语言”下方查看），再选择“控制台应用程序”模板，并将其命名为“EchoClient”。
	3. 单击“确定”。<br />

2. 在解决方案资源管理中，双击“EchoClient”项目中的 Program.cs 文件以在编辑器中将其打开。

3. 将命名空间名称从其默认名称 `EchoClient` 更改为 `Microsoft.ServiceBus.Samples`。

4. 在项目中添加对 System.ServiceModel.dll 的引用：
	1. 在解决方案资源管理器中的“EchoClient”项目下，右键单击“引用”。然后单击“添加引用”。
	2. 因为在本教程的第一步你已添加对此程序集的引用，现在此引用列在“最近”选项卡中。单击“最近”，然后从列表中选择 **System.ServiceModel.dll**。然后，单击“确定”。如果你在“最近”选项卡上没有看到“System.ServiceModel.dll”，单击“浏览”选项卡，然后转到 **C:\\Windows\\Microsoft.NET\\Framework\\v3.0\\Windows Communication Foundation**。然后从此处选择程序集。<br />

5. 为 Program.cs 文件中的 [System.ServiceModel](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.aspx) 命名空间添加 `using` 语句。

	```
	using System.ServiceModel;
	```

6. 重复前面的步骤，将对 Microsoft.ServiceBus.dll 的引用和 [Microsoft.ServiceBus](https://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.aspx) 命名空间添加到你的项目中。

7. 如下面的示例中所示，将服务协定定义添加到命名空间。请注意，此定义等同于“服务”项目中所使用的定义。应将此代码添加到 `Microsoft.ServiceBus.Samples` 命名空间的顶部。

	```
	[ServiceContract(Name = "IEchoContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
	publicinterface IEchoContract
	{
	    [OperationContract]
	    string Echo(string text);
	}

	publicinterface IEchoChannel : IEchoContract, IClientChannel { }
	```

8. 按“F6”生成客户端。

### 示例

下面的代码显示了 EchoClient 项目中的 Program.cs 文件的当前状态。

```
using System;
using Microsoft.ServiceBus;
using System.ServiceModel;

namespace Microsoft.ServiceBus.Samples
{

[ServiceContract(Name = "IEchoContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
        [OperationContract]
        string Echo(string text);
    }

    public interface IEchoChannel : IEchoContract, IClientChannel { }


    class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

## 配置 WCF 客户端

在此步骤中，你可以为之前在本教程中创建的访问服务的基本客户端应用程序创建 App.config 文件。此 App.config 文件用于定义终结点的协定、绑定和名称。该过程后面的示例中提供了这些任务所用的代码。

1. 在解决方案资源管理器中的客户端项目中，双击“App.config”以打开文件，该文件当前包含以下 XML 元素：

	```
	<?xmlversion="1.0"?>
	<configuration>
	  <startup>
	    <supportedRuntimeversion="v4.0"sku=".NETFramework,Version=v4.0"/>
	  </startup>
	</configuration>
	```

2. 在 App.config 文件中为 `system.serviceModel` 添加一个 XML 元素。

	```
	<?xmlversion="1.0"encoding="utf-8"?>
	<configuration>
	  <system.serviceModel>
	
	  </system.serviceModel>
	</configuration>
	```
	
	此元素声明你的应用程序使用 WCF 样式终结点。如前面所述，服务总线应用程序的大部分配置都与 WCF 应用程序的配置相同；主要的区别在于配置文件所指向的位置。

3. 在 system.serviceModel 元素中，添加 `<client>` 元素。

	```
	<?xmlversion="1.0"encoding="utf-8"?>
	<configuration>
	  <system.serviceModel>
	    <client>
	    </client>
	  </system.serviceModel>
	</configuration>
	```

	此步骤声明你正在定义一个 WCF 样式的客户端应用程序。

4. 在 `client` 元素中，定义终结点的名称、协定和绑定类型。

	```
	<endpointname="RelayEndpoint"
					contract="Microsoft.ServiceBus.Samples.IEchoContract"
					binding="netTcpRelayBinding"/>
	```

	此步骤中定义终结点的名称、服务中定义的协定，以及客户端应用程序使用 TCP 与服务总线进行通信的事实。终结点名称在下一步中用于将此终结点配置与服务 URI 链接。

5. 直接在 <client> 元素的后面，添加以下绑定扩展。
 
	```
	<extensions>
	  <bindingExtensions>
	    <addname="netTcpRelayBinding"type="Microsoft.ServiceBus.Configuration.NetTcpRelayBindingCollectionElement, Microsoft.ServiceBus, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
	  </bindingExtensions>
	</extensions>
	```

6. 单击“文件”，然后单击“全部保存”。

## 示例

下面的代码显示了 Echo 客户端的 App.config 文件。

```
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <system.serviceModel>
    <client>
      <endpoint name="RelayEndpoint"
                      contract="Microsoft.ServiceBus.Samples.IEchoContract"
                      binding="netTcpRelayBinding"/>
    </client>
    <extensions>
      <bindingExtensions>
        <add name="netTcpRelayBinding" type="Microsoft.ServiceBus.Configuration.NetTcpRelayBindingCollectionElement, Microsoft.ServiceBus, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" />
      </bindingExtensions>
    </extensions>
  </system.serviceModel>
</configuration>
```

## 实现 WCF 客户端以调用服务总线

在此步骤中，你实现了可访问之前在本教程中创建的服务的基本客户端应用程序。与服务相似，该客户端访问服务总线的操作步骤大多数都相同：

1. 设置连接模式。

2. 创建用于定位主机服务的 URI。

3. 定义安全凭据。

4. 将凭据应用到连接。

5. 打开连接。

6. 执行应用程序特定的任务。

7. 关闭连接。

但是，主要的区别之一在于，客户端应用程序使用通道连接到服务总线，而服务则使用一个对 **ServiceHost** 的调用。该过程后面的示例中提供了这些任务所用的代码。

### 实现客户端应用程序

1. 将连接模式设置为 **AutoDetect**。添加客户端应用程序的 `Main()` 方法中的以下代码。

	```
	ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.AutoDetect;
	```

2. 定义变量以保存用于服务命名空间的值，以及从控制台读取的 SAS 密钥。

	```
	Console.Write("Your Service Namespace: ");
	string serviceNamespace = Console.ReadLine();
	Console.Write("Your SAS Key: ");
	string sasKey = Console.ReadLine();
	```

3. 创建用于定义服务总线项目中托管位置的 URI。

	```
	Uri serviceUri = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, "EchoService");
	```

4. 创建服务命名空间终结点的凭据对象。

	```
	TransportClientEndpointBehavior sasCredential = new TransportClientEndpointBehavior();
	sasCredential.TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider("RootManageSharedAccessKey", sasKey);
	```

5. 创建加载在 App.config 文件中所述的配置的通道工厂。

	```
	ChannelFactory<IEchoChannel> channelFactory = new ChannelFactory<IEchoChannel>("RelayEndpoint", new EndpointAddress(serviceUri));
	```

	通道工厂是创建通道（通过该通道，服务和客户端可以进行通信）的一个 WCF 对象。

6. 应用服务总线凭据

	```
	channelFactory.Endpoint.Behaviors.Add(sasCredential);
	```

7. 创建并打开服务通道。

	```
	IEchoChannel channel = channelFactory.CreateChannel();
	channel.Open();
	```

8. 编写用于回显的基本用户界面和功能。

	```
	Console.WriteLine("Enter text to echo (or [Enter] to exit):");
	string input = Console.ReadLine();
	while (input != String.Empty)
	{
	    try
	    {
	        Console.WriteLine("Server echoed: {0}", channel.Echo(input));
	    }
	    catch (Exception e)
	    {
	        Console.WriteLine("Error: " + e.Message);
	    }
	    input = Console.ReadLine();
	}
	```

	请注意，代码使用通道对象的实例作为服务代理。

9. 关闭通道，然后关闭工厂。

	```
	channel.Close();
	channelFactory.Close();
	```

## 启动客户端应用程序

1. 按“F6”生成解决方案。这将生成客户端项目和你在本教程的上一步创建的服务项目，并为每一个项目创建一个可执行文件。

2. 在运行客户端应用程序之前，请确保服务应用程序正在运行。

	现在，你应该具有 Echo 服务应用程序的名为 EchoService.exe 的可执行文件，该文件位于 \\bin\\Debug\\EchoService.exe（针对调试配置）或 \\bin\\Release\\EchoService.exe（针对发布配置）下的服务项目文件夹中。双击此文件以启动服务应用程序。

3. 将打开一个控制台窗口并提示你输入命名空间。在此控制台窗口中，输入服务命名空间并按“Enter”。

4. 接下来，将提示你提供 SAS 密钥。输入 SAS 密钥并按“ENTER”。

	以下是来自控制台窗口的示例输出。请注意，此处提供的值仅限于示例目的。

	`Your Service Namespace: myNamespace`

	`Your SAS Key: <SAS key value>`

	启动服务应用程序，并将其正在侦听的地址打印到控制台窗口中，如下面的示例中所示。

    `Service address: sb://mynamespace.servicebus.chinacloudapi.cn/EchoService/`

    `Press [Enter] to exit`
    
5. 运行客户端应用程序。你应该具有 Echo 客户端应用程序的名为 EchoClient.exe 的可执行文件，该文件位于 .\\bin\\Debug\\EchoClient.exe（针对调试配置）或 .\\bin\\Release\\EchoClient.exe（针对发布配置）下的客户端项目中。双击此文件以启动客户端应用程序。

6. 将打开控制台窗口并提示你输入之前输入的用于服务应用程序的相同信息。请按照前面的步骤，在服务命名空间、颁发者名称和颁发者密钥中输入相同的客户端应用程序值。

7. 输入这些值后，客户端将打开服务通道并提示你输入如以下控制台输出示例中所示的某些文本。

	`Enter text to echo (or [Enter] to exit):`

	输入将发送到服务应用程序的某些文本，并按“Enter”。此文本通过 Echo 服务操作发送到服务并显示在服务控制台窗口中，如下面的示例输出所示。

	`Echoing: My sample text`

	客户端应用程序接收 `Echo` 操作的返回值（此为原始文本），并将其打印到控制台窗口。以下是来自客户端控制台窗口的示例输出。

	`Server echoed: My sample text`

8. 你可以继续以这种方式将来自客户端的短信发送至服务。完成后，在客户端和服务控制台窗口中按 Enter 以结束这两个应用程序。

## 示例

下面的示例演示了如何创建客户端应用程序、如何调用服务操作以及如何在完成操作调用后关闭客户端。

```
using System;
using Microsoft.ServiceBus;
using System.ServiceModel;

namespace Microsoft.ServiceBus.Samples
{
    [ServiceContract(Name = "IEchoContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IEchoContract
    {
        [OperationContract]
        String Echo(string text);
    }

    public interface IEchoChannel : IEchoContract, IClientChannel { }

    class Program
    {
        static void Main(string[] args)
        {
            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.AutoDetect;

            
            Console.Write("Your Service Namespace: ");
            string serviceNamespace = Console.ReadLine();
            Console.Write("Your SAS Key: ");
            string sasKey = Console.ReadLine();



            Uri serviceUri = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, "EchoService");

            TransportClientEndpointBehavior sasCredential = new TransportClientEndpointBehavior();
            sasCredential.TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider("RootManageSharedAccessKey", sasKey);

            ChannelFactory<IEchoChannel> channelFactory = new ChannelFactory<IEchoChannel>("RelayEndpoint", new EndpointAddress(serviceUri));

            channelFactory.Endpoint.Behaviors.Add(sasCredential);

            IEchoChannel channel = channelFactory.CreateChannel();
            channel.Open();

            Console.WriteLine("Enter text to echo (or [Enter] to exit):");
            string input = Console.ReadLine();
            while (input != String.Empty)
            {
                try
                {
                    Console.WriteLine("Server echoed: {0}", channel.Echo(input));
                }
                catch (Exception e)
                {
                    Console.WriteLine("Error: " + e.Message);
                }
                input = Console.ReadLine();
            }

            channel.Close();
            channelFactory.Close();

        }
    }
}
```

请确保在启动客户端之前服务正在运行。

## 后续步骤

本教程介绍了如何使用服务总线“中继”功能，构建服务总线客户端应用程序和服务。有关使用服务总线[中转消息传送](/documentation/articles/service-bus-messaging-overview/#Brokered-messaging)的类似教程，请参阅[服务总线中转消息传送 .NET 教程](/documentation/articles/service-bus-brokered-tutorial-dotnet/)。

若要了解有关服务总线的详细信息，请参阅以下主题。

- [服务总线消息传送概述](/documentation/articles/service-bus-messaging-overview/)
- [服务总线基础知识](/documentation/articles/service-bus-fundamentals-hybrid-solutions/)
- [服务总线体系结构](/documentation/articles/service-bus-architecture/)
[Azure 管理门户]: http://manage.windowsazure.cn

<!---HONumber=Mooncake_0104_2016-->