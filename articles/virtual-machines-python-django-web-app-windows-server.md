<properties linkid="develop-python-web-app-with-django" urlDisplayName="Web with Django (Windows)" pageTitle="Python web app with Django - Azure tutorial" metaKeywords="Azure Django web app, Azure Django virtual machine" description="A tutorial that teaches you how to host a Django-based website on Azure using a Windows Server 2008 R2 virtual machine." metaCanonical="" services="virtual-machines" documentationCenter="Python" title="Django Hello World Web Application" authors="" solutions="" manager="" editor="" />

# Django Hello World Web 应用程序

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/develop/python/tutorials/web-app-with-django/" title="Windows" class="current">Windows</a><a href="/zh-cn/develop/python/tutorials/django-hello-world-(maclinux)/" title="MacLinux">Mac/Linux</a></div>

本教程介绍如何在 Windows Azure 中使用 Windows Server 虚拟机托管基于 Django 的网站。本教程假定你之前未使用过 Azure。完成本指南之后，你将能够在云中启动和运行基于 Django 的应用程序。

你将了解如何执行以下操作：

-   设置 Azure 虚拟机以托管 Django。虽然本教程介绍如何在 **Windows Server** 下实现此目的，但也可以使用托管在 Azure 中的 Linux VM 实现相同目的。
-   从 Windows 创建新的 Django 应用程序。

遵循本教程中的说明，你可以生成一个简单的 Hello World Web应用程序。该应用程序将托管在 Azure 虚拟机中。

以下是已完成应用程序的屏幕快照：

![显示 Azure 上的 hello world 页面的浏览器窗口][显示 Azure 上的 hello world 页面的浏览器窗口]

[WACOM.INCLUDE [create-account-and-vms-note](../includes/create-account-and-vms-note.md)]

## 创建并配置 Azure 虚拟机以托管 Django

1.  按照[此处][此处]提供的说明可创建 *Windows Server 2012 Datacenter* 分发的 Azure 虚拟机。

1.  指示 Azure 将来自 Web 的端口 **80** 通信定向到虚拟机上的端口 **80**：

 - 导航到 Azure 门户中你新创建的虚拟机，然后单击“终结点”选项卡。
 - 单击屏幕底部的“添加终结点”按钮。
    ![添加终结点][添加终结点]

 - 打开 *TCP* 协议的*公用端口 80* 作为*专用端口 80*。
    ![][0]

1.  使用 Windows *远程桌面* 远程登录到新创建的 Azure 虚拟机。

**重要事项：**下面的所有说明假定你已正确登录到虚拟机并在那里而非在你的本地计算机发出命令！

## <span id="setup"></span> </a>设置开发环境

若要设置你的 Python 和 Django 环境，请参阅[安装指南][安装指南]以获取更多信息。

**注意 1：**你*只* 需在 Azure 虚拟机上从 Windows WebPI 安装程序安装 **Django** 产品即可使用*本* 特定教程。

**注意 2：**若要下载 WebPI 安装程序，你可能必须配置 IE ESC 设置（“开始”/“管理工具”/“服务器管理器”，然后单击“配置 IE ESC”，设置为“关闭”）。

## 设置具有 FastCGI 的 IIS

1.  安装具有 FastCGI 支持的 IIS

        start /wait %windir%\System32\\PkgMgr.exe /iu:IIS-WebServerRole;IIS-WebServer;IIS-CommonHttpFeatures;IIS-StaticContent;IIS-DefaultDocument;IIS-DirectoryBrowsing;IIS-HttpErrors;IIS-HealthAndDiagnostics;IIS-HttpLogging;IIS-LoggingLibraries;IIS-RequestMonitor;IIS-Security;IIS-RequestFiltering;IIS-HttpCompressionStatic;IIS-WebServerManagementTools;IIS-ManagementConsole;WAS-WindowsActivationService;WAS-ProcessModel;WAS-NetFxEnvironment;WAS-ConfigurationAPI;IIS-CGI

2.  设置 Python Fast CGI 处理程序

        %windir%\system32\inetsrv\appcmd set config /section:system.webServer/fastCGI "/+[fullPath='c:\Python27\python.exe', arguments='C:\Python27\Scripts\wfastcgi.py']"

3.  为此网站注册处理程序

        %windir%\system32\inetsrv\appcmd set config /section:system.webServer/handlers "/+[name='Python_via_FastCGI',path='*',verb='*',modules='FastCgiModule',scriptProcessor='c:\Python27\python.exe|C:\Python27\Scripts\wfastcgi.py',resourceType='Unspecified']"

