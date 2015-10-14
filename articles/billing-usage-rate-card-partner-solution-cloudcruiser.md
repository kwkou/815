<properties
   pageTitle="Cloud Cruiser 和 Microsoft Azure 计费 API 集成"
   description="Microsoft Azure 计费合作伙伴 Cloud Cruiser 将 Azure 计费 API 集成到了其产品中，从而能够根据亲身体验提供独特的观点。对于有兴趣使用/试用 Cloud Cruiser for Microsoft Azure Pack 服务的 Azure 和 Cloud Cruiser 客户而言，这是非常有用的。"
   services="billing"
   documentationCenter=""
   authors="BryanLa"
   manager="mbaldwin"
   editor=""/>

<tags
   ms.service="billing"
   ms.date="08/10/2015"
   wacn.date="10/3/2015"/>

# Cloud Cruiser 和 Microsoft Azure 计费 API 集成

本文介绍如何在 Cloud Cruiser 中使用从新的 Microsoft Azure 计费 API 中收集到的信息，以进行工作流成本模拟和分析。

## Azure RateCard API
RateCard API 提供来自 Azure 的费率信息。在使用正确的凭据进行身份验证之后，您可以查询此 API 以收集关于 Azure 上可用服务的元数据以及与您的产品/服务 ID 相关联的费率。

以下是来自该 API 的示例响应，显示了 A0 (Windows) 实例的价格：

    {
		"MeterId": "0e59ad56-03e5-4c3d-90d4-6670874d7e29",
		"MeterName": "Compute Hours",
		"MeterCategory": "Virtual Machines",
		"MeterSubCategory": "A0 VM (Windows)",
		"Unit": "Hours",
		"MeterRates":
		{
			"0": 0.029
		},
		"EffectiveDate": "2014-08-01T00:00:00Z",
		"IncludedQuantity": 0.0
	},

### Cloud Cruiser 到 Azure RateCard API 的接口
Cloud Cruiser 可以多种方式利用 RateCard API 信息。在本文中，我们将展示如何使用它来进行 IaaS 工作负荷成本模拟和分析。

为了演示此用例，设想了一个在 Microsoft Azure Pack (WAP) 上运行的多个实例的工作负荷。目标是在 Azure 上模拟同样的工作负荷以及估计执行此类迁移的成本。要创建此模拟，需要执行两项主要任务：

1. **导入和处理从 RateCard API 收集的服务信息** -在工作簿上也执行该任务，将对从 RateCard API 提取的信息进行转换并发布到新的费率计划。使用此新的费率计划进行模拟以估计 Azure 价格。

2. **规范化 IaaS 的 WAP 服务和 Azure 服务** - 默认情况下，WAP 服务是基于各个资源（CPU、内存大小、磁盘大小等）的，而 Azure 服务是基于实例大小（A0、A1、A2 等）的。可以由 Cloud Cruiser ETL 引擎（称作工作簿）执行第一个任务，其中可以按照实例尺寸对这些资源进行捆绑，类似于 Azure 实例服务。

### 从 RateCard API 导入数据

Cloud Cruiser 工作簿提供自动化的方式来从 RateCard API 收集信息并进行处理。ETL（提取-转换-加载）工作簿允许您对数据的收集、转换以及发布到 Cloud Cruiser 数据库进行配置。

每个工作簿可以有一个或多个集合。这样，您可以关联来自不同源的信息以补充或增加使用数据。在如下的两个屏幕快照中，我们展示了在现有工作簿中创建一个新的*集合*，以及将信息从 RateCard API 导入到*集合*：

![图 1 - 创建新集合][1]

![图 2 - 从新集合导入数据][2]

将数据导入工作簿后，则可以创建多个步骤和转换过程，以对数据进行修改和建模。在此示例中，由于我们只对服务架构 (IaaS) 感兴趣，因此我们可以使用转换步骤来删除与 IaaS 服务无关的不必要行或记录。

如下的屏幕快照展示了对从 RateCard API 收集到的数据进行处理的转换步骤：

![图 3 - 对从 RateCard API 收集的数据进行处理的转换步骤][3]

### 定义新的服务和费率计划

有多种方法可以定义 Cloud Cruiser 上的服务。其中一种选择是从使用数据导入服务。在使用提供商已经定义了服务的公有云的情况下，通常使用此方法。

