<properties linkid="dev-net-tutorials-hybrid-solution" urlDisplayName="Hybrid Application" pageTitle="本地/云混合应用程序 (.NET) - Azure" metaKeywords="Azure Service Bus tutorial,hybrid .NET" description="了解如何使用 Azure Service Bus 中继创建 .NET 本地/云混合应用程序。" metaCanonical="" services="service-bus" documentationCenter=".NET" title="使用 Service Bus 中继创建 .NET 本地/云混合应用程序" authors="sethm" solutions="" manager="dwrede" editor="mattshel" />

# 使用 Service Bus 中继创建 .NET 本地/云混合应用程序

## <span class="short-header">简介</span>简介

使用 Visual Studio 2013 和免费的 Azure SDK for .NET，
可以轻松地开发针对 Azure 的混合云应用程序。本指南
假定你之前未使用过 Azure。在不到
30 分钟的时间内，你就能让使用多个 Windows
Azure 资源的应用程序在云中启动并运行。

你将了解到以下内容：

-   如何创建或修改现有 Web 服务
    以供 Web 解决 方案使用。
-   如何使用 Azure Service Bus 中继功能
    在 Azure 应用程序和托管于其他某处的 Web 服务之间共享数据。

[WACOM.INCLUDE [create-account-note](../includes/create-account-note.md)]

### Service Bus 中继功能将为混合解决方案带来哪些帮助

业务解决方案通常由为处理独特的
新业务需求而编写的自定义代码和已有的
解决方案和系统所提供的现有功能
组成。

解决方案架构师开始使用云来
轻松地处理缩放需求和降低运营成本。在此过程中，他们发现
要用作其解决方案构建基块的现有
服务资产位于企业防火墙内，无法通过云解决方案
轻松访问。许多内部服务的
构建或托管方式使得它们无法
在企业网络边缘轻松公开。

*“Service Bus 中继”*的设计考虑到如何利用
现有的 Windows Communication Foundation (WCF) Web 服务
，使得位于企业外部的解决方案
能够安全地访问这些服务，而无需对企业网络基础结构
进行彻底的更改。虽然此类 Service Bus 中继服务
仍托管在现有环境中，但它们会将侦听传入会话和请求
这一任务委托给云托管的 Service Bus。Service Bus 还会通过
使用 Azure Active Directory Access Control
来保护这些服务免遭未经授权的访问。

### 解决方案应用场景

在本教程中，你将创建一个 ASP.NET MVC 4 网站，用于查看
产品库存页上的产品列表。

![][0]

本教程假定你的产品信息位于现有的
本地系统中，而且你使用 Service Bus 中继
来访问该系统。这是由在简单的控制台应用程序中运行的 Web 服务
模拟的，并由一系列内存中产品提供支持。你
将能够在你自己的计算机上运行此控制台应用程序并
将 Web 角色部署到 Azure 中。通过此操作，你将看到
在 Azure 数据中心运行的 Web 角色
确实会调入你的计算机，即使你的计算机几乎肯定
会驻留在至少一个防火墙和一个网络地址转换 (NAT) 层后面，
情况也是如此。

下面是已完成的 Web 应用程序的起始页的屏幕截图。

![][1]

## <span class="short-header">设置环境</span>设置开发环境

在你可以开始开发 Azure 应用程序之前，你需要
获取相应工具并设置开发环境。

1.  若要安装 Azure SDK for .NET，请单击以下按钮：

    [获取工具和 SDK][获取工具和 SDK]

2.  单击“安装 SDK”。

3.  选择要使用的 Visual Studio 版本的链接。本教程中的步骤使用 Visual Studio 2013：

    ![][2]

4.  当提示你运行或保存 **MicrosoftAzureSDKForNet.exe** 时，
    请单击“运行”：

    ![][3]

5.  在 Web 平台安装程序中，单击“安装”，然后继续安装：

    ![][4]

