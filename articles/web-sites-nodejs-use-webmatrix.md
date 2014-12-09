<properties linkid="web-site-with-webmatrix" urlDisplayName="Web site with WebMatrix" pageTitle="使用 WebMatrix 构建 Node.js 网站 - Azure 教程" metaKeywords="" description="本教程将介绍如何使用 WebMatrix 开发 Node.js 应用程序并将其部署到 Azure 网站。" metaCanonical="" services="web-sites" documentationCenter="Node.js" title="使用 WebMatrix 构建 Node.js 网站并将其部署到 Azure" authors="larryf" solutions="" manager="paulettm" editor="mollybos" />

# 使用 WebMatrix 构建 Node.js 网站并将其部署到 Azure

本教程将介绍如何使用 WebMatrix 开发 Node.js 应用程序并将其部署到 Azure 网站。WebMatrix 是 Microsoft 提供的一类免费的 Web 开发工具，此工具包含进行网站开发所需的一切。WebMatrix 包含一些使您能够轻松使用 Node.js 的功能，包括代码完成、预构建的模板以及对 Jade、LESS 和 CoffeeScript 的编辑器支持。了解有关 [WebMatrix for Azure][WebMatrix for Azure] 的详细信息。

完成本指南之后，您将拥有一个在 Azure 中运行的 Node.js 网站。

以下是已完成应用程序的屏幕快照：

![Azure Node 网站][Azure Node 网站]

[WACOM.INCLUDE [create-account-and-websites-note](../includes/create-account-and-websites-note.md)]

## 登录 Azure

按照以下步骤创建 Azure 网站。

<div class="dev-callout">

**说明**
若要完成本教程，你需要一个启用了 Azure 网站功能的 Azure 帐户。

