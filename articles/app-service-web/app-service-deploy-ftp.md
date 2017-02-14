<properties
    pageTitle="使用 FTP/S 将应用部署到 Azure 应用服务 | Azure"
    description="了解如何使用 FTP 或 FTPS 将应用部署到 Azure 应用服务。"
    services="app-service"
    documentationcenter=""
    author="cephalin"
    manager="erikre"
    editor="" />
<tags
    ms.assetid="ae78b410-1bc0-4d72-8fc4-ac69801247ae"
    ms.service="app-service"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/05/2016"
    wacn.date="02/10/2017"
    ms.author="cephalin;dariac" />  


# 使用 FTP/S 将应用部署到 Azure 应用服务
本文演示如何使用 FTP 或 FTPS 将 Web 应用、移动应用后端或 API 应用部署到 [Azure 应用服务](/documentation/articles/app-service-changes-existing-services/)。

应用的 FTP/S 终结点已处于活动状态。启用 FTP/S 部署不需要进行任何配置。

## <a name="step1"></a>步骤 1：设置部署凭据

若要访问应用的 FTP 服务器，首先需要部署凭据。

若要设置或重置部署凭据，请参阅 [Azure 应用服务部署凭据](/documentation/articles/app-service-deployment-credentials/)。本教程演示如何使用用户级凭据。

## 步骤 2：获取 FTP 连接信息

1. 在 [Azure 门户预览](https://portal.azure.cn)中，打开应用的[资源边栏选项卡](/documentation/articles/resource-group-portal/#manage-resources)。
2. 选择左侧菜单中的“概述”，然后记下 **FTP/部署用户**、**FTP 主机名**和 **FTPS 主机名**的值。

    ![FTP 连接信息](./media/web-sites-deploy/FTP-Connection-Info.PNG)  

    > [AZURE.NOTE]
    必须输入 Azure 门户预览中显示的“FTP/部署用户”值（包括应用名称），以便为 FTP 服务器提供适当的上下文。在左侧菜单中选择“属性”时，可以找到相同信息。
    ><p>
    > 此外，永远不会显示部署密码。如果忘记了部署密码，请返回到[步骤 1](#step1)，重置部署密码。
    >
    >

## 步骤 3：将文件部署到 Azure

1. 从 FTP 客户端（[Visual Studio](https://www.visualstudio.com/vs/community/)、[FileZilla](https://filezilla-project.org/download.php?type=client) 等），使用收集到的连接信息连接到你的应用。
3. 将你的文件及其各自的目录结构复制到 Azure 中的 [**/site/wwwroot** 目录](https://github.com/projectkudu/kudu/wiki/File-structure-on-azure)（或者将 Web 作业复制到 **/site/wwwroot/App\_Data/Jobs/** 目录）。
4. 浏览到你的应用的 URL，以验证该应用是否正在正常运行。

> [AZURE.NOTE] 
与[基于 Git 的部署](/documentation/articles/app-service-deploy-local-git/)不同，FTP 部署不支持以下部署自动化：
><p>
><p> - 依赖项还原（如 NuGet、NPM、PIP 和 Composer 自动化）
><p> - 编译 .NET 二进制文件 
><p> - 生成 web.config（下面是 [Node.js 示例](https://github.com/projectkudu/kudu/wiki/Using-a-custom-web.config-for-Node-apps)）
><p> 
> 必须在本地计算机上手动还原、构建和生成这些必需的文件并将其与你的应用一起部署。
>
>

## 后续步骤

有关更高级的部署方案，请尝试[使用 Git 部署到 Azure](/documentation/articles/app-service-deploy-local-git/)。通过基于 Git 的 Azure 部署可实现版本控制、包还原、MSBuild 等功能。

## 更多资源

* [创建 PHP-MySQL Web 应用并使用 FTP 进行部署](/documentation/articles/web-sites-php-mysql-deploy-use-ftp/)。
* [Azure App Service 部署凭据](/documentation/articles/app-service-deployment-credentials/)

<!---HONumber=Mooncake_0206_2017-->