<properties
  pageTitle="Azure IoT 套件常见问题 | Azure"
  description="有关 IoT 套件的常见问题"
  services=""
  suite="iot-suite"
  documentationCenter=""
  author="dominicbetts"
  manager="timlt"
  editor=""/>

<tags
  ms.service="iot-suite"
  ms.devlang="na"
  ms.topic="article"
  ms.tgt_pltfrm="na"
  ms.workload="na"
  ms.date="01/04/2017"
  wacn.date="03/03/2017"
  ms.author="corywink"/>

   
# 有关 IoT 套件的常见问题
### 在 Azure 门户预览中删除资源组与在 azureiotsuite.cn 中对预配置解决方案单击删除之间的区别是什么？

* 如果在 [azureiotsuite.cn][lnk-azureiotsuite] 中删除预配置解决方案，则会删除在创建预配置解决方案时设置的所有资源。如果向资源组添加了其他资源，则也会删除这些资源。

* 如果删除 [Azure 门户预览][lnk-azure-portal]中的资源组，则只会删除该资源组中的资源。此外还需在 [Azure 经典管理门户][lnk-classic-portal]中删除与预配置的解决方案关联的 Azure Active Directory 应用程序。

### 在一个订阅中可以设置多少个 IoT 中心实例？

每个订阅可以预配 10 个 IoT 中心。可以创建 [Azure 在线申请支持](/support/support-ticket-form/?l=zh-cn)创建工单，提出申请以提高此限制，但默认情况下，如 [Azure subscription limits][link-azuresublimits]（Azure 订阅限制）中所述，对每个订阅只能预配 10 个 IoT 中心。由于每个预配置的解决方案将预配一个新的 IoT 中心，因此，在给定的订阅中，最多只能预配 10 个预配置的解决方案。

### 在订阅中可以设置多少个 DocumentDB 实例？

50 个。可以[在线申请支持](/support/support-ticket-form/?l=zh-cn)创建工单以提高此限制，但在默认情况下，对每个订阅只能预配 50 个 DocumentDB 实例。



### 如果有 Azure for DreamSpark，是否可以创建预配置解决方案？
当前无法使用 [Azure for DreamSpark][lnk-dreamspark] 帐户创建预配置解决方案。但是，可以在几分钟内创建一个[试用帐户][1rmb-trial]，以便创建预配置的解决方案。



### 后续步骤
你还可以浏览 IoT 套件预配置的解决方案的一些其他特性和功能：

- [预见性维护预配置解决方案概述][lnk-predictive-overview]
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

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description:update wording and delete Bing Map API related content-->