<properties 
	pageTitle="Azure 上的应用程序体系结构" 
	description="包括常见设计模式的体系结构概述" 
	services="" 
	documentationCenter="" 
	authors="Rboucher" 
	manager="jwhit" 
	editor="mattshel"/>

<tags 
	ms.service="multiple" 
	ms.date="10/16/2015" 
	wacn.date="12/17/2015"/>

#Azure 上的应用程序体系结构
用于构建使用 Azure 的应用程序的资源。这包括一些工具，帮助您画图以直观地描述软件系统。



##Azure 体系结构设计模式
Microsoft 发布一系列体系结构设计模式，以帮助您编写自己的自定义设计。这些模式都应是简洁的体系结构指南，可以按顺序组合在一起，为如何充分利用 Azure 平台提供指导，以满足您组织的业务需求。


[概述](/documentation/articles/azure-architectures-cpif-overview) -
[混合网络](/documentation/articles/azure-architectures-cpif-infrastructure-hybrid-networking) -
[异地批处理](/documentation/articles/azure-architectures-cpif-foundation-offsite-batch-processing-tier) -
[多站点数据层](/documentation/articles/azure-architectures-cpif-foundation-multi-site-data-tier) -
[全局负载平衡的 Web 层](/documentation/articles/azure-architectures-cpif-foundation-global-load-balanced-web-tier) -
[Azure 搜索层](/documentation/articles/azure-architectures-cpif-foundation-azure-search-tier)
 
每个模式包含
 
- 一份服务说明
- 一份利用此模式所需的 Azure 服务列表
- 若干体系结构图
- 若干体系结构依赖项
- 一些设计限制或可能会影响该模式的注意事项
- 若干接口和终结点
- 若干反模式
- 高级别体系结构方面的关键注意事项包括可用性和复原能力、使用了服务的复合 SLA、规模和性能、成本和操作方面的注意事项。

![Azure 体系结构设计模式](./media/architecture-overview/AzureArchPatterns.jpg)


##设计模式海报
Microsoft Patterns and Practices 已发布了[云设计模式](http://msdn.microsoft.com/zh-cn/library/dn568099.aspx)一书，在 MSDN 上可用，也可以 PDF 格式下载。此外还提供了一张可用的大画幅海报，列出了所有的模式。

![模式与实践云模式海报](./media/architecture-overview/PnPPatternPosterThumb.jpg)



##Microsoft 体系结构认证课程

Microsoft 刚刚发布了支持 Microsoft 认证考试 70-534 的新的体系结构课程。在 [EDX.ORG 上免费提供](https://www.edx.org/course/architecting-microsoft-azure-solutions-microsoft-dev205x)。使用新的 [3D Blueprint Visio Template](#3d-blueprint-visio-template)。

![Microsoft 体系结构认证课程](./media/architecture-overview/EDXCourse.png)


##Microsoft 体系结构蓝图

Microsoft 发布了一套高级别的 [体系结构蓝图](http://aka.ms/azblueprints)，介绍如何使用 Microsoft 产品构建特定类型的系统。

每个蓝图包含一个

- 平面的基于**2D Visio** 2003 的文件，供你下载和修改 
- 彩色 **3D perspective PDF** 文件为较少的技术受众介绍蓝图
- **Video** 通过 3D 版本运行。 

该蓝图使用[云和企业符号集](#symbol-and-icon-sets)。

![Microsoft 体系结构蓝图 3D 图](./media/architecture-overview/BluePrintThumb.jpg)



##3D Blueprint Visio Template

3D 版 [Microsoft 体系结构蓝图](http://aka.ms/azblueprints)最初在非 Microsoft 工具中创建。新的 Visio 2013（和更高版本）模板于 2015 年 8 月 5 日发布，作为 [Microsoft 体系结构认证课程的一部分，分发在 EDX.ORG 上](#microsoft-architecture-certification-course)。

该模板在本课程外也可用。

- 下载 [3D Blueprint Visio Template](http://aka.ms/3DBlueprintTemplate)
- 下载与 3D 模板配合使用的[云和企业符号](#drawing-symbol-and-icon-sets) 

提供反馈，或者需要提培训材料无法答复的具体问题，请发邮件至 [CnESymbols@microsoft.com](mailto:CnESymbols@microsoft.com)。可用性是模板的主要目标之一，让我们了解其优点和缺点

![Microsoft 3D Blueprint Visio Template](./media/architecture-overview/3DBlueprintVisioTemplate.jpg)



##绘制符号和图标集 

[下载云和企业符号集](http://aka.ms/CnESymbols)来帮助创建描述 Azure、Windows Server 和 SQL Server 等的技术资料。如果此书培训用户如何使用 Microsoft 产品，您可以使用体系结构图的符号、培训材料、演示文稿、数据表、信息图、白皮书和甚至是第三方书籍。但是，它们并不适合在用户界面中使用。

CnE 符号采用 Visio 和 PNG 格式。有关如何在 PowerPoint 中使用 PNG 的其他说明包括在集中。

符号集按季度发布，一旦新服务发布就会更新。

Microsoft Office 和相关技术的其他符号可在 [Microsoft Office Visio stencil](http://www.microsoft.com/zh-CN/download/details.aspx?id=35772) 中使用，但并没有对体系结构图如 CnE 集进行优化。

**反馈：**如果你曾经使用过 CnE 符号，请填写 5 个简短的[调查](http://aka.ms/azuresymbolssurveyv2)问题，如果有具体的问题或事宜需要咨询，请发电子邮件至 [CnESymbols@microsoft.com](mailto:CnESymbols@microsoft.com)。我们想知道您的想法，包括积极的反馈，以便我们知道可以继续对其投入时间。

![云和企业符号/图标集](./media/architecture-overview/CnESymbols.png)


##体系结构信息图

![Azure 体系结构信息图](./media/architecture-overview/AzureArchInfographicThumb.jpg)

<!---HONumber=Mooncake_1207_2015-->