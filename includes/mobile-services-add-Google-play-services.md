

1. 通过单击 Eclipse 顶部工具栏的"窗口"，打开 Android SDK 管理器。找到并打开在您的项目属性中指定的 Android SDK 目标版本，然后选择"Google API"。

2. 向下滚动到"附加程序"，将其展开，然后选择"Google Play 服务"，如下所示。单击"安装包"。记下 SDK 路径，以备在下面的步骤中使用。重新启动 Eclipse。

   	![](./media/notification-hubs-android-get-started/notification-hub-create-android-app4.png)


3. 在您的项目中安装 Google Play 服务 SDK。在 Eclipse 中，单击"文件"，然后单击"导入"。选择"Android"，然后选择"现有 Android 代码到工作区"并单击"下一步"。单击"浏览"、导航到 Android SDK 路径 （通常位于包含 Eclipse 的文件夹内的名为  `adt-bundle-windows-x86_64` 的文件夹中）、转到  `\extras\google\google_play_services\libproject` 子文件夹、选择 google-play-services-lib 文件夹，然后单击"确定"。选中"将项目复制到工作区"复选框，然后单击"完成"。

	![](./media/mobile-services-android-get-started-push/mobile-eclipse-import-Play-library.png)

4. 接下来，您必须从项目引用刚才导入的 Google Play 服务 SDK 库。 

5. 在"程序包资源管理器"中，右键单击您的项目并选择  *Properties*。
 
6. 在"属性"窗口中，选择左侧的"Android"。

	![](./media/mobile-services-android-get-started-push/mobile-google-set-project-properties.png)


7. 在"项目生成目标"下，确保为相应 SDK 级别选择了  `Google APIs x86`（或 `Google APIs`，具体取决于您的开发平台）。

 
8. 在"库"部分中，选择"添加"，然后选择"Google Play 服务"项目 (*google-play-services-lib*) 并单击"确定"。

9. 单击"应用"，然后单击"确定"。



<!--HONumber=41-->
