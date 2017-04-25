<properties
    pageTitle="使用门户创建应用程序网关 | Azure"
    description="了解如何使用门户创建应用程序网关"
    services="application-gateway"
    documentationcenter="na"
    author="georgewallace"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />  

<tags
    ms.assetid="54dffe95-d802-4f86-9e2e-293f49bd1e06"
    ms.service="application-gateway"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="12/12/2016"
    wacn.date="02/21/2017"
    ms.author="gwallace" />

# 使用门户创建应用程序网关
> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/application-gateway-create-gateway-portal/)
- [Azure Resource Manager PowerShell](/documentation/articles/application-gateway-create-gateway-arm/)
- [Azure 经典 PowerShell](/documentation/articles/application-gateway-create-gateway/)
- [Azure Resource Manager 模板](/documentation/articles/application-gateway-create-gateway-arm-template/)
- [Azure CLI](/documentation/articles/application-gateway-create-gateway-cli/)

Azure 应用程序网关是第 7 层负载均衡器。它在不同服务器之间提供故障转移和性能路由 HTTP 请求，而不管它们是在云中还是本地。应用程序网关提供许多应用程序传送控制器 (ADC) 功能，包括 HTTP 负载均衡、基于 cookie 的会话相关性、安全套接字层 (SSL) 卸载、自定义运行状况探测、多站点支持，以及许多其他功能。若要查找支持的功能的完整列表，请参阅[应用程序网关概述](/documentation/articles/application-gateway-introduction/)

## <a name="scenario"></a>方案

此方案介绍如何使用 Azure 门户预览创建应用程序网关。

此方案将：

* 创建包含两个实例的中型应用程序网关。
* 创建名为 AdatumAppGatewayVNET 且包含 10.0.0.0/16 保留 CIDR 块的虚拟网络。
* 创建名为 Appgatewaysubnet 且使用 10.0.0.0/28 作为其 CIDR 块的子网。
* 配置进行 SSL 卸载的证书。

![方案示例][scenario]  


> [AZURE.IMPORTANT]
针对应用程序网关进行的其他配置（包括自定义运行状况探测、后端池地址以及其他规则）是在对应用程序网关配置以后配置的，不是在初始部署期间配置的。
> 
> 

## 开始之前

Azure 应用程序网关需要自己的子网。在创建虚拟网络时，请确保保留足够的地址空间，以便设置多个子网。将应用程序网关部署到子网后，只能向该子网添加其他应用程序网关。

## 创建应用程序网关

### 步骤 1

导航到 Azure 门户预览，单击“新建”>“网络”>“应用程序网关”

![创建应用程序网关][1]  


### 步骤 2

下一步，填写有关应用程序网关的基本信息。完成后，单击“确定”

基本设置需要的信息如下：

* **Name** - 应用程序网关的名称。
* **层** - 此设置是应用程序网关的层。可以使用两个层，即 **WAF** 层和**标准**层。WAF 启用 Web 应用程序防火墙功能。
* **SKU 大小** - 此设置是指应用程序网关的大小，可用选项包括**小型**、**中型**和**大型**。选择 WAF 层时，小型不可用。
* **实例计数** - 实例的数目。此值应该是 2 到 10 之间的数字。
* **资源组** - 用于保存应用程序网关的资源组，可以是现有资源组，也可以是新的资源组。
* **位置** - 应用程序网关所在的区域，与资源组的位置相同。位置很重要，因为虚拟网络和公共 IP 必须与网关位于同一位置。

![显示基本设置的边栏选项卡][2]  


> [AZURE.NOTE]
进行测试时，可以选择 1 作为实例计数。必须知道的是，2 以下的实例计数不受 SLA 支持，因此不建议使用。小型网关用于开发/测试，不用于生产。
> 
> 

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

### 步骤 7

单击“选择公共 IP 地址”。如果现有的公共 IP 地址可用，则此时可以选择该地址。在本方案中，你需要创建新的公共 IP 地址。单击“新建”

![选择公共 IP 地址边栏选项卡][6]

### 步骤 8

接下来为公共 IP 地址提供一个友好名称，然后单击“确定”

![创建公共 IP 地址边栏选项卡][7]  


### 步骤 9

创建应用程序网关时，需要配置的最后一个设置是侦听器配置。如果使用的是 **http**，不需进行任何配置，单击“确定”即可。若要使用 **https**，需要进一步配置。

若要使用 **https**，需要提供证书。由于需要证书的私钥，请提供证书的 .pfx 导出结果以及密码。

### 步骤 10

单击“HTTPS”，然后单击“上载 PFX 证书”文本框旁边的“文件夹”图标。导航到文件系统中的 .pfx 证书文件。选中该文件后，为证书提供一个友好名称，然后键入 .pfx 文件的密码。

完成后，单击“确定”查看应用程序网关的设置。

![“设置”边栏选项卡上的“侦听器配置”部分][9]

### 步骤 11

查看“摘要”页，然后单击“确定”。现在会让应用程序网关排队，然后创建它。

### 步骤 12

创建应用程序网关以后，可在门户中导航到该网关，然后继续进行配置。

![应用程序网关资源视图][10]  


这些步骤会创建基本的应用程序网关，提供侦听器、后端池、后端 http 设置以及规则的默认设置。预配成功后，即可根据部署修改这些设置。

## 将服务器添加到后端池

创建应用程序网关后，仍需将系统（托管着将进行负载均衡的应用程序）添加到应用程序网关。这些服务器的 IP 地址 或 FQDN 值已添加到后端地址池。

### 步骤 1

单击创建的应用程序网关，单击“后端池”，然后选择当前后端池。

![应用程序网关后端池][11]  


### 步骤 2

在文本框中添加 IP 地址或 FQDN 值，然后单击“保存”

![向应用程序网关后端池添加值][12]  


此操作会将值保存到后端池。更新应用程序网关后，进入应用程序网关的流量将路由到在此步骤中添加的后端地址。

## 后续步骤

此方案创建默认应用程序网关。后续步骤是通过修改设置以及调整网关中的规则，配置应用程序网关。通过访问以下文章，可找到这些步骤：

访问[创建自定义运行状况探测](/documentation/articles/application-gateway-create-probe-portal/)，了解如何创建自定义运行状况探测

访问[配置 SSL 卸载](/documentation/articles/application-gateway-ssl-portal/)，了解如何配置 SSL 卸载并从 Web 服务器中剥离开销较高的 SSL 解密

了解如何使用应用程序网关的 [Web 应用程序防火墙](/documentation/articles/application-gateway-web-application-firewall-overview/)功能保护应用程序。

<!--Image references-->

[1]: ./media/application-gateway-create-gateway-portal/figure1.png
[2]: ./media/application-gateway-create-gateway-portal/figure2.png
[3]: ./media/application-gateway-create-gateway-portal/figure3.png
[4]: ./media/application-gateway-create-gateway-portal/figure4.png
[5]: ./media/application-gateway-create-gateway-portal/figure5.png
[6]: ./media/application-gateway-create-gateway-portal/figure6.png
[7]: ./media/application-gateway-create-gateway-portal/figure7.png
[8]: ./media/application-gateway-create-gateway-portal/figure8.png
[9]: ./media/application-gateway-create-gateway-portal/figure9.png
[10]: ./media/application-gateway-create-gateway-portal/figure10.png
[11]: ./media/application-gateway-create-gateway-portal/figure11.png
[12]: ./media/application-gateway-create-gateway-portal/figure12.png
[scenario]: ./media/application-gateway-create-gateway-portal/scenario.png

<!---HONumber=Mooncake_1226_2016-->