<properties
   pageTitle="使用 Visual Studio 对 Windows 排查 Docker 客户端错误 | Azure"
   description="在 Windows 上通过使用 Visual Studio 排查在使用 Visual Studio 创建网站并将其部署到 Docker 时遇到的问题。"
   services="visual-studio-online"
   documentationCenter="na"
   authors="AllenClark"
   manager="douge"
   editor="" />
<tags
   ms.service="multiple"
   ms.date="05/13/2016"
   wacn.date="06/20/2016" />

# Visual Studio Docker 开发故障排除

## 概述
在使用 Visual Studio Tools for Docker 预览版时，可能会因预览特性而遇到一些问题。以下是一些常见问题和解决方法。

##故障排除 

- **运行应用会导致 PowerShell 打开、显示错误，然后关闭。浏览器页面未打开。**

	这可能是由于执行 `docker-compose-up` 期间出错。如要查看该错误，请执行以下步骤：

	1. 打开 `Properties\launchSettings.json` 文件
	
	1. 找到 Docker 条目。
	
	1. 找出开头为以下内容的行：

			"commandLineArgs": "-ExecutionPolicy RemoteSigned ¡­¡±
	
	1. 添加 `-noexit` 参数，使该行类似于以下内容。这将使 PowerShell 保持打开，方便你查看错误。

			"commandLineArgs": "-noexit -ExecutionPolicy RemoteSigned ¡­¡±

- **生成: 无法生成映像，检查 TLS 连接时出错: 主机未运行**

	验证默认 Docker 主机是否正在运行。请参阅 [Configure the Docker client（配置 Docker 客户端）](/documentation/articles/vs-azure-tools-docker-setup)一文。

- **找不到卷映射**

	默认情况下，VirtualBox 会将 `C:\Users` 共享为 `c:/Users`。如果项目不在 `c:\Users` 下，请将项目手动添加到 VirtualBox 的[共享文件夹](https://www.virtualbox.org/manual/ch04.html#sharedfolders)。

- **使用 Microsoft Edge 作为默认浏览器**

	如果你使用 Microsoft Edge 浏览器，站点可能不会打开，因为 Edge 认为该 IP 地址不安全。如要解决此问题，请执行以下步骤：
	1. 在 Windows 的“运行”框中，键入 `Internet Options`。
	2. 出现“Internet 选项”时，点击该选项。 
	2. 点击“安全性”选项卡。
	3. 选择“本地 Intranet”区域。
	4. 点击“站点”。 
	5. 将虚拟机（在本例中为 Docker 主机）的 IP 地址添加到列表中。 
	6. 在 Edge 中刷新网页，然后你应会看到站点已正常运行。 
	7. 有关此问题的详细信息，请访问 Scott Hanselman 的博客文章：[Microsoft Edge can't see or open VirtualBox-hosted local web sites（在 Microsoft Edge 中无法查看或打开 VirtualBox 托管的本地网站）](http://www.hanselman.com/blog/FixedMicrosoftEdgeCantSeeOrOpenVirtualBoxhostedLocalWebSites.aspx)。 

<!---HONumber=Mooncake_0620_2016-->