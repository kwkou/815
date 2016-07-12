<properties 
	pageTitle="在 Linux 上使用 Django 创建 Python Web 应用 | Azure" 
	description="了解如何在 Azure 中使用 Linux 虚拟机托管基于 Django 的 Web 应用程序。" 
	services="virtual-machines-linux" 
	documentationCenter="python" 
	authors="huguesv" 
	manager="wpickett" 
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="11/17/2015"
	wacn.date="06/13/2016"/>
	
# Linux VM 上的 Django Hello World Web 应用程序

> [AZURE.SELECTOR]
- [Windows](/documentation/articles/virtual-machines-windows-classic-python-django-web-app/)
- [Mac/Linux](/documentation/articles/virtual-machines-linux-python-django-web-app/)

<br>

本教程介绍如何在 Azure 中使用 Linux 虚拟机托管基于 Django 的网站。本教程假定您之前未使用过 Azure。完成本指南之后，你将能够在云中启动和运行基于 Django 的应用程序。

你将了解如何执行以下操作：

* 设置 Azure 虚拟机以托管 Django。虽然本教程介绍如何在 **Linux** 下实现此目的，但也可以使用托管在 Azure 中的 Windows Server VM 实现相同目的。 
* 从 Linux 创建新的 Django 应用程序。

通过按照本教程中的说明进行操作，您将构建一个简单的 Hello World Web 应用程序。该应用程序将托管在 Azure 虚拟机中。

以下是已完成应用程序的屏幕快照：

![显示 Azure 上的 hello world 页面的浏览器窗口](./media/virtual-machines-linux-python-django-web-app/mac-linux-django-helloworld-browser.png)

[AZURE.INCLUDE [create-account-and-vms-note](../includes/create-account-and-vms-note.md)]

## 创建并配置 Azure 虚拟机以托管 Django

1. 按照[此处](/documentation/articles/virtual-machines-linux-quick-create-portal/)提供的说明可创建 Ubuntu Server 14.04 LTS 分发的 Azure 虚拟机。如果你喜欢，你可以选择密码登陆，而不是 SSH 公钥。

1. 按照[这里](/documentation/articles/virtual-networks-create-nsg-arm-ps/)的说明，编辑网络安全组，允许 http 访问端口 80

1. 默认情况下，你的心虚拟机没有完全限定域名（FQDN）。你可以按照[这里](/documentation/articles/virtual-machines-linux-portal-create-fqdn/)的说明，创建一个。这个步骤对于本教程而言只是可选的。

## <a id="setup"> </a>设置开发环境

**注意：**如果你需要安装 Python 或希望使用客户端库，请参阅 [Python 安装指南](/documentation/articles/python-how-to-install/)。

Ubuntu Linux VM 已预安装了 Python 2.7，但它没有安装 Apache 或 Django。按照以下步骤可连接到你的 VM 并安装 Apache 和 Django。

1.  启动新的**终端**窗口。
    
1.  输入以下命令来连接到 Azure VM。如果未创建 FQDN，可使用 Azure 经典管理门户的虚拟机摘要中显示的公共 IP 地址进行连接。

		$ ssh yourusername@yourVmUrl

1.  输入以下命令来安装 Django：

		$ sudo apt-get install python-setuptools python-pip
		$ sudo pip install django

1.  输入以下带 mod-wsgi 的命令来安装 Apache：

		$ sudo apt-get install apache2 libapache2-mod-wsgi


## 创建新的 Django 应用程序

1.  打开你在上一节中使用的**终端**窗口，通过 ssh 进入你的 VM。
    
1.  输入以下命令来创建新的 Django 项目：

		$ cd /var/www
		$ sudo django-admin.py startproject helloworld

    **django-admin.py** 脚本为基于 Django 的网站生成基本结构：
    -   **helloworld/manage.py** 可帮助你启动托管和停止托管基于 Django 的网站；
    -   **helloworld/helloworld/settings.py** 包含应用程序的 Django 设置；
    -   **helloworld/helloworld/urls.py** 包含每个 url 及其视图之间的映射代码。

1.  在 **/var/www/helloworld/helloworld** 目录中创建一个名为 **views.py** 的新文件。这会包含呈现“hello world”页面的视图。启动编辑器并输入以下代码：
		
		from django.http import HttpResponse
		def home(request):
    		html = "<html><body>Hello World!</body></html>"
    		return HttpResponse(html)

1.  现在，将 **urls.py** 文件的内容替换为以下代码：

		from django.conf.urls import patterns, url
		urlpatterns = patterns('',
			url(r'^$', 'helloworld.views.home', name='home'),
		)


## 设置 Apache

1.  创建 Apache 虚拟主机配置文件 **/etc/apache2/sites-available/helloworld.conf**。将内容设置为以下项，并将 yourVmName 替换为你所使用的计算机的实际名称（例如 pyubuntu）。

		<VirtualHost *:80>
		ServerName yourVmName
		</VirtualHost>
		WSGIScriptAlias / /var/www/helloworld/helloworld/wsgi.py
		WSGIPythonPath /var/www/helloworld

1.  使用以下命令启用该站点：

        $ sudo a2ensite helloworld

1.  使用以下命令重新启动 Apache：

        $ sudo service apache2 reload

1.  最后，在你的浏览器中加载网页：

	![显示 Azure 上的 hello world 页面的浏览器窗口](./media/virtual-machines-linux-python-django-web-app/mac-linux-django-helloworld-browser.png)


## 关闭你的 Azure 虚拟机

在你完成本教程后，关闭并/或删除你新创建的 Azure 虚拟机以为其他教程释放资源并避免产生 Azure 使用费。

<!---HONumber=Mooncake_0314_2016-->