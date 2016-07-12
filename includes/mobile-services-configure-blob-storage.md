注册新的插入脚本，当插入新的 Todo 项时，它可以生成 SAS。

0. 如果你尚未创建你的存储帐户，请参阅[如何创建存储帐户](/documentation/articles/storage-create-storage-account/)。

1. 在 [Azure 管理门户](https://manage.windowsazure.cn/)中，单击“存储”，单击存储帐户，然后单击“管理密钥”。

2. 记下“存储帐户名称”和“访问密钥”。

   	![](./media/mobile-services-configure-blob-storage/mobile-blob-storage-account-keys.png)

3. 在你的移动服务中，单击“配置”选项卡、向下滚动到“应用设置”、输入你从存储帐户获取的下述每个项的“名称”和“值”对，然后单击“保存”。

	+ `STORAGE_ACCOUNT_NAME`
	+ `STORAGE_ACCOUNT_ACCESS_KEY`

	![](./media/mobile-services-configure-blob-storage/mobile-blob-storage-app-settings.png)

	存储帐户访问密钥将会加密存储在应用设置中。你可在运行时从任何服务器脚本访问此密钥。有关详细信息，请参阅[应用设置]。

4. 在“配置”选项卡中，请确保已启用“[动态架构](http://msdn.microsoft.com/zh-cn/library/windowsazure/b6bb7d2d-35ae-47eb-a03f-6ee393e170f7)”。你需要启用动态架构以能够将新列添加到 TodoItem 表。不应在任何生产服务中启用动态架构。

5. 单击“数据”选项卡，然后单击 **TodoItem** 表。


6.  在 **todoitem** 中，单击“脚本”选项卡，然后选择“插入”，将 insert 函数替换为以下代码，然后单击“保存”：

		var azure = require('azure');
		var qs = require('querystring');
		var appSettings = require('mobileservice-config').appSettings;
		
		function insert(item, user, request) {
		    // Get storage account settings from app settings. 
		    var accountName = appSettings.STORAGE_ACCOUNT_NAME;
		    var accountKey = appSettings.STORAGE_ACCOUNT_ACCESS_KEY;
		    var host = accountName + '.blob.core.chinacloudapi.cn';
		
		    if ((typeof item.containerName !== "undefined") && (
		    item.containerName !== null)) {
		        // Set the BLOB store container name on the item, which must be lowercase.
		        item.containerName = item.containerName.toLowerCase();
		
		        // If it does not already exist, create the container 
		        // with public read access for blobs.        
		        var blobService = azure.createBlobService(accountName, accountKey, host);
		        blobService.createContainerIfNotExists(item.containerName, {
		            publicAccessLevel: 'blob'
		        }, function(error) {
		            if (!error) {
		
		                // Provide write access to the container for the next 5 mins.        
		                var sharedAccessPolicy = {
		                    AccessPolicy: {
		                        Permissions: azure.Constants.BlobConstants.SharedAccessPermissions.WRITE,
		                        Expiry: new Date(new Date().getTime() + 5 * 60 * 1000)
		                    }
		                };
		
		                // Generate the upload URL with SAS for the new image.
		                var sasQueryUrl = 
		                blobService.generateSharedAccessSignature(item.containerName, 
		                item.resourceName, sharedAccessPolicy);
		
		                // Set the query string.
		                item.sasQueryString = qs.stringify(sasQueryUrl.queryString);
		
		                // Set the full path on the new new item, 
		                // which is used for data binding on the client. 
		                item.imageUri = sasQueryUrl.baseUrl + sasQueryUrl.path;
		
		            } else {
		                console.error(error);
		            }
		            request.execute();
		        });
		    } else {
		        request.execute();
		    }
		}

   	这样可将当 TodoItem 表中发生插入时所调用的函数替换为新脚本。此新脚本将为插入生成新 SAS（它的有效时间为 5 分钟）并将生成的 SAS 的值分配给所返回项目的 `sasQueryString` 属性。还将 `imageUri` 属性设置为新 BLOB 的资源路径，以便在绑定时在客户端 UI 中启用图像显示。

	>[AZURE.NOTE] 这段代码为单个 BLOB 创建 SAS。如果你需要使用同一个 SAS 将多个 blob 上载到容器，可以改为使用空 blob 资源名称调用 [generateSharedAccessSignature 方法](http://go.microsoft.com/fwlink/?LinkId=390455)</a>，如下所示：
	>                 
	>     blobService.generateSharedAccessSignature(containerName, '', sharedAccessPolicy);

接下来，你将更新快速启动应用，通过使用在发生插入时生成的 SAS，添加图像上载功能。
 
<!-- Anchors. -->

<!-- Images. -->

<!-- URLs. -->
[应用设置]: http://msdn.microsoft.com/zh-cn/library/windowsazure/b6bb7d2d-35ae-47eb-a03f-6ee393e170f7

<!---HONumber=Mooncake_0118_2016-->