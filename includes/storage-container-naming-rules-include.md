Azure 存储中的每个 Blob 必须驻留在一个容器中。该容器构成 Blob 名称的一部分。例如，在这些示例 Blob URI 中，`mycontainer` 是容器的名称：

	https://storagesample.blob.core.chinacloudapi.cn/mycontainer/blob1.txt
	https://storagesample.blob.core.chinacloudapi.cn/mycontainer/photos/myphoto.jpg
 
> [AZURE.IMPORTANT]请注意，容器的名称必须始终为小写。如果你在容器名称中包括大写字母或以其他方式违反了容器命名规则，则可能会收到 400 错误（错误请求）。有关命名容器的规则，请参阅[命名和引用容器、Blob 和元数据](https://msdn.microsoft.com/zh-cn/library/azure/dd135715.aspx)。

<!---HONumber=60-->