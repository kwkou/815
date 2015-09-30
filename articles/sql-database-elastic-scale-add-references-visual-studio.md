<properties 
	pageTitle="将 Azure SQL DB 灵活扩展引用添加到 Visual Studio 项目" 
	description="如何使用 Nuget 将灵活扩展 API 的 .NET 引用添加到 Visual Studio 项目。" 
	services="sql-database" 
	documentationCenter="" 
	manager="jeffreyg" 
	authors="sidneyh" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.date="07/24/2015" 
	wacn.date="09/15/2015"/>

# 如何：添加对 Visual Studio 项目的弹性数据库客户端库引用 

###先决条件： 

- 为 Visual Studio 安装 [NuGet Visual Studio 扩展库](http://docs.nuget.org/docs/start-here/installing-nuget) 

### 在 Visual Studio 中添加弹性数据库客户端库引用 

1. 打开现有项目，或者使用“文件”-->“新建”-->“项目”中的“新建项目”对话框创建新的项目 
2. 在“解决方案资源管理器”中，右键单击“引用”，然后选择“管理 NuGet 包”
3. 在“管理 NuGet 程序包”窗口左侧的菜单中，依次选择“联机”、“nuget.org”或“全部” 
4. 在“联机搜索”对话框中，键入“弹性数据库”，选择“弹性数据库客户端库”，然后单击“安装”。

	![联机搜索][1]
4. 查看许可证，然后单击“我接受”。 

现在，客户端库引用已添加到你的项目。

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-scale-add-references-visual-studio/search-online.png
<!--anchors-->

<!---HONumber=69-->