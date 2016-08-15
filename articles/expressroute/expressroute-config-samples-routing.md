<properties
   pageTitle="ExpressRoute 客户路由器配置示例 | Azure"
   description="本页提供 Cisco 和 Juniper 路由器的路由器配置示例。"
   documentationCenter="na"
   services="expressroute"
   authors="cherylmc"
   manager="carmonm"
   editor="" />
<tags
   ms.service="expressroute"
   ms.date="04/19/2016"
   wacn.date="06/06/2016"/>

# 用于设置和管理路由的路由器配置示例

本页提供适用于 Cisco IOS XE 和 Juniper MX 系列路由器的接口与路由配置示例。这些示例仅供指导，不能按原样使用。你可以与你的供应商合作，以便为你的网络指定适当的配置。

>[AZURE.IMPORTANT]本页中的示例仅供指导。你必须与供应商的销售/技术团队和你的网络团队合作，以便指定符合需要的适当配置。对于本页中所列配置的相关问题，Azure 将不提供支持。出现问题时，你必须联系设备供应商来获得支持。

以下路由器配置示例适用于所有对等互连互连。有关路由的详细信息，请查看 [ExpressRoute 对等互连](/documentation/articles/expressroute-circuit-peerings/)和 [ExpressRoute 路由要求](/documentation/articles/expressroute-routing/)。

## 基于 Cisco IOS-XE 的路由器

本部分中的示例适用于任何运行 IOS-XE 操作系统系列的路由器。

### 1\.配置接口和子接口

在连接到 Azure 的每个路由器中，针对每个对等互连互连都需要有一个子接口。子接口可使用 VLAN ID 或一组堆栈的 VLAN ID 和 IP 地址来标识。

#### Dot1Q 接口定义

本示例针对包含单个 VLAN ID 的子接口提供子接口定义。在每个对等互连中，VLAN ID 是唯一的。IPv4 地址的最后一个八位字节始终是奇数。

    interface GigabitEthernet<Interface_Number>.<Number>
     encapsulation dot1Q <VLAN_ID>
     ip address <IPv4_Address><Subnet_Mask>

#### QinQ 接口定义

本示例针对包含两个 VLAN ID 的子接口提供子接口定义。外部 VLAN ID (s-tag)，如果使用方式在所有对等互连上维持不变。在每个对等互连中，内部 VLAN ID (c-tag) 是唯一的。IPv4 地址的最后一个八位字节始终是奇数。

    interface GigabitEthernet<Interface_Number>.<Number>
     encapsulation dot1Q <s-tag> seconddot1Q <c-tag>
     ip address <IPv4_Address><Subnet_Mask>
    
### 2\.设置 eBGP 会话

必须针对每个对等互连设置与 Azure 的 BGP 会话。以下示例可让你设置与 Azure 的 BGP 会话。如果对子接口使用的 IPv4 地址是 a.b.c.d，则 BGP 邻居 (Azure) 的 IP 地址将是 a.b.c.d+1。BGP 邻居 IPv4 地址的最后一个八位字节始终是偶数。

	router bgp <Customer_ASN>
	 bgp log-neighbor-changes
	 neighbor <IP#2_used_by_Azure> remote-as 12076
	 !        
	 address-family ipv4
	 neighbor <IP#2_used_by_Azure> activate
	 exit-address-family
	!

### 3\.将前缀设置为通过 BGP 会话播发

你可以配置路由器，以将选定前缀播发给 Azure。可以使用以下示例来执行此操作。

	router bgp <Customer_ASN>
	 bgp log-neighbor-changes
	 neighbor <IP#2_used_by_Azure> remote-as 12076
	 !        
	 address-family ipv4
	  network <Prefix_to_be_advertised> mask <Subnet_mask>
	  neighbor <IP#2_used_by_Azure> activate
	 exit-address-family
	!

### 4\.路由映射

