<properties
   pageTitle="在本地 Docker 容器中调试应用 | Azure"
   description="了解如何通过编辑和刷新以及设置调试断点功能来修改本地 Docker 容器中运行的应用以及刷新容器"
   services="azure-container-service"
   documentationCenter="na"
   authors="allclark"
   manager="douge"
   editor="" />
<tags
   ms.service="multiple"
   ms.date="07/22/2016"
   wacn.date="09/19/2016" />

# 在本地 Docker 容器中调试应用

## 概述
Visual Studio Tools for Docker 提供了一致的方法在本地 Linux Docker 容器中开发和验证应用程序。每次进行代码更改后，不需要重新启动该容器。本文将演示如何使用“编辑和刷新”功能在本地 Docker 容器中启动 ASP.NET Core Web 应用，进行任何必要的更改，然后刷新浏览器以查看这些更改。它还将说明如何为调试设置断点。

> [AZURE.NOTE] Windows 容器支持会在将来的版本中推出

## 先决条件
需要安装以下工具。

- [Visual Studio 2015 Update 2](https://go.microsoft.com/fwlink/?LinkId=691978)
- [Microsoft ASP .NET Core RC 2](http://go.microsoft.com/fwlink/?LinkId=798481)
- [Visual Studio 2015 Tools for Docker](https://aka.ms/DockerToolsForVS)

若要在本地运行 Docker 容器，需要本地 docker 客户端。你可以使用发布的 [Docker 工具箱](https://www.docker.com/products/overview#/docker_toolbox)（需要禁用 Hyper-V），也可以使用 [Docker for Windows Beta 版](https://beta.docker.com)（它使用 Hyper-V，并需要 Windows 10）。

如果使用 Docker 工具箱，则需要[配置 Docker 客户端](/documentation/articles/vs-azure-tools-docker-setup/)

## 1\.创建 Web 应用

[AZURE.INCLUDE [create-aspnet5-app](../../includes/create-aspnet5-app.md)]

## 2\.添加 Docker 支持

[AZURE.INCLUDE [添加 Docker 支持](../../includes/vs-azure-tools-docker-add-docker-support.md)]


## 3\.编辑代码并刷新

若要快速重复更改，可以在容器中启动应用程序，并继续进行更改，然后就像使用 IIS Express 一样查看这些更改。

1. 将解决方案配置设置为 `Debug`，并按 **&lt;CTRL + F5>** 以生成 docker 映像并在本地运行它。

    容器映像已生成并在 Docker 容器中运行后，Visual Studio 将在默认浏览器中启动 Web 应用。
    如果使用的是 Microsoft Edge 浏览器或以其他方式出现错误，请参阅[故障排除](vs-azure-tools-docker-troubleshooting-docker-errors.md)部分。

1. 请转到“关于”页，我们将在此页中进行更改。

1. 返回到 Visual Studio 并打开 `Views\Home\About.cshtml`。

1. 将以下 HTML 内容添加到文件末尾，并保存更改。

	    <h1>Hello from a Docker Container!</h1>

1. 查看输出窗口，当 .NET 生成完成并且你看到这些行时，切换回浏览器并刷新“关于”页。


	    Now listening on: http://*:80
	    Application started. Press Ctrl+C to shut down

1. 你的更改已应用！

## 4\.使用断点进行调试

通常，更改将需要利用 Visual Studio 的调试功能进行进一步检查。

1.	返回到 Visual Studio 并打开 `Controllers\HomeController.cs`

1.  将 About() 方法的内容替换为以下代码：

    	string message = "Your application description page from wthin a Container";
    	ViewData["Message"] = message;

1.  在 `string message`.行的左侧设置一个断点。

1.  按 **&lt;F5>** 开始调试。

1.  导航到“关于”页以命中断点。

1.  切换到 Visual Studio 以查看断点，并检查消息的值。

	![][2]

##摘要

使用 [Visual Studio 2015 Tools for Docker](https://aka.ms/DockerToolsForVS)，可以通过在 Docker 容器内开发的生产真实性，获得在本地工作的生产效率。

## 故障排除

[Visual Studio Docker 开发故障排除](/documentation/articles/vs-azure-tools-docker-troubleshooting-docker-errors/)

## 提供有关在 Visual Studio、Windows 和 Azure 中使用 Docker 的更多信息

- [Docker Tools for Visual Studio](http://aka.ms/dockertoolsforvs) - 在容器中开发 .NET Core 代码
- [Docker Tools for Visual Studio Team Services](http://aka.ms/dockertoolsforvsts) - 生成和部署 docker 容器
- [Docker Tools for Visual Studio Code](http://aka.ms/dockertoolsforvscode) - 用于编辑 docker 文件的语言服务，随后将推出更多 e2e 方案
- [Windows 容器信息](http://aka.ms/containers) - Windows Server 和 Nano Server 信息
- [Azure 容器服务](https://azure.microsoft.com/services/container-service/) - [Azure 容器服务内容](http://aka.ms/AzureContainerService)

## 各种 Docker 工具

[一些好用的 docker 工具（Steve Lasker 的博客）](https://blogs.msdn.microsoft.com/stevelasker/2016/03/25/some-great-docker-tools/)

## 优秀文章

[NGINX 中的微服务简介](https://www.nginx.com/blog/introduction-to-microservices/)

## 演示

- [Steve Lasker：VS Live Las Vegas 2016 - Docker e2e](https://github.com/SteveLasker/Presentations/blob/master/VSLive2016/Vegas/)
- [ASP.NET Core @ build 2016 简介 - 其中你在演示中](https://channel9.msdn.com/Events/Build/2016/B810)
- [在容器中开发 .NET 应用，第 9 频道](https://blogs.msdn.microsoft.com/stevelasker/2016/02/19/developing-asp-net-apps-in-docker-containers/)

[2]: ./media/vs-azure-tools-docker-edit-and-refresh/breakpoint.png

<!---HONumber=Mooncake_0912_2016-->