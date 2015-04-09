<properties linkid="develop-python-web-site-with-django" urlDisplayName="Web Sites with Django" pageTitle="Python Web Sites with Django - Azure tutorial" metaKeywords="Azure django, django website" description="A tutorial that introduces you to running a Python web site on Azure." metaCanonical="" services="web-sites" documentationCenter="Python" title="Creating Web Sites with Django" authors="" solutions="" manager="" editor="" />

# 使用 Django 创建网站

在本教程中，我们将介绍如何开始在 Azure 网站上运行 Python。Azure 网站提供受限制的免费托管和快速部署功能，现在，你可以使用 Python！随着你的应用程序的增长，你可以切换到付费托管，并且还可以与所有其他 Azure 服务集成。

本教程将演示如何部署使用 Django Web 框架构建的应用程序。本教程将指导你完成部署你的应用程序和任何所需库（包括 Django）的步骤。你会将所有这些内容放入 Git 存储库中，从而可以快速简单地将更新推送到你的网站。最后，你将通过 Azure 配置新创建的网站，以便它能够运行你的 Python 应用程序。

[WACOM.INCLUDE [create-account-and-websites-note][create-account-and-websites-note]]

本教程使用 Python 2.7 和 Django 1.4。你可以自行获取这些软件，也可以通过使用 [][]<http://azure.microsoft.com/zh-cn/develop/python/></a> 上的 Windows Installer 链接来快速轻松地安装这些软件。

**注意**：Azure 网站现在预安装了 Python 2.7 和 wfastcgi 处理程序。不过，未包括 Web 框架，例如 Django。如果愿意，你仍可以使用其他 Python 解释程序。你只需将它包括在 Git 存储库中，并将网站配置为使用该解释程序而非已安装的 Python 2.7 解释程序。

你还将需要安装一个部署选项，以便将网站推送到 Azure。还可以使用各种不同的部署工具，但本教程使用 Git。我们推荐 [msysgit][msysgit]。

**注意**：Python 项目目前不支持 TFS 发布。

在安装 Python、Django 和 Git 之后，你将拥有开始操作所需的一切内容。

## 在门户中创建网站

创建你的应用程序的第一步是通过 Azure 管理门户创建网站。为此，你将需要登录到该门户，然后单击左下角的“新建”按钮。将出现一个窗口。单击“快速创建”，输入 URL，然后选择“创建网站”。

![][]

将快速设置网站。接下来，你要为通过 Git 进行发布提供相应支持。这一点可通过选择“从源代码管理设置部署”来完成。

![][1]

从“设置部署”对话框中，向下滚动并选择“本地 Git”选项。单击向右箭头以继续。

![][2]

在设置 Git 发布之后，你将立即看到通知你正在创建存储库的页面。在存储库就绪时，会将你转至“部署”选项卡。“部署”选项卡包括有关如何连接的说明。

![][3]

## 网站开发

现在，你已在 Azure 中创建了 Git 存储库，将开始从本地计算机使用网站填充它。第一步是使用提供的 url 克隆现有空网站：

![][4]

从这里，你已准备好使用网站设置登记。你需要做一些事情：

1.  包括 Django 库以及你将用于运行网站的其他库。
2.  包括 Django 应用程序代码。

首先，要包括 Django 库。为此，要通过使用以下命令来创建一个名为 site-packages 的新目录，并将你安装的 Django 版本复制到那里：

    mkdir site-packages
    cd site-packages
    xcopy /s C:\Python27\lib\site-packages\* .

这些命令会复制位于 site-packages 中的所有库，包括 Django。如果你的网站不使用某些库，可以删除它们。

![][5]

接下来，创建初始 Django 应用程序。你可以像从命令行创建任何其他 Django 应用程序一样执行此操作，也可以使用 [Python Tools for Visual Studio][Python Tools for Visual Studio] 创建项目。两个选项都会在此演示。

**选项 1：**
若要从命令行创建新项目，请运行以下命令。该命令将在 DjangoApplication 文件夹中创建 Django 应用程序：

     C:\Python27\python.exe -m django.bin.django-admin startproject DjangoApplication

![][6]

**选项 2：**
你还可以使用 Python Tools for Visual Studio 创建新网站。启动安装了 Python Tools for Visual Studio 的 Visual Studio，然后选择“文件”-\>“新建项目”。钻取到“其他语言”下的“Python”项目，然后选择“Django 应用程序”。输入“DjangoApplication”作为项目名称，并确保未选中“创建解决方案的目录”以获得与从命令行创建 Django 应用程序时完全相同的目录结构。此选项可以为你配备好 Visual Studio 解决方案和项目文件，从而为你提供优质的本地开发体验，包括模板调试和智能感知。

![][7]

现在，你只需添加你刚添加的所有文件并将网站推送到 Git。为此，请运行以下命令：

    git add DjangoApplication site-packages
    git commit -m "Initial site"
    git remote add azure https://dinov@pythonwebsite.scm.chinacloudsites.cn/PythonWebSite.git
    git push azure master

