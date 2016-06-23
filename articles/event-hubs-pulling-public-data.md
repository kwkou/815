<properties
   pageTitle="将公共数据拉取到 Azure 事件中心中 | Azure"
   description="从 Web 示例进行事件中心导入的概述"
   services="event-hubs"
   documentationCenter="na"
   authors="spyrossak"
   manager="timlt"
   editor=""/>

<tags 
   ms.service="event-hubs"
    ms.date="04/26/2016"
   wacn.date="06/20/2016" />

# 将公共数据拉取到 Azure 事件中心中

在典型物联网 (IoT) 方案中，可以对相关设备进行编程以将数据推送到 Azure（推送到 Azure 事件中心或 IoT 中心）。这两个中心都是 Azure 的入口点，用于通过在  Azure 上提供的多种工具进行存储、分析和可视化。但是，它们都要求将数据推送到它们（格式化为 JSON，并采用特定方式进行保护）。这会带来以下问题。如果要从数据作为 Web 服务或某种类型的源进行公开的公共或私有源导入数据，但是无法更改数据的发布方式，那么该怎么做？ 请考虑天气、交通或股票报价 — 你无法告知 NOAA、WSDOT 或 NASDAQ 来配置发送到你的事件中心的推送。为了解决此问题，我们编写了一个小型云示例并开放了其源代码，可以修改和部署它，从某个这类源拉取数据并推送到你的事件中心。在其中可以执行任何你要通过它实现的操作，当然，需要遵从创建方提供的许可条款。可以在[此处](https://azure.microsoft.com/documentation/samples/event-hubs-dotnet-importfromweb/)找到该应用程序。

请注意，此示例中的代码仅演示如何从典型 Web 源拉取数据，以及如何向 Azure 事件中心写入。此示例并不旨在作为生产应用程序，并且未尝试使它适合于在这类环境中使用 — 它完全是个 DIY，仅仅是以开发人员为中心的示例。而且，存在此示例并不等同于建议你应将数据**拉取**到 Azure 而不是**推送**它。你在确定端到端体系结构之前，应审查安全、性能、功能和成本因素。

## 应用程序结构

该应用程序是以 C# 编写的，[示例说明](https://github.com/Azure-Samples/event-hubs-dotnet-importfromweb)包含修改、生成和发布应用程序所需的全部信息。以下部分提供了有关该应用程序的功能的全面概述。

我们首先假设你有权访问一个数据馈送。例如，你可能要从华盛顿州交通部拉取交通数据（或是从 NOAA 拉取天气数据）以在应用程序中显示自定义报告或是将该数据与其他数据合并。你还需要设置 Azure 事件中心，并了解访问它所需的连接字符串。

当 GenericWebToEH 解决方案启动时，它会读取配置文件 (App.config) 来获取一些信息：

1. 发布数据的站点的 URL 或 URL 列表。理想情况下，这是采用 JSON 发布数据的站点（例如 WSDOT 在[此处](http://www.wsdot.wa.gov/Traffic/api/)引用的站点）。 
2. URL 的凭据（如果需要）。许多公共源不需要凭据，或者可以将凭据置于 URL 字符串中。其他则需要你单独提供。（请注意，在此应用程序中只能指定一组凭据，因此它只适用于仅指定一个 URL 而不是 URL 列表的情况。）
3. 服务总线连接字符串以及该服务总线命名空间中的事件中心的名称（你会向其推送数据）。可以在 Azure 经典门户中找到此信息。
4. 轮询公共数据站点之间的间隔的休眠间隔（以毫秒为单位）。设置此值需要深思熟虑。如果轮询频率太低，则你可能会遗漏数据；另一方面，如果轮询太频繁，则你可能会收到大量重复数据，更糟的是，你可能会被当作恶意自动程序而受到阻止。请考虑更新数据源的频率 — 天气或交通数据可能每 15 分钟刷新一次，但是股票报价可能每几秒刷新一次（具体取决于获取它们的位置）。 
5. 用于告知应用程序数据是以 JSON 还是 XML 形式进入的标志。由于你需要将数据推送到事件中心，因此应用程序具有一个用于在发送之前将 XML 转换为 JSON 的模块。

读取配置文件之后，应用程序会进入一个循环 — 访问公共网站，在需要时转换数据，将数据写入事件中心，然后等待休眠间隔，之后再次执行以上过程。具体而言：

  * 读取公共网站。为接收准备好发送的数据，通过 Azure/GenericWebToEH/ApiReaders/RawXMLWithHeaderToJsonReader.cs 使用 RawXMLWithHeaderToJsonReader 类的实例。它在 GetData() 方法中读取源流，然后使用 GetXmlFromOriginalText 将它拆分为较小部分（例如记录）。此方法会读取 XML 以及格式正确的 JSON 或 JSON 数组。随后使用 App.config 中的 MergeToXML 配置（默认值为空）启动处理。
  * 接收和发送数据在 Program.cs 中的 Process() 方法中作为循环来实现。从 GetData() 接收输出结果之后，该方法会将分隔的值排队到事件中心中。

## 后续步骤

若要部署解决方案，请克隆或下载 [GenericWebToEH](https://github.com/Azure-Samples/event-hubs-dotnet-importfromweb) 应用程序，编辑 App.config 文件，生成它，最后发布它。发布了该应用程序之后，便可以在 Azure 经典门户中的云服务下看到它运行，而且可以在“配置”选项卡中更改某些配置设置（例如事件中心目标和休眠间隔）。

请参阅[MSDN](https://code.msdn.microsoft.com/site/search?query=event%20hubs&f%5B0%5D.Value=event%20hubs&f%5B0%5D.Type=SearchText&ac=5) 中的更多事件中心示例。

<!---HONumber=Mooncake_0321_2016-->