6.  安装完成后，你便做好了
    开发准备工作。SDK 包含了一些工具，可利用这些工具
    在 Visual Studio 中轻松开发 Azure 应用程序。如果你
    未安装 Visual Studio，它还会安装免费的
    Visual Studio Express。

## <span class="short-header">创建命名空间</span>创建服务命名空间

若要开始在 Azure 中使用 Service Bus 功能，必须先
创建一个服务命名空间。服务命名空间提供了用于对应用程序
中的 Service Bus 资源进行寻址的范围容器。

你可以使用 [Azure 管理门户][Azure 管理门户]或 Visual Studio 服务器资源管理器来管理命名空间和 Service Bus 消息传送实体，但只能在门户内创建新的命名空间。

### 若要使用门户创建服务命名空间，请执行以下操作：

1.  登录到 [Azure 管理门户][Azure 管理门户]。

2.  在该管理门户的左侧导航窗格中，单击
    **Service Bus**。

3.  在该管理门户的下方窗格中，单击“创建”。
    ![][5]

4.  在“添加新命名空间”对话框中，输入命名空间名称。
    系统会立即检查该名称是否可用。
    ![][6]

5.  在确保命名空间名称可用后，选择
    应托管你的命名空间的国家或地区（确保
    使用在其中部署计算资源的同一
    国家/地区）。

    重要说明：选取要部署应用程序的**相同区域**。
    这将为你提供最佳性能。

6.  单击复选标记。系统现在会创建你的服务
    命名空间并启用它。你可能需要等待几分钟，因为
    系统将为你的帐户配置资源。

    ![][7]

你创建的命名空间随后将显示在管理门户中，并且需要
一段时间来激活。请等到状态变为“活动”后
再继续。

## <span class="short-header">获取管理凭据</span>获取命名空间的默认管理凭据

若要在新命名空间上执行管理操作
（如创建队列），则必须获取该命名空间的管理
凭据。

1.  在主窗口中，单击你的服务命名空间的名称。

    ![][8]

2.  单击“连接信息”。

    ![][9]

3.  在“访问连接信息”面板中，找到“默认颁发者”和“默认密钥”条目。

4.  记下该密钥或将其复制到剪贴板。

### 使用 Visual Studio 服务器资源管理器管理服务命名空间：

若要使用 Visual Studio 而非管理门户来管理命名空间并获取连接信息，请按[此处][此处]所述过程进行操作，详见**从 Visual Studio 连接到 Azure** 这一节。当你登录到 Azure 时，服务器资源管理器中 **Microsoft Azure** 树下的 **Service Bus** 节点中会自动填充你所创建的任何命名空间。右键单击任意命名空间，然后单击“属性”，此时就会看到在 Visual Studio 的“属性”窗格中显示与该命名空间关联的连接字符串和其他元数据。

![][10]

记下 **SharedAccessKey** 值或将其复制到剪贴板。

## <span class="short-header">创建本地服务器</span>创建本地服务器

首先，你将构建 (mock) 本地产品目录系统。这
将非常简单；可以认为，此系统
代表一个实际存在的本地产品目录系统，
其中包含我们将尝试集成的完整服务图面。

此项目将作为 Visual Studio 控制台应用程序启动。此
项目使用 Service Bus NuGet 包，其中包含 Service Bus 库
和配置设置。利用 NuGet Visual Studio 扩展，可以
轻松地在 Visual Studio 和 Visual Studio Express 中安装和更新
库和工具。Service Bus NuGet 包是
获取 Service Bus API 和使用所有 Service Bus 依赖项配置
应用程序的最简单的方法。有关使用 NuGet 和 Service Bus 包的详细信息，请参阅
[使用 NuGet Service Bus 包][使用 NuGet Service Bus 包]。

### 创建项目

