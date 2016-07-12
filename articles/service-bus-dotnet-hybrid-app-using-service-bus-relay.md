<properties
	pageTitle="本地/云混合应用程序 (.NET) | Azure"
	description="了解如何使用 Azure 服务总线中继创建 .NET 本地/云混合应用程序。"
	services="service-bus"
	documentationCenter=".net"
	authors="sethmanheim"
	manager="timlt"
	editor=""/>

<tags
	ms.service="service-bus"
	ms.date="10/07/2015"
	wacn.date="05/23/2016"/>

# 使用 Azure 服务总线中继创建 .NET 本地/云混合应用程序

##介绍

使用 Visual Studio 2013 和免费的 Azure SDK for .NET，可以轻松地开发针对 Azure 的混合云应用程序。本文假设你之前未使用过 Azure。在不到 30 分钟的时间内，你就能让使用多个 Azure 资源的应用程序在云中启动并运行。

你将学习以下内容：

-   如何创建或修改现有 Web 服务以供 Web 解决方案使用。
-   如何使用 Azure 服务总线中继功能在 Azure 应用程序和托管于其他某处的 Web 服务之间共享数据。

[AZURE.INCLUDE [create-account-note](../includes/create-account-note.md)]

##服务总线中继功能将为混合解决方案带来哪些帮助

业务解决方案通常由为处理独特的新业务需求而编写的自定义代码和已有的解决方案和系统所提供的现有功能组成。

解决方案架构师开始使用云来轻松地处理缩放需求和降低运营成本。在此过程中，他们发现希望用作其解决方案的构建基块的现有服务资产位于企业防火墙内，无法通过云解决方案轻松访问。许多内部服务的构建或托管方式使得它们无法在企业网络边缘轻松公开。

