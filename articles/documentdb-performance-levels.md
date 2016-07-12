<properties 
	pageTitle="DocumentDB 的性能级别 | Azure" 
	description="了解 DocumentDB 中的性能级别如何让你能够在每个集合的基础上保留吞吐量。" 
	services="documentdb" 
	authors="johnfmacintyre" 
	manager="jhubbard" 
	editor="monicar" 
	documentationCenter=""/>

<tags 
	ms.service="documentdb" 
	ms.date="05/16/2016" 
	wacn.date="07/04/2016"/>

# DocumentDB 中的性能级别

本文概述了 [Microsoft Azure DocumentDB](/services/documentdb/) 中的性能级别。

阅读本文之后，你将能够回答以下问题：

-	什么是性能级别？
-	数据库帐户的吞吐量是如何保留的？
-	如何使用性能级别工作？
-	性能级别是怎么计费的？

## 性能级别简介

每个在标准帐户下创建的 DocumentDB 集合都会使用相关联的性能级别进行预配。数据库中的每个集合均有不同的性能级别，从而让你能够为经常访问的集合指定更多的吞吐量，为不常访问的集合指定更少的吞吐量。DocumentDB 支持用户定义的性能级别和预定义的性能级别。

每个性能级别都有一个关联的[请求单位 (RU)](http://go.microsoft.com/fwlink/?LinkId=735027) 速率限制。这是将基于集合的性能级别而为其保留的吞吐量，且只能由该集合使用。

<table border="0" cellspacing="0" cellpadding="0">
    <tbody>
        <tr>
        	<td valign="top"><p></p></td>
            <td valign="top"><p>详细信息</p></td>
            <td valign="top"><p>吞吐量限制</p></td>
            <td valign="top"><p>存储限制</p></td>
            <td valign="top"><p>版本</p></td>
 			<td valign="top"><p>API</p></td>            
        </tr>
        <tr>
        	<td valign="top"><p>用户定义的性能</p></td>
            <td valign="top"><p>基于使用情况（以 GB 为单位）按流量计费的存储。</p><p>100 RU/s 单位的吞吐量</p></td>
            <td valign="top"><p>不受限制。默认为每秒 400 - 250,000 请求单位（应请求可更高）</p></td>
            <td valign="top"><p>不受限制。默认为 250 GB（应请求可更高） </p></td>
            <td valign="top"><p>V2</p></td>
 			<td valign="top"><p>API 2015-12-16 及更新版本</p></td>  
        </tr>
        <tr>
        	<td valign="top"><p>预定义性能</p></td>
            <td valign="top"><p>10 GB 保留存储。</p><p>S1 = 250 RU/s、S2 = 1000 RU/s、S3 = 2500 RU/s</p></td>
            <td valign="top"><p>2500 RU/s</p></td>
            <td valign="top"><p>10 GB</p></td>
            <td valign="top"><p>V1</p></td>
 			<td valign="top"><p>任意</p></td>  
        </tr>        
    </tbody>
</table>                


DocumentDB 允许一组丰富的数据库操作，包括查询、利用用户定义的功能 (UDF) 的查询、存储过程和触发器。与这些操作中的每一个关联的处理成本取决于完成操作所需的 CPU、IO 和内存。与考虑和管理硬件资源不同的是，你可以考虑将请求单位作为所需资源的单个措施，以执行各种数据库操作和服务应用程序请求。

可以通过 [Microsoft Azure 门户预览](https://portal.azure.cn)、[REST API](https://msdn.microsoft.com/library/azure/mt489078.aspx) 或任意 [DocumentDB SDK](https://msdn.microsoft.com/library/azure/dn781482.aspx) 创建集合。DocumentDB API 让你能够指定集合的性能级别。

> [AZURE.NOTE] 集合的性能级别可以通过 API 或 [Microsoft Azure 门户预览](https://portal.azure.cn/)进行调整。性能级别更改应在 3 分钟内完成。

## 设置集合的性能级别
创建集合之后，基于指定的性能级别的 RU 的完整分配为集合所保留。

注意，同时具有用户定义的性能级别和预定义的性能级别，DocumentDB 可基于保留的吞吐量进行运作。通过创建集合，应用程序已保留吞吐量且不论实际使用了多少该吞吐量，都会对已保留的吞吐量计费。对于用户定义的性能级别，存储基于消耗量按流量计费，但是对于预定义性能级别，将在创建集合时保留 10 GB 的存储。

在创建集合后，可以通过 DocumentDB SDK 或通过 Azure 经典管理门户对性能级别进行修改。

> [AZURE.IMPORTANT] DocumentDB 标准集合以小时计费，且你创建的每个集合都将以最低一小时的使用量计费。

如果在一小时内调整集合的性能级别，将以这一小时内最高性能级别集进行计费。例如，如果你在上午 8:53 提高了集合的性能级别，则将从上午 8:00 开始，向你收取新级别的费用。同样，如果在上午 8:53 降低了性能级别，新费率将在上午 9:00 生效。

请求单位基于性能级别设置而为每个集合保留。请求单位消耗以每秒速率评估。将限制集合上超出预配请求单位速率（或性能级别）的应用程序，直到速率降低到集合的保留级别之下。如果你的应用程序要求更高级别的吞吐量，可以提高每个集合的性能级别。

> [AZURE.NOTE] 当你的应用程序超出一个或多个集合的性能级别时，将按每个集合限制请求。这意味着一些应用程序请求可能成功，而另一些则可能受限制。建议添加

## 使用性能级别工作
DocumentDB 集合允许你根据应用程序的查询模式和性能需求来对数据进行分组。利用 DocumentDB 的自动索引和查询支持，在同一集合内并置异类文档是相当常见的做法。决定是否应该使用单独集合的关键注意事项包括：

- 查询 – 集合是查询执行的范围。如果需要在一组文档间进行查询，最有效的读取模式就是在单个集合中并置文档。
- 事务 – 所有事务都处于单个集合的范围内。如果你有文档必须在单个存储过程或触发器内进行更新，则它们必须存储在相同的集合内。更具体地说，集合内的分区键就是事务边界。有关更多详细信息，请参阅 [DocumentDB 中的分区](/documentation/articles/documentdb-partition-data/)。
- 性能隔离 – 一个集合具有关联的性能级别。这可确保每个集合通过保留的 RU 都有可预测的性能。可以根据访问频率将数据分配到具有不同性能级别的不同集合。

> [AZURE.IMPORTANT] 重要的是，要了解将基于你的应用程序所创建的集合数，以全标准费率向你收费。

建议你的应用程序利用小数量的集合，除非你有大的存储或吞吐量要求。确保你已清楚理解创建新集合的应用程序模式。你可以选择将集合创建保留为在应用程序外进行处理的管理操作。同样，调整集合的性能级别将更改集合计费的小时费率。如果你的应用程序对集合性能级别进行动态调整，那么你应监视它们。

## 使用 Azure 门户预览更改性能级别

在管理集合的性能级别时，Azure 门户预览是你可用的一个选项。按照下列步骤在 Azure 门户预览中从使用预定义性能级别更改为使用用户定义的性能级别，或者观看此 75 秒的 [Channel 9 视频](https://channel9.msdn.com/Blogs/AzureDocumentDB/ChangeDocumentDBCollectionPerformance)。有关对定价选项的更改的详细信息，请参阅博客文章 [DocumentDB：关于使用新的定价选项所需要了解的一切](https://azure.microsoft.com/blog/documentdb-use-the-new-pricing-options-on-your-existing-collections/)。

1. 从你的浏览器中导航至 [**Azure 门户预览**](https://portal.azure.cn)。
2. 从左侧的跳转栏单击“浏览”。
3. 在“浏览”中心中，单击“筛选方式”标签下的“DocumentDB 帐户”。
4. 在“DocumentDB 帐户”边栏选项卡中，单击包含所需集合的 DocumentDB 帐户。
5. 在“DocumentDB 帐户”边栏选项卡中，向下滚动至“数据库”可重用功能区并单击包含所需集合的数据库。 
6. 在新打开的“数据库”边栏选项卡中，向下滚动至“集合”可重用功能区并选择所需集合。
7. 在“管理集合”边栏选项卡中，单击“定价层”。

    ![Azure DocumentDB 的“管理集合”和“选择你的定价层”边栏选项卡的屏幕截图，显示在何处更改集合的定价层][1]

8. 在“选择你的定价层”边栏选项卡中，单击“标准”。

9. 在“选择你的定价层”边栏选项卡中，单击“选择”。

10. 回到“管理集合”边栏选项卡中，“定价层”被更改为了“标准”，且显示了“吞吐量 (RU/s)”框。

    将“吞吐量”框中的值更改为 400 到 10,000 [请求单位](/documentation/articles/documentdb-request-units/)/秒 (RU/s) 之间的值。页面底部的“定价摘要”将自动更新以提供月度成本估算。

    ![显示在何处更改集合的吞吐量值的“管理集合”边栏选项卡的屏幕截图][2]

9. 在“管理集合”边栏选项卡上，单击“确定”以将你的集合更新为用户定义的性能。

如果你确定需要更多吞吐量（大于 10,000 RU/s）或更多存储（大于 10GB），可以创建分区集合。若要创建分区集合，请参阅[创建集合](/documentation/articles/documentdb-create-collection/)。

>[AZURE.NOTE] 更改集合的性能级别可能会花费 2 分钟。

## 使用 .NET SDK 更改性能级别

另一个更改集合的性能级别的选项便是通过我们的 SDK 进行操作。本节只涵盖使用我们的 [.NET SDK](https://msdn.microsoft.com/library/azure/dn948556.aspx) 更改集合的性能级别，但对于其他的 [SDK](https://msdn.microsoft.com/library/azure/dn781482.aspx)，过程也是相似的。如果你对我们的 .NET SDK 还不熟悉的话，请访问我们的[入门教程](/documentation/articles/documentdb-get-started/)。

此为将服务吞吐量更改为每秒 50,000 请求单位的代码片段：

	//Fetch the resource to be updated
	Offer offer = client.CreateOfferQuery()
		              .Where(r => r.ResourceLink == collection.SelfLink)    
		              .AsEnumerable()
		              .SingleOrDefault();
	                          
	// Set the throughput to 5000 request units per second
	offer = new OfferV2(offer, 5000);
	                    
	//Now persist these changes to the database by replacing the original resource
	await client.ReplaceOfferAsync(offer);

	// Set the throughput to S2
	offer = new Offer(offer);
	offer.OfferType = "S2";
	                    
	//Now persist these changes to the database by replacing the original resource
	await client.ReplaceOfferAsync(offer);



> [AZURE.NOTE] 预配为每秒 10,000 请求单位以下的集合可以于任何时间在用户定义的吞吐量服务（S1、S2、S3）和预定义吞吐量服务之间进行迁移。预配为每秒 10,000 请求单位以上的集合不能转换为预定义吞吐量级别。

请访问 [MSDN](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.documentclient.aspx) 以查看其他示例并了解更多有关我们的服务方法的信息：

- [**ReadOfferAsync**](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.documentclient.readofferasync.aspx)
- [**ReadOffersFeedAsync**](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.documentclient.readoffersfeedasync.aspx)
- [**ReplaceOfferAsync**](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.documentclient.replaceofferasync.aspx)
- [**CreateOfferQuery**](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.linq.documentqueryable.createofferquery.aspx) 

## 后续步骤

若要了解更多有关 Azure DocumentDB 的定价和管理数据的信息，请浏览以下资源：
 
- [管理 DocumentDB 容量](/documentation/articles/documentdb-manage/) 
- [对 DocumentDB 中的数据进行建模](/documentation/articles/documentdb-modeling-data/)
- [对 DocumentDB 中的数据进行分区](/documentation/articles/documentdb-partition-data/)
- [请求单位](http://go.microsoft.com/fwlink/?LinkId=735027)

若要了解有关 DocumentDB 的详细信息，请参阅 Azure DocumentDB [文档](/documentation/services/documentdb/)。

[1]: ./media/documentdb-performance-levels/documentdb-change-collection-performance7-9.png
[2]: ./media/documentdb-performance-levels/documentdb-change-collection-performance10-11.png
<!---HONumber=Mooncake_0627_2016-->