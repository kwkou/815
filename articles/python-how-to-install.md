<properties linkid="develop-python-install-python" urlDisplayName="Install Python" pageTitle="Install Python and the SDK - Azure" metaKeywords="Azure Python SDK" description="Learn how to install Python and the SDK to use with Azure." metaCanonical="" services="" documentationCenter="Python" title="Installing Python and the SDK" authors="" solutions="" manager="" editor="" />

# 安装 Python 和 SDK

在 Windows 上安装 Python 相当简单，并且它可能已预安装在 Mac 和 Linux 上。本指南将指导你完成安装过程，并使你的计算机可随时用于 Azure。本指南将帮助你了解以下信息：

-   Python Azure SDK 包含哪些内容？
-   要使用哪种 Python 以及哪个版本
-   在 Windows 上安装
-   在 Mac 和 Linux 上安装

## Python Azure SDK 包含哪些内容？

Azure SDK for Python 包括允许你针对 Azure 开发、部署和管理 Python 应用程序的组件。具体而言，Azure SDK for Python 包括以下组件：

-   **Azure 的 Python 客户端库**。这些类库提供用于访问 Azure 功能（例如数据管理服务和云服务）的接口。
-   **适用于 Mac 和 Linux 的 Azure 命令行工具**。这是一组用于部署和管理 Azure 服务（例如 Azure 网站和 Azure 虚拟机）的命令行工具。这些工具可在任何平台（包括 Mac、Linux 和 Windows）上使用。
-   **PowerShell for Azure（仅限 Windows）**。这是一组用于部署和管理 Azure 服务（例如云服务和虚拟机）的 PowerShell cmdlet。
-   **Azure 模拟器（仅限 Windows）**。计算和存储模拟器是一系列云服务和数据管理服务的本地模拟器，允许你在本地测试应用程序。Azure 模拟器仅在 Windows 上运行。

针对本次发布的核心方案是：

-   **Windows**：云服务 - 例如使用 Web 角色的 Django 网站
-   **Mac/Linux**：IaaS - 在 VM 中运行所需内容；通过 Python 使用 Azure 服务

## 要使用哪种 Python 以及哪个版本

提供了多种形式的 Python 解释程序 - 示例包括：

-   CPython - 最常用的标准 Python 解释程序
-   IronPython - 在 .Net/CLR 上运行的 Python 解释程序
-   Jython - 在 JVM 上运行的 Python 解释程序

在本发行版中，仅 **CPython** 经过测试且受支持。还建议至少使用版本 2.7。在不久的将来，还将增加 **IronPython** 支持。

## 从哪里获得 Python？

有多种方法可获得 CPython：

-   直接从 www.python.org 下载和安装
-   从知名发行版本（例如 www.enthought.com 或 www.ActiveState.com）下载和安装
-   从源构建！

除非你有特定需求，否则建议使用前两个选项，如下所述。

## 在 Windows 上安装

对于 Windows，你可以使用主 Python 开发人员中心提供的 [WebPI 安装程序][WebPI 安装程序]来简化安装（它将从 www.python.org 获取 CPython）。

**注意：**在 Windows Server 上，若要下载 WebPI 安装程序，你可能必须配置 IE ESC 设置（“开始”/“管理工具”/“服务器管理器”，然后单击“配置 IE ESC”，将其设置为“关闭”）

![how-to-install-python-webpi-1][how-to-install-python-webpi-1]

WebPI 安装程序提供安装 Python Azure 应用程序所需的一切内容，并为 Django 应用程序提供特定支持：

![how-to-install-Python-webpi-2][how-to-install-Python-webpi-2]

完成后，你应该看到确认你的安装选择的以下屏幕：

![how-to-install-python-webpi-3][how-to-install-python-webpi-3]

安装完成后，在提示符处键入 python 以确保一切正常。根据你的安装方式，你可能需要设置“path”变量以找到（正确版本的）Python：

![how-to-install-python-win-run][how-to-install-python-win-run]

虽然本次发布侧重于“使用 Django 生成 Web 应用程序”，但也可以浏览 [Python 包索引 (PyPI)][Python 包索引 (PyPI)] 以选择其他更多软件。如果你选择安装发行版本，表明你重点关注的是从 Web 开发到技术计算的各种方案。

若要查看在 **site-packages** 中安装了哪些 Python 包，请输入以下内容来查找其位置：

![how-to-install-python-win-site][how-to-install-python-win-site]

这将列出已在你的系统上安装的内容。

安装后，你应该具有位于默认位置的 Python、Django 和客户端库：

        C:\Python27\Lib\site-packages\windowsazure
        C:\Python27\Lib\site-packages\django

### Python Tools for Visual Studio

Python Tools for Visual Studio 是 Microsoft 提供的免费/OSS 插件，可将 VS 转换为完备的 Python IDE：

![how-to-install-python-ptvs][how-to-install-python-ptvs]

使用 Python Tools for Visual Studio 是可选的，但建议使用，因为它提供 Python 和 Django 项目/解决方案支持、调试、分析、模板编辑和智能感知、到云的部署等等。此加载项使用你的现有 VS2010 安装。如果你没有 VS2010，则 WebPI 将安装免费的集成外壳 + PTVS，这实际上为你提供了**完全免费的**“VS Python Express”IDE。有关详细信息，请参阅 [CodePlex 上的 Python Tools for Visual Studio][CodePlex 上的 Python Tools for Visual Studio]。

注意：虽然 PTVS 插件很小，但集成外壳将增加你的下载时间。集成外壳版本当前还不支持“添加 Azure 部署项目”功能。

## Windows 卸载

一般而言，**Azure SDK for Python June 2012** WebPI 产品不是应用程序，它实际上是捆绑在一起的不同产品（例如 32 位 Python 2.7、用于 Python 的 Azure 客户端 API、Django 等）的集合。由于它没有自己的常规卸载程序，因此你需要从 Windows 控制面板中逐个删除它所安装的程序。

如果你希望重新安装 **Azure SDK for Python**，只需以管理员身份打开 PowerShell 命令提示符并运行以下命令：

    rm -force "HKLM:\SOFTWARE\Microsoft\Windows Azure SDK for Python - June 2012"

然后重新运行 WebPI。

## 在 Linux 和 MacOS 上安装

Python 很可能已安装在你的开发计算机上。你可以通过输入以下内容来进行检查：

![how-to-install-python-linux-run][how-to-install-python-linux-run]

在这里，我们看到此 Azure Suse VM 安装了 CPython 2.7.2，它适合运行 Azure 教程和 Django 示例。如果你需要升级，请按照操作系统建议的包升级说明操作。不过请注意，一般来说，最好不要影响系统 Python（其他软件可能依赖该版本），因此最好是通过 [Virtualenv][Virtualenv] 安装更新版本。

若要安装 Python Azure 客户端库，请使用 **pip** 从 **PyPI** 获取它：

    curl https://raw.github.com/pypa/pip/master/contrib/get-pip.py | sudo python

以上命令将自动提示输入根密码。键入它，然后按 Enter。下一步：

    sudo /usr/local/bin/pip2.7 install azure

你现在应该看到已在 **site-packages** 下安装了这些客户端库。在 MacOS 上：

![MacOS site-packages][MacOS site-packages]

在从 mac/linux 进行开发时，本次发布支持两个主要方案：

1.  通过使用 Python 的客户端库来使用 Azure 服务

2.  在 Linux VM 中运行你的应用程序

第一个方案使你能够通过 Azure REST API 的 Pythonic 包装来创作利用 Azure PaaS 功能（例如 Blob 存储、队列等）的丰富 Web 应用程序。这些应用程序的工作方式与在 Windows、Mac 和 Linux 上相同。请参阅教程和操作方法指南来获取示例。你还可以从 Linux VM 中使用这些客户端库。

对于 VM 方案，你只需启动所选的 Linux VM（Ubuntu、CentOS、Suse）并运行/管理所需内容。例如，你可以在 Windows/Mac/Linux 计算机上运行 [IPython][IPython] REPL/notebook，并使你的浏览器指向在 Azure 上运行 IPython 引擎的 Linux 或 Windows 多处理器 VM。有关 IPython 安装的详细信息，请参阅其教程。

有关如何设置 Linux VM 的信息，请参阅 [Linux 管理][Linux 管理]部分。

## 其他软件和资源：

-   [Enthought Python 分发][Enthought Python 分发]
-   [ActiveState Python 分发][ActiveState Python 分发]
-   [SciPy - Scientific Python 库套件][SciPy - Scientific Python 库套件]
-   [NumPy - Python 的数字库][NumPy - Python 的数字库]
-   [Django 项目 - 成熟的 Web 框架/CMS][Django 项目 - 成熟的 Web 框架/CMS]
-   [IPython - Python 的高级 REPL/Notebook][IPython]
-   [Azure 上的 IPython Notebook][Azure 上的 IPython Notebook]
-   [CodePlex 上的 Python Tools for Visual Studio][CodePlex 上的 Python Tools for Visual Studio]
-   [Virtualenv][Virtualenv]

  [WebPI 安装程序]: http://go.microsoft.com/fwlink/?LinkId=254281&clcid=0x409
  [how-to-install-python-webpi-1]: ./media/python-how-to-install/how-to-install-python-webpi-1.png
  [how-to-install-Python-webpi-2]: ./media/python-how-to-install/how-to-install-python-webpi-2.png
  [how-to-install-python-webpi-3]: ./media/python-how-to-install/how-to-install-python-webpi-3.png
  [how-to-install-python-win-run]: ./media/python-how-to-install/how-to-install-python-win-run.png
  [Python 包索引 (PyPI)]: http://pypi.python.org/pypi
  [how-to-install-python-win-site]: ./media/python-how-to-install/how-to-install-python-win-site.png
  [how-to-install-python-ptvs]: ./media/python-how-to-install/how-to-install-python-ptvs.png
  [CodePlex 上的 Python Tools for Visual Studio]: http://pytools.codeplex.com
  [how-to-install-python-linux-run]: ./media/python-how-to-install/how-to-install-python-linux-run.png
  [Virtualenv]: http://pypi.python.org/pypi/virtualenv
  [MacOS site-packages]: ./media/python-how-to-install/how-to-install-python-mac-site.png
  [IPython]: http://ipython.org
  [Linux 管理]: /en-us/manage/linux/
  [Enthought Python 分发]: http://www.enthought.com
  [ActiveState Python 分发]: http://www.activestate.com
  [SciPy - Scientific Python 库套件]: http://www.scipy.org
  [NumPy - Python 的数字库]: http://www.numpy.org
  [Django 项目 - 成熟的 Web 框架/CMS]: http://www.djangoproject.com
  [Azure 上的 IPython Notebook]: http://windowsazure.com/zh-cn/documentation/articles/virtual-machines-python-ipython-notebook
