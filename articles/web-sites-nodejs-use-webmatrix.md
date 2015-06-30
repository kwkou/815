<properties linkid="web-site-with-webmatrix" urlDisplayName=" 使用 WebMatrix 构建网站" pageTitle="使用 WebMatrix 构建 Node.js 网站 - Azure 教程" metaKeywords="" description="本教程将介绍如何使用 WebMatrix 开发 Node.js 应用程序并将其部署到 Azure 网站 Website。" metaCanonical="" services="web-sites" documentationCenter="Node.js" title="Build and deploy a Node.js Website to Azure using WebMatrix" authors="larryf" solutions="" manager="paulettm" editor="mollybos" />

<tags ms.service="web-sites"
    ms.date="02/19/2015"
    wacn.date="04/11/2015"
    />


# 使用 WebMatrix 构建 Node.js 网站并将其部署到 Azure

本教程将介绍如何使用 WebMatrix 开发 Node.js 应用程序并将其部署到 Azure 网站。WebMatrix 是 Microsoft 提供的一种免费 Web 开发工具，其中包含 Website 开发所需的一切。WebMatrix 包含一些使您能够轻松使用 Node.js 的功能，包括代码完成、预构建模板以及对 Jade、LESS 和 CoffeeScript 的编辑器支持。<!--了解有关 [WebMatrix for Azure](http://go.microsoft.com/fwlink/?LinkID=253622&clcid=0x409) 的详细信息。-->

完成本指南之后，您将拥有一个在 Azure 中运行的 Node.js 网站。
 
以下是已完成应用程序的屏幕快照：

![Azure node Website][webmatrix-node-completed]

[WACOM.INCLUDE [create-account-and-websites-note](../includes/create-account-and-websites-note.md)]

## 登录 Azure

请按照以下步骤创建 Azure 网站。

> [AZURE.NOTE] 若要完成本教程，你需要一个启用了 Azure 网站功能的 Azure 帐户。 <br /> 如果你没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial)。


1. 启动 WebMatrix
2. 如果这是你首次使用 WebMatrix 3，系统将会提示你登录 Azure。否则，你可以单击**"登录"(Sign In)**按钮，然后选择**"添加帐户"(Add Account)**。选择使用您的 Microsoft 帐户**"登录"(Sign in)**。

	![Add Account][addaccount]

3. 如果你已注册某一 Azure 帐户，则可以使用你的 Microsoft 帐户登录：

	![Sign into Azure][signin]	


## 使用 Azure 的内置模板创建网站

1. 在开始屏幕上，单击**"新建"(New)**按钮，然后选择**"模板库"(Template Gallery)**以便从模板库创建一个新网站：

	![New site from Template Gallery][sitefromtemplate]

2. 在**"从模板创建网站"(Site from Template)**对话框中，选择**"节点"(Node)**，然后选择**"Express 网站"(Express Site)**。最后，单击**"下一步"(Next)**。如果您缺少**"Express 网站"(Express Site)** 模板的任何必备组件，系统将提示您安装它们。

	![select express template][webmatrix-templates]

3. 如果你已登录到 Azure，现在可以选择为本地网站创建一个 Azure 网站。选择一个独有名称，然后选择你希望创建网站的数据中心： 

	![Create site on Azure][nodesitefromtemplateazure]
	
4. 在 WebMatrix 生成网站完毕后，将显示 WebMatrix IDE。

	![webmatrix ide][webmatrix-ide]

## 将应用程序发布到 Azure

1. 在 WebMatrix中，从**"主页"(Home)**功能区点击**"发布"(Publish)**，为网站显示**"发布预览"(Publish Preview)**对话框。

	![publish preview][webmatrix-node-publishpreview]

2. 单击**"继续"(Continue)**。发布完成后， Website Azure 的 URL 将显示在 WebMatrix IDE 底部

	![publish complete][webmatrix-publish-complete]

3. 单击链接在您的浏览器中打开该网站。

	![Express Website][webmatrix-node-express-site]

## 修改并重新发布应用程序

您可轻松修改并重新发布应用程序。在此，您将简单更改**index.jade**文件中的标题，然后重新发布应用程序。

1. 在 WebMatrix 中，选择**"文件"(Files)**，然后展开 **"视图"(views)**文件夹。双击打开**index.jade**文件。

	![webmatrix viewing index.jade][webmatrix-modify-index]

2. 将第二行更改为以下内容：

		p Welcome to #{title} with WebMatrix on Azure!

3. 保存所做更改，然后单击发布图标。最后，单击**"发布预览"(Publish Preview)**对话框中的**"继续"(Continue)**，并等待发布更新。

	![publish preview][webmatrix-republish]

4. 发布完成后，使用发布过程完成时返回的链接查看更新的网站。

	![Azure node Website][webmatrix-node-completed]

## 后续步骤

若要了解随 Azure 一起提供的 Node.js 的版本以及如何指定与您的应用程序一起使用的版本，请参见[在 Windows Azure 应用程序中指定 Node.js 版本](/documentation/articles/nodejs-specify-node-version-azure-apps)。

如果您在将应用程序部署到 Azure 后遇到与应用程序有关的问题，请参见[如何在 Azure 中调试 Node.js 应用程序 Website](/documentation/articles/web-sites-nodejs-debug)，了解有关诊断问题的信息。


[Azure 管理门户]: http://manage.windowsazure.cn
[WebMatrix 网站]: http://www.microsoft.com/click/services/Redirect2.ashx?CR_CC=200106398
<!--
[WebMatrix for Azure]: http://go.microsoft.com/fwlink/?LinkID=253622&clcid=0x409
-->
[使用 Git 发布 Azure 网站]:/documentation/articles/web-sites-publish-source-control/
[免费]: /pricing/1rmb-trial
[webmatrix-node-completed]: ./media/web-sites-nodejs-use-webmatrix/webmatrix-node-complete.png
[webmatrix-templates]: ./media/web-sites-nodejs-use-webmatrix/webmatrix-templates.png
[webmatrix-node-publishpreview]: ./media/web-sites-nodejs-use-webmatrix/webmatrix-publishpreview.png
[webmatrix-ide]: ./media/web-sites-nodejs-use-webmatrix/webmatrix-ide.png
[webmatrix-publish-complete]: ./media/web-sites-nodejs-use-webmatrix/webmatrix-publish-complete.png
[webmatrix-node-express-site]: ./media/web-sites-nodejs-use-webmatrix/webmatrix-express-webiste.png
[webmatrix-modify-index]: ./media/web-sites-nodejs-use-webmatrix/webmatrix-node-edit.png
[webmatrix-republish]: ./media/web-sites-nodejs-use-webmatrix/webmatrix-republish.png
[addaccount]: ./media/web-sites-nodejs-use-webmatrix/webmatrix-add-account.png
[signin]: ./media/web-sites-nodejs-use-webmatrix/webmatrix-sign-in.png
[sitefromtemplate]: ./media/web-sites-nodejs-use-webmatrix/webmatrix-site-from-template.png
[nodesitefromtemplateazure]: ./media/web-sites-nodejs-use-webmatrix/webmatrix-node-site-azure.png
