<properties linkid="manage-linux-other-resources-endorsed-distributions" urlDisplayName="Endorsed distributions" pageTitle="Azure 中的 Linux 的认可分发" metaKeywords="" description="了解 Azure 认可的分发中的 Linux，包括 Ubuntu、OpenLogic 和 SUSE 的指南。" metaCanonical="" services="virtual-machines" documentationCenter="" title="Azure 认可的分发中的 Linux" authors="kathydav" solutions="" manager="jeffreyg" editor="tysonn" />

# Azure 认可的分发中的 Linux

库中的分发映像由以下合作伙伴提供，我们正在与社区协作以便提供更多认可的分发。同时，您始终可通过按照本页中的指南进行操作来引入您自己的 Linux。

## Canonical

<http://www.ubuntu.com/cloud/azure>

Canonical 工程和开放社区监管对 Ubuntu 在客户端、服务器和云计算（包括用户的个人云服务）方面获得成功起到了推动作用。Canonical 期望使用 Ubuntu 开发一个统一的免费平台（从手机到云），该平台带有一系列适用于手机、平板电脑、TV 和桌面的相关接口，从而使 Ubuntu 成为各种机构（从公有云提供商到消费类电子产品制造商）的首选以及各个技术专家的最爱。

借助其遍布全球的开发人员和工程中心，Canonical 在与硬件制造商、内容提供商和软件开发人员合作以将 Ubuntu 解决方案推向市场（从 PC 到服务器和手持设备）方面拥有独特的优势。

## OpenLogic

<http://www.openlogic.com/azure>

OpenLogic 是针对云和数据中心的企业开放源解决方案的行业领先的提供商。OpenLogic 帮助各个行业数以百计的领先企业安全获取、支持和控制开源软件。OpenLogic 为 OpenLogic 专家社区支持的 600 个开放源包提供商业级技术支持和保护（包括针对 CentOS 的企业级支持），同时作为在 Azure 上提供 Centos 映像的产品发布合作伙伴。

## Oracle

<http://www.oracle.com/technetwork/topics/cloud/faq-1963009.html>

Oracle 的策略是为公有和私有云提供广泛的解决方案，同时针对如何在 Oracle 云以及其他云中部署 Oracle 软件方面给予客户选择权和灵活性。通过 Oracle 与 Microsoft 的合作关系，客户可以凭借可信的证书和 Oracle 支持在 Microsoft 公有和私有云中部署 Oracle 软件。Oracle 对 Oracle 公有和私有云的承诺和投资保持不变。

## SUSE

<http://www.suse.com/suse-linux-enterprise-server-on-azure>

Azure 上的 SUSE Linux Enterprise Server 是一个已验证的平台，该平台为云计算提供了高级可靠性和安全性。SUSE 的通用 Linux 平台可与 Azure 云服务无缝集成，以便交付易于管理的云环境。借助 1,800 多个独立软件供应商提供的适用于 SUSE Linux Enterprise Server 的 9,200 多个认证应用程序，SUSE 可确保满怀信心地在 Azure 上部署数据中心内支持的运行负载。

## 支持的版本

下表显示了不同的分发版本，Linux Integration Services (LIS) 驱动程序和已经过测试可在 Azure 上运行的 Azure Linux 代理版本。LIS 驱动程序在 <http://www.microsoft.com/zh-cn/download/details.aspx?id=34603> 中提供。Linux 代理版本在 <https://github.com/Azure/WALinuxAgent> 中提供。

该表还包括指向某些分发版本在 Azure 中以最佳方式运行所需的内核兼容性修补程序的链接。

<table border="1" width="600">
<tr bgcolor="#E9E7E7">
<th>
分发

</th>
<th>
版本

</th>
<th>
驱动程序

</th>
<th>
内核兼容性修补程序

</th>
<th>
代理

</th>
</tr>
<tr>
<th>
Canonical UBUNTU

</th>
<td>
Ubuntu 12.04.1、12.10 和 13.04

</td>
<td>
在内核中

</td>
<td>
[仅需要 12.04 或 12.04.01][仅需要 12.04 或 12.04.01]

</td>
<td>
包：在 walinuxagent 下的包存储库中
 源：[GITHUB][GITHUB]

</td>
</tr>
<tr>
<th>
CENTOS by Open Logic

</th>
<td>
CentOS 6.3+

</td>
<td>
CentOS 6.3：[LIS 驱动程序][LIS 驱动程序]；CentOS 6.4+ 驱动程序：在内核中

</td>
<td>
[仅需要 6.3][仅需要 6.3]

</td>
<td>
包：在 walinuxagent 下的 [Open Logic 包存储库][Open Logic 包存储库]中
 源：[GITHUB][GITHUB]

</td>
</tr>
<tr>
<th>
Oracle Linux

</th>
<td>
6.4+

</td>
<td>
在内核中

</td>
<td>
不适用

</td>
<td>
包：在存储库中，名称：WALinuxAgent
 源：[GITHUB][GITHUB]

</td>
</tr>
<tr>
<th>
SUSE Linux Enterprise

</th>
<td>
SLES 11 SP3+

</td>
<td>
在内核中

</td>
<td>
不适用

</td>
<td>
包：在 [Cloud:Tools][Cloud:Tools] 存储库中，名称：WALinuxAgent
 源代码：[GITHUB][GITHUB]

</td>
</tr>
<tr>
<th>
openSUSE

</th>
<td>
OpenSUSE 13.1+

</td>
<td>
在内核中

</td>
<td>
不适用

</td>
<td>
包：在 [Cloud:Tools][Cloud:Tools] 存储库中，名称：WALinuxAgent
 源代码：[GITHUB][GITHUB]

</td>
</tr>
</table>

  [仅需要 12.04 或 12.04.01]: http://go.microsoft.com/fwlink/?LinkID=275152&clcid=0x409
  [GITHUB]: http://go.microsoft.com/fwlink/p/?LinkID=250998
  [LIS 驱动程序]: http://go.microsoft.com/fwlink/p/?LinkID=254263
  [仅需要 6.3]: http://go.microsoft.com/fwlink/?LinkID=275153&clcid=0x409
  [Open Logic 包存储库]: http://olcentgbl.trafficmanager.net/openlogic/6/openlogic/x86_64/RPMS/
  [Cloud:Tools]: https://build.opensuse.org/project/show/Cloud:Tools
