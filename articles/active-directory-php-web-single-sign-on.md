<properties linkid="develop-php-how-to-guides-web-sso" urlDisplayName="Web SSO" pageTitle="Single sign-on with Azure Active Directory (PHP)" metaKeywords="Azure PHP web app, Azure single sign-on, Azure PHP Active Directory" description="Learn how to create a PHP web application that uses single sign-on with Azure Active Directory." metaCanonical="" services="active-directory" documentationCenter="PHP" title="Web Single Sign-On with PHP and Azure Active Directory" authors="robmcm" solutions="" manager="wpickett" editor="mollybos" videoId="" scriptId="" />

# 通过 PHP 和 Azure Active Directory 实现的 Web 单一登录

## <a name="introduction"></a>介绍

本教程将向 PHP 开发人员介绍如何利用 Azure Active Directory 为 Office 365 客户的用户实现单一登录。你将了解如何执行以下操作：

-   在客户的租户中设置 Web 应用程序
-   使用 WS 联合身份验证保护应用程序

### 先决条件

本演练具有以下开发环境先决条件：

-   [针对 Azure Active Directory 的 PHP 示例代码][针对 Azure Active Directory 的 PHP 示例代码]
-   [Eclipse PDT 3.0.x 一体式][Eclipse PDT 3.0.x 一体式]
-   PHP 5.3.1（通过 Web 平台安装程序）
-   启用了 SSL 的 Internet Information Services (IIS) 7.5
-   Windows PowerShell
-   [Office 365 PowerShell Commandlets][Office 365 PowerShell Commandlets]

### 目录

-   [介绍][介绍]
-   [步骤 1：创建 PHP 应用程序][步骤 1：创建 PHP 应用程序]
-   [步骤 2：在公司的目录租户中设置应用程序][步骤 2：在公司的目录租户中设置应用程序]
-   [步骤 3：通过对员工登录使用 WS 联合身份验证来保护应用程序][步骤 3：通过对员工登录使用 WS 联合身份验证来保护应用程序]
-   [摘要][摘要]

## <a name="createapp"></a>步骤 1：创建 PHP 应用程序

此步骤介绍如何创建将表示受保护资源的简单 PHP 应用程序。对此资源的访问权限将通过公司的 STS 管理的联合身份验证（将在本教程后面部分介绍）授予。

1.  打开 Eclipse 的新实例。
2.  在**“文件”**菜单中，单击**“新建”**，然后单击**“新建 PHP 项目”**。
3.  在**“新建 PHP 项目”**对话框中，将项目命名为 *phpSample*，然后单击**“完成”**。
4.  在左侧的**“PHP 资源管理器”**菜单中，右键单击 *phpProject*，单击**“新建”**，然后单击**“PHP 文件”**。
5.  在**“新建 PHP 文件”**对话框中，将文件命名为 **index.php**，然后单击**“完成”**。
6.  将生成的标记替换为以下内容，然后保存 **index.php**：

        <!DOCTYPE>
        <html>
        <head>
            <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
            <title>Index</title>
        </head>
        <body>
            ##Index Page
        </body>
        </html> 

7.  通过在运行提示下键入 *inetmgr* 并按 Enter 来打开 **Internet Information Services (IIS) 管理器**。

8.  在 IIS 管理器中，展开左侧菜单中的 **Sites** 文件夹，右键单击**“默认网站”**，然后单击**“添加应用程序...”**。

9.  在**“添加应用程序”**对话框中，将**“别名”**值设置为 *phpSample*，将**“物理路径”**设置为用于创建 PHP 项目的文件路径。

10. 在 Eclipse 的**“运行”**菜单中，单击**“运行”**。

11. 在**“运行 PHP Web 应用程序”**菜单中，单击**“确定”**。

12. **index.php** 页将在 Eclipse 的新选项卡中打开。该页面只应显示以下文本：*索引页*。

