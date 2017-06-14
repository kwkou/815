<properties
    pageTitle="连续部署到 Azure App Service | Azure"
    description="了解如何实现连续部署到 Azure App Service。"
    services="app-service"
    documentationcenter=""
    author="dariagrigoriu"
    manager="erikre"
    editor="mollybos"
    translationtype="Human Translation" />
<tags
    ms.assetid="6adb5c84-6cf3-424e-a336-c554f23b4000"
    ms.service="app-service"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/28/2016"
    wacn.date="04/24/2017"
    ms.author="dariagrigoriu"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="1f53d356062251966107053a9aab341cbf0a5e4c"
    ms.lasthandoff="04/14/2017" />

# <a name="continuous-deployment-to-azure-app-service"></a>连续部署到 Azure App Service

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

本教程说明如何为 [Azure 应用服务]应用配置连续部署工作流。 将应用服务与 BitBucket 和 GitHub 集成可以实现一种连续部署工作流，在其中，Azure 可以从发布到其中一种服务的项目中拉取最近的更新。 连续部署选项非常适合用于频繁集成多个分发内容的项目。

若要了解如何通过 Azure 门户中未列出的云存储库（如 [GitLab](https://gitlab.com/)）手动配置连续部署，请参阅 [Setting up continuous deployment using manual steps](https://github.com/projectkudu/kudu/wiki/Continuous-deployment#setting-up-continuous-deployment-using-manual-steps)（使用手动步骤设置连续部署）。

## <a name="overview"></a>启用连续部署
若要启用连续部署，请执行以下操作：

1. 将应用内容发布到用于连续部署的存储库。  

    若要深入了解如何将项目发布到这些服务，请参阅[创建存储库 (GitHub)]。 以下步骤仅适用于 GitHub 存储库。 有关 GitHub 以外的存储库，请参阅 [手动触发部署](https://github.com/projectkudu/kudu/wiki/Manually-triggering-a-deployment) 以设置 Webhook。

2. 访问 [Azure 门户]，在你的应用的菜单边栏选项卡中单击“应用部署”>“部署选项”。 单击“选择源”，然后选择“外部存储库”。
2. 输入 GitHub 存储库的 URL 并输入要跟踪的分支。 将“存储库类型”保留为“Git”，并单击“确定”

    > [AZURE.NOTE]
    > Azure 中国目前只支持公共存储库，因为我们无法输入 Web 应用的 Git 凭据。

    Azure 将创建与所选存储库的关联，从指定的分支提取文件，并保留 Azure Web 应用存储库的副本。
3. 在应用的“概览”边栏选项卡中，单击“获取发布配置文件”以下载 Web 应用的发布配置文件。
4. 使用文本编辑器打开发布配置文件，并获取以下值。

        userName="$<YOUR_WEB_APP>" userPWD="<SOME_BIG_RANDOM_TOKEN>"

5. 在 GitHub 存储库页面上，单击“设置”，选择“Webhook 与服务”，然后单击“添加 Webhook”
6. 在“有效负载 URL”中，输入 `https://$<YOUR_WEB_APP>:<SOME_BIG_RANDOM_TOKEN>@<YOUR_WEB_APP>.scm.chinacloudsites.cn/deploy`，按原样保留所有其他项目，然后单击“添加 Webhook”以保存新的 Webhook。
5. 若要验证是否已成功部署应用，请在 Azure 门户的应用边栏选项卡顶部单击“URL”。
6. 若要验证是否能够在从所选存储库进行连续部署，请将更改推送到该存储库。 推送到存储库完成后，你的应用应该很快更新以反映更改。 可在应用的“部署选项”边栏选项卡中验证是否已拉取更新。

## <a name="VSsolution"></a>连续部署 Visual Studio 解决方案

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-visual-studio-login-guide.md)]

将 Visual Studio 解决方案推送到 Azure App Service 就像推送简单的 index.html 文件一样容易。 应用服务部署过程简化了所有细节，包括还原 NuGet 依赖项和生成应用程序二进制文件。 可以按照仅在 Git 存储库中维护代码的源控件最佳实践操作，并让应用服务部署处理其余工作。

将 Visual Studio 解决方案推送到应用服务的步骤与[上一部分](#overview)中的步骤相同，前提是按以下方式配置解决方案和存储库：

* 使用 Visual Studio 源代码管理选项生成如下图所示的 `.gitignore` 文件，或者在内容存储库根目录中手动添加一个 `.gitignore` 文件，其内容类似于此 [.gitignore 示例](https://github.com/github/gitignore/blob/master/VisualStudio.gitignore)。

    ![](./media/app-service-continuous-deployment/VS_source_control.png)
* 将整个解决方案的目录树添加到你的存储库中，其中 .sln 文件位于存储库根中。

你按照说明设置存储库并将 Azure 中的应用配置为从某个联机 Git 存储库连续发布后，你就可以在 Visual Studio 中从本地开发 ASP.NET 应用程序，并且只需通过将所做的更改推送到联机的 Git 存储库即可连续部署代码。

## <a name="disableCD"></a>禁用连续部署
若要禁用连续部署，请执行以下操作：

1. 访问 [Azure 门户]，在你的应用的菜单边栏选项卡中单击“应用部署”>“部署选项”。 然后单击“部署选项”边栏选项卡中的“断开连接”。

    ![](./media/app-service-continuous-deployment/cd_disconnect.png)
2. 在显示确认消息时点击“是”后，若要从其他源设置发布，可返回到应用边栏选项卡，再单击“应用部署”>“部署选项”。

## <a name="additional-resources"></a>其他资源
* [How to investigate common issues with continuous deployment（如何调查连续部署的常见问题）](https://github.com/projectkudu/kudu/wiki/Investigating-continuous-deployment)
* [如何使用适用于 Azure 的 PowerShell]
* [如何使用针对 Mac 和 Linux 的 Azure 命令行工具]
* [Git 文档]
* [项目 Kudu](https://github.com/projectkudu/kudu/wiki)
* [Use Azure to automatically generate a CI/CD pipeline to deploy an ASP.NET 4 app](https://www.visualstudio.com/docs/build/get-started/aspnet-4-ci-cd-azure-automatic)（使用 Azure 自动生成部署 ASP.NET 4 应用的 CI/CD 管道）

[Azure 应用服务]: /documentation/articles/app-service-changes-existing-services/
[Azure 门户]: https://portal.azure.cn
[VSTS Portal]: https://www.visualstudio.com/products/visual-studio-team-services-vs.aspx
[Installing Git]: http://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git
[如何使用适用于 Azure 的 PowerShell]: https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs
[如何使用针对 Mac 和 Linux 的 Azure 命令行工具]: /documentation/articles/cli-install-nodejs/
[Git 文档]: http://git-scm.com/documentation

[创建存储库 (GitHub)]: https://help.github.com/articles/create-a-repo
[创建存储库 (BitBucket)]: https://confluence.atlassian.com/display/BITBUCKET/Create+an+Account+and+a+Git+Repo
[VSTS 入门]: https://www.visualstudio.com/docs/vsts-tfs-overview
[Continuous delivery to Azure using Visual Studio Team Services]: /documentation/articles/cloud-services-continuous-delivery-use-vso/
<!--Update_Description: change steps to Azure portal preview-->