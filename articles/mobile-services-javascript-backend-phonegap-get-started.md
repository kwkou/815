<properties
	pageTitle="使用 Azure 移动服务开发 PhoneGap/cordova 应用入门 | Microsoft Azure"
	description="请按照本教程中的说明操作，开始使用用于 PhoneGap 开发的 Azure 移动服务（面向 iOS、, Android 和 Windows Phone）。"
	services="mobile-services"
	documentationCenter=""
	authors="ggailey777"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="mobile-services"
	ms.date="02/10/2016"
	wacn.date="03/21/2016"/>

#  移动服务入门

[AZURE.INCLUDE [mobile-services-selector-get-started](../includes/mobile-services-selector-get-started.md)]
&nbsp;

[AZURE.INCLUDE [mobile-services-hero-slug](../includes/mobile-services-hero-slug.md)]

本教程说明如何使用 Azure 移动服务向应用程序添加基于云的后端服务。在本教程中，你将要创建一个新的移动服务，以及一个在新移动服务中存储应用程序数据的简单_待办事项列表_应用程序。

以下是完成的应用程序的屏幕快照：

![][3]

###  其他要求

完成本教程需要使用：

+ PhoneGap 工具（Windows Phone 8 项目需要 v3.2+）。

+ 有效的 Microsoft Azure 帐户。

+ PhoneGap 支持针对多个平台进行开发。除了 PhoneGap 工具本身以外，还必须为你所要针对的每个平台安装工具：

	- Windows Phone：安装 [Visual Studio 2012 Express for Windows Phone](https://go.microsoft.com/fwLink/p/?LinkID=268374)
	- iOS：安装 [Xcode]（需要 v4.4+）
	- Android：[安装 Android 开发人员工具][Android SDK]
	<br/>（适用于 Android 的移动服务 SDK 支持用于 Android 2.2 或更高版本的应用程序。运行快速入门应用程序需要安装 Android 4.2 或更高版本。)

##  创建新的移动服务

[AZURE.INCLUDE [mobile-services-create-new-service](../includes/mobile-services-create-new-service.md)]

##  创建新的 PhoneGap 应用程序

在本部分中，你将要创建一个连接到移动服务的新的 PhoneGap 应用程序。

1.  在 [Azure 经典门户]中单击“移动服务”，然后单击你刚刚创建的移动服务。

2. 在快速启动选项卡中，单击“选择平台”下的“PhoneGap”，然后展开“创建新的 PhoneGap 应用程序”。

   	![][0]

   	此时将显示三个简单步骤，描述如何创建与移动服务连接的 PhoneGap 应用程序。

  	![][1]

3. 下载并安装 PhoneGap 以及至少一个平台开发工具（Windows Phone、iOS 或 Android）（如果尚未这么做）。

4. 单击“创建 TodoItem 表”以创建用于存储应用程序数据的表。

5. 在“下载并运行应用程序”下面单击“下载”。

	随即将会下载已连接到移动服务的示例_待办事项列表_应用程序的项目，以及移动服务 JavaScript SDK。将压缩的项目文件保存到本地计算机，并记下保存位置。

##  运行新的 PhoneGap 应用程序

本教程的最后一个阶段是生成和运行你的新应用程序。

1.	浏览到压缩的项目文件所保存到的位置，然后在计算机上展开这些文件。 

2.	对于每个平台，请根据下面的说明打开并运行该项目。

	+ **Windows Phone 8**

	1. Windows Phone 8：在 Visual Studio 2012 Express for Windows Phone 中，打开 **platforms\\wp8** 文件夹中的 .sln 文件。
	
	2. 按 **F5** 键以重新构建项目并启动此应用。
	
	  	![][2]

	+ **iOS**

	1. 在 Xcode 中，打开 **platforms/ios** 文件夹中的项目。
	
	2. 按“运行”按钮以生成项目，并在 iPhone 模拟器中启动应用，这是此项目的默认设置。
	
	  	![][3]

	+ **Android**

		1. 在 Eclipse 中，依次单击“文件”、“导入”，展开“Android”，单击“工作区中的现有 Android 代码”，然后单击“下一步”。 
		
		2. 单击“浏览”，浏览到已展开项目文件所在的位置，单击“确定”，并确认 TodoActivity 项目已选中，然后单击“完成”。<p>这样可将项目文件导入当前工作区。</p>
		
		3. 在“运行”菜单中，单击“运行”，以便在 Android 模拟器中启动项目。
		
			![][4]
	
		>[AZURE.NOTE]若要在 Android 模拟器中运行项目，必须至少定义一个 Android 虚拟设备 (AVD)。使用 AVD 管理器创建和管理这些设备。
			
	
