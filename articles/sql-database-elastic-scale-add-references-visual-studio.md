<properties title="Add Azure SQL DB Elastic Scale References to a Visual Studio Project" pageTitle="将 Azure SQL DB 灵活扩展引用添加到 Visual Studio 项目" description="如何使用 Nuget 将灵活扩展 API 的 .NET 引用添加到 Visual Studio 项目。" metaKeywords="Azure SQL Database, elastic scale, Nuget references" services="sql-database" documentationCenter="" manager="jhubbard" authors="sidneyh@microsoft.com"/>

#如何：将 Azure SQL DB 灵活扩展引用添加到 Visual Studio 项目 

###先决条件： 

- 为 Visual Studio 安装 [NuGet Visual Studio 扩展库](http://docs.nuget.org/docs/start-here/installing-nuget) 

###将灵活扩展引用添加到 Visual Studio 中 

1. 通过位于"文件"->"新建"-->"项目"中的"新建项目"对话框创建新的项目************ 
2. 在解决方案资源管理器中，右键单击"引用"，然后选择"管理 NuGet 程序包"********
3. 在"管理 NuGet 程序包"窗口左侧的菜单中，依次选择"联机"、"nuget.org"或"全部"******** 
4. 在"联机搜索"对话框中，键入"弹性缩放"、选择"弹性缩放客户端库"，然后单击"安装"****************。
4. 查看许可协议，然后单击"我接受"****。 

Azure SQL Database 灵活扩展引用现已添加到您的项目。 

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]