费率计划包括一组费率或价格。基于生效日期、客户群或其他因素，不同的费率适用于不同的服务。也可在 Cloud Cruiser 上使用费率计划以创建模拟或“假设”情况，从而了解服务变化如何影响工作负荷的总成本。

在此示例中，我们将使用来自 RateCard API 的服务信息在 Cloud Cruiser 中定义新的服务。同样地，我们可以使用与服务相关联的费率在 Cloud Cruiser 上创建新的费率计划。

在转换过程结束时，就可以创建一个新的步骤，并将来自 RateCard API 的数据作为新服务和费率进行发布。

![图 4 - 将来自 RateCard API 的数据作为新的服务和费率进行发布][4]

### 验证 Azure 服务和费率

在发布服务和费率后，您可以在 Cloud Cruiser 的*服务*选项卡中验证导入的服务列表：

![图 5 - 验证新的服务][5]

在*费率计划*选项卡上，您可以查看名为“AzureSimulation”的新费率计划，其费率是从 RateCard API 导入的。

![图 6 - 验证新的费率计划和关联的费率][6]

### 规范化 WAP 和 Azure 服务

默认情况下，WAP 提供基于计算、内存和网络资源的使用信息。在 Cloud Cruiser 中，您可以直接根据这些资源的分配情况或计费使用情况来定义您的服务。例如，可以对每小时的 CPU 使用率设置一个基本费率，或根据分配给实例的内存 GB 数进行收费。

在此示例中，为了对 WAP 和 Azure 的成本进行比较，我们需要汇集 WAP 上的资源使用数据并进行捆绑，然后映射到 Azure 服务。在工作簿中可以轻松地实现这种转换：

![图 7 - 对 WAP 数据进行转换以规范化服务][7]

该工作簿的最后一步是将数据发布到 Cloud Cruiser 数据库。在此步骤中，使用数据现在捆绑到服务（即映射到 Azure 服务）并与默认费率绑定以创建费用。

完成该工作簿后，您可以通过以下方法实现数据的自动化处理：在计划程序中添加一个新任务，然后指定工作簿要运行的频率和时间。

![图 8 - 工作簿计划][8]

### 为工作负荷成本模拟分析创建报表

在收集了使用数据以及将费用加载到 Cloud Cruiser 数据库后，我们就可以利用 Cloud Cruiser Insights 模块（一种高级的分析报告工具）来创建所需的工作负荷成本模拟。

要演示此情况，我们将创建下列报表：

![成本比较][9]

上面的图表显示了按服务细分的成本比较，对各个具体服务在 WAP（深蓝色）和 Azure（浅蓝色）上运行工作负荷的价格进行了比较。

下面的图表显示了相同数据，但按部门进行了细分。它显示了各部门在 WAP 和 Azure 上运行其工作负荷的成本，以及两者之间的差额 –“节省额”栏（绿色）。

## Azure 使用情况 API


### 介绍

Microsoft 最近推出了 Azure 使用情况 API，允许订户以编程方式提取使用数据以深入了解其使用情况。对于 Cloud Cruiser 客户而言，这是个好消息。他们可以充分利用此 API 可提供的更丰富数据集。

Cloud Cruiser 可以通过多种方式利用与使用情况 API 的集成。通过 API 可用的粒度（每小时使用情况信息）和资源元数据信息提供了支持灵活的 Showback 或 Chargeback 模型所必需的数据集。

在本教程中，我们将通过一个示例来说明 Cloud Cruiser 如何从使用情况 API 信息中获益。更具体地说，我们将在 Azure 上创建一个新的资源组，对帐户结构的标记进行关联，然后描述向 Cloud Cruiser 中提取和处理标记信息的过程。
 
最终目标是能够创建如下报表，并基于由标记填充的帐户结构对成本和消耗数据进行分析。

![图 10 - 使用标记的明细报表][10]

### Microsoft Azure 标记

通过 Azure 使用情况 API 不仅可获得消耗信息，还可以获得资源元数据（包括与之相关联的任何标记）。标记提供了一种简单的方法来组织您的资源，但是，为了使其行之有效，您需要确保：

- 在预配时，将标记正确地应用到资源
- 在 Showback/Chargeback 过程中，正确地使用标记以将使用数据连接到组织的帐户结构。

