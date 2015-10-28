由于 Android SDK 的持续开发，安装在 Eclipse 中的 Android SDK 版本可能与你的代码中的版本不匹配。本教程中引用的 Android SDK 是版本 21，也是撰写本教程时的最新版本。随着 SDK 的新版本问世，版本号也会随之增加，我们建议使用最新版本。

版本不匹配的两种表现是：

1. 查看底部窗格中的 Eclipse 控制台。你可能看到以下形式的错误消息：“无法解析目标‘android-n’”。

2. 应该基于 `import` 语句解析的代码中的标准 Android 对象可能产生错误消息。

如果出现上述任何一种情况，则说明在 Eclipse 中安装的 Android SDK 版本与所下载项目的 SDK 目标可能不匹配。若要验证版本，请进行以下更改：


3. 在 Eclipse 中，单击“窗口”，然后单击“Android SDK 管理器”。如果你尚未安装最新版本的 SDK 平台，请单击安装。记下版本号。

4. 打开项目文件 **AndroidManifest.xml**。确认在 **uses-sdk** 元素中，**targetSdkVersion** 设置为安装的最新版本。**uses-sdk** 标记可能如下所示：
 
	 	    <uses-sdk
	 	        android:minSdkVersion="8"
	 	        android:targetSdkVersion="21" />
	
5. 在 Eclipse 程序包资源管理器中，右键单击项目节点，选择“属性”，并在左栏中选择“Android”。确认“项目生成目标”设置为与 **targetSdkVersion** 相同的 SDK 版本。

<!---HONumber=74-->