## <a name="provisionapp"></a>步骤 2：在公司的目录租户中设置应用程序

此步骤介绍 Azure Active Directory 客户的管理员如何在其租户中设置 PHP 应用程序以及如何配置单一登录。完成此步骤后，公司的员工可以使用其 Office 365 帐户对 Web 应用程序进行身份验证。

设置过程的第一步是为应用程序创建新的服务主体。Azure Active Directory 将使用服务主体来向目录注册和验证应用程序。

1.  如果尚未下载和安装 Office 365 PowerShell Commandlets，则执行此操作。
2.  在**“开始”**菜单中，运行**“用于 Windows PowerShell 的 Microsoft Online Services 模块”**控制台。此控制台提供了用于配置有关 Office 365 租户的属性（如创建和修改服务主体）的命令行环境。
3.  若要导入必需的 **MSOnlineExtended** 模块，键入以下命令并按 Enter：

        Import-Module MSOnlineExtended -Force

4.  若要连接到 Office 365 目录，你需要提供公司的管理员凭据。请键入以下命令并按 Enter，然后在提示符处输入凭据：

        Connect-MsolService

5.  现在，你将为应用程序创建新的服务主体。请键入以下命令并按 Enter：

        New-MsolServicePrincipal -ServicePrincipalNames @("phpSample/localhost") -DisplayName "Federation Sample Web Site" -Type Symmetric -Usage Verify -StartDate "12/01/2012" -EndDate "12/01/2013" 

    此步骤将输出类似于下面的信息：

        The following symmetric key was created as one was not supplied qY+Drf20Zz+A4t2w e3PebCopoCugO76My+JMVsqNBFc=
        DisplayName           : Federation Sample PHP Web Site
        ServicePrincipalNames : {phpSample/localhost}
        ObjectId              : 59cab09a-3f5d-4e86-999c-2e69f682d90d
        AppPrincipalId        : 7829c758-2bef-43df-a685-717089474505
        TrustedForDelegation  : False
        AccountEnabled        : True
        KeyType               : Symmetric
        KeyId                 : f1735cbe-aa46-421b-8a1c-03b8f9bb3565
        StartDate             : 12/01/2012 08:00:00 a.m.
        EndDate               : 12/01/2013 08:00:00 a.m.
        Usage                 : Verify 

    > [WACOM.NOTE]
    > 你应该保存此输出，尤其是生成的对称密钥。此密钥仅在服务主体创建期间对你显示，你以后将无法检索它。使用 Graph API 在目录中读取和写入信息还需要其他值。

6.  最后一个步骤是为应用程序设置答复 URL。在进行身份验证尝试之后，将在答复 URL 中发送响应。请键入以下命令并按 Enter：

        $replyUrl = New-MsolServicePrincipalAddresses –Address "https://localhost/phpSample" 

        Set-MsolServicePrincipal –AppPrincipalId "7829c758-2bef-43df-a685-717089474505" –Addresses $replyUrl 

现在已经在目录中设置 Web 应用程序，公司员工可以使用它来进行 Web 单一登录。

## <a name="protectapp"></a>步骤 3：通过对员工登录使用 WS 联合身份验证来保护应用程序

此步骤演示如何使用 Windows Identity Foundation (WIF) 以及与必备组件中的示例代码一起下载的 simpleSAML.php 库来添加对联合登录的支持。你还将添加登录页并配置应用程序和目录租户之间的信任关系。

1.  在 Eclipse 中，右键单击 **phpSample** 项目，单击**“新建”**，然后单击**“文件”**。

2.  在**“新建文件”**对话框中，将文件命名为 **federation.ini**，然后单击**“完成”**。

