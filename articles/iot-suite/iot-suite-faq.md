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
  ms.date="06/27/2016"
  wacn.date="09/05/2016"/>
   
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

### 我的远程监视解决方案部署具有静态地图，我如何添加交互式必应地图？ 
1. 从 [Azure 门户预览][lnk-azure-portal]获取用于企业的必应地图 API 查询密钥。
1. 在 [Azure 门户预览][lnk-azure-portal]中导航到用于企业的必应地图 API 所处的资源组。
2. 单击“所有设置”，然后单击“密钥管理”。
3. 你会注意到两个密钥：主密钥和查询密钥。复制查询密钥的值。

     > [AZURE.NOTE] 没有用于企业的必应地图 API 帐户？ 通过单击“+ 新建”、搜索用于企业的必应地图 API 并按照提示进行创建，在 [Azure 门户预览][lnk-azure-portal]中创建一个帐户。

2. 从 [Azure-IoT-Remote-Monitoring][lnk-remote-monitoring-github] 下拉最新代码。

3. 在存储库的 /docs/ 文件夹中按照命令行部署指南运行本地或云部署。

4. 运行了本地或云部署之后，在根文件夹中查找在部署过程中创建的 *.user.config 文件。在文本编辑器中打开此文件。

5. 更改以下行，以包含为查询密钥复制的值：
   
  `<setting name="MapApiQueryKey" value="" />`


### 如果我具有 Azure for DreamSpark，是否可以创建预配置解决方案？
当前无法使用 [Azure for DreamSpark][lnk-dreamspark] 帐户创建预配置解决方案。但是，你可以在几分钟内创建一个[试用帐户][1rmb-trial]，以便创建预配置的解决方案。



## 后续步骤

你还可以浏览 IoT 套件预配置的解决方案的一些其他特性和功能：

- [从头开始保障 IoT 安全][lnk-security-groundup]

[lnk-predictive-overview]: /documentation/articles/iot-suite/iot-suite-predictive-overview/
[lnk-security-groundup]: /documentation/articles/iot-suite/securing-iot-ground-up/
[link-azuresublimits]: /documentation/articles/azure-subscription-service-limits/#iot-hub-limits
[lnk-azure-portal]: https://portal.azure.cn
[lnk-azureiotsuite]: https://www.azureiotsuite.cn/
[lnk-classic-portal]: https://manage.windowsazure.cn
[lnk-remote-monitoring-github]: https://github.com/Azure/azure-iot-remote-monitoring
[lnk-dreamspark]: https://www.dreamspark.com/Product/Product.aspx?productid=99
[1rmb-trial]: /pricing/1rmb-trial
[lnk-delete-aad-tennant]: http://blogs.msdn.com/b/ericgolpe/archive/2015/04/30/walkthrough-of-deleting-an-azure-ad-tenant.aspx

<!---HONumber=Mooncake_0815_2016-->