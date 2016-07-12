<properties
	pageTitle="创建 Jupyter/IPython Notebook | Azure"
	description="了解如何在使用 Azure 中的资源管理器部署模型创建的 Linux 虚拟机上部署 Jupyter/IPython Notebook。"
	services="virtual-machines-linux"
	documentationCenter="python"
	authors="crwilcox"
	manager="wpickett"
	editor=""
	tags="azure-service-management,azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="11/10/2015"
	wacn.date="05/24/2016"/>


# Azure 上的 Jupyter Notebook

[Jupyter 项目](http://jupyter.org)（以前称为 [IPython 项目](http://ipython.org)），提供了一套使用功能强大的交互式 shell 进行科学计算的工具，实现了将代码执行与创建实时计算文档相结合。这些 Notebook 文件可以包含任意文本、数学公式、输入代码、结果、图形、视频以及新型 Web 浏览器能够显示的任何其他种类的媒体。无论你是第一次使用 Python 并希望在有趣的交互式环境中了解它，还是执行一些严格的并行/技术计算，Jupyter Notebook 都是一个很好的选择。

![屏幕快照](./media/virtual-machines-linux-jupyter-notebook/ipy-notebook-spectral.png)
使用 SciPy 和 Matplotlib 包来分析录音结构。


## Jupyter 的两种方法：Azure Notebook 或自定义部署
可以使用 Azure 提供的服务[快速开始使用 Jupyter](http://blogs.technet.com/b/machinelearning/archive/2015/07/24/introducing-jupyter-notebooks-in-azure-ml-studio.aspx)。通过使用 Azure Notebook 服务，你可以轻松访问 Jupyter 的可缩放计算资源（包含 Python 的所有功能及其许多库）的可 Web 访问接口。由于安装由该服务处理，用户无需管理和用户配置即可访问这些资源。

如果 Notebook 服务不适用于你的方案，请继续阅读本文，它将会说明如何在 Azure 上使用 Linux 虚拟机 (VM) 部署 Jupyter Notebook。

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代经典部署模型。

[AZURE.INCLUDE [create-account-and-vms-note](../includes/create-account-and-vms-note.md)]

## 在 Azure 上创建并配置 VM

第一步是创建在 Azure 上运行的虚拟机 (VM)。此 VM 是云中的完整操作系统，它将用于运行 Jupyter Notebook。Azure 能够同时运行 Linux 和 Windows 虚拟机，我们将介绍如何在这两种类型的虚拟机上设置 Jupyter。

### 为 Jupyter 创建 Linux VM 并打开端口

按照[此处][portal-vm-linux]提供的说明可创建 *Ubuntu* 分发的虚拟机。本教程使用 Ubuntu Server 14.04 LTS。我们将假定用户名为 *azureuser*。

此步骤同时适用于 Linux 和 Windows VM。稍后我们将配置 Jupyter 以在端口 9999 上运行其 notebook 服务器。若要使此端口向公众提供的情况下，我们必须在 Azure 经典管理门户中创建一个终结点。此终结点打开 Azure 防火墙中的一个端口，并将公用端口 （HTTPS，443） 映射到虚拟机 (9999) 上的专用端口。

若要创建一个终结点，请转到 VM 仪表板单击"终结点"，然后"添加终结点"，创建新的终结点 （在此示例中称为 ipython_nb）。对于协议，443 （对于公共端口和专用端口 9999 选取 TCP:

![Screenshot](./media/virtual-machines-linux-jupyter-notebook/ipy-azure-linux-005.png)

此步骤之后，"终结点"仪表板选项卡将如下所示：

![Screenshot](./media/virtual-machines-linux-jupyter-notebook/ipy-azure-linux-006.png)

## 在 VM 上安装必需软件

若要在我们的 VM 上运行 Jupyter Notebook，必须首先安装 Jupyter 及其依赖项。使用 ssh 以及在创建 VM 时选择的用户名/密码对连接到你的 linux vm。在本教程中，我们将使用 PuTTY，并从 Windows 进行连接。

### 在 Ubuntu 上安装 Jupyter
使用 [Continuum Analytics](https://www.continuum.io/downloads) 所提供的链接之一安装 Anaconda，后者是常用的数据科学 python 分发。在撰写本文档之时，以下链接是最新版本。

#### 适用于 Linux 的 Anaconda 安装
<table>
  <th>Python 3.4</th>
  <th>Python 2.7</th>
  <tr>
    <td>
		<a href='https://3230d63b5fc54e62148e-c95ac804525aac4b6dba79b00b39d1d3.ssl.cf1.rackcdn.com/Anaconda3-2.3.0-Linux-x86_64.sh'>64 位</href>
	</td>
    <td>
		<a href='https://3230d63b5fc54e62148e-c95ac804525aac4b6dba79b00b39d1d3.ssl.cf1.rackcdn.com/Anaconda-2.3.0-Linux-x86_64.sh'>64 位</href>
	</td>
  </tr>
  <tr>
    <td>
		<a href='https://3230d63b5fc54e62148e-c95ac804525aac4b6dba79b00b39d1d3.ssl.cf1.rackcdn.com/Anaconda3-2.3.0-Linux-x86.sh'>32 位</href>
	</td>
    <td>
		<a href='https://3230d63b5fc54e62148e-c95ac804525aac4b6dba79b00b39d1d3.ssl.cf1.rackcdn.com/Anaconda-2.3.0-Linux-x86.sh'>32 位</href>
	</td>  
  </tr>
</table>


#### 在 Ubuntu 上安装 Anaconda3 2.3.0（64 位）
作为一个示例，这演示如何在 Ubuntu 上安装 Anaconda

	# install anaconda
	cd ~
	mkdir -p anaconda
	cd anaconda/
	curl -O https://3230d63b5fc54e62148e-c95ac804525aac4b6dba79b00b39d1d3.ssl.cf1.rackcdn.com/Anaconda3-2.3.0-Linux-x86_64.sh
	sudo bash Anaconda3-2.3.0-Linux-x86_64.sh -b -f -p /anaconda3

	# clean up home directory
	cd ..
	rm -rf anaconda/

	# Update Jupyter to the latest install and generate its config file
	sudo /anaconda3/bin/conda install jupyter -y
	/anaconda3/bin/jupyter-notebook --generate-config


![屏幕快照](./media/virtual-machines-linux-jupyter-notebook/anaconda-install.png)


### 配置 Jupyter 和使用 SSL
在安装后，我们需要花一些时间来为 Jupyter 设置配置文件。如果你在配置 Jupyter 时遇到问题，查看[针对运行 Notebook 服务器的 Jupyter 文档](http://jupyter-notebook.readthedocs.org/en/latest/public_server.html)可能会有帮助。

然后，通过 **cd** 命令进入配置文件目录来创建 SSL 证书并编辑配置文件。

在 Linux 上使用以下命令。

    cd ~/.jupyter

使用以下命令创建 SSL 证书（Linux 和 Windows）。

    openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout mycert.pem -out mycert.pem

请注意，由于我们创建的是自签名 SSL 证书，因此在连接到 Notebook 时，您的浏览器将显示安全警告。若要进行长期生产使用，您将需要使用与您的组织关联的正确签名的证书。由于证书管理超出了本演示的范围，我们目前将只讨论自签名证书。

除了使用证书外，您还必须提供密码以确保您的 Notebook 免遭未授权使用。出于安全原因，Jupyter 在其配置文件中使用加密密码，因此你将首先需要加密你的密码。IPython 提供了一个实用工具来执行此操作；在命令提示符下运行以下命令。

    /anaconda3/bin/python -c "import IPython;print(IPython.lib.passwd())"

这会提示你提供密码并确认，然后将输出密码。记下此密码，以便下一步使用。

    Enter password:
    Verify password:
    sha1:b86e933199ad:a02e9592e59723da722.. (elided the rest for security)

接下来，我们将编辑配置文件的配置文件，它是你所在目录中的 `jupyter_notebook_config.py` 文件。请注意，此文件可能不存在 - 只需创建它（如果不存在）。此文件包含多个字段，默认情况下会全部注释掉。你可以使用你喜欢的任何文本编辑器打开此文件，并应确保它至少具有以下内容。请务必将 config 里的 c.NotebookApp.password 替换为前一步中的 sha1。

    c = get_config()

    # You must give the path to the certificate file.
    c.NotebookApp.certfile = u'/home/azureuser/.jupyter/mycert.pem'

    # Create your own password as indicated above
    c.NotebookApp.password = u'sha1:b86e933199ad:a02e9592e5 etc... '

    # Network and browser details. We use a fixed port (9999) so it matches
    # our Azure setup, where we've allowed traffic on that port
    c.NotebookApp.ip = '*'
    c.NotebookApp.port = 9999
    c.NotebookApp.open_browser = False

### 运行 Jupyter Notebook

此时，我们已准备好启动 Jupyter Notebook。为此，请导航到要在其中存储 Notebook 的目录并使用以下命令启动 Jupyter Notebook 服务器。

    /anaconda3/bin/jupyter-notebook

你现在应能够通过地址 `https://[PUBLIC-IP-ADDRESS]:9999` 访问你的 Jupyter Notebook。

在你首次访问 Notebook 时，登录页会询问你的密码。在登录后，你会看到“Jupyter Notebook 仪表板”，它是所有 Notebook 操作的控制中心。从此页中，你可以创建新 Notebook，打开现有 Notebook。

![屏幕快照](./media/virtual-machines-linux-jupyter-notebook/jupyter-tree-view.png)

### 使用 Jupyter Notebook

如果单击“新建”按钮，你将看到以下打开页。

![屏幕快照](./media/virtual-machines-linux-jupyter-notebook/jupyter-untitled-notebook.png)

标记有 `In []:` 提示的区域是输入区域，在这里，你可以键入任何有效的 Python 代码，并且它会在你按 `Shift-Enter` 或单击“播放”图标（工具栏中的右指三角形）时执行。

## 强大的范例：具有丰富媒体的实时计算文档

Notebook 本身对使用过 Python 和字处理器的任何人来说应感觉非常自然，因为它在某些方面是这二者的融合：你可以执行 Python 代码块，但也可以通过使用工具栏中的下拉菜单将单元格的样式从“代码”更改为“标记”来保留笔记及其他文本。

Jupyter 不仅仅是字处理器，因为它允许混合计算和丰富媒体（文本、图形、视频以及新型 Web 浏览器可以显示的几乎任何内容）。你可以混合文本、代码、视频等内容！

![屏幕快照](./media/virtual-machines-linux-jupyter-notebook/jupyter-editing-experience.png)

在以下屏幕截图中，利用 Python 的许多用于科技计算的优秀库的功能，可以与进行复杂网络分析一样轻松地执行简单计算，且全部操作都在一个环境中完成。

将最新 Web 的功能与实时计算混合的此范例提供了许多可能性，并且非常适合云；Notebook 可以：

* 用作计算暂存器以记录对问题进行的探索工作。

* 用于以“实时”计算形式或以硬拷贝格式（HTML、PDF）与同事共享结果。

* 用于分发并显示涉及计算的实时教学材料，以便学生可以立即使用真实代码进行试验，对其进行修改，然后以交互方式重新执行它。

* 用于提供可由他人以立即重现、验证和扩展的方式显示研究结果的“可执行论文”。

* 用作协作计算的平台：多个用户可以登录到同一 notebook 服务器来共享实时计算会话。


如果你转到 IPython 源代码[存储库][]，你将找到具有 notebook 示例的整个目录，你可以下载这些示例，然后在自己的 Azure Jupyter VM 上进行试用。只需从网站中下载 `.ipynb` 文件并将它们上载到你的 notebook Azure VM 的仪表板上（或将它们直接下载到 VM 中）。

## 结束语

Jupyter Notebook 为交互访问 Azure 上的 Python 生态系统的功能提供了强大接口。它涵盖范围广泛的用例，包括简单的探索和学习 Python、数据分析和可视化、模拟和并行计算。生成的 Notebook 文档包含所执行的可与其他 Jupyter 用户共享的计算的完整记录。Jupyter Notebook 可用作本地应用程序，但它非常适合 Azure 上的云部署

还可通过 [Python Tools for Visual Studio][](PTVS) 在 Visual Studio 中使用 Jupyter 的核心功能。PTVS 是 Microsoft 提供的免费开放源代码插件，它可将 Visual Studio 转变为高级 Python 开发环境，其中包括具有 IntelliSense、调试、分析和并行计算集成功能的高级编辑器。

## 后续步骤

有关详细信息，请参阅 [Python 开发人员中心](/develop/python/)。

[portal-vm-linux]: /documentation/articles/virtual-machines-linux-quick-create-portal/
[存储库]: https://github.com/ipython/ipython
[Python Tools for Visual Studio]: http://aka.ms/ptvs

<!---HONumber=Mooncake_1221_2015-->