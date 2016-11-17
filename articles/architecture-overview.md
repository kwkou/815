<properties 
	pageTitle="Azure 上的应用程序体系结构 | Azure" 
	description="包括常见设计模式的体系结构概述" 
	services="" 
	documentationCenter="" 
	authors="Rboucher" 
	manager="jwhit" 
	editor="mattshel"/>  


<tags 
	ms.service="multiple" 
	ms.workload="na" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="09/13/2016" 
	wacn.date="10/24/2016" 
	ms.author="robb"/>  


#Azure 上的应用程序体系结构
用于构建使用 Azure 的应用程序的资源。这包括一些工具，帮助您画图以直观地描述软件系统。

##设计模式海报

Microsoft 模式与实践部门已发布了[云设计模式](http://msdn.microsoft.com/zh-cn/library/dn568099.aspx)一书，在 MSDN 上可用，也可以 PDF 格式下载。此外还提供了一张可用的大画幅海报，列出了所有的模式。

![模式与实践：云模式海报](./media/architecture-overview/PnPPatternPosterThumb.jpg)

## <a name="microsoft-architecture-certification-course"></a>Microsoft 体系结构认证课程

Microsoft 创建了支持 Microsoft 认证考试 70-534 的体系结构课程。在 [EDX.ORG 上免费提供](https://www.edx.org/course/architecting-microsoft-azure-solutions-microsoft-dev205x)。它使用 [3D Blueprint Visio 模板](#3d-blueprint-visio-template)。

![Microsoft 体系结构认证课程](./media/architecture-overview/EDXCourse.png)  



##Microsoft 解决方案

Microsoft 发布了一套高级别的[解决方案体系结构](http://aka.ms/azblueprints)，介绍如何使用 Microsoft 产品构建特定类型的系统。

之前，Microsoft 发布了一组展示示例体系结构的蓝图。它们已替换为前面提到的解决方案体系结构，并已将该蓝图链接重定向为指向这些体系结构。如果出于某种原因，需访问之前的蓝图材料，请发送电子邮件到 [CnESymbols@microsoft.com](mailto:CnESymbols@microsoft.com) 说明请求。

蓝图和解决方案体系结构图都会使用部分[云和企业符号集](#Drawing-symbol-and-icon-sets)。

![Microsoft 体系结构蓝图 3D 图](./media/architecture-overview/BluePrintThumb.jpg)  




## <a name="3d-blueprint-visio-template"></a>3D Blueprint Visio Template

3D 版 [Microsoft 体系结构蓝图](http://aka.ms/azblueprints)最初在非 Microsoft 工具中创建，现在已失效。Visio 2013（和更高版本）模板于 2015 年 8 月 5 日发布，作为 [Microsoft 体系结构认证课程的一部分，分发在 EDX.ORG 上](#microsoft-architecture-certification-course)。

该模板在本课程外也可用。

- 下载 [Microsoft 3d Blueprint Visio Template](http://aka.ms/3DBlueprintTemplate)
- 下载与 3D 模板配合使用的[云和企业符号](#drawing-symbol-and-icon-sets)

提供反馈，或者需要提培训材料无法答复的具体问题，请发邮件至 [CnESymbols@microsoft.com](mailto:CnESymbols@microsoft.com)。该模板不再处于积极开发阶段，但仍具有用途和相关性，因为它可以使用任何 PNG 或更新的[云和企业符号](#drawing-symbol-and-icon-sets)。

![Microsoft 3D Blueprint Visio Template](./media/architecture-overview/3DBlueprintVisioTemplate.jpg)  



## <a name="Drawing-symbol-and-icon-sets" id="drawing-symbol-and-icon-sets"></a>绘制符号和图标集 

[下载云和企业符号集](http://aka.ms/CnESymbols)来帮助创建描述 Azure、Windows Server 和 SQL Server 等的技术资料。如果此书培训用户如何使用 Microsoft 产品，您可以使用体系结构图的符号、培训材料、演示文稿、数据表、信息图、白皮书和甚至是第三方书籍。但是，它们并不适合在用户界面中使用。

CnE 符号采用 Visio、SVG 和 PNG 格式。有关如何在 PowerPoint 中轻松使用符号的其他说明包括在集中。

符号集按季度发布，一旦新服务发布就会更新。

Microsoft Office 和相关技术的其他符号可在 [Microsoft Office Visio stencil](http://www.microsoft.com/download/details.aspx?id=35772) 中使用，但并没有对体系结构图如 CnE 集进行优化。

**反馈：**如果你曾经使用过 CnE 符号，请填写 5 个简短的[调查](http://aka.ms/azuresymbolssurveyv2)问题，如果有具体的问题或事宜需要咨询，请发电子邮件至 [CnESymbols@microsoft.com](mailto:CnESymbols@microsoft.com)。我们想知道您的想法，包括积极的反馈，以便我们知道可以继续对其投入时间。

![云和企业符号/图标集](./media/architecture-overview/CnESymbols.png)  


##体系结构信息图

Microsoft 发布了几个与体系结构相关的海报/信息图。它们包括构建实际云应用程序和与云服务一起成长。

![Azure 体系结构信息图](./media/architecture-overview/AzureArchInfographicThumb.jpg)

<!---HONumber=Mooncake_1017_2016-->