这两个要求都很具有挑战性，尤其是当在预配或计费方存在某种形式的手动过程时。存在输入错误、不正确的标记甚至缺少标记都是客户在使用标记时的常见抱怨，这些错误给计费方造成很大的困难。

使用新的 Azure 使用情况 API，Cloud Cruiser 可以提取资源标记信息，并通过名为工作簿的非常复杂的 ETL 工具修复这些常见的标记错误。通过利用正则表达式和数据关联的转换步骤，Cloud Cruiser 能够识别错误标记的资源并应用正确的标记，从而弥补差异并确保与使用者的资源进行正确关联。

在计费方，Cloud Cruiser 自动化了 Showback/Chargeback 过程，而且可以利用标记信息将使用数据连接到相应的使用者（部门、区域、项目等）。这种自动化实现了一个巨大的改进，可以确保计费过程具有一致性且可审核。
 

### 在 Microsoft Azure 上创建具有标记的资源组
在本教程中的第一步是在 Azure 门户中创建一个新的资源组，然后创建与资源相关联的新标记。在此示例中，我们将创建以下标记：部门、环境、所有者和项目。

以下的 Azure 门户屏幕快照显示了具有关联标记的资源组示例。

![图 11 - Azure 门户中具有关联标记的资源组][11]

下一步是将信息从使用情况 API 提取到 Cloud Cruiser 中。使用情况 API 目前提供 JSON 格式的响应。下面是检索到的数据示例：


    {
      "id": "/subscriptions/bb678b04-0e48-4b44-XXXX-XXXXXXX/providers/Microsoft.Commerce/UsageAggregates/Daily_BRSDT_20150623_0000",
      "name": "Daily_BRSDT_20150623_0000",
      "type": "Microsoft.Commerce/UsageAggregate",
      "properties":
      {
        "subscriptionId": "bb678b04-0e48-4b44-XXXX-XXXXXXXXX",
        "usageStartTime": "2015-06-22T00:00:00+00:00",
        "usageEndTime": "2015-06-23T00:00:00+00:00",
        "meterName": "Compute Hours",
        "meterRegion": "",
        "meterCategory": "Virtual Machines",
        "meterSubCategory": "Standard_D1 VM (Non-Windows)",
        "unit": "Hours",
        "instanceData": "{"Microsoft.Resources":{"resourceUri":"/subscriptions/bb678b04-0e48-4b44-XXXX-XXXXXXXX/resourceGroups/DEMOUSAGEAPI/providers/Microsoft.Compute/virtualMachines/MyDockerVM","location":"eastus","tags":{"Department":"Sales","Project":"Demo Usage API","Environment":"Test","Owner":"RSE"},"additionalInfo":{"ImageType":"Canonical","ServiceType":"Standard_D1"}}}",
        "meterId": "e60caf26-9ba0-413d-a422-6141f58081d6",
        "infoFields": {},
        "quantity": 8

      },
	},


### 将数据从使用情况 API 导入到 Cloud Cruiser

Cloud Cruiser 工作簿提供自动化的方式来从使用情况 API 收集信息并进行处理。ETL（提取-转换-加载）工作簿允许您对数据的收集、转换以及发布到 Cloud Cruiser 数据库进行配置。

每个工作簿可以有一个或多个集合。这样，您可以关联来自不同源的信息以补充或增加使用数据。在此示例中，我们将在 Azure 模板工作簿 (_UsageAPI) 中创建一个新工作表_和设置一个新的_集合_以从使用情况 API 导入信息。

![图 3 - 导入到 UsageAPI 工作表的使用情况 API 数据][12]

请注意，此工作簿已有其他工作表要从 Azure 导入服务 (_ImportServices_)，并处理计费 API (_PublishData_) 的消耗信息。

我们要在 _UsageAPI_ 工作表上从使用情况 API 提取信息并进行处理，并在 _PublishData_ 工作表上将该信息与来自计费 API 的消耗数据进行关联。

### 处理来自使用情况 API 的标记信息

在将数据导入工作簿之后，我们将在 _UsageAPI_ 工作表中创建转换步骤以对来自该 API 的信息进行处理。第一步是使用“JSON 拆分”处理器从单个字段中提取标记（当从 API 导入它们时）并为每个标记创建新的字段（部门、项目、所有者和环境）。

![图 4 - 为标记信息创建新字段][13]

