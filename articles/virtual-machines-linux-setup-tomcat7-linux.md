<properties
	pageTitle="在 Linux VM 上设置 Apache Tomcat | Azure"
	description="了解如何使用运行 Linux 的 Azure 虚拟机 (VM) 设置 Apache Tomcat7。"
	services="virtual-machines"
	documentationCenter=""
	authors="NingKuang"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="12/15/2015"
	wacn.date="01/29/2016"/>

# 如何使用 Azure 在 Linux 虚拟机上设置 Tomcat7 

Apache Tomcat（或简称 Tomcat，以前也称为 Jakarta Tomcat）是由 Apache Software Foundation (ASF) 开发的一个开源 Web 服务器和 servlet 容器。Tomcat 实现了 Sun Microsystems 提出的 Java Servlet 和 JavaServer Pages (JSP) 规范，并提供了用于运行 Java 代码的纯 Java HTTP Web 服务器环境。在最简单的配置中，Tomcat 在单个操作系统进程中运行。此进程运行 Java 虚拟机 (JVM)。浏览器向 Tomcat 发出的每个 HTTP 请求都作为 Tomcat 进程中的单独线程进行处理。

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

在本指南中，将在 Linux 映像上安装 tomcat7，并将其部署在 Azure 中。

你将学习以下内容：

-	如何在 Azure 中创建虚拟机。
-	如何准备用于 tomcat7 的虚拟机。
-	如何安装 tomcat7。

