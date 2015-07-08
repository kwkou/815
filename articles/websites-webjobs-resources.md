<properties 
	pageTitle="有关 Azure Web 作业的推荐资源" 
	description="学习如何使用 Azure Web 作业和 Azure WebJobs SDK 时可以参考的推荐资源." 
	services="web-sites, storage" 
	documentationCenter=".net" 
	authors="tdykstra" 
	manager="wpickett" 
	editor="jimbe"/>
<tags ms.service="web-sites, storage"
    ms.date="04/03/2015"
    wacn.date="04/15/2015"
    />


# 有关 Azure Web 作业的推荐资源

## 概述

本主题提供了有关如何使用 Azure Web 作业和 Azure WebJobs SDK 的文档资源的链接。Azure Web 作业可让你在 Azure 网站上以后台进程的形式运行脚本或程序。你可以上载和运行可执行文件，例如 cmd、bat、exe (.NET)、ps1、sh、php、py、js 和 jar。这些程序将按计划 (cron) 或者连续地以 Web 作业的形式运行。

借助 WebJobs SDK 可以更方便地使用 Azure 存储空间。WebJobs SDK 还有一个绑定和触发器系统，该系统可与 Windows Azure 存储 Blob、队列和表以及 Service Bus 队列配合工作。

使用 Visual Studio 中集成的工具，可以顺利地创建、部署和管理 Web 作业。你可以从模板创建 Web 作业，还可以发布和管理（运行/停止/监视/调试）这些作业。 

Azure 管理门户中的 Web 作业仪表板提供了强大的管理功能，可让你全面控制 Web 作业的执行，包括调用 Web 作业中的各个函数。该仪表板还会显示函数运行时和日志记录输出。 

## <a name="getstarted"></a>Web 作业和 WebJobs SDK 入门

