本教程的最后一个阶段是生成并运行你的新应用程序。

1. 浏览到你保存压缩项目文件的位置，并在计算机上将这些文件展开到 Android Studio 项目目录。

2. 打开 Android Studio。如果你正在使用某个项目并且显示该项目，请关闭该项目（"文件"= >"关闭项目"）。

3. 选择**"打开现有 Android Studio 项目"**，浏览到该项目位置，然后单击**"确定"**。 

 	![][14]

4. 在左侧**"项目资源管理器"**窗口中，确保选择了 *"项目"*选项卡，然后依次打开**"应用程序"**、**"src"**、**"java"**，并双击 **ToDoactivity**，

   	![][8]


4. 如果你下载了 SDK 版本 2.0，则需要使用移动服务的 Url 和密钥更新代码：
	- 	在 **TodoActivity.java** 中找到 **OnCreate** 方法，并找到实例化移动服务客户端的代码。上图中显示了该代码。
	- 	将"MobileServiceUrl"替换为移动服务的实际 Url。
	- 	将"AppKey"替换为移动服务的密钥。
	- 	有关详细信息，请查阅教程<a href="/documentation/articles/mobile-services-android-get-started-data/">将移动服务添加到现有应用程序</a>。 

5. 在"运行"菜单中，单击"运行"，以便在 Android 模拟器中启动项目。

	> [AZURE.NOTE] 若要在 Android 模拟器中运行项目，必须至少定义一个 Android 虚拟设备 (AVD)。使用 AVD 管理器创建和管理这些设备。

6. 在应用程序中键入有意义的文本（例如 _Complete the tutorial_），然后单击**"添加"**。

   	![][10]

   	这样可向在 Azure 中托管的新移动服务发送 POST 请求。请求中的数据将插入到 TodoItem 表中。移动服务返回存储在表中的项，数据显示在列表中。

	> [AZURE.NOTE] 你可以查看访问你的移动服务以查询和插入数据的代码，这些代码在 ToDoActivity.java 文件中。

6. 返回管理门户，单击"数据"选项卡，然后单击**TodoItem** 表。

   	![][11]

   	此时，你可以浏览应用程序在表中插入的数据。

   	![][12]


<!-- Images. -->
[0]: ./media/mobile-services-android-get-started/mobile-quickstart-completed-android.png
[6]: ./media/mobile-services-android-get-started/mobile-portal-quickstart-android.png
[7]: ./media/mobile-services-android-get-started/mobile-quickstart-steps-android.png
[8]: ./media/mobile-services-android-get-started/Android-Studio-quickstart.png
[10]: ./media/mobile-services-android-get-started/mobile-quickstart-startup-android.png
[11]: ./media/mobile-services-android-get-started/mobile-data-tab.png
[12]: ./media/mobile-services-android-get-started/mobile-data-browse.png
[14]: ./media/mobile-services-android-get-started/android-studio-import-project.png
[15]: ./media/mobile-services-android-get-started/mobile-services-import-android-project.png

<!-- URLs. -->
[数据处理入门]: /documentation/articles/mobile-services-android-get-started-data/
[身份验证入门]: /documentation/articles/mobile-services-android-get-started-users/
[推送通知入门]: /documentation/articles/mobile-services-javascript-backend-android-get-started-push/
[Android SDK]: http://developer.android.com/sdk/
[Android Studio]: https://developer.android.com/sdk/index.html
[移动服务 Android SDK]: https://github.com/Azure/azure-mobile-services/blob/master/CHANGELOG.ios.md#sdk-downloads

[管理门户]: https://manage.windowsazure.cn/

<!--HONumber=50-->