4.  配置处理程序以运行你的 Django 应用程序

        %windir%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='C:\Python27\python.exe', arguments='C:\Python27\Scripts\wfastcgi.py'].environmentVariables.[name='DJANGO_SETTINGS_MODULE',value='DjangoApplication.settings']" /commit:apphost

5.  配置 PYTHONPATH，以便 Python 解释程序可以找到你的 Django 应用程序

        %windir%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='C:\Python27\python.exe', arguments='C:\Python27\Scripts\wfastcgi.py'].environmentVariables.[name='PYTHONPATH',value='C:\inetpub\wwwroot\DjangoApplication']" /commit:apphost

    你应该看到以下内容：

    ![IIS config1][IIS config1]

6.  将 FastCGI 告诉给 WSGI 处理程序要使用的 WSGI 网关：

        %windir%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='C:\Python27\python.exe', arguments='C:\Python27\Scripts\wfastcgi.py'].environmentVariables.[name='WSGI_HANDLER',value='django.core.handlers.wsgi.WSGIHandler()']" /commit:apphost

7.  从 [codeplex][codeplex] 下载 wfastcgi.py 并且将其保存到 C:\\Python27\\Scripts。这是前面的命令用于注册 FastCGI 处理程序的位置。或者，你可以使用 Web 平台安装程序安装它。搜索“WFastCGI”。

## 创建新的 Django 应用程序

1.  启动 cmd.exe

2.  通过 cd 进入 C:\\inetpub\\wwwroot

3.  输入以下命令来创建新的 Django 项目：

    C:\\Python27\\python.exe -m django.bin.django-admin startproject DjangoApplication

    ![New-AzureService 命令的结果][New-AzureService 命令的结果]

    **django-admin.py** 脚本生成基于 Django 的网站的基本结构：

-   **manage.py** 帮助你开始托管和停止托管你的基于 Django 的网站
-   **DjangoApplication\\settings.py** 包含你的应用程序的 Django 设置。
-   **DjangoApplication\\urls.py** 包含每个 url 和其视图之间的映射代码。

1.  在 *C:\\inetpub\\wwwroot\\DjangoApplication* 的 *DjangoApplication* 子目录中创建一个名为 **views.py** 的新文件作为 **urls.py** 的同级。这会包含呈现“hello world”页面的视图。启动编辑器并输入以下代码：

        from django.http import HttpResponse
        def hello(request):
            html = "<html><body>Hello World!</body></html>"
            return HttpResponse(html)

2.  现在，将 **urls.py** 文件的内容替换为以下代码：

        from django.conf.urls.defaults import patterns, include, url
        from DjangoApplication.views import hello
        urlpatterns = patterns('',
            (r'^$',hello),
        )

3.  最后，在你的浏览器中加载网页。

![显示 Azure 上的 hello world 页面的浏览器窗口][显示 Azure 上的 hello world 页面的浏览器窗口]

## 关闭你的 Azure 虚拟机

在你完成本教程后，关闭并/或删除你新创建的 Azure 虚拟机以为其他教程释放资源并避免产生 Azure 使用费。

  [Windows]: /zh-cn/develop/python/tutorials/web-app-with-django/ "Windows"
  [Mac/Linux]: /zh-cn/develop/python/tutorials/django-hello-world-(maclinux)/ "MacLinux"
  [显示 Azure 上的 hello world 页面的浏览器窗口]: ./media/virtual-machines-python-django-web-app-windows-server/django-helloworld-browser-azure.png
  [create-account-and-vms-note]: ../includes/create-account-and-vms-note.md
  [此处]: /zh-cn/manage/windows/tutorials/virtual-machine-from-gallery/
  [添加终结点]: ./media/virtual-machines-python-django-web-app-windows-server/django-helloworld-addendpoint.png
  [0]: ./media/virtual-machines-python-django-web-app-windows-server/django-helloworld-port80.png
  [安装指南]: ../python-how-to-install/
  [IIS config1]: ./media/virtual-machines-python-django-web-app-windows-server/django-helloworld-iis1.png
  [codeplex]: http://go.microsoft.com/fwlink/?LinkID=316392&clcid=0x409
  [New-AzureService 命令的结果]: ./media/virtual-machines-python-django-web-app-windows-server/django-helloworld-cmd-new-azure-service.png
