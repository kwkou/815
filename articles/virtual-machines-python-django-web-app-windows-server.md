<properties linkid="develop-python-web-app-with-django" urlDisplayName="Web with Django (Windows)" pageTitle="Python web 应用程序使用 Django-Azure 教程" metaKeywords="Azure Django web app, Azure Django virtual machine" description="本教程教您如何托管基于 Django 的网站在 Azure 上使用 Windows Server 2008 R2 虚拟机。" metaCanonical="" services="virtual-machines" documentationCenter="Python" title="Django Hello World Web Application" authors="" solutions="" manager="" editor="" />






# Django Hello World Web 应用程序

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/develop/python/tutorials/web-app-with-django/" title="Windows" class="current">Windows</a><a href="/zh-cn/develop/python/tutorials/django-hello-world-(maclinux)/" title="MacLinux">Mac/Linux</a></div>

本教程介绍如何承载在 Microsoft 的基于 Django 的网站 
Azure 中使用 Windows Server 虚拟机。本教程假定您之前未使用过 Azure。完成本指南，您将拥有基于 Django 的启动和应用程序在云中运行。

您将了解如何执行以下操作：

* 设置 Azure 虚拟机以托管 Django。虽然本教程介绍如何完成这一切**Windows Server**，还可以使用在 Azure 中托管的 Linux VM 实现相同。 
* 从 Windows 创建新的 Django 应用程序。

通过学习本教程，您将构建一个简单的 Hello World web 应用程序。该应用程序将托管在 Azure 虚拟机中。

以下是完整的应用程序的屏幕快照：

![A browser window displaying the hello world page on Azure][1]

[WACOM.INCLUDE [create-account-and-vms-note](../includes/create-account-and-vms-note.md)]

## 创建并配置 Azure 虚拟机以托管 Django

1. 请按照提供的说明[此处][门户 vm]若要创建 Azure 虚拟机的 *Windows Server 2012 R2 Datacenter*分发。

1. 指示 Azure 直接端口**80**通信从 web 传输到端口**80**在虚拟机上：
 - 导航到在 Azure 门户中您新创建的虚拟机，然后单击 *ENDPOINTS*选项卡。
 - 单击 *ADD*屏幕底部的按钮。
	![add endpoint](./media/virtual-machines-python-django-web-app-windows-server/django-helloworld-addendpoint.png)

 - Open up the *TCP* protocol's *PUBLIC PORT 80* as *PRIVATE PORT 80*.
![][port80]
1. 从 *DASHBOARD*选项卡上，单击 *CONNECT*若要使用 *Remote Desktop*远程登录到新创建的 Azure 虚拟机。  

**重要说明：**下面的所有说明假定您正确地登录到虚拟机和其中的命令，而不是本地计算机发布 ！

## <a id="setup"> </a>设置 Python 和 Django

**注意：**若要下载使用 Internet Explorer，您可能必须配置 IE ESC 设置 （开始/管理工具中的服务器管理器/本地服务器，然后单击**IE 增强的安全配置**，请将设置为 Off）。

1. 安装[Web 平台安装程序][]。
1. 安装 Python 和 WFastCGI 使用 Web 平台安装程序。这将在您的 Python 脚本文件夹中安装 wfastcgi.py。
	1. 启动 Web 平台安装程序。
	1. 在搜索栏中键入 WFastCGI。 
	1. 选择您想要使用 （2.7 或 3.4) 的 Python 的版本的 WFactCGI 条目。请注意这将作为一个依赖项的 WFastCGI 安装 Python。 
1. 如果您安装了 Python 2.7[按照这些说明来手动安装 pip](https://pip.pypa.io/en/latest/installing.html) (Python 3.4 附带了已安装 pip)。
1. 安装 Django 使用 pip。

    Python 2.7:

        c:\python27\scripts\pip install django

    Python 3.4:

        c:\python34\scripts\pip install django


## Setting up IIS with FastCGI

1. 安裝含 FastCGI 支援的 IIS。  執行的時間可能需要幾分鐘。

		start /wait %windir%\System32\\PkgMgr.exe /iu:IIS-WebServerRole;IIS-WebServer;IIS-CommonHttpFeatures;IIS-StaticContent;IIS-DefaultDocument;IIS-DirectoryBrowsing;IIS-HttpErrors;IIS-HealthAndDiagnostics;IIS-HttpLogging;IIS-LoggingLibraries;IIS-RequestMonitor;IIS-Security;IIS-RequestFiltering;IIS-HttpCompressionStatic;IIS-WebServerManagementTools;IIS-ManagementConsole;WAS-WindowsActivationService;WAS-ProcessModel;WAS-NetFxEnvironment;WAS-ConfigurationAPI;IIS-CGI


### Python 2.7

仅当使用 Python 2.7，请运行这些命令。

1. 设置 Python Fast CGI 处理程序。

		%windir%\system32\inetsrv\appcmd set config /section:system.webServer/fastCGI "/+[fullPath='c:\Python27\python.exe', arguments='C:\Python27\Scripts\wfastcgi.py']"


1. 注册用于此站点的处理程序。

		%windir%\system32\inetsrv\appcmd set config /section:system.webServer/handlers "/+[name='Python_via_FastCGI',path='*',verb='*',modules='FastCgiModule',scriptProcessor='c:\Python27\python.exe|C:\Python27\Scripts\wfastcgi.py',resourceType='Unspecified']"


1. 配置要运行 Django 应用程序的处理程序。

		%windir%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='C:\Python27\python.exe', arguments='C:\Python27\Scripts\wfastcgi.py'].environmentVariables.[name='DJANGO_SETTINGS_MODULE',value='helloworld.settings']" /commit:apphost


