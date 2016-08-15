<properties 
   pageTitle="如何使用 PowerShell 在经典模式下创建 NSG | Azure"
   description="了解如何使用 PowerShell 在经典模式下创建和部署 NSG"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor="tysonn"
   tags="azure-service-management"
/>
<tags
	ms.service="virtual-network"
	ms.date="02/02/2016"
	wacn.date="03/28/2016"/>

# 如何在 PowerShell 中创建 NSG（经典）

[AZURE.INCLUDE [virtual-networks-create-nsg-selectors-classic-include](../includes/virtual-networks-create-nsg-selectors-classic-include.md)]

[AZURE.INCLUDE [virtual-networks-create-nsg-intro-include](../includes/virtual-networks-create-nsg-intro-include.md)]

>[AZURE.IMPORTANT]在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：资源管理器部署模型和经典部署模型。在使用任何 Azure 资源之前，请确保你了解[部署模型和工具](/documentation/articles/azure-classic-rm/)。可以通过单击本文顶部的选项卡来查看不同工具的文档。本文介绍经典部署模型。你还可以[在资源管理器部署模型中创建 NSG](/documentation/articles/virtual-networks-create-nsg-arm-ps/)。

[AZURE.INCLUDE [virtual-networks-create-nsg-scenario-include](../includes/virtual-networks-create-nsg-scenario-include.md)]

下面的示例 PowerShell 命令需要一个已经基于上述方案创建的简单环境。如果你想要运行本文档中所显示的命令，首先通过[创建 VNet](/documentation/articles/virtual-networks-create-vnet-classic-netcfg-ps/) 构建测试环境。

## 如何为前端子网创建 NSG
若要基于上述方案创建名为 **NSG-FrontEnd** 的 NSG，请执行下面的步骤：

1. 如果你从未使用过 Azure PowerShell，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)，并始终按照说明进行操作，以登录到 Azure 并选择你的订阅。

3. 创建名为 **NSG-FrontEnd** 的网络安全组。

		New-AzureNetworkSecurityGroup -Name "NSG-FrontEnd" -Location uswest `
		    -Label "Front end subnet NSG"

	预期输出：

		Name         Location   Label               
		----         --------   -----               
		NSG-FrontEnd China North 	Front end subnet NSG


4. 创建允许从 Internet 访问端口 3389 的安全规则。

		Get-AzureNetworkSecurityGroup -Name "NSG-FrontEnd" `
		| Set-AzureNetworkSecurityRule -Name rdp-rule `
		    -Action Allow -Protocol TCP -Type Inbound -Priority 100 `
		    -SourceAddressPrefix Internet  -SourcePortRange '*' `
		    -DestinationAddressPrefix '*' -DestinationPortRange '3389' 

	预期输出：

		Name     : NSG-FrontEnd
		Location : China North
		Label    : Front end subnet NSG
		Rules    : 
		           
		              Type: Inbound
		           
		           Name                 Priority  Action   Source Address  Source Port   Destination      Destination    Protocol
		                                                   Prefix          Range         Address Prefix   Port Range             
		           ----                 --------  ------   --------------- ------------- ---------------- -------------- --------
		           rdp-rule             100       Allow    INTERNET        *             *                3389           TCP     
		           ALLOW VNET INBOUND   65000     Allow    VIRTUAL_NETWORK *             VIRTUAL_NETWORK  *              *       
		           ALLOW AZURE LOAD     65001     Allow    AZURE_LOADBALAN *             *                *              *       
		           BALANCER INBOUND                        CER                                                                   
		           DENY ALL INBOUND     65500     Deny     *               *             *                *              *       
		           
		           
		              Type: Outbound
		           
		           Name                 Priority  Action   Source Address  Source Port   Destination      Destination    Protocol
		                                                   Prefix          Range         Address Prefix   Port Range             
		           ----                 --------  ------   --------------- ------------- ---------------- -------------- --------
		           ALLOW VNET OUTBOUND  65000     Allow    VIRTUAL_NETWORK *             VIRTUAL_NETWORK  *              *       
		           ALLOW INTERNET       65001     Allow    *               *             INTERNET         *              *       
		           OUTBOUND                                                                                                      
		           DENY ALL OUTBOUND    65500     Deny     *               *             *                *              *

4. 创建允许从 Internet 访问端口 80 的安全规则。

		Get-AzureNetworkSecurityGroup -Name "NSG-FrontEnd" `
		| Set-AzureNetworkSecurityRule -Name web-rule `
		    -Action Allow -Protocol TCP -Type Inbound -Priority 200 `
		    -SourceAddressPrefix Internet  -SourcePortRange '*' `
		    -DestinationAddressPrefix '*' -DestinationPortRange '80' 

	预期输出：
		

		Name     : NSG-FrontEnd
		Location : China North
		Label    : Front end subnet NSG
		Rules    : 
		           
		              Type: Inbound
		           
		           Name                 Priority  Action   Source Address  Source Port   Destination      Destination    Protocol
		                                                   Prefix          Range         Address Prefix   Port Range             
		           ----                 --------  ------   --------------- ------------- ---------------- -------------- --------
		           rdp-rule             100       Allow    INTERNET        *             *                3389           TCP     
		           web-rule             200       Allow    INTERNET        *             *                80             TCP     
		           ALLOW VNET INBOUND   65000     Allow    VIRTUAL_NETWORK *             VIRTUAL_NETWORK  *              *       
		           ALLOW AZURE LOAD     65001     Allow    AZURE_LOADBALAN *             *                *              *       
		           BALANCER INBOUND                        CER                                                                   
		           DENY ALL INBOUND     65500     Deny     *               *             *                *              *       
		           
		           
		              Type: Outbound
		           
		           Name                 Priority  Action   Source Address  Source Port   Destination      Destination    Protocol
		                                                   Prefix          Range         Address Prefix   Port Range             
		           ----                 --------  ------   --------------- ------------- ---------------- -------------- --------
		           ALLOW VNET OUTBOUND  65000     Allow    VIRTUAL_NETWORK *             VIRTUAL_NETWORK  *              *       
		           ALLOW INTERNET       65001     Allow    *               *             INTERNET         *              *       
		           OUTBOUND                                                                                                      
		           DENY ALL OUTBOUND    65500     Deny     *               *             *                *              *   

## 如何为后端子网创建 NSG
3. 创建名为 **NSG-BackEnd** 的网络安全组。

		New-AzureNetworkSecurityGroup -Name "NSG-BackEnd" -Location uswest `
		    -Label "Back end subnet NSG"

	预期输出：

		Name        Location   Label              
		----        --------   -----              
		NSG-BackEnd China North    Back end subnet NSG


