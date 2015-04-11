<properties linkid="dev-net-virtual-machine" urlDisplayName="Windows Azure 虚拟机" pageTitle="虚拟机技术 - Azure 微软云" metaKeywords="Virtual Machine,虚拟机,设置,迁移,管理,Visual Studio,映像,镜像,image,VHD,磁盘管理,镜像管理,面向开发,面向企业,SQL Server,sharepoint,SDK下载,虚拟机常见问题,开发,测试,.Net,NuGet,虚拟机可用性,SDK下载" description="本文是用户了解微软云提供的虚拟机服务的入口页面，无论是虚拟机新手、开发人员，还是企业用户，都可以在本文中找到如何设置、迁移和管理虚拟机的相关文档链接。还可以透过本页下载微软提供的丰富的SDK供开发者使用。使用虚拟机可在你需要灵活的资源时配置可缩放的按需计算基础结构。创建运行 Windows、Linux 和企业应用程序的 VM。或者，捕获你自己的映像以便创建自定义虚拟机。" metaCanonical="" services="Virtual Machine" documentationCenter="Services" title="虚拟机技术的相关指南" authors="" solutions="" manager="" editor="" />
<tags ms.service="Virtual Machine"
    ms.date="10/14/2014"
    wacn.date="04/11/2015"
    />

#虚拟机

##设置、迁移和管理虚拟机

使用虚拟机可在你需要灵活的资源时配置可缩放的按需计算基础结构。创建运行 Windows、Linux 和企业应用程序的 VM。或者，捕获你自己的映像以便创建自定义虚拟机。

##快速链接

