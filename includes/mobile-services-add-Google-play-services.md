

1. 通过单击 Android Studio 工具栏上的图标，或者通过单击“工具”->“Android”->“SDK Manager”，打开 Android SDK Manager。找到并打开在你的项目中使用的 Android SDK 目标版本，然后选择“Google API”（如果尚未安装）。

2. 向下滚动到“附加程序”，将其展开，然后选择“Google Play 服务”，如下所示。单击“安装包”。记下 SDK 路径，以备在下面的步骤中使用。

   	![](./media/notification-hubs-android-get-started/notification-hub-create-android-app4.png)


3. 打开应用目录中的 **build.gradle** 文件。

	![](./media/mobile-services-android-get-started-push/android-studio-push-build-gradle.png)

4. 在 *dependencies* 下面添加以下行：

   		compile 'com.google.android.gms:play-services-base:6.5.87'

5. 在 *defaultConfig* 下面，将 *minSdkVersion* 更改为 9。
 
6. 在工具栏中，单击“将项目与 Gradle 文件同步”图标。

7. 打开 **AndroidManifest.xml** 并将此标记添加到 *application* 标记中。

        <meta-data android:name="com.google.android.gms.version"
            android:value="@integer/google_play_services_version" />
 

<!---HONumber=71-->