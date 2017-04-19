<properties
    pageTitle="安装 Azure CLI 1.0 | Azure"
    description="安装适用于 Mac、Linux 和 Windows 的 Azure CLI 1.0 即可使用 Azure 服务"
    editor=""
    manager="timlt"
    documentationcenter=""
    author="squillace"
    services="virtual-machines-linux,virtual-network,storage,azure-resource-manager"
    tags="azure-resource-manager,azure-service-management"
    translationtype="Human Translation" />
    
<tags
    ms.assetid="bdb776c8-7a76-4f3a-887c-236b4fffee10"
    ms.service="multiple"
    ms.workload="multiple"
    ms.tgt_pltfrm="command-line-interface"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/20/2017"
    wacn.date="04/17/2017"
    ms.author="rasquill"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="dd572eaa992332aee52b50b74a375ee3f9a43012"
    ms.lasthandoff="04/07/2017" />

# <a name="install-the-azure-cli-10"></a>安装 Azure CLI 1.0
> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/powershell-install-configure/)
- [Azure CLI 1.0](/documentation/articles/cli-install-nodejs/)
- [Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli/)

> [AZURE.IMPORTANT]
> 本主题介绍如何安装 Azure CLI 1.0，它基于 nodeJs 生成，并支持所有经典部署 API 调用以及大量资源管理器部署活动。 应使用 [Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/overview) 进行新的或前瞻性的 CLI 部署和管理。

快速安装 Azure 命令行接口 (Azure CLI 1.0)，以便使用一组基于 shell 的开源命令在 Azure 中创建和管理资源。 在计算机上安装这些跨平台工具时有多个选项：

- **npm 包** - 运行 npm（JavaScript 的包管理器）可在 Linux 分发版或操作系统中安装最新的 Azure CLI 1.0 包。 计算机上需要存在 node.js 和 npm。
- **安装程序** ：下载安装程序，在 Mac 或 Windows 上轻松地执行安装。
- **Docker 容器** ：在已经准备好运行的 Docker 容器中开始使用的最新 CLI。 计算机上需要有 Docker 主机。

有关更多选项和背景信息，请参阅 [GitHub](https://github.com/azure/azure-xplat-cli) 上的项目存储库。

安装 Azure CLI 1.0 后，[将它连接到 Azure 订阅](/documentation/articles/xplat-cli-connect/)，并从命令行接口（Bash、终端、命令提示符等）运行 **azure** 命令，从而使用 Azure 资源。

## <a name="option-1-install-an-npm-package"></a>选项 1：安装 npm 包
若要从 npm 包安装 CLI，请确保已下载并安装 [最新的 Node.js 和 npm](https://nodejs.org/en/download/package-manager/)。 然后，运行 **npm install** 来安装 azure-cli 包：

    npm install -g azure-cli

在 Linux 分发版中，可能需要使用 **sudo** 才能成功运行 **npm** 命令，如下所示：

    sudo npm install -g azure-cli

> [AZURE.NOTE]
> 如果需要在 Linux 分发版或 OS 中安装或更新 Node.js 和 npm，建议安装最新的 Node.js LTS 版本 (4.x)。 如果使用旧版本，可能会遇到安装错误。

如果喜欢，也可以将 npm 包的最新 Linux [tar 文件][linux-installer]下载到本地。 然后，如下所示安装所下载的 npm 包（在 Linux 分发版中可能需要使用 **sudo**）：

    npm install -g <path to downloaded tar file>

## <a name="option-2-use-an-installer"></a>选项 2：使用安装程序
如果使用 Mac 或 Windows 计算机，以下 CLI 安装程序可供下载：

- [Mac OS X 安装程序][mac-installer]
- [Windows MSI][windows-installer]

> [AZURE.TIP]
> 在 Windows 上，还可以下载 [Web 平台安装程序](https://go.microsoft.com/?linkid=9828653) 来安装 CLI。 借助此安装程序，可选择在安装 CLI 后再安装其他 Azure SDK 和命令行工具。

## <a name="option-3-use-a-docker-container"></a>选项 3：使用 Docker 容器
如果已将计算机设置为 [Docker](https://docs.docker.com/engine/understanding-docker/) 主机，可以在 Docker 容器中运行最新的 Azure CLI 1.0。 运行以下命令（在 Linux 分发版中，可能需要使用 **sudo**）：

    docker run -it microsoft/azure-cli

## <a name="run-azure-cli-10-commands"></a>运行 Azure CLI 1.0 命令
安装 Azure CLI 1.0 后，从命令行用户界面（Bash、终端、命令提示符等）运行 **azure** 命令。 例如，若要运行帮助命令，请键入以下命令：

    azure help

> [AZURE.NOTE]
> 在某些 Linux 分发版中，可能会收到类似于“`/usr/bin/env: ‘node’: No such file or directory`”的错误。 此错误来自最近安装在 /usr/bin/nodejs 中的 Node.js 安装。 若要解决此错误，请运行以下命令创建 /usr/bin/node 的符号链接：

    sudo ln -s /usr/bin/nodejs /usr/bin/node

若要查看安装的 Azure CLI 1.0 版本，请键入以下命令：

    azure --version

你现在已准备就绪！ 若要访问所有 CLI 命令来使用自己的资源，请[从 Azure CLI 连接到 Azure 订阅](/documentation/articles/xplat-cli-connect/)。

> [AZURE.NOTE]
> 首次使用 Azure CLI 时，会看到一条消息，询问是否允许 Microsoft 收集使用情况信息。 参与为自愿性质。 如果选择参与，通过运行 `azure telemetry --disable`即可随时停止参与。 若要随时启用参与，请运行 `azure telemetry --enable`。

## <a name="update-the-cli"></a>更新 CLI
Microsoft 会频繁发布 Azure CLI 的更新版本。 使用适用于操作系统的安装程序来重新安装 CLI，或运行最新的 Docker 容器。 如果已安装最新的 Node.js 和 npm，请键入以下命令（在 Linux 分发版中可能需要使用 **sudo**）进行更新。

    npm update -g azure-cli

## <a name="enable-tab-completion"></a>启用 tab 自动补全
Mac 和 Linux 支持 tab 自动补全 CLI 命令。

如要在 zsh 中启用，运行：

    echo '. <(azure --completion)' >> .zshrc

若要在 bash 中启用，运行：

    azure --completion >> ~/azure.completion.sh
    echo 'source ~/azure.completion.sh' >> ~/.bash_profile

## <a name="next-steps"></a>后续步骤
- [从 CLI 连接到 Azure 订阅](/documentation/articles/xplat-cli-connect/)以创建和管理 Azure 资源。
- 若要了解有关 Azure CLI、下载源代码、报告问题或贡献项目的详细信息，请访问 [适用于 Azure CLI 的 GitHub 存储库](https://github.com/azure/azure-xplat-cli)。
- 如果在使用 Azure CLI 或 Azure 方面有疑问，请访问 [Azure 论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=azurescripting)。

[mac-installer]: http://aka.ms/mac-azure-cli
[windows-installer]: http://aka.ms/webpi-azure-cli
[linux-installer]: http://aka.ms/linux-azure-cli
[cliasm]:https://docs.microsoft.com/zh-cn/cli/azure/get-started-with-azure-cli
[cliarm]: /documentation/articles/azure-cli-arm-commands/

