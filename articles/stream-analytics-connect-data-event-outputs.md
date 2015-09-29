<properties 
	pageTitle="创建流分析输出 | Windows Azure" 
	description="了解如何连接到用于流分析解决方案的输出目标并对其进行配置。" 
	documentationCenter="" 
	services="stream-analytics"
	authors="jeffstokes72" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="stream-analytics" 
	ms.date="08/19/2015" 
	wacn.date="09/15/2015"/>

# 创建流分析输出

## 了解流分析输出 ##
---
创建流分析作业时，其中一个考虑因素是如何使用作业的输出。数据转换的使用者如何查看流分析作业的结果？ 他们会使用什么工具来分析输出？ 是否必须进行数据保留或数据仓库操作？

Azure 流分析提供了七种不同的方法来存储和查看作业输出。SQL 数据库、Blob 存储、事件中心、Service Bus 队列、Service Bus 主题、Power BI 和表存储。这样可以轻松地查看作业输出，并可灵活地使用和存储作业输出，以便进行数据仓库操作和其他操作。

## 使用 SQL 数据库作为输出 ##
---
可以将 SQL 数据库用作本质上为关系型数据的数据的输出，也可以将其用于所依赖的内容在关系数据库中托管的应用程序。有关 Azure SQL 数据库的详细信息，请参阅 [Azure SQL 数据库](/documentation/services/sql-database/)。

### Parameters ###

开始使用 SQL 数据库时，需要提供有关 SQL 数据库的以下信息：

1. 服务器名称
2. 数据库名称
3. 有效的用户名/密码组合
4. 输出表名。此表必须已存在，因为作业不会创建它。

### 添加 SQL 数据库作为输出 ###

![graphic1][graphic1]

转到作业的输出选项卡，单击**“添加输出”**命令，然后单击“下一步”。

![graphic2][graphic2]

选择**“SQL 数据库”**作为输出。

![graphic3][graphic3]

在下一页上输入数据库信息。输出别名是在查询中使用的友好名称，用于将查询输出定向到此数据库。在仅提供一个输出的情况下，默认别名为“输出”。

![graphic4][graphic4]

如果数据库与流分析作业存在于同一个订阅中，则会提供选择“从当前订阅中使用 SQL 数据库”的选项。然后，从下拉列表中选择数据库。

![graphic5][graphic5]

单击“下一步”以添加此输出。此时两个操作都会开始，第一个操作是添加输出。

![graphic6][graphic6]

第二个操作是对连接进行测试。Azure 流分析会尝试连接到 SQL 数据库，并验证输入的凭据是否正确以及表是否可以访问。

## 使用 Blob 存储作为输出 ##
---
如需 Azure Blob 存储及其用法的简介，请参阅文档：[如何使用 Blob](/documentation/articles/storage-dotnet-how-to-use-blobs)。

### Parameters ###

开始使用 Blob 存储输出时，需要提供以下信息：

1. 如果存储帐户与流式处理作业不属于同一个订阅，则会显示相应的字段，要求你提供“存储帐户名称”和“存储帐户密钥”。
2. 容器名称。
3. 文件名前缀。
4. 为数据使用何种序列化格式（Avro、CSV、JSON）。

### 添加 Blob 存储作为输出 ###

选择到 Blob 存储的输出。

![graphic20][graphic20]

然后提供详细信息，如下所示：

![graphic21][graphic21]

## 使用事件中心作为输出 ##
---
### 概述 ###
 
事件中心是高度可伸缩的事件引入器，通常是最常用的进行流分析数据引入的方法。另外，事件中心可以稳定地处理大量的事件，因此尤其适合作业输出。当流分析作业的输出将要成为另一个流式处理作业的输入时，可以将事件中心用作输出。有关事件中心的更多详细信息，请访问位于[事件中心](/documentation/services/event-hubs/ "事件中心")的门户。
 
### Parameters ###

对事件中心数据流进行配置时，需要使用几个参数。

1. Service Bus 命名空间：事件中心的 Service Bus 命名空间。Service Bus 命名空间是包含一组消息传递实体的容器。在创建新的事件中心时，还会创建 Service Bus 命名空间。 
2. 事件中心名称：事件中心的名称。该名称是在创建新的事件中心时指定的名称。 
3. 事件中心策略名称：访问事件中心时适用的共享访问策略的名称。可以在“配置”选项卡上为事件中心配置共享访问策略。每个共享访问策略都会有名称、所设权限以及生成的访问密钥。
4. 事件中心策略密钥：访问事件中心时适用的共享访问策略的主密钥或辅助密钥。  
5. 分区键列：事件中心输出的可选参数。此列包含事件中心输出的分区键。