服务总线中继的设计考虑到如何利用现有的 Windows Communication Foundation (WCF) Web 服务，使得位于企业外部的解决方案能够安全地访问这些服务，而无需对企业网络基础结构进行彻底的更改。虽然此类服务总线中继服务仍托管在现有环境中，但它们会将侦听传入会话和请求这一任务委托给云托管的服务总线。服务总线还会通过使用[共享访问签名](https://msdn.microsoft.com/library/dn170478.aspx) (SAS) 身份验证来保护这些服务，以阻止未经授权的访问。

## 解决方案应用场景

在本教程中，你将创建一个 ASP.NET MVC 网站，用于查看产品库存页上的产品列表。

![][0]

本教程假定你的产品信息位于现有的本地系统中，而且你使用服务总线中继来访问该系统。这是由在简单的控制台应用程序中运行的 Web 服务模拟的，并由一系列内存中产品提供支持。你将能够在你自己的计算机上运行此控制台应用程序并将 Web 角色部署到 Azure 中。通过此操作，你将看到在 Azure 数据中心运行的 Web 角色确实会调入你的计算机，即使你的计算机几乎肯定会驻留在至少一个防火墙和一个网络地址转换 (NAT) 层后面，情况也是如此。

下面是已完成的 Web 应用程序的起始页的屏幕截图。

![][1]

##设置开发环境

在你可以开始开发 Azure 应用程序之前，需要获取相应工具并设置开发环境。

1.  在[获取工具和 SDK][] 安装 Azure SDK for .NET

2. 	单击你正在使用的 Visual Studio 版本的“安装 SDK”。本教程中的步骤使用 Visual Studio 2013。

	![][42]

3.  当提示你是要运行还是保存安装程序时，单击“运行”。

    ![][2]

4.  在“Web 平台安装程序”中，单击“安装”，然后继续安装。

    ![][3]

5.  安装完成后，你就有了开始开发应用所需的一切。SDK 包含了一些工具，可利用这些工具在 Visual Studio 中轻松开发 Azure 应用程序。如果你未安装 Visual Studio，SDK 还会安装免费的 Visual Studio Express。

## 创建服务命名空间

若要开始在 Azure 中使用服务总线功能，必须先创建一个服务命名空间。命名空间提供了用于对应用程序中的 Service Bus 资源进行寻址的范围容器。

你可以使用[Azure 经典门户][]或 Visual Studio 服务器资源管理器管理命名空间和服务总线消息传送实体，但只能在门户内创建新命名空间。

### 使用 Azure 经典门户创建命名空间：

1.  登录到 [Azure 经典门户][]。

2.  在门户的左侧导航窗格中，单击“服务总线”。

3.  在门户的下方窗格中，单击“创建”。

	![][5]

4.  在“添加新命名空间”对话框中，输入命名空间名称。系统会立即检查该名称是否可用。

	![][6]

5.  在确保命名空间名称可用后，选择应承载您的命名空间的国家或地区（确保使用在其中部署计算资源的同一国家/地区）。

    > [AZURE.IMPORTANT] 选取要选择用于部署应用程序的“相同区域”。这将为您提供最佳性能。

6.	将对话框中的其他字段保留为其默认值（“消息传送”和“标准”层），然后单击复选标记。系统现已创建命名空间并已将其启用。您可能需要等待几分钟，因为系统将为您的帐户配置资源。

	![][38]

你创建的命名空间随后将显示在 Azure 经典门户中，不过需要花费一段时间来激活。请等到状态变为“活动”后再继续。

## 获取命名空间的默认管理凭据

若要在新命名空间上执行管理操作（如创建消息实体），你必须获取该命名空间的凭据。

1.  在主窗口中，单击你的服务命名空间的名称。

	![][39]


2.  单击“连接信息”。

	![][40]


3.  在“访问连接信息”窗格中，找到包含 SAS 密钥和密钥名称的连接字符串。

	![][45]


4.  记下这些凭据，或将它们复制到剪贴板。

## 创建本地服务器

首先，你将构建 (mock) 本地产品目录系统。这将非常简单；可以认为，此系统代表一个实际存在的本地产品目录系统，其中包含我们将尝试集成的完整服务图面。

此项目将作为 Visual Studio 控制台应用程序启动。此项目使用服务总线 NuGet 包，其中包含服务总线库和配置设置。利用 NuGet Visual Studio 扩展，可以轻松地在 Visual Studio 和 Visual Studio Express 中安装和更新库和工具。服务总线 NuGet 包是获取服务总线 API 并为应用程序配置所有服务总线依赖项的最简单的方法。有关使用 NuGet 和服务总线包的详细信息，请参阅[使用 NuGet 服务总线包][]。

### 创建项目

1.  使用管理员权限启动 Microsoft VisualStudio 2013 或 Microsoft Visual Studio Express。若要使用管理员权限启动 Visual Studio，请右键单击“Microsoft Visual Studio 2013”（或“Microsoft Visual Studio Express”），然后单击“以管理员身份运行”。

2.  在 Visual Studio 的“文件”菜单中，单击“新建”，然后单击“项目”。

    ![][10]

3.  从“已安装的模板”的“Visual C#”下单击“控制台应用程序”。在“名称”框中，键入名称 “ProductsServer”：

    ![][11]

4.  单击“确定”以创建 “ProductsServer” 项目。

5.  在“解决方案资源管理器”中，右键单击“ProductsServer”，然后单击“属性”。

6.  单击左侧的“应用程序”选项卡，然后确保“.NET Framework 4”或“.NET Framework 4.5”显示在“目标框架”列表中。如未显示，则从列表中将其选中，然后当提示是否重新加载该项目时单击“是”。

    ![][12]

7.  如果你已为 Visual Studio 安装 NuGet 包管理器，请跳到下一步骤。否则，请访问 [NuGet][]，然后单击[安装 NuGet](http://visualstudiogallery.msdn.microsoft.com/27077b70-9dad-4c64-adcf-c7cf6bc9970c)。按照提示操作以安装 NuGet 包管理器，然后重新启动 Visual Studio。

8.  在解决方案资源管理器中，右键单击“引用”，然后单击“管理 NuGet 包”。

9.  在“NuGet”对话框的左栏中，单击“联机”。

10. 	在右栏中，单击“搜索”框，键入“**服务总线**”，然后选择“Azure 服务总线”项。单击“安装”以完成安装，然后关闭此对话框。

    ![][13]

    请注意，现已引用所需的客户端程序集。

11.  为产品协定添加新类。在“解决方案资源管理器”中，右键单击“ProductsServer”项目，单击“添加”，然后单击“类”。

    ![][14]

12. 在“名称”框中，键入名称 “ProductsContract.cs”。然后单击“添加”。

13. 在“ProductsContract.cs”中，将命名空间定义替换为以下代码，以定义服务的协定。

        namespace ProductsServer
        {
            using System.Collections.Generic;
            using System.Runtime.Serialization;
            using System.ServiceModel;

            // Define the data contract for the service
            [DataContract]
            // Declare the serializable properties.
            public class ProductData
            {
                [DataMember]
                public string Id { get; set; }
                [DataMember]
                public string Name { get; set; }
                [DataMember]
                public string Quantity { get; set; }
            }

            // Define the service contract.
            [ServiceContract]
            interface IProducts
            {
                [OperationContract]
                IList<ProductData> GetProducts();

            }

            interface IProductsChannel : IProducts, IClientChannel
            {
            }
        }

14. 在 Program.cs 中，将命名空间定义替换为以下代码，以为其添加配置文件服务和主机。

        namespace ProductsServer
        {
            using System;
            using System.Linq;
            using System.Collections.Generic;
            using System.ServiceModel;

            // Implement the IProducts interface.
            class ProductsService : IProducts
            {

                // Populate array of products for display on website.
                ProductData[] products =
                    new []
                        {
                            new ProductData{ Id = "1", Name = "Rock",
                                             Quantity = "1"},
                            new ProductData{ Id = "2", Name = "Paper",
                                             Quantity = "3"},
                            new ProductData{ Id = "3", Name = "Scissors",
                                             Quantity = "5"},
                            new ProductData{ Id = "4", Name = "Well",
                                             Quantity = "2500"},
                        };

                // Display a message in the service console application
                // when the list of products is retrieved.
                public IList<ProductData> GetProducts()
                {
                    Console.WriteLine("GetProducts called.");
                    return products;
                }

            }

            class Program
            {
                // Define the Main() function in the service application.
                static void Main(string[] args)
                {
                    var sh = new ServiceHost(typeof(ProductsService));
                    sh.Open();

                    Console.WriteLine("Press ENTER to close");
                    Console.ReadLine();

                    sh.Close();
                }
            }
        }

15. 在解决方案资源管理器中，双击“App.config”文件以在 Visual Studio 编辑器中将其打开。将 “&lt;system.ServiceModel&gt;” 的内容替换为以下 XML 代码。确保将 “yourServiceNamespace” 替换为你的服务命名空间的名称，并将 “yourKey” 替换为之前从 Azure 经典门户中检索到的 SAS 密钥：

        <system.serviceModel>
          <extensions>
             <behaviorExtensions>
                <add name="transportClientEndpointBehavior" type="Microsoft.ServiceBus.Configuration.TransportClientEndpointBehaviorElement, Microsoft.ServiceBus, Version=2.6.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
              </behaviorExtensions>
              <bindingExtensions>
                 <add name="netTcpRelayBinding" type="Microsoft.ServiceBus.Configuration.NetTcpRelayBindingCollectionElement, Microsoft.ServiceBus, Version=2.6.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
              </bindingExtensions>
          </extensions>
          <services>
             <service name="ProductsServer.ProductsService">
               <endpoint address="sb://yourServiceNamespace.servicebus.chinacloudapi.cn/products" binding="netTcpRelayBinding" contract="ProductsServer.IProducts"
        behaviorConfiguration="products"/>
             </service>
          </services>
          <behaviors>
             <endpointBehaviors>
               <behavior name="products">
                 <transportClientEndpointBehavior>
                    <tokenProvider>
                       <sharedAccessSignature keyName="RootManageSharedAccessKey" key="yourKey" />
                    </tokenProvider>
                 </transportClientEndpointBehavior>
               </behavior>
             </endpointBehaviors>
          </behaviors>
        </system.serviceModel>

16. 按 F6 或从“生成”菜单中单击“生成解决方案”生成该应用程序，以验证你到目前为止的工作的准确性。

## 创建 ASP.NET MVC 应用程序

在本部分中，你将生成一个简单的 ASP.NET 应用程序，以便显示你的产品服务中检索到的数据。

### 创建项目

1.  确保使用管理员权限运行 Visual Studio。若不能，要使用管理员权限启动 Visual Studio，请右键单击“Microsoft Visual Studio 2013”（或“Microsoft Visual Studio Express”），然后单击“以管理员身份运行”。Azure 计算模拟器（本文后面会讨论）要求使用管理员权限启动 Visual Studio。

2.  在 Visual Studio 的“文件”菜单中，单击“新建”，然后单击“项目”。

3.  从“已安装的模板”的“Visual C#”下单击“ASP.NET Web 应用程序”。将项目命名为 “ProductsPortal”。然后，单击“确定”。

    ![][15]

4.  从“选择模板”列表中，单击“MVC”，然后单击“确定”。

    ![][16]

5.  在解决方案资源管理器中，右键单击“模型”，再单击“添加”，然后单击“类”。在“名称”框中，键入名称 “Product.cs”。然后单击“添加”。

    ![][17]

### 修改 Web 应用程序

1.  在 Visual Studio 的 Product.cs 文件中将现有命名空间定义替换为以下代码。

        // Declare properties for the products inventory.
        namespace ProductsWeb.Models
        {
            public class Product
            {
                public string Id { get; set; }
                public string Name { get; set; }
                public string Quantity { get; set; }
            }
        }

2.  在 Visual Studio 的 HomeController.cs 文件中，将现有命名空间定义替换为以下代码。

        namespace ProductsWeb.Controllers
        {
            using System.Collections.Generic;
            using System.Web.Mvc;
            using Models;

            public class HomeController : Controller
            {
                // Return a view of the products inventory.
                public ActionResult Index(string Identifier, string ProductName)
                {
                    var products = new List<Product>
                        {new Product {Id = Identifier, Name = ProductName}};
                    return View(products);
                }

            }
        }

3.  在解决方案资源管理器中，展开 Views\\Shared 文件夹。

    ![][18]

4.  双击 “\_Layout.cshtml” 以在 Visual Studio 编辑器中将其打开。

5.  将每一处 "My ASP.NET Application" 更改为 “LITWARE'S Products”。

6. 删除“Home”、“About”和“Contact”链接。在下面的示例中，删除突出显示的代码。

	![][41]

7.  在解决方案资源管理器中，展开 Views\\Home 文件夹：

    ![][20]

8.  双击“Index.cshtml”在 Visual Studio 编辑器中将其打开。将文件的全部内容替换为以下代码。

		@model IEnumerable<ProductsWeb.Models.Product>

		@{
    		ViewBag.Title = "Index";
		}

		<h2>Prod Inventory</h2>

		<table>
    		<tr>
        		<th>
            		@Html.DisplayNameFor(model => model.Name)
        		</th>
                <th></th>
        		<th>
            		@Html.DisplayNameFor(model => model.Quantity)
        		</th>
    		</tr>

		@foreach (var item in Model) {
    		<tr>
        		<td>
            		@Html.DisplayFor(modelItem => item.Name)
        		</td>
        		<td>
            		@Html.DisplayFor(modelItem => item.Quantity)
        		</td>
    		</tr>
		}

		</table>


9.  若要验证你到目前为止的工作的准确性，可以按 “F6” 或 “Ctrl+Shift+B” 来生成项目。


### 在本地运行应用程序

运行应用程序以验证其是否正常运行。

1.  确保“ProductsPortal”是活动项目。在“解决方案资源管理器”中，右键单击项目名称并选择“设置为启动项目”。
2.  在 “Visual Studio” 中，按 F5。
3.  你的应用程序应在浏览器中显示为正在运行。

    ![][21]

## 准备好将应用程序部署到 Azure

你可以将应用程序部署到 Azure [云服务][executionmodels]。若要了解如何将应用程序部署到 Azure 网站，请参阅[将 ASP.NET Web 应用程序部署到 Azure 网站](/develop/net/)。本部分包含有关如何将应用程序部署到 Azure 云服务的详细步骤。

若要将应用程序部署到云服务，需要在解决方案中添加一个云服务项目部署项目。该部署项目包含在云中正常运行应用程序所需的配置信息。

1.  若要使应用程序能够部署到云中，请右键单击“解决方案资源管理器”中的 “ProductsPortal” 项目，再单击“转换”，然后单击“转换为 Azure 云服务项目”。

    ![][22]

2.  若要测试应用程序，请按 F5。

3.  这将启动 Azure 计算模拟器。此计算模拟器使用本地计算机来模拟在 Azure 中运行的应用程序。可以通过查看系统托盘来确认此模拟器已启动。

	![][23]

4.  浏览器仍将显示你的应用程序正在本地运行，并且其外观和功能与你之前将其作为常规 ASP.NET MVC 4 应用程序运行时的外观和功能相同。

## 将各个部分组合在一起

下一步是将本地产品服务器与 ASP.NET MVC 应用程序挂钩。

1.  如果它尚未打开，则在 Visual Studio 中重新打开你在"Creating an ASP.NET MVC Application"部分中创建的 “ProductsPortal” 项目。

2.  采用与“创建本地服务器”部分类似的步骤，将 NuGet 包添加到项目“引用”中。在解决方案资源管理器中，右键单击“引用”，然后单击“管理 NuGet 包”。

3.  搜索“服务总线”并选择“Azure 服务总线”项。然后，完成安装过程并关闭此对话框。

4.  在解决方案资源管理器中，右键单击“ProductsPortal”项目，然后单击“添加”，再单击“现有项”。

5.  从 “ProductsServer” 控制台项目导航到 “ProductsContract.cs” 文件。单击以突出显示 ProductsContract.cs。单击“添加”旁边的向下箭头，然后单击“添加为链接”。

	![][24]

6.  现在，在 Visual Studio 编辑器中打开 “HomeController.cs” 文件，并将命名空间定义替换为以下代码。确保将 “yourServiceNamespace” 替换为你的服务命名空间的名称，并将 “yourKey” 替换为你的 SAS 密钥。这将使客户端能够调用本地服务，并返回调用的结果。

            namespace ProductsWeb.Controllers
            {
                using System.Linq;
                using System.ServiceModel;
                using System.Web.Mvc;
                using Microsoft.ServiceBus;
                using Models;
                using ProductsServer;

                public class HomeController : Controller
                {
                    // Declare the channel factory.
                    static ChannelFactory<IProductsChannel> channelFactory;

                    static HomeController()
                    {
                        // Create shared secret token credentials for authentication.
                        channelFactory = new ChannelFactory<IProductsChannel>(new NetTcpRelayBinding(),
                            "sb://yourServiceNamespace.servicebus.chinacloudapi.cn/products");
                        channelFactory.Endpoint.Behaviors.Add(new TransportClientEndpointBehavior {
                            TokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider(
                                "RootManageSharedAccessKey", "yourKey") });
                    }

                    public ActionResult Index()
                    {
                        using (IProductsChannel channel = channelFactory.CreateChannel())
                        {
                            // Return a view of the products inventory.
                            return this.View(from prod in channel.GetProducts()
                                             select
                                                 new Product { Id = prod.Id, Name = prod.Name,
                                                     Quantity = prod.Quantity });
                        }
                    }
                }
            }
7.  在解决方案资源管理器中，右键单击“ProductsPortal”解决方案，单击“添加”，然后单击“现有项目”。

8.  导航到 “ProductsServer” 项目，然后双击“ProductsServer.csproj”解决方案文件以将其添加。

9.  在解决方案资源管理器中，右键单击“ProductsPortal”解决方案并单击“属性”。

10. 在左侧，单击“启动项目”。在右侧，单击“多个启动项目”。确保 “ProductsServer”、“ProductsPortal.Azure” 和 “ProductsPortal” 按这样的顺序显示，并且将“启动”设置为针对 “ProductsServer” 和 “ProductsPortal.Azure” 的操作，将“无”设置为针对 “ProductsPortal” 的操作。

      ![][25]

11. 仍然在“属性”对话框中，单击左侧的“ProjectDependencies”。

12. 在“项目”列表中，单击“ProductsServer”。确保未选中“ProductsPortal”，但选中“ProductsPortal.Azure”。然后单击“确定”：

    ![][26]

## 运行应用程序

1.  从 Visual Studio 的“文件”菜单中，单击“全部保存”。

2.  按 F5 生成并运行应用程序。应先启动本地服务器（“ProductsServer” 控制台应用程序），再在浏览器窗口中启动 “ProductsWeb” 应用程序，如以下屏幕截图中所示。这次，你将看到产品库存列出了从产品服务本地系统中检索到的数据。

    ![][1]

## 将你的应用程序部署到 Azure

1.  在解决方案资源管理器中，右键单击“ProductsPortal”项目，然后单击“发布到 Azure”。

2.  你可能必须登录才能查看你的所有订阅。

    单击“登录以查看更多订阅”：

    ![][27]

3.  使用你的 Microsoft 帐户登录

4.  单击“下一步”。如果你的订阅尚未包含任何托管服务，则系统将要求你创建一个托管服务。托管服务在 Azure 订阅中充当应用程序的容器。输入标识应用程序的名称，然后选择应为其优化应用程序的区域。（用户从此区域访问应用程序所花的加载时间会更少。）

5.  选择要将应用程序发布到其中的托管服务。为其余设置保留以下所示的默认值。单击“下一步”。

    ![][33]

6. 在最后一页上，单击“发布”以开始部署过程。

    ![][34]
    
     此过程需要大约 5-7 分钟时间。由于这是你首次发布，因此 Azure 会依次执行以下操作以便公开应用程序：预配一台虚拟机 (VM)，执行安全强化，在 VM 上创建一个 Web 角色以承载应用程序，将代码部署到该 Web 角色以及配置负载平衡器和网络。

7. 当发布正在进行时，你可以在“Azure 活动日志”窗口中监视活动，该窗口通常位于 Visual Studio 或 Visual Web Developer 的底部。

    ![][35]

8. 部署完成后，你可以通过单击监视窗口中的“网站 URL”链接来查看网站。

    ![][36]

    你的网站取决于本地服务器，因此你必须在本地运行 **ProductsServer** 应用程序，网站才能正常运行。在云网站上执行请求时，你将看到请求传入本地控制台应用程序，如以下屏幕快照中显示的“GetProducts called”输出所示。

    ![][37]

## 停止并删除应用程序

在部署应用程序后，你可能希望禁用它，以便能在 750 小时/月（31 天/月）的免费服务器时间内生成和部署其他应用程序。

Azure 将按使用的服务器小时数对 Web 角色实例计费。你的应用程序部署之后就会开始使用服务器时间，即使相关实例并未运行且处于停止状态也是如此。免费帐户包含 750 小时/月（31 天/月）的专用虚拟机服务器时间（用于承载这些 Web 角色实例）。

以下步骤演示了如何停止和删除应用程序。

1.  登录到 [Azure 经典门户][]，单击“云服务”，然后单击服务的名称。

2.  单击“仪表板”选项卡，然后单击“停止”以暂时挂起应用程序。单击“启动”即可重新启动它。单击“删除”即可从 Azure 中完全删除应用程序，但无法将其还原。

	![][43]

## 后续步骤  

若要了解有关 Service Bus 的详细信息，请参阅以下资源：

* [Azure 服务总线][sbwacom]  
* [如何使用 Service Bus 队列][sbwacomqhowto]  


  [0]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hybrid.png
  [1]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/App2.png
  [获取工具和 SDK]: http://go.microsoft.com/fwlink/?LinkId=271920
  [NuGet]: http://nuget.org
  [2]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-3.png
  [3]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-42-webpi.png


  [Azure 经典门户]: http://manage.windowsazure.cn
  [5]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/sb-queues-03.png
  [6]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/sb-queues-04.png



  [使用 NuGet 服务总线包]: https://msdn.microsoft.com/zh-cn/library/azure/dn741354.aspx
  [10]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-1.png
  [11]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-con-1.png
  [12]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-con-3.png
  [13]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-multi-tier-13.png
  [14]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-con-4.png
  [15]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-2.png
  [16]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-4.png
  [17]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-7.jpg
  [18]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-10.jpg

  [20]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-11.png
  [21]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/App1.png
  [22]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-21.png
  [23]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-22.png
  [24]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-12.png
  [25]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-13.png
  [26]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-14.png
  [27]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-33.png


  [30]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-36.png
  [31]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-37.png
  [32]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-38.png
  [33]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-39.png
  [34]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-40.png
  [35]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-41.png
  [36]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/App2.png
  [37]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-service1.png
  [38]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-multi-tier-27.png
  [39]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/sb-queues-09.png
  [40]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/sb-queues-06.png
  [41]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-multi-tier-40.png
  [42]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-41.png
  [43]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-43.png
  [45]: ./media/service-bus-dotnet-hybrid-app-using-service-bus-relay/hy-web-45.png

  [sbwacom]: /documentation/services/service-bus/
  [sbwacomqhowto]: /documentation/articles/service-bus-dotnet-how-to-use-queues/
  [executionmodels]: /documentation/articles/fundamentals-application-models/
  [azureweb]: /documentation/articles/app-service-web-overview/

<!---HONumber=Mooncake_0104_2016-->