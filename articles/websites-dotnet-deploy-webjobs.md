<properties 
	pageTitle="将 Azure Web 作业部署到 Azure 网站" 
	description="了解如何使用 Visual Studio 将 Azure Web 作业部署到 Azure 网站。" 
	services="web-sites" 
	documentationCenter="" 
	authors="tdykstra" 
	manager="wpickett" 
	editor="mollybos"/>
<tags ms.service="web-sites"
    ms.date="04/03/2015"
    wacn.date="04/15/2015"
    />

# 如何将 Azure Web 作业部署到 Azure 网站

## 概述

本主题说明如何使用 Visual Studio 将控制台应用程序项目作为 [Azure Web 作业](/documentation/articles/websites-webjobs-resources)部署到 Azure 网站。部署 Web 作业的另一个方法是使用 Azure 管理门户，具体请参阅[使用 Web 作业在 Microsoft Azure 网站中运行后台任务](/documentation/articles/web-sites-create-web-jobs)。

当 Visual Studio 部署启用 Web 作业的控制台应用程序项目时，它会执行两个任务：

* 将运行时文件复制到 Azure 网站中的相应文件夹（对于连续 Web 作业，该文件夹为 *App_Data/jobs/continuous*，对于计划的按需 Web 作业，则为  *App_Data/jobs/triggered*）。
* 为已计划在特定时间运行的 Web 作业设置 [Azure 计划程序作业](#scheduler) 。（无需为连续 Web 作业执行此操作。）

已启用 Web 作业的项目中添加了以下项：

* [Microsoft.Web.WebJobs.Publish](http://www.nuget.org/packages/Microsoft.Web.WebJobs.Publish) NuGet 包。
* 一个 [webjob-publish-settings.json](#publishsettings) 文件，其中包含部署和计划程序设置。 

![Diagram showing what is added to a Console App to enable deployment as a WebJob](./media/websites-dotnet-deploy-webjobs/convert.png)

可以将这些项添加到现有控制台应用程序项目，或使用模板创建已启用 Web 作业的新控制台应用程序项目。 

以 Web 作业的方式部署项目，或者将项目链接到 Web 项目，因此每当你要部署 Web 项目时，项目便会自动部署。为了链接项目，Visual Studio 会在 Web 项目的 [webjobs-list.json](#webjobslist) 文件中包含启用 Web 作业的 项目的名称。

![Diagram showing WebJob project linking to web project](./media/websites-dotnet-deploy-webjobs/link.png)
 


## 先决条件

安装 Azure SDK 2.4 或更高版本后，便可以在 Visual Studio 2013 中使用 Web 作业部署功能：

* [Azure SDK for Visual Studio 2013](https://www.microsoft.com/web/handlers/webpi.ashx/getinstaller/VWDOrVs2013AzurePack.appids)。

[Visual Studio 2013 Update 3](https://www.microsoft.com/zh-CN/downloads/details.aspx?FamilyID=769b58b3-9e2e-4b57-9070-2159fac2b5fb) 和更高版本的更新也包含了 Web 作业部署功能。

## <a id="convert"></a>为现有的控制台应用程序项目启用 WebJobs 部署

可以使用两个选项：

* [使用 Web 项目启用自动部署](#convertlink)。

	配置现有控制台应用程序项目，以便在部署 Web 项目时，它以 Web 作业的方式自动部署。当你要在与运行相关 Web 应用程序相同的网站中运行 Web 作业时，请使用此选项。

* [不使用 Web 项目启用部署](#convertnolink)。

	配置现有控制台应用程序项目以 Web 作业的方式部署，且不提供 Web 项目的链接。当你要在网站本身中运行 Web 作业，且网站中没有正在运行的 Web 应用程序时，请使用此选项。为了能够缩放不受 Web 应用程序资源影响的 Web 作业资源，你可能想要执行此操作。

### <a id="convertlink"></a> 使用 Web 项目启用自动 Web 作业部署
  
1. 右键单击"解决方案资源管理器"中的 Web 项目，然后单击"添加">"用作 Azure Web 作业的现有项目"。

	![用作 Azure Web 作业的现有项目](./media/websites-dotnet-deploy-webjobs/eawj.png)
	
	此时将显示["添加 Azure Web 作业"](#configure) 对话框。

1. 在"项目名称"下拉列表中，选择要添加为 Web 作业的控制台应用程序项目。

	![在"添加 Azure Web 作业"对话框中选择项目](./media/websites-dotnet-deploy-webjobs/aaw1.png)

2. 完成["添加 Azure Web 作业"](#configure)对话框，然后单击"确定"。 

### <a id="convertnolink"></a> 不使用 Web 项目启用 Web 作业部署
  
1. 右键单击"解决方案资源管理器"中的控制台应用程序项目，然后单击"发布为 Azure Web 作业"。 

	![Publish as Azure WebJob](./media/websites-dotnet-deploy-webjobs/paw.png)
	
	此时将显示["添加 Azure Web 作业"](#configure) 对话框，其"项目名称"框中已选中该项目。

2.  完成["添加 Azure Web 作业"](#configure)， 对话框，然后单击"确定"。

	此时将显示"发布 Web"向导。  如果你不打算立即发布，请关闭向导。输入的设置将会保存，以便在[部署项目](#deploy)时使用。

## <a id="create"></a>创建已启用 Web 作业的新项目

若要创建已启用 Web 作业的新项目，可以使用控制台应用程序项目模板，并根据[上一部分](#convert)中所述启用 Web 作业部署。或者，你可以使用 Web 作业新建项目模板：

* [为独立的 Web 作业使用 Web 作业新建项目模板](#createnolink)

	创建一个项目，并将它配置为以 Web 作业的方式部署，且不提供 Web 项目的链接。当你要在网站本身中运行 Web 作业，且网站中没有正在运行的 Web 应用程序时，请使用此选项。为了能够缩放不受 Web 应用程序资源影响的 Web 作业资源，你可能想要执行此操作。

* [在链接到 Web 项目的 Web 作业中使用 Web 作业新建项目模板](#createlink)

	创建一个项目，该项目配置为在针对位于相同解决方案中的 Web 项目进行部署时，自动以 Web 作业的方式部署。当你要在与运行相关 Web 应用程序相同的网站中运行 Web 作业时，请使用此选项。

在 SDK 2.4 版本中，Web 作业新建项目模板并不比创建控制台应用程序项目并启用 Web 作业部署容易。将来，Web 作业新建项目模板将更有助于 [WebJobs SDK](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/getting-started-with-windows-azure-webjobs) 开发，因为它会自动安装相应的 WebJobs SDK NuGet 包。在此之前，你可以手动安装此包，以将项目配置为使用 Webjobs SDK，如 [Webjobs SDK 教程](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/getting-started-with-windows-azure-webjobs)中所示。 


### <a id="createnolink"></a> 为独立的 Web 作业使用 Web 作业新建项目模板
  
1. 单击"文件">"新建项目"，然后在"新建项目"对话框中，单击"云">"Microsoft Azure Web 作业"。

	![New Project dialog showing WebJob template](./media/websites-dotnet-deploy-webjobs/np.png)
	
2. 遵循前面所示的说明，[将控制台应用程序项目设为独立的 Web 作业项目](#convertnolink)。

### <a id="createlink"></a> 在链接到 Web 项目的 Web 作业中使用 Web 作业新建项目模板

1. 右键单击"解决方案资源管理器"中的 Web 项目，然后单击"添加" >"新建 Azure Web 作业项目"。

	!["新建 Azure Web 作业项目"菜单项](./media/websites-dotnet-deploy-webjobs/nawj.png)

	此时将显示["添加 Azure Web 作业"](#configure) 对话框。

2. 完成["添加 Azure Web 作业"](#configure) ，对话框，然后单击"确定"。

## <a id="configure"></a>"添加 Azure Web 作业"对话框

"添加 Azure Web 作业"对话框可让你输入 Web 作业的名称和计划设置。 

!["添加 Azure Web 作业"对话框](./media/websites-dotnet-deploy-webjobs/aaw2.png)

此对话框中的字段对应于 Azure 管理门户中"新建作业"对话框上的字段。有关详细信息，请参阅[使用 Web 作业在 Microsoft Azure 网站中运行后台任务](/documentation/articles/web-sites-create-web-jobs)。

对于计划的 Web 作业（而不是连续 Web 作业），Visual Studio 将创建 [Azure 计划程序](/home/features/scheduler) 作业集合（如果尚不存在），然后在该集合中创建一个作业：

* 计划程序作业集合命名为  *WebJobs-{regionname}*，其中的 *{regionname}* 表示托管网站的区域。例如：WebJobs-WestUS。
* 计划程序作业命名为 *{websitename}-{webjobname}*。例如：MyWebSite-MyWebJob。 
 
>[AZURE.NOTE]
> 
>* 有关命令行部署的信息，请参阅[启用 Azure Web 作业的命令行或连续传送](http://azure.microsoft.com/blog/2014/08/18/enabling-command-line-or-continuous-delivery-of-azure-webjobs)。
>* 如果你配置了"定期作业"，并将周期频率设置为某个分钟数，则 Azure 计划程序不是免费的。其他频率（小时数、天数等）是免费的。
>* 如果你部署了某个 Web 作业，并随后将运行模式从连续更改为非连续（或相反），则在你重新部署时，Visual Studio 将在 Azure 中创建新的 Web 作业。如果更改了其他计划设置但保持运行模式不变，或在计划模式与按需模式之间切换，则 Visual Studio 会更新现有的作业，而不是创建新的作业。

## <a id="publishsettings"></a>webjob-publish-settings.json

当你设置 Web 作业部署的控制台应用程序时，Visual Studio 将会安装 [Microsoft.Web.WebJobs.Publish](http://www.nuget.org/packages/Microsoft.Web.WebJobs.Publish) NuGet 包， 
并将计划信息存储在 Web 作业项目的项目 *Properties* 文件夹中的 *webjob-publish-settings.json* 文件内。以下是该文件的示例：

		{
		  "$schema": "http://schemastore.org/schemas/json/webjob-publish-settings.json",
		  "webJobName": "WebJob1",
		  "startTime": "2014-06-23T00:00:00-08:00",
		  "endTime": "2014-06-27T00:00:00-08:00",
		  "jobRecurrenceFrequency": "Minute",
		  "interval": 5,
		  "runMode": "Scheduled"
		}

你可以编辑此文件目录，Visual Studio 将提供 IntelliSense。文件架构存储在 [http://schemastore.org](http://schemastore.org/schemas/json/webjob-publish-settings.json) 中，你可以在那里查看。  

>[AZURE.NOTE]
>
>* 如果你配置了"定期作业"，并将周期频率设置为某个分钟数，则 Azure 计划程序不是免费的。其他频率（小时数、天数等）是免费的。

## <a id="webjobslist"></a>webjobs-list.json

当你将已启用 Web 作业的项目链接到 Web 项目时，Visual Studio 会将 Web 作业项目的名称存储在 Web 项目的 *Properties* 文件夹中的 *webjobs-list.json* 文件内。该列表可能包含多个 Web 作业项目，如以下示例中所示：

		{
		  "$schema": "http://schemastore.org/schemas/json/webjobs-list.json",
		  "WebJobs": [
		    {
		      "filePath": "../ConsoleApplication1/ConsoleApplication1.csproj"
		    },
		    {
		      "filePath": "../WebJob1/WebJob1.csproj"
		    }
		  ]
		}

你可以编辑此文件目录，Visual Studio 将提供 IntelliSense。文件架构存储在 [http://schemastore.org](http://schemastore.org/schemas/json/webjobs-list.json) 中，你可以在那里查看。
  
## <a id="deploy"></a>部署 Web 作业项目

已链接到 Web 项目的 Web 作业项目会通过 Web 项目自动部署。有关 Web 项目部署的信息，请参阅[如何部署 Azure 网站](/documentation/articles/websites-dotnet-deploy)。

若要自动部署某个 Web 作业项目，请在"解决方案资源管理器"中右键单击该项目，然后单击"发布为 Azure Web 作业"。 

![发布为 Azure Web 作业](./media/websites-dotnet-deploy-webjobs/paw.png)
	
对于独立的 Web 作业，将会显示 Web 项目使用的相同"发布 Web"向导，但其中的可更改设置更少。

## <a id="nextsteps"></a>后续步骤

有关如何在 Visual Studio 中使用连续传送过程部署 Azure Web 作业的详细信息，请参阅 [Azure Web 作业 - 推荐的资源 - 部署](http://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/azure-webjobs-recommended-resources#deploying)。

<!--HONumber=50-->