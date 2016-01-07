1. 通过单击 Android Studio 工具栏上的图标，或者通过单击“工具”->“Android”->“SDK Manager”，打开 Android SDK Manager。找到并打开在你的项目中使用的 Android SDK 目标版本，然后选择“Google API”（如果尚未安装）。

2. 单击“SDK 工具”选项卡。如果尚未安装 Google Play 服务，请单击“Google Play 服务”，如下所示。然后单击“应用”进行安装。
 
	记下 SDK 路径，因为后面的步骤将要用到。

   	![](./media/notification-hubs-android-studio-add-google-play-services/notification-hubs-android-studio-sdk-manager.png)


3. 打开应用目录中的 **build.gradle** 文件。

	![](./media/notification-hubs-android-studio-add-google-play-services/notification-hubs-android-studio-add-google-play-dependency.png)

4. 在 *android* 下面添加以下行：

		useLibrary 'org.apache.http.legacy'

5. 在 *dependencies* 下面添加以下行：

   		compile 'com.google.android.gms:play-services-base:6.5.87'

7. 在 *defaultConfig* 下面，将 *minSdkVersion* 更改为 9。
 
8. 在工具栏中，单击“将项目与 Gradle 文件同步”图标。

9. 打开 **AndroidManifest.xml** 并将以下标记添加到 *application* 标记中。

        <meta-data android:name="com.google.android.gms.version"
            android:value="@integer/google_play_services_version" />
 

<!---HONumber=Mooncake_1207_2015-->