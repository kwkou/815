<properties linkid="manage-services-sql-databases-premium" urlDisplayName="Premium SQL Database" pageTitle="注册 Azure SQL Database 高级版" metaKeywords="" description="介绍了如何注册 SQL Database 高级预览版、请求高级版数据库配额，然后在 Azure SQL Database 中将数据库升级到高级版。" metaCanonical="" services="cloud-services" documentationCenter="" title="注册 Azure SQL Database 高级预览版" authors="karaman" solutions="" manager="" editor="tysonn" />

# 注册 Azure SQL Database 高级预览版

在本教程中，您将了解参与 SQL Database 高级预览版所需的步骤。

Azure SQL Database 已发布了新服务 SQL Database 高级版的有限预览版。高级版数据库提供了保留的资源，可为云应用程序提供更易于预测的性能。

[本主题中所述的功能仅在预览版中提供。本主题是预发布的文档，在将来发布的版本中可能会有更改。]

## 目录

-   [步骤 1：注册 SQL Database 高级预览版][步骤 1：注册 SQL Database 高级预览版]
-   [步骤 2：请求高级版数据库配额][步骤 2：请求高级版数据库配额]
-   [步骤 3：创建高级版数据库][步骤 3：创建高级版数据库]

## <span id="SignUp"></span></a>步骤 1：注册 SQL Database 高级预览版

利用此功能的第一步是注册您对 SQL Database 高级预览版的订阅。

1.  使用您的 Microsoft 帐户登录到 [Azure 预览功能页][Azure 预览功能页]。

    **注意** - 只有 Azure 帐户管理员才能访问帐户门户。如果您不是您的订阅的帐户管理员，请让帐户管理员完成订阅的注册过程。有关 Azure 帐户和订阅的详细信息，请参阅[购买选项][Azure 预览功能页]。

    ![图像 1][图像 1]

2.  在预览功能列表中找到“SQL Database 高级版”项，然后单击与该项关联的“立即试用”按钮。

    ![图像 2][图像 2]

3.  选择您要注册预览版的订阅。

    ![图像 3][图像 3]

    仅当处于活动状态，付费的 Azure 订阅才可供预览。您可以注册多个订阅，但每个订阅只能注册一次。

    注册预览版的订阅不会产生额外费用，但是一旦激活并授予高级版配额，创建或升级到高级版数据库时便需要按 [SQL Database 定价页][SQL Database 定价页]中所述付费。

    注册请求的当前状态将反映在预览功能列表中。

    ![图像 4][图像 4]

4.  请求将基于当前容量和需求进行批准。您最多需要等待 2 天即可激活您的订阅，等待时间较长表明需求较高或公开预览容量已用完。

5.  一旦您的预览版订阅得到激活，您将收到电子邮件通知。

## <span id="Quota"></span></a>步骤 2：请求高级版数据库配额

一旦您的预览版订阅得到激活，您需要为计划创建高级版数据库的每台服务器请求高级版数据库配额。当容量受限时，请只为您计划创建高级版数据库的服务器请求配额，并取消所有不需要的待定请求。

1.  使用您的 Windows 帐户登录到 [Azure 管理门户][Azure 管理门户]。

    **注意** - 一旦注册了预览版订阅，订阅的帐户管理员、服务管理员和共同管理员就可以请求配额。

2.  导航到“SQL Database”扩展中的“服务器”列表。
3.  选择您计划为其请求高级数据库配额的服务器。
4.  通过单击位于顶部导航栏中的闪电形图标，导航到所选服务器的“快速启动”。
5.  在“高级版数据库”部分中单击“请求高级版数据库配额”。

    ![图像 6][图像 6]

    提交请求后，您可能需要等待多达 5 天才能被授予配额。等待时间较长可能表明需求较高或预览版容量已用完。

    有关高级数据库配额请求的一些额外注意事项：

    -   配额不适用于具有免费试用订阅的客户。
    -   配额最初是受限的，将根据当前要求和可用容量对请求授予配额。
    -   Microsoft 可能在 15 天后收回未使用的配额。
    -   对于订阅中的每台逻辑服务器，只能提交一个配额请求。
    -   最初，配额限制为每台逻辑服务器一个数据库。
    -   可免费请求数据库配额，但创建高级版数据库或将现有的 Web 版或企业版数据库升级到高级版将增加数据库的成本。

6.  您可以在服务器的“快速启动”页上查看配额请求的状态。

    ![图像 7][图像 7]

7.  当同意您的高级数据库配额请求且配额可供使用时，您将收到电子邮件通知。
8.  一旦获得授权，您就可以在服务器的“快速启动”或“仪表板”选项卡上看到该服务器的高级版数据库的剩余配额。

    ![图像 8][图像 8]

## <span id="Upgrade"></span></a>步骤 3：创建高级版数据库

一旦您获得了配额，就可以创建新的高级版数据库或将现有的 Web 版或企业版数据库升级到高级版，以利用保留的容量并获得更易预测的性能。

![图像 9][图像 9]

![图像 10][图像 10]

有关详细信息，请参阅[管理高级版数据库][管理高级版数据库]。

**注意：** 在 24 小时期间内，您只能更改数据库的高级版状态或保留大小一次。

## <span id="NextSteps"></span></a> 后续步骤

有关高级版数据库的其他信息，请参阅：

</p>
-   [管理高级数据库][管理高级版数据库]
-   [SQL Database 高级版指南][SQL Database 高级版指南]

  [步骤 1：注册 SQL Database 高级预览版]: #SignUp
  [步骤 2：请求高级版数据库配额]: #Quota
  [步骤 3：创建高级版数据库]: #Upgrade
  [Azure 预览功能页]: http://account.windowsazure.com/PreviewFeatures
  [图像 1]: ./media/sql-database-premium-sign-up/AccountSignup-Figure1.png
  [图像 2]: ./media/sql-database-premium-sign-up/AccountSignupButton-Figure2.png
  [图像 3]: ./media/sql-database-premium-sign-up/Subscription-Figure3.png
  [SQL Database 定价页]: http://www.windowsazure.com/zh-cn/pricing/details/sql-database/
  [图像 4]: ./media/sql-database-premium-sign-up/Status-Figure4.png
  [Azure 管理门户]: https://manage.windowsazure.com
  [图像 6]: ./media/sql-database-premium-sign-up/RequestQuota-Figure6.png
  [图像 7]: ./media/sql-database-premium-sign-up/PendingApproval-Figure7.png
  [图像 8]: ./media/sql-database-premium-sign-up/QuotaApproved-Figure8.png
  [图像 9]: ./media/sql-database-premium-sign-up/SpecifyDBSettings-Figure9.png
  [图像 10]: ./media/sql-database-premium-sign-up/PremiumDBSettings-Figure10.png
  [管理高级版数据库]: http://go.microsoft.com/fwlink/p/?LinkID=311927
  [SQL Database 高级版指南]: http://go.microsoft.com/fwlink/p/?LinkId=313650
