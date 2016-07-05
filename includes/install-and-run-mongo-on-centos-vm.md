按照以下步骤操作可在运行 CentOS Linux 的虚拟机上安装和运行 MongoDB。

> [AZURE.WARNING]默认情况下，不启用 MongoDB 安全功能，例如身份验证和 IP 地址绑定。在将 MongoDB 部署到生产环境之前，应启用安全功能。有关详细信息，请参阅[安全性和身份验证](http://www.mongodb.org/display/DOCS/Security+and+Authentication)。

1. 配置程序包管理系统 (YUM) 以便能够安装 MongoDB。创建 */etc/yum.repos.d/10gen.repo* 文件以保存有关你的存储库的信息并添加以下内容：

		[10gen]
		name=10gen Repository
		baseurl=http://downloads-distro.mongodb.org/repo/redhat/os/x86_64
		gpgcheck=0
		enabled=1

2. 保存 repo 文件，然后运行以下命令以更新本地程序包数据库：

		$ sudo yum update

3. 若要安装程序包，请运行以下命令以安装最新的 MongoDB 稳定版本及相关工具：

		$ sudo yum install mongo-10gen mongo-10gen-server

	下载和安装 MongoDB 时，请等待。

4. 创建数据目录。默认情况下，MongoDB 将数据存储在 */data/db* 目录中，但你必须创建该目录。若要创建它，请运行：

		$ sudo mkdir -p /srv/datadrive/data
		$ sudo chown `id -u` /srv/datadrive/data

	有关在 Linux 上安装 MongoDB 的详细信息，请参阅[快速启动 Unix][QuickstartUnix]。

5. 若要启动数据库，请运行：

		$ mongod --dbpath /srv/datadrive/data --logpath /srv/datadrive/data/mongod.log

	当 MongoDB 服务器启动和预分配日志文件时，所有日志消息都将定向到 */srv/datadrive/data/mongod.log* 文件。MongoDB 可能需要几分钟来预分配日志文件和开始侦听连接。

6. 若要启动 MongoDB 命令行管理程序，请打开一个单独的 SSH 或 PuTTY 窗口并运行：

		$ mongo
		> db.foo.save ( { a:1 } )
		> db.foo.find()
		{ _id : ..., a : 1 }
		> show dbs  
		...
		> show collections  
		...  
		> help  

	通过 insert 创建数据库。

7. 在安装 MongoDB 后，您必须配置终结点才能远程访问 MongoDB。在“经典管理门户”中，依次单击“虚拟机”、你的新虚拟机的名称和“终结点”。
	
	![终结点][Image7]

8. 单击页面底部的“添加终结点”。
	
	![终结点][Image8]

9. 添加一个具有下列设置的终结点：

 - **名称**：Mongo
 - **协议**：TCP
 - **公用端口**：27017
 - **专用端口**：27017
 
 这将允许对 MongoDB 进行远程访问。



[QuickStartUnix]: http://www.mongodb.org/display/DOCS/Quickstart+Unix


[Image7]: ./media/install-and-run-mongo-on-centos-vm/LinuxVmAddEndpoint.png
[Image8]: ./media/install-and-run-mongo-on-centos-vm/LinuxVmAddEndpoint2.png

<!---HONumber=Mooncake_1207_2015-->