3.  在新的 **federation.ini** 文件中，输入以下信息，并提供包含创建服务主体时在第 2 步中保存的信息的值：

        federation.trustedissuers.issuer=https://accounts.accesscontrol.windows.net/v2/wsfederation
        federation.trustedissuers.thumbprint=qY+Drf20Zz+A4t2we3PebCopoCugO76My+JMVsqNBFc=
        federation.trustedissuers.friendlyname=Fabrikam
        federation.audienceuris=spn:7829c758-2bef-43df-a685-717089474505
        federation.realm=spn:7829c758-2bef-43df-a685-717089474505
        federation.reply=https://localhost/phpSample/index.php 

        <div class="dev-callout"><strong>Note</strong><p>The <b>audienceuris</b> and <b>realm</b> values must be prefaced by "spn:".</p></div>

4.  在 Eclipse 中，右键单击 **phpSample** 项目，单击**“新建”**，然后单击**“文件”**。

5.  在**“新建 PHP 文件”**对话框中，将文件命名为 **secureResource.php**，然后单击**“完成”**。

6.  在新 **secureResource.php** 文件中，输入以下代码，并将 **c:\\phpLibraries** 路径替换为已下载示例代码的根位置。根位置应包含 **simpleSAML.php** 文件和 **federation** 文件夹：

        <?php
        ini_set('include_path', ini_get('include_path').';c:\phpLibraries\;');
        require_once ('federation/FederatedLoginManager.php');
        session_start();
        $token = $_POST['wresult'];
        $loginManager = new FederatedLoginManager();
        if (!$loginManager->isAuthenticated()) {
            if (isset ($token)) {
                try {
                    $loginManager->authenticate($token);
                } catch (Exception $e) {
                    print_r($e->getMessage());
                }  
            } else {
                $returnUrl = "https://" . $_SERVER['HTTP_HOST'] . $_SERVER['PHP_SELF'];
                header('Pragma: no-cache');
                header('Cache-Control: no-cache, must-revalidate');
                header("Location: " . FederatedLoginManager :: getFederatedLoginUrl($returnUrl), true, 302);
                exit();
            }
        }
        ?> 

7.  打开 **index.php** 页并更新其内容以保护页面，然后将其保存：

        <?php
        require_once (dirname(__FILE__) . '/secureResource.php');
        ?>
        <!DOCTYPE html>
        <html>
        <head>
            <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
            <title>Index Page</title>
        </head>
        <body>
            <h2>Index Page</h2>
            <h3>Welcome <strong><?php print_r($loginManager->getPrincipal()->getName()); ?></strong>!</h3>
            <h4>Claim list:</h4>
            <ul>
                <?php
                foreach ($loginManager->getClaims() as $claim) {
                    print_r('<li>' . $claim->toString() . '</li>');
                }
                ?>  
            </ul>
        </body>
        </html> 

8.  在**“运行”**菜单中，单击**“运行”**。你将自动重定向到“Office 365 标识提供程序”页，你可在其中使用目录租户凭据进行登录。例如 *john.doe@fabrikam.onmicrosoft.com*。

## <a name="summary"></a>摘要

本教程说明了如何创建和配置使用 Azure Active Directory 的单一登录功能的单租户 PHP 应用程序。

<https://github.com/WindowsAzure/azure-sdk-for-php-samples/tree/master/WAAD.WebSSO.PHP> 上提供了一个演示如何对 PHP 网站使用 Azure Active Directory 和单一登录的示例。

  [针对 Azure Active Directory 的 PHP 示例代码]: https://github.com/WindowsAzure/azure-sdk-for-php-samples/tree/master/WAAD.WebSSO.PHP
  [Eclipse PDT 3.0.x 一体式]: http://www.eclipse.org/pdt/downloads/
  [Office 365 PowerShell Commandlets]: http://technet.microsoft.com/zh-cn/library/jj151814.aspx
  [介绍]: #introduction
  [步骤 1：创建 PHP 应用程序]: #createapp
  [步骤 2：在公司的目录租户中设置应用程序]: #provisionapp
  [步骤 3：通过对员工登录使用 WS 联合身份验证来保护应用程序]: #protectapp
  [摘要]: #summary
