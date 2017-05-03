<properties
    pageTitle="Azure IoT 套件常见问题 | Azure"
    description="有关 IoT 套件的常见问题"
    services=""
    suite="iot-suite"
    documentationcenter=""
    author="dominicbetts"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="cb537749-a8a1-4e53-b3bf-f1b64a38188a"
    ms.service="iot-suite"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="02/15/2017"
    wacn.date="05/02/2017"
    ms.author="corywink"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="57642c199b090384d225101bded7ebff64d3a10c"
    ms.lasthandoff="04/22/2017" />

# <a name="frequently-asked-questions-for-iot-suite"></a>有关 IoT 套件的常见问题

### <a name="where-can-i-find-the-source-code-for-the-preconfigured-solutions"></a>在哪里可以找到预配置解决方案的源代码？
源代码存储在以下 GitHub 存储库中：

- [远程监控预配置解决方案][lnk-remote-monitoring-github]
- [预测性维护预配置解决方案][lnk-predictive-maintenance-github]

### <a name="how-do-i-update-to-the-latest-version-of-the-remote-monitoring-preconfigured-solution-that-uses-the-iot-hub-device-management-features"></a>如何更新到最新版的远程监控预配置解决方案，以便使用 IoT 中心设备管理功能？
* 如果从 https://www.azureiotsuite.cn/ 站点部署预配置解决方案，则始终部署的是最新版解决方案的全新实例。
* 如果使用命令行部署预配置解决方案，可以使用新代码更新现有部署。请参阅 GitHub [存储库][lnk-remote-monitoring-github]中的[云部署][lnk-cloud-deployment]。

### <a name="how-can-i-add-support-for-a-new-device-method-to-the-remote-monitoring-preconfigured-solution"></a>如何向远程监控预配置解决方案添加新设备方法的支持？
请参阅[自定义预配置解决方案][lnk-customize]一文的[向模拟器添加新方法的支持][lnk-add-method]部分。

### <a name="the-simulated-device-is-ignoring-my-desired-property-changes-why"></a>模拟设备忽略所需属性更改，这是什么原因？
在远程监控预配置解决方案中，模拟设备代码仅使用所需的属性 **Desired.Config.TemperatureMeanValue** 和 **Desired.Config.TelemetryInterval** 来更新报告的属性。将忽略所有其他的所需属性更改请求。

### <a name="my-device-does-not-appear-in-the-list-of-devices-in-the-solution-dashboard-why"></a>我的设备未显示在解决方案仪表板的设备列表中，这是什么原因？
解决方案仪表板中的设备列表使用查询来返回设备列表。 目前，查询返回的设备数不能超过 10000。 可以尝试让查询的搜索条件更具限制性。

### <a name="whats-the-difference-between-deleting-a-resource-group-in-the-azure-portal-and-clicking-delete-on-a-preconfigured-solution-in-azureiotsuitecom"></a>在 Azure 门户预览中删除资源组与在 azureiotsuite.cn 中对预配置解决方案单击删除之间的区别是什么？
* 如果在 [azureiotsuite.cn][lnk-azureiotsuite] 中删除预配置解决方案，则会删除在创建预配置解决方案时设置的所有资源。如果向资源组添加了其他资源，则也会删除这些资源。
* 如果删除 [Azure 门户预览][lnk-azure-portal]中的资源组，则只会删除该资源组中的资源。此外还需在 [Azure 经典管理门户][lnk-classic-portal]中删除与预配置的解决方案关联的 Azure Active Directory 应用程序。

### <a name="how-many-iot-hub-instances-can-i-provision-in-a-subscription"></a>在一个订阅中可以设置多少个 IoT 中心实例？
每个订阅可以预配 10 个 IoT 中心。可以创建 [Azure 在线申请支持](/support/support-ticket-form/?l=zh-cn)创建工单，提出申请以提高此限制，但默认情况下，如 [Azure subscription limits][link-azuresublimits]（Azure 订阅限制）中所述，对每个订阅只能预配 10 个 IoT 中心。由于每个预配置的解决方案将预配一个新的 IoT 中心，因此，在给定的订阅中，最多只能预配 10 个预配置的解决方案。

### <a name="how-many-documentdb-instances-can-i-provision-in-a-subscription"></a>在订阅中可以设置多少个 DocumentDB 实例？
50 个。可以创建[Azure 在线申请支持](/support/support-ticket-form/?l=zh-cn)创建工单，提出申请以提高此限制，但在默认情况下，对每个订阅只能预配 50 个 DocumentDB 实例。






### <a name="can-i-create-a-preconfigured-solution-if-i-have-microsoft-azure-for-dreamspark"></a> 如果有 Azure for DreamSpark，是否可以创建预配置解决方案？
当前无法使用 [Azure for DreamSpark][lnk-dreamspark] 帐户创建预配置解决方案。但是，可以在几分钟内创建一个[试用帐户][1rmb-trial]，以便创建预配置的解决方案。

### <a name="can-i-create-a-preconfigured-solution-if-i-have-cloud-solution-provider-csp-subscription"></a>如果我有云解决方案提供商 (CSP) 订阅，是否可以创建预配置的解决方案？
当前无法通过云解决方案提供商 (CSP) 订阅创建预配置的解决方案。 但是，可以在几分钟内创建一个 [Azure 试用帐户][1rmb-trial]，以便创建预配置的解决方案。

### <a name="next-steps"></a>后续步骤
你还可以浏览 IoT 套件预配置的解决方案的一些其他特性和功能：

- [预测性维护预配置解决方案概述][lnk-predictive-overview]
- [从头开始保障 IoT 安全][lnk-security-groundup]

[lnk-predictive-overview]: /documentation/articles/iot-suite-predictive-overview/
[lnk-security-groundup]: /documentation/articles/securing-iot-ground-up/
[link-azuresublimits]: /documentation/articles/azure-subscription-service-limits/#iot-hub-limits
[lnk-azure-portal]: https://portal.azure.cn
[lnk-azureiotsuite]: https://www.azureiotsuite.cn/
[lnk-classic-portal]: https://manage.windowsazure.cn
[lnk-remote-monitoring-github]: https://github.com/Azure/azure-iot-remote-monitoring
[lnk-dreamspark]: https://www.dreamspark.com/Product/Product.aspx?productid=99
[1rmb-trial]: /pricing/1rmb-trial
[lnk-delete-aad-tennant]: http://blogs.msdn.com/b/ericgolpe/archive/2015/04/30/walkthrough-of-deleting-an-azure-ad-tenant.aspx
[lnk-cloud-deployment]: https://github.com/Azure/azure-iot-remote-monitoring/blob/master/Docs/cloud-deployment.md
[lnk-add-method]: /documentation/articles/iot-suite-guidance-on-customizing-preconfigured-solutions/#add-support-for-a-new-method-to-the-simulator
[lnk-customize]: /documentation/articles/iot-suite-guidance-on-customizing-preconfigured-solutions/
[lnk-remote-monitoring-github]: https://github.com/Azure/azure-iot-remote-monitoring
[lnk-predictive-maintenance-github]: https://github.com/Azure/azure-iot-predictive-maintenance

<!---HONumber=Mooncake_0327_2017-->
<!--Update_Description:update wording and add the section of CSP preconfigured solution-->