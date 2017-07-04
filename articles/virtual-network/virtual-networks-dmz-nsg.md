<properties
    pageTitle="Azure 外围网络示例 - 使用 NSG 构建简单的外围网络 | Azure"
    description="使用网络安全组 (NSG) 构建外围网络"
    services="virtual-network"
    documentationcenter="na"
    author="tracsman"
    manager="rossort"
    editor="" />
<tags
    ms.assetid=""
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="01/03/2017"
    wacn.date="02/10/2017"
    ms.author="jonor" />  


# 示例 1 - 将 NSG 与 Azure Resource Manager 模板配合使用，构建简单的外围网络
[返回安全边界最佳实践页面][HOME]
> [AZURE.SELECTOR]
- [Resource Manager 模板](/documentation/articles/virtual-networks-dmz-nsg/)
- [经典 - PowerShell](/documentation/articles/virtual-networks-dmz-nsg-asm/)

本示例将创建一个原始的外围网络，其中包含四个 Windows Server 和网络安全组。本示例介绍了每个相关的模板部分，帮助用户更好地理解每一步。另外还提供了“流量方案”部分，帮助用户逐步深入了解流量如何流经外围网络的各个防御层。最后的“参考”部分提供了完整的模板代码，并说明如何构建此环境来测试和试验各种方案。

[AZURE.INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)]

![使用 NSG 的入站外围网络][1]  


## 环境描述
本示例中的一个订阅包含以下资源：

* 单个资源组
* 一个虚拟网络，其中包含两个子网：“FrontEnd”和“BackEnd”
* 应用到这两个子网的网络安全组
* 一个代表应用程序 Web 服务器的 Windows Server（“IIS01”）
* 两个代表应用程序后端服务器的 Windows Server（“AppVM01”、“AppVM02”）
* 一个代表 DNS 服务器的 Windows Server（“DNS01”）
* 与应用程序 Web 服务器关联的公共 IP 地址

在参考部分有一个 Azure Resource Manager 模板的链接，该模板可构建本示例中描述的环境。尽管该示例模板还完成了 VM 和虚拟网络的构建，但本文档未对其进行详细描述。

**构建此环境**（详细说明见本文档的参考部分）；

1. 部署 [Azure 快速启动模板][Template]中的 Azure Resource Manager 模板
2. 安装[示例应用程序脚本][SampleApp]中的示例应用程序

>[AZURE.NOTE]
若要通过 RDP 连接到本实例中的任何后端服务器，可使用 IIS 服务器作为“跳转盒”。 首先通过 RDP 连接到 IIS 服务器，然后再从 IIS 服务器通过 RDP 连接到后端服务器。也可将一个公共 IP 与每个服务器 NIC 关联，方便 RDP 的执行。
> 
>

以下部分通过演示 Azure Resource Manager 模板的关键代码行，详细说明了本示例的网络安全组及其运行方式。

## 网络安全组 (NSG)
此示例将构建一个 NSG 组，然后为其加载六个规则。

>[AZURE.TIP]
一般而言，应该先创建特定的“允许”规则，然后创建一般的“拒绝”规则。分配的优先级确定先评估哪些规则。发现要向流量应用的特定规则后，不再需要评估后续规则。可以朝入站或出站方向（从子网的角度看）应用 NSG 规则。
>
>

以声明性的方式为入站流量构建以下规则：

1. 允许内部 DNS 流量（端口 53）
2. 允许从 Internet 到任何 VM 的 RDP 流量（端口 3389）
3. 允许从 Internet 到 Web 服务器 (IIS01) 的 HTTP 流量（端口 80）
4. 允许从 IIS01 到 AppVM1 的任何流量（所有端口）
5. 拒绝从 Internet 到整个 VNet（两个子网）的任何流量（所有端口）
6. 拒绝从前端子网到后端子网的任何流量（所有端口）

