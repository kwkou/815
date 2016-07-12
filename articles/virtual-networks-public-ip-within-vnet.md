<properties 
   pageTitle="如何在虚拟网络中使用公共 IP 地址"
   description="了解如何配置虚拟网络以使用公共 IP 地址"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="carmonm"
   editor="tysonn" />
<tags
	ms.service="virtual-network"
	ms.date="04/27/2016"
	wacn.date="05/30/2016"/>

# 虚拟网络 (VNet) 中的公共 IP 地址空间

虚拟网络 (VNet) 可以包含公共和专用（RFC 1918 地址块）IP 地址空间。当你添加一个公共 IP 地址范围时，该地址将被视为只能在 VNet 内部、从互连 VNet 以及本地位置访问的专用 VNet IP 地址空间的一部分。

下图显示了包含公共和专用 IP 地址空间的 VNet。

![公共 IP 概念](./media/virtual-networks-public-ip-within-vnet/IC775683.jpg)

## 如何添加公共 IP 地址范围？

像添加专用 IP 地址范围一样添加公共 IP 地址范围 - 使用 netcfg 文件添加配置。可以在创建 VNet 时添加公共 IP 地址范围，也可以稍后再来添加。以下示例演示了在同一个 VNet 中配置的公共和专用 IP 地址空间。

![经典管理门户中的公共 IP 地址](./media/virtual-networks-public-ip-within-vnet/IC775684.png)

## 是否有任何限制？

有几个不允许使用的 IP 地址范围：

- 224\.0.0.0/4（多播）

- 255\.255.255.255/32（广播）

- 127\.0.0.0/8（环回）

- 169\.254.0.0/16（本地链路）

- 168\.63.129.16/32（内部 DNS）

## 后续步骤

[如何管理 VNet 使用的 DNS 服务器](/documentation/articles/virtual-networks-manage-dns-in-vnet/)

<!---HONumber=Mooncake_0523_2016-->