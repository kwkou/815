<properties linkid="develop-python-web-app-with-django-mac" urlDisplayName="Web with Django" pageTitle="Python web 应用程序使用 Django 上 Mac-Azure 教程" metaKeywords="" description="本教程演示如何托管基于 Django 的网站在 Azure 上使用 Linux 虚拟机。" metaCanonical="" services="virtual-machines" documentationCenter="Python" title="Django Hello World Web Application (mac-linux)" authors="larryf" solutions="" manager="" editor="" />





# Django Hello World Web 应用程序 (mac-linux)

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/develop/python/tutorials/web-app-with-django/" title="Windows">Windows</a><a href="/zh-cn/develop/python/tutorials/django-hello-world-(maclinux)/" title="Mac/Linux" class="current">Mac/Linux</a></div>

本教程介绍如何在 Windows 上的基于 Django 的网站中承载 
Azure 中使用 Linux 虚拟机。本教程假定您之前未使用过 Azure。完成本指南，您将拥有基于 Django 的启动和应用程序在云中运行。

您将了解如何执行以下操作：

* 设置 Azure 虚拟机以托管 Django。虽然本教程介绍如何完成这一切**Linux**，还可以在 Azure 中托管 Windows Server vm 实现相同。 
* 从 Linux 创建新的 Django 应用程序。

通过学习本教程，您将构建一个简单的 Hello World web 应用程序。该应用程序将托管在 Azure 虚拟机中。

以下是完整的应用程序的屏幕快照：

![A browser window displaying the hello world page on Azure](./media/virtual-machines-python-django-web-app-linux/mac-linux-django-helloworld-browser.png)

[WACOM.INCLUDE [create-account-and-vms-note](../includes/create-account-and-vms-note.md)]

## 创建并配置 Azure 虚拟机以托管 Django

1. 请按照提供的说明[此处][门户 vm]若要创建 Azure 虚拟机的 *Ubuntu Server 14.04 LTS*分发。

  **注意：**您 * 仅 * 需要创建虚拟机。在标题为部分停止 *如何登录到虚拟机创建表后*。

1. 指示 Azure 直接端口**80**通信从 web 传输到端口**80**在虚拟机上：
	* 导航到在 Azure 门户中您新创建的虚拟机，然后单击 *ENDPOINTS*选项卡。
	* 单击 *ADD*屏幕底部的按钮。
	![add endpoint](./media/virtual-machines-python-django-web-app-linux/mac-linux-django-helloworld-add-endpoint.png)
	* Open up the *TCP* protocol's *PUBLIC PORT 80* as *PRIVATE PORT 80*.
	![port80](./media/virtual-machines-python-django-web-app-linux/mac-linux-django-helloworld-port80.png)

## <a id="setup"> </a>设置开发环境

**注意：**如果您需要安装 Python 或希望使用客户端库，请参阅[Python 安装指南](../python-how-to-install/).

Ubuntu Linux 虚拟机已经都附带了 Python 2.7 预安装，但它没有安装 Apache 或 django。按照以下步骤可连接到你的 VM 并安装 Apache 和 Django。

1.  启动一个新**终端**窗口。
    
1.  输入以下命令来连接到 Azure VM。

		$ ssh yourusername@yourVmUrl

1.  输入以下命令来安装 django：

		$ sudo apt-get install python-setuptools
		$ sudo easy_install django

1.  输入以下命令来安装 Apache 带 mod-wsgi 的：

		$ sudo apt-get install apache2 libapache2-mod-wsgi


## 创建新的 Django 应用程序

1.  打开**终端**您使用上一节中 ssh 进入您的 VM 的窗口。
    
1.  输入以下命令以创建新的 Django 项目：

		$ cd /var/www
		$ sudo django-admin.py startproject helloworld

    **Django-admin.py**脚本将生成基于 Django 的网站的基本结构：
    -   **helloworld/manage.py**可帮助您开始托管和停止托管基于 Django 的网站
    -   **helloworld/helloworld/settings.py**包含您的应用程序的 Django 设置。
    -   **helloworld/helloworld/urls.py**包含每个 url 及其视图之间的映射代码。

1.  创建一个新的文件，名为**views.py**中**/var/www/helloworld/helloworld**目录。这将包含呈现"hello world"页面的视图。启动您的编辑器，然后输入以下：
		
		from django.http import HttpResponse
		def home(request):
    		html = "<html><body>Hello World!</body></html>"
    		return HttpResponse(html)

1.  现在的内容替换**urls.py**替换为以下文件：

		from django.conf.urls import patterns, url
		urlpatterns = patterns('',
			url(r'^$', 'helloworld.views.home', name='home'),
		)


## 设置 Apache

1.  创建一个 Apache 虚拟主机配置文件**/etc/apache2/sites-available/helloworld.conf**。将内容设置为以下内容，并确保替换 * yourVmUrl * 替换所使用的计算机的实际 URL （例如 * pyubuntu.cloudapp.net*)。

		<VirtualHost *:80>
		ServerName yourVmUrl
		</VirtualHost>
		WSGIScriptAlias / /var/www/helloworld/helloworld/wsgi.py
		WSGIPythonPath /var/www/helloworld

1.  使得网站可以使用以下命令：

        $ sudo a2ensite helloworld

1.  使用以下命令来重新启动 Apache：

        $ sudo service apache2 reload

1.  最后，加载您的浏览器中的 web 页：

	![A browser window displaying the hello world page on Azure](./media/virtual-machines-python-django-web-app-linux/mac-linux-django-helloworld-browser.png)


## 关闭你的 Azure 虚拟机

当您处理完本教程、关闭和/或删除您新创建 Azure 的虚拟机，以为其他教程释放资源并避免会导致 Azure 的使用费。


[门户 vm]: /zh-cn/documentation/articles/virtual-machines-linux-tutorial/
