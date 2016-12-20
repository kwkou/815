<properties
    pageTitle="DocumentDB 防火墙支持 | Azure"
    description="了解如何在 Azure DocumentDB 数据库帐户的防火墙支持中使用 IP 访问控制策略。"
    keywords="IP 访问控制, 防火墙支持"
    services="documentdb"
    author="shahankur11"
    manager="jhubbard"
    editor=""
    tags="azure-resource-manager"
    documentationcenter="" />  

<tags
    ms.assetid="c1b9ede0-ed93-411a-ac9a-62c113a8e887"
    ms.service="documentdb"
    ms.workload="data-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/17/2016"
    wacn.date="12/20/2016"
    ms.author="ankshah; kraman" />  


# DocumentDB 防火墙支持
为了保护存储在 Azure DocumentDB 数据库帐户中的数据，DocumentDB 支持基于密码的[授权方式](https://msdn.microsoft.com/zh-cn/library/azure/dn783368.aspx)，该方式利用基于哈希的强消息身份验证代码 (HMAC)。目前，除了基于密码的授权方式，DocumentDB 还支持用于入站防火墙支持的 IP 的策略驱动型访问控制。此方式非常类似于传统数据库系统的防火墙规则，为 DocumentDB 数据库帐户提供了额外的安全级别。使用此方式，现在可以配置仅批准的一组计算机和/或云服务可以访问 DocumentDB 数据库帐户。这些已批准的计算机和服务对 DocumentDB 资源的访问仍要求调用方提供有效的授权令牌。

## IP 访问控制概览
默认情况下，只要请求附带有效的授权令牌，就可以从公共 Internet 访问 DocumentDB 数据库帐户。若要配置基于 IP 策略的访问控制，用户必须提供 IP 地址集合或者 CIDR 格式的 IP 地址范围，以便将这些地址作为指定数据库帐户的允许的客户端 IP 列表。应用此配置后，服务器将阻止从该允许的列表以外的计算机发出的所有请求。下图描述了基于 IP 的访问控制的连接处理流程

![显示基于 IP 的访问控制的连接过程的关系图](./media/documentdb-firewall-support/documentdb-firewall-support-flow.png)  


## 从云服务进行连接
在 Azure 中，云服务是一种很常见托管使用 DocumentDB 的中间层服务逻辑的方法。若要从云服务访问 DocumentDB 数据库帐户，则必须[与 Azure 支持部门联系](#configure-ip-policy)将云服务的公共 IP 地址添加到与 DocumentDB 数据库帐户关联的允许的 IP 地址列表中。这样可以确保云服务的所有角色实例都可以访问 DocumentDB 数据库帐户。如下面的屏幕截图所示，可以在 Azure 门户预览中检索云服务的 IP 地址。

![显示在 Azure 门户预览中显示的云服务的公共 IP 地址的屏幕截图](./media/documentdb-firewall-support/documentdb-public-ip-addresses.png)  


当你添加其他角色实例以扩大云服务时，这些新的实例自动具有 DocumentDB 数据库帐户的访问权限，因为它们属于相同的云服务。

## 从虚拟机进行连接
[虚拟机](/home/features/virtual-machines/)或[虚拟机规模集](/documentation/articles/virtual-machine-scale-sets-overview/)也可以用于托管使用 DocumentDB 的中间层服务。若要将 DocumentDB 数据库帐户配置为允许从虚拟机访问，则必须[与 Azure 支持部门联系](#configure-ip-policy)将虚拟机和/或虚拟机规模集的公共 IP 地址配置为 DocumentDB 数据库帐户允许的 IP 地址之一。如下面的屏幕截图所示，可以在 Azure 门户预览中检索虚拟机的 IP 地址。

![显示在 Azure 门户预览中显示的虚拟机的公共 IP 地址的屏幕截图](./media/documentdb-firewall-support/documentdb-public-ip-addresses-dns.png)  


将其他虚拟机实例添加到组中时，它们自动具有 DocumentDB 数据库帐户的访问权限。

## 从 Internet 进行连接
从 Internet 上的计算机访问 DocumentDB 数据库帐户时，必须将此计算机的客户端 IP 地址或 IP 地址范围添加到 DocumentDB 数据库帐户的允许的 IP 地址列表中。

## <a id="configure-ip-policy"></a> 配置 IP 访问控制策略
在 Azure 门户预览中通过 [Azure 支持](https://portal.azure.cn/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)提交请求，以便对数据库帐户启用 IP 访问控制策略。

1. 在“帮助 + 支持”边栏选项卡上，选择“新建支持请求”。[](https://portal.azure.cn/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)
2. 在“新支持请求”边栏选项卡上，选择“基本”。
3. 在“基本”边栏选项卡中选择以下设置：
   - **问题类型**：配额
   - **订阅**：与要添加 IP 访问控制策略的帐户关联的订阅。
   - **配额类型**：DocumentDB
   - **支持计划**：配额支持 - 包括。
4. 在“问题”边栏选项卡中执行以下操作：
   - **严重性**：选择 C - 最小的影响
   - **详细信息**：将以下文本复制到框中，并包含帐户名称和 IP 地址：“我想要为我的 DocumentDB 数据库帐户启用防火墙支持。数据库帐户： *包含帐户名称*。允许的 IP 地址/范围：*包括 CIDR 格式的 IP 地址/范围，例如 13.91.6.132，13.91.6.1/24*。”
   - 单击“下一步”。
5. 在“联系信息”边栏选项卡中输入详细联系信息，然后单击“创建”。

一旦收到你的请求，将在 24 小时内启用 IP 访问控制。请求完成后就会通知你。

![“帮助 + 支持”边栏选项卡的屏幕截图](./media/documentdb-firewall-support/documentdb-firewall-support-request-access.png)  


![“问题”边栏选项卡的屏幕截图](./media/documentdb-firewall-support/documentdb-firewall-support-request-access-ticket.png)  


## IP 访问控制策略的故障排除
### 门户操作
为 DocumentDB 数据库帐户启用 IP 访问控制策略后，从配置的允许的 IP 地址范围列表以外的计算机对 DocumentDB 数据库帐户的所有访问都被阻止。此方式还阻止在门户中浏览数据平面操作，以确保访问控制的完整性。

### SDK 和 Rest API
出于安全的原因，从不在允许的列表中的计算机通过 SDK 或 REST API 访问时，返回一个常规的“404 找不到”响应，并且不包含其他详细信息。请确认为 DocumentDB 数据库帐户配置的允许的 IP 列表，以确保将正确的策略配置应用到 DocumentDB 数据库帐户。

## 后续步骤
有关与网络相关的性能提示，请参阅[性能提示](/documentation/articles/documentdb-performance-tips/)。

<!---HONumber=Mooncake_1212_2016-->