3. 在上述某个移动模拟器中启动应用程序后，在文本框中键入一些文本，然后单击“添加”。

	这样可向在 Azure 中托管的新移动服务发送 POST 请求。来自请求的数据将插入到 **TodoItem** 表。移动服务返回存储在表中的项，数据显示在列表中。

	> [AZURE.IMPORTANT]如果使用 PhoneGap 工具重新生成主项目，将会覆盖对此平台项目所做的更改。请根据以下部分中所述，在项目的 www 根目录中进行更改。

4. 返回 [Azure 经典门户]，单击“数据”选项卡，然后单击“TodoItem”表。

	![](./media/mobile-services-javascript-backend-phonegap-get-started/mobile-data-tab.png)

	这使您可以浏览此应用插入表中的数据。

	![](./media/mobile-services-javascript-backend-phonegap-get-started/mobile-data-browse.png)
	

##  针对每个平台进行应用程序更新并重新生成项目

1. 在“www”目录（在本例中为“todolist/www”）中对代码文件进行更改。

2. 验证是否可以访问系统路径中的所有目标平台工具。

2. 在项目根目录中打开命令提示符，然后运行下列平台特定的命令之一：

	+ **Windows Phone**

		在 Visual Studio Developer 命令提示符下运行以下命令：

    		phonegap local build wp8

	+ **iOS**
 
		打开终端并运行以下命令：

    		phonegap local build ios

	+ **Android**

		打开命令提示符或终端窗口并运行以下命令：

		    phonegap local build android

4. 根据前面部分中所述，在相应的开发环境中打开每个项目。

>[AZURE.NOTE]你可以查看访问你的移动服务以查询和插入数据的代码，这些代码在 js/index.js 文件中。

##  后续步骤
完成快速入门后，请了解如何在移动服务中执行其他重要任务：

* **[向应用程序添加身份验证]**
了解如何使用标识提供者对应用程序的用户进行身份验证。  

* **[向应用程序添加推送通知](https://msdn.microsoft.com/magazine/dn879353.aspx)**
了解如何向应用程序注册和发送推送通知。

* **[移动服务 HTML/JavaScript 操作方法概念性参考](/documentation/articles/mobile-services-html-how-to-use-client-library/)**
了解如何使用 JavaScript 客户端库来访问数据、调用自定义 API 和执行身份验证。

[AZURE.INCLUDE [app-service-disqus-feedback-slug](../includes/app-service-disqus-feedback-slug.md)]

<!-- Images. -->
[0]: ./media/mobile-services-javascript-backend-phonegap-get-started/portal-screenshot1.png
[1]: ./media/mobile-services-javascript-backend-phonegap-get-started/portal-screenshot2.png
[2]: ./media/mobile-services-javascript-backend-phonegap-get-started/mobile-portal-quickstart-wp8.png
[3]: ./media/mobile-services-javascript-backend-phonegap-get-started/mobile-portal-quickstart-ios.png
[4]: ./media/mobile-services-javascript-backend-phonegap-get-started/mobile-portal-quickstart-android.png

<!-- URLs. -->
[向应用程序添加身份验证]: /documentation/articles/mobile-services-html-get-started-users/
[Android SDK]: https://go.microsoft.com/fwLink/p/?LinkID=280125
[Azure 经典门户]: https://manage.windowsazure.cn/
[Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[Visual Studio 2012 Express for Windows Phone]: https://go.microsoft.com/fwLink/p/?LinkID=268374

<!---HONumber=Mooncake_0118_2016-->