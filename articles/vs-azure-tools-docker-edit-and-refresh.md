<properties
   pageTitle="在本地 Docker 容器中调试应用 | Azure"
   description="了解如何通过编辑和刷新以及设置调试断点功能来修改本地 Docker 容器中运行的应用以及刷新容器"
   services="visual-studio-online"
   documentationCenter="na"
   authors="AllenClark"
   manager="douge"
   editor="" />
<tags
   ms.service="multiple"
   ms.date="03/25/2016"
   wacn.date="06/27/2016" />

# 在本地 Docker 容器中调试应用

## 概述
Visual Studio Tools for Docker 提供了一致的方法在本地 Linux Docker 容器中开发和验证应用程序。每次进行代码更改后，不需要重新启动该容器。本文将演示如何使用“编辑和刷新”功能在本地 Docker 容器中启动 ASP.NET Core Web 应用，进行任何必要的更改，然后刷新浏览器以查看这些更改。它还将说明如何为调试设置断点。

> [AZURE.NOTE] Windows 容器支持会在将来的版本中推出

## 先决条件
需要安装以下工具。

- [Visual Studio 2015 Update 2](https://go.microsoft.com/fwlink/?LinkId=691978)
- [Microsoft ASP .NET Core RC 2](http://go.microsoft.com/fwlink/?LinkId=798481)
- [Visual Studio 2015 Tools for Docker](https://aka.ms/DockerToolsForVS)

若要在本地运行 Docker 容器，需要本地 docker 客户端。你可以使用发布的 [Docker 工具箱](https://www.docker.com/products/overview#/docker_toolbox)（需要禁用 Hyper-V），也可以选择使用 [Docker for Windows Beta 版](https://beta.docker.com)（它使用 Hyper-V，并需要 Windows 10）。

如果使用 Docker 工具箱，则需要[配置 Docker 客户端](/documentation/articles/vs-azure-tools-docker-setup)

## 编辑本地 Docker 容器中运行的应用
Visual Studio 2015 Tools for Docker 可让 ASP .NET Core RC2 Web 应用开发人员在 Docker 容器中测试和运行应用程序、在 Visual Studio 中更改应用程序，以及刷新浏览器来查看已应用到容器中运行的应用的更改。使用 .NET Core 和 Visual Studio Tools for Docker 版本 0.20，还可以为使用 Docker 容器运行的代码设置断点。

1. 从 Visual Studio 菜单中，选择“文件”>“新建”>“项目”。

1. 在“新建项目”对话框的“模板”部分下，选择“Visual C#”>“Web”。

1. 选择“ASP.NET Core Web 应用程序(.NET Core)”。

1. 为新应用程序指定名称（或使用默认值），然后点击“确定”。

1. 在“ASP.NET Core 模板”下，选择“Web 应用程序”，然后点击“确定”。

1. 取消选中“在云中托管”，因为你将使用 Docker 作为部署解决方案。

1. 在 Visual Studio 的“解决方案资源管理器”中，右键单击项目，然后选择“添加”>“Docker 支持”。

	![][0]

1. 随即会在项目节点下面创建以下文件：

	![][1]

> [AZURE.NOTE] 如果使用 [Docker for Windows Beta 版](https://beta.docker.com)，请打开 Properties\\Docker.props，删除默认值并重新启动 Visaul Studio 使值生效。

##编辑和刷新
若要快速重复更改，可以在容器中启动应用程序，并继续进行更改，然后就像使用 IIS Express 一样查看这些更改。

1. 将解决方案配置设置为 `Debug`，并按 **&lt;CTRL + F5>** 以生成 docker 映像并在本地运行它。使用内部版本查看输出窗口，或者

1. 容器映像已生成并在 Docker 容器中运行后，Visual Studio 将尝试在默认浏览器中启动 Web 应用。如果你使用的是 Microsoft Edge 浏览器或以其他方式出现错误，请参阅[故障排除](/documentation/articles/vs-azure-tools-docker-troubleshooting-docker-errors)部分。

1. 返回到 Visual Studio 并打开 `Views\Home\About.cshtml`。

1. 将以下 HTML 内容追加到文件末尾，并保存更改。

    	<h1>Hello from a Docker Container!</h1>

1.	查看输出窗口，当 .NET 生成完成并且你看到 `Application started. Press Ctrl+C to shut down` 时，切换回浏览器并刷新页面。

1.	你应看到更改已应用！

##断点调试
通常，更改将需要利用 Visual Studio 的调试功能进行进一步检查。

1.	返回到 Visual Studio 并打开 `Controllers\HomeController.cs`

1.  将 About() 方法的内容替换为以下代码：

    	string message = "Your application description page from wthin a Container";
    	ViewData["Message"] = message;

1.  在 `string message`.行的左侧设置一个断点。

1.  按 **&lt;F5>** 开始调试。

1.  导航到“关于”页以命中断点。

1.  切换到 Visual Studio 以查看断点，并检查消息的值。

	![][3]

##摘要
使用 [Visual Studio 2015 Tools for Docker](https://aka.ms/DockerToolsForVS)，可以通过在 Docker 容器内开发的生产真实性，获得在本地工作的生产效率。

## 故障排除
[Visual Studio Docker 开发故障排除](/documentation/articles/vs-azure-tools-docker-troubleshooting-docker-errors)

## 提供有关在 Visual Studio、Windows 和 Azure 中使用 Docker 的更多信息

- [Docker Tools for Visual Studio](http://aka.ms/dockertoolsforvs) - 在容器中开发 .NET Core 代码
- [Docker Tools for Visual Studio Team Services](http://aka.ms/dockertoolsforvsts) - 生成和部署 docker 容器
- [Docker Tools for Visual Studio Code](http://aka.ms/dockertoolsforvscode) - 用于编辑 docker 文件的语言服务，随后将推出更多 e2e 方案
- [Windows 容器信息](http://aka.ms/containers) - Windows Server 和 Nano Server 信息

## 各种 Docker 工具

[一些好用的 docker 工具（Steve Lasker 的博客）](https://blogs.msdn.microsoft.com/stevelasker/2016/03/25/some-great-docker-tools/)

## 优秀文章

[NGINX 中的微服务简介](https://www.nginx.com/blog/introduction-to-microservices/)

## 演示

- [Steve Lasker：VS Live Las Vegas 2016 - Docker e2e](https://github.com/SteveLasker/Presentations/blob/master/VSLive2016/Vegas/)
- [ASP.NET Core @ build 2016 简介 - 其中你在演示中](https://channel9.msdn.com/Events/Build/2016/B810)
- [在容器中开发 .NET 应用，第 9 频道](https://blogs.msdn.microsoft.com/stevelasker/2016/02/19/developing-asp-net-apps-in-docker-containers/)

[0]: ./media/vs-azure-tools-docker-edit-and-refresh/add-docker-support.png
[1]: ./media/vs-azure-tools-docker-edit-and-refresh/docker-files-added.png
[2]: ./media/vs-azure-tools-docker-edit-and-refresh/docker-props.png
[3]: ./media/vs-azure-tools-docker-edit-and-refresh/breakpoint.png
<!---HONumber=Mooncake_0620_2016-->