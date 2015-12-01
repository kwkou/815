你可以通过使用 Visual Studio **服务器资源管理器**创建 Azure 存储队列。

![服务器资源管理器 Blob][Image1]

1. 在“视图”菜单中，单击“服务器资源管理器”。
2. 在服务器资源管理器，展开你的订阅的 **Azure** 节点，展开“存储”节点以及在 Azure 存储连接服务中指定的存储帐户的节点。
3. 选择“队列”节点并从上下文菜单中选择“创建队列”。
4. 输入容器的名称，然后选择“确定”。   

默认情况下，新容器是专用容器，因此你必须指定存储访问密钥才能从该容器下载 Blob。如果你想要使容器中的文件成为公共，请在**服务器资源管理器**中选择该容器，然后按 `F4` 以显示“属性”窗口。将“公共读取访问权限”设置为“Blob”。Internet 中的所有人都可以查看公共容器中的 Blob，但是，仅在你具有相应的访问密钥时，才能修改或删除它们。


[Image1]: ./media/vs-create-blob-container-in-server-explorer/vs-storage-create-blob-containers-in-Server-Explorer.png

<!---HONumber=79-->