### 添加事件中心作为输出 ###

1. 单击页面顶部的**“输出”**，然后单击**“添加输出”**。选择“事件中心”作为输出选项，然后单击窗口底部右侧的按钮。

    ![graphic38][graphic38]

2. 将相关信息提供到作为输出的字段中，完成时单击窗口底部右侧的按钮即可继续。

    ![graphic36][graphic36]

3. 验证事件序列化格式、编码类型和格式是否已设置为适当的值，然后单击复选框以完成操作。

    ![graphic37][graphic37]

## 使用 Power BI 作为输出 ##
---
### 概述 ###
Power BI 可以用作流分析作业的输出，以便为流分析用户提供丰富的可视化体验。此功能可用于操作仪表板、生成报告以及进行指标驱动型报告。有关 Power BI 的详细信息，请访问 [Power BI](https://powerbi.microsoft.com/) 站点。

### Parameters ###

有几个参数是配置 Power BI 输出所必需的。

1. 输出别名 – 任何可以作为友好名称且容易引用的输出别名。如果决定为某个作业设置多个输出，则此输出别名特别有用。在那样的情况下，可以在你的查询中引用此别名。例如，可以使用输出别名值“OutPbi”。
2. 数据集名称 - 提供数据集名称，供 Power BI 输出使用。例如，使用“pbidemo”。
2. 表名 - 在 Power BI 输出数据集下提供表名。例如，使用“pbidemo”。**目前，流分析作业的 Power BI 输出只能在数据集中设置一个表。**

### 添加 Power BI 作为输出 ###

1.  单击页面顶部的**“输出”**，然后单击**“添加输出”**。选择 Power BI 作为输出选项。

    ![graphic22][graphic22]

2.  将显示如下所示的屏幕。

    ![graphic23][graphic23]

3.  在此步骤中，提供用于授权 Power BI 输出的工作或学校帐户。如果你还没有注册 Power BI，请选择**“立即注册”**。
4.  接下来，将显示如下所示的屏幕。

    ![graphic24][graphic24]


>	[AZURE.NOTE] 不得在 Power BI 仪表板中显式创建数据集和表。当作业已启动并且作业开始向 Power BI 发送输出时，系统将自动填充数据集和表。请注意，如果作业查询未返回任何结果，则不会创建数据集和表。另请注意，如果 Power BI 已具有与此流分析作业中提供的数据集和表名称相同的数据集和表，则现有数据将被覆盖。

*	单击**“确定”**和**“测试连接”**。现在，输出配置已完成。


## 使用 Azure 表存储进行输出 ##
---
表存储提供了具有高可用性且可大规模伸缩的存储，因此应用程序可以自动伸缩以满足用户需求。表存储是 Microsoft 推出的 NoSQL 键/属性存储，适用于对架构的约束性较少的结构化数据。Azure 表存储可用于持久地存储数据，方便进行高效的检索。有关 Azure 表存储的详细信息，请访问 [Azure 表存储](/documentation/articles/storage-introduction)。

### Parameters ###

开始使用 Azure 表存储时，需要提供以下信息：

1. 存储帐户名称（如果此存储与流式处理作业属于不同的订阅）。
2. 存储帐户密钥（如果此存储与流式处理作业属于不同的订阅）。
3. 输出表名称（如果不存在，则会创建一个）。
4. 分区键（必需）。
5. 行键

若要更好地设计分区键和行键，请参阅下面的文章：[为 Azure 表存储设计可扩展分区策略](https://msdn.microsoft.com/zh-cn/library/azure/hh508997.aspx)。

### 添加 Azure 表存储作为输出 ###

![graphic11][graphic11]

转到作业的输出选项卡，单击**“添加输出”**命令，然后单击“下一步”。

![graphic12][graphic12]

选择**“表存储”**作为输出。

![graphic13][graphic13]

在下一页上输入 Azure 表信息。输出别名是可以用在查询中的名称，目的是将查询输出定向到此表。

![graphic14][graphic14]

![graphic15][graphic15]

批处理大小是进行批处理操作的记录数。通常情况下，默认值对于大多数作业来说已经足够。若要修改此设置，请参阅[表批处理操作规范](https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.table.tablebatchoperation.aspx)以获取详细信息。

如果在同一个将用于创建作业的订阅中存在 Azure 存储帐户，则可选择“使用当前订阅中的存储帐户”，然后从下拉菜单中选择“存储帐户”。

单击“下一步”以添加此输出。此时两个操作都会开始，第一个操作是添加输出。

![graphic16][graphic16]

第二个操作是对连接进行测试。Azure 流分析会尝试连接到该存储帐户，并验证输入的凭据是否正确。

## Service Bus 队列 ##
---
### Service Bus 队列概念简介 ###
Service Bus 队列为一个或多个竞争使用方提供先入先出 (FIFO) 消息传递方式。通常情况下，接收方会按照消息添加到队列中的临时顺序来接收并处理消息，并且每条消息仅由一个消息使用方接收并处理。

有关 Service Bus 队列的详细信息，请参阅 [Service Bus 队列、主题和订阅](https://msdn.microsoft.com/zh-cn/library/azure/hh367516.aspx "Service Bus 队列、主题和订阅")和 [Service Bus 队列简介](http://blogs.msdn.com/b/appfabric/archive/2011/05/17/an-introduction-to-service-bus-queues.aspx "Service Bus 队列简介")。

### Parameters ###

开始使用 Service Bus 队列输出时，需要提供以下信息：

1. 输出别名 – 任何可以作为友好名称且容易引用的输出别名。如果决定为某个作业设置多个输出，则此输出别名特别有用。在那样的情况下，可以在作业查询中引用此别名。
2. 命名空间和 Service Bus 名称。
3. 队列名称 - 队列是消息传送实体，类似于事件中心和主题。之所以设计队列，是为了从多个不同的设备和服务收集事件流。在创建队列时，还会为其提供特定的名称。
4. 为数据使用何种序列化格式（Avro、CSV、JSON）。

### 添加 Service Bus 队列作为输出 ###

![graphic31][graphic31]

然后提供如下所示的详细信息，并选择“下一步”。

![graphic32][graphic32]

验证你的数据格式和序列化是否正确。

![graphic33][graphic33]

## Service Bus 主题 ##
---
### Service Bus 主题概念简介 ###
Service Bus 队列提供的是一对一的从发送方到接收方的通信方法，而 Service Bus 主题提供的则是一对多形式的通信。

有关 Service Bus 主题的详细信息，请参阅 [Service Bus 队列、主题和订阅](https://msdn.microsoft.com/zh-cn/library/azure/hh367516.aspx "Service Bus 队列、主题和订阅")和 [Service Bus 主题简介](http://blogs.msdn.com/b/appfabric/archive/2011/05/25/an-introduction-to-service-bus-topics.aspx "Service Bus 主题简介")。

### Parameters ###

开始使用 Service Bus 主题输出时，需要提供以下信息：

1. 输出别名 – 任何可以作为友好名称且容易引用的输出别名。如果决定为某个作业设置多个输出，则此输出别名特别有用。在那样的情况下，可以在你的查询中引用此别名。
2. 命名空间和 Service Bus 名称。
3. 主题名称 - 主题是消息传送实体，类似于事件中心和队列。之所以设计队列，是为了从多个不同的设备和服务收集事件流。在创建主题时，还会为其提供特定的名称。发送到主题的消息在创建订阅后才会提供给用户，因此请确保主题下存在一个或多个订阅。
4. 为数据使用何种序列化格式（Avro、CSV、JSON）。

### 添加 Service Bus 主题作为输出 ###

选择传送到 Service Bus 主题的输出。

![graphic34][graphic34]

然后提供如下所示的详细信息，并选择“下一步”。

![graphic35][graphic35]

验证你的数据格式和序列化是否正确。

![graphic33][graphic33]


## 获取帮助
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=AzureStreamAnalytics)

## 后续步骤

- [Azure 流分析简介](/documentation/articles/stream-analytics-introduction)
- [Azure 流分析入门](/documentation/articles/stream-analytics-get-started)
- [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs)
- [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
- [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)




[graphic1]: ./media/stream-analytics-connect-data-event-outputs/1-stream-analytics-connect-data-event-input-output.png
[graphic2]: ./media/stream-analytics-connect-data-event-outputs/2-stream-analytics-connect-data-event-input-output.png
[graphic3]: ./media/stream-analytics-connect-data-event-outputs/3-stream-analytics-connect-data-event-input-output.png
[graphic4]: ./media/stream-analytics-connect-data-event-outputs/4-stream-analytics-connect-data-event-input-output.png
[graphic5]: ./media/stream-analytics-connect-data-event-outputs/5-stream-analytics-connect-data-event-input-output.png
[graphic6]: ./media/stream-analytics-connect-data-event-outputs/6-stream-analytics-connect-data-event-input-output.png
[graphic7]: ./media/stream-analytics-connect-data-event-outputs/7-stream-analytics-connect-data-event-input-output.png
[graphic8]: ./media/stream-analytics-connect-data-event-outputs/8-stream-analytics-connect-data-event-input-output.png
[graphic9]: ./media/stream-analytics-connect-data-event-outputs/9-stream-analytics-connect-data-event-input-output.png
[graphic10]: ./media/stream-analytics-connect-data-event-outputs/10-stream-analytics-connect-data-event-input-output.png
[graphic11]: ./media/stream-analytics-connect-data-event-outputs/11-stream-analytics-connect-data-event-input-output.png
[graphic12]: ./media/stream-analytics-connect-data-event-outputs/12-stream-analytics-connect-data-event-input-output.png
[graphic13]: ./media/stream-analytics-connect-data-event-outputs/13-stream-analytics-connect-data-event-input-output.png
[graphic14]: ./media/stream-analytics-connect-data-event-outputs/14-stream-analytics-connect-data-event-input-output.png
[graphic15]: ./media/stream-analytics-connect-data-event-outputs/15-stream-analytics-connect-data-event-input-output.png
[graphic16]: ./media/stream-analytics-connect-data-event-outputs/16-stream-analytics-connect-data-event-input-output.png
[graphic17]: ./media/stream-analytics-connect-data-event-outputs/17-stream-analytics-connect-data-event-input-output.png
[graphic18]: ./media/stream-analytics-connect-data-event-outputs/18-stream-analytics-connect-data-event-input-output.png
[graphic19]: ./media/stream-analytics-connect-data-event-outputs/19-stream-analytics-connect-data-event-input-output.png
[graphic20]: ./media/stream-analytics-connect-data-event-outputs/20-stream-analytics-connect-data-event-input-output.png
[graphic21]: ./media/stream-analytics-connect-data-event-outputs/21-stream-analytics-connect-data-event-input-output.png
[graphic22]: ./media/stream-analytics-connect-data-event-outputs/22-stream-analytics-connect-data-event-input-output.png
[graphic23]: ./media/stream-analytics-connect-data-event-outputs/23-stream-analytics-connect-data-event-input-output.png
[graphic24]: ./media/stream-analytics-connect-data-event-outputs/24-stream-analytics-connect-data-event-input-output.png
[graphic25]: ./media/stream-analytics-connect-data-event-outputs/25-stream-analytics-connect-data-event-input-output.png
[graphic26]: ./media/stream-analytics-connect-data-event-outputs/26-stream-analytics-connect-data-event-input-output.png
[graphic27]: ./media/stream-analytics-connect-data-event-outputs/27-stream-analytics-connect-data-event-input-output.png
[graphic28]: ./media/stream-analytics-connect-data-event-outputs/28-stream-analytics-connect-data-event-input-output.png
[graphic29]: ./media/stream-analytics-connect-data-event-outputs/29-stream-analytics-connect-data-event-input-output.png
[graphic30]: ./media/stream-analytics-connect-data-event-outputs/30-stream-analytics-connect-data-event-input-output.png
[graphic31]: ./media/stream-analytics-connect-data-event-outputs/31-stream-analytics-connect-data-event-input-output.png
[graphic32]: ./media/stream-analytics-connect-data-event-outputs/32-stream-analytics-connect-data-event-input-output.png
[graphic33]: ./media/stream-analytics-connect-data-event-outputs/33-stream-analytics-connect-data-event-input-output.png
[graphic34]: ./media/stream-analytics-connect-data-event-outputs/34-stream-analytics-connect-data-event-input-output.png
[graphic35]: ./media/stream-analytics-connect-data-event-outputs/35-stream-analytics-connect-data-event-input-output.png
[graphic36]: ./media/stream-analytics-connect-data-event-outputs/36-stream-analytics-connect-data-event-input-output.png
[graphic37]: ./media/stream-analytics-connect-data-event-outputs/37-stream-analytics-connect-data-event-input-output.png
[graphic38]: ./media/stream-analytics-connect-data-event-outputs/38-stream-analytics-connect-data-event-input-output.png

<!---HONumber=69-->