第一个命令将添加要跟踪的未跟踪文件。第二个命令将提交你刚添加到存储库中的文件。第三个命令为你的存储库添加名为 *azure* 的远程存储库。最后一个命令会采用所做的更改，并将它们推送到远程存储库。这最后一个命令还会启动部署。完成后，你应该会看到这样的结果：

![][8]

在执行推送后，你将看到 Azure 门户刷新并显示活动部署：

![][9]

## 网站配置

现在你需要配置网站，以便让它了解你的 Django 项目并使用 wfastcgi 处理程序。为此，你可以单击屏幕顶部的“配置”选项卡，然后向下滚动到该页的下半部分，其中包含“应用程序设置”和“处理程序映射”部分。

此处设置的所有设置将在实际请求时转变为环境变量。这意味着你可以使用它们来配置 DJANGO\_SETTINGS\_MODULE 环境变量以及 PYTHONPATH 和 WSGI\_HANDLER。如果你的应用程序具有其他配置值，则你可以在此处分配这些值，然后从环境中选取它们。有时，你将需要设置你的网站中文件的路径。例如，你要为 PYTHONPATH 执行此操作。在作为 Azure 网站运行时，你的网站将位于“D:\\home\\site\\wwwroot\\”，因此你可以在需要磁盘上文件的完整路径的任何位置使用它。

若要设置 Django 应用程序，需要创建三个环境变量。第一个是 DJANGO\_SETTINGS\_MODULE，它提供将用于配置的 Django 应用程序的模块名称。第二个是 PYTHONPATH 环境变量，它指定设置模块所在的包。第三个是 WSGI\_HANDLER。此变量是模块/包名称，后跟要使用的模块中的属性（例如，mypackage.mymodule.handler）。添加括号以指示应调用该属性。因此，对于这些变量，可将它们设置为：

    DJANGO_SETTINGS_MODULE    DjangoApplication.settings
    PYTHONPATH                D:\home\site\wwwroot\DjangoApplication;D:\home\site\wwwroot\site-packages
    WSGI_HANDLER              django.core.handlers.wsgi.WSGIHandler()

![][10]

现在你需要配置处理程序映射。为此，请使用 Python 解释程序的路径和 wfastcgi.py 脚本的路径为所有扩展注册处理程序：

    EXTENSION                 *
    SCRIPT PROCESSOR PATH     D:\python27\python.exe
    ADDITIONAL ARGUMENTS      D:\python27\scripts\wfastcgi.py

![][11]

此时，你可以单击底部的“保存”按钮。

最后，返回仪表板。向下转到左侧的“网站 URL”。单击该链接并打开你的新 Django 网站，它看上去像这样：

![][12]

## 后续步骤

从这里，你可以通过使用已使用的工具继续开发 Django 应用程序。如果你使用 [Python Tools for Visual Studio][Python Tools for Visual Studio] 进行开发，则你将很可能需要安装 [VisualGit][VisualGit] 来获取 Visual Studio 中的源代码管理集成。

你的应用程序可能具有除 Python 和 Django 以外的其他依赖项。如果你通过使用 [][]<http://azure.microsoft.com/zh-cn/develop/python/></a> 中的安装程序安装了 Python，表明你已安装 PIP，并可以使用它来快速添加新依赖项。例如，若要安装自然语言工具包及其所有依赖项，请键入：

    pip install nltk

然后，你需要通过从 C:\\Python27\\Lib\\site-packages 将文件复制到你的本地 site-packages 目录来更新 site-packages 目录。

复制文件后，运行命令 **git status** 来查看新添加的文件，再依次运行 **git add** 和 **git commit** 向存储库提交更改。最后，你可以执行 **git push**，将更新的网站部署到 Azure。

现在你可以像通常一样转到 DjangoApplication 目录并使用 manage.py 来开始将新应用程序添加到 Django 项目中。

  [create-account-and-websites-note]: ../includes/create-account-and-websites-note.md
  []: http://azure.microsoft.com/zh-cn/develop/python/
  [msysgit]: http://code.google.com/p/msysgit/
  []: ./media/web-sites-python-create-deploy-django-app/django-ws-003.png
  [1]: ./media/web-sites-python-create-deploy-django-app/django-ws-004.png
  [2]: ./media/web-sites-python-create-deploy-django-app/django-ws-005.png
  [3]: ./media/web-sites-python-create-deploy-django-app/django-ws-006.png
  [4]: ./media/web-sites-python-create-deploy-django-app/django-ws-007.png
  [5]: ./media/web-sites-python-create-deploy-django-app/django-ws-008.png
  [Python Tools for Visual Studio]: http://pytools.codeplex.com/
  [6]: ./media/web-sites-python-create-deploy-django-app/django-ws-010.png
  [7]: ./media/web-sites-python-create-deploy-django-app/django-ws-011.png
  [8]: ./media/web-sites-python-create-deploy-django-app/django-ws-013.png
  [9]: ./media/web-sites-python-create-deploy-django-app/django-ws-014.png
  [10]: ./media/web-sites-python-create-deploy-django-app/django-ws-015.png
  [11]: ./media/web-sites-python-create-deploy-django-app/django-ws-016.png
  [12]: ./media/web-sites-python-create-deploy-django-app/django-ws-017.png
  [VisualGit]: http://code.google.com/p/visualgit/
