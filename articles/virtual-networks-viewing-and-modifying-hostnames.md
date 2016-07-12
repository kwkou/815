<properties 
   pageTitle="查看和修改主机名 | Azure"
   description="如何查看和更改 Azure 虚拟机、Web 角色和辅助角色的主机名以进行名称解析"
   services="virtual-network"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
   editor="tysonn" />
<tags
	ms.service="virtual-network"
	ms.date="04/27/2016"
	wacn.date="05/30/2016"/>

# 查看和修改主机名

若要允许通过主机名引用角色实例，必须在服务配置文件中为每个角色设置主机名的值。可以通过将所需主机名添加到 **Role** 元素的 **vmName** 属性来执行该操作。**vmName** 属性的值将用作每个角色实例的主机名的基本元素。例如，如果 **vmName** 是 webrole，并且该角色有三个实例，则这些实例的主机名将为 webrole0、 webrole1 和 webrole2。无需在配置文件中为虚拟机指定主机名，因为虚拟机的主机名会基于虚拟机名称填充。有关配置 Azure 服务的详细信息，请参阅 [Azure Service Configuration Schema (.cscfg File)（Azure 服务配置架构（.cscfg 文件））](https://msdn.microsoft.com/zh-cn/library/azure/ee758710.aspx)

## 查看主机名

可以使用下列任一工具来查看云服务中虚拟机和角色实例的主机名。

### Azure 经典管理门户

可以使用 Azure 经典管理门户在虚拟机仪表板页上查看虚拟机的主机名。请记住，仪表板显示**名称**和**主机名**的值。尽管它们最初是相同的，但更改主机名不会更改虚拟机或角色实例的名称。

角色实例也可以位于 Azure 经典管理门户中，但当你列出云服务中的实例时，将不会显示主机名。你将看到每个实例的名称，但该名称不表示主机名。

### 服务配置文件

你可以从 Azure 经典管理门户中服务的**“配置”**页下载已部署服务的服务配置文件。然后可以查找**角色名称**元素的 **vmName** 属性以查看主机名。请记住，此主机名将用作每个角色实例的主机名的基本元素。例如，如果 **vmName** 是 *webrole*，并且该角色有三个实例，则这些实例的主机名将为 *webrole0*、*webrole1* 和 *webrole2*。

### 远程桌面

启用与你的虚拟机或角色实例的远程桌面 (Windows) 连接、Windows PowerShell 远程处理 (Windows) 连接或 SSH（Linux 和 Windows）连接后，你可以通过多种方式从活动的远程桌面连接查看主机名：

- 在命令提示符下或 SSH 终端键入主机名。

- 在命令提示符下键入 ipconfig /all（仅限 Windows）。

- 查看系统设置中的计算机名称（仅限 Windows）。

### Azure 服务管理 REST API

从 REST 客户端，按照以下说明进行操作：

1. 确保你有用于连接到 Azure 经典管理门户的客户端证书。若要获取客户端证书，请执行 [How to: Download and Import Publish Settings and Subscription Information（如何：下载和导入发布设置和订阅信息）](https://msdn.microsoft.com/zh-cn/library/dn385850.aspx)中提供的步骤。 

1. 使用值 2013-11-01 设置名为 x-ms-version 的标头条目。

1. 使用以下格式发送请求：https://management.core.chinacloudapi.cn/\<subscrition-id>/services/hostedservices/<service-name>?embed-detail=true

1. 在 **HostName** 元素中查找每个 **RoleInstance** 元素。

>[AZURE.WARNING] 你还可以通过以下方式从 REST 调用响应查看云服务的内部域后缀：查看 **InternalDnsSuffix** 元素，或通过在远程桌面会话 (Windows) 中的命令提示符下运行 ipconfig /all 或通过从 SSH 终端 (Linux) 运行 cat /etc/resolv.conf。

## 修改主机名

可以通过上载已修改的服务配置文件，或从远程桌面会话重命名计算机来修改任何虚拟机或角色实例的主机名。

## 后续步骤

[名称解析 (DNS)](/documentation/articles/virtual-networks-name-resolution-for-vms-and-role-instances/)

[Azure 服务配置架构 (.cscfg)](https://msdn.microsoft.com/zh-cn/library/azure/ee758710.aspx)

[Azure 虚拟网络配置架构](https://msdn.microsoft.com/zh-cn/library/azure/jj157100)

[使用网络配置文件指定 DNS 设置](/documentation/articles/virtual-networks-specifying-a-dns-settings-in-a-virtual-network-configuration-file/)

<!---HONumber=Mooncake_0523_2016-->