* [Azure Web 作业简介](http://www.hanselman.com/blog/IntroducingWindowsAzureWebJobs.aspx)
* [Azure Web 作业功能](http://azure.microsoft.com/blog/2014/10/22/webjobs-goes-into-full-production)
* [什么是 WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk)
* [宣布推出 Windows Azure WebJobs SDK 1.0.0 RTM](http://azure.microsoft.com/blog/2014/10/25/announcing-the-1-0-0-rtm-of-microsoft-azure-webjobs-sdk)
* [Azure WebJobs SDK 入门](/documentation/articles/websites-dotnet-webjobs-sdk-get-started)
* [如何通过 WebJobs SDK 使用 Azure 队列存储](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to)
* [如何通过 WebJobs SDK 使用 Azure Blob 存储](/documentation/articles/websites-dotnet-webjobs-sdk-storage-blobs-how-to)
* [如何通过 WebJobs SDK 使用 Azure 表存储](/documentation/articles/websites-dotnet-webjobs-sdk-storage-tables-how-to)
* [如何通过 WebJobs SDK 使用 Azure Service Bus](/documentation/articles/websites-dotnet-webjobs-sdk-service-bus)
* [Azure WebJobs SDK 快速参考（PDF 下载）](http://download.microsoft.com/download/2/2/0/220DE2F1-8AB3-474D-8F8B-C998F7C56B5D/Azure%20WebJobs%20SDK%20Cheat%20Sheet%202014.pdf)
另请参阅以下有关[部署 Web 作业](#deploy) 以及[测试和调试 Web 作业](#debug)的部分。

## <a name="deploy"></a>部署 Web 作业

* [如何将 Azure Web 作业部署到 Azure 网站](/documentation/articles/websites-dotnet-deploy-webjobs)
* [如何使用 Azure 管理门户部署 Web 作业](/documentation/articles/web-sites-create-web-jobs)
* [启用 Azure Web 作业的命令行或连续传送](http://azure.microsoft.com/blog/2014/08/18/enabling-command-line-or-continuous-delivery-of-azure-webjobs)
* [使用 Web 作业通过 Git 将 .NET 控制台应用程序部署到 Azure](http://blog.amitapple.com/post/73574681678/git-deploy-console-app)
 

## <a name="schedule"></a>计划 Web 作业

* [添加 Azure Web 作业对话框](/documentation/articles/websites-dotnet-deploy-webjobs/#configure)
* [在 Azure 管理门户中创建计划的 Web 作业](/documentation/articles/web-sites-create-web-jobs/#CreateScheduled)

## <a name="debug"></a>测试和调试 Web 作业

* [Visual Studio 中 Azure Web 作业的新开发人员和调试功能](http://blogs.msdn.com/b/webdev/archive/2014/11/12/new-developer-and-debugging-features-for-azure-webjobs-in-visual-studio.aspx)
* [查看 Web 作业仪表板](/documentation/articles/websites-dotnet-webjobs-sdk-get-started/#view-the-webjobs-sdk-dashboard)
* [如何使用 WebJobs SDK 写入日志并在仪表板中查看日志](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/#logs)
* [远程调试 Web 作业](/documentation/articles/web-sites-dotnet-troubleshoot-visual-studio/#remotedebugwj)
* [谁写入了 Blob？](http://blogs.msdn.com/b/jmstall/archive/2014/02/19/who-wrote-that-blob.aspx)
* [在云中托管交互式代码](http://blogs.msdn.com/b/jmstall/archive/2014/04/26/hosting-interactive-code-in-the-cloud.aspx)
* [获取一个仪表板以使用 WebJobs SDK 进行本地开发](http://blogs.msdn.com/b/jmstall/archive/2014/01/27/getting-a-dashboard-for-local-development-with-the-webjobs-sdk.aspx)
* [在 Azure 网站和 Web 作业中添加跟踪](http://blogs.msdn.com/b/mcsuksoldev/archive/2014/09/04/adding-trace-to-azure-web-sites-and-web-jobs.aspx)
* [监视、诊断和排查 Windows Azure 存储空间问题](/documentation/articles/storage-monitoring-diagnosing-troubleshooting)


## <a name="scale"></a>缩放 Web 作业

* [缩放 Azure 网站上的 Web 应用程序](http://msdn.microsoft.com/magazine/dn786914.aspx)
* [Azure 网站：构建随时可投入业务的大规模 Web 应用程序](https://channel9.msdn.com/Events/Build/2014/3-626)。包括使用 Web 作业和 WebJobs SDK 缩放 Azure 网站。


## <a name="additional"></a>其他 Web 作业资源

* [Magnus Mårtensson 发表的 Azure Web 作业 GA 博客文章](http://magnusmartensson.com/azure-webjobs-ga)
* [GitHub 中的 Web 作业设置文档](https://github.com/projectkudu/kudu/wiki/Web-jobs)。
* [在 Azure 网站上运行 Powershell Web 作业](http://blogs.msdn.com/b/nicktrog/archive/2014/01/22/running-powershell-web-jobs-on-azure-websites.aspx)
* [在 Azure 完成触发 Web 作业时接收通知](http://blog.amitapple.com/post/2014/03/webjobs-notification)
* [使用 Web 作业创建简单的网站备份保留策略](http://azure.microsoft.com/blog/2014/04/28/simple-web-site-backup-retention-policy-with-webjobs)
* [在处理第一个请求时 Azure 网站和云服务速度缓慢](http://wp.sjkp.dk/windows-azure-websites-and-cloud-services-slow-on-first-request)。演示如何使用 Web 作业来模拟仅适用于标准网站层的 AlwaysOn 功能。
* [Web 作业正常关闭](http://blog.amitapple.com/post/2014/05/webjobs-graceful-shutdown/#.U72Il_5OWUl)。对于 WebJobs SDK 正常关闭，请参阅[正常关闭](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/#graceful)。）


## <a name="additionalsdk"></a>其他 WebJobs SDK 资源

* [Azure 作业中的触发器、绑定和路由参数](http://blogs.msdn.com/b/jmstall/archive/2014/01/28/trigger-bindings-and-route-parameters-in-azurejobs.aspx)
* Azure 存储绑定
	* [Blob](http://blogs.msdn.com/b/jmstall/archive/2014/02/18/azure-storage-bindings-part-1-blobs.aspx)
	* [队列](http://blogs.msdn.com/b/jmstall/archive/2014/02/18/azure-storage-bindings-part-2-queues.aspx)
	* [表](http://blogs.msdn.com/b/jmstall/archive/2014/03/06/azure-storage-bindings-part-3-tables.aspx)
* [[BlobTrigger] 的工作原理](http://blogs.msdn.com/b/jmstall/archive/2014/04/17/how-does-blobinput-work.aspx)
* [Azure WebJobs SDK 的高级绑定](http://victorhurdugaci.com/advanced-bindings-with-the-windows-azure-web-jobs-sdk)
* [使用 WebJobs SDK 将 FREB 文件上载到 Azure 存储空间的 Web 作业](http://thenextdoorgeek.com/post/WAWS-WebJob-to-upload-FREB-files-to-Azure-Storage-using-the-WebJobs-SDK)
* [在 Azure 外部托管 Web 作业，获得 Azure 托管 Web 作业的日志记录优势](http://bypassion.dk/?p=510)
* [使用 Azure Web 作业生成数据导入工具](http://www.freshconsulting.com/building-data-import-tool-azure-webjobs)


## <a name="samples"></a>示例 Web 作业应用程序

* [Web 作业团队在 GitHub 上提供的示例应用程序](https://github.com/azure/azure-webjobs-sdk-samples)
* [使用 WebJobs SDK 创建包含 Web 作业后端的简单 Azure 网站](http://code.msdn.microsoft.com/Simple-Azure-Website-with-b4391eeb)
* [SiteMonitR](http://code.msdn.microsoft.com/SiteMonitR-dd4fcf77)。演示计划 Web 作业和事件驱动型 Web 作业的用法。请参阅博客文章[使用 Azure WebJobs SDK 重新生成 SiteMonitR](http://www.bradygaster.com/post/rebuilding-the-sitemonitr-using-windows-azure-webjobs)。

## <a name="blogs"></a>博客

* [Azure 博客](http://azure.microsoft.com/blog)
* [Amit Apple 的博客](http://blog.amitapple.com)。重点探讨 Web 作业（而不是 SDK）。
* [Magnus Mårtensson 的博客](http://magnusmartensson.com)

## <a name="gethelp"></a>获取有关 Web 作业的帮助

* [Web 作业堆栈溢出](http://stackoverflow.com/questions/tagged/azure-webjobs)
* [WebJobs SDK 堆栈溢出](http://stackoverflow.com/questions/tagged/azure-webjobssdk)
* [Azure 和 ASP.NET 论坛](http://forums.asp.net/1247.aspx)
* [Azure 网站论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=windowsazurezhchs)


<!--HONumber=50-->