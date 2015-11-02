
<properties linkid="dev-net-Networking" urlDisplayName="Windows Azure Networking" pageTitle="虚拟网络 - Azure 微软云" metaKeywords="Networking,虚拟网络,托管应用,数据中心,虚拟机,站点到站点VPN,名称解析,Azure 门户" description="在 Azure 中配置和监视虚拟网络。使用虚拟网络可将云基础结构连接到本地数据中心、连接在混合环境中托管的云应用程序以及连接 Azure 中的开发计算机和虚拟机。" metaCanonical="" services="Networking" documentationCenter="Services" title="Configure and monitor virtual networks in Azure" authors="" solutions="" manager="" editor="" />
<tags ms.service="Networking"
    ms.date=""
    wacn.date="07/23/2015"
    />



#虚拟网络

####在 Azure 中配置和监视虚拟网络</h4>

使用虚拟网络可将云基础结构连接到本地数据中心、连接在混合环境中托管的云应用程序以及连接 Azure 中的开发计算机和虚拟机。

####快速链接

-   [服务概述](/home/features/networking)
-   [定价详细信息](/home/features/networking/#price)
      
####特色

-   [虚拟网络技术概述](http://msdn.microsoft.com/zh-cn/library/azure/jj156007.aspx)
-   <!--[-->Azure 订阅和服务限制、配额和约束条件<!--](/zh-cn/documentation/articles/azure-subscription-service-limits)-->

##教程和指南

###探究

####[使用管理门户向导配置站点到站点 VPN](/documentation/articles/virtual-networks-configure-vnet-to-vnet-connection)

本教程将指导你完成创建 Azure 虚拟网络的步骤。完成本教程后，您将拥有一个虚拟网络，可将 Azure 服务部署到该网络。

####[Azure 虚拟网络概述](http://msdn.microsoft.com/zh-cn/library/azure/jj156007.aspx)

帮助您开始使用虚拟网络的技术库指南。了解虚拟网络的网络配置、VPN 设备和地缘组，并了解如何监视网络流量。

###计划

<!--####[Azure 名称解析](/documentation/articles/virtual-networks-name-resolution-for-vms-and-role-instances
)

了解如何通过此相关技术使用 Azure 提供的名称解析服务，直接按照主机名连接到云服务中的虚拟机和角色实例。-->

####[Azure 架构参考](https://msdn.microsoft.com/zh-CN/library/azure/dd179398)      

网络配置文件中使用的架构的文档，可用于指定虚拟网络配置设置。</p>

####[了解网络安全性的基本知识](http://go.microsoft.com/fwlink/p/?linkid=389558&clcid=0x804)

本白皮书概述了如何使用 Azure 网络内置的可配置安全功能。      

###管理

####[关于在管理门户中配置虚拟网络](/documentation/articles/virtual-networks-create-vnet)

本教程将指导您完成创建连接到公司网络的虚拟网络的步骤。

####[使用网络配置文件配置虚拟网络](/documentation/articles/virtual-networks-using-network-configuration-file)

用于创建地缘组、导入配置文件、更改网络属性和配置网关的过程。  

####[用户定义的路由和 IP 转发概述](/documentation/articles/virtual-networks-udr-overview)  
将虚拟机 (VM) 添加到 Azure 中同一虚拟网络 (VNet) 的不同子网时，你会注意到，VM 可以与其他 VM 通过网络自动通信。你不需要指定网关，即使这些 VM 位于不同子网中。存在从 Azure 到你自己的数据中心的混合连接时，这同样适用于从 VM 到公共 Internet 甚至到本地网络的通信。  
####[如何在 Azure 中创建路由并启用 IP 转发](/documentation/articles/virtual-networks-udr-how-to)  
了解如何管理 UDR 和 IP 转发


