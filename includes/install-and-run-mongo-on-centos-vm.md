按照以下步骤操作可在运行 CentOS Linux 的虚拟机上安装和运行 MongoDB。

<div class="dev-callout">
<b>警告</b>
<p>默认情况下不启用 MongoDB 安全功能，例如身份验证和 IP 地址绑定。在将 MongoDB 部署到生产环境之前，应启用安全功能。有关更多信息，请参见<a href="http://www.mongodb.org/display/DOCS/Security+and+Authentication">安全性和身份验证（可能为英文页面）</a>。</p>
</div>

1. 配置程序包管理系统 (YUM) 以便能够安装 MongoDB。创建 */etc/yum.repos.d/10gen.repo* 文件以保存有关您的存储库的信息并添加以下内容：

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

4. 创建数据目录。默认情况下，MongoDB 将数据存储在 */data/db* 目录中，但您必须创建该目录。若要创建它，请运行：

		$ sudo mkdir -p /mnt/datadrive/data
		$ sudo chown `id -u` /mnt/datadrive/data

	有关在 Linux 上安装 MongoDB 的更多信息，请参见[快速启动 Unix][QuickstartUnix]。

5. 若要启动数据库，请运行：

		$ mongod --dbpath /mnt/datadrive/data --logpath /mnt/datadrive/data/mongod.log

	当 mongod.exe 服务器启动和预分配日志文件时，所有日志消息都将定向到 */mnt/datadrive/data/mongod.log* 文件。MongoDB 可能需要几分钟来预分配日志文件和开始侦听连接。

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

7. 在安装 MongoDB 后，您必须配置终结点才能远程访问 MongoDB。在“管理门户”中，依次单击“虚拟机”、您的新虚拟机的名称和“终结点”。
	
	![终结点][Image7]

8. 单击页面底部的“添加终结点”。
	
	![终结点][Image8]

9. 添加名为“Mongo”的终结点、协议 TCP，并将“公用”和“专用”端口均设置为“27017”。这将允许对 MongoDB 进行远程访问。
	
	![终结点][Image9]


[QuickStartUnix]: http://www.mongodb.org/display/DOCS/Quickstart+Unix


[Image7]: ./media/install-and-run-mongo-on-centos-vm/LinuxVmAddEndpoint.png
[Image8]: ./media/install-and-run-mongo-on-centos-vm/LinuxVmAddEndpoint2.png
[Image9]: ./media/install-and-run-mongo-on-centos-vm/LinuxVmAddEndpoint3.png

