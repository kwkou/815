
##在移动服务项目中安装存储客户端

要可以生成 SAS 来将图像上载到 Blob 存储，必须先添加 NuGet 软件包，该软件包将存储客户端库安装在移动服务项目中。

1. 在 Visual Studio 中的“解决方案资源管理器”内，右键单击移动服务项目，然后选择“管理 NuGet 包”。

2. 在左窗格中，选择“联机”类别，选择“仅限稳定版”，搜索 **WindowsAzure.Storage**，在“Azure 存储空间”包上单击“安装”，然后接受许可协议。

  	![](./media/mobile-services-configure-blob-storage/mobile-add-storage-nuget-package-dotnet.png)

  	这会将 Azure 存储服务的客户端库添加到移动服务项目。

##更新数据模型中的 TodoItem 定义

TodoItem 类定义数据对象，并且您需要将相同属性添加到此类，正如您在客户端上的操作一样。

1. 在 Visual Studio 2013 中，打开您的移动服务项目，展开 DataObjects 文件夹，然后打开 TodoItem.cs 项目文件。
	
2. 将以下新属性添加到 **TodoItem** 类：

        public string containerName { get; set; }
		public string resourceName { get; set; }
		public string sasQueryString { get; set; }
		public string imageUri { get; set; } 

	这些属性用于生成 SAS 并存储映像信息。请注意，这些属性的大小写与 JavaScript 后端版本相匹配。

	>[AZURE.NOTE]使用默认数据库初始程序时，如果实体框架在代码优先模型定义中检测到数据模型更改，它就会删除并重新创建数据库。若要进行此数据模型更改并维护数据库中的现有数据，必须使用代码优先迁移。不能为 Azure 中的 SQL 数据库使用默认的初始值设定项。有关更多信息，请参阅[如何使用代码优先迁移来更新数据模型](/zh-cn/documentation/articles/mobile-services-dotnet-backend-how-to-use-code-first-migrations/)。

##更新 TodoItem 控制器以生成共享的访问签名 

现有 **TodoItemController** 已更新，以便在插入新的 TodoItem 时 **PostTodoItem** 方法生成 SAS。您还

0. 如果你尚未创建你的存储帐户，请参阅[如何创建存储帐户]。

1. 在 [Azure 管理门户](https://manage.windowsazure.cn/)中，单击“存储”，单击存储帐户，然后单击“管理密钥”。

2. 记下“存储帐户名称”和“访问密钥”。
 
3. 在你的移动服务中，单击“配置”选项卡、向下滚动到“应用设置”、输入你从存储帐户获取的下述每个项的“名称”和“值”对，然后单击“保存”。

	+ `STORAGE_ACCOUNT_NAME`
	+ `STORAGE_ACCOUNT_ACCESS_KEY`

	![](./media/mobile-services-configure-blob-storage/mobile-blob-storage-app-settings.png)

	存储帐户访问密钥将会加密存储在应用设置中。你可在运行时从任何服务器脚本访问此密钥。有关详细信息，请参阅[应用设置]。

4. 在 Visual Studio 中的 Solution Explorer 内，打开移动服务项目的 Web.config 文件并添加以下新应用设置，将占位符替换为您刚才在门户中设置的存储帐户名称和访问密钥：

		<add key="STORAGE_ACCOUNT_NAME" value="**your_account_name**" />
		<add key="STORAGE_ACCOUNT_ACCESS_KEY" value="**your_access_token_secret**" />

	移动服务在本地计算机上运行时将使用这些存储的设置，这让你在发布代码之前对代码进行测试。在 Azure 中运行时，移动服务将改用门户中设置的应用设置值，并忽略这些项目设置。

7.  在控制器文件夹中，打开 TodoItemController.cs 文件并添加以下 **using** 指令：

		using System;
		using Microsoft.WindowsAzure.Storage.Auth;
		using Microsoft.WindowsAzure.Storage.Blob;
  
8.  将现有 **PostTodoItem** 方法替换为以下代码：

        public async Task<IHttpActionResult> PostTodoItem(TodoItem item)
        {
            string storageAccountName;
            string storageAccountKey;

            // Try to get the Azure storage account token from app settings.  
            if (!(Services.Settings.TryGetValue("STORAGE_ACCOUNT_NAME", out storageAccountName) |
            Services.Settings.TryGetValue("STORAGE_ACCOUNT_ACCESS_KEY", out storageAccountKey)))
            {
                Services.Log.Error("Could not retrieve storage account settings.");
            }

            // Set the URI for the Blob Storage service.
            Uri blobEndpoint = new Uri(string.Format("https://{0}.blob.core.chinacloudapi.cn", storageAccountName));

            // Create the BLOB service client.
            CloudBlobClient blobClient = new CloudBlobClient(blobEndpoint, 
                new StorageCredentials(storageAccountName, storageAccountKey));

            if (item.containerName != null)
            {
                // Set the BLOB store container name on the item, which must be lowercase.
                item.containerName = item.containerName.ToLower();

                // Create a container, if it doesn't already exist.
                CloudBlobContainer container = blobClient.GetContainerReference(item.containerName);
                await container.CreateIfNotExistsAsync();

                // Create a shared access permission policy. 
                BlobContainerPermissions containerPermissions = new BlobContainerPermissions();

                // Enable anonymous read access to BLOBs.
                containerPermissions.PublicAccess = BlobContainerPublicAccessType.Blob;
                container.SetPermissions(containerPermissions);

                // Define a policy that gives write access to the container for 5 minutes.                                   
                SharedAccessBlobPolicy sasPolicy = new SharedAccessBlobPolicy()
                {
                    SharedAccessStartTime = DateTime.UtcNow,
                    SharedAccessExpiryTime = DateTime.UtcNow.AddMinutes(5),
                    Permissions = SharedAccessBlobPermissions.Write
                };

                // Get the SAS as a string.
                item.sasQueryString = container.GetSharedAccessSignature(sasPolicy); 

                // Set the URL used to store the image.
                item.imageUri = string.Format("{0}{1}/{2}", blobEndpoint.ToString(), 
                    item.containerName, item.resourceName);
            }

            // Complete the insert operation.
            TodoItem current = await InsertAsync(item);
            return CreatedAtRoute("Tables", new { id = current.Id }, current);
        }

   	此 POST 方法现在为插入项生成新的 SAS，有效时间为 5 分钟，然后将生成的 SAS 的值分配给所返回项目的 `sasQueryString` 属性。还将 `imageUri` 属性设置为新 BLOB 的资源路径，以便在绑定时在客户端 UI 中启用图像显示。

	>[AZURE.NOTE]这段代码为单个 BLOB 创建 SAS。如果需要使用同一个 SAS 将多个 Blob 上载到容器，你可以使用空 Blob 资源名称来调用 <a href="http://go.microsoft.com/fwlink/?LinkId=390455" target="_blank">generateSharedAccessSignature 方法</a>，如下所示：<pre><code>blobService.generateSharedAccessSignature(containerName, '', sharedAccessPolicy);</code></pre>

接下来，您将更新快速启动应用，通过使用在发生插入时生成的 SAS，添加图像上载功能。
 
<!-- Anchors. -->

<!-- Images. -->

<!-- URLs. -->
[如何创建存储帐户]: /zh-cn/documentation/articles/storage-create-storage-account
[应用设置]: http://msdn.microsoft.com/zh-cn/library/windowsazure/b6bb7d2d-35ae-47eb-a03f-6ee393e170f7

<!---HONumber=Mooncake_0118_2016-->