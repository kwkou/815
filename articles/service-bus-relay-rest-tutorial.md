<properties
   pageTitle="使用中继消息传送的服务总线 REST 教程 | Microsoft Azure"
   description="生成一个简单的服务总线中继主机应用程序来公开基于 REST 的接口。"
   services="service-bus"
   documentationCenter="na"
   authors="sethmanheim"
   manager="timlt"
   editor="" />
<tags 
   ms.service="service-bus"
   ms.date="10/14/2015"
   wacn.date="03/28/2016" />

# 服务总线 REST 教程

本教程介绍了如何生成简单的服务总线主机应用程序来公开基于 REST 的接口。REST 使 Web 客户端（例如 Web 浏览器）可通过 HTTP 请求访问服务总线 API。

本教程使用 Windows Communication Foundation (WCF) REST 编程模型在服务总线上构建 REST 服务。有关详细信息，请参阅 WCF 文档中的 [WCF REST 编程模型](https://msdn.microsoft.com/zh-cn/library/bb412169.aspx)和[设计和实现服务](https://msdn.microsoft.com/zh-cn/library/ms729746.aspx)。

## 步骤 1：注册 Azure 帐户

第一步是创建服务命名空间并获取共享访问签名 (SAS) 密钥。命名空间为每个通过服务总线公开的应用程序提供应用程序边界。创建服务命名空间时，系统将自动生成 SAS 密钥。服务命名空间与 SAS 密钥的组合为服务总线提供了一个用于验证应用程序访问权限的凭据。

### 创建服务命名空间并获取 SAS 密钥

1. 若要创建服务命名空间，请访问 [Azure 管理门户][]。单击左侧的“服务总线”，然后单击“创建”。为你的命名空间键入一个名称，然后单击复选标记。

2. 在门户的主窗口中，单击在上一步中创建的服务命名空间的名称。

3. 单击“配置”以查看服务命名空间的共享访问策略。

4. 记下 **RootManageSharedAccessKey** 策略的主键，或将其复制到剪贴板上。你将在本教程的后面部分使用此值。

## 步骤 2：定义基于 REST 的 WCF 服务约定以用于服务总线

与其他服务总线服务一样，创建 REST 样式的服务时，必须定义约定。约定指定主机支持的操作。服务操作可以看作是 Web 服务方法。约定通过定义 C++、C# 或 Visual Basic 接口来创建。接口中的每个方法都对应一个特定的服务操作。必须将 [ServiceContractAttribute](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.servicecontractattribute.aspx) 属性应用到每个接口，且必须将 [OperationContractAttribute](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.operationcontractattribute.aspx) 属性应用到每个操作。如果具有 [ServiceContractAttribute](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.servicecontractattribute.aspx) 的接口中的方法没有 [OperationContractAttribute](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.operationcontractattribute.aspx)，则该方法是不公开的。该过程后面的示例中显示了这些任务所用的代码。

基本服务总线协定和 REST 样式的协定的主要区别在于是否向 [OperationContractAttribute](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.operationcontractattribute.aspx) 添加一个属性：[WebGetAttribute](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.web.webgetattribute.aspx)。此属性允许你将接口中的方法映射到该接口另一侧的方法。在此示例中，我们使用 [WebGetAttribute](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.web.webgetattribute.aspx) 将一个方法链接到 HTTP GET。这将使服务总线可以准确地检索并解释发送到接口的命令。

### 使用接口创建服务总线约定

1. 以管理员身份打开 Visual Studio：在“开始”菜单中右键单击该程序，然后选择“以管理员身份运行”。

2. 创建新的控制台应用程序项目。单击“文件”菜单并选择“新建”，然后单击“项目”。在“新建项目”对话框中，单击“Visual C#”（如果未显示“Visual C#”，则在“其他语言”下方查看），再选择“控制台应用程序”模板，并将其命名为“ImageListener”。使用默认“位置”。单击“确定”以创建该项目。

3. 对于 C# 项目，Visual Studio 会创建 `Program.cs` 文件。此类包含一个空的 `Main()` 方法，需要此方法才能正确生成控制台应用程序项目。

4. 在项目中添加对 **System.ServiceModel.dll** 的引用：

	a.在“解决方案资源管理器”中，右键单击项目文件夹下的“引用”文件夹，然后单击“添加引用”。

	b.在“添加引用”对话框中选择“.NET”选项卡并向下滚动，直到看到“System.ServiceModel”。然后单击“确定”。

5. 重复上述步骤以添加对 **System.ServiceModel.Web.dll** 程序集的引用。

6. 为 **System.ServiceModel**、**System.ServiceModel.Channels**、**System.ServiceModel.Web** 和 **System.IO** 命名空间添加 `using` 语句。

	```c
  	using System.ServiceModel;
  	using System.ServiceModel.Channels;
  	using System.ServiceModel.Web;
  	using System.IO;
	```

	[System.ServiceModel](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.aspx) 是让你以通过编程方式访问 WCF 基本功能的命名空间。服务总线使用 WCF 的许多对象和属性来定义服务约定。你将在大多数服务总线中继应用程序中使用此命名空间。同样，[System.ServiceModel.Channels](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.channels.aspx) 可帮助定义通道，通道是用来与服务总线和客户端 Web 浏览器通信的对象。最后，[System.ServiceModel.Web](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.web.aspx) 包含的类型可用于创建基于 Web 的应用程序。

7. 从 Visual Studio 重命名默认为 **Microsoft.ServiceBus.Samples** 的程序的命名空间。

 	```
	namespace Microsoft.ServiceBus.Samples
	{
		...
	```

8. 在完成命名空间声明后，紧接着定义一个名为 **IImageContract** 的新接口，然后将 **ServiceContractAttribute** 属性应用于该接口，其值为 `http://samples.microsoft.com/ServiceModel/Relay/`。该命名空间值不同于你在整个代码范围内使用的命名空间。该命名空间值将用作此约定的唯一标识符，并应有版本控制信息。有关详细信息，请参阅[服务版本控制](http://go.microsoft.com/fwlink/?LinkID=180498)。显式指定命名空间可防止将默认的命名空间值添加到约定名称中。

	```
	[ServiceContract(Name = "ImageContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/RESTTutorial1")]
	public interface IImageContract
	{
	}
	```

9. 在 `IImageContract` 接口中，为 `IImageContract` 约定在接口中公开的单个操作声明一个方法，然后将 `OperationContractAttribute` 属性应用到你希望将其作为公共服务总线约定的一部分进行公开的方法中。

	```
	public interface IImageContract
	{
		[OperationContract]
		Stream GetImage();
	}
	```

10. 紧接在 **OperationContract** 属性后面，应用 **WebGet** 属性。

	```
	public interface IImageContract
	{
		[OperationContract, WebGet]
		Stream GetImage();
	}
	```

	这样做可以让服务总线将 HTTP GET 请求路由到 **GetImage**，并将 **GetImage** 的返回值转换为 HTTP GETRESPONSE 答复。稍后在本教程中，你将使用 Web 浏览器访问此方法，并将在浏览器中显示图像。

11. 直接在 `IImageContract` 定义的后面，声明从 `IImageContract` 和 `IClientChannel` 接口继承的通道。

	```
	[ServiceContract(Name = "IImageContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
	public interface IImageContract
	{
		[OperationContract, WebGet]
		Stream GetImage();
	}
  
	public interface IImageChannel : IImageContract, IClientChannel { }
	```

	通道是服务和客户端用来互相传递信息的 WCF 对象。稍后，你将在主机应用程序中创建通道。然后服务总线将使用该通道将浏览器的 HTTP GET 请求传递到你的 **GetImage** 实现。服务总线还使用该通道获取 **GetImage** 返回值并将其转换为客户端浏览器的 HTTP GETRESPONSE。

12. 在“生成”菜单中，单击“生成解决方案”以确认工作的准确性。

### 示例

以下代码显示了一个用于定义服务总线协定的基本接口。

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.ServiceModel;
using System.ServiceModel.Channels;
using System.ServiceModel.Web;
using System.IO;
	
namespace Microsoft.ServiceBus.Samples
{
	
    [ServiceContract(Name = "IImageContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IImageContract
    {
        [OperationContract, WebGet]
        Stream GetImage();
    }
	
    public interface IImageChannel : IImageContract, IClientChannel { }
	
    class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

## 步骤 3：实现基于 REST 的 WCF 服务约定以使用服务总线

创建 REST 样式的服务总线服务首先需要你创建使用接口定义的约定。下一步是实现该接口。此步骤包括创建名为 **ImageService** 的类，用于实现用户定义的 **IImageContract** 接口。实现约定后，即可使用 App.config 文件配置接口。该配置文件包含应用程序所需的信息，如服务的名称、约定的名称，以及用来与服务总线通信的协议类型。该过程后面的示例中提供了这些任务所用的代码。

与前面的步骤一样，实现 REST 样式的约定与实现基本服务总线约定之间的差别很小。

### 实现 REST 样式的服务总线约定

1. 在 **IImageContract** 接口定义的正下方创建名为 **ImageService** 的新类。**ImageService** 类实现 **IImageContract** 接口。。 

	```
	class ImageService : IImageContract
	{
	}
	```
	与其他接口实现类似，你可以在另一个文件中实现定义。但是，在本教程中，实现所在的文件与接口定义和 `Main()` 方法所在的文件相同。

2. 将 [ServiceBehaviorAttribute](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.servicebehaviorattribute.aspx) 属性应用到 **IImageService** 类，以指示该类是 WCF 协定的实现。

	```
	[ServiceBehavior(Name = "ImageService", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
	class ImageService : IImageContract
	{
	}
	```

	如前所述，此命名空间不是传统的命名空间，而是用于标识约定的 WCF 体系结构的一部分。有关详细信息，请参阅 WCF 文档中的[数据约定名称](https://msdn.microsoft.com/zh-cn/library/ms731045.aspx)主题。

3. 将一幅 .jpg 图像添加到项目中。

	这是服务在接收浏览器中显示的图片。右键单击你的项目并单击“添加”。然后单击“现有项”。使用“添加现有项”对话框浏览到相应的 .jpg，然后单击“添加”。
    
	添加文件时，请确保在“文件名:”旁的下拉列表中选择“所有文件(*.*)”。本教程的余下部分假定图像的名称为“image.jpg”。如果你的 .jpg 文件名不是这样，则必须重命名图像，或更改代码进行弥补。

4. 为了确保正在运行的服务可以找到该图像文件，请在“解决方案资源管理器”中右键单击该图像文件。在“属性”窗格中，将“复制到输出目录”设置为“如果较新则复制”。

5. 在项目中添加对 **System.Drawing.dll**、**System.Runtime.Serialization.dll** 和 **Microsoft.ServiceBus.dll** 程序集的引用，并添加以下关联的 `using` 语句。

	```
	using System.Drawing;
	using System.Drawing.Imaging;
	using Microsoft.ServiceBus;
	using Microsoft.ServiceBus.Web;
	```

6. 在 **ImageService** 类中定义以下构造函数，以便加载位图并准备将该位图发送到客户端浏览器。

	```
	class ImageService : IImageContract
	{
		const string imageFileName = "image.jpg";
  
		Image bitmap;
  
		public ImageService()
		{
			this.bitmap = Image.FromFile(imageFileName);
		}
	}
	```

7. 直接在上一代码后面，在 **ImageService** 类中添加以下 **GetImage** 方法，以返回包含该映像的 HTTP 消息。

	```
	public Stream GetImage()
	{
		MemoryStream stream = new MemoryStream();
		this.bitmap.Save(stream, ImageFormat.Jpeg);
  
		stream.Position = 0;
		WebOperationContext.Current.OutgoingResponse.ContentType = "image/jpeg";
  
		return stream;
	}
	```
  
	此实现使用 **MemoryStream** 检索映像并准备将其流式传输到浏览器。它将流位置设置为从零开始，将流内容声明为 jpeg，然后流式传输信息。

8. 在“生成”菜单中，单击“生成解决方案”。

### 定义配置以便在服务总线上运行 Web 服务

1. 右键单击“ImageListener”项目。然后依次单击“添加”、“新建项”。

2. 在“解决方案资源管理器”中双击“App.config”，该文件当前包含以下 XML 元素。

	```
	<?xml version="1.0" encoding="utf-8" ?>
	<configuration>
	</configuration>
	```
  
	该配置文件与 WCF 配置文件类似，包括服务名称、终结点（即，服务总线公开的、让客户端和主机相互通信的位置）和绑定（用于通信的协议类型）。此处的主要差别在于，配置的服务终结点是指 [WebHttpRelayBinding](https://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.webhttprelaybinding.aspx) 绑定，它不是 .NET Framework 的一部分。有关如何配置服务总线应用程序的详细信息，请参阅[ 配置 WCF 服务以注册到服务总线](https://msdn.microsoft.com/zh-cn/library/ee173579.aspx)。
  

3. 在 App.config 文件中添加一个 `<system.serviceModel>` XML 元素。该元素是一个 WCF 元素，用于定义一个或多个服务。在这里，它用于定义服务名称和终结点。

	```
	<?xml version="1.0" encoding="utf-8" ?>
	<configuration>
		<system.serviceModel>
      
		</system.serviceModel>
    
	</configuration>
	```

4. 在 `system.serviceModel` 元素中，添加包含以下内容的 `<bindings>` 元素。这样就定义了应用程序中使用的绑定。你可以定义多个绑定，但在本教程中，你只要定义一个绑定。

	```
	<bindings>
		<!-- Application Binding -->
		<webHttpRelayBinding>
			<binding name="default">
				<security relayClientAuthenticationType="None" />
			</binding>
		</webHttpRelayBinding>
	</bindings>
	```
  
	此步骤定义了一个服务总线 [WebHttpRelayBinding](https://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.webhttprelaybinding.aspx) 绑定，其中的 **relayClientAuthenticationType** 为 **None**。此设置表明使用此绑定的终结点将不需要客户端凭据。

5. 在 `<bindings>` 元素后面添加 `<services>` 元素。与绑定类似，可以在单个配置文件中定义多个服务。但是，在本教程中，你只要定义一个服务。

	```
	<services>
		<!-- Application Service -->
		<service name="Microsoft.ServiceBus.Samples.ImageService"
             behaviorConfiguration="default">
			<endpoint name="RelayEndpoint"
					contract="Microsoft.ServiceBus.Samples.IImageContract"
					binding="webHttpRelayBinding"
					bindingConfiguration="default"
					behaviorConfiguration="sbTokenProvider"
					address="" />
		</service>
	</services>
	```
  
	此步骤将配置一个服务，该服务使用前面定义的默认 **webHttpRelayBinding**。此外，它还使用下一步骤中定义的默认 **sbTokenProvider**。

6. 在 `<services>` 元素的后面，使用以下内容创建 `<behaviors>` 元素，并将 “SAS\_KEY” 替换为你在通过 [Azure 管理门户][] 执行步骤 1 时获取的*共享访问签名* (SAS) 密钥。
  
	```
	<behaviors>
		<endpointBehaviors>
			<behavior name="sbTokenProvider">
				<transportClientEndpointBehavior>
					<tokenProvider>
						<sharedAccessSignature keyName="RootManageSharedAccessKey" key="SAS_KEY" />
					</tokenProvider>
				</transportClientEndpointBehavior>
			</behavior>
			</endpointBehaviors>
			<serviceBehaviors>
				<behavior name="default">
					<serviceDebug httpHelpPageEnabled="false" httpsHelpPageEnabled="false" />
				</behavior>
			</serviceBehaviors>
	</behaviors>
	```
  
7. 在“生成”菜单中，单击“生成解决方案”以生成整个解决方案。

### 示例

以下代码演示了一个在服务总线上运行并使用 **WebHttpRelayBinding** 绑定的、基于 REST 的服务的约定和服务实现。

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.ServiceModel;
using System.ServiceModel.Channels;
using System.ServiceModel.Web;
using System.IO;
using System.Drawing;
using System.Drawing.Imaging;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Web;

namespace Microsoft.ServiceBus.Samples
{
    

    [ServiceContract(Name = "ImageContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IImageContract
    {
        [OperationContract, WebGet]
        Stream GetImage();
    }

    public interface IImageChannel : IImageContract, IClientChannel { }

    [ServiceBehavior(Name = "ImageService", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    class ImageService : IImageContract
    {
        const string imageFileName = "image.jpg";

        Image bitmap;

        public ImageService()
        {
            this.bitmap = Image.FromFile(imageFileName);
        }

        public Stream GetImage()
        {
            MemoryStream stream = new MemoryStream();
            this.bitmap.Save(stream, ImageFormat.Jpeg);

            stream.Position = 0;
            WebOperationContext.Current.OutgoingResponse.ContentType = "image/jpeg";

            return stream;
        }    
    }

    class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

以下示例显示了与该服务关联的 App.config 文件。

	```
	<?xml version="1.0" encoding="utf-8" ?>
	<configuration>
	  <system.serviceModel>
	    <bindings>
	      <!-- Application Binding -->
	      <webHttpRelayBinding>
	        <binding name="default">
	          <!-- Turn off client authentication so that client does not need to present credential through browser or fiddler -->
	          <security relayClientAuthenticationType="None" />
	        </binding>
	      </webHttpRelayBinding>
	    </bindings>
	
	    <services>
	      <!-- Application Service -->
	      <service name="Microsoft.ServiceBus.Samples.ImageService"
	               behaviorConfiguration="default">
	        <endpoint name="RelayEndpoint"
	                  contract="Microsoft.ServiceBus.Samples.IImageContract"
	                  binding="webHttpRelayBinding"
	                  bindingConfiguration="default"
	                  behaviorConfiguration="sbTokenProvider"
	                  address="" />
	      </service>
	    </services>
	
	    <behaviors>
	      <endpointBehaviors>
	        <behavior name="sbTokenProvider">
	          <transportClientEndpointBehavior>
	            <tokenProvider>
	              <sharedAccessSignature keyName="RootManageSharedAccessKey" key="SAS_KEY" />
	            </tokenProvider>
	          <transportClientEndpointBehavior>
	        </behavior>
	      </endpointBehaviors>
	      <serviceBehaviors>
	        <behavior name="default">
	          <serviceDebug httpHelpPageEnabled="false" httpsHelpPageEnabled="false" />
	        </behavior>
	      </serviceBehaviors>
	    </behaviors>
	
	  </system.serviceModel>
	</configuration>
	```

## 步骤 4：托管基于 REST 的 WCF 服务以使用服务总线

此步骤描述如何使用控制台应用程序在服务总线上运行 Web 服务。此步骤中编写的代码的完整列表将在过程后面的示例中提供。

### 为服务创建基本地址

1. 在 `Main()` 函数声明中，创建一个变量以存储服务总线项目的服务命名空间。

	```
	string serviceNamespace = "InsertServiceNamespaceHere";
	```
	服务总线使用服务命名空间的名称来创建唯一 URI。

2. 为基于服务命名空间的服务的基本地址创建 `Uri` 实例。

	```
	Uri address = ServiceBusEnvironment.CreateServiceUri("https", serviceNamespace, "Image");
	```

### 创建并配置 Web 服务主机

- 使用之前在本部分中创建的 URI 地址创建 Web 服务主机。

	```
	WebServiceHost host = new WebServiceHost(typeof(ImageService), address);
	```
	该服务主机是可实例化主机应用程序的 WCF 对象。本示例会将要创建的主机类型 (**ImageService**)，以及要公开主机应用程序的地址传递给它。

### 运行 Web 服务主机

1. 打开服务。

	```
	host.Open();
	```
	服务现在正在运行。

2. 显示表明服务正在运行以及如何停止服务的消息。

	```
	Console.WriteLine("Copy the following address into a browser to see the image: ");
	Console.WriteLine(address + "GetImage");
	Console.WriteLine();
	Console.WriteLine("Press [Enter] to exit");
	Console.ReadLine();
	```

3. 完成后，关闭服务主机。

	```c
	host.Close();
	```

## 示例

以下示例包括本教程中前面步骤中使用的服务约定和实现，并将服务托管在控制台应用程序中。将以下代码编译到名为 ImageListener.exe 的可执行文件中。

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.ServiceModel;
using System.ServiceModel.Channels;
using System.ServiceModel.Web;
using System.IO;
using System.Drawing;
using System.Drawing.Imaging;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Web;

namespace Microsoft.ServiceBus.Samples
{
    
    [ServiceContract(Name = "ImageContract", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    public interface IImageContract
    {
        [OperationContract, WebGet]
        Stream GetImage();
    }

    public interface IImageChannel : IImageContract, IClientChannel { }

    [ServiceBehavior(Name = "ImageService", Namespace = "http://samples.microsoft.com/ServiceModel/Relay/")]
    class ImageService : IImageContract
    {
        const string imageFileName = "image.jpg";

        Image bitmap;

        public ImageService()
        {
            this.bitmap = Image.FromFile(imageFileName);
        }

        public Stream GetImage()
        {
            MemoryStream stream = new MemoryStream();
            this.bitmap.Save(stream, ImageFormat.Jpeg);

            stream.Position = 0;
            WebOperationContext.Current.OutgoingResponse.ContentType = "image/jpeg";

            return stream;
        }    
    }

    class Program
    {
        static void Main(string[] args)
        {
            string serviceNamespace = "InsertServiceNamespaceHere";
            Uri address = ServiceBusEnvironment.CreateServiceUri("https", serviceNamespace, "Image");

            WebServiceHost host = new WebServiceHost(typeof(ImageService), address);
            host.Open();

            Console.WriteLine("Copy the following address into a browser to see the image: ");
            Console.WriteLine(address + "GetImage");
            Console.WriteLine();
            Console.WriteLine("Press [Enter] to exit");
            Console.ReadLine();

            host.Close();
        }
    }
}
```

### 编译代码

生成解决方案之后，请执行以下代码来运行应用程序：

1. 从命令提示符运行服务 (ImageListener\\bin\\Debug\\ImageListener.exe)。

2. 将命令提示符中的地址复制并粘贴到浏览器中以查看图像。

## 后续步骤

在生成使用服务总线中继服务的应用程序后，请参阅以下文章了解有关中继消息传送的详细信息。

- [Azure 服务总线体系结构概述](/documentation/articles/service-bus-fundamentals-hybrid-solutions/#relays)

- [如何使用服务总线中继服务](/documentation/articles/service-bus-dotnet-how-to-use-relay/)
[Azure 管理门户]: http://manage.windowsazure.cn

<!---HONumber=Mooncake_0104_2016-->