请注意，标记信息中缺少“网络”服务（黄色框），但是通过查看 _ResourceGroupName_ 字段，我们可以得知此服务属于相同的资源组。因为我们有相同资源组中其他资源的标记，所以我们可以在以后的过程中使用此信息来将缺少的标记应用到该资源。

下一步是创建查找表，将标记中的信息关联到 _ResourceGroupName_。接下来，将使用此查找表为消耗数据扩充标记信息。

### 将标记信息添加到消耗数据

现在我们可以跳转到 _PublishData_ 表，它对来自计费 API 的消耗信息进行处理，并添加从标记提取的字段。通过查看在上一步中创建的查找表（使用 _ResourceGroupName_ 作为查找关键字），就可以执行该过程。

![图 5 - 使用查找到的信息填充帐户结构][14]

请注意，已经对“网络”服务应用了相应的帐户结构字段，从而解决了缺少标记的问题。对不属于目标资源组的其他资源，我们还使用“其他”字段填充了其帐户结构字段，以在报表中进行区分。

现在，我们只需再完成一个步骤即可发布使用数据。在此步骤中，将把在费率计划中为每个服务定义的相应费率应用到使用情况信息中，然后将计算的费用加载到数据库。

最值得称道的是，您只需要执行该步骤一次。完成该工作簿后，您只需将其添加到计划程序中，然后它将于计划时间以每小时或每日的频率运行。然后，要使数据可视化和可分析，以从云使用数据中获取有意义的信息，您仅需要创建新的报表或自定义现有报表。

### 后续步骤

+ 有关创建 Cloud Cruiser 工作簿和报表的详细说明，请参阅 Cloud Cruiser 联机[文档](http://docs.cloudcruiser.com/)（需要有效登录）。有关 Cloud Cruiser 的更多信息，请联系 [info@cloudcruiser.com](mailto:info@cloudcruiser.com)。
+ 有关 Azure 资源使用情况 API 和 RateCard API 的概述，请参阅[深入了解 Microsoft Azure 资源消耗](/documentation/articles/billing-usage-rate-card-overview)。
+ 请查看 [Azure 计费 REST API 参考](https://msdn.microsoft.com/library/azure/1ea5b323-54bb-423d-916f-190de96c6a3c)了解有关这两种 API（属于 Azure 资源管理器提供的 API 集）的更多信息。
+ 如果您想直接深入了解示例代码，请查看我们在 [Github 上的 Microsoft Azure 计费 API 代码示例](https://github.com/Azure/BillingCodeSamples)。

### 了解详细信息
+ 若要了解有关 Azure 资源管理器的详细信息，请参阅 [Azure 资源管理器概述](/documentation/articles/resource-group-overview)。

<!--Image references-->
[1]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/Create-New-Workbook-Collection.png "图 1 - 创建新集合"
[2]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/Import-Data-From-RateCard.png "图 2 - 从新集合导入数据"
[3]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/Transformation-Steps-Process-RateCard-Data.png "图 3 - 对从 RateCard API 收集的数据进行处理的转换步骤"
[4]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/Publish-RateCard-Data-New-Services-Rates.png "图 4 - 将来自 RateCard API 的数据作为新的服务和费率进行发布"
[5]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/Verify-Azure-Services-And-Pricing1.png "图 5 - 验证新的服务"
[6]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/Verify-Azure-Services-And-Pricing2.png "图 6 - 验证新的费率计划和关联的费率"
[7]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/Transforming-WAP-Normalize-Services.png "图 7 - 对 WAP 数据进行转换以规范化服务"
[8]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/Workbook-Scheduling.png "图 8 - 工作簿计划"
[9]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/Workload-Cost-Simulation-Report.png "图 9 - 工作负荷成本比较方案的示例报表"
[10]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/1_ReportWithTags.png "图 10 - 使用标记的明细报表"
[11]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/2_ResourceGroupsWithTags.png "图 11 - Azure 门户中具有关联标记的资源组"
[12]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/3_ImportIntoUsageAPISheet.png "图 12 - 导入到 UsageAPI 工作表的使用情况 API 数据"
[13]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/4_NewTagField.png "图 13 - 为标记信息创建新字段"
[14]: ./media/billing-usage-rate-card-partner-solution-cloudcruiser/5_PopulateAccountStructure.png "图 14 - 使用查找到的信息填充帐户结构"

<!---HONumber=71-->