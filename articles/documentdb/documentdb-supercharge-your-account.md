<properties
    pageTitle="迁移 DocumentDB S1 帐户 | Azure"
    description="通过在 Azure 门户预览中进行一些简单的更改，利用 DocumentDB S1 帐户中增加的吞吐量。"
    services="documentdb"
    author="mimig1"
    manager="jhubbard"
    editor="monicar"
    documentationcenter="" />
<tags
    ms.assetid="6f373fb6-b0d9-4745-b17c-88e8bc5f906a"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/04/2017"
    wacn.date="02/27/2017"
    ms.author="mimig" />  


# 迁移 DocumentDB S1、S2 或 S3 帐户
请执行以下步骤，提升到标准定价层，然后即可利用 Azure DocumentDB 帐户所增加的吞吐量。几乎不需要额外的费用，就可以将现有 S1 帐户的吞吐量从 250 [RU/秒](/documentation/articles/documentdb-request-units/) 增加到 400 RU/秒，甚至更多！ 所有标准帐户均可充分利用根据应用程序需求进行吞吐量伸缩这一功能。可以根据需要随时进行伸缩以达到应用程序所需的吞吐量，而不再需要从预定义的吞吐量选项中进行选择。

## 在 Azure 门户预览中更改为用户定义的性能
1. 在浏览器中导航至 [**Azure 门户预览**](https://portal.azure.cn)。
2. 在左侧菜单中单击“NoSQL \(DocumentDB\)”，或者单击“更多服务”，滚动到“数据库”，然后选择“NoSQL \(DocumentDB\)”。
3. 在“NoSQL \(DocumentDB\)”边栏选项卡中，选择要修改的帐户。
4. 在新边栏选项卡“集合”下的菜单中，单击“缩放”。

      ![“DocumentDB 设置”和“选择定价层”边栏选项卡屏幕截图](./media/documentdb-supercharge-your-account/documentdb-change-performance.png)  

5. 如以上屏幕截图中所示执行以下操作：

 - 在新的边栏选项卡中，使用下拉菜单选择定价层为 S1、S2 或 S3 的集合。
 - 单击“定价层 S1”、“定价层 S2”或“定价层 S3”。
 - 在“选择定价层”边栏选项卡中，单击“标准”，然后单击“选择”保存更改。
   
6. 返回到“缩放”边栏选项卡，可以发现“吞吐量\(RU/s\)”框显示默认值 400，“定价层”已更改为“标准”。可将吞吐量更改为应用程序所需的任何值。建议使用 [DocumentDB Capacity Planner](https://www.documentdb.com/capacityplanner) 确定应用程序所需的[请求单位数](/documentation/articles/documentdb-request-units/)/秒 \(RU/s\)。页面底部的“每月预期账单”区域将自动更新，提供月度费用估算。单击“保存”以保存更改。
      
    ![显示在何处更改吞吐量值的“设置”边栏选项卡屏幕截图](./media/documentdb-supercharge-your-account/documentdb-change-performance-set-thoughput.png)  

7. 集合会根据新的要求进行缩放，并显示成功消息。
   
    ![具有已修改的集合的“数据库”边栏选项卡屏幕截图](./media/documentdb-supercharge-your-account/documentdb-change-performance-confirmation.png)  


有关对用户定义和预定义的吞吐量的更改的详细信息，请参阅博客文章 [DocumentDB: Everything you need to know about using the new pricing options](https://azure.microsoft.com/blog/documentdb-use-the-new-pricing-options-on-your-existing-collections/)（DocumentDB：关于使用新的定价选项所需要了解的一切）。

## 后续步骤

若要详细了解如何通过 DocumentDB 进行数据分区和实现全局缩放，请参阅 [Azure DocumentDB 中的分区和缩放](/documentation/articles/documentdb-partition-data/)。

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: wording update-->