1.  使用管理员权限启动 Microsoft Visual
    Studio 2013 或 Microsoft Visual Studio Express。若要
    使用管理员权限启动 Visual Studio，请右键单击
    “Microsoft Visual Studio 2013”（或“Microsoft Visual Studio Express”），然后单击“以管理员身份运行”。
2.  在 Visual Studio 的“文件”菜单中，单击“新建”，然后
    单击“项目”。

    ![][11]

3.  **从“已安装的模板”**的“Visual C#”下
    单击“控制台应用程序”。在“名称”框中，键入
    名称“ProductsServer”：

    ![][12]

4.  单击“确定”以创建 **ProductsServer** 项目。

5.  **在“解决方案资源管理器”**中，右键单击“ProductsServer”，然后
    单击“属性”。
6.  单击左侧的“应用程序”选项卡，然后确保“.NET Framework 4”
    或“.NET Framework 4.5”显示在“目标框架:”下拉列表中。如果未显示，请从下拉列表中选择它，然后
    在提示你重新加载项目时单击“是”。

    ![][13]

7.  如果你已为 Visual Studio 安装 NuGet 包管理器，请跳到下一步骤。否则，请访问 [NuGet][NuGet]，然后单击[安装 NuGet][安装 NuGet]。按照提示操作以安装 NuGet 包管理器，然后重新启动 Visual Studio。

8.  在“解决方案资源管理器”中，右键单击“引用”，然后单击
    “管理 NuGet 包”。
9.  在 NuGet 对话框的左栏中，单击“联机”。

10. 在右栏中，单击“搜索”框，键入“MicrosoftAzure”，
    并选择“Microsoft Azure Service Bus”项。单击“安装”以完成安装，
    然后关闭此对话框。

    ![][14]

    请注意，现已引用所需的客户端程序集。

11. 为产品协定添加新类。在“解决方案资源管理器”中，
    右键单击“ProductsServer”项目，
    单击“添加”，然后单击“类”。

    ![][15]

12. 在“名称”框中，键入名称“ProductsContract.cs”。然后
    单击“添加”。