1. 配置 PYTHONPATH，以便通过 Python 解释程序可以找到您的 Django 应用程序。

		%windir%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='C:\Python27\python.exe', arguments='C:\Python27\Scripts\wfastcgi.py'].environmentVariables.[name='PYTHONPATH',value='C:\inetpub\wwwroot\helloworld']" /commit:apphost


1. 将 FastCGI 告诉给 WSGI 网关要使用的 WSGI 处理程序。

		%windir%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='C:\Python27\python.exe', arguments='C:\Python27\Scripts\wfastcgi.py'].environmentVariables.[name='WSGI_HANDLER',value='django.core.handlers.wsgi.WSGIHandler()']" /commit:apphost


1. 您应该会看到如下：

	![IIS config1](./media/virtual-machines-python-django-web-app-windows-server/django-helloworld-iis-27.png) 

### Python 3.4

仅当使用 Python 3.4，请运行这些命令。

1. 设置 Python Fast CGI 处理程序。

		%windir%\system32\inetsrv\appcmd set config /section:system.webServer/fastCGI "/+[fullPath='c:\Python34\python.exe', arguments='C:\Python34\Scripts\wfastcgi.py']"


1. 注册用于此站点的处理程序。

		%windir%\system32\inetsrv\appcmd set config /section:system.webServer/handlers "/+[name='Python_via_FastCGI',path='*',verb='*',modules='FastCgiModule',scriptProcessor='c:\Python34\python.exe|C:\Python34\Scripts\wfastcgi.py',resourceType='Unspecified']"


1. 配置要运行 Django 应用程序的处理程序。

		%windir%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='C:\Python34\python.exe', arguments='C:\Python34\Scripts\wfastcgi.py'].environmentVariables.[name='DJANGO_SETTINGS_MODULE',value='helloworld.settings']" /commit:apphost


1. 配置 PYTHONPATH，以便通过 Python 解释程序可以找到您的 Django 应用程序。

		%windir%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='C:\Python34\python.exe', arguments='C:\Python34\Scripts\wfastcgi.py'].environmentVariables.[name='PYTHONPATH',value='C:\inetpub\wwwroot\helloworld']" /commit:apphost


1. 将 FastCGI 告诉给 WSGI 网关要使用的 WSGI 处理程序。

		%windir%\system32\inetsrv\appcmd.exe set config -section:system.webServer/fastCgi /+"[fullPath='C:\Python34\python.exe', arguments='C:\Python34\Scripts\wfastcgi.py'].environmentVariables.[name='WSGI_HANDLER',value='django.core.handlers.wsgi.WSGIHandler()']" /commit:apphost

1. 您应该会看到如下：

	![IIS config1](./media/virtual-machines-python-django-web-app-windows-server/django-helloworld-iis-34.png) 


## 创建新的 Django 应用程序


1.  从 *C:\inetpub\wwwroot*，输入以下命令来创建新的 Django 项目：

    Python 2.7:

		C:\Python27\Scripts\django-admin.exe startproject helloworld

    Python 3.4:

		C:\Python34\Scripts\django-admin.exe startproject helloworld
    
	![The result of the New-AzureService command](./media/virtual-machines-python-django-web-app-windows-server/django-helloworld-cmd-new-azure-service.png)

1.  **Django 管理**命令生成针对基于 Django 的网站的基本结构：
    
  -   **helloworld\manage.py**可帮助您开始托管和停止托管基于 Django 的网站
  -   **helloworld\helloworld\settings.py**包含您的应用程序的 Django 设置。
  -   **helloworld\helloworld\urls.py**包含每个 url 及其视图之间的映射代码。



1.  创建一个新的文件，名为**views.py**中 *C:\inetpub\wwwroot\helloworld\helloworld*目录。这将包含呈现"hello world"页面的视图。启动您的编辑器，然后输入以下：
		
		from django.http import HttpResponse
		def home(request):
    		html = "<html><body>Hello World!</body></html>"
    		return HttpResponse(html)

1.  内容替换**urls.py**替换为以下文件：

		from django.conf.urls import patterns, url
		urlpatterns = patterns('',
			url(r'^$', 'helloworld.views.home', name='home'),
		)

1. 最后，加载您的浏览器中的 web 页。

![A browser window displaying the hello world page on Azure][1]

## 关闭你的 Azure 虚拟机

当您处理完本教程、关闭和/或删除您新创建 Azure 的虚拟机，以为其他教程释放资源并避免会导致 Azure 的使用费。

[1]: ./media/virtual-machines-python-django-web-app-windows-server/django-helloworld-browser-azure.png

[port80]: ./media/virtual-machines-python-django-web-app-windows-server/django-helloworld-port80.png

[portal-vm]: /zh-cn/documentation/articles/virtual-machines-windows-tutorial/

[Web 平台安装程序]: http://www.microsoft.com/web/downloads/platform.aspx

