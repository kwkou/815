<properties
	pageTitle="从移动服务迁移到应用服务移动应用"
	description="了解如何轻松将移动服务应用程序迁移到应用服务移动应用"
	services="app-service\mobile"
	documentationCenter=""
	authors="adrianhall"
	manager="dwrede"
	editor=""/>  


<tags
	ms.service="app-service-mobile"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile"
	ms.devlang="na"
	ms.topic="article"
	ms.date="10/03/2016"
	wacn.date="12/26/2016"
	ms.author="adrianha"/>

# <a name="article-top"></a>将现有的 Azure 移动服务迁移到 Azure 应用服务

通过 Azure 应用服务正式版，可以轻松迁移 Azure 移动服务站点，使用 Azure 应用服务的所有功能。本文档说明将站点从 Azure 移动服务迁移到 Azure 应用服务时的情形。

## <a name="what-does-migration-do"></a>迁移对站点有何作用

迁移 Azure 移动服务可使移动服务变成 [Azure App Service] 应用，且不会对代码造成影响。通知中心、SQL 数据连接、身份验证设置、计划作业和域名都将保持不变。使用 Azure 移动服务的移动客户端仍可正常运行。迁移操作将服务传输到 Azure App Service 之后，服务会重新启动。

[AZURE.INCLUDE [app-service-mobile-migrate-vs-upgrade](../../includes/app-service-mobile-migrate-vs-upgrade.md)]

## <a name="why-migrate"></a>为何要迁移站点

Microsoft 建议迁移 Azure 移动服务来利用 Azure 应用服务的各项功能，其中包括：

  *  新的主机功能，包括 [Web 作业]和[自定义域名]。
  *  除了混合连接以外，还可使用 VNet 连接到本地资源。
  *  使用 New Relic 进行监视和故障排除。
  *  内置的 DevOps 工具，包括[过渡槽]、回滚和生产环境内测试。
  *  [自动缩放]、负载均衡和[性能监视]。

有关 Azure 应用服务优点的详细信息，请参阅 [Mobile Services vs.App Service]（移动服务与应用服务的比较）主题。

## <a name="before-you-begin"></a>准备工作

在站点上开始任何主要任务之前，应该先[备份移动服务]脚本和 SQL 数据库。

## <a name="migrating-site"></a>迁移站点

迁移过程将迁移单个 Azure 区域中的所有站点。

若要迁移站点，请执行以下操作：

  1.  登录到 [Azure 经典管理门户]。
  2.  在区域中选择要迁移的移动服务。
  3.  单击“迁移到应用服务”按钮。

    ![迁移按钮][0]  


  4.  阅读“迁移到应用服务”对话框。
  5.  在提供的框中输入移动服务名称。例如，如果域名是 contoso.azure-mobile.cn，请在提供的框中输入 _contoso_ 。
  6.  单击打勾按钮。

在活动监视器中监视迁移状态。站点将在 Azure 经典管理门户中列为“正在迁移”。

![迁移活动监视器][1]  


迁移每个移动服务可能需要 3 到 15 分钟时间不等。站点在迁移期间仍可用。迁移过程结束时，将重启站点。站点在重启期间无法使用，此状况可能持续几秒钟。

## <a name="finalizing-migration"></a>完成迁移

计划在迁移过程结束后，从移动客户端测试站点。请确保可以执行所有一般客户端操作，而不会更改移动客户端。

### <a name="update-app-service-tier"></a>选择适当的应用服务定价层

在迁移到 Azure 应用服务之后，定价方面有更多弹性。

  1.  登录到 [Azure 门户预览]。
  2.  选择“所有资源”或“应用程序服务”，然后单击已迁移的移动应用的名称。
  3.  默认打开“设置”边栏选项卡。
  4.  在“设置”菜单中单击“应用服务计划”。
  5.  单击“定价层”磁贴。
  6.  单击符合要求的磁贴，然后单击“选择”。可能需要单击“全部查看”，才能查看可用的定价层。

