注册新的插入脚本，当插入新的 Todo 项时，它可以生成 SAS。

1.  如果你尚未创建存储帐户，请参阅[如何创建存储帐户][]。

2.  在管理门户中，依次单击“存储” 、存储帐户和“管理密钥” 。

    ![][0]

3.  记下"存储帐户名称"和"访问密钥"。

    ![][1]

4.  在你的移动服务中，单击“配置”选项卡， 向下滚动到“应用程序设置” ，输入你从存储帐户获取的下述每个项的“名称” 和 “值”对，然后单击“保存”。 

    -   `STORAGE_ACCOUNT_NAME`
    -   `STORAGE_ACCOUNT_ACCESS_KEY`

    ![][2]

    存储帐户访问密钥将会加密存储在应用程序设置中。你可在运行时从任何服务器脚本访问此密钥。有关详细信息，请参阅[应用程序设置][]。

5.  单击“数据”选项卡 ，然后单击 "TodoItem" 表。

    ![][3]

6.  在 "todoitem" 中，单击“脚本” 选项卡，然后选择“插入” ，将 insert 函数替换为以下代码，然后单击“保存” ：

        var azure = require('azure');
        var qs = require('querystring');
        var appSettings = require('mobileservice-config').appSettings;

        function insert(item, user, request) {
        // Get storage account settings from app settings. 
        var accountName = appSettings.STORAGE_ACCOUNT_NAME;
        var accountKey = appSettings.STORAGE_ACCOUNT_ACCESS_KEY;
        var host = accountName + '.blob.core.windows.net';

        if ((typeof item.containerName !== "undefined") && (
        item.containerName !== null)) {
        // Set the BLOB store container name on the item, which must be lowercase.
        item.containerName = item.containerName.toLowerCase();

        // If it does not already exist, create the container 
        // with public read access for blobs.        
        var blobService = azure.createBlobService(accountName, accountKey, host);
        blobService.createContainerIfNotExists(item.containerName, {
        publicAccessLevel:'blob'
        }, function(error) {
        if (!error) {

        // Provide write access to the container for the next 5 mins.        
        var sharedAccessPolicy = {
        AccessPolicy: {
        Permissions:azure.Constants.BlobConstants.SharedAccessPermissions.WRITE,
        Expiry:new Date(new Date().getTime() + 5 * 60 * 1000)
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

    ![][4]

    这样可将当 TodoItem 表中发生插入时所调用的函数替换为新脚本。此新脚本可为插入生成新的 SAS，有效时间为 5 分钟，然后将生成的 SAS 的值分配给所返回项目的 `sasQueryString` 属性。还将 `imageUri` 属性设置为新 BLOB 的资源路径，以便在绑定时在客户端用户界面中启用图像显示。

    > [WACOM.NOTE] 这段代码为单个 BLOB 创建 SAS。如果需要使用同一个 SAS 将多个 Blob 上载到容器，你可以使用空 Blob 资源名称来调用 [generateSharedAccessSignature 方法][]，如下所示：
    >
    >     blobService.generateSharedAccessSignature(containerName, '', sharedAccessPolicy);

接下来，你将更新快速启动应用程序，通过使用在发生插入时生成的 SAS，添加图像上载功能。

  [如何创建存储帐户]: /en-us/manage/services/storage/how-to-create-a-storage-account
  [0]: ./media/mobile-services-configure-blob-storage/mobile-blob-storage-account.png
  [1]: ./media/mobile-services-configure-blob-storage/mobile-blob-storage-account-keys.png
  [2]: ./media/mobile-services-configure-blob-storage/mobile-blob-storage-app-settings.png
  [应用程序设置]: http://msdn.microsoft.com/en-us/library/windowsazure/b6bb7d2d-35ae-47eb-a03f-6ee393e170f7
  [3]: ./media/mobile-services-configure-blob-storage/mobile-portal-data-tables.png
  [4]: ./media/mobile-services-configure-blob-storage/mobile-insert-script-blob.png
  [generateSharedAccessSignature 方法]: http://go.microsoft.com/fwlink/?LinkId=390455