假定读者已拥有 Azure 订阅。如果没有，你可以在 [http://azure.cn](http://azure.cn) 中注册 1rmb 试用版。

本主题假定你具有 tomcat 和 Linux 的基本专业知识。

##阶段 1：创建映像
在此阶段中，你将在 Azure 中使用 Linux 映像创建虚拟机。

###步骤 1：生成 SSH 身份验证密钥
SSH 是面向系统管理员的重要工具。但是，基于人工确定的密码配置访问安全性并不是最佳做法。恶意用户可以根据用户名和弱密码侵入你的系统。

好消息是，有办法使远程访问保持打开状态，而无需担心密码。此方法包括使用非对称加密进行身份验证。用户的私钥是授予身份验证的密钥。你甚至可以锁定用户的帐户，以完全禁止密码身份验证。

此方法的另一个优点是不需要使用不同的密码来登录到不同的服务器。你可以使用个人私钥在所有服务器上进行身份验证，从而使你不必记住多个密码。

使用此方法还可以要求提供密码进行登录。

按照下列步骤进行操作可生成 SSH 身份验证密钥。

1.	从以下位置下载并安装 puttygen：[http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)
2.	运行 PUTTYGEN.EXE。
3.	单击**“生成”**以生成密钥。在此过程中，可以通过将鼠标放在窗口中的空白区域上来增加随机性。  
![][1]
4.	在生成过程结束后，Puttygen.exe 将显示生成的密钥。例如：  
![][2]
5.	在**“密钥”**中选择并复制公钥，然后将它保存在一个名为 publicKey.pem 的文件中。不要单击**“保存公钥”**，因为保存的公钥的文件格式不同于我们所需的公钥。
6.	单击**“保存私钥”**，并将其保存到名为 privateKey.ppk 的文件中。

###步骤 2：在 Azure 经典管理门户中创建映像。
在 [Azure 经典管理门户](https://manage.windowsazure.cn)中，单击任务栏中的**“新建”**以创建映像，并根据你的需要选择 Linux 映像。以下示例使用 Ubuntu 14.04 映像。
![][3]

对于**“主机名”**，指定你和 Internet 客户端将用于访问此虚拟机的 URL 的名称。定义 DNS 名称的最后部分（例如 tomcatdemo），Azure 将生成 tomcatdemo.chinacloudapp.cn 作为 URL。

对于**“SSH 身份验证密钥”**，从 **publicKey.pem** 文件中复制密钥值，其中包含由 puttygen 生成的公钥。  
![][4]

根据需要配置其他设置，然后单击“创建”。

##阶段 2：准备用于 Tomcat7 的虚拟机
在此阶段中，将为 tomcat 通信配置终结点，然后连接到新的虚拟机。
###步骤 1：打开 HTTP 端口，以允许 Web 访问
Azure 中的终结点由协议（TCP 或 UDP）以及公用和专用端口组成。专用端口是服务侦听虚拟机的端口。公用端口是 Azure 云服务从外部侦听基于 Internet 的传入通信的端口。

TCP 端口 8080 是 tomcat 侦听的默认端口号。使用 Azure 终结点打开此端口将允许你和其他 Internet 客户端访问 tomcat 页。

1.	在 Azure 经典管理门户中，单击**“虚拟机”**，然后单击你创建的虚拟机。  
![][5]
2.	若要将终结点添加到虚拟机，请单击**“终结点”** 框。
![][6]
3.	单击**“添加”**。  
	1.	对于**终结点**，在“终结点”中键入终结点的名称，然后在**“公用端口”**中键入 80。  

		如果将其设置为 80，则无需在 URL 中包括端口号即可允许你访问 tomcat。例如，http://tomcatdemo.chinacloudapp.cn。

		如果将其设置为其他值（例如 81），则需要将端口号添加到 URL 才能访问 tomcat。例如，http://tomcatdemo.chinacloudapp.cn:81/。
	2.	在“专用端口”中键入 8080。默认情况下，tomcat 侦听 TCP 端口 8080。如果你更改了 tomcat 的默认侦听端口，则应将专用端口更新为与 tomcat 侦听端口相同。  
	![][7]

4.	单击**“确定”**将该终结点添加到你的虚拟机。



###步骤 2：连接到你创建的映像
你可以选择用于连接到虚拟机的任何 SSH 工具。在此示例中，我们使用 Putty。

首先，从 Azure 经典管理门户获取你的虚拟机的 DNS 名称。依次单击**“虚拟机”**->你的虚拟机的名称->**“仪表板”**。

从 **SSH Details** 字段获取 SSH 连接的端口号。下面是一个示例。  
![][8]

从[此处](http://www.putty.org/)下载 Putty。

下载后，单击可执行文件 PUTTY.EXE。使用从虚拟机的属性获取的主机名和端口号配置基本选项。下面是一个示例：  
![][9]

在左窗格中，单击**“连接”**-> **SSH** ->**“身份验证”**，然后单击**“浏览”**以指定 **privateKey.ppk** 文件（其中包含在“第 1 阶段：创建映像”中 puttygen 生成的私钥）的位置。下面是一个示例：  
![][10]

单击**“打开”**。此时可能会通过一个消息框提醒你。如果你已正确配置 DNS 名称和端口号，请单击**“是”**。
![][11]  


你应该看到以下内容：  
![][12]

输入在“第 1 阶段：创建映像”中创建虚拟机时指定的用户名。你会看到如下内容：  
![][13]





##第 3 阶段：安装软件
在此阶段中，你将安装 Java 运行时环境、tomcat 和其他 tomcat 组件。

###Java 运行时环境
Tomcat 是用 Java 编写的。有两种类型的 Java 开发工具包 (Jdk)（OpenJDK 和 Oracle JDK），你可以选择所需的那个。

>AZURE.NOTE：这两个 JDK 对于 Java API 中的类，几乎包含相同的代码，但虚拟机的代码实际上不同。涉及到库时，OpenJDK 倾向于使用开放库，而 Oracle 倾向于使用已关闭的库。但 Oracle JDK 包含更多类和一些修复的 bug，并且 Oracle JDK 比 OpenJDK 更稳定。

以下命令下载不同 JDK。

open-jdk

	sudo apt-get update  
	sudo apt-get install openjdk-7-jre  

oracle-jdk

-	若要从 Oracle 网站下载 JDK，请执行以下命令：  

		wget --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u5-b13/jdk-8u5-linux-x64.tar.gz  

-	若要创建一个包含 JDK 文件的目录，请执行以下命令：

		sudo mkdir /usr/lib/jvm  

-	若要将 JDK 文件解压到 /usr/lib/jvm/ 目录，请执行以下命令：

		sudo tar -zxf jdk-8u5-linux-x64.tar.gz  -C /usr/lib/jvm/  

-	若要将 Oracle JDK 设置为默认 JVM，请执行以下命令：

		sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/jdk1.8.0_05/bin/java 100  
		sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/jdk1.8.0_05/bin/javac 100  

####测试：
可以使用如下命令测试是否已正确安装 Java 运行时环境：

	java -version  

如果已安装 open-jdk，你应看到如下消息：
![][14]

如果已安装 oracle-jdk，你应看到如下消息：
![][15]

###Tomcat7
使用以下命令安装 tomcat7：

	sudo apt-get install tomcat7  

如果你未使用 tomcat7，请使用此命令的相应变体。

####测试：

若要检查 tomcat7 是否已成功安装，请浏览到 tomcat 服务器的 DNS 名称（http://tomcatexample.chinacloudapp.cn/ 是本文中的示例 URL）。如果你看到如下页面，则 tomcat7 已正确安装。
![][16]


###安装其他 Tomcat 组件
你还可以安装其他可选 tomcat 组件。

使用 **sudo apt-cache search tomcat7** 命令可查看所有可用的组件。以下命令是安装某些有用部件的示例。

	sudo apt-get install tomcat7-admin      #admin web applications
	sudo apt-get install tomcat7-user         #tools to create user instances  

##第 4 阶段：配置 Tomcat
在此阶段中，你可以管理 tomcat。
###启动和停止 tomcat7
当你安装 tomcat7 服务器时，该服务器会自动启动。你也可以使用以下命令自己启动它：

	sudo /etc/init.d/tomcat7 start

若要停止 tomcat7，请执行以下命令：

	sudo /etc/init.d/tomcat7 stop

若要查看 tomcat7 的状态，请执行以下命令：

	sudo /etc/init.d/tomcat7 status

若要重启 tomcat 服务，请执行以下命令：

	sudo /etc/init.d/tomcat7 restart

###Tomcat 管理
可以编辑 Tomcat 用户配置文件，以使用以下命令设置管理员凭据：

	sudo vi  /etc/tomcat7/tomcat-users.xml   

下面是一个示例：
![][17]  

>AZURE.NOTE：为管理员用户名创建强密码。

编辑此文件之后，你应使用以下命令重启 tomcat7 服务，以确保所做的更改生效：

	sudo /etc/init.d/tomcat7 restart  

打开浏览器，并输入 URL **http://<your tomcat server DNS name>/manager/html**。对于本文中的示例，URL 为 http://tomcatexample.chinacloudapp.cn/manager/html。

连接后，你应该会看到如下内容：  
![][18]

##常见问题

###无法通过 Internet 使用 Tomcat 和 Moodle 访问虚拟机

-	**症状**  
Tomcat 正在运行，但你使用浏览器看不到 Tomcat 默认页。
-	**可能的根本原因**   
	1.	tomcat 侦听端口与用于 tomcat 通信的虚拟机终结点的专用端口不同。  

		检查你的公用端口和专用端口终结点设置，并确保专用端口与 tomcat 侦听端口相同。有关如何为你的虚拟机配置终结点的说明，请参阅“第 1 阶段：创建映像”。

		若要确定 tomcat 侦听端口，请打开 /etc/httpd/conf/httpd.conf（Red Hat 发行版）或 /etc/tomcat7/server.xml（Debian 发行版）。默认情况下，tomcat 侦听端口为 8080。下面是一个示例：

			<Connector port="8080" protocol="HTTP/1.1"  connectionTimeout="20000"  URIEncoding="UTF-8"            redirectPort="8443" />  

		如果要使用 Debian 或 Ubuntu 等虚拟机并且要更改 Tomcat 侦听的默认端口（例如 8081），则还应为操作系统打开该端口。首先，打开配置文件：

			sudo vi /etc/default/tomcat7  

		然后，取消注释最后一行并将“no”更改为“yes”。

			AUTHBIND=yes

	2.	防火墙已禁用 tomcat 侦听端口。

		如果你只能在本地主机上看到 Tomcat 默认页，则问题很可能出在 Tomcat 所侦听的端口被防火墙阻止。可以使用 w3m 工具来浏览网页。以下命令安装 w3m 并浏览到 Tomcat 默认页：

			sudo yum  install w3m w3m-img
			w3m http://localhost:8080  

-	**解决方案**
	1. 如果 tomcat 侦听端口与发往虚拟机的通信的终结点专用端口不同，则需要将该专用端口更改为与 tomcat 侦听端口相同。   

	2.	如果此问题是由防火墙/iptables 导致的，请将以下行添加到 /etc/sysconfig/iptables：

			-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
			-A INPUT -p tcp -m tcp --dport 443 -j ACCEPT  

		请注意，只有 https 通信才需要第二行。

		请确保该行在全局限制访问权限的所有行上方，如下所示：

			-A INPUT -j REJECT --reject-with icmp-host-prohibited  

		若要重新加载 iptables，请运行以下命令：

			service iptables restart  

		这已在 CentOS 6.3 上进行了测试。

###将项目文件上载到 /var/lib/tomcat7/webapps/ 时，权限被拒绝  

-	**症状**  
当你使用任何 SFTP 客户端（例如 FileZilla）连接到虚拟机并导航到 /var/lib/tomcat7/webapps/ 来发布站点时，你收到如下错误消息：  

		status:	Listing directory /var/lib/tomcat7/webapps
		Command:	put "C:\Users\liang\Desktop\info.jsp" "info.jsp"
		Error:	/var/lib/tomcat7/webapps/info.jsp: open for write: permission denied
		Error:	File transfer failed

-	**可能的根本原因**
你无权访问 /var/lib/tomcat7/webapps 文件夹。
-	**解决方案**  
你需要获得根帐户权限。你可以将该文件夹的所有权从 root 更改为在设置计算机时使用的用户名。下面是使用 azureuser 帐户名称的示例：  

		sudo chown azureuser -R /var/lib/tomcat7/webapps

	也使用 -R 选项对目录内的所有文件应用权限。

	请注意，此命令也适用于目录。-R 选项可更改目录内的所有文件和目录的权限。下面是一个示例：

		sudo chown -R username:group directory  

	此命令将更改目录内的所有文件和目录以及目录本身的所有权（用户和组）。

	以下命令只会更改文件夹目录的权限，但不会更改目录内的文件和文件夹的权限。

		sudo chown username:group directory



[1]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-01.png
[2]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-02.png
[3]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-03.png
[4]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-04.png
[5]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-05.png
[6]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-06.png
[7]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-07.png
[8]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-08.png
[9]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-09.png
[10]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-10.png
[11]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-11.png
[12]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-12.png
[13]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-13.png
[14]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-14.png
[15]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-15.png
[16]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-16.png
[17]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-17.png
[18]: ./media/virtual-machines-linux-classic-setup-tomcat/virtual-machines-linux-setup-tomcat7-linux-18.png

<!---HONumber=Mooncake_0118_2016-->