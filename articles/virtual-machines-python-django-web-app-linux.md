<properties linkid="develop-python-web-app-with-django-mac" urlDisplayName="Web with Django" pageTitle="Python web app with Django on Mac - Azure tutorial" metaKeywords="" description="A tutorial that shows how to host a Django-based web site on Azure using a Linux virtual machine." metaCanonical="" services="virtual-machines" documentationCenter="Python" title="Django Hello World Web Application (mac-linux)" authors="larryf" solutions="" manager="" editor="" />

# Django Hello World Web 应用程序 (mac-linux)

<div class="dev-center-tutorial-selector sublanding"><a href="/en-us/develop/python/tutorials/web-app-with-django/" title="Windows">Windows</a><a href="/en-us/develop/python/tutorials/django-hello-world-(maclinux)/" title="Mac/Linux" class="current">Mac/Linux</a></div>

本教程介绍如何在 Windows Azure 中使用 Linux 虚拟机托管基于 Django 的网站。本教程假定你之前未使用过 Azure。完成本指南之后，你将能够在云中启动和运行基于 Django 的应用程序。

你将了解如何执行以下操作：

-   设置 Azure 虚拟机以托管 Django。虽然本教程介绍如何在 **Linux** 下实现此目的，但也可以使用托管在 Azure 中的 Windows Server VM 实现相同目的。
-   从 Linux 创建新的 Django 应用程序。

遵循本教程中的说明，你可以生成一个简单的 Hello World Web应用程序。该应用程序将托管在 Azure 虚拟机中。

以下是已完成应用程序的屏幕快照：

![显示 Azure 上的 hello world 页面的浏览器窗口][显示 Azure 上的 hello world 页面的浏览器窗口]

[WACOM.INCLUDE [create-account-and-vms-note][create-account-and-vms-note]]

## 创建并配置 Azure 虚拟机以托管 Django

1.  按照[此处][此处]提供的说明可创建 *Ubuntu Server 12.04* 分发的 Azure 虚拟机。

	**注意：**你*只* 需创建虚拟机。停在标题为*创建虚拟机后如何登录该虚拟机* 的一节。

1.  指示 Azure 将来自 Web 的端口 **80** 通信定向到虚拟机上的端口 **80**：

    -   导航到 Azure 门户中你新创建的虚拟机，然后单击“终结点”选项卡。
    -   单击屏幕底部的“添加终结点”按钮。
        ![添加终结点][添加终结点]
    -   打开 *TCP* 协议的*公用端口 80* 作为*专用端口 80*。
        ![port80][port80]

## <span id="setup"></span> </a>设置开发环境

**注意：**如果你需要安装 Python 或希望使用客户端库，请参阅 [Python 安装指南][Python 安装指南]。

Ubuntu Linux VM 已预安装了 Python 2.7，但它没有安装 Apache 或 Django。按照以下步骤可连接到你的 VM 并安装 Apache 和 Django。

1.  启动新的 **Terminal** 窗口。

2.  输入以下命令来连接到 Azure VM。

        $ ssh yourusername@yourVmUrl

3.  输入以下命令来安装 Django：

        $ sudo apt-get install python-setuptools
        $ sudo easy_install django

4.  输入以下带 mod-wsgi 的命令来安装 Apache：

        $ sudo apt-get install apache2 libapache2-mod-wsgi

## 创建新的 Django 应用程序

1.  打开你在上一节中使用的 **Terminal** 窗口，通过 ssh 进入你的 VM。

2.  输入以下命令来创建新的 Django 项目：

    ![django-admin 命令的结果][django-admin 命令的结果]

    **django-admin.py** 脚本生成基于 Django 的网站的基本结构：

    -   **manage.py** 帮助你开始托管和停止托管你的基于 Django 的网站
    -   **helloworld\\settings.py** 包含你的应用程序的 Django 设置。
    -   **helloworld\\urls.py** 包含每个 url 及其视图之间的映射代码。

3.  在 *django\\helloworld* 的 *helloworld* 子目录中创建一个名为 **views.py** 的新文件作为 **urls.py** 的同级。这会包含呈现“hello world”页面的视图。启动编辑器并输入以下代码：

        from django.http import HttpResponse
        def hello(request):
            html = "<html><body>Hello World!</body></html>"
            return HttpResponse(html)

4.  现在，将 **urls.py** 文件的内容替换为以下代码：

        from django.conf.urls.defaults import patterns, include, url
        from helloworld.views import hello
        urlpatterns = patterns('',
            (r'^$',hello),
        )

## 部署并运行你的 Django 网站

1.  编辑 apache 配置文件 **/etc/apache2/httpd.conf** 并添加以下代码，将 *username* 替换为你在创建 VM 时所指定的用户名：

        WSGIScriptAlias / /home/*username*/django/helloworld/helloworld/wsgi.py
        WSGIPythonPath /home/*username*/django/helloworld

        <Directory /home/*username*/django/helloworld/helloworld>
        <Files wsgi.py>
        Order deny,allow
        Allow from all
        </Files>
        </Directory>

2.  使用以下命令重新启动 apache：

        $ sudo apachectl restart

3.  最后，在你的浏览器中加载网页：

    ![显示 Azure 上的 hello world 页面的浏览器窗口][显示 Azure 上的 hello world 页面的浏览器窗口]

## 关闭你的 Azure 虚拟机

在你完成本教程后，关闭并/或删除你新创建的 Azure 虚拟机以为其他教程释放资源并避免产生 Azure 使用费。

  [Windows]: /en-us/develop/python/tutorials/web-app-with-django/ "Windows"
  [Mac/Linux]: /en-us/develop/python/tutorials/django-hello-world-(maclinux)/ "Mac/Linux"
  [显示 Azure 上的 hello world 页面的浏览器窗口]: ./media/virtual-machines-python-django-web-app-linux/mac-linux-django-helloworld-browser.png
  [create-account-and-vms-note]: ../includes/create-account-and-vms-note.md
  [此处]: /en-us/manage/linux/tutorials/virtual-machine-from-gallery/
  [添加终结点]: ./media/virtual-machines-python-django-web-app-linux/mac-linux-django-helloworld-add-endpoint.png
  [port80]: ./media/virtual-machines-python-django-web-app-linux/mac-linux-django-helloworld-port80.png
  [Python 安装指南]: ../python-how-to-install/
  [django-admin 命令的结果]: ./media/virtual-machines-python-django-web-app-linux/mac-linux-django-helloworld-dir.png
