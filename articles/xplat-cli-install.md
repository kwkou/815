<properties
	pageTitle="安装 Azure 命令行界面 | Azure"
	description="安装适用于 Mac、Linux 和 Windows 的 Azure CLI 即可使用 Azure 服务"
	editor=""
	manager="timlt"
	documentationCenter=""
	authors="dlepow"
	services=""
	tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="multiple"
	ms.date="04/07/2016"
	wacn.date="05/05/2016"/>

# 安装 Azure CLI

快速安装 Azure 命令行界面 (Azure CLI)，以便使用一组基于 shell 的开源命令在 Azure 中创建和管理资源。可以使用多个安装选项：使用针对不同操作系统提供的安装包之一、从 npm 包安装，或者在 Docker 主机中以容器的形式安装 Azure CLI。有关更多选项和背景信息，请参阅 [GitHub](https://github.com/azure/azure-xplat-cli) 上的项目存储库。


安装了 Azure CLI 之后，你将能够[将它与 Azure 订阅关联](/documentation/articles/xplat-cli-connect)，并从命令行界面（Bash、终端、命令提示符等）运行 **azure** 命令，以使用 Azure 资源。


## 使用安装程序

提供以下安装程序包：

* [Windows 安装程序][windows-installer]

* [OS X 安装程序](http://go.microsoft.com/fwlink/?LinkId=252249)

* [Linux 安装程序][linux-installer]


## 安装 npm 包

如果你的系统上已安装最新的 Node.js 和 npm，请运行以下命令来安装 Azure CLI 包。（在 Linux 分发版中，可能需要使用 **sudo** 才能成功运行 __npm__ 命令。）

	npm install azure-cli -g

> [AZURE.NOTE]如果需要为操作系统安装或更新 Node.js 和 npm，请参阅 [Nodejs.org](https://nodejs.org/en/download/package-manager/) 上的文档。我们建议安装最新的 Node.js LTS 版本 (4.x)。如果你使用旧版本，可能会遇到安装错误。


## 使用 Docker 容器

在 Docker 主机中，运行：

```
docker run -it microsoft/azure-cli
```

## 运行 Azure CLI 命令
安装了 Azure CLI 之后，你将可以从命令行用户界面（Bash、终端、命令提示符等）运行 **azure** 命令。例如，若要运行帮助命令，请键入以下命令：

```
azure help
```

若要查看安装的 Azure CLI 版本，请键入以下命令：

```
azure --version
```

你现在已准备就绪！ 若要访问所有 CLI 命令以使用自己的资源，请[从 Azure CLI 连接到你的 Azure 订阅](xplat-cli-connect.md)。


## 更新 CLI

Microsoft 会频繁发布 Azure CLI 的更新版本。使用适用于你的操作系统的安装程序重新安装 CLI，或者如果已安装最新的 Node.js 和 npm，请键入以下命令（在 Linux 分发上可能需要使用 **sudo**）进行更新。

```
npm update -g azure-cli
```

## 后续步骤 

* [从 CLI 连接到 Azure 订阅](/documentation/articles/xplat-cli-connect)以创建和管理 Azure 资源。

* 若要了解有关 Azure CLI、下载源代码、报告问题或贡献项目的详细信息，请访问[适用于 Azure CLI 的 GitHub 存储库](https://github.com/azure/azure-xplat-cli)。

* 如果你遇到有关使用 Azure CLI 或 Azure 的问题，请访问 [Azure 论坛](http://social.msdn.microsoft.com/Forums/windowsazure/home)。

* 对于 Linux 系统，你还可以通过从[源](http://aka.ms/linux-azure-cli)进行构建的方式来安装 Azure CLI。有关从源代码生成的详细信息，请参阅源存档中随附的 INSTALL 文件。

[mac-installer]: http://go.microsoft.com/fwlink/?LinkId=252249
[windows-installer]: http://go.microsoft.com/?linkid=9828653&clcid=0x409
[linux-installer]: http://go.microsoft.com/fwlink/?linkid=253472
[cliasm]: /documentation/articles/virtual-machines-command-line-tools
[cliarm]: /documentation/articles/xplat-cli-azure-resource-manager

<!---HONumber=Mooncake_0425_2016-->