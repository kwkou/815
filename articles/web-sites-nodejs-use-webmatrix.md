<properties 
	pageTitle="使用 WebMatrix 构建 Node.js Web 应用并将其部署到 Azure" 
	description="本教程演示如何使用 WebMatrix 开发 Node.js 应用程序并将其部署到 Azure Web 应用。" 
	services="app-service\web" 
	documentationCenter="nodejs" 
	authors="rmcmurray" 
	manager="wpickett" 
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.date="05/04/2016"
	wacn.date="06/29/2016"/>


# 使用 WebMatrix 构建 Node.js Web 应用并将其部署到 Azure

本教程向你演示如何使用 WebMatrix 开发 Node.js 应用程序并将其部署到 [Azure Web 应用](/documentation/services/web-sites/)中的 Web 应用。WebMatrix 是 Microsoft 提供的一个免费的 Web 开发工具，此工具包含进行 Web 应用开发所需的一切。WebMatrix 包含一些使您能够轻松使用 Node.js 的功能，包括代码完成、预构建模板以及对 Jade、LESS 和 CoffeeScript 的编辑器支持。了解有关 [WebMatrix](https://www.microsoft.com/web/webmatrix/) 的更多信息。

完成本指南之后，你将拥有一个在 Azure Web 应用中运行的 Node.js Web 应用。
 
以下是已完成应用程序的屏幕快照：

![Azure Node Web 应用][webmatrix-node-completed]

[AZURE.INCLUDE [create-account-and-websites-note](../includes/create-account-and-websites-note.md)]

## 登录 Azure

遵照以下步骤在 Azure Web 应用中创建 Web 应用。

1. 启动 WebMatrix
2. 登录到 [Azure 经典管理门户]，然后单击“[发布配置文件](https://manage.windowsazure.cn/publishsettings/index?client=powershell&schemaversion=1.0)”以下载帐户的发布配置文件。
2. 如果这是你首次使用 WebMatrix 3，系统将会提示你登录 Azure。您也可以单击“登录”按钮，并选择“添加帐户”。选择此项以使用 Microsoft 帐户**导入帐户**，并选择在上一步中下载的配置文件。

	![添加帐户][addaccount]

4. 系统将提示你输入帐户的名称。输入所需的任何内容，然后单击“确定”。


## 使用 Azure 的内置模板创建 Web 应用

1. 在开始屏幕上，单击“新建”按钮，然后选择“模板库”，从模板库创建新站点：

	![从模板库创建新 Web 应用][sitefromtemplate]

2. 在“从模板创建站点”对话框中，选择“节点”，然后选择“Express 站点”。最后单击“下一步”。如果你缺少“Express 站点”模板的任何必备组件，系统将提示你进行安装。

	![选择 Express 模板][webmatrix-templates]

3. 如果你已登录到 Azure 中，现在可以选择为本地站点创建 Azure Web 应用。选择一个唯一名称，然后选择要在其中创建 Azure Web 应用的数据中心：

	![在 Azure 中创建 Web 应用][nodesitefromtemplateazure]
	
4. 在 WebMatrix 完成生成本地站点和创建 Azure Web 应用后，将显示 WebMatrix IDE。

	![WebMatrix IDE][webmatrix-ide]

##将应用程序发布到 Azure

1. 在 WebMatrix 中，从“主页”功能区单击“发布”，为站点显示“发布预览”对话框。

	![发布预览][webmatrix-node-publishpreview]

2. 单击“继续”。发布完成后，Azure Web 应用的 URL 将显示在 WebMatrix IDE 底部

	![发布完成][webmatrix-publish-complete]

3. 单击链接以在浏览器中打开 Azure Web 应用。

	![Express Web 应用][webmatrix-node-express-site]

##修改并重新发布应用程序

您可轻松修改并重新发布应用程序。此处，你将简单更改 **index.jade** 文件的标题，然后重新发布应用程序。

1. 在 WebMatrix 中，选择“文件”，然后展开“views”文件夹。双击打开 **index.jade** 文件。

	![显示 index.jade 的 WebMatrix][webmatrix-modify-index]

2. 将段落行更改为以下内容：

		p Welcome to #{title} with WebMatrix on Azure!

3. 保存所做更改，然后单击发布图标。最后，在“发布预览”对话框中单击“继续”，并等待发布更新。

	![发布预览][webmatrix-republish]

4. 发布完成后，使用发布过程完成时返回的链接查看更新的 Azure Web 应用。

	![Azure Node Web 应用][webmatrix-node-completed]

##后续步骤

若要了解有关随 Azure 提供的 Node.js 版本的更多信息以及如何指定要与你的应用程序结合使用的版本，请参阅[在 Azure 应用程序中指定 Node.js 版本](/documentation/articles/nodejs-specify-node-version-azure-apps/)。

如果在将应用程序部署到 Azure 后遇到应用程序问题，请参阅[如何在 Azure Web 应用中调试 Node.js Web 应用](/documentation/articles/web-sites-nodejs-debug/)以了解有关诊断问题的信息。
[Azure 经典管理门户]: http://manage.windowsazure.cn
[使用 Git 发布 Azure Web 应用]: /documentation/articles/web-sites-publish-source-control/
[免费]: /zh-cn/pricing/1rmb-trial
[WebMatrix Web 应用]: http://www.microsoft.com/click/services/Redirect2.ashx?CR_CC=200106398
[WebMatrix for Azure]: http://go.microsoft.com/fwlink/?LinkID=253622&clcid=0x409

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
 

<!---HONumber=Mooncake_0118_2016-->