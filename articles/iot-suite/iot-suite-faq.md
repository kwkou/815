<properties
  pageTitle="Azure IoT 套件常见问题 | Azure"
  description="有关 IoT 套件的常见问题"
  services=""
  suite="iot-suite"
  documentationCenter=""
  authors="aguilaaj"
  manager="timlt"
  editor=""/>  


<tags
  ms.service="iot-suite"
  ms.devlang="na"
  ms.topic="article"
  ms.tgt_pltfrm="na"
  ms.workload="na"
  ms.date="09/26/2016"
  ms.author="araguila"
  wacn.date="10/31/2016"/>

   
# 有关 IoT 套件的常见问题

### 在 Azure 门户预览中删除资源组与在 azureiotsuite.cn 中删除预配置解决方案有什么区别？

- 如果在 [azureiotsuite.cn][lnk-azureiotsuite] 中删除预配置解决方案，则会删除在创建该解决方案时设置的所有资源；如果向资源组添加了其他资源，则也会删除这些资源。

- 如果在 [Azure 门户预览][lnk-azure-portal]中删除某个资源组，则只删除该资源组中的资源；还需要在 [Azure 经典管理门户][lnk-classic-portal]中删除与预配置解决方案关联的 Azure Active Directory 应用程序。

### 一个订阅中可以设置多少个 IoT 中心实例？ 

10 个。如需要设置更多，请通过 Azure [在线申请支持](/support/support-ticket-form/?l=zh-cn)创建工单，提出申请。默认情况下，如 [Azure subscription limits][link-azuresublimits]（Azure 订阅上限）中所述，对每个订阅只能预配 10 个 IoT 中心。由于每个预配置的解决方案只能预配一个新的 IoT 中心，因此，在给定的订阅中，最多只能预配 10 个该解决方案。

### 在订阅中可以设置多少个 DocumentDB 实例？

50 个。可以[在线申请支持](/support/support-ticket-form/?l=zh-cn)创建工单以提高此限制，但是在默认情况下，对每个订阅只能设置 50 个 DocumentDB 实例。

### 在订阅中可以设置多少个免费必应地图 API？

两个。在一个 Azure 订阅中，只能创建两个面向企业的内部事务级别 1 必应地图计划。默认情况下，远程监视解决方案是使用内部事务级别 1 计划预配的。因此，如果不进行修改，则在订阅中只能设置最多两个远程监视解决方案。


### 如果我具有 Azure for DreamSpark，是否可以创建预配置解决方案？
当前无法使用 [Azure for DreamSpark][lnk-dreamspark] 帐户创建预配置解决方案。但是，你可以在几分钟内创建一个[试用帐户][1rmb-trial]，以便创建预配置的解决方案。



## 后续步骤

你还可以浏览 IoT 套件预配置的解决方案的一些其他特性和功能：

- [从头开始保障 IoT 安全][lnk-security-groundup]

[lnk-security-groundup]: /documentation/articles/securing-iot-ground-up/
[link-azuresublimits]: /documentation/articles/azure-subscription-service-limits/#iot-hub-limits
[lnk-azure-portal]: https://portal.azure.cn
[lnk-azureiotsuite]: https://www.azureiotsuite.cn/
[lnk-classic-portal]: https://manage.windowsazure.cn
[lnk-remote-monitoring-github]: https://github.com/Azure/azure-iot-remote-monitoring
[lnk-dreamspark]: https://www.dreamspark.com/Product/Product.aspx?productid=99
[1rmb-trial]: /pricing/1rmb-trial
[lnk-delete-aad-tennant]: http://blogs.msdn.com/b/ericgolpe/archive/2015/04/30/walkthrough-of-deleting-an-azure-ad-tenant.aspx

<!---HONumber=Mooncake_0815_2016-->