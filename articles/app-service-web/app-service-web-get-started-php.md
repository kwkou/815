<properties 
	pageTitle="在 5 分钟内将第一个 PHP Web 应用部署到 Azure | Azure" 
	description="了解如何部署示例应用，轻松地在应用服务中运行 Web 应用。快速进行实际的开发，立即查看结果。" 
	services="app-service\web"
	documentationCenter=""
	authors="cephalin"
	manager="wpickett"
	editor=""
/>  


<tags
	ms.service="app-service-web"
	ms.workload="web"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="hero-article"
	ms.date="10/13/2016"
	wacn.date="10/31/2016" 
	ms.author="cephalin"
/>  

	
# 在 5 分钟内将第一个 PHP Web 应用部署到 Azure

本教程旨在帮助用户将第一个 PHP Web 应用部署到 [Azure App Service](/documentation/articles/app-service-value-prop-what-is/)。应用服务可用于创建 Web 应用、[移动应用后端](/documentation/services/app-service/mobile/)和 [API 应用](/documentation/articles/app-service-api-apps-why-best-platform/)。

用户将能够：

- 在 Azure App Service 中创建 Web 应用。
- 部署示例 PHP 代码。
- 查看代码在生产环境中的实时运行。
- 以[推送 Git 提交](https://git-scm.com/docs/git-push)的相同方式来更新 Web 应用。

## <a name="Prerequisites"></a> 先决条件

- [Git](http://www.git-scm.com/downloads)。
- [Azure CLI](/documentation/articles/xplat-cli-install/)。
- 一个 Azure 帐户。如果你没有帐户，可以[注册试用版](/pricing/1rmb-trial/?WT.mc_id=A261C142F)。

## 部署 PHP Web 应用

1. 打开新的 Windows 命令提示符、PowerShell 窗口、Linux shell 或 OS X 终端。运行 `git --version` 和 `azure --version`，验证计算机上是否已安装 Git 和 Azure CLI。

    ![在 Azure 中测试第一个 Web 应用的 CLI 工具安装](./media/app-service-web-get-started/1-test-tools.png)

    如果尚未安装这些工具，请参阅[先决条件](#Prerequisites)中的下载链接。

3. 如下所示登录 Azure ：

        azure login -e AzureChinaCloud

    按照帮助消息的提示继续此登录过程。

    ![登录到 Azure 以创建第一个 Web 应用](./media/app-service-web-get-started/3-azure-login.png)  


4. 将 Azure CLI 更改为 ASM 模式，然后设置应用服务的部署用户。稍后使用凭据部署代码。

        azure config mode asm
        azure site deployment user set --username <username> --pass <password>

1. 切换到工作目录 (`CD`) 并克隆示例应用，如下所示：

        git clone https://github.com/Azure-Samples/app-service-web-php-get-started.git

2. 更改为示例应用的存储库。例如：

        cd app-service-web-php-get-started

4. 在 Azure 中创建具有唯一应用名称及之前配置的部署用户的应用服务应用资源。出现提示时，指定所需的区域数目。

        azure site create <app_name> --git --gitusername <username>

    ![在 Azure 中创建第一个 Web 应用的 Azure 资源](./media/app-service-web-get-started-languages/php-site-create.png)  


    现在已在 Azure 中创建应用。而且当前的目录也已进行 Git 初始化并作为 Git 远程连接到新的 App Service 应用。可以浏览到应用的 URL (http://&lt;app_name>.chinacloudsites.cn) 查看优雅的默认 HTML 页面，不过现在要做的是真正获取自己的代码。

4. 像使用 Git 推送任何代码一样，将示例代码部署到 Azure 应用。出现提示时，使用之前配置的密码。

        git push azure master

    ![在 Azure 中将代码推送到第一个 Web 应用](./media/app-service-web-get-started-languages/php-git-push.png)  


    `git push` 不仅将代码放在 Azure 中，也在部署引擎中触发部署任务。另外，还可以[启用编辑器扩展](/documentation/articles/web-sites-php-mysql-deploy-use-git/#composer)，以在 PHP 应用中自动处理 composer.json 文件。

恭喜！应用已部署到 Azure App Service。

## 查看应用实时运行

若要查看 Azure 中实时运行的应用，请从存储库中的任何目录运行以下命令：

    azure site browse

## 更新应用

现在可以使用 Git 随时从项目（存储库）根目录进行推送，以更新实时站点。操作方式与首次部署代码时相同。例如，每次想要推送已在本地测试的新更改时，只需从项目（存储库）根目录运行以下命令：

    git add .
    git commit -m "<your_message>"
    git push azure master

## 后续步骤

[创建、配置 Laravel Web 应用，并将其部署到 Azure](/documentation/articles/app-service-web-php-get-started/)。通过学习本教程，可以了解在 Azure 中运行 PHP Web 应用所需的基本技能，如：

- 使用 PowerShell/Bash 在 Azure 中创建和配置应用。
- 设置 PHP 版本。
- 使用不在根应用程序目录中的开始文件。
- 实现 Commposer 自动化。
- 访问环境特定的变量。
- 排查常见错误。

或者，对第一个 Web 应用执行更多操作。例如：

- 尝试[将代码部署到 Azure 的其他方法](/documentation/articles/web-sites-deploy/)。
- 使 Azure 应用上升到更高的层次。对用户进行身份验证。按需缩放。设置一些性能警报。所有这些操作只需按几下鼠标即可完成。请参阅[在第一个 Web 应用中添加功能](/documentation/articles/app-service-web-get-started-2/)。

<!---HONumber=Mooncake_1024_2016-->