建议使用以下定价层作为起点：

| 移动服务定价层 | 应用服务定价层 |
| :-------------------------- | :----------------------- |
| 免费 | F1 免费 |
| 基本 | B1 基本 |
| 标准 | S1 标准 |

为应用程序选择适当的定价层有相当大的灵活性。请参阅 [App Service Pricing]（应用服务定价），全面了解新应用服务定价的详细信息。

> [AZURE.TIP] 应用服务标准层包含可能要使用的多种功能的访问权限，包括[过渡槽]、自动备份和自动缩放。可以在相关位置查看新功能。

### <a name="review-migration-scheduler-jobs"></a>查看已迁移的计划程序作业

计划程序作业在迁移后约 30 分钟内不会显示。计划的作业将在后台持续运行。若要在计划的作业再次显示后进行查看，请执行以下操作：

  1.  登录到 [Azure 门户预览]。
  2.  选择“浏览>”，在“筛选”框中输入“计划程序”，后选择“计划程序集合”。

迁移后可用的计划程序作业数量有所限制。查看使用情况和 [Azure 计划程序计划]。

### <a name="configure-cors"></a>根据需要配置 CORS

跨域资源共享是可以让网站访问不同域上的 Web API 的技术。如果使用的 Azure 移动服务具有关联的网站，则需要将 CORS 配置为迁移的一部分。如果从移动设备以独占方式访问 Azure 移动服务，则在绝大多数情况下都不需要配置 CORS。

已迁移的 CORS 设置可用作 **MS\_CrossDomainWhitelist** 应用设置。若要将站点迁移到应用服务CORS 设施，请执行以下操作：

  1.  登录到 [Azure 门户预览]。
  2.  选择“所有资源”或“应用程序服务”，然后单击已迁移的移动应用的名称。
  3.  默认打开“设置”边栏选项卡。
  4.  在“API”菜单中单击“CORS”。
  5.  在提供的框中输入任何允许的来源，每输入一个就按 Enter。
  6.  如果“允许的来源”列表正确无误，请单击“保存”按钮。