13. 在“ProductsContract.cs”中，将命名空间定义替换为
    以下代码（用于定义服务的协定）：

        namespace ProductsServer
        {
            using System.Collections.Generic;
            using System.Runtime.Serialization;
            using System.ServiceModel;

            // Define the data contract for the service
            [DataContract]
            // Declare the serializable properties
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

14. 在 Program.cs 中，将命名空间定义替换为以下
    代码，以便添加配置文件服务及其宿主：

        namespace ProductsServer
        {
            using System;
            using System.Linq;
            using System.Collections.Generic;
            using System.ServiceModel;

            // Implement the IProducts interface
            class ProductsService : IProducts
            {

                // Populate array of products for display on Web site
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
                // when the list of products is retrieved
                public IList<ProductData> GetProducts()
                {
                    Console.WriteLine("GetProducts called.");
                    return products;
                }

            }

            class Program
            {
                // Define the Main() function in the service application
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

15. 在“解决方案资源管理器”中，双击 **app.config** 文件
    以在 **Visual Studio** 编辑器中将其打开。将
    **\<system.ServiceModel\>** 的内容替换为以下 XML 代码。确保将
    *yourServiceNamespace* 替换为你的服务
    命名空间的名称，并将 *yourIssuerSecret*
     替换为之前从 Azure 管理门户中检索到的密钥：

        <system.serviceModel>
          <extensions>
             <behaviorExtensions>
                <add name="transportClientEndpointBehavior" type="Microsoft.ServiceBus.Configuration.TransportClientEndpointBehaviorElement, Microsoft.ServiceBus, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
              </behaviorExtensions>
              <bindingExtensions>
                 <add name="netTcpRelayBinding" type="Microsoft.ServiceBus.Configuration.NetTcpRelayBindingCollectionElement, Microsoft.ServiceBus, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
              </bindingExtensions>
          </extensions>
          <services>
             <service name="ProductsServer.ProductsService">
               <endpoint address="sb://yourServiceNamespace.servicebus.windows.net/products" binding="netTcpRelayBinding" contract="ProductsServer.IProducts"
        behaviorConfiguration="products"/>
             </service>
          </services>
          <behaviors>
             <endpointBehaviors>
               <behavior name="products">
                 <transportClientEndpointBehavior>
                    <tokenProvider>
                       <sharedSecret issuerName="owner" issuerSecret="yourIssuerSecret" />
                    </tokenProvider>
                 </transportClientEndpointBehavior>
               </behavior>
             </endpointBehaviors>
          </behaviors>
        </system.serviceModel>

16. 按 **F6** 或从“构建”菜单中单击“构建解决方案”生成该应用程序，以验证你目前工作的准确性。

## <span class="short-header">创建 ASP.NET MVC 应用程序</span>创建 ASP.NET MVC 应用程序

在本节中，你将生成一个简单的 MVC 4 应用程序，以便
显示你的产品服务中检索到的数据。

### 创建项目

1.  确保使用管理员权限运行 Microsoft Visual Studio 2013。否则，
    请右键单击
    “Microsoft Visual Studio 2013”（或“Microsoft Visual Studio Express”），然后单击“以管理员身份运行”，以便使用管理员权限启动 Visual Studio。Microsoft
    Azure 计算模拟器（本指南后面会讨论）要求
    使用管理员权限启动 Visual Studio。

2.  在 Visual Studio 的“文件”菜单中，单击“新建”，然后
    单击“项目”。

3.  **从“已安装的模板”**的“Visual C#”下单击“ASP.NET Web 应用程序”。将项目命名为“ProductsPortal”。然后，
    单击“确定”。

    ![][16]

4.  从“选择模板”列表中，单击“MVC”，
    然后单击“确定”。

    ![][17]

5.  在“解决方案资源管理器”中，右键单击“模型”，单击“添加”，
    然后单击“类”。在“名称”框中，键入名称
    “Product.cs”。然后单击“添加”。

    ![][18]

### 修改 Web 应用程序

1.  在 Visual Studio 的 Product.cs 文件中将现有
    命名空间定义替换为以下代码：

        // Declare properties for the products inventory
        namespace ProductsWeb.Models
        {
            public class Product
            {
                public string Id { get; set; }
                public string Name { get; set; }
                public string Quantity { get; set; }
            }
        }

2.  在 Visual Studio 的 HomeController.cs 文件中将现有
    命名空间定义替换为以下代码：

        namespace ProductsWeb.Controllers
        {
            using System.Collections.Generic;
            using System.Web.Mvc;
            using Models;

            public class HomeController : Controller
            {
                // Return a view of the products inventory
                public ActionResult Index(string Identifier, string ProductName)
                {
                    var products = new List<Product> 
                        {new Product {Id = Identifier, Name = ProductName}};
                    return View(products);
                }

            }
        }

3.  在“解决方案资源管理器”中，展开 Views\\Shared：

    ![][19]

4.  接下来，双击 \_Layout.cshtml 以在 Visual Studio 编辑器中将其打开。

5.  将每一处 **My ASP.NET Application** 更改为 **LITWARE'S Products**。

6.  删除“Home”、“About”和“Contact”链接。删除突出显示的代码：

    ![][20]

7.  在“解决方案资源管理器”中，展开 Views\\Home：

    ![][21]

8.  双击 Index.cshtml 将其在 Visual Studio 编辑器中打开。
    将该文件的整个内容替换为以下代码：

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

9.  若要验证你目前工作的准确性，可以
    按 **F6** 或 **Ctrl+Shift+B** 来生成项目。

### 本地运行应用程序

运行应用程序以验证其是否正常运行。

1.  确保“ProductsPortal”是活动项目。右键单击
    “解决方案资源管理器”中的项目名称，
    然后选择“设为启动项目”
2.  在 **Visual Studio** 中，按 **F5**。
3.  你的应用程序应在浏览器中显示为正在运行：

    ![][22]

    ## <span class="short-header">部署到 Azure</span>准备好将应用程序部署到 Azure

    你可以将应用程序部署到 Azure 云服务或 Azure 网站。若要详细了解网站和云服务之间的差异，请参阅 [Azure 执行模型][Azure 执行模型]。若要了解如何将应用程序部署到 Azure 网站，请参阅[将 ASP.NET Web 应用程序部署到 Azure 网站][将 ASP.NET Web 应用程序部署到 Azure 网站]。本部分包含有关如何将应用程序部署到 Azure 云服务的详细步骤。

    若要将应用程序部署到云服务，你需要将云服务项目部署项目添加到解决方案。
    部署项目包含在云中正常
    运行你的应用程序所需的
    配置信息。

    1.  若要使应用程序能够部署到云中，请右键单击“解决方案资源管理器”中
        的 **ProductsPortal** 项目，
        单击“转换”，然后单击“转换为 Azure 云服务项目”。

        ![][23]

    2.  若要测试应用程序，请按 **F5**。
    3.  这将启动 Azure 计算模拟器。此计算
        模拟器使用本地计算机来模拟在 Azure 中
        运行的应用程序。可以通过查看系统任务栏来确认此模拟器
        已启动：

        ![][24]

    4.  浏览器仍将显示你的应用程序正在本地运行，
        并且其外观和功能与你之前将其作为常规 ASP.NET MVC 4 应用
        程序运行时的外观和功能相同。

    ## <span class="short-header">将这些片段组合在一起</span>将这些片段组合在一起

    下一步是将本地产品服务器与 ASP.NET MVC4 应
    用程序挂钩。

    1.  如果尚未打开在“创建 ASP.NET MVC 应用程序”部分中
        创建的“ProductsPortal”项目，
        请在 Visual Studio 中重新打开该项目。

    2.  采用与“创建本地服务器”部分类似的步骤，
        将 NuGet 包添加到项目“引用”中。在
        “解决方案资源管理器”中，右键单击“引用”，然后单击
        “管理 NuGet 包”。

    3.  搜索“MicrosoftAzure.ServiceBus”，
        然后选择“Microsoft Azure Service Bus”项。然后，完成安装过程
        并关闭此对话框。

    4.  在“解决方案资源管理器”中，右键单击“ProductsPortal”项目，
        然后单击“添加”，再单击“现有项”。

    5.  从“ProductsServer”控制台项目
        导航到 **ProductsContract.cs** 文件。单击即可突出显示
        ProductsContract.cs。单击“添加”旁边的向下箭头**Add**，然后
        单击“添加为链接”。

        ![][25]

    6.  现在，在 Visual Studio 编辑器中
        打开 **HomeController.cs** 文件，
        并将命名空间定义替换为以下代码。请确保将 *yourServiceNamespace* 替换为
        你的服务命名空间的名称，将 *yourIssuerSecret* 替换为你的密钥。
        这样客户端就可以调用本地服务，
        然后会返回调用的结果。

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
                    // Declare the channel factory
                    static ChannelFactory<IProductsChannel> channelFactory;

                    static HomeController()
                    {
                        // Create shared secret token credentials for authentication 
                        channelFactory = new ChannelFactory<IProductsChannel>(new NetTcpRelayBinding(), 
                            "sb://yourServiceNamespace.servicebus.windows.net/products");
                        channelFactory.Endpoint.Behaviors.Add(new TransportClientEndpointBehavior { 
                            TokenProvider = TokenProvider.CreateSharedSecretTokenProvider(
                                "owner", "yourIssuerSecret") });
                    }

                    public ActionResult Index()
                    {
                        using (IProductsChannel channel = channelFactory.CreateChannel())
                        {
                            // Return a view of the products inventory
                            return this.View(from prod in channel.GetProducts()
                                             select
                                                 new Product { Id = prod.Id, Name = prod.Name, 
                                                     Quantity = prod.Quantity });
                        }
                    }
                }
            }

    7.  在“解决方案资源管理器”中，右键单击“ProductsPortal”解决方案，
        单击“添加”，然后单击“现有项目”。

    8.  导航到 **ProductsServer** 项目，
        然后双击 **ProductsServer.csproj** 解决方案文件以添加它。

    9.  在“解决方案资源管理器”中，右键单击“ProductsPortal”解决方案
        并单击“属性”。

    10. 在左侧，单击“启动项目”。在右侧
        单击“多个启动项目”。确保
        **ProductsServer**、**ProductsPortal.Azure** 和
        **ProductsPortal** 按这样的顺序显示，并且将“启动”设置为
        针对 **ProductsServer** 和 **ProductsPortal.Azure** 的操作，
        将“无”设置为针对 **ProductsPortal** 的操作。例
        如：

        ![][26]

    11. 仍在“属性”对话框中，
        单击左侧的“ProjectDependencies”。

    12. 在“项目”下拉列表中，
        单击“ProductsServer”。确保取消选中“ProductsPortal”
        并选中“ProductsPortal.Azure”。然后单击“确定”：

        ![][27]

    ## <span class="short-header">运行应用程序</span>运行应用程序

    1.  从 Visual Studio 的“文件”菜单中，单击“全部保存”。

    2.  按 **F5** 生成并运行应用程序。应先启动本地
        服务器（**ProductsServer**控制台应用程序），
        再在浏览器窗口中启动**ProductsWeb**应用程序，
        如下面的屏幕快照中所示。这次，
        你将看到产品库存列出了从产品服务本地系统中
        检索到的数据。

        ![][1]

    ## <span class="short-header">部署应用程序</span>将你的应用程序部署到 Azure

    1.  右键单击解决方案资源管理器中的 **ProductsPortal** 项目，
        然后单击“发布到 Azure”。

    2.  你可能必须登录才能查看你的所有订阅。

        单击“登录以查看更多订阅”：

        ![][28]

    3.  使用 Microsoft 帐户登录。

    4.  单击“下一步”。如果你的订阅尚未包含任何托管
        服务，则系统将要求你创建一个托管服务。托管服务
        在 Microsoft Azure 订阅中充当应用程序的
        容器。输入标识应用
        程序的名称，然后选择应为其优化应用
        程序的区域。（用户从此区域访问应用程序所花的
        加载时间会更少。）

        ![][29]

    5.  选择要将应用程序发布到的
        托管服务。为其余设置保留以下所示的
        默认值。单击“下一步”：

        ![][30]

    6.  在最后一页上，单击“发布”以开始部署
        过程：

        ![][31]

        此过程花费的时间约为 5 到 7 分钟。由于这是你首次发布，
        因此 Azure 会依次执行以下操作以便公开应用程序：
        设置一台虚拟机 (VM)，执行安全强化，
        在 VM 上创建一个 Web 角色以托管应用程序，
        将代码部署到该 Web 角色以及配置负载平衡器
        和网络。

    7.  当发布正在进行时，
        你可以在“Azure 活动日志”窗口中监视活动，
        该窗口通常位于 Visual Studio 或 Visual Web Developer 的
        底部：

        ![][32]

    8.  部署完成后，你可以通过单击监视窗口中
        的“网站 URL”链接来查看网站。

        ![][1]

        你的网站依赖于本地服务器，因此你必须
        本地运行 **ProductsServer** 应用程序，
        网站才能正常运行。在云网站上执行请求时，
        你将看到请求传入本地控制台应用程序，
        如下面的屏幕快照中显示的“GetProducts called”
        输出所示。

        ![][33]

若要详细了解网站和云服务之间的差异，请参阅 [Azure 执行模型][Azure 执行模型]。

## <span class="short-header">删除应用程序</span>停止和删除应用程序

在部署应用程序后，你可能希望禁用它，
以便能在 750 小时/月（31 天/月）的免费服务器时间内生成和
部署其他应用程序。

Azure 将按使用的服务器小时数对 Web 角色
实例计费。一旦部署你的应用程序，系统就会开始使用服务器时间，
即使相关实例尚未运行且都处于停止状态。
一个免费帐户包括 750 小时/月（31 天/月）的专用
虚拟机服务器时间，可用于托管这些 Web 角色实例。

以下步骤演示了如何停止和删除
应用程序。

1.  登录 [Azure 管理门户][Azure 管理门户]，单击“云服务”，
    然后单击服务的名称。

2.  单击“仪表板”选项卡，然后单击“停止”以暂时挂起应用程序。只需
    单击“启动”即可重新启动应用程序。单击“删除”即可从 Azure 中完全删除应用程序，
    但无法还原它。

    ![][34]

## <a name="nextsteps"></a><span class="short-header">后续步骤</span>后续步骤

若要了解有关 Service Bus 的详细信息，请参阅以下资源：

-   [Azure Service Bus][Azure Service Bus]
-   [Service Bus 操作方法][Service Bus 操作方法]
-   [如何使用 Service Bus 队列][如何使用 Service Bus 队列]

  [0]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/hybrid.png
  [1]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/App2.png
  [获取工具和 SDK]: http://go.microsoft.com/fwlink/?LinkId=271920
  [2]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/getting-started-41.png
  [3]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/getting-started-3.png
  [4]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/getting-started-4-2-WebPI.png
  [Azure 管理门户]: http://manage.windowsazure.cn
  [5]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/sb-queues-03.png
  [6]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/sb-queues-04.png
  [7]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/getting-started-multi-tier-27.png
  [8]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/sb-queues-09.png
  [9]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/sb-queues-06.png
  [此处]: http://http://msdn.microsoft.com/zh-cn/library/windowsazure/ff687127.aspx
  [10]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/VSProperties.png
  [使用 NuGet Service Bus 包]: http://go.microsoft.com/fwlink/?LinkId=234589
  [11]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/hy-web-1.png
  [12]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/hy-con-1.png
  [13]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/hy-con-3.png
  [NuGet]: http://nuget.org
  [安装 NuGet]: http://visualstudiogallery.msdn.microsoft.com/27077b70-9dad-4c64-adcf-c7cf6bc9970c
  [14]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/getting-started-multi-tier-13.png
  [15]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/hy-con-4.png
  [16]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/hy-web-2.png
  [17]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/hy-web-4.png
  [18]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/hy-web-7.jpg
  [19]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/hy-web-10.jpg
  [20]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/getting-started-multi-tier-40.png
  [21]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/hy-web-11.png
  [22]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/App1.png
  [Azure 执行模型]: http://www.windowsazure.com/zh-cn/develop/net/fundamentals/compute/
  [将 ASP.NET Web 应用程序部署到 Azure 网站]: http://www.windowsazure.com/zh-cn/develop/net/tutorials/get-started/
  [23]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-21.png
  [24]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-22.png
  [25]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/hy-web-12.png
  [26]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/hy-web-13.png
  [27]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/hy-web-14.png
  [28]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-33.png
  [29]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-38.png
  [30]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-39.png
  [31]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-40.png
  [32]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-41.png
  [33]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/hy-service1.png
  [34]: ./media/cloud-services-dotnet-hybrid-app-using-service-bus-relay/getting-started-hybrid-43.png
  [Azure Service Bus]: http://msdn.microsoft.com/zh-cn/library/windowsazure/ee732537.aspx
  [Service Bus 操作方法]: /zh-cn/documentation/services/service-bus/
  [如何使用 Service Bus 队列]: /zh-cn/develop/net/how-to-guides/service-bus-queues/
