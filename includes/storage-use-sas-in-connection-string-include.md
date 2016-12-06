如果拥有的共享访问签名 (SAS) URL 能够授予对存储帐户中资源的访问权限，则可以在连接字符串中使用 SAS。由于 SAS 在 URI 上包含验证请求所需的信息，因此 SAS URI 将提供协议、服务终结点以及访问资源所需的凭据。

若要创建包含共享访问签名的连接字符串，请按以下格式指定该字符串：

    BlobEndpoint=myBlobEndpoint;
	QueueEndpoint=myQueueEndpoint;
	TableEndpoint=myTableEndpoint;
	FileEndpoint=myFileEndpoint;
	SharedAccessSignature=sasToken

尽管连接字符串必须至少包含一个服务终结点，但每个服务终结点都是可选的。

>[AZURE.NOTE] 建议最好配合使用 HTTPS 与 SAS。
>
>如果在配置文件的连接字符串中指定 SAS，可能需要为 URL 中的特殊字符编码。

### 服务 SAS 示例

下面是包含 Blob 存储服务 SAS 的连接字符串示例：

	BlobEndpoint=https://storagesample.blob.core.chinacloudapi.cn;SharedAccessSignature=sv=2015-04-05&sr=b&si=tutorial-policy-635959936145100803&sig=9aCzs76n0E7y5BpEi2GvsSv433BZa22leDOZXX%2BXXIU%3D

下面是具有特殊字符编码的同一个连接字符串的示例：

	BlobEndpoint=https://storagesample.blob.core.chinacloudapi.cn;SharedAccessSignature=sv=2015-04-05&amp;sr=b&amp;si=tutorial-policy-635959936145100803&amp;sig=9aCzs76n0E7y5BpEi2GvsSv433BZa22leDOZXX%2BXXIU%3D

### 帐户 SAS 示例

下面是包含 Blob 和文件存储帐户 SAS 的连接字符串示例。请注意，其中指定了两个服务的终结点：

	BlobEndpoint=https://storagesample.blob.core.chinacloudapi.cn;
	FileEndpoint=https://storagesample.file.core.chinacloudapi.cn;
	SharedAccessSignature=sv=2015-07-08&sig=iCvQmdZngZNW%2F4vw43j6%2BVz6fndHF5LI639QJba4r8o%3D&spr=https&st=2016-04-12T03%3A24%3A31Z&se=2016-04-13T03%3A29%3A31Z&srt=s&ss=bf&sp=rwl

下面是具有 URL 编码的同一个连接字符串的示例：

	BlobEndpoint=https://storagesample.blob.core.chinacloudapi.cn;
	FileEndpoint=https://storagesample.file.core.chinacloudapi.cn;
	SharedAccessSignature=sv=2015-07-08&amp;sig=iCvQmdZngZNW%2F4vw43j6%2BVz6fndHF5LI639QJba4r8o%3D&amp;spr=https&amp;st=2016-04-12T03%3A24%3A31Z&amp;se=2016-04-13T03%3A29%3A31Z&amp;srt=s&amp;ss=bf&amp;sp=rwl

<!---HONumber=Mooncake_1031_2016-->