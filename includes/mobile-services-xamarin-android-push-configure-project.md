
1. 在"解决方案"视图中，展开 Xamarin.Android 应用程序中的 **Components** 文件夹，确保 Azure 移动服务包已安装。 

2. 右键单击 **Components** 文件夹，单击"获取更多组件..."，搜索 **Google Cloud Messaging Client** 组件，将其添加到项目中。 

1. 打开 ToDoActivity.cs 项目文件，将以下 using 语句添加到类：

		using Gcm.Client;

2. 在 **ToDoActivity** 类中，添加以下新代码： 

        // Create a new instance field for this activity.
        static ToDoActivity instance = new ToDoActivity();

        // Return the current activity instance.
        public static ToDoActivity CurrentActivity
        {
            get
            {
                return instance;
            }
        }
        // Return the Mobile Services client.
        public MobileServiceClient CurrentClient
        {
            get
            {
                return client;
            }
        }

	这样你就可以通过服务流程访问移动服务客户端实例。

3. 将现有的移动服务客户端声明更改为公开，如下所示：

		public MobileServiceClient client { get; private set; }

4.	创建 **MobileServiceClient** 后，将以下代码添加到 **OnCreate** 方法：

        // Set the current instance of TodoActivity.
        instance = this;

        // Make sure the GCM client is set up correctly.
        GcmClient.CheckDevice(this);
        GcmClient.CheckManifest(this);

        // Register the app for push notifications.
        GcmClient.Register(this, ToDoBroadcastReceiver.senderIDs);

你的 **ToDoActivity** 现已准备就绪，可以添加推送通知了。

<!---HONumber=56-->