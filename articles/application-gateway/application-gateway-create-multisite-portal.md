<properties
    pageTitle="在 Azure 门户中配置托管多个站点的现有应用程序网关 | Azure"
    description="此页说明了如何通过 Azure 门户配置现有的 Azure 应用程序网关，以便在同一网关托管多个 Web 应用程序。"
    documentationcenter="na"
    services="application-gateway"
    author="georgewallace"
    manager="carmonm"
    editor="tysonn" />  

<tags
    ms.assetid="95f892f6-fa27-47ee-b980-7abf4f2c66a9"
    ms.service="application-gateway"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="11/16/2016"
    wacn.date="12/05/2016"
    ms.author="gwallace" />  


# 配置托管多个 Web 应用程序的现有应用程序网关
> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/application-gateway-create-multisite-portal/)
- [Azure Resource Manager PowerShell](/documentation/articles/application-gateway-create-multisite-azureresourcemanager-powershell/)

托管多个站点可以让你在同一应用程序网关上部署多个 Web 应用程序。系统会通过传入 HTTP 请求中存在的主机标头来确定接收流量的侦听器。然后，侦听器会根据网关规则定义中的配置将流量定向到适当的后端池。在启用了 SSL 的 Web 应用程序中，应用程序网关将根据服务器名称指示 (SNI) 扩展来选择 Web 流量的适当侦听器。通常会通过托管多个站点将不同 Web 域的请求负载均衡到不同的后端服务器池。同样还可以将同一根域的多个子域托管到同一应用程序网关。

## 方案
在以下示例中，应用程序网关使用两个后端服务器池来为 contoso.com 和 fabrikam.com 提供流量：contoso 服务器池和 fabrikam 服务器池。可以使用类似的设置来托管 app.contoso.com 和 blog.contoso.com 这样的子域。

![多站点方案][multisite]  


## 开始之前
此方案将多站点支持添加到现有应用程序网关。若要完成此方案，需要使用现有应用程序网关进行配置。请访问[使用门户创建应用程序网关](/documentation/articles/application-gateway-create-gateway-portal/)，了解如何在门户中创建基本的应用程序网关。

## 要求
* **后端服务器池：**后端服务器的 IP 地址列表。列出的 IP 地址应属于虚拟网络子网，或者是公共 IP/VIP。也可使用 FQDN。
* **后端服务器池设置：**每个池都有一些设置，例如端口、协议和基于 Cookie 的关联性。这些设置绑定到池，并会应用到池中的所有服务器。
* **前端端口：**此端口是应用程序网关上打开的公共端口。流量将抵达此端口，然后重定向到后端服务器之一。
* **侦听器：**侦听器具有前端端口、协议（Http 或 Https，这些值区分大小写）和 SSL 证书名称（如果要配置 SSL 卸载）。对于启用了多个站点的应用程序网关，还会添加主机名和 SNI 指示器。
* **规则：**规则将会绑定侦听器和后端服务器池，并定义流量抵达特定侦听器时应定向到的后端服务器池。
* **证书：**每个侦听器都需要唯一的证书。在此示例中，为多站点创建了 2 个侦听器。需要为侦听器创建两个 .pfx 证书和密码。

## 创建应用程序网关
以下是更新应用程序网关所需执行的步骤：

1. 创建用于每个站点的后端池。
2. 为应用程序网关会支持的每个站点创建新的侦听器。
3. 创建规则，为每个侦听器映射相应的后端。

## 为每个站点创建后端池
应用程序网关支持的每个站点都需要一个后端池。在此示例中，将创建 2 个后端池，一个用于 contoso11.com，另一个用于 fabrikam11.com。

### 步骤 1
在 Azure 门户 (https://portal.azure.cn) 中导航到现有的应用程序网关。选择“后端池”，单击“添加”

![添加后端池][7]  


### 步骤 2
填写后端池“pool1”的信息，为后端服务器添加 IP 地址或 FQDN，然后单击“确定”

![后端池 pool1 设置][8]  


### 步骤 3
在后端池边栏选项卡上单击“添加”以添加另一后端池“pool2”，为后端服务器添加 IP 地址或 FQDN，然后单击“确定”

![后端池 pool2 设置][9]  


### 为每个后端创建侦听器
应用程序网关需要使用 HTTP 1.1 主机标头才能在相同的公共 IP 地址和端口上托管多个网站。在门户中创建的基本侦听器不包含此属性。

### 步骤 1
单击现有应用程序网关上的“侦听器”，然后单击“多站点”添加第一个侦听器。

![侦听器概述边栏选项卡][1]  


### 步骤 2
填写侦听器的信息。此示例将配置 SSL 终止，创建新的前端端口。上载用于 SSL 终止的 .pfx 证书。此边栏选项卡与标准的基本侦听器边栏选项卡的唯一区别是主机名。

![侦听器属性边栏选项卡][2]  


### 步骤 3
根据上一步中针对第二个站点的说明，单击“多站点”并创建另一个侦听器。请确保对第二个侦听器使用不同的证书。此边栏选项卡与标准的基本侦听器边栏选项卡的唯一区别是主机名。填写侦听器的信息，然后单击“确定”。

![侦听器属性边栏选项卡][3]  


> [AZURE.NOTE]
在 Azure 门户中创建应用程序网关的侦听器是一项运行时间长的任务，可能需要一些时间才能在此方案中创建两个侦听器。完成时，侦听器会显示在门户中，如下图所示。
> 
> 

![侦听器概述][4]  


### 创建规则，将侦听器映射到后端池
### 步骤 1
在 Azure 门户 (https://portal.azure.cn) 中导航到现有的应用程序网关。选择“规则”，再选择现有的默认规则“rule1”，然后单击“编辑”。

### 步骤 2
填写规则边栏选项卡，如下图所示。选择第一个侦听器和第一个池，完成时单击“保存”。

![编辑现有规则][6]  


### 步骤 3
单击“基本规则”创建第二个规则。在窗体中填写第二个侦听器和第二个后端池，单击“确定”保存。

![添加基本规则边栏选项卡][10]  


通过 Azure 门户为现有应用程序网关配置多站点支持至此操作完毕。

## 后续步骤
通过[应用程序网关 - Web 应用程序防火墙](/documentation/articles/application-gateway-webapplicationfirewall-overview/)了解如何保护网站

<!--Image references-->

[1]: ./media/application-gateway-create-multisite-portal/figure1.png
[2]: ./media/application-gateway-create-multisite-portal/figure2.png
[3]: ./media/application-gateway-create-multisite-portal/figure3.png
[4]: ./media/application-gateway-create-multisite-portal/figure4.png
[5]: ./media/application-gateway-create-multisite-portal/figure5.png
[6]: ./media/application-gateway-create-multisite-portal/figure6.png
[7]: ./media/application-gateway-create-multisite-portal/figure7.png
[8]: ./media/application-gateway-create-multisite-portal/figure8.png
[9]: ./media/application-gateway-create-multisite-portal/figure9.png
[10]: ./media/application-gateway-create-multisite-portal/figure10.png
[multisite]: ./media/application-gateway-create-multisite-portal/multisite.png

<!---HONumber=Mooncake_1128_2016-->