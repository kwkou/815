<properties linkid="develop-mobile-how-to-guides-import-publish-settings" urlDisplayName="Import your subscription publish settings file in Visual Studio 2013" pageTitle="Import your publish settings file in Visual Studio 2013 | Mobile Services" metaKeywords="Azure import publishsettings, mobile services" description="Learn how to import a subscription publish settings file for your Azure Mobile Services application in Visual Studio 2013." title="Import your subscription publish settings file in Visual Studio 2013" documentationCenter="Mobile" services="" solutions="" manager="" editor="" videoId="" scriptId="" authors="" />

# 在 Visual Studio 2013 中导入订阅发布设置文件

在你能够创建移动服务之前，必须将来自 Azure 订阅的发布设置文件导入 Visual Studio。这使 Visual Studio 能够代表你连接到 Azure。

1.  在 Visual Studio 2013 中，打开解决方案资源管理器，右键单击项目，单击“添加”，然后单击“连接的服务...” 。

    ![添加连接的服务](./media/mobile-services-create-new-service-vs2013/mobile-add-connected-service.png)

2.  在“服务管理器”对话框中，单击“创建服务...” ，然后从“创建移动服务”对话框的“订阅”中选择“\<导入...\>”。 

    ![从 VS 2013 创建新移动服务](./media/mobile-services-create-new-service-vs2013/mobile-create-service-from-vs2013.png)

3.  在“导入 Azure 订阅”中，单击“下载订阅文件” ，登录到你的 Azure 帐户（如果需要），当浏览器请求保存文件时，单击“保存”。 

    ![在 VS 中下载订阅文件](./media/mobile-services-create-new-service-vs2013/mobile-import-azure-subscription.png)

    <div class="dev-callout"><strong>说明</strong> <p>登录窗口显示在浏览器中，可能位于 Visual Studio 窗口后面。请记住，应记下你保存下载的 .publishsettings 文件的位置。如果你的项目已连接到 Azure 订阅，可以跳过此步骤。</p></div>

4.  单击“浏览” ，导航到你保存 .publishsettings 文件的位置，选择该文件，然后单击“打开”， 再单击“导入” 。

    ![在 VS 中导入订阅](./media/mobile-services-create-new-service-vs2013/mobile-import-azure-subscription-2.png)

    Visual Studio 导入连接到你的 Azure 订阅所需的数据。当你的订阅已经具有一个或多个现有移动服务时，则会显示服务名称。

	<div class="dev-callout"><b>安全说明</b>

    <p>导入发布设置之后，应考虑删除所下载的 .publishsettings 文件，因为该文件包含可供他人用来访问你的帐户的信息。如果你打算保留该文件，以备在其他连接的应用程序项目中使用，请保护该文件的安全。</p>
	</div>

  [添加连接的服务]: ./media/mobile-services-create-new-service-vs2013/mobile-add-connected-service.png
  [从 VS 2013 创建新移动服务]: ./media/mobile-services-create-new-service-vs2013/mobile-create-service-from-vs2013.png
  [在 VS 中下载订阅文件]: ./media/mobile-services-create-new-service-vs2013/mobile-import-azure-subscription.png
  [在 VS 中导入订阅]: ./media/mobile-services-create-new-service-vs2013/mobile-import-azure-subscription-2.png