将这些规则绑定到每个子网后，如果有从 Internet 到 Web 服务器的入站 HTTP 请求，那么规则 3（允许）和规则 5（拒绝）均适用，但由于规则 3 具有较高的优先级，因此只应用规则 3，忽略规则 5。这样就会允许 HTTP 请求传往 Web 服务器。如果相同的流量尝试传往 DNS01 服务器，则会先应用规则 5（拒绝），因此不允许该流量传递到服务器。规则 6（拒绝）阻止前端子网与后端子网对话（规则 1 和 4 允许的流量除外），此规则集可在攻击者入侵前端 Web 应用程序时保护后端网络，攻击者只能对后端“受保护”的网络进行有限制的访问（只能访问 AppVM01 服务器上公开的资源）。

有一个默认出站规则可允许流量外流到 Internet。在此示例中，我们允许出站流量，且未修改任何出站规则。若要对两个方向的流量应用安全策略，需使用“用户定义的路由”，详见[安全边界最佳实践页面][HOME]上的“示例 3”。

下面对每个规则进行了更详细的讨论：

1. 网络安全组资源必须实例化才能在其中保留以下规则：

        "resources": [
          {
            "apiVersion": "2015-05-01-preview",
            "type": "Microsoft.Network/networkSecurityGroups",
            "name": "[variables('NSGName')]",
            "location": "[resourceGroup().location]",
            "properties": { }
          }
        ]

2. 本示例中的第一个规则允许所有内部网络之间的 DNS 流量发往后端子网上的 DNS 服务器。该规则有一些重要参数：
    * "destinationAddressPrefix" - 规则可以使用名为“默认标记”的特殊类型的地址前缀，这些标记是系统提供的标识符，方便更大类别的地址前缀的寻址。此规则使用默认标记 "Internet" 表示 VNet 外部的任何地址。其他前缀标签还有 VirtualNetwork 和 AzureLoadBalancer。
    * "Direction" 表示此规则对哪个方向的流量生效。方向从子网或虚拟机的角度来定（具体取决于此 NSG 绑定到哪里）。因此，如果 Direction 是 "Inbound" 并且流量进入子网，则适用此规则，而离开子网的流量则不受此规则影响。
    * "Priority" 设置流量的评估顺序。编号越低，优先级就越高。将某个规则应用于特定的流量后，就不再处理其他规则。因此，如果优先级为 1 的规则允许流量，优先级为 2 的规则拒绝流量，并将这两个规则同时应用于流量，则允许流量流动（规则 1 的优先级更高，因此将发生作用，并且不再应用其他规则）。
    * "Access" 表示是阻止 ("Deny") 还是允许 ("Allow") 受此规则影响的流量。

            "properties": {
            "securityRules": [
              {
                "name": "enable_dns_rule",
                "properties": {
                  "description": "Enable Internal DNS",
                  "protocol": "*",
                  "sourcePortRange": "*",
                  "destinationPortRange": "53",
                  "sourceAddressPrefix": "VirtualNetwork",
                  "destinationAddressPrefix": "10.0.2.4",
                  "access": "Allow",
                  "priority": 100,
                  "direction": "Inbound"
                }
              },

3. 此规则允许 RDP 流量从 Internet 发往绑定子网上任何服务器的 RDP 端口。

        {
          "name": "enable_rdp_rule",
          "properties": {
            "description": "Allow RDP",
            "protocol": "Tcp",
            "sourcePortRange": "*",
            "destinationPortRange": "3389",
            "sourceAddressPrefix": "*",
            "destinationAddressPrefix": "*",
            "access": "Allow",
            "priority": 110,
            "direction": "Inbound"
          }
        },

4. 此规则允许入站 Internet 流量抵达 Web 服务器。此规则不会更改路由行为。该规则仅允许发往 IIS01 的流量通过。因此，如果来自 Internet 的流量将 Web 服务器作为其目标，此规则将允许流量，并停止处理其他规则。（在优先级为 140 的规则中，其他所有入站 Internet 流量均被阻止）。如果你只要处理 HTTP 流量，可将此规则进一步限制为只允许目标端口 80。

        {
          "name": "enable_web_rule",
          "properties": {
            "description": "Enable Internet to [variables('VM01Name')]",
            "protocol": "Tcp",
            "sourcePortRange": "*",
            "destinationPortRange": "80",
            "sourceAddressPrefix": "Internet",
            "destinationAddressPrefix": "10.0.1.5",
            "access": "Allow",
            "priority": 120,
            "direction": "Inbound"
            }
          },

5. 此规则允许流量从 IIS01 服务器传递到 AppVM01 服务器，后面的规则将阻止其他所有从前端到后端的流量。如果要添加的端口是已知的，则可以改善此规则。例如，如果 IIS 服务器的流量只抵达 AppVM01 上的 SQL Server，并且 Web 应用程序曾遭到入侵，则目标端口范围应该从 "*"（任意端口）更改为 "1433"（SQL 端口），以缩小 AppVM01 上的入站攻击面。

        {
          "name": "enable_app_rule",
          "properties": {
            "description": "Enable [variables('VM01Name')] to [variables('VM02Name')]",
            "protocol": "*",
            "sourcePortRange": "*",
            "destinationPortRange": "*",
            "sourceAddressPrefix": "10.0.1.5",
            "destinationAddressPrefix": "10.0.2.5",
            "access": "Allow",
            "priority": 130,
            "direction": "Inbound"
          }
        },

6. 此规则将拒绝从 Internet 到网络上任何服务器的流量。规则的优先级为 110 和 120 时，只有入站 Internet 流量能够发往防火墙和服务器上的 RDP 端口，其他流量将被阻止。此规则属于“防故障”规则，可以阻止所有意外的流量。

        {
          "name": "deny_internet_rule",
          "properties": {
            "description": "Isolate the [variables('VNetName')] VNet from the Internet",
            "protocol": "*",
            "sourcePortRange": "*",
            "destinationPortRange": "*",
            "sourceAddressPrefix": "Internet",
            "destinationAddressPrefix": "VirtualNetwork",
            "access": "Deny",
            "priority": 140,
            "direction": "Inbound"
          }
        },

7. 最后一个规则拒绝从前端子网到后端子网的流量。此规则为仅针对入站的规则，因此允许反向流量（从后端到前端）。

        {
          "name": "deny_frontend_rule",
          "properties": {
            "description": "Isolate the [variables('Subnet1Name')] subnet from the [variables('Subnet2Name')] subnet",
            "protocol": "*",
            "sourcePortRange": "*",
            "destinationPortRange": "*",
            "sourceAddressPrefix": "[variables('Subnet1Prefix')]",
            "destinationAddressPrefix": "[variables('Subnet2Prefix')]",
            "access": "Deny",
            "priority": 150,
            "direction": "Inbound"
          }
        }

## 流量方案
#### （*允许*）从 Internet 访问 Web 服务器
1. Internet 用户从与 IIS01 NIC 关联的 NIC 的公共 IP 地址请求 HTTP 页
2. 该公共 IP 地址将流量传递到 VNet，再到 IIS01（Web 服务器）
3. 前端子网开始处理入站规则：
    1. NSG 规则 1 (DNS) 不适用，将转到下一条规则
    2. NSG 规则 2 (RDP) 不适用，将转到下一条规则
    3. NSG 规则 3（Internet 到 IIS01）适用，允许流量，停止处理规则
4. 流量抵达 Web 服务器 IIS01 的内部 IP 地址 (10.0.1.5)
5. IIS01 正在侦听 Web 流量，将接收此请求并开始处理请求
6. IIS01 请求 AppVM01 上的 SQL Server 提供信息
7. 前端子网上没有出站规则，允许流量
8. 后端子网开始处理入站规则：
    1. NSG 规则 1 (DNS) 不适用，将转到下一条规则
    2. NSG 规则 2 (RDP) 不适用，将转到下一条规则
    3. NSG 规则 3（Internet 到防火墙）不适用，将转到下一条规则
    4. NSG 规则 4（IIS01 到 AppVM01）适用，允许流量，停止规则处理
9. AppVM01 接收 SQL 查询并做出响应
10. 后端子网上没有出站规则，因此允许响应
11. 前端子网开始处理入站规则：
    1. 后端子网到前端子网的入站流量没有适用的 NSG 规则，因此不会应用任何 NSG 规则
    2. 允许子网间流量的默认系统规则允许此流量，因此允许流量。
12. IIS 服务器接收 SQL 响应，完成 HTTP 响应并将其发送给请求方
13. 前端子网上没有出站规则，因此允许响应，Internet 用户将收到请求的网页。

#### （*允许*）通过 RDP 访问 IIS 服务器
1. Internet 上的服务器管理员请求通过 RDP 会话访问 IIS01，后者位于与 IIS01 NIC 关联的 NIC 的公共 IP 地址上（此公共 IP 地址可以通过门户或 PowerShell 找到）
2. 该公共 IP 地址将流量传递到 VNet，再到 IIS01（Web 服务器）
3. 前端子网开始处理入站规则：
    1. NSG 规则 1 (DNS) 不适用，将转到下一条规则
    2. NSG 规则 2 (RDP) 适用，允许流量，停止规则处理
4. 由于没有出站规则，将应用默认规则并允许返回流量
5. 已启用 RDP 会话
6. IIS01 会提示用户提供用户名和密码

>[AZURE.NOTE]
若要通过 RDP 连接到本实例中的任何后端服务器，可使用 IIS 服务器作为“跳转盒”。 首先通过 RDP 连接到 IIS 服务器，然后再从 IIS 服务器通过 RDP 连接到后端服务器。
>
>

#### （*允许*）在 DNS 服务器上执行 Web 服务器 DNS 查找
1. Web 服务器 IIS01 需要 www.data.gov 上的数据源，但需要解析地址。
2. VNet 的网络配置将 DNS01（后端子网上的 10.0.2.4）列为主 DNS 服务器，IIS01 将 DNS 请求发送到 DNS01
3. 前端子网上没有出站规则，允许流量
4. 后端子网开始处理入站规则：
    * NSG 规则 1 (DNS) 适用，允许流量，停止规则处理
5. DNS 服务器接收请求
6. DNS 服务器没有缓存的地址，请求 Internet 上的根 DNS 服务器
7. 后端子网上没有出站规则，允许流量
8. Internet DNS 服务器做出响应，由于此会话是从内部发起的，因此允许响应
9. DNS 服务器缓存响应，然后将初始请求响应发送给 IIS01
10. 后端子网上没有出站规则，允许流量
11. 前端子网开始处理入站规则：
    1. 后端子网到前端子网的入站流量没有适用的 NSG 规则，因此不会应用任何 NSG 规则
    2. 允许子网间流量的默认系统规则允许此流量，因此允许流量
12. IIS01 从 DNS01 接收响应

#### （*允许*）Web 服务器访问 AppVM01 上的文件
1. IIS01 请求 AppVM01 上的文件
2. 前端子网上没有出站规则，允许流量
3. 后端子网开始处理入站规则：
    1. NSG 规则 1 (DNS) 不适用，将转到下一条规则
    2. NSG 规则 2 (RDP) 不适用，将转到下一条规则
    3. NSG 规则 3（Internet 到 IIS01）不适用，将转到下一条规则
    4. NSG 规则 4（IIS01 到 AppVM01）适用，允许流量，停止规则处理
4. AppVM01 接收请求并以文件做出响应（假设已获得访问授权）
5. 后端子网上没有出站规则，因此允许响应
6. 前端子网开始处理入站规则：
    1. 后端子网到前端子网的入站流量没有适用的 NSG 规则，因此不会应用任何 NSG 规则
    2. 允许子网间流量的默认系统规则允许此流量，因此允许流量。
7. IIS 服务器接收文件

#### （*拒绝*）通过 RDP 访问后端
1. Internet 用户尝试通过 RDP 访问服务器 AppVM01
2. 由于没有公共 IP 地址与此服务器 NIC 关联，因此该流量不会进入 VNet，不会抵达服务器
3. 但是，如果某个公共 IP 地址因某种原因而启用，则 NSG 规则 2 (RDP) 会允许此流量

>[AZURE.NOTE]
若要通过 RDP 连接到本实例中的任何后端服务器，可使用 IIS 服务器作为“跳转盒”。 首先通过 RDP 连接到 IIS 服务器，然后再从 IIS 服务器通过 RDP 连接到后端服务器。
>
>

#### （*拒绝*）通过 Web 访问后端服务器
1. Internet 用户尝试访问 AppVM01 上的文件
2. 由于没有公共 IP 地址与此服务器 NIC 关联，因此该流量不会进入 VNet，不会抵达服务器
3. 如果某个公共 IP 地址因某种原因而启用，则 NSG 规则 5（Internet 到 VNet）会阻止此流量

#### （*拒绝*）在 DNS 服务器上执行 Web DNS 查找
1. Internet 用户尝试查找 DNS01 上的内部 DNS 记录
2. 由于没有公共 IP 地址与此服务器 NIC 关联，因此该流量不会进入 VNet，不会抵达服务器
3. 如果某个公共 IP 地址因某种原因而启用，则 NSG 规则 5（Internet 到 VNet）会阻止此流量（注意：由于请求源地址为 Internet 地址，而规则 1 (DNS) 仅适用于作为源的本地 VNet，因此规则 1 不适用）

#### （*拒绝*）在 Web 服务器上进行 SQL 访问
1. Internet 用户从 IIS01 请求 SQL 数据
2. 由于没有公共 IP 地址与此服务器 NIC 关联，因此该流量不会进入 VNet，不会抵达服务器
3. 如果某个公共 IP 地址因某种原因而启用，则前端子网将开始处理入站规则：
    1. NSG 规则 1 (DNS) 不适用，将转到下一条规则
    2. NSG 规则 2 (RDP) 不适用，将转到下一条规则
    3. NSG 规则 3（Internet 到 IIS01）适用，允许流量，停止处理规则
4. 流量抵达 IIS01 的内部 IP 地址 (10.0.1.5)
5. IIS01 未侦听端口 1433，因此不会对请求做出响应

## 结束语
这是一个相对简单的示例，直观演示了如何将后端子网与入站流量隔离。

可以在[此处][HOME]找到更多示例和网络安全边界的概述。

## 参考
### Azure Resource Manager 模板
本示例使用由 Microsoft 维护并对社区开放的 GitHub 存储库的预定义 Azure Resource Manager 模板。该模板可直接从 GitHub 部署，也可在下载后根据需要进行修改。

主模板位于名为“azuredeploy.json”的文件中。 该模板可以通过 PowerShell 或 CLI 提交（与关联的“azuredeploy.parameters.json”文件一起），以便进行部署。我发现，最简单的方式是使用 GitHub 的 README.md 页上的“部署到 Azure”按钮。

若要部署从 GitHub 和 Azure 门户生成本示例的模板，请执行以下步骤：

1. 在浏览器中导航到[模板][Template]
2. 单击“部署到 Azure”按钮（或者可查看该模板图形表示形式的“可视化”按钮）
3. 在“参数”边栏选项卡中输入存储帐户、用户名和密码，然后单击“确定”
5. 为此部署创建资源组（可以使用现有资源组，但建议创建一个新的，以便获得最佳结果）
6. 如有必要，请更改 VNet 的“订阅”和“位置”设置。
7. 单击“查看法律条款”，阅读这些条款，然后单击“购买”表示同意。
8. 单击“创建”开始部署此模板。
9. 部署成功完成以后，导航到为此部署创建的“资源组”，查看其中配置的资源。

>[AZURE.NOTE]
此模板仅允许通过 RDP 访问 IIS01 服务器（在门户中查找 IIS01 的公共 IP）。若要通过 RDP 连接到本实例中的任何后端服务器，可使用 IIS 服务器作为“跳转盒”。 首先通过 RDP 连接到 IIS 服务器，然后再从 IIS 服务器通过 RDP 连接到后端服务器。
>
>

若要删除此部署，请删除该资源组，然后所有子资源也会被删除。

#### 示例应用程序脚本
模板成功运行以后，即可设置 Web 服务器、应用服务器和简单的 Web 应用程序，以便测试此外围网络配置。若要为其安装示例应用程序和其他外围网络示例，可使用以下链接提供的：[示例应用程序脚本][SampleApp]

## 后续步骤

* 部署本示例
* 生成示例应用程序
* 通过此外围网络测试不同的流量

<!--Image References-->

[1]: ./media/virtual-networks-dmz-nsg-arm/example1design.png "使用 NSG 的入站外围网络"

<!--Link References-->

[HOME]: /documentation/articles/best-practices-network-security/
[Template]: https://github.com/Azure/azure-quickstart-templates/tree/master/301-dmz-nsg
[SampleApp]: /documentation/articles/virtual-networks-sample-app/

<!---HONumber=Mooncake_0206_2017-->