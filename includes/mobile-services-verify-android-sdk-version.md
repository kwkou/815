由于 Android SDK 的持续开发，安装在 Android Studio 中的 Android SDK 版本可能与你的代码中的版本不匹配。本教程中引用的 Android SDK 是版本 21，也是撰写本教程时的最新版本。随着 SDK 的新版本问世，版本号也会随之增加，我们建议使用最新版本。

版本不匹配的两种表现是：

1. 当你生成或重新生成项目时，可能会收到 类似的错误消息“找不到目标 Google Inc.:Google APIs:n”的 Gradle 错误消息。

2. 应该基于  `import` 语句解析的代码中的标准 Android 对象可能产生错误消息。

如果出现上述任何一种情况，则说明在 Android Studio 中安装的 Android SDK 版本与所下载项目的 SDK 目标可能不匹配。若要验证版本，请进行以下更改：


3. 在 Android Studio 中，单击“工具”=>“Android”=>“SDK Manager”。如果您尚未安装最新版本的 SDK 平台，请单击安装。记下版本号。

4. 在“项目资源管理器”选项卡中的“Gradle 脚本”下，打开文件 **build.gradle (modeule: app)**。确保 **compileSdkVersion** 和 **buildToolsVersion** 设置为安装的最新 SDK 版本。标记可能如下所示：
 
	 	    compileSdkVersion 'Google Inc.:Google APIs:21'
    		buildToolsVersion "21.1.2"
	
5. 在 Android Studio 项目资源管理器中，右键单击项目节点，选择“属性”，并在左栏中选择“Android”。确认“项目生成目标”设置为与 **targetSdkVersion** 相同的 SDK 版本。

6. 与 Eclipse 的情况不同，在 Android Studio 中，清单文件不再用于指定目标 SDK 版本和最低 SDK 版本。

<!---HONumber=71-->