如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 \<a href="http://www.windowsazure.cn/zh-cn/pricing/free-trial/\>Azure 免费试用版\</a\>。\</p\> \</div\> \<p\>\<br /\>\</p\> \<ol\> \<li\>启动 WebMatrix\</li\> \<li\>\<p\>如果这是您首次使用 WebMatrix，系统将提示您登录 Azure。您也可以单击“登录”按钮，并选择“添加帐户”。\<strong\>\</strong\>\<strong\>\</strong\>选择使用您的 Microsoft 帐户“登录”\<strong\>\</strong\>。\</p\> \<p\>\<img src="./media/web-sites-nodejs-use-webmatrix/webmatrix-add-account.png" alt="添加帐户" /\>\</p\>\</li\> \<li\>\<p\>如果您已注册 Azure 帐户，您可以使用 Microsoft 帐户登录：\</p\> \<p\>\<img src="./media/web-sites-nodejs-use-webmatrix/webmatrix-sign-in.png" alt="登录到 Azure" /\>\</p\>\</li\> \</ol\> \<h2 id="create-a-site-using-a-built-in-template-for-azure"\>使用 Azure 内置模板创建站点\</h2\> \<ol\> \<li\>\<p\>在开始屏幕上，单击“新建”按钮，然后选择“模板库”，从模板库创建新站点：\<strong\>\</strong\>\<strong\>\</strong\>\</p\> \<p\>\<img src="./media/web-sites-nodejs-use-webmatrix/webmatrix-site-from-template.png" alt="从模板库新建站点" /\>\</p\>\</li\> \<li\>\<p\>在“从模板创建站点”对话框中，选择“Node”，然后选择“Express 网站”。\<strong\>\</strong\>\<strong\>\</strong\>\<strong\>\</strong\>最后单击“下一步”。\<strong\>\</strong\>如果您缺少“Express 网站”模板的任何先决条件，系统将提示您进行安装。\<strong\>\</strong\>\</p\> \<p\>\<img src="./media/web-sites-nodejs-use-webmatrix/webmatrix-templates.png" alt="选择 Express 模板" /\>\</p\>\</li\> \<li\>\<p\>如果您已登录到 Azure，现在您可以选择为本地站点创建 Azure 网站。选择一个唯一名称，然后选择要用于创建站点的数据中心：\</p\> \<p\>\<img src="./media/web-sites-nodejs-use-webmatrix/webmatrix-node-site-azure.png" alt="在 Azure 上创建站点" /\>\</p\>\</li\> \<li\>\<p\>WebMatrix 构建网站完成之后，将显示 WebMatrix IDE。\</p\> \<p\>\<img src="./media/web-sites-nodejs-use-webmatrix/webmatrix-ide.png" alt="WebMatrix IDE" /\>\</p\>\</li\> \</ol\> \<h2 id="publish-your-application-to-azure"\>将应用程序发布到 Azure\</h2\> \<ol\> \<li\>\<p\>在 WebMatrix 中，从“主页”功能区中单击“发布”，显示网站的“发布预览”对话框。\<strong\>\</strong\>\<strong\>\</strong\>\<strong\>\</strong\>\</p\> \<p\>\<img src="./media/web-sites-nodejs-use-webmatrix/webmatrix-publishpreview.png" alt="发布预览" /\>\</p\>\</li\> \<li\>\<p\>单击“继续”。\<strong\>\</strong\>发布完成后，网站在 Azure 上的 URL 将显示在 WebMatrix IDE 底部。\</p\> \<p\>\<img src="./media/web-sites-nodejs-use-webmatrix/webmatrix-publish-complete.png" alt="发布完成" /\>\</p\>\</li\> \<li\>\<p\>单击该链接，在浏览器中打开网站。\</p\> \<p\>\<img src="./media/web-sites-nodejs-use-webmatrix/webmatrix-express-webiste.png" alt="Express 网站" /\>\</p\>\</li\> \</ol\> \<h2 id="modify-and-republish-your-application"\>修改并重新发布应用程序\</h2\> \<p\>您可以轻松地修改并重新发布应用程序。此处，您将简单更改 \<strong\>index.jade\</strong\> 文件的标题，然后重新发布应用程序。\</p\> \<ol\> \<li\>\<p\>在 WebMatrix 中，选择“文件”，然后展开“views”文件夹。\<strong\>\</strong\>\<strong\>\</strong\>双击打开 \<strong\>index.jade\</strong\> 文件。\</p\> \<p\>\<img src="./media/web-sites-nodejs-use-webmatrix/webmatrix-node-edit.png" alt="查看 index.jade 的 WebMatrix" /\>\</p\>\</li\> \<li\>\<p\>将第二行更改为以下内容：\</p\> \<pre\>\<code data-inline="1"\>p Welcome to \#{title} with WebMatrix on Azure!\</code\>\</pre\>\</li\> \<li\>\<p\>保存更改，然后单击发布图标。最后，在“发布预览”对话框中单击“继续”，并等待发布更新。\<strong\>\</strong\>\<strong\>\</strong\>\</p\> \<p\>\<img src="./media/web-sites-nodejs-use-webmatrix/webmatrix-republish.png" alt="发布预览" /\>\</p\>\</li\> \<li\>\<p\>发布完成后，使用发布过程完成时返回的链接查看更新的站点。\</p\> \<p\>\<img src="./media/web-sites-nodejs-use-webmatrix/webmatrix-node-complete.png" alt="Azure Node 网站" /\>\</p\>\</li\> \</ol\> \<h2 id="next-steps"\>后续步骤\</h2\> \<p\>要了解有关随 Azure 提供的 Node.js 版本的更多信息以及如何指定要与您的应用程序结合使用的版本，请参阅\<a href="/zh-cn/documentation/articles/nodejs-specify-node-version-azure-apps/"\>在 Azure 应用程序中指定 Node.js 版本\</a\>。\</p\> \<p\>如果您将应用程序部署到 Azure 后遇到问题，有关如何诊断问题的信息，请参阅\<a href="http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-nodejs-debug/"\>如何在 Azure 网站中调试 Node.js 应用程序\</a\>。\</p\>

</div>

  [WebMatrix for Azure]: http://go.microsoft.com/fwlink/?LinkID=253622&clcid=0x409
  [Azure Node 网站]: ./media/web-sites-nodejs-use-webmatrix/webmatrix-node-complete.png
