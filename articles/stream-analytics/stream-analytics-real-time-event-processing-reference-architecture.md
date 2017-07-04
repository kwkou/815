<properties
    pageTitle="使用流分析事件处理进行实时事件处理 | Azure"
    description="了解如何让一组 Azure 服务通过互操作实现实时事件处理和分析。"
    keywords="实时处理, 事件处理, 参考体系结构"
    services="stream-analytics,event-hubs,storage,sql-database"
    documentationcenter=""
    author="jeffstokes72"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="11af48bc-313c-4527-8c80-91088dc9f3c6"
    ms.service="stream-analytics"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/24/2017"
    wacn.date="03/10/2017"
    ms.author="jeffstok" />  


# 参考体系结构：使用 Azure 流分析进行实时事件处理

通过 Azure 流分析进行实时事件处理的参考体系结构的用途是提供一个通用的蓝图，以便通过 Azure 部署实时平台即服务 (PaaS) 流式处理解决方案。

## 摘要

一直以来，分析解决方案始终基于 ETL（提取、转换、加载）这样的功能以及用于存储要分析数据的数据仓库。由于要求在不管变化，而数据到达速度也越来越快，这个现有模式的使用也已经到了极限。一个解决方案是允许在进行存储之前，在移动的流中分析数据。虽然这不是一项新的功能，但该方法尚未在所有行业类别中广泛采用。

Azure 提供了各种类别的分析技术，支持一系列不同的解决方案和要求。由于提供的产品/服务的多样性，因此选择为端到端解决方案部署何种 Azure 服务并不那么容易。本文旨在介绍各种支持事件流式处理解决方案的 Azure 服务的功能和互操作性。本文还介绍了允许客户充分利用此类方法的某些方案。

## 内容

* 执行摘要
* 实时分析简介
* Azure 中实时数据的价值定位
* 实时分析常见方案
* 体系结构和组件
  * 数据源
  * 数据集成层
  * 实时分析层
  * 数据存储层
  * 呈现/消耗层
* 结束语

**作者：**Charles Feddersen，Microsoft Corporation Data Insights 卓越中心解决方案体系结构部门

**发布时间：**2015 年 1 月

**修订版：**1.0

**下载：**[使用 Azure 流分析进行实时事件处理](http://download.microsoft.com/download/6/2/3/623924DE-B083-4561-9624-C1AB62B5F82B/real-time-event-processing-with-microsoft-azure-stream-analytics.pdf)


## 获取帮助
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=AzureStreamAnalytics)

## 后续步骤

* [Azure 流分析简介](/documentation/articles/stream-analytics-introduction/)
* [Azure 流分析入门](/documentation/articles/stream-analytics-get-started/)
* [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs/)
* [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
* [Azure 流分析管理 REST API 参考](https://docs.microsoft.com/zh-cn/rest/api/streamanalytics/)

 

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description:update meta properties;wording update-->