<properties
	pageTitle="请求提高 DocumentDB 帐户限制| Azure"
	description="了解如何请求对 DocumentDB 限制做出调整，例如允许的集合数、存储的过程数和查询子句数。"
	services="documentdb"
	authors="AndrewHoh"
	manager="jhubbard"
	editor="monicar"
	documentationCenter=""/>

<tags
	ms.service="documentdb"
	ms.date="04/07/2016"
	wacn.date="06/28/2016"/>

# 请求提高 DocumentDB 帐户限制

[Azure DocumentDB](/services/documentdb/) 具有一组默认限制和配额强制要求。可联系 Azure 技术支持部门来调整配额。本文演示了如何请求提高帐户限制。

阅读本文之后，你将能够回答以下问题：

-	可联系 Azure 技术支持部门来调整配额来调整哪些 DocumentDB 帐户配额？
-	我如何请求对 DocumentDB 帐户配额进行调整？

##<a id="Quotas"></a> DocumentDB 帐户配额

下表描述了 DocumentDB 配额。可联系 Azure 技术支持部门调整带星号 (*) 的配额：

[AZURE.INCLUDE [azure-documentdb-limits](../includes/azure-documentdb-limits.md)]


##<a id="RequestQuotaIncrease"></a> 请求配额调整
以下步骤演示了如何请求配额调整。

1. 在 [Azure 门户预览](https://portal.azure.cn)中，单击**浏览**，然后单击**帮助与支持**。

	![启动帮助与支持的屏幕截图](media/documentdb-increase-limits/helpsupport.png)

2. 在**帮助与支持**边栏选项卡上，单击**获取支持**。

	![创建支持票证的屏幕截图](media/documentdb-increase-limits/getsupport.png)

3. 在**新支持请求**边栏选项卡上，单击**基本**。接下来，将**问题类型**设置为**配额**，将**订阅**设置为托管你的 DocumentDB 帐户的订阅，将**服务**设置为 **DocumentDB**，并将**支持计划**设置为**配额支持-包括**。最后单击“下一步”。

	![支持票证请求类型的屏幕截图](media/documentdb-increase-limits/supportrequest1.png)

4. 在**问题**边栏选项卡，选择严重级别。将**问题类型**设置为 **DocumentDB**，并在**详细信息**中包含关于你的配额增加的信息。单击**“下一步”**。

	![支持票证订阅选取器的屏幕截图](media/documentdb-increase-limits/supportrequest2.png)

5. 最后，在**联系信息**边栏选项卡中填写你的联系信息。

	![支持票证资源选取器的屏幕截图](media/documentdb-increase-limits/supportrequest3.png)

创建支持票证后，你会通过电子邮件收到支持请求编号。你还可以通过单击**帮助与支持**边栏选项卡上的**管理支持请求**来查看支持请求。

![支持请求边栏选项卡的屏幕截图](./media/documentdb-increase-limits/supportrequest4.png)


##<a name="NextSteps"></a>后续步骤
- 若要了解有关 DocumentDB 的详细信息，请单击[此处](http://azure.com/docdb)。

<!---HONumber=Mooncake_0425_2016-->