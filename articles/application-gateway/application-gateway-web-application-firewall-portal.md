<properties
   pageTitle="使用门户创建具有 Web 应用程序防火墙的应用程序网关 | Azure"
   description="了解如何使用门户创建具有 Web 应用程序防火墙的应用程序网关"
   services="application-gateway"
   documentationCenter="na"
   authors="georgewallace"
   manager="carmonm"
   editor="tysonn"
   tags="azure-resource-manager"
/>  

<tags  
   ms.service="application-gateway"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="09/26/2016"
   wacn.date="11/07/2016"
   ms.author="gwallace" />  


# 使用门户创建具有 Web 应用程序防火墙的应用程序网关

> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/application-gateway-web-application-firewall-portal/)
- [Azure Resource Manager PowerShell](/documentation/articles/application-gateway-web-application-firewall-powershell/)

Azure 应用程序网关是第 7 层负载均衡器。它在不同服务器之间提供故障转移和性能路由 HTTP 请求，而不管它们是在云中还是本地。应用程序网关提供许多应用程序传送控制器 (ADC) 功能，包括 HTTP 负载均衡、基于 cookie 的会话相关性、安全套接字层 (SSL) 卸载、自定义运行状况探测、多站点支持，以及许多其他功能。若要查找支持的功能的完整列表，请参阅[应用程序网关概述](/documentation/articles/application-gateway-introduction/)

Azure 应用程序网关中的 Web 应用程序防火墙 (WAF) 可保护 Web 应用程序，使其免受常见 Web 攻击的威胁，例如 SQL 注入、跨站点脚本攻击和会话劫持。

## <a name="scenario"></a> 方案

本文介绍两个方案：

