<properties
   pageTitle="使用 SQL 数据仓库构建集成解决方案 | Windows Azure"
   description="用于集成 SQL 数据仓库的工具以及提供相应解决方案的合作伙伴。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="09/22/2015"
   wacn.date="01/20/2016"/>

#在 SQL 数据仓库中利用其他服务
除了本身的核心功能以外，SQL 数据仓库还允许用户利用 Azure 中的其他许多服务。具体而言，我们目前已采取多种措施深度集成了以下服务：

+ Power BI
+ Azure 数据工厂
+ Azure 机器学习
+ Azure 流分析

我们正努力连接多个跨 Azure 生态系统的服务。

##Power BI
借助 Power BI 集成，你可以利用 SQL 数据仓库的计算能力以及 Power BI 的动态报告和视觉效果。Power BI 集成当前包括：

+ **直接连接**：使用逻辑下推与 SQL 数据仓库建立更高级的连接。它提供更快且更大规模的分析。
+ **在 Power BI 中打开**：“在 Power BI 中打开”按钮将实例信息传递给 Power BI，以建立更紧密的连接。 

有关详细信息，请参阅[与 Power BI 集成](/documentation/articles/sql-data-warehouse-integrate-power-bi)或 [Power BI 文档](http://blogs.msdn.com/b/powerbi/archive/2015/06/24/exploring-azure-sql-data-warehouse-with-power-bi.aspx)。

##Azure 数据工厂
Azure 数据工厂为用户提供一个托管平台，用于创建复杂的提取-加载管道。SQL 数据仓库与 Azure 数据工厂的集成包括以下功能：

+ **存储过程**：协调 SQL 数据仓库上存储过程的运行。

有关详细信息，请参阅[与 Azure 数据工厂集成](/documentation/articles/sql-data-warehouse-integrate-azure-data-factory)或 [Azure 数据工厂文档](/documentation/services/data-factory)。

##Azure 机器学习
Azure 机器学习是完全托管的分析服务，可让用户创建利用大量预测工具的复杂模型。支持将 SQL 数据仓库用作模型的源和目标，这些模型具有以下功能：

+ **读取数据**：使用 T-SQL 针对 SQL 数据仓库大规模驱动模型。 
+ **写入数据**：将任一模型中的更改提交回到 SQL 数据仓库。

有关详细信息，请参阅[与 Azure 机器学习集成](/documentation/articles/sql-data-warehouse-integrate-azure-machine-learning)或 [Azure 机器学习文档](/documentation/services/machine-learning)。

##Azure 流分析
Azure 流分析是复杂、完全托管的基础结构，用于处理和使用从 Azure 事件中心生成的事件数据。通过与 SQL 数据仓库集成，可以有效地处理流数据，并将其与关系数据一起存储以实现更深入、更高级的分析。

+ **作业输出**：将流分析作业的输出直接发送到 SQL 数据仓库。

有关详细信息，请参阅[与 Azure 流分析集成](/documentation/articles/sql-data-warehouse-integrate-azure-stream-analytics)或 [Azure 流分析文档](/documentation/services/stream-analytics)。

<!--Image references-->

<!--Article references-->
[development overview]: /documentation/articles/sql-data-warehouse-overview-develop

[Azure Data Factory]: /documentation/articles/sql-data-warehouse-integrate-azure-data-factory
[Azure Machine Learning]: /documentation/articles/sql-data-warehouse-integrate-azure-machine-learning
[Azure Stream Analytics]: /documentation/articles/sql-data-warehouse-integrate-azure-stream-analytics
[Power BI]: /documentation/articles/sql-data-warehouse-integrate-power-bi
[Partners]: /documentation/articles/sql-data-warehouse-integrate-solution-partners

<!--MSDN references-->

<!--Other Web references-->

<!---HONumber=Mooncake_1207_2015-->
