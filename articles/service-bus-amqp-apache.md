<properties 
   pageTitle="如何在 Linux VM 上安装 Apache Qpid Proton-C | Azure"
   description="如何使用 Azure 虚拟机创建 CentOS Linux VM 以及如何生成和安装 Apache Qpid Proton-C 库。"
   services="service-bus"
   documentationCenter="na"
   authors="sethmanheim"
   manager="timlt"
    editor="" /> 
<tags 
   ms.service="service-bus"
    ms.date="05/06/2016"
   wacn.date="06/27/2016" />

# 在 Azure Linux VM 上安装 Apache Qpid Proton-C

[AZURE.INCLUDE [service-bus-selector-amqp](../includes/service-bus-selector-amqp.md)]

本部分演示如何使用 Azure 虚拟机创建 CentOS Linux VM，以及如何下载、构建和安装 Apache Qpid Proton-C 库及 Python 和 PHP 语言绑定。完成这些步骤后，你将能够运行本指南附带的 Python 和 PHP 示例。

第一步是使用 [Azure 管理门户][]执行的。以下屏幕截图显示了如何创建名为“scott-centos”的 CentOS VM：

![Azure Linux VM 上的 Proton][0]

预配后，该门户显示以下信息：

![Azure Linux VM 上的 Proton][1]

若要登录到计算机，必须知道 SSH 的终结点端口。可以通过 [Azure 管理门户][]选择新创建的 VM 并单击“终结点”选项卡从门户中获取此值。以下屏幕截图显示此计算机的公共 SSH 端口为 57146。

![Azure Linux VM 上的 Proton][2]

现在可以使用 SSH 连接到此计算机。此示例使用 PuTTY 工具，如以下屏幕截图所示：

![Azure Linux VM 上的 Proton][3]

对于 Python 和 PHP 应用，此示例使用 Apache 的 Proton 客户端库。这些库可从 [http://qpid.apache.org/download.html](http://qpid.apache.org/download.html) 下载。分发程序包中的自述文件说明了安装依赖项和构建 Proton 所需的步骤。下面是这些步骤的摘要：

1.  编辑 yum 配置文件 (/etc/yum.conf) 并注释掉对内核标头更新的排除 (#exclude=kernel*)。这是安装 gcc 编译器所必需的。

2.  安装必备组件包：

		# required dependencies 
		yum install gcc cmake libuuid-devel
		
		# dependencies needed for ssl support
		yum install openssl-devel
		
		# dependencies needed for bindings
		yum install swig python-devel ruby-devel php-devel java-1.6.0-openjdk
		
		# dependencies needed for python docs
		yum install epydoc


1.  下载 Proton 库：

	```
	[azureuser@this-user ~]$ wget http://apache.panu.it/qpid/proton/0.9/qpid-proton-0.9.tar.gz
	--2016-04-17 14:45:03--  http://apache.panu.it/qpid/proton/0.9/qpid-proton-0.9.tar.gz
	Resolving apache.panu.it (apache.panu.it)... 81.208.22.71
	Connecting to apache.panu.it (apache.panu.it)|81.208.22.71|:80... connected.
	HTTP request sent, awaiting response... 200 OK
	Length: 868226 (848K) [application/x-gzip]
	Saving to: ‘qpid-proton-0.9.tar.gz’
	
	qpid-proton-0.9.tar.gz                               
	
	100%[====================================================================================================================>] 847.88K   102KB/s    in 8.4s    
	
	2016-04-17 14:45:12 (101 KB/s) - ‘qpid-proton-0.9.tar.gz’ saved [868226/868226]
	```

1.  从分发存档中提取 Proton 代码：


		tar xvfz qpid-proton-0.9.tar.gz


1.  使用从自述文件中获取的以下步骤生成并安装代码：


		From the directory where you found this README file:	
		
		mkdir build cd build
				
		# Set the install prefix. You may need to adjust depending on your		
		# system.		
		cmake -DCMAKE\_INSTALL\_PREFIX=/usr ..
				
		# Omit the docs target if you do not wish to build or install		
		# documentation.		
		make all docs
				
		# Note that this step will require root privileges.		
		make install


执行这些步骤后，Proton 将安装在计算机上并可供使用。

## 后续步骤

准备好了解详细信息？ 请访问以下链接：

- [服务总线 AMQP 概述]

[服务总线 AMQP 概述]: /documentation/articles/service-bus-amqp-overview/
[0]: ./media/service-bus-amqp-apache/amqp-apache-1.png
[1]: ./media/service-bus-amqp-apache/amqp-apache-2.png
[2]: ./media/service-bus-amqp-apache/amqp-apache-3.png
[3]: ./media/service-bus-amqp-apache/amqp-apache-4.png

[Azure 管理门户]: http://manage.windowsazure.cn

<!---HONumber=Mooncake_0104_2016-->