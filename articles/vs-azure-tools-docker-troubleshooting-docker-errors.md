<properties
   pageTitle="使用 Visual Studio 对 Windows 排查 Docker 客户端错误 | Azure"
   description="在 Windows 上通过使用 Visual Studio 排查在使用 Visual Studio 创建 Web 应用并将其部署到 Docker 时遇到的问题。"
   services="azure-container-service"
   documentationCenter="na"
   authors="allclark"
   manager="douge"
   editor="" />
<tags
   ms.service="multiple"
   ms.date="06/08/2016"
   wacn.date="06/20/2016" />

# Visual Studio Docker 开发故障排除

在使用 Visual Studio Tools for Docker 预览版时，可能会因预览特性而遇到一些问题。以下是一些常见问题和解决方法。

##无法配置 Program.cs for Docker 支持

添加 docker 支持时，必须将 `.UseUrls(Environment.GetEnvironmentVariable("ASPNETCORE_SERVER.URLS"))` 添加到 WebHostBuilder()。如果在 Program.cs 中找不到 `Main()` 函数或新的 WebHostBuilder 类，则将显示一条警告。在 docker 容器内运行时，除了 localhost 外，`.UseUrls()` 是允许 Kestrel 侦听传入流量所必需的。完成后，典型代码将如下所示：


	public class Program
	{
	    public static void Main(string[] args)
	    {
	        var host = new WebHostBuilder()
	            .UseUrls(Environment.GetEnvironmentVariable("ASPNETCORE_URLS") ?? String.Empty)
	            .UseKestrel()
	            .UseContentRoot(Directory.GetCurrentDirectory() ?? "")
	            .UseIISIntegration()
	            .UseStartup<Startup>()
	            .Build();

	        host.Run();
	    }
	}

UseUrls() 将 WebHost 配置为侦听传入 URL 流量。[Docker Tools for Visual Studio](http://aka.ms/DockerToolsForVS) 将在 dockerfile.debug/release 模式下配置环境变量，如下所示：


	# Configure the listening port to 80
	ENV ASPNETCORE_SERVER.URLS http://*:80


## 卷映射不正常工作
为启用编辑和刷新功能，卷映射配置为将项目的源代码共享到容器内的 .app 文件夹。在主机上更改了文件时，容器 /app 目录会使用相同的目录。在 docker-compose.debug.yml 中，以下配置启用卷映射


	volumes:
	    - ..:/app


若要测试卷映射是否正常工作，请尝试以下命令：

**通过 Windows**

	a
	/ # ls


你应看到 Users/Public 文件夹中的目录列表。如果未显示任何文件，并且 /c/Users/Public 文件夹不为空，则卷映射未正确配置。

```
bin       etc       proc      sys       usr       wormhole
dev       home      root      tmp       var
```

转到旋涡式星体目录，以查看 `/c/Users/Public` 目录的内容：

	/ # cd wormhole/
	/wormhole # ls
	AccountPictures  Downloads        Music            Videos
	Desktop          Host             NuGet.Config     a.txt
	Documents        Libraries        Pictures         desktop.ini
	/wormhole #


**注意：** 使用 Linux VM 时，容器文件系统区分大小写。

如果无法看到内容，请尝试以下操作：

**Docker for Windows beta 版**
- 通过查看系统任务栏中的魔比图标，并确保它为白色且正常工作来验证 Docker for Windows 桌面应用是否正常运行。
- 通过右键单击系统任务栏中的魔比图标，选择设置并单击“管理共享驱动器...”来验证是否已配置卷映射

**Docker 工具箱 w/VirtualBox**

默认情况下，VirtualBox 会将 `C:\Users` 共享为 `c:/Users`。如果可能，将你的项目移到此目录下。否则，可将项目手动添加到 VirtualBox 的[共享文件夹](https://www.virtualbox.org/manual/ch04.html#sharedfolders)。
	
##生成: 无法生成映像，检查 TLS 连接时出错: 主机未运行

- 验证默认 Docker 主机是否正在运行。请参阅 [Configure the Docker client](/documentation/articles/vs-azure-tools-docker-setup/)（配置 Docker 客户端）一文。

##使用 Microsoft Edge 作为默认浏览器

如果你使用 Microsoft Edge 浏览器，站点可能不会打开，因为 Edge 认为该 IP 地址不安全。如要解决此问题，请执行以下步骤：

1. 转到“Internet 选项”。
    - 在 Windows 10 中，可以在 Windows“运行”框中键入 `Internet Options`。
    - 在 Internet Explorer 中，可以转到“设置”菜单，然后选择“Internet 选项”。
1. 出现“Internet 选项”时，选择该对话框。
1. 选择“安全性”选项卡。
1. 选择“本地 Intranet”区域。
1. 选择“站点”。
1. 将虚拟机（在本例中为 Docker 主机）的 IP 地址添加到列表中。
1. 在 Edge 中刷新网页，然后你应会看到站点已正常运行。
1. 有关此问题的详细信息，请访问 Scott Hanselman 的博客文章：[Microsoft Edge can't see or open VirtualBox-hosted local web sites](http://www.hanselman.com/blog/FixedMicrosoftEdgeCantSeeOrOpenVirtualBoxhostedLocalWebSites.aspx)（Microsoft Edge 无法查看或打开 VirtualBox 托管的本地网站）。

##版本 0.15 或更早版本故障排除


###运行应用会导致 PowerShell 打开、显示错误，然后关闭。浏览器页面未打开。

这可能是由于执行 `docker-compose-up` 期间出错。如要查看该错误，请执行以下步骤：

1. 打开 `Properties\launchSettings.json` 文件
1. 找到 Docker 条目。
1. 找出开头为以下内容的行：

    
    	"commandLineArgs": "-ExecutionPolicy RemoteSigned …”
    
	
1. 添加 `-noexit` 参数，使该行类似于以下内容。这将使 PowerShell 保持打开，方便你查看错误。


	    "commandLineArgs": "-noexit -ExecutionPolicy RemoteSigned …”
 

<!---HONumber=AcomDC_0718_2016-->