<properties
    pageTitle="将公司 Internet 域指向流量管理器域名 | Azure"
    description="本文将帮助你将公司域名指向流量管理器域名。"
    services="traffic-manager"
    documentationCenter=""
    authors="sdwheeler"
    manager="carmonm"
    editor=""
/>  

<tags
    ms.service="traffic-manager"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="10/11/2016"
    wacn.date=""
    ms.author="sewhee"
/>  


# 将公司 Internet 域指向 Azure 流量管理器域

创建流量管理器配置文件时，Azure 会自动为该配置文件分配一个 DNS 名称。若要使用 DNS 区域中的某个名称，请创建一条映射到流量管理器配置文件域名的 CNAME DNS 记录。可以在流量管理器配置文件的“配置”页上的“常规”部分中找到流量管理器域名。

例如，若要将名称 www.contoso.com 指向流量管理器 DNS 名称 contoso.trafficmanager.cn，请创建以下 DNS 资源记录：

    www.contoso.com IN CNAME contoso.trafficmanager.cn

对 *www.contoso.com* 发出的所有流量请求将定向到 *contoso.trafficmanager.cn* 。

>[AZURE.IMPORTANT] 无法将第二级域（例如 *contoso.com* ）指向流量管理器域。DNS 协议标准不允许对二级域名使用 CNAME 记录。

## 后续步骤

[流量管理器路由方法](/documentation/articles/traffic-manager-routing-methods/)

[流量管理器 - 禁用、启用或删除配置文件](/documentation/articles/disable-enable-or-delete-a-profile/)

[流量管理器 - 禁用或启用终结点](/documentation/articles/disable-or-enable-an-endpoint/)

<!---HONumber=Mooncake_1031_2016-->