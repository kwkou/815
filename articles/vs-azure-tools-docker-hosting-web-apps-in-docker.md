<properties
   pageTitle="在 Docker 中托管 Web Apps | Microsoft Azure"
   description="了解如何使用 Visual Studio 来在 Docker 容器中托管 Web 应用程序。"
   services="visual-studio-online"
   documentationCenter="na"
   authors="kempb"
   manager="douge"
   editor="tglee" />
<tags
   ms.service="multiple"
   ms.date="08/17/2015"
   wacn.date="10/3/2015" />

# 在 Docker 中托管 Web Apps

[Docker](https://www.docker.com/whatisdocker/) 是轻型容器引擎，在某些方面类似于虚拟机，您可以将其用于托管应用程序和服务。Visual Studio 支持在 Ubuntu、CoreOS 和 Windows 上的 Docker。

此示例演示了如何使用 Visual Studio 2015 Tools for Docker 扩展将 ASP.NET 5 应用程序发布到 Azure 上的 Ubuntu Linux 虚拟机（此处称为 Docker 主机），其中 Azure 上已安装了 Docker 扩展和 ASP.NET 5 Web 应用程序。可使用相同的体验将应用程序发布到 Windows 容器。

您可以通过使用**自定义主机**设置将应用程序发布到在 Azure 上托管的新 Docker 主机，或发布到本地服务器、HYPER-V 或 Boot2Docker 主机。将您的应用程序发布到 Docker 主机之后，可以使用 Docker 命令行工具与您的应用程序已发布到的容器进行交互。

## 创建并发布新的 Docker 容器

在这些过程中，您将新建一个 ASP.NET 5 Web 应用程序项目，并创建容器主机，然后在 Docker 容器中生成并运行 Web 应用程序项目。若要开始，请下载并安装 [Visual Studio 2015 Tools for Docker](aka.ms/DockerToolsForVS)。

### 添加 ASP.NET 5 Web 应用程序项目

1. 新建 ASP.NET Web 应用程序项目。在主菜单上，选择“文件”，“新建项目”。在“C#”、“Web”下，选择“ASP.NET Web 应用程序”。

1. 在“ASP.NET 5 Preview 模板”的列表中，选择“网站”。

1. 由于 Web 应用将在 Docker 中托管/运行，请清除“在云中托管”复选框（如已选中），然后选择“确定”按钮。

  ![][0]

  这对于您通常将代码添加到 Web 应用以使其执行一些有帮助的操作非常关键，但对于此示例，我们只需将其保留为其默认设置即可。（请注意，您还可以选择使用现有的 ASP.NET 5 Web 应用。）

### 发布项目

1. 在 ASP.NET 项目的上下文菜单上选择“发布”。

1. 在“发布 Web”对话框中的“选择发布目标”部分，选择“Docker 容器”按钮。

    如果看不到“Docker 容器”选项，请确保您已安装 Visual Studio 2015 Tools for Docker，并确保您在前一过程中选择了 ASP.NET 5 网站模板。

    ![][1]

    随即会显示“选择 Docker 虚拟机”对话框。这允许您指定要在其中发布项目的 Docker 主机。您可以新建 Docker 主机或选择在 Azure 或其他位置托管的现有 VM。稍后在此示例中，我们将新建一个 Azure Docker 主机。

1. 如果您已登录到 Azure 帐户，请跳到步骤 5。如果您尚未登录到帐户，请选择“添加帐户”按钮。

    ![][2]

1. 在“登录到 Visual Studio”对话框中，输入您的 Azure 订阅的电子邮件帐户，然后选择“继续”按钮。

1. 选择“新建”按钮以创建新的 Azure Docker VM，然后选择“确定”按钮。

    ![][3]

    请注意，您还可以选择使用现有的 Docker 主机。若要执行此操作，可以在“现有的 Azure Docker 虚拟机”下拉列表中选择 Docker 主机，而不选择 “新建”按钮。此列表不会仅显示容器主机，它会列出您的 Azure 租户中所有 VM。

    或者您也可以选择发布到自定义的 Docker 主机。有关详细信息，请参阅本主题后面的[提供自定义的 Docker 主机](#BKMK_CustomHost)。

1. 在“在 Microsoft Azure 上创建虚拟机”对话框中输入以下信息。完成后，选择“确定”按钮。这将创建包含已配置的 Docker 扩展的 Linux 虚拟机。

    ![][4]

    请注意，您现在还可以选择使用 Windows Server 2016 Technical Preview 3 (TP3) 创建 Windows 容器主机。

    ![][8]

    |属性名称|设置|
    |---|---|
    |位置|将此设置更改为最接近您的区域设置的区域。|
    |DNS 名称|输入虚拟机的唯一名称。如果 Azure 接受该名称，则右侧将显示一个带有白色复选标记的绿色圆圈。如果该名称不被接受，将显示一个带有白色 x 的红色圆圈。在这种情况下，请输入新的唯一名称。|
    |映像|请选择要在 Docker 主机中使用的 OS 映像（如存在）。对于此示例，请选择 Ubuntu 服务器映像。（请注意，目前可用映像列表中已提供 Windows Server 映像。）|
    |用户名|输入虚拟机的唯一用户名。|
    |密码|输入用户密码并进行确认。|
    |证书目录 |这将指定存储 Docker 证书的文件夹。虽然您可以新建文件夹或指向现有的文件夹，但建议您使用默认证书文件夹 (C:\\Users\\[*username*]\\.docker)。否则，如果在另一个项目或系统上重用同一个主机，则身份验证选项将无法进行自动检索。|

1. 选择**证书目录**条目旁边的省略号 (...) 按钮，然后新建 Docker 证书文件夹，或导航到现有的 Docker 证书文件夹。

    如果未找到 VM 所需的 Docker 证书，将在条目旁边显示感叹号图标，以便通知您未找到所需的证书，如果继续将删除并重新创建任何现有的证书。

1. 选择“确定”按钮以开始创建 VM。您将收到一条消息，告知 Azure 中正在创建虚拟机。

    Visual Studio 创建 Azure 资源管理器 (ARM) 模板文件、参数文件和 PowerShell 脚本，以便将来可以再次执行这些命令。

    ![][7]

    Visual Studio 将此操作的进程输出到“输出”窗口。Visual Studio 调用 PowerShell 脚本以部署 VM。此脚本使用 Azure PowerShell cmdlet 部署 Azure 资源组。之后，另一个 PowerShell 脚本使用发布 Docker 命令来进行发布，这与您手动创建 Docker 主机所执行的操作相同。

    预配 Docker 主机需要等待一段时间，因此请检查“输出”窗口中的状态，以查看作业的完成时间。

1. 在 Azure 中对 Docker 主机预配完毕后，可以检查您在 Azure 门户上的帐户。您应该能够在 Azure 门户的“虚拟机”类别下看到新的虚拟机。

1. 在 Docker 主机准备就绪后，请返回并发布 Web 应用项目。在“解决方案资源管理器”中的 Web 应用程序项目节点的上下文菜单上，选择“发布”。Visual Studio 将基于您所创建的 VM 创建发布文件。

1. 在“发布 Web”对话框中的“连接” 选项卡上，选择“验证连接”框确保 Docker 主机已准备就绪。如果连接正常，请选择“发布”按钮以发布 Web 应用。

    第一次将应用发布到 Docker 主机时，将需要一些时间来下载在 Docker 文件中引用的任何基础映像（例如 **FROM** *imagename*）。

    请注意，Docker 文件特定于操作系统。如果您选择重新发布到其他操作系统，则需要重命名 Docker 文件，以便 Visual Studio 可以基于目标 OS 创建新的默认值。例如，如果先发布到 Linux 容器，之后决定要发布到 Windows，那么您应将 Docker 文件重命名为一个唯一名称，例如 DockerLinux。然后，在重新发布到 Windows 时，Visual studio 将为 Windows 重新创建默认的 Docker 文件。之后，当您重新发布两者中任何一个，只需针对 OS 选择适当的 Docker 文件。

## 提供自定义的 Docker 主机

前面的过程要求您创建在 Azure 上托管的 Docker 虚拟机。但是，如果您已在其他位置创建了现有的 Docker 主机，则可以选择发布到该主机而不是 Azure。

### 如何提供自定义的 Docker 主机

1. 在“选择 Docker 虚拟机”对话框中，选中“自定义的 Docker 主机”复选框。

    ![][5]

1. 选择“确定”按钮。

1. 在“发布 Web”对话框中，将值添加到“CustomDockerHost”部分中的设置，如：服务器 URL、映像名称、Dockerfile 位置以及主机和容器端口号。

    在“Docker 高级选项”部分中，您可以查看或更改“身份验证”选项和“运行”选项，以及 Docker 命令行。

    ![][6]

1. 输入所需的所有值后，选择“验证连接”按钮以确保与 Docker 主机的连接可正常工作。

1. 如果连接正常，则可以选择“下一步”按钮以查看将要发布的组件列表，或者您可以选择“发布”按钮以立即发布该项目。

## 测试 Docker 主机

现在，该项目已发布到 Azure 上的 Docker 主机，您可以通过检查其设置来对其进行测试。因为 Docker 命令行工具安装有 Visual Studio 扩展，您可以直接从 Windows 命令提示符对 Docker 发出命令。

下面的过程用于与已部署到 Azure 的 Docker 主机进行通信。

### 如何测试 Docker 主机

1. 打开 Windows 命令提示符。

1. 分配 Docker 主机，并验证环境变量。若要执行此操作，请在命令提示符处输入以下命令。（将您的 Docker 主机名称替换为 *NameofAzureVM*。）

    ```
    Set DOCKER_HOST=tcp://<NameofAzureVM>.cloudapp.net:2376
    Set DOCKER_TLS_VERIFY=1
    ```

    调用这些命令可避免必须向您发出的每一个命令都添加 `–H (Host) tcp://<NameofAzureVM>.cloudapp.net:2376` 和 `--TLSVERIFY`。

1. 现在您可以发出类似这样的命令来测试 Docker 主机是否已存在且正常工作。

    |命令行|说明|
    |---|---|
    |docker 信息|获取 Docker 版本信息。|
    |docker ps|获取正在运行的容器的列表。|
    |docker ps – a|获取容器列表，包括已停止的容器。|
    |docker 日志<Docker container name>|获取指定容器的日志。|
    |docker 映像|获取映像的列表。|

    有关 Docker 命令的完整列表，只需在命令提示符处输入命令 `docker`。有关详细信息，请参阅 [Docker 命令行](https://docs.docker.com/reference/commandline/cli/)。

## 后续步骤

现在您已经拥有一个 Docker 主机，您可以向其发出 Docker 命令。若要了解有关 Docker 的详细信息，请参阅 [Docker 文档](https://docs.docker.com/)和 [Docker 在线教程](https://www.docker.com/tryit/)。

有关在 Visual Studio 中使用 Docker 的问题，请参阅 [Docker 错误疑难解答](vs-docker-troubleshooting-docker-errors.md)。

[0]: ./media/vs-azure-tools-docker-hosting-web-apps-in-docker/IC796678.png
[1]: ./media/vs-azure-tools-docker-hosting-web-apps-in-docker/IC796679.png
[2]: ./media/vs-azure-tools-docker-hosting-web-apps-in-docker/IC796680.png
[3]: ./media/vs-azure-tools-docker-hosting-web-apps-in-docker/IC796681.png
[4]: ./media/vs-azure-tools-docker-hosting-web-apps-in-docker/IC796682.png
[5]: ./media/vs-azure-tools-docker-hosting-web-apps-in-docker/IC796683.png
[6]: ./media/vs-azure-tools-docker-hosting-web-apps-in-docker/IC796684.png
[7]: ./media/vs-azure-tools-docker-hosting-web-apps-in-docker/IC796685.png
[8]: ./media/vs-azure-tools-docker-hosting-web-apps-in-docker/IC796686.png

<!---HONumber=71-->