可以使用路由映射和前缀列表来筛选要传播到网络中的前缀。可以使用以下示例来完成此任务。确保已设置适当的前缀列表。

	router bgp <Customer_ASN>
	 bgp log-neighbor-changes
	 neighbor <IP#2_used_by_Azure> remote-as 12076
	 !        
	 address-family ipv4
	  network <Prefix_to_be_advertised> mask <Subnet_mask>
	  neighbor <IP#2_used_by_Azure> activate
	  neighbor <IP#2_used_by_Azure> route-map <MS_Prefixes_Inbound> in
	 exit-address-family
	!
	route-map <MS_Prefixes_Inbound> permit 10
	 match ip address prefix-list <MS_Prefixes>
	!


## Juniper MX 系列路由器 

本部分中的示例适用于所有 Juniper MX 系列路由器。

### 1\.配置接口和子接口

#### Dot1Q 接口定义

本示例针对包含单个 VLAN ID 的子接口提供子接口定义。在每个对等互连中，VLAN ID 是唯一的。IPv4 地址的最后一个八位字节始终是奇数。

    interfaces {
    	vlan-tagging;
    	<Interface_Number> {
    		unit <Number> {
    			vlan-id <VLAN_ID>;
    			family inet {
    				address <IPv4_Address/Subnet_Mask>;
    			}
    		}
    	}
    }


#### QinQ 接口定义

本示例针对包含两个 VLAN ID 的子接口提供子接口定义。外部 VLAN ID (s-tag)，如果使用方式在所有对等互连上维持不变。在每个对等互连中，内部 VLAN ID (c-tag) 是唯一的。IPv4 地址的最后一个八位字节始终是奇数。

	interfaces {
	    <Interface_Number> {
	        flexible-vlan-tagging;
	        unit <Number> {
	            vlan-tags outer <S-tag> inner <C-tag>;
	            family inet {
	                address <IPv4_Address/Subnet_Mask>;
	            }                           
	        }                               
	    }                                   
	}                           

### 2\.设置 eBGP 会话

必须针对每个对等互连设置与 Azure 的 BGP 会话。以下示例可让你设置与 Azure 的 BGP 会话。如果对子接口使用的 IPv4 地址是 a.b.c.d，则 BGP 邻居 (Azure) 的 IP 地址将是 a.b.c.d+1。BGP 邻居 IPv4 地址的最后一个八位字节始终是偶数。

	routing-options {
	    autonomous-system <Customer_ASN>;
	}
	}
	protocols {
	    bgp { 
	        group <Group_Name> { 
	            peer-as 12076;              
	            neighbor <IP#2_used_by_Azure>;
	        }                               
	    }                                   
	}

### 3\.将前缀设置为通过 BGP 会话播发

你可以配置路由器，以将选定前缀播发给 Azure。可以使用以下示例来执行此操作。

	policy-options {
	    policy-statement <Policy_Name> {
	        term 1 {
	            from protocol OSPF;
		route-filter <Prefix_to_be_advertised/Subnet_Mask> exact;
	            then {
	                accept;
	            }
	        }
	    }
	}
	protocols {
	    bgp { 
	        group <Group_Name> { 
	            export <Policy_Name>
	            peer-as 12076;              
	            neighbor <IP#2_used_by_Azure>;
	        }                               
	    }                                   
	}


### 4\.路由映射

可以使用路由映射和前缀列表来筛选要传播到网络中的前缀。可以使用以下示例来完成此任务。确保已设置适当的前缀列表。

	policy-options {
	    prefix-list MS_Prefixes {
	        <IP_Prefix_1/Subnet_Mask>;
	        <IP_Prefix_2/Subnet_Mask>;
	    }
	    policy-statement <MS_Prefixes_Inbound> {
	        term 1 {
	            from {
		prefix-list MS_Prefixes;
	            }
	            then {
	                accept;
	            }
	        }
	    }
	}
	protocols {
	    bgp { 
	        group <Group_Name> { 
	            export <Policy_Name>
	            import <MS_Prefixes_Inbound>
	            peer-as 12076;              
	            neighbor <IP#2_used_by_Azure>;
	        }                               
	    }                                   
	}

## 后续步骤

有关详细信息，请参阅 [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs/)。

<!---HONumber=82-->