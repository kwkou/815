## 预配 Azure 存储帐户
由于模拟设备将在 Azure 存储 Blob 中上载文件，因此你必须有一个 Azure 存储帐户。你可以使用现有的存储帐户，或是根据[关于 Azure 存储空间]中的说明创建新的帐户。请记下存储帐户连接字符串。

## 将 Azure Blob URI 发送到模拟设备

在本部分中，你将修改在 [使用 IoT 中心发送云到设备的消息] 中创建的 SendCloudtoDevice 控制台应用程序，以包含具有共享访问签名的 Azure Blob URI。这样，云后端只会向云到设备消息的接收方授予 Blob 写访问权限。

1. 在 Visual Studio 中，右键单击“SendCloudtoDevice”解决方案，然后单击“管理 NuGet 包...”。 

    此时将显示“管理 NuGet 包”窗口。

2. 搜索 `WindowsAzure.Storage`、单击“安装”，并接受使用条款。

    这将下载、安装 [Microsoft Azure 存储空间 SDK](https://www.nuget.org/packages/WindowsAzure.Storage/) 并添加对它的引用。

3. 在 Program.cs 文件的顶部添加以下语句：

        using Microsoft.WindowsAzure.Storage;
        using Microsoft.WindowsAzure.Storage.Blob;

4. 在 Program 类中添加以下类字段，并将连接字符串替换为你的存储帐户。

        static string storageConnectionString = "{storage connection string}";

    然后添加以下方法（可以替换任何 Blob 容器名称，本教程使用 iothubfileuploadtutorial）：
   
        private static async Task<string> GenerateBlobUriAsync()
        {
            var storageAccount = CloudStorageAccount.Parse(storageConnectionString);
            var blobClient = storageAccount.CreateCloudBlobClient();
            var blobContainer = blobClient.GetContainerReference("iothubfileuploadtutorial");
            await blobContainer.CreateIfNotExistsAsync();

            var blobName = String.Format("deviceUpload_{0}", Guid.NewGuid().ToString());
            CloudBlockBlob blob = blobContainer.GetBlockBlobReference(blobName);

            SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy();
            sasConstraints.SharedAccessStartTime = DateTime.UtcNow.AddMinutes(-5);
            sasConstraints.SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24);
            sasConstraints.Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.Write;
            string sasBlobToken = blob.GetSharedAccessSignature(sasConstraints);

            return blob.Uri + sasBlobToken;
        }

    此方法将创建新的 Blob 引用，并生成[创建和使用包含 Blob 存储的 SAS](/documentation/articles/storage-dotnet-shared-access-signature-part-2/) 中所述的共享访问签名 URI。请注意，上述方法将生成一个有效期为 24 小时的签名 URI。如果目标设备需要更多时间来上载文件（例如，它不经常连接，或者上载大文件时的连接不稳定），则你可以考虑对签名使用更长的过期时间。

5. 按如下方式修改 SendCloudToDeviceMessageAsync：

        private async static Task SendCloudToDeviceMessageAsync()
        {
            var commandMessage = new Message();
            commandMessage.Properties["command"] = "FileUpload";
            commandMessage.Properties["fileUri"] = await GenerateBlobUriAsync();
            commandMessage.Ack = DeliveryAcknowledgement.Full;

            await serviceClient.SendAsync("myFirstDevice", commandMessage);
        }

    此方法将发送包含两个应用程序属性的云到设备消息：一个属性将此消息标识为上载文件的命令，另一个属性包含 Blob URI。它还将请求完整的送达确认。请注意，可以在消息正文序列化两个应用程序属性中的信息，但序列化和反序列化信息会产生额外的处理开销。

<!-- Links -->

[关于 Azure 存储空间]: /documentation/articles/storage-create-storage-account/#create-a-storage-account

[IoT Hub Developer Guide - C2D]: /documentation/articles/iot-hub-devguide/#c2d
[Azure IoT - Service SDK NuGet package]: https://www.nuget.org/packages/Microsoft.Azure.Devices/
[Transient Fault Handling]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx
[Get started with IoT Hub]: /documentation/articles/iot-hub-csharp-csharp-getstarted/


[使用 IoT 中心发送云到设备的消息]: /documentation/articles/iot-hub-csharp-csharp-c2d/
<!-- Images -->

<!---HONumber=Mooncake_0321_2016-->