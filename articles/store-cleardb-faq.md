<properties
	pageTitle="ClearDB MySql 数据库搭配 Azure App Service 的 FAQ | Azure"
	description="解答将 ClearDB MySQL 数据库与 Azure App Service 搭配使用时的常见问题。"
	documentationCenter="php"
	services=""
	authors="sunbuild"
	manager="yochayk"
	editor=""
	tags="mysql"/>

<tags
	ms.service="multiple"
	ms.date="02/08/2016"
	ms.author="sumuth"/>

# ClearDB MySql 数据库搭配 Azure App Service 的 FAQ

此 FAQ 解答了为 Azure Web Apps 使用和购买 ClearDB MySQL 数据库的常见问题。

## Azure 上的 MySQL 有哪些选项？

你有几种选项：

* [ClearDB 共享 MySQL 数据库](/marketplace/partners/cleardb/databases/)

* [ClearDB MySQL Premium 群集](/marketplace/partners/cleardb-clusters/cluster/)

* [Azure VM 上运行的 MySQL 群集](https://github.com/azure/azure-quickstart-templates/tree/master/mysql-replication)

* [Azure VM 上运行的单个 MySQL 实例](virtual-machines/virtual-machines-windows-classic-mysql-2008r2.md)

ClearDB 是一种 MySQL 托管服务，可为你管理 MySQL 基础结构。在 Azure 虚拟机上运行自己的 MySQL 群集或数据库时，必须设置 MySQL 服务器，并使用补丁让它保持更新。

## 要获取 Azure 应用商店中的 Web 应用和 MySQL 模板需要使用信用卡吗？

这取决于你使用的订阅类型。以下是一些常用的订阅类型：

* [即用即付](/offers/ms-azr-0003p/)：需要信用卡，购买付费的 MySQL 数据库时，将向你的信用卡收费。

* [免费试用版](/pricing/free-trial/)：包括可用于 Azure 服务的信用额度，但不允许购买第三方资源。若要购买第三方服务或付费的 MySQL 数据库，需使用支持信用卡的订阅。针对 Web Apps，可以创建免费的 ClearDB MySQL 数据库。

* [MSDN 订阅](/pricing/member-offers/msdn-benefits/)和 **MSDN 即用即付开发/测试**：与免费试用版类似，MSDN 订阅要求使用信用卡向 ClearDB 购买付费的 MySQL 解决方案。

* [企业协议 (EA)](/pricing/enterprise-agreement/)：每季以单独的合并发票就 EA 向 EA 客户的所有 Azure 应用商店（第三方）购买商品收费。对于任何应用商店购买商品，将向你就货币承诺付款以外收费。请注意，Azure 应用商店目前无法供在阿塞拜疆、克罗地亚、挪威和波多黎各注册的客户使用。

*   [DreamSpark](https://www.dreamspark.com/Product/Product.aspx?productid=99)：只能为 Web Apps 创建免费的 ClearDB 数据库。可以创建的免费 ClearDB MySQL 数据库数目没有任何限制。请注意，免费数据库不适用于生产 Web Apps，因为此服务仅供试用。

## 为什么我要为 Azure 应用商店中的 Web 应用和 MySQL 付 3.50 美元的费用？

默认数据库选项是 Titan，需 3.50 美元。创建数据库时我们不会显示成本，你可能会错误地购买不想要的数据库。我们正在设法改善此体验，但在那之前，你需要先检查为 Web 应用和数据库选择的定价层，然后再单击“创建”并开始部署资源。

## 我在自己的 Azure 虚拟机上运行 MySQL。我是否可以将 Azure Web 应用连接到数据库？

是的。只要 Azure VM 已向 Web 应用授予远程访问权限，Web 应用就能连接到数据库。有关详细信息，请参阅[在虚拟机上安装 MySQL](virtual-machines/virtual-machines-windows-classic-mysql-2008r2.md)。

## 支持 ClearDB Premium MySQL 群集的国家/地区有哪些？

除印度、澳大利亚、巴西南部和中国之外的所有 Azure 区域都可以使用 [ClearDB Premium MySQL 群集](/marketplace/partners/cleardb-clusters/cluster/)。

## 是否可以在创建数据库之前使用 ClearDB Premium 群集解决方案创建新群集？

否，不支持创建空的 ClearDB 群集。Azure 门户可让你在群集中创建数据库，这可能会同时创建新的群集。

## 如果我尝试删除正由我的某个应用程序使用的 ClearDB MySQL 数据库，会收到警告吗？

不会，如果删除应用程序所依赖的应用商店购买商品，Azure 不会发出警告。

## 可以在哪些区域创建 ClearDB 数据库？

Azure 应用商店无法供在阿塞拜疆、克罗地亚、挪威或波多黎各注册的客户使用。这些区域不提供 ClearDB。

## 针对生产 Web 应用和数据库，应该选择哪个定价层？

对 Web Apps 使用“基本”或更高的定价层。对于 ClearDB，建议使用 Saturn 或 Jupiter 计划。请查看 [Web Apps](/pricing/details/app-service/) 和 [ClearDB MySQL 数据库](/marketplace/partners/cleardb/databases/)每个定价层的功能和限制，以选择符合需要的定价层。

## 如何将 ClearDB 数据库从一个计划升级到另一个计划？

可以使用 [ClearDB 升级向导](https://www.cleardb.com/store/azure/upgrade)。目前我们在 Azure 门户中没有升级路径。

## 在 Azure 门户中看不到我的 ClearDB 数据库？

如果我们使用 Azure Resource Manager 或[新的 Azure 门户](https://portal.azure.com)创建 ClearDB 数据库，将无法在[旧的 Azure 门户](https://manage.windowsazure.com)中看到此数据库。若要解决此问题，请将数据库手动链接至 Web 应用。同样地，如果在[旧门户](https://manage.windowsazure.com)中创建 ClearDB 数据库，则无法在[新的 Azure 门户](https://portal.azure.com)中看到此数据库。对于后一种情况，没有任何解决之道。

## 数据库关闭时应联系谁寻求支持？

如有任何数据库相关的问题，请联系 [ClearDB 支持人员](https://www.cleardb.com/developers/help/support)。准备好向其提供你的 Azure 订阅信息。

## 我是否可以为自己的 ClearDB MySQL 数据库群集解决方案创建其他用户？  

否。你无法创建其他用户，但可以在自己的 ClearDB 数据库群集上创建其他数据库。

## 是否可以像 ClearDB 门户上目前的 Planetary 计划一样，就地升级 Basic/Pro 系列数据库？

是，可以就地升级 Basic 系列数据库（Basic 60 到 Basic 500）。可以就地升级 Pro 系列（Pro 125 到 Pro 1000），但 Pro 60 除外。我们目前不支持升级 Pro 60 数据库。

## 当我将资源从一个订阅迁移到另一个订阅时，我的 ClearDB MySQL 数据库也会跟着迁移吗？  

当跨订阅执行资源迁移时，存在某些[限制](./app-service-web/app-service-move-resources.md)。ClearDB MySQL 数据库是第三方服务，因此在 Azure 订阅迁移期间不会迁移。如果在迁移 Azure 资源之前，没有管理 MySQL 数据库的迁移，你的 ClearDB MySQL 数据库可能会被禁用。请先手动迁移数据库，然后再为 Web 应用执行 Azure 订阅迁移。

## 是否可以使用企业协议 (EA) 订阅购买可缩放的 WordPress？

此过程对任何订阅都是相同的。请转到 [Azure 门户](https://portal.azure.com/)中的 Azure 应用商店，然后选择[可缩放的 WordPress](https://portal.azure.com/?feature.customportal=false#create/WordPress.ScalableWordPress) 以便开始创建该应用。可缩放的 WordPress 仅支持 ClearDB Saturn 和 Jupiter 定价层，而且你的 EA 信用额度将用于标准 Web Apps 定价层上运行的 Web 应用程序，以及付费的 ClearDB（共享）MySQL 数据库。[/marketplace/faq/](/marketplace/faq/) 每季将以单独的合并发票就 EA 向你的所有应用商店购买商品收费。

## 我是否可以将 ClearDB 数据库从信用卡订阅转为 EA 订阅？

现有 ClearDB 数据库将使用与现有订阅关联的信用卡。若要使用 EA 订阅，你需要将数据迁移到新数据库：

* 使用 EA 订阅购买新的 ClearDB 数据库。
* 将数据迁移至新数据库。
* 更新应用程序以使用新数据库。
* 删除旧的 ClearDB 数据库。

使用 MySQL (ClearDB) 创建新的 Web 应用，或创建 MySQL 数据库 (ClearDB) 时，所选择的订阅将决定服务的支付方式。请注意，使用 EA 订阅时，我们不会阻止在 Azure 门户中采购第三方服务，例如 ClearDB。对于 EA 订阅，将于每季季末就货币承诺付款以外收费。EA 客户必须设置付款方式，例如信用卡，才能支付任何第三方应用商店服务。

## 在哪里可以查看 EA 订阅中 ClearDB 资源的费用？

直接 EA 客户可以在企业门户上看到 Azure 应用商店费用。请注意，所有应用商店购买和使用均会在每季季末就货币承诺付款以外收费。EA 客户需要直接付费给第三方服务提供商，并且能够通过启用其 EA 帐户的付款方式（例如信用卡）来完成。

间接 EA 客户可以在企业门户的“管理订阅”页上找到自己的 Azure 应用商店订阅，但定价是隐藏的。客户应该联系其 LSP 以了解应用商店费用的相关信息。

EA Azure 注册管理员可以管理对 Azure 应用商店第三方服务的访问权限。他们可以通过企业门户中“帐户”部分下的“管理帐户”和“订阅”，禁用或重新启用对从应用商店购买的第三方商品的访问权限。

## 如果对 EA 订阅中的 ClearDB 服务帐单有疑问，应该联系谁？

有关根据 EA 注册计费的问题，请联系[企业客户支持人员](http://aka.ms/AzureEntSupport)。EA 门户支持团队会回答你的问题或帮你解决问题。

 



## 详细信息

[Azure 应用商店 FAQ](/marketplace/faq/)

<!---HONumber=Mooncake_0711_2016-->