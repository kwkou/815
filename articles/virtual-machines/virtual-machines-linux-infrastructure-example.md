<properties
	pageTitle="示例基础结构演练 | Azure"
	description="了解用于在 Azure 中部署示例基础结构的关键设计和实施准则。"
	documentationCenter=""
	services="virtual-machines-linux"
	authors="iainfoulds"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="06/24/2016"
	wacn.date="08/08/2016"/>

# 示例 Azure 基础结构演练

[AZURE.INCLUDE [virtual-machines-linux-infrastructure-guidelines-intro](../../includes/virtual-machines-linux-infrastructure-guidelines-intro.md)]

本文将逐步讲述如何构建示例应用程序基础结构。我们将详细介绍如何设计简单在线商店的基础结构，此在线商店可将关于命名约定、可用性集、虚拟网络及负载平衡器的所有准则和决策聚集在一起；以及如何实际部署你的虚拟机 (VM)。


## 示例工作负荷

Adventure Works Cycles 想要在 Azure 中生成一个在线商店应用程序，该应用程序将包含：

- 两个在 Web 层中运行客户端前端的 nginx 服务器
- 两个在应用程序层中处理数据和订单的 nginx 服务器
- 包含在分片群集中的两个 MongoDB 服务器，用于在数据库层中存储产品数据和订单
- 位于身份验证层中、用于客户帐户和供应商的两个 Active Directory 域控制器
- 所有服务器皆位于两个子网中：
	- Web 服务器位于前端子网中
	- 应用程序服务器、MongoDB 群集和域控制器位于后端子网中

![不同应用程序基础结构层的关系图](./media/virtual-machines-common-infrastructure-service-guidelines/example-tiers.png)

当客户浏览在线商店时，传入的安全 Web 流量需要在 Web 服务器之间进行负载平衡。来自 Web 服务器的 HTTP 请求形式的订单处理流量需要在应用程序服务器之间进行平衡。此外，基础结构必须设计为具有高可用性。

生成的设计必须引入：

- Azure 订阅和帐户
- 单个资源组
- 存储帐户
- 包含两个子网的虚拟网络
- 具有类似角色的 VM 的可用性集
- 虚拟机

以上各项都将遵循以下命名约定：

- Adventure Works Cycles 使用 **[IT 工作负荷]-[位置]-[Azure 资源]** 作为前缀
	- 在本示例中，IT 工作负荷名为 **azos**（Azure On-line Store，Azure 在线商店），位置为 **che**（China East，中国东部）是
- 存储帐户使用 adventureazoschesa**[描述]**
	- 请注意，“adventure”已添加到前缀以提供唯一性，并且存储帐户名称不支持使用连字符。
- 虚拟网络使用 AZOS-CHE-VN**[数字]**
- 可用性集使用 use azos-che-as-**[角色]**
- 虚拟机名称使用 azos-che-vm-**[VM 名称]**


## Azure 订阅和帐户

Adventure Works Cycles 使用名为 Adventure Works 企业订阅的企业订阅来提供此 IT 工作负荷的计费。


## 存储帐户

Adventure Works Cycles 确定他们需要以下两个存储帐户：

- **adventureazoschesawebapp** 用于 Web 服务器、应用程序服务器和域控制器及其数据磁盘的标准存储。
- **adventureazoschesadbclust** 用于 MongoDB 分片群集服务器及其数据磁盘的高级存储。


## 虚拟网络和子网

由于虚拟网络不需要持续连接到 Adventure Work Cycles 本地网络，因此，他们决定选择仅限云的虚拟网络。

他们通过 Azure 门户预览使用以下设置创建了仅限云的虚拟网络：

- 名称：AZOS-CHE-VN01
- 位置：中国东部
- 虚拟网络地址空间：10.0.0.0/8
- 第一个子网：
	- 名称：FrontEnd
	- 地址空间：10.0.1.0/24
- 第二个子网：
	- 名称：BackEnd
	- 地址空间：10.0.2.0/24


## 可用性集

为了维护其在线商店的所有四个层的高可用性，Adventure Works Cycles 决定使用四个可用性集：

- **azos-che-as-web** 用于 Web 服务器
- **azos-che-as-app** 用于应用程序服务器
- **azos-che-as-db** 用于 MongoDB 分片群集中的服务器
- **azos-che-as-dc** 用于域控制器


## 虚拟机

Adventure Works Cycles 决定为其 Azure VM 使用以下名称：

- **azos-che-vm-web01** 用于第一个 Web 服务器
- **azos-che-vm-web02** 用于第二个 Web 服务器
- **azos-che-vm-app01** 用于第一个应用程序服务器
- **azos-che-vm-app02** 用于第二个应用程序服务器
- **azos-che-vm-db01** 用于群集中的第一个 MongoDB 服务器
- **azos-che-vm-db02** 用于群集中的第二个 MongoDB 服务器
- **azos-che-vm-dc01** 用于第一个域控制器
- **azos-che-vm-dc02** 用于第二个域控制器

这是生成的配置。

![在 Azure 中部署的最终应用程序基础结构](./media/virtual-machines-common-infrastructure-service-guidelines/example-config.png)

此配置引入以下项：

- 包含两个子网（FrontEnd 和 BackEnd）的仅限云虚拟网络
- 两个存储帐户
- 四个可用性集，每个在线商店层一个
- 四个层中的虚拟机
- 用于从 Internet 到 Web 服务器的基于 HTTPS 的 Web 流量的外部负载平衡集
- 用于从 Web 服务器到应用程序服务器的未加密 Web 流量的内部负载平衡集
- 单个资源组


## <a name="next-steps"></a> 后续步骤

[AZURE.INCLUDE [virtual-machines-linux-infrastructure-guidelines-next-steps](../../includes/virtual-machines-linux-infrastructure-guidelines-next-steps.md)]

<!---HONumber=Mooncake_0801_2016-->