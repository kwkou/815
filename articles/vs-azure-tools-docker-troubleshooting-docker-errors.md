<properties
   pageTitle="使用 Visual Studio 对 Windows 排查 Docker 客户端错误 | Microsoft Azure"
   description="在 Windows 上通过使用 Visual Studio 排查在使用 Visual Studio 创建网站并将其部署到 Docker 时遇到的问题。"
   services="visual-studio-online"
   documentationCenter="na"
   authors="kempb"
   manager="douge"
   editor="tglee" />
<tags
   ms.service="multiple"
   ms.date="05/13/2016"
   wacn.date="06/20/2016" />

# Docker 错误故障排除

为应用程序的 Docker 容器配置所有设置后，应确保设置和路径是正确的。Visual Studio 在“发布”对话框提供了“验证”按钮，可帮助您执行此操作。

此主题可帮助您诊断并修复或解决在 Docker 中托管 Visual Studio 应用程序时遇到的最常见问题。更多问题在遇到时将添加到本主题。

## 当您尝试在“发布 Web”对话框中验证与 Docker 主机的连接时，获得失败消息

以下是此问题的一些可能解决方案。

- 在“发布”对话框的“连接”选项卡中，确保“服务器 URL”是正确的，并且“服务器 URL”中尾随的 `:<port_number>` 是 Docker 后台程序侦听的端口。

- 在“发布”对话框的“连接”选项卡中，展开“Docker 高级选项”部分并确保指定了正确的“身份验证”选项。
  - 如果服务器上的 Docker 后台程序配置为使用 TLS 安全，则 Windows Docker 命令行界面 (docker.exe) 在默认情况下将在 `<%userprofile%>\.docker` 文件夹下查找客户端密钥 (key.pem) 和证书 (cert.pem)。如果这些项不存在，则需要使用 OpenSSL 生成这些项。有关配置 Docker for TLS 的详细信息，请参阅[使用 HTTPS 保护 Docker 后台程序套接字](https://docs.docker.com/articles/https/)。

	确保 Docker 正确从 Windows 客户端向 Linux 服务器进行身份验证的一种方法是将“预览”文本框的内容复制到一个新的命令窗口并将 `<command>` 更改为“info”，如以下所示：

    ```
    // This example assumes the Docker daemon is configured to use the default port
    // of 2376 to listen for connections.docker.

    --tls -H tcp://contoso.cloudapp.net:2376 info
    ```

    作为将客户端证书和密钥文件复制到 .docker 文件夹的替代方法，可以通过添加以下参数来更改“身份验证”选项：

    ```
    --tls --tlscert=C:\mycert\cert.pem --tlskey=C:\mycert\key.pem
    ```
- 确保 Docker 主机上的 Docker 后台程序是 1.6 或更高版本。

## 在 Docker 文件夹中没有客户端证书的情况下使用您自己的证书时发生超时错误

如果你选择在 Visual Studio 中创建 Docker 主机时使用你自己的证书（即，清除“在 Microsoft Azure 上创建虚拟机”对话框中的“自动生成 Docker 证书”复选框），则需要将客户端证书和密钥文件（cert.pem 和 key.pem）复制到 Docker 文件夹 (`<%userprofile%>\.docker`)。否则，当您发布项目时，将在一个小时后收到超时错误，并且发布操作将失败。

## 需要 PowerShell 3.0 才能发布到 Docker 容器

如果您的操作系统是 Windows 7 或 Windows Server 2008，则需要安装 PowerShell 3.0，然后才能发布到 Docker 容器。PowerShell 3.0 包括在 [Windows Management Framework 3.0](https://www.microsoft.com/zh-cn/download/details.aspx?id=34595) 中。在安装它后需要重启系统。

作为备用解决方法，您可以升级到已装载 PowerShell 3.0 的 Windows 8.1 或 Windows 10。

## PowerShell 窗口不会自动关闭

在创建 VM 后，PowerShell 窗口有时不会自动关闭。关闭此窗口也会关闭 Visual Studio。由于该窗口不会影响任何 Visual Studio 或 Docker 工具的功能，请将其保持打开状态直到完成工作。

## 常见问题

问：支持哪些用于发布到 Linux Docker 容器的 Visual Studio 项目模板？

答：Visual Studio 当前支持“C# 控制台应用程序（程序包）”和“C# ASP.NET 5 Preview Web 模板”，包括：

- 空

- Web API

- 网站

问：如何从命令行使用 MSBUILD 将我的 ASP.NET 5 Web 或控制台项目发布到 Docker？

答：使用以下 MSBuild 命令：

    `msbuild <projectname.xproj> /p:deployOnBuild=true;publishProfile=<profilename>`

问：如何从命令行使用 PowerShell 将我的 ASP.NET 5 Web 或控制台项目发布到 Docker？

答：使用以下 PowerShell 命令：

```
.\contoso-Docker-publish.ps1 -packOutput $env:USERPROFILE\AppData\Local\Temp\PublishTemp -pubxmlFile .\contoso-Docker.pubxml
```

问：我有自己的安装了 Docker 的 Linux 服务器，如何在“Web 发布”对话框中指定此服务器？

答：请参阅[在 Docker 中托管网站](/documentation/articles/vs-azure-tools-docker-hosting-web-apps-in-docker)主题中的**提供自定义的 Docker 主机**部分。

问：我使用的是安装了 Docker 的自己的 Linux 服务器。为使用 TLS 配置身份验证，如何生成密钥和证书？

答：一种方法是在服务器上使用 OpenSSL 为 CA、服务器和客户端生成所需的证书和密钥。然后可以使用第三方软件来建立 SSH/SFTP 连接，然后再将证书复制到本地的 Windows 开发计算机中。默认情况下，Docker (CLI) 将尝试使用位于 `<userprofile>\.docker` 文件夹的证书。

另一个选项是下载 OpenSSL for Windows 并生成所需的证书和密钥，并将 CA、服务器证书和密钥上载到 Linux 计算机上。有关建立与 Docker 的安全连接的详细信息，请参阅[使用 HTTPS 保护 Docker 后台程序套接字](https://docs.docker.com/articles/https/)。

<!---HONumber=Mooncake_0509_2016-->