-   [技术概述](http://msdn.microsoft.com/zh-cn/library/azure/jj156143.aspx)
-   [定价详细信息](/pricing/details/virtual-machines/)

##Azure VM 新手？

####[使用 RDP 或 SSH 连接到虚拟机](http://msdn.microsoft.com/zh-cn/library/azure/dn535788.aspx)

请遵照这些步骤使用 RDP 或 SSH 连接到您的新虚拟机。其中包括有关排查连接问题的提示。

####[管理虚拟机的可用性](/zh-cn/documentation/articles/virtual-machines-manage-availability/)

由于计划或非计划维护而重新启动 VM 时如何避免出现问题 - 这些有关可用性集的最佳实践可帮助您应对维护事件带来的影响。

####[管理磁盘和映像](http://msdn.microsoft.com/zh-cn/library/azure/jj672979.aspx)

了解如何管理磁盘、虚拟硬盘 (VHD) 和 Azure 虚拟机中的映像。

####[Windows PowerShell 中的 Azure cmdlet 入门](http://msdn.microsoft.com/zh-cn/library/azure/jj554332.aspx)

了解如何从命令行使用适用于 Windows PowerShell 的 Azure cmdlet 来管理基于 Windows 的虚拟机。

####[Azure 虚拟机常见问题](http://msdn.microsoft.com/zh-cn/library/azure/dn683781.aspx)

这正是您所期待的内容：客户对 Azure VM 经常提出的问题列表！

##面向开发人员

###Visual Studio

####[在 Visual Studio 中创建 Azure 虚拟机](http://msdn.microsoft.com/zh-cn/library/azure/dn569263.aspx)

了解如何直接通过 Visual Studio 创建 Azure VM。如果您是 MSDN 订户，则甚至还可以创建预装了 Visual Studio 的 Azure VM。

####[从服务器资源管理器访问 Azure 虚拟机](http://msdn.microsoft.com/zh-cn/library/azure/jj131259.aspx)

请执行以下步骤，通过服务器资源管理器设置对虚拟机的访问。

####[适用于 .NET 的管理库入门](http://msdn.microsoft.com/zh-cn/library/azure/dn722415.aspx)

参考此教程开始使用 .NET 管理库来管理您的服务 - 此教程说明如何使用某些可用的客户端来创建和删除虚拟机。

####[使用 Windows PowerShell 脚本发布到开发和测试环境](http://msdn.microsoft.com/zh-cn/library/azure/dn642480.aspx)

了解如何使用 Visual Studio 生成发布脚本，用于自动将 Web 应用程序发布到虚拟机。

####[下载 Azure 管理库的 NuGet 包](http://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Libraries)

下载可用于自动化、部署和测试 Azure 虚拟机的 NuGet 包。

####[在 Visual Studio 中调试云服务或虚拟机](http://msdn.microsoft.com/zh-cn/library/azure/ff683670.aspx)

了解如何使用针对虚拟机的远程调试功能和本机代码调试功能来远程诊断应用程序问题。

###指南

####[Azure 网站、云服务和虚拟机对比](/zh-cn/documentation/articles/choose-web-site-cloud-service-vm/)

Azure 提供三种可用于托管 Web 应用程序的计算模型：网站、云服务和虚拟机。本主题概述了三种模型和信息，以帮助你确定适用于你的应用程序的模型。

####[使用 Azure 虚拟机进行自动测试](http://justazure.com/automated-testing-in-microsoft-azure/)

Cerebrata 团队说明他们如何以及为何决定使用 Azure 虚拟机进行自动性能测试

###参考

-   [SDK 下载](http://www.windowsazure.cn/downloads/)
-   [管理库 (NuGet)](http://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Libraries)
-   [服务管理 (REST)：虚拟机操作](http://msdn.microsoft.com/zh-cn/library/azure/jj157206.aspx)
-   [服务管理 (REST)：虚拟机磁盘操作](http://msdn.microsoft.com/zh-cn/library/azure/jj157188.aspx)
-   [服务管理 (REST)：操作系统映像操作](http://msdn.microsoft.com/zh-cn/library/azure/jj157175.aspx)
-   [服务管理客户端库（计算）](http://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.management.compute.aspx)

##面向企业

###SQL Server

####[了解适用于 Azure 虚拟机中 SQL Server 的应用程序模式和开发策略](http://msdn.microsoft.com/zh-cn/library/azure/dn574746.aspx)

本文为解决方案架构师和开发人员提供必要的基础知识，以帮助他们完善应用程序的体系结构和设计，不管他们是要迁移现有的应用程序还是开发新的应用程序。

####[准备迁移到 Azure VM 中的 SQL Server](http://msdn.microsoft.com/zh-cn/library/azure/dn133142.aspx)

准备迁移计划，了解针对在 Azure 虚拟机上运行的 SQL Server 的许可，以及评估迁移途径。

####[了解 Azure 虚拟机中的 SQL Server 部署选项](http://msdn.microsoft.com/zh-cn/library/azure/dn133141.aspx)

帮助您决定如何在 Windows Azure 虚拟机中部署 SQL Server 的指导 – 您可以对多种方法进行评估以便决定适合您的方法。

###SharePoint

####[判断您是否适合使用 Azure VM 上的 SharePoint 2013](http://msdn.microsoft.com/zh-cn/library/azure/dn275958.aspx?amp;clcid=0x804)

此白皮书可以帮助 IT 专业人员、架构师和系统管理员确定自己的组织是否适合在 Azure 虚拟机上部署 Microsoft SharePoint 2013。

####[在 Azure 虚拟机中建立 SharePoint 2013 开发/测试实验室](http://blogs.technet.com/b/keithmayer/archive/2013/01/07/step-by-step-build-a-free-sharepoint-2013-lab-in-the-cloud-with-windows-azure-31-days-of-servers-in-the-cloud-part-7-of-31.aspx#.Uxe4bXmPKUl)

由于 SharePoint 2013 的硬件要求较高，您可能无法提供合格的备用硬件来实施本地实验环境。此教程说明如何在 Azure 中构建一个用于研究、测试、演示、开发环境或概念验证的 SharePoint 2013 实验环境。

####[在 Azure 基础结构服务上配置和部署 SharePoint 2013 场](http://msdn.microsoft.com/zh-cn/library/azure/dn275959.aspx?amp;clcid=0x804)

在一组虚拟机上配置和部署 SharePoint 场。在本教程中，您将使用市场中的映像创建一组虚拟机，然后将创建一个域。之后，您将计算机加入该域中并运行 SharePoint 配置向导。最后，您将了解如何启用 SQL Server AlwaysOn 功能以实现高可用性。

##更多

####[Active Directory](http://msdn.microsoft.com/zh-cn/library/azure/jj156090.aspx)

提供有关在 Azure 中的虚拟机上使用 Windows Server Active Directory 时将需要的背景信息以及针对建议的部署方案的指南。

####[BizTalk Server](http://msdn.microsoft.com/zh-cn/library/azure/jj248689)

介绍如何在 Azure 中的虚拟机上配置 BizTalk Server 2013。

####[桌面托管](http://msdn.microsoft.com/zh-cn/library/azure/dn451351.aspx)

主机托管提供商可以参考这些体系结构和部署指南，针对用户数不超过 1,500 的中小型组织创建安全、可靠、可伸缩的桌面托管解决方案产品。

####[Dynamics AX](http://technet.microsoft.com/zh-cn/library/dn741581.aspx)

提供必要的指导，以帮助您使用 Microsoft Dynamics 生命周期服务中的云托管环境工具在 Azure 上部署 Microsoft Dynamics AX 2012 R3 环境。

####[HPC Pack](http://msdn.microsoft.com/zh-cn/library/azure/dn518135.aspx)

指导您在 Azure 虚拟机群集上部署 HPC Pack 以运行大型计算工作负载，例如本质并行的应用程序。

####[Oracle 软件](http://msdn.microsoft.com/zh-cn/library/azure/dn439770.aspx)

用于设置运行 Oracle Database、WebLogic Server 和 Java 平台的虚拟机的指南和教程。

####[SAP 软件](http://msdn.microsoft.com/zh-cn/library/azure/dn745892.aspx)

有关在 Azure 虚拟机中使用 SAP NetWeaver 和 SAP DBMS 的规划、实施和部署指导。