在第一个方案中，可了解如何[将 Web 应用程序防火墙添加到现有的应用程序网关](#add-web-application-firewall-to-an-existing-application-gateway)。

在第二个方案中，可了解如何[创建具有 Web 应用程序防火墙的应用程序网关](#create-an-application-gateway-with-web-application-firewall)

![方案示例][scenario]  


>[AZURE.NOTE] 针对应用程序网关进行的其他配置（包括自定义运行状况探测、后端池地址以及其他规则）是在对应用程序网关配置以后配置的，不是在初始部署期间配置的。

## 开始之前

Azure 应用程序网关需要自己的子网。在创建虚拟网络时，请确保保留足够的地址空间，以便设置多个子网。将应用程序网关部署到子网后，只能向该子网添加其他应用程序网关。

## <a name="add-web-application-firewall-to-an-existing-application-gateway"></a> 将 Web 应用程序防火墙添加到现有的应用程序网关

此方案会更新现有的应用程序网关，以在阻止模式下支持 Web 应用程序防火墙。

### 步骤 1

导航到 Azure 门户预览，然后选择现有的应用程序网关。

![创建应用程序网关][1]  


### 步骤 2

单击“配置”并更新应用程序网关设置。完成后，单击“保存”

用于更新现有应用程序网关以支持 Web 应用程序防火墙的设置如下：

- **层** - 所选的层必须是 **WAF** 才能支持 Web 应用程序防火墙
- **SKU 大小** - 此设置是指具有 Web 应用程序防火墙的应用程序网关的大小，可用选项包括**中型**和**大型**。
- **防火墙状态** - 此设置会禁用或启用 Web 应用程序防火墙。
- **防火墙模式** - 此设置是指 Web 应用程序防火墙处理恶意流量的方式。**检测**模式只记录事件，而**阻止**模式则会记录事件并停止恶意流量。

![显示基本设置的边栏选项卡][2]  


>[AZURE.NOTE] 若要查看 Web 应用程序防火墙日志，必须启用诊断功能并选择 ApplicationGatewayFirewallLog。进行测试时，可以选择 1 作为实例计数。必须知道的是，2 以下的实例计数不受 SLA 支持，因此不建议使用。使用 Web 应用程序防火墙时，无法使用小型网关。

## <a name="create-an-application-gateway-with-web-application-firewall"></a> 创建具有 Web 应用程序防火墙的应用程序网关

此方案将：

- 创建包含两个实例的中型应用程序防火墙应用程序网关。
- 创建名为 AdatumAppGatewayVNET 且包含 10.0.0.0/16 保留 CIDR 块的虚拟网络。
- 创建名为 Appgatewaysubnet 且使用 10.0.0.0/28 作为其 CIDR 块的子网。
- 配置进行 SSL 卸载的证书。

### 步骤 1

导航到 Azure 门户预览，单击“新建”>“网络”>“应用程序网关”

![创建应用程序网关][1-1]  


### 步骤 2

下一步，填写有关应用程序网关的基本信息。请务必选择“WAF”作为层。完成后，单击“确定”

基本设置需要的信息如下：

- **Name** - 应用程序网关的名称。
- **层** - 应用程序网关的层，可用选项包括**标准**和 **WAF**。Web 应用程序防火墙仅在 WAF 层中提供。
- **SKU 大小** - 此设置是指应用程序网关的大小，可用选项包括**中型**和**大型**。
- **实例计数** - 实例的数目，此值应该是 **2** 到 **10** 之间的数字。
- **资源组** - 用于保存应用程序网关的资源组，可以是现有资源组，也可以是新的资源组。
- **位置** - 应用程序网关所在的区域，与资源组的位置相同。*位置很重要，因为虚拟网络和公共 IP 必须与网关位于同一位置*。

![显示基本设置的边栏选项卡][2-2]  


>[AZURE.NOTE] 进行测试时，可以选择 1 作为实例计数。必须知道的是，2 以下的实例计数不受 SLA 支持，因此不建议使用。Web 应用程序防火墙方案不支持小型网关。

### 步骤 3

定义基本设置以后，下一步是定义要使用的虚拟网络。虚拟网络托管的应用程序也是通过应用程序网关进行负载均衡的应用程序。

单击“选择虚拟网络”对虚拟网络进行配置。

![显示应用程序网关设置的边栏选项卡][3]

### 步骤 4

在“选择虚拟网络”边栏选项卡中，单击“新建”

虽然此方案未进行说明，但此时可选择现有的虚拟网络。如果使用现有的虚拟网络，请务必了解，需要空的子网或只限应用程序网关资源的子网才能使用该虚拟网络。

![选择虚拟网络边栏选项卡][4]  


### 步骤 5

在“创建虚拟网络”边栏选项卡中填写网络信息，如前面的[方案](#scenario)说明中所述。

![使用输入的信息创建虚拟网络边栏选项卡][5]  


### 步骤 6

创建虚拟网络后，下一步是定义应用程序网关的前端 IP。此时，你可以为前端选择公共 IP 地址或专用 IP 地址。具体选择取决于应用程序是面向 Internet 的还是仅供内部使用的。本方案假定使用的是公共 IP 地址。若要选择专用 IP 地址，可单击“专用”按钮。此时系统会选择自动分配的 IP 地址，你也可以单击“选择特定的专用 IP 地址”复选框手动输入一个。

单击“选择公共 IP 地址”。如果现有的公共 IP 地址可用，则此时可以选择该地址。在本方案中，你需要创建新的公共 IP 地址。单击“新建”

![选择公共 IP 地址边栏选项卡][6]  


### 步骤 7

接下来为公共 IP 地址提供一个友好名称，然后单击“确定”

![创建公共 IP 地址边栏选项卡][7]  


### 步骤 8

接下来，设置侦听器配置。如果使用的是 **http**，不需进行任何配置，单击“确定”即可。若要使用 **https**，需要进一步配置。

若要使用 **https**，需要提供证书。需要提供证书的私钥，这样就需要提供证书的 .pfx 导出结果以及文件的密码。

单击“HTTPS”，然后单击“上载 PFX 证书”文本框旁边的“文件夹”图标。导航到文件系统中的 .pfx 证书文件。选中该文件后，为证书提供一个友好名称，然后键入 .pfx 文件的密码。

完成后，单击“确定”查看应用程序网关的设置。

![“设置”边栏选项卡上的“侦听器配置”部分][8]  


### 步骤 9

配置 **WAF** 特定设置。

- **防火墙状态** - 此设置可打开或关闭 WAF。
- **防火墙模式** - 此设置确定 WAF 对恶意流量采取的操作。如果选择**检测**，只会记录流量。如果选择**阻止**，会记录并停止流量，同时提供“403 未授权”响应。

![Web 应用程序防火墙设置][9]  


### 步骤 10

查看“摘要”页，然后单击“确定”。现在会让应用程序网关排队，然后创建它。

### 步骤 12

创建应用程序网关以后，可在门户中导航到该网关，然后继续进行配置。

![应用程序网关资源视图][10]  


这些步骤会创建基本的应用程序网关，提供侦听器、后端池、后端 http 设置以及规则的默认设置。预配成功后，即可根据部署修改这些设置

## 后续步骤

若要了解如何配置诊断日志记录，以及如何记录通过 Web 应用程序防火墙检测到或阻止的事件，请参阅[应用程序网关诊断](/documentation/articles/application-gateway-diagnostics/)

访问[创建自定义运行状况探测](/documentation/articles/application-gateway-create-probe-portal/)，了解如何创建自定义运行状况探测

访问[配置 SSL 卸载](/documentation/articles/application-gateway-ssl-portal/)，了解如何配置 SSL 卸载并从 Web 服务器中剥离开销较高的 SSL 解密

<!--Image references-->

[1]: ./media/application-gateway-web-application-firewall-portal/figure1.png
[2]: ./media/application-gateway-web-application-firewall-portal/figure2.png
[1-1]: ./media/application-gateway-web-application-firewall-portal/figure1-1.png
[2-2]: ./media/application-gateway-web-application-firewall-portal/figure2-2.png
[3]: ./media/application-gateway-web-application-firewall-portal/figure3.png
[4]: ./media/application-gateway-web-application-firewall-portal/figure4.png
[5]: ./media/application-gateway-web-application-firewall-portal/figure5.png
[6]: ./media/application-gateway-web-application-firewall-portal/figure6.png
[7]: ./media/application-gateway-web-application-firewall-portal/figure7.png
[8]: ./media/application-gateway-web-application-firewall-portal/figure8.png
[9]: ./media/application-gateway-web-application-firewall-portal/figure9.png
[10]: ./media/application-gateway-web-application-firewall-portal/figure10.png
[scenario]: ./media/application-gateway-web-application-firewall-portal/scenario.png

<!---HONumber=Mooncake_1024_2016-->