> [AZURE.TIP]  使用 Azure 应用服务的优势之一是可以在相同站点上运行网站和移动服务。有关详细信息，请参阅[后续步骤](#next-steps)部分。

### <a name="download-publish-profile"></a>下载新的发布配置文件

站点发布配置文件迁移到 Azure 应用服务时会发生更改。如果想要从 Visual Studio 内部发布站点，则需要新的发布配置文件。若要下载新的发布配置文件，请执行以下操作：

  1.  登录到 [Azure 门户预览]。
  2.  选择“所有资源”或“应用程序服务”，然后单击已迁移的移动应用的名称。
  3.  单击“获取发布配置文件”。

PublishSettings 文件将下载到计算机。此文件通常名为 _站点名称_.PublishSettings。将发布设置导入现有项目：

  1.  打开 Visual Studio 和 Azure 移动服务项目。
  2.  在“解决方案资源管理器”中右键单击该项目并选择“发布...”
  3.  单击“导入”
  4.  单击“浏览”，然后选择已下载的发布配置文件。单击“确定”
  5.  单击“验证连接”，确保发布设置有效。
  6.  单击“发布”以发布站点。


## <a name="working-with-your-site"></a>在迁移后使用站点

迁移后，可在 [Azure 门户预览]中开始使用新的应用服务。下面是曾在 [Azure 经典管理门户]中执行的特定操作的某些注意事项，及其应用服务的等效功能。

### <a name="publishing-your-site"></a>下载和发布已迁移的站点

站点可通过 git 或 ftp 来使用，并可通过各种不同的机制重新发布，包括 WebDeploy、TFS、Mercurial、GitHub 和 FTP。部署凭据随着站点的其余部分迁移。如果未设置部署凭据或不记得，可以将其重置：

  1. 登录到 [Azure 门户预览]。
  2. 选择“所有资源”或“应用程序服务”，然后单击已迁移的移动应用的名称。
  3. 默认打开“设置”边栏选项卡。
  4. 在“发布”菜单中单击“部署凭据”。
  5. 在提供的框中输入新部署凭据，然后单击“保存”按钮。

可以使用这些凭据通过 git 克隆站点，或者从 GitHub、TFS 或 Mercurial 设置自动化部署。有关详细信息，请参阅 [Azure 应用服务部署文档]。

### <a name="appsettings"></a>应用程序设置

可以通过“应用设置”使用已迁移的移动服务的大多数设置。可以从 [Azure 门户预览]获取应用设置列表。若要查看或更改应用设置，请执行以下操作：

  1. 登录到 [Azure 门户预览]。
  2. 选择“所有资源”或“应用程序服务”，然后单击已迁移的移动应用的名称。
  3. 默认打开“设置”边栏选项卡。
  4. 在“常规”菜单中单击“应用程序设置”。
  5. 滚动到“应用设置”部分，并找到应用设置。
  6. 单击应用设置的值，编辑该值。单击“保存”以保存该值。

可以同时更新多个应用设置。

> [AZURE.TIP]  有两项应用程序设置的值相同。例如，可能会出现 _ApplicationKey_ 和 _MS_ApplicationKey_ 。同时更新这两个应用程序设置。

### <a name="authentication"></a>身份验证

所有身份验证设置都可用作已迁移的站点中的应用设置。若要更新身份验证设置，必须更改相应的应用设置。下表显示身份验证提供程序的相应应用设置：

| 提供程序 | 客户端 ID | 客户端机密 | 其他设置 |
| :---------------- | :------------------------ | :--------------------------- | :------------------------- |
| Microsoft 帐户 | **MS\_MicrosoftClientID** | **MS\_MicrosoftClientSecret** | **MS\_MicrosoftPackageSID** |
| Azure AD | **MS\_AadClientID** | | **MS\_AadTenants** |

注意：**MS\_AadTenants** 存储为租户域（移动服务门户中的“允许的租户”字段）的逗号分隔列表。

> [AZURE.WARNING] **请不要使用“设置”菜单中的身份验证机制**
>
> Azure 应用服务分别在“身份验证/授权设置”菜单下提供“无代码”身份验证和授权系统，以及在“设置”菜单下提供已过时的“移动身份验证”选项。这些选项与已迁移的 Azure 移动服务不兼容。可以[升级站点](/documentation/articles/app-service-mobile-net-upgrading-from-mobile-services/)来利用 Azure 应用服务身份验证。

### <a name="easytables"></a>数据

移动服务中的“数据”选项卡在 Azure 门户预览中已被“简易表”取代。若要访问“简易表”，请执行以下操作：

  1. 登录到 [Azure 门户预览]。
  2. 选择“所有资源”或“应用程序服务”，然后单击已迁移的移动应用的名称。
  3. 默认打开“设置”边栏选项卡。
  4. 在“移动”菜单中单击“简易表”。

可以单击“添加”按钮来添加表，或者单击表名称来访问现有表。在此边栏选项卡中可以执行多种操作，包括：

  * 更改表权限
  * 编辑操作脚本
  * 管理表架构
  * 删除表
  * 清除表内容
  * 删除特定的表行

### <a name="easyapis"></a>API

移动服务中的“API”选项卡在 Azure 门户预览中已被“简易 API”取代。若要访问“简易 API”，请执行以下操作：

  1. 登录到 [Azure 门户预览]。
  2. 选择“所有资源”或“应用程序服务”，然后单击已迁移的移动应用的名称。
  3. 默认打开“设置”边栏选项卡。
  4. 在“移动”菜单中单击“简易 API”。

边栏选项卡中已列出已迁移的 API。也可从此边栏选项卡添加 API。若要管理特定 API，请单击该 API。从新的边栏选项卡中，可以调整权限以及编辑 API 的脚本。

### <a name="on-demand-jobs"></a>计划程序作业

可通过“计划程序作业集合”部分使用所有计划程序作业。若要访问计划程序作业，请执行以下操作：

  1. 登录到 [Azure 门户预览]。
  2. 选择“浏览>”，在“筛选”框中输入“计划程序”，后选择“计划程序集合”。
  3. 选择站点的作业集合。其名称为 _站点名称_-Jobs。
  4. 单击“设置”。
  5. 单击“管理”下的“计划程序作业”。

计划的作业以迁移之前指定的频率列出。按需作业将禁用。若要运行按需作业，请执行以下操作：

  1. 选择想要运行的作业。
  2. 如果需要，请单击“启用”以启用该作业。
  3. 单击“设置”，然后单击“计划”。
  4. 为周期选择“一次”，然后单击“保存”

按需作业位于 `App_Data/config/scripts/scheduler post-migration` 中。建议将所有按需作业转换为 [Web 作业]或 [函数]。编写新的计划程序作业作为 [Web 作业]或 [函数]。

### <a name="notification-hubs"></a>通知中心

移动服务使用通知中心来处理推送通知。迁移后，可使用以下应用设置将通知中心链接到移动服务：

| 应用程序设置 | 说明 |
| :------------------------------------- | :--------------------------------------- |
| **MS\_PushEntityNamespace** | 通知中心命名空间 |
| **MS\_NotificationHubName** | 通知中心名称 |
| **MS\_NotificationHubConnectionString** | 通知中心连接字符串 |
| **MS\_NamespaceName** | MS\_PushEntityNamespace 的别名 |

可通过 [Azure 门户预览]管理通知中心。记下通知中心名称（可以使用“应用设置”找到此信息）：

  1. 登录到 [Azure 门户预览]。
  2. 选择“浏览>”，然后选择“通知中心”
  3. 单击与移动服务关联的通知中心名称。

> [AZURE.NOTE] 如果通知中心是“混合”类型，则它不可见。“混合”类型的通知中心同时利用通知中心功能和旧式服务总线功能。[转换混合命名空间]，然后继续。转换完成后，通知中心将出现在 [Azure 门户预览]中。


> [AZURE.TIP] [Azure 门户预览]中的通知中心管理功能仍以预览版提供。仍可使用 [Azure 经典管理门户]管理所有通知中心。

### <a name="legacy-push"></a>传统推送设置

如果在通知中心引入之前即已配置移动服务的推送，则使用的是传统推送。如果使用推送，但配置中没有列出通知中心，则可能使用的是传统推送。此功能与其他所有功能一起迁移。但是，建议完成迁移后尽快升级到通知中心。

在过渡期，所有传统推送设置（APNS 证书除外）都可以在应用设置中获取。通过替换文件系统中相应的文件即可更新 APNs 证书。

### <a name="app-settings"></a>其他应用设置

下面是从移动服务迁移的其他应用设置，可从“设置”>“应用设置”来使用：

| 应用程序设置 | 说明 |
| :------------------------------- | :-------------------------------------- |
| **MS\_MobileServiceName** | 应用的名称 |
| **MS\_MobileServiceDomainSuffix** | 域前缀，例如 azure-mobile.cn |
| **MS\_ApplicationKey** | 应用程序密钥 |
| **MS\_MasterKey** | 应用主密钥 |

应用程序密钥和主密钥应与原始移动服务中的应用程序密钥完全相同。具体而言，移动客户端发送应用程序密钥，验证它们对移动 API 的使用。

### <a name="cliequivalents"></a>命令行等效项

不能再使用 _azure mobile_ 命令管理 Azure 移动服务站点。有许多函数已由 _azure site_ 命令取代。使用下表查找常用命令的等效项：

| *Azure 移动* 命令 | 等效的 *Azure 站点* 命令 |
| :----------------------------------------- | :----------------------------------------- |
| mobile locations | site location list |
| mobile list | site list |
| mobile show _名称_ | site show _名称_ |
| mobile restart _名称_ | site restart _名称_ |
| mobile redeploy _名称_ | site deployment redeploy _commitId_ _名称_ |
| mobile key set _名称_ _类型_ _值_ | site appsetting delete _键_ _名称_ <br/> site appsetting add _键_=_值_ _名称_ |
| mobile config list _名称_ | site appsetting list _名称_ |
| mobile config get _名称_ _键_ | site appsetting show _键_ _名称_ |
| mobile config set _名称_ _键_ | site appsetting delete _键_ _名称_ <br/> site appsetting add _键_=_值_ _名称_ |
| mobile domain list _名称_ | site domain list _名称_ |
| mobile domain add _名称_ _域_ | site domain add _域_ _名称_ |
| mobile domain delete _名称_ | site domain delete _域_ _名称_ |
| mobile scale show _名称_ | site show _名称_ |
| mobile scale change _名称_ | site scale mode _模式_ _名称_ <br /> site scale instances _实例_ _名称_ |
| mobile appsetting list _名称_ | site appsetting list _名称_ |
| mobile appsetting add _名称_ _键_ _值_ | site appsetting add _键_=_值_ _名称_ |
| mobile appsetting delete _名称_ _键_ | site appsetting delete _键_ _名称_ |
| mobile appsetting show _名称_ _键_ | site appsetting delete _键_ _名称_ |

通过更新相应的应用程序设置，可以更新身份验证或推送通知设置。请编辑文件，并通过 ftp 或 git 发布站点。

### <a name="diagnostics"></a>诊断和日志记录

Azure 应用服务中通常会禁用诊断日志记录。若要启用诊断日志记录，请执行以下操作：

  1. 登录到 [Azure 门户预览]。
  2. 选择“所有资源”或“应用程序服务”，然后单击已迁移的移动应用的名称。
  3. 默认打开“设置”边栏选项卡。
  4. 选择“功能”菜单下的“诊断日志”。
  5. 针对以下日志单击“打开”：“应用程序日志记录(文件系统)”、“详细错误消息”和“失败请求跟踪”
  6. 对于 Web 服务器日志记录，请单击“文件系统”
  7. 单击“保存”

若要查看日志，请执行以下操作：

  1. 登录到 [Azure 门户预览]。
  2. 选择“所有资源”或“应用程序服务”，然后单击已迁移的移动应用的名称。
  3. 单击“工具”按钮
  4. 在“观察”菜单下选择“日志流”。

生成日志后，会将其显示在窗口中。还可以下载日志，以便后续使用部署凭据进行分析。有关详细信息，请参阅[日志记录]文档。

## <a name="known-issues"></a>已知问题

### 删除迁移的移动应用副本会导致站点中断

如果使用 Azure PowerShell 克隆迁移的移动服务，然后又删除该副本，则会删除生产服务的 DNS 项。不能再从 Internet 访问该站点。

解决方案：如果想要克隆站点，请使用门户。

### 无法更改 Web.config

如果具有 ASP.NET 站点，则无法应用 `Web.config` 文件的更改。Azure 应用服务在启动期间构建适当的 `Web.config` 文件来支持移动服务运行时。可以使用 XML 转换文件来覆盖特定的设置（例如自定义标头）。创建名为 `applicationHost.xdt` 的文件 - 此文件必须在 Azure 服务上的 `D:\home\site` 目录中结束。可通过自定义部署脚本或直接使用 Kudu 上传 `applicationHost.xdt` 文件。下面显示了一个示例文档：

```
<?xml version="1.0" encoding="utf-8"?>
<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
  <system.webServer>
    <httpProtocol>
      <customHeaders>
        <add name="X-Frame-Options" value="DENY" xdt:Transform="Replace" />
        <remove name="X-Powered-By" xdt:Transform="Insert" />
      </customHeaders>
    </httpProtocol>
    <security>
      <requestFiltering removeServerHeader="true" xdt:Transform="SetAttributes(removeServerHeader)" />
    </security>
  </system.webServer>
</configuration>
```

有关详细信息，请参阅 GitHub 上的 [XDT Transform Samples]（XDT 转换示例）文档。

### 无法将迁移的移动服务添加到流量管理器
创建流量管理器配置文件时，无法直接选择配置文件的迁移移动服务。使用“外部终结点”。 外部终结点只能通过 PowerShell 来添加。有关详细信息，请参阅[流量管理器教程](https://azure.microsoft.com/blog/azure-traffic-manager-external-endpoints-and-weighted-round-robin-via-powershell/)。

## <a name="next-steps"></a>后续步骤

除了将应用程序迁移到应用服务以外，还可使用其他一些功能：

  * 使用部署[过渡槽]可以暂存站点更改，以及执行 A/B 测试。
  * [Web 作业]可以取代按需计划作业。
  * 可通过将站点链接到 GitHub、TFS 或 Mercurial [连续部署]站点。
  * 以相同的代码为网站和移动 API 提供服务。

### <a name="upgrading-your-site"></a>将移动服务站点升级到 Azure 移动应用 SDK

  * 对于基于 Node.js 的服务器项目，新的[移动应用 Node.js SDK] 提供了一些新功能。例如，现在可以执行本地开发和调试、使用 0.10 以上的任何 Node.js 版本，以及使用任何 Express.js 中间件自定义。

  * 对于基于 .NET 的服务器项目，新的[移动应用 SDK NuGet 包](https://www.nuget.org/packages/Microsoft.Azure.Mobile.Server/)在 NuGet 依赖性上更为灵活。这些包支持新的应用服务身份验证，并可与任何 ASP.NET 项目组合使用。有关升级的详细信息，请参阅 [Upgrade your existing .NET Mobile Service to App Service](/documentation/articles/app-service-mobile-net-upgrading-from-mobile-services/)（将现有 .NET 移动服务升级到应用服务）。

<!-- Images -->
[0]: ./media/app-service-mobile-migrating-from-mobile-services/migrate-to-app-service-button.PNG
[1]: ./media/app-service-mobile-migrating-from-mobile-services/migration-activity-monitor.png
[2]: ./media/app-service-mobile-migrating-from-mobile-services/triggering-job-with-postman.png

<!-- Links -->
[App Service pricing]: /pricing/details/app-service/
[自动缩放]: /documentation/articles/web-sites-scale/
[Azure App Service]: /documentation/articles/app-service-value-prop-what-is/
[Azure 应用服务部署文档]: /documentation/articles/web-sites-deploy/
[Azure 经典管理门户]: https://manage.windowsazure.cn
[Azure 门户预览]: https://portal.azure.cn
[Azure 计划程序计划]: /documentation/articles/scheduler-plans-billing/
[连续部署]: /documentation/articles/app-service-continuous-deployment/
[转换混合命名空间]: https://azure.microsoft.com/blog/updates-from-notification-hubs-independent-nuget-installation-model-pmt-and-more/
[curl]: http://curl.haxx.se/
[自定义域名]: /documentation/articles/web-sites-custom-domain-name/
[Fiddler]: http://www.telerik.com/fiddler

[日志记录]: /documentation/articles/web-sites-enable-diagnostic-log/
[移动应用 Node.js SDK]: https://github.com/azure/azure-mobile-apps-node
[Mobile Services vs.App Service]: /documentation/articles/app-service-mobile-value-prop-migration-from-mobile-services/
[性能监视]: /documentation/articles/web-sites-monitor/
[Postman]: http://www.getpostman.com/
[备份移动服务]: /documentation/articles/mobile-services-disaster-recovery/
[过渡槽]: /documentation/articles/web-sites-staged-publishing/
[Web 作业]: /documentation/articles/websites-webjobs-resources/
[XDT Transform Samples]: https://github.com/projectkudu/kudu/wiki/Xdt-transform-samples

<!---HONumber=Mooncake_1219_2016-->