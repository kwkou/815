<properties
    pageTitle="DocumentDB 门户问题故障排除 | Azure"
    description="找出并解决 DocumentDB Azure 门户预览中的问题。" 
    services="documentdb"
    documentationCenter=""
    authors="mimig1"
    manager="jhubbard"
    editor="monicar"/>

<tags
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="08/29/2016"
    wacn.date="11/21/2016"
    ms.author="mimig"/>  


# Azure DocumentDB 门户故障排除提示

本文介绍如何解决 Azure 门户预览中的 DocumentDB 问题。

## 缺少资源

**症状**：门户边栏选项卡中缺少数据库或集合。

**解决方案**：减少应用程序使用量，以在集合的最大吞吐量配额下运行。

**说明**：门户是一个应用程序，就像任何其他应用程序一样，对 DocumentDB 数据库和集合进行调用。如果当前由于从单独的应用程序进行调用，请求受到限制，门户可能也同样受到限制，导致资源未显示在门户中。若要解决此问题，需解决高吞吐量使用率的原因，然后刷新门户边栏选项卡。有关如何测量和降低吞吐量使用率的信息，请参阅[性能提示](/documentation/articles/documentdb-performance-tips/)一文的[吞吐量](/documentation/articles/documentdb-performance-tips/#measure-rus/)部分。
 
## 页面或边栏选项卡无法加载

**症状**：门户中的页面和边栏选项卡未显示。

**解决方案**：减少应用程序使用量，以在集合的最大吞吐量配额下运行。

**说明**：门户是一个应用程序，就像任何其他应用程序一样，对 DocumentDB 数据库和集合进行调用。如果当前由于从单独的应用程序进行调用，请求受到限制，门户可能也同样受到限制，导致资源未显示在门户中。若要解决此问题，需解决高吞吐量使用率的原因，然后刷新门户边栏选项卡。有关如何测量和降低吞吐量使用率的信息，请参阅[性能提示](/documentation/articles/documentdb-performance-tips/)一文的[吞吐量](/documentation/articles/documentdb-performance-tips/#measure-rus/)部分。

## “添加集合”按钮处于禁用状态

**症状**：在“数据库”边栏选项卡上，“添加集合”按钮处于禁用状态。

**说明**：如果 Azure 订阅已与权益信用额度（如 MSDN 订阅提供的免费信用额度）关联，并且已使用这个月的所有信用额度，则无法在 DocumentDB 中创建任何其他集合。

**解决方案**：从帐户中删除支出限制。

1. 在 Azure 门户预览中，单击跳转栏中的“订阅”，单击与 DocumentDB 数据库关联的订阅，然后在“订阅”边栏选项卡中，单击“管理”。
    ![DocumentDB 提供多个妥善定义的（宽松）一致性模型供你选择](./media/documentdb-portal-troubleshooting/documentdb-change-billing.png)

2. 在新的浏览器窗口中，将看到已没有剩余的信用额度。单击“删除支出限制”按钮可只删除当前计费周期的支出限制或无限期地删除支出限制。然后完成向导以添加或确认信用卡信息。
    ![DocumentDB 提供多个妥善定义的（宽松）一致性模型供你选择](./media/documentdb-portal-troubleshooting/documentdb-remove-spending-limit.png)

 
## 查询浏览器完成时出错

请参阅[查询浏览器故障排除](/documentation/articles/documentdb-query-collections-query-explorer/#troubleshoot/)。

## 监视磁贴中未提供数据

请参阅[监视磁贴故障排除](/documentation/articles/documentdb-monitor-accounts/#troubleshooting/)。

## 文档资源管理器中未返回任何文档

请参阅[文档资源管理器故障排除](/documentation/articles/documentdb-view-json-document-explorer/#troubleshoot/)。

## 后续步骤

如果在门户中仍遇到问题，请发送电子邮件至 [askdocdb@microsoft.com](mailto:askdocdb@microsoft.com) 寻求帮助，或者通过依次单击“浏览”、“帮助 + 支持”、“创建支持请求”来提出支持请求。

<!---HONumber=Mooncake_1010_2016-->
