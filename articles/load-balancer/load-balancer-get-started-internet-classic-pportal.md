<properties 
   pageTitle="开始使用 Azure 门户预览在经典部署模型中创建面向 Internet 的负载均衡器 | Azure"
   description="了解如何使用 Azure 门户预览在经典部署模型中创建面向 Internet 的负载均衡器"
   services="load-balancer"
   documentationCenter="na"
   authors="joaoma"
   manager="carolz"
   editor=""
   tags="azure-service-management"
/>
<tags  
   ms.service="load-balancer"
   ms.date="03/17/2016"
   wacn.date="08/29/2016" />

# 开始在 Azure 门户预览中创建面向 Internet 的负载均衡器（经典）

[AZURE.INCLUDE [load-balancer-get-started-internet-classic-selectors-include.md](../../includes/load-balancer-get-started-internet-classic-selectors-include.md)]

[AZURE.INCLUDE [load-balancer-get-started-internet-intro-include.md](../../includes/load-balancer-get-started-internet-intro-include.md)]

>[AZURE.IMPORTANT]在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：资源管理器部署模型和经典部署模型。在使用任何 Azure 资源之前，请确保你了解[部署模型和工具](/documentation/articles/azure-classic-rm/)。可以通过单击本文顶部的选项卡来查看不同工具的文档。本文介绍经典部署模型。你还可以[了解如何使用 Azure Resource Manager 创建面向 Internet 的负载均衡器](/documentation/articles/load-balancer-get-started-internet-arm-ps/)。

 
[AZURE.INCLUDE [load-balancer-get-started-internet-scenario-include.md](../../includes/load-balancer-get-started-internet-scenario-include.md)]



## 开始使用 Azure 门户预览创建负载均衡器终结点	

若要从 Azure 门户预览创建面向 Internet 的负载均衡器（经典）部署模型，请执行以下步骤。

1. 从浏览器导航到 http://portal.azure.cn，如有必要，请使用 Azure 帐户登录。

2. 转到“虚拟机(经典)”边栏选项卡 > 选择虚拟机。

3. 在虚拟机的“概要”边栏选项卡中，选择“所有设置”

4. 单击“负载平衡集”。

5. 若要创建新的负载均衡器，请单击负载平衡集边栏选项卡顶部的“加入”图标。

6. 选择对面向 Internet 的负载均衡器公开的“负载平衡集类型”。

7. 单击“配置所需的设置”以打开“选择负载平衡集”，然后单击“创建负载平衡集”。

8. 在“创建负载平衡集”边栏选项卡中，为负载均衡器集创建名称。填写名称、公用端口、探测协议、探测端口。

9. 如果需要，更改探测时间间隔和重试次数。

10. （可选）如果需要，可以在“创建负载均衡器集”边栏选项卡中配置访问控制规则。

11. 单击“确定”以返回到“加入负载平衡集”边栏选项卡。

12. 单击“确定”，并等待新的负载均衡器资源显示在“负载均衡器集”边栏选项卡中。
 
## 后续步骤

[开始配置内部负载均衡器](/documentation/articles/load-balancer-get-started-ilb-arm-ps/)

[配置负载均衡器分发模式](/documentation/articles/load-balancer-distribution-mode/)

[为负载均衡器配置空闲 TCP 超时设置](/documentation/articles/load-balancer-tcp-idle-timeout/)

<!---HONumber=Mooncake_0822_2016-->