4. 创建允许从前端子网访问端口 1433（SQL Server 使用的默认端口）的安全规则。

		Get-AzureNetworkSecurityGroup -Name "NSG-FrontEnd" `
		| Set-AzureNetworkSecurityRule -Name rdp-rule `
		    -Action Allow -Protocol TCP -Type Inbound -Priority 100 `
		    -SourceAddressPrefix 192.168.1.0/24  -SourcePortRange '*' `
		    -DestinationAddressPrefix '*' -DestinationPortRange '1433' 

	预期输出：

		Name     : NSG-BackEnd
		Location : China North
		Label    : Back end subnet NSG
		Rules    : 
		           
		              Type: Inbound
		           
		           Name                 Priority  Action   Source Address  Source Port   Destination      Destination    Protocol
		                                                   Prefix          Range         Address Prefix   Port Range             
		           ----                 --------  ------   --------------- ------------- ---------------- -------------- --------
		           fe-rule              100       Allow    192.168.1.0/24  *             *                1433           TCP     
		           ALLOW VNET INBOUND   65000     Allow    VIRTUAL_NETWORK *             VIRTUAL_NETWORK  *              *       
		           ALLOW AZURE LOAD     65001     Allow    AZURE_LOADBALAN *             *                *              *       
		           BALANCER INBOUND                        CER                                                                   
		           DENY ALL INBOUND     65500     Deny     *               *             *                *              *       
		           
		           
		              Type: Outbound
		           
		           Name                 Priority  Action   Source Address  Source Port   Destination      Destination    Protocol
		                                                   Prefix          Range         Address Prefix   Port Range             
		           ----                 --------  ------   --------------- ------------- ---------------- -------------- --------
		           ALLOW VNET OUTBOUND  65000     Allow    VIRTUAL_NETWORK *             VIRTUAL_NETWORK  *              *       
		           ALLOW INTERNET       65001     Allow    *               *             INTERNET         *              *       
		           OUTBOUND                                                                                                      
		           DENY ALL OUTBOUND    65500     Deny     *               *             *                *              *      

4. 创建阻止从子网访问 Internet 的安全规则。

		Get-AzureNetworkSecurityGroup -Name "NSG-BackEnd" `
		| Set-AzureNetworkSecurityRule -Name block-internet `
		    -Action Deny -Protocol '*' -Type Outbound -Priority 200 `
		    -SourceAddressPrefix '*'  -SourcePortRange '*' `
		    -DestinationAddressPrefix Internet -DestinationPortRange '*' 

	预期输出：

		Name     : NSG-BackEnd
		Location : China North
		Label    : Back end subnet NSG
		Rules    : 
		           
		              Type: Inbound
		           
		           Name                 Priority  Action   Source Address  Source Port   Destination      Destination    Protocol
		                                                   Prefix          Range         Address Prefix   Port Range             
		           ----                 --------  ------   --------------- ------------- ---------------- -------------- --------
		           fe-rule              100       Allow    192.168.1.0/24  *             *                1433           TCP     
		           ALLOW VNET INBOUND   65000     Allow    VIRTUAL_NETWORK *             VIRTUAL_NETWORK  *              *       
		           ALLOW AZURE LOAD     65001     Allow    AZURE_LOADBALAN *             *                *              *       
		           BALANCER INBOUND                        CER                                                                   
		           DENY ALL INBOUND     65500     Deny     *               *             *                *              *       
		           
		           
		              Type: Outbound
		           
		           Name                 Priority  Action   Source Address  Source Port   Destination      Destination    Protocol
		                                                   Prefix          Range         Address Prefix   Port Range             
		           ----                 --------  ------   --------------- ------------- ---------------- -------------- --------
		           block-internet       200       Deny     *               *             INTERNET         *              *       
		           ALLOW VNET OUTBOUND  65000     Allow    VIRTUAL_NETWORK *             VIRTUAL_NETWORK  *              *       
		           ALLOW INTERNET       65001     Allow    *               *             INTERNET         *              *       
		           OUTBOUND                                                                                                      
		           DENY ALL OUTBOUND    65500     Deny     *               *             *                *              *   

<!---HONumber=Mooncake_1221_2015-->