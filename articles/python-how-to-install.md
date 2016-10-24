<properties
	pageTitle="安装 Python 和 SDK - Azure"
	description="了解如何安装 Python 和 SDK 以与 Azure 一起使用。"
	services=""
	documentationCenter="python"
	authors="lmazuel"
	manager="wpickett"
	editor=""/>  


<tags
	ms.service="multiple"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="python"
	ms.topic="article"
	ms.date="09/06/2016"
	wacn.date="10/24/2016"/>  


# 安装 Python 和 SDK

在 Windows 上安装 Python 很简单，并且 Mac、Linux 和 [Bash for Windows](https://msdn.microsoft.com/commandline/wsl/about) 上可能已预安装了 Python。本指南将指导你完成安装过程，并使你的计算机可随时用于 Azure。

## Python Azure SDK 包含哪些内容？

Azure SDK for Python 包括允许您针对 Azure 开发、部署和管理 Python 应用程序的组件。具体而言，Azure SDK for Python 包括以下组件：

* **管理库**。这些类库提供管理 Azure 资源（例如存储帐户、虚拟机）的接口。

* **运行时库**。这些类库提供用于访问 Azure 功能（例如存储和服务总线）的接口。

## 要使用哪种 Python 以及哪个版本

提供了多种形式的 Python 解释程序 - 示例包括：

* CPython - 最常用的标准 Python 解释程序
* PyPy - 快速、CPython 的合规替代实现
* IronPython - 在 .Net/CLR 上运行的 Python 解释程序
* Jython - 在 Java 虚拟机上运行的 Python 解释程序

**CPython** v2.7 或 v3.3+ 和 PyPy 5.4.0 已经过 Python Azure SDK 测试，且受其支持。

## 从哪里获得 Python？

有多种方法可获得 CPython：

* 直接从 [www.python.org][] 获取
* 从知名发行版本（例如 [www.continuum.io][]、[www.enthought.com][] 或 [www.activestate.com][]）获取
* 从源构建！

除非有特定需求，否则建议使用前两个选项。

## Windows、Linux 和 MacOS 上的 SDK 安装（仅限客户端库）

如果你已安装 Python，则可以使用 pip 在现有的 Python 2.7 或 Python 3.3+ 环境中安装所有客户端库的捆绑包。这将从 [Python 包索引][] (PyPI) 中下载包。

可能需要管理员权限：

- 对于 Linux 和 MacOS，使用 `sudo` 命令：`sudo pip install azure-mgmt-compute`。
- Windows：以管理员身份打开 PowerShell 提示符或命令提示符

可以为每个 Azure 服务分别安装每个库：

    $ pip install azure-batch          # Install the latest Batch runtime library
    $ pip install azure-mgmt-scheduler # Install the latest Storage management library


可以使用 `--pre` 标志安装预览包：


    $ pip install --pre azure-mgmt-compute # will install only the latest Compute Management library


还可以使用 `azure` 元程序包在单个行中安装一组 Azure 库。由于此元程序包中并非所有包都已作为稳定版本发布，因此 `azure` 元程序包仍为预览版。但是这一次，核心程序包的代码质量/完整性方面都可以被视为是“稳定”的
- 我们将尽快将其正式标记为“稳定”（与其他语言同步）。在那之前，我们不会作出任何重大的更改。

由于这是预览版本，需要使用 `--pre` 标志：


    $ pip install --pre azure

   
或直接


    $ pip install azure==2.0.0rc6

## 获取多个软件包

[Python 包索引][] (PyPI) 提供丰富的 Python 库。如果选择安装发行版本，表明你重点关注的是从 Web 开发到技术计算的各种方案。


## Python Tools for Visual Studio

[Python Tools for Visual Studio][] (PTVS) 是 Microsoft 提供的免费/OSS 插件，可将 VS 转换为完备的 Python IDE：

![how-to-install-python-ptvs](./media/python-how-to-install/how-to-install-python-ptvs.png)  


可以选择是否使用 PTVS，但建议使用，因为它能够提供 Python 和 Web 项目/解决方案支持、调试、分析、交互式窗口、模板编辑和智能感知。

PTVS 还可以轻松实现部署到 Azure，同时支持部署到[云服务][]和[ Web 应用][]。

PTVS 适用于你现有的 Visual Studio 2013 或 2015 版本的安装。有关文档、下载和讨论的信息，请参阅 [Python Tools for Visual Studio]。

## Python Azure 方案适用于 Linux 和 MacOS

对于 Linux 或 MacOS，支持的主要 Azure 方案为：

1. 通过使用 Python 的客户端库来使用 Azure 服务

2. 在 Linux VM 中运行你的应用程序

3. 使用 Git 开发和发布到 Azure 网站

第一个方案使你能够通过 Azure REST API 的 Pythonic 包装来创作利用 Azure PaaS 功能（例如 [Blob 存储][]、[队列存储][]、[表存储][]等）的丰富 Web 应用。这些应用程序在 Windows、Mac 和 Linux 上的工作方式是相同的。此外可以从本地开发计算机或在 Azure 上运行的 Linux VM 中使用这些客户端库。

对于 VM 方案，你只需启动所选的 Linux VM（Ubuntu、CentOS、Suse）并运行/管理所需内容。例如，您可以在 Windows/Mac/Linux 计算机上运行 [IPython][] REPL/notebook，并使您的浏览器指向在 Azure 上运行 IPython 引擎的 Linux 或 Windows 多处理器 VM。请参阅 [Azure 上的 IPython Notebook][] 教程，以了解详细信息。

有关如何安装 Linux VM 的信息，请参阅[创建运行 Linux 的虚拟机][]教程。

使用 Git 部署，可以从任何操作系统开发 Python Web 应用并将其发布到 Azure Web 应用。当将您的存储库推送到 Azure 时，它将自动创建虚拟环境和 pip 安装所需的包。

有关开发和发布 Azure Web 应用的详细信息，请参阅有关教程：[使用 Django 创建 Web 应用][]、[使用 Bottle 创建 Web 应用][]和[使用 Flask 创建 Web 应用][]。有关使用任何 WSGI 合规框架的更多常规信息，请参阅[使用 Azure Web 应用配置 Python][]。


## 其他软件和资源：

* [Azure SDK for Python ReadTheDocs](http://azure-sdk-for-python.readthedocs.io/en/latest/)
* [Azure SDK for Python Github](https://github.com/Azure/azure-sdk-for-python)
* [Python 的官方 Azure 示例](https://azure.microsoft.com/documentation/samples/?platform=python)
* [Continuum Analytics Python 分发][]
* [Enthought Python 分发][]
* [ActiveState Python 分发][]
* [SciPy - Scientific Python 库套件][]
* [NumPy - Python 的数字库][]
* [Django 项目 - 成熟的 Web 框架/CMS][]
* [IPython - Python 的高级 REPL/Notebook][]
* [Azure 上的 IPython Notebook][]
* [GitHub 上的 Python Tools for Visual Studio][]
* [Python 开发人员中心](/develop/python/)

[Continuum Analytics Python 分发]: http://continuum.io
[Enthought Python 分发]: http://www.enthought.com
[ActiveState Python 分发]: http://www.activestate.com
[www.python.org]: http://www.python.org
[www.continuum.io]: http://continuum.io
[www.enthought.com]: http://www.enthought.com
[www.activestate.com]: http://www.activestate.com
[SciPy - Scientific Python 库套件]: http://www.scipy.org
[NumPy - Python 的数字库]: http://www.numpy.org
[Django 项目 - 成熟的 Web 框架/CMS]: http://www.djangoproject.com
[IPython - Python 的高级 REPL/Notebook]: http://ipython.org
[IPython]: http://ipython.org
[Azure 上的 IPython Notebook]: /documentation/articles/virtual-machines-linux-jupyter-notebook/
[云服务]: /documentation/articles/cloud-services-python-ptvs/
[ Web 应用]: /documentation/articles/web-sites-python-ptvs-django-mysql/
[Python Tools for Visual Studio]: http://aka.ms/ptvs
[GitHub 上的 Python Tools for Visual Studio]: http://microsoft.github.io/PTVS/
[Python 包索引]: http://pypi.python.org/pypi
[Microsoft Azure SDK for Python 2.7]: http://go.microsoft.com/fwlink/?LinkId=254281
[Microsoft Azure SDK for Python 3.4]: http://go.microsoft.com/fwlink/?LinkID=516990
[Setting up a Linux VM via the Azure portal]: /documentation/articles/create-and-configure-opensuse-vm-in-portal/
[How to use the Azure Command-Line Interface]: /documentation/articles/crossplat-cmd-tools/
[创建运行 Linux 的虚拟机]: /documentation/articles/virtual-machines-linux-quick-create-portal/
[使用 Django 创建 Web 应用]: /documentation/articles/web-sites-python-create-deploy-django-app/
[使用 Bottle 创建 Web 应用]: /documentation/articles/web-sites-python-create-deploy-bottle-app/
[使用 Flask 创建 Web 应用]: /documentation/articles/web-sites-python-create-deploy-flask-app/
[使用 Azure Web 应用配置 Python]: /documentation/articles/web-sites-python-configure/
[表存储]: /documentation/articles/storage-python-how-to-use-table-storage/
[队列存储]: /documentation/articles/storage-python-how-to-use-queue-storage/
[Blob 存储]: /documentation/articles/storage-python-how-to-use-blob-storage/

<!---HONumber=Mooncake_1017_2016-->