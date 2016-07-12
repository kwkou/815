<properties
   pageTitle="测试流量管理器设置"
   description="本文将帮助你测试流量管理器设置。"
   services="traffic-manager"
   documentationCenter="na"
   authors="joaoma"
   manager="adinah"
   editor="tysonn" />
<tags
	ms.service="traffic-manager"
	ms.date="03/17/2016"
	wacn.date="04/18/2016"/>

# 测试流量管理器设置

测试流量管理器设置的最佳方法是设置许多客户端，然后逐个关闭配置文件中的终结点（包括云服务和网站）。以下技巧可帮助你测试流量管理器配置文件。

## 基本测试步骤

-**将 DNS TTL 设置得非常低**以便快速传播更改，例如，30 秒。

-**了解你要测试的配置文件中的 Azure 云服务和网站的 IP 地址**。

-**使用能够让你将 DNS 名称解析为 IP 地址**并显示该地址的工具。你将查看公司域名是否可以解析为配置文件中的终结点的 IP 地址。解析方式应与流量管理器配置文件中的负载平衡方法一致。如果你的计算机运行的是 Windows，则可以在命令提示符或 Windows PowerShell 提示符下使用 Nslookup.exe 工具。在 Internet 上还可以找到其他公开发布的用于“挖掘”IP 地址的现成工具。

### 使用 nslookup 检查流量管理器配置文件

1-以管理员身份打开命令提示符或 Windows PowerShell 提示符。

2-键入 `ipconfig /flushdns` 以刷新 DNS 解析程序缓存。

3-键入 `nslookup <your Traffic Manager domain name>`。例如，以下命令将检查前缀为 *myapp.contoso* nslookup myapp.contoso.trafficmanager.cn 的域名。典型结果将显示以下：
- 为解析此流量管理器域名而访问的 DNS 服务器的 DNS 名称和 IP 地址。
- 在命令行中“nslookup”后键入的流量管理器域名以及该流量管理器域解析为的 IP 地址。需要重点检查第二个 IP 地址。它应当与所测试的流量管理器配置文件中某个云服务或网站的公用虚拟 IP (VIP) 地址匹配。

## 测试负载平衡方法


### 测试“故障转移”负载平衡方法

1-使所有终结点保持运行状态。
2-使用单一客户端。
3-使用 Nslookup.exe 工具或类似的实用工具请求对公司域名进行 DNS 解析。
4-确保你得到的已解析 IP 地址是主终结点的 IP 地址。
5-关闭主终结点或删除监视文件，使流量管理器认为主终结点已关闭。
6-等待流量管理器配置文件的 DNS 生存时间 (TTL) 再额外等待 2 分钟。例如，如果 DNS TTL 为 300 秒（5 分钟），则你必须等待 7 分钟。
7-刷新 DNS 客户端缓存，然后请求 DNS 解析。在 Windows 中，可以通过在命令提示符或 Windows PowerShell 提示符下发出 ipconfig /flushdns 命令来刷新 DNS 缓存。
8-确保你得到的 IP 地址是第二个终结点的 IP 地址。
9-重复该过程，关闭第二个终结点，然后再关闭第三个终结点，依此类推。每次都要确保 DNS 解析返回列表中下一个终结点的 IP 地址。关闭所有终结点后，应该再次得到主终结点的 IP 地址。

### 测试“循环”负载平衡方法

1-使所有终结点保持运行状态。
2-使用单一客户端。
3-使用 Nslookup.exe 工具或类似的实用工具请求对公司域进行 DNS 解析。
4-确保你得到的 IP 地址是你的列表中的某个地址。
5-刷新 DNS 客户端缓存，并一次又一次地重复步骤 3 和 4。你应该会看到，为每个终结点返回的 IP 地址都不相同。接下来，请重复该过程。

### 测试“性能”负载平衡方法

若要有效地测试“性能”负载平衡方法，你必须在世界各地拥有客户端。可以在 Azure 中创建客户端，该客户端将尝试通过公司域名调用你的服务。另外，如果你的公司在全球经营，则你可以远程登录到位于世界其他区域的客户端，并从这些客户端进行测试。

你可以使用基于 Web 的免费 DNS 查找和挖掘服务。这些服务中，有些可以从不同位置检查 DNS 名称解析。例如，针对“DNS 查找”执行搜索。另一种做法是使用 Gomez 或 Keynote 等第三方解决方案来确认配置文件是否按预期分配流量。

## 后续步骤

[关于流量管理器流量路由方法](/documentation/articles/traffic-manager-routing-methods/)[流量管理器](/documentation/articles/traffic-manager-overview/)

<!---HONumber=Mooncake_0411_2016-->