<properties linkid="develop-python-web-app-with-django-and-mysql" urlDisplayName="Web with Django + MySQL" pageTitle="Python web app with Django and MySQL - Azure tutorial" metaKeywords="Azure django web app, Azure Django MySQL, Azure django Python" description="A tutorial that teaches you how to use MySQL in with Django on an Azure virtual machine. Code samples written in Python." metaCanonical="" services="virtual-machines" documentationCenter="Python" title="Django Hello World - MySQL Windows Edition" authors="" solutions="" manager="" editor="" />

# Django Hello World - MySQL Windows 版本

本教程介绍如何在单个 Azure 虚拟机上结合使用 MySQL 与 Django。本指南假定你以前使用过 Azure 和 Django。有关 Azure 和 Django 的简介，请参阅 [Django Hello World][djangohelloworld]。本指南还假定你对 MySQL 有一些了解。有关 MySQL 的概述，请参阅 [MySQL 网站][mysqldoc]。

在本教程中，你将了解如何：

-   设置 Azure 虚拟机以托管 MySQL 和 Django。虽然本教程介绍如何在 Windows Server 2008 R2 下实现此目的，但也可以使用托管在 Azure 中的 Linux VM 实现相同目的。
-   为 Python 安装 [MySQL 驱动程序][mysqlpy]。
-   配置现有 Django 应用程序以使用 MySQL 数据库。
-   直接从 Python 使用 MySQL。
-   托管并运行你的 MySQL Django 应用程序。

通过利用托管在 Azure VM 中的 MySQL 数据库，你将扩展 [Django Hello World][djangohelloworld] 示例以查找所需的 *World* 替换项。将通过支持 MySQL 的 Django *counter* 应用程序来决定此替换项。如同 Hello World 示例一样，此 Django 应用程序仍将托管在 Azure 虚拟机中。

本教程中的项目文件将存储在 **C:\\django\\helloworld** 中，并且已完成的应用程序将类似于下图：

![][0]

[WACOM.INCLUDE [create-account-and-vms-note](../includes/create-account-and-vms-note.md)]

## 设置虚拟机以托管 MySQL 和 Django

1.  按照[此处][preview-portal-vm]提供的说明可创建 *Windows Server 2008 R2* 分发的 Azure 虚拟机。

2.  在该虚拟机上为 MySQL 事务打开 TCP 端口：

 * 导航到 Azure 门户中你新创建的虚拟机，然后单击“终结点”选项卡。
 * 单击屏幕底部的“添加终结点”按钮。
 * 添加名称为 **mysql**、协议为 **TCP**、公关端口为 **3306**、专用端口为 **3306** 的一个终结点。

3.  添加名称为 **web**、协议为 **TCP**、公关端口为 **80**、专用端口为 **80** 的另一个终结点。这会将外部 Internet 请求重定向到运行 Django 的端口，即 *80*。

4.  使用 Windows *远程桌面* 远程登录到新创建的 Azure 虚拟机。

5.  打开虚拟机上的 TCP 端口 **80**：

 * 从“开始”菜单中，选择“管理工具”，然后选择“高级安全 Windows 防火墙”。
 * 在左窗格中，选择“入站规则”。在右侧的“操作”窗格中，选择“新建规则...”。
 * 在“新建入站规则向导”中，选择“端口”，然后单击“下一步”。
 * 选择“TCP”，然后选择“特定本地端口”。指定端口“80”（Django 侦听的端口），然后单击“下一步”。
 * 选择“允许连接”，然后单击“下一步”。
 * 再次单击“下一步”。
 * 为规则指定名称（例如“DjangoPort”），然后单击“完成”。

6.  在虚拟机上安装适用于 Windows 的最新版本的 [MySQL Community Server][mysqlcommunity]：

    **注意**：建议将你的数据库安装在与 OS 不同的数据分区中。

 * 运行[此处][mysqlcommunity]的 *Windows（x86，64 位），MSI 安装程序* 链接，并从离你最近的下载镜像下载适当的 MSI 安装程序。有帮助的提示是：

    -   选择“完全”安装类型。
    -   选择配置向导中的“详细配置”。
    -   **确保你在端口号 3306 上启用了 TCP/IP 网络并为该端口添加了防火墙例外。**
    -   设置根密码并从远程计算机进行根目录访问。
 * 安装[“world”数据库][“world”数据库]示例（MyISAM 版本）：

    -   将[此][mysqlworlddl] zip 文件下载到 Azure 虚拟机上。
    -   **将它解压缩到 *C:\\Users\\Administrator\\Desktop\\world.sql*。**

1.  安装 MySQL 之后，单击 Windows“开始”菜单，然后运行刚才安装的“MySQL 5.5 Command Line Client”。发出以下命令：

        CREATE DATABASE world;
        USE world;
        SOURCE C:\Users\Administrator\Desktop\world.sql
        CREATE USER 'testazureuser'@'%' IDENTIFIED BY 'testazure';
        CREATE DATABASE djangoazure;
        GRANT ALL ON djangoazure.* TO 'testazureuser'@'%';
        GRANT ALL ON world.* TO 'testazureuser'@'%';
        SELECT name from country LIMIT 1;

  你现在应该看到与下图类似的响应：

  ![][2]

8.  在你开始开发你的 Django 应用程序之前，我们当然需要在虚拟机上安装 Python+Django。通过 [Web 平台安装程序](http://www.microsoft.com/web/downloads/platform.aspx)可以完成这一步。在安装 Web PI 后，使用它搜索“Django”并且安装 Django (Python) 产品。

  **注意：**你只需从 WebPI 安装 *Django* 产品即可使用本教程。你**无**需安装 *Python Tools for Visual Studio* 或 Azure Python SDK 即可实现所需目的。

1.  安装 MySQL Python 客户端包。你可以直接[从此链接][mysqlpydl]安装它。完成后，运行以下命令来验证你的安装：

  ![][1]

## 扩展 Django Hello World 应用程序

1.  按照 [Django Hello World][djangohelloworld] 教程中提供的说明在 Django 中创建一个简单的“Hello World”Web 应用程序。

2.  在你喜欢的文本编辑器中打开 **C:\\django\\helloworld\\helloworld\\settings.py**。修改 **DATABASES** 全局字典以读取以下内容：

        DATABASES = {
            'default': {
                'ENGINE': 'django.db.backends.mysql',
                'NAME': 'djangoazure',               
                'USER': 'testazureuser',  
                'PASSWORD': 'testazure',
                'HOST': '127.0.0.1', 
                'PORT': '3306',
                },
            'world': {
                'ENGINE': 'django.db.backends.mysql',
                'NAME': 'world',               
                'USER': 'testazureuser',  
                'PASSWORD': 'testazure',
                'HOST': '127.0.0.1', 
                'PORT': '3306',
                }
            }

  正如你看到的，我们已为 Django 提供有关在哪里查找我们的 MySQL 数据库的说明。

  **注意：**你**必须**更改 *HOST* 密钥以匹配你的 Azure (MySQL) VM 的**永久** IP 地址。此时，*HOST* 应设置为 *ipconfig* Windows 命令为它报告的内容。

  在你修改 *HOST* 以匹配 MySQL VM 的 IP 地址之后，请保存此文件并关闭它。

1.  现在，我们已引用 *djangoazure* 数据库，让我们使用它执行一些有用的操作！为此，我们将为一个简单的 *counter* 应用程序创建模型。若要指示 Django 创建该模型，请运行以下命令：

        cd C:\django\helloworld
        C:\Python27\python.exe manage.py startapp counter

  如果 Django 没有从上面最后一个命令报告任何输出，则它成功了。

1.  将以下文本附加到 **C:\\django\\helloworld\\counter\\models.py** 中：

        class Counter(models.Model):
            count = models.IntegerField()
            def __unicode__(self):
                return u'%s' % (self.count)

  我们在此处只是使用单个整数字段 *count* 定义 Django 的 *Model* 类的名为 *Counter* 的子类。此简单的 counter 模型将最终记录我们的 Django 应用程序的点击数。

1.  接下来，我们使 Django 知道 *Counter* 的存在：

 * 再次编辑 **C:\\django\\helloworld\\helloworld\\settings.py**。将“counter”添加到 *INSTALLED\_APPS* 元组中。
 * 从命令提示符处，请运行：

            cd C:\django\helloworld
            C:\Python27\python manage.py sql counter
            C:\Python27\python manage.py syncdb

    这些命令将 *Counter* 模型存储在实时 Django 数据库中，并且输出结果类似于以下内容：

        C:\django\helloworld> C:\Python27\python manage.py sql counter
        BEGIN;
        CREATE TABLE `counter_counter` (
            `id` integer AUTO_INCREMENT NOT NULL PRIMARY KEY,
            `count` integer NOT NULL
        )
        ;
        COMMIT;

        C:\django\helloworld> C:\Python27\python manage.py syncdb
        Creating tables ...
        Creating table auth_permission
        Creating table auth_group_permissions
        Creating table auth_group
        Creating table auth_user_user_permissions
        Creating table auth_user_groups
        Creating table auth_user
        Creating table django_content_type
        Creating table django_session
        Creating table django_site
        Creating table counter_counter

        You just installed Django's auth system, which means you don't have any superusers defined.
        Would you like to create one now? (yes/no): no
        Installing custom SQL ...
        Installing indexes ...
        Installed 0 object(s) from 0 fixture(s)

1.  替换 **C:\\django\\helloworld\\helloworld\\views.py** 的内容。下面新实现的 *hello* 函数将我们的 *Counter* 模型与我们以前安装的单独示例数据库 *world* 结合使用来生成“World”字符串的合适替换项：

        from django.http import HttpResponse
        import django.db
        from counter.models import Counter

        def getCountry(intId):
            #Connect to the MySQL sample database 'world'
            cur = django.db.connections['world'].cursor()
            #Execute a trivial SQL query which returns the name of 
            #all countries contained in 'world'
            cur.execute("SELECT name from country")
            tmp = cur.fetchall()
            #Clean-up after ourselves
            cur.close()
            if intId >= len(tmp):
                return "countries exhausted"
            return tmp[intId][0]

        def hello(request):
            if len(Counter.objects.all())==0:
                #when the database corresponding to 'helloworld.counter' is 
                #initially empty...
                c = Counter(count=0)
            else:
                c = Counter.objects.all()[0]
                c.count += 1
            c.save()       
            world = getCountry(int(c.count))
            return HttpResponse("<html><body>Hello <em>" + world + "</em></body></html>")

## 部署并运行你的 Django 网站

注意：下面演示如何在测试环境中运行 Django。若要在生产环境中运行它，请按照“Django Hello World 教程”中的“设置具有 FastCGI 的 IIS”一节操作。使用 Windows Firewall Client 为 Windows Server 2K8 R2 虚拟机上的 Internet 流量打开端口 80 对 FastCGI 来说不是必需的。

1.  切换回 Windows PowerShell 窗口，并键入以下命令来公开部署你的 Django 网站：

        PS C:\django\helloworld> $ipPort = 
        PS C:\django\helloworld> $ipPort = [string]$ipPort.AddressList[1]
        PS C:\django\helloworld> $ipPort += ":80"
        PS C:\django\helloworld> C:\Python27\python.exe .\manage.py runserver $ipPort

    **runserver** 参数指示 Django 在 TCP 端口 *80* 上运行我们的 *helloworld* 网站。此命令的结果应类似于：

        PS C:\django\helloworld> C:\Python27\python.exe .\manage.py runserver $ipPort
        Validating models...

        0 errors found
        Django version 1.4, using settings 'helloworld.settings'
        Development server is running at http://123.34.56.78:80
        Quit the server with CTRL-BREAK.

2.  从你的本地 Web 浏览器中，打开 **http://*yourVmName*.cloudapp.net**（其中，*yourVmName* 是你在虚拟机创建步骤中使用的任何名称）。你应看到显示了“Hello ...!" 如下面的屏幕快照所示。这表明 Django 正运行在虚拟机中并能够正常运行。

    ![][5]

  刷新 Web 浏览器几次，你应该看到消息从*“Hello **\<country abc\>**”*更改为*“Hello **\<some other country\>**”*。

1.  若要停止 Django 托管网站，只需切换到 PowerShell 窗口并按 **Ctrl-C**。

## 关闭你的 Azure 虚拟机

在你完成本教程后，关闭并/或删除你新创建的 Azure 虚拟机以为其他教程释放资源并避免产生 Azure 使用费。

[0]: ./media/virtual-machines-python-use-mysql-django/mysql_tutorial01.png
[1]: ./media/virtual-machines-python-use-mysql-django/mysql_tutorial01-1.png
[2]: ./media/virtual-machines-python-use-mysql-django/mysql_tutorial01-2.png
[5]: ./media/virtual-machines-python-use-mysql-django/mysql_tutorial01.png

[djangohelloworld]: http://windowsazure.com/zh-cn/documentation/articles/virtual-machines-python-django-web-app-windows-server

[mysqldoc]: http://dev.mysql.com/doc/
[mysqlpy]: http://pypi.python.org/pypi/MySQL-python/1.2.3

[mysqlpydl]: http://code.google.com/p/soemin/downloads/detail?name=MySQL-python-1.2.3.win32-py2.7.exe&can=2&q=
[mysqlcommunity]:http://dev.mysql.com/downloads/mysql/

[mysqlworld]:http://dev.mysql.com/doc/index-other.html
[mysqlworlddl]:http://downloads.mysql.com/docs/world.sql.zip


[preview-portal-vm]: /en-us/manage/windows/tutorials/virtual-machine-from-gallery/

[The status of the Stop-AzureService command]: ../Media/django-helloworld-ps-stop.png
[The status of the Remove-AzureService command]: ../Media/django-helloworld-ps-remove.png

[Installation Guide]: http://windowsazure.com/zh-cn/documentation/articles/python-how-to-install
 