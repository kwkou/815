<properties
   pageTitle="使用 Visual Studio 创建 Azure 项目 | Azure"
   description="使用 Visual Studio 创建 Azure 项目"
   services="visual-studio-online"
   documentationCenter="na"
   authors="TomArcher"
   manager="douge"
   editor="" />
<tags
   ms.service="multiple"
   ms.date="04/19/2016"
   wacn.date="05/23/2016" />

# 使用 Visual Studio 创建 Azure 项目

Azure Tools for Visual Studio 提供可用于创建 Azure 云服务的模板。这些工具还可帮助你配置、调试云服务并将其部署到 Azure。

Azure 云服务解决方案包含以下类型的项目：

- **Azure 项目**

    Azure 项目与解决方案中的角色项目具有关联。它还包括服务定义和服务配置文件。服务定义文件定义了应用程序的运行时设置，包括所需角色、终结点和虚拟机大小。服务配置文件配置了角色的运行中实例数以及为角色定义的设置值。

- **Web 角色项目**

    辅助角色执行后台处理。辅助角色可与存储服务和其他基于 Internet 的服务通信。一个辅助角色可以有任意数量的 HTTP、HTTPS 或 TCP 终结点。

    - **ASP.NET Web 角色**，用于生成具有 Web 前端的 ASP.NET 应用程序
    - **ASP.NET MVC5 Web 角色**
    - **ASP.NET MVC4 Web 角色**
    - **ASP.NET MVC3 Web 角色**
    - **WCF 服务 Web 角色**，用于生成 WCF 服务
    - **Silverlight 业务应用程序 Web 角色**（需要 Visual Studio 2012）

- **缓存辅助角色**

    为应用程序提供专用缓存的角色。

- **具有服务总线队列的辅助角色**

    提供消息队列功能以便与辅助进程通信的服务总线队列。有关详细信息，请参阅[如何使用服务总线队列](/documentation/articles/service-bus-dotnet-how-to-use-queues/)。

## 在 Visual Studio 中创建 Azure 云服务项目

1. 以管理员身份启动 Microsoft Visual Studio。

1. 在菜单栏上，选择“文件”、“新建”、“项目”。

1. 在“项目类型”窗格中，从 Visual C# 或 Visual Basic 项目模板节点选择“云”。

1. 在“模板”窗格中，选择“Azure 云服务”。

1. 指定要用于开发项目的 .NET Framework 版本。

1. 输入项目的名称和位置以及解决方案的名称。选择“确定”按钮。

1. 在“新建 Azure 项目”对话框中，选择要添加的角色，然后选择右箭头按钮以将其添加到解决方案。你可以根据需要添加任意多个角色。

1. 若要重命名已添加到项目的角色，请在“新建 Azure 项目”对话框中将鼠标悬停在该角色上，然后选择该角色右侧的“重命名”图标。还可在添加角色后在解决方案内对其进行重命名。

<!---HONumber=Mooncake_0516_2016-->