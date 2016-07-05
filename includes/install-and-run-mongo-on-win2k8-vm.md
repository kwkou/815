按照以下步骤操作可在运行 Windows Server 的虚拟机上安装和运行 MongoDB。

> [AZURE.IMPORTANT]默认情况下，不启用 MongoDB 安全功能，例如身份验证和 IP 地址绑定。在将 MongoDB 部署到生产环境之前，应启用安全功能。有关详细信息，请参阅[安全性和身份验证](http://www.mongodb.org/display/DOCS/Security+and+Authentication)。

1. 使用远程桌面连接到该虚拟机后，从虚拟机上的“开始”菜单打开 Internet Explorer。

2. 选择右上角的“工具”按钮。在“Internet 选项”中，选择“安全”选项卡，然后选择“可信站点”图标，最后单击“站点”按钮。将 \__http://*.mongodb.org_ 添加到受信任站点列表中。

3. 转到“下载 - MongoDB”。[][MongoDownloads]

4. 查找**当前稳定版本**，在 Windows 专栏中，选择最新 **64 位**版本，下载并运行 MSI 安装程序。

5. MongoDB 通常安装在 C:\\Program Files\\MongoDB 下。在桌面上搜索环境变量并将 MongoDB 二进制文件路径添加到 PATH 变量。例如，可在你的计算机上的 C:\\Program Files\\MongoDB\\Server\\3.0\\bin 中找到这些二进制文件。

6. 在上述步骤中创建的数据磁盘（例如 **F:** 盘）中创建 MongoDB 数据和日志目录。从“开始”中，选择“命令提示符”以打开命令提示符窗口。键入：

		C:\> F:
		F:> mkdir \MongoData
		F:> mkdir \MongoLogs

7. 若要运行数据库，请运行：

		F:> C:
		C:\> mongod --dbpath F:\MongoData\ --logpath F:\MongoLogs\mongolog.log

	当 mongod.exe 服务器启动和预分配日志文件时，所有日志消息都将定向到 *F:\\MongoLogs\\mongolog.log* 文件。MongoDB 可能需要几分钟来预分配日志文件和开始侦听连接。

8. 若要启动 MongoDB 命令行管理程序，请从“开始”中打开另一个窗口并键入以下内容：

		C:\> cd \my_mongo_dir\bin  
		C:\my_mongo_dir\bin> mongo  
		>db  
		test
		> db.foo.insert( { a : 1 } )  
		> db.foo.find()  
		{ _id : ..., a : 1 }  
		> show dbs  
		...  
		> show collections  
		...  
		> help  

	通过 insert 创建数据库。

9. 或者，你可以将 mongod.exe 作为一项服务来安装：

		C:\mongodb\bin>mongod --logpath F:\MongoLogs\mongolog.log --logappend --dbpath F:\MongoData\ --install

	这将创建一个名为“MongoDB”的服务，其描述为“Mongo DB”。必须使用 **--logpath** 选项指定日志文件，因为正运行的服务不会在命令窗口中显示输出。**--logpath** 选项指定重新启动服务可将输出附加到现有日志文件。**--dbpath** 选项指定数据目录的位置。有关与服务相关的更多命令行选项，请参阅[与服务相关的命令行选项][MongoWindowsSvcOptions]。

	若要启动该服务，请运行以下命令：

		C:\mongodb\bin>net start MongoDB

10. 现在，MongoDB 已安装且处于运行状态，你需要在 Windows 防火墙中打开一个端口才能远程连接到 MongoDB。从“开始”菜单中，选择“管理工具”，然后选择“高级安全 Windows 防火墙”。

11. 在左窗格中，选择“入站规则”。在右侧的“操作”窗格中，选择“新建规则...”。

	![Windows 防火墙][Image1]

	在“新建入站规则向导”中，选择“端口”，然后单击“下一步”。

	![Windows 防火墙][Image2]

	选择“TCP”，然后选择“特定本地端口”。指定端口“27017”（MongoDB 侦听的默认端口），然后单击“下一步”。

	![Windows 防火墙][Image3]

	选择“允许连接”，然后单击“下一步”。

	![Windows 防火墙][Image4]

	再次单击“下一步”。

	![Windows 防火墙][Image5]

	指定规则名称（如“MongoPort”），然后单击“完成”。

	![Windows 防火墙][Image6]

12. 如果你在创建虚拟机时未配置 MongoDB 的终结点，你可以现在完成此操作。你需要防火墙规则和终结点能够远程连接到 MongoDB。在经典管理门户中，依次单击“虚拟机”、你的新虚拟机的名称和“终结点”。

	![终结点][Image7]

13. 单击页面底部的“添加”。选择“添加独立终结点”，然后单击“下一步”。

	![终结点][Image8]

14. 添加名为“Mongo”的终结点、协议 **TCP**，并将“公用”和“专用”端口均设置为“27017”。这将允许对 MongoDB 进行远程访问。

	![终结点][Image9]

> [AZURE.NOTE]端口 27017 是 MongoDB 使用的默认端口。你可以在启动 mongod.exe 服务器时使用 _--port_ 子命令来更改端口。请确保提供防火墙中的同一个端口号以及上面说明中的"Mongo"终结点。


[MongoDownloads]: http://www.mongodb.org/downloads

[MongoWindowsSvcOptions]: http://www.mongodb.org/display/DOCS/Windows+Service


[Image1]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall1.png
[Image2]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall2.png
[Image3]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall3.png
[Image4]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall4.png
[Image5]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall5.png
[Image6]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall6.png
[Image7]: ./media/install-and-run-mongo-on-win2k8-vm/WinVmAddEndpoint.png
[Image8]: ./media/install-and-run-mongo-on-win2k8-vm/WinVmAddEndpoint2.png
[Image9]: ./media/install-and-run-mongo-on-win2k8-vm/WinVmAddEndpoint3.png

<!---HONumber=79-->