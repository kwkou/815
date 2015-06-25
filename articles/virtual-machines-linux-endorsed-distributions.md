<properties urlDisplayName="Endorsed distributions" pageTitle="Azure 中的 Linux 的认可分发" metaKeywords="" description="了解 Azure 认可的分发中的 Linux，包括 Ubuntu、OpenLogic 和 SUSE 的指南。" metaCanonical="" services="virtual-machines" documentationCenter="" title="Linux on Azure-Endorsed Distributions" authors="szark" solutions="" manager="timlt" editor="tysonn" />

<tags ms.service="virtual-machines" ms.date="04/15/2015" wacn.date="06/19/2015"/>



# Azure 认可的分发中的 Linux

Azure 库中的 Linux 映像由很多合作伙伴提供，并且我们正在与各个 Linux 社区合作，以便向“认可的分发”列表添加更多风格。在此期间，对于该库未提供的分发，你始终可以按照[本页](virtual-machines-linux-create-upload-vhd)中的指南自备 Linux。


## 支持的分发和版本 ##
 
下表列出了 Azure 支持的 Linux 分发和版本。

Hyper-V 和 Azure 的 Linux 集成服务 (LIS) 驱动程序是 Microsoft 直接为上游 Linux 内核提供的内核模块。LIS 驱动程序在默认情况下内置于分发的内核中，或者作为较旧的基于 RHEL/CentOS 的分发在[此处](http://go.microsoft.com/fwlink/?LinkID=403033&clcid=0x409)作为单独的下载提供。有关 LIS 驱动程序的详细信息，请参阅[此文](virtual-machines-linux-create-upload-vhd-generic#linux-kernel-requirements)。

Azure Linux 代理已预安装在 Linux 库映像中，并通常可从分发的包存储库中获得。源代码可在 [GitHub](https://github.com/azure/walinuxagent) 上找到。

<table border="1" width="600">
  <tr bgcolor="#E9E7E7">
		<th>分发</th>		
	    <th>版本</th>
	    <th>驱动程序</th>
		<th>代理</th>
			</tr>
	<tr>
		<th>  Canonical Ubuntu </th>
		<td> Ubuntu 12.04.1+、14.04 和 14.10 </td>
		<td>在内核中</td>
		<td>包：在“walinuxagent”下的存储库中 <br />
			源：<a href="http://go.microsoft.com/fwlink/p/?LinkID=250998">GitHub</a></td>
			</tr>
	<tr>
		<th> CentOS by OpenLogic </th>
		<td> CentOS 6.3+</td>
	    <td>CentOS 6.3：<a href="http://www.microsoft.com/zh-CN/download/details.aspx?id=41554">LIS 下载</a>；CentOS 6.4+ 驱动程序：在内核中</td>
	        CentOS 6.4+：在内核中</td>
		<td>包：在“WALinuxAgent”下的 <a href="http://olcentgbl.trafficmanager.net/openlogic/6/openlogic/x86_64/RPMS/">OpenLogic 存储库</a>中<br />
			源：<a href="http://go.microsoft.com/fwlink/p/?LinkID=250998">GitHub</a></td>
 		
	</tr>
	<!--tr>
		<th> <a href="https://coreos.com/docs/running-coreos/cloud-providers/azure/">CoreOS</a> </th>
		<td> 494.4.0+ </td>
        <td> 在内核中 </td>
		<td> 源：<a href="https://github.com/coreos/coreos-overlay/tree/master/app-emulation/wa-linux-agent">GitHub</a></td>
		
	</tr-->
	<tr>
		<th> Oracle Linux </th>
		<td> 6.4+</td>
        <td>在内核中</td>
		<td>包：在“WALinuxAgent”下的存储库中<br />
			源：<a href="http://go.microsoft.com/fwlink/p/?LinkID=250998">GitHub</a></td>
		
	</tr>
	<tr>
		<th> SUSE Linux Enterprise </th>
		<td> SLES 11 SP3+</td>
        <td>在内核中</td>
		<td>包：在“WALinuxAgent”下的 <a href="https://build.opensuse.org/project/show/Cloud:Tools" >Cloud:Tools</a> 存储库中<br />
			源：<a href="http://go.microsoft.com/fwlink/p/?LinkID=250998">GitHub</a></td>
		
	</tr>
	<tr>
		<th> openSUSE </th>
		<td> openSUSE 13.1+</td>
		<td>在内核中</td>
		<td>包：在“WALinuxAgent”下的 <a href="https://build.opensuse.org/project/show/Cloud:Tools" >Cloud:Tools</a> 存储库中<br />
			源代码：<a href="http://go.microsoft.com/fwlink/p/?LinkID=250998">GitHub</a></td>
		
	</tr>
</table>

## 合作伙伴

### Canonical
[http://www.ubuntu.com/cloud/azure](http://www.ubuntu.com/cloud/azure)

Canonical 工程和开放社区监管对 Ubuntu 在客户端、服务器和云计算（包括用户的个人云服务）方面获得成功起到了推动作用。Canonical 期望使用 Ubuntu 开发一个统一的免费平台（从手机到云），该平台带有一系列适用于手机、平板电脑、TV 和桌面的相关接口，从而使 Ubuntu 成为各种机构（从公有云提供商到消费类电子产品制造商）的首选以及各个技术专家的最爱。

借助其遍布全球的开发人员和工程中心，Canonical 在与硬件制造商、内容提供商和软件开发人员合作以将 Ubuntu 解决方案推向市场（从 PC 到服务器和手持设备）方面拥有独特的优势。


### OpenLogic
[http://www.openlogic.com/azure](http://www.openlogic.com/azure)

OpenLogic 是针对云和数据中心的企业开放源解决方案的行业领先的提供商。OpenLogic 帮助各个行业数以百计的领先企业安全获取、支持和控制开源软件。OpenLogic 为 OpenLogic 专家社区支持的 600 个开放源包提供商业级技术支持和保护（包括针对 CentOS 的企业级支持），同时作为在 Azure 上提供基于 CentOS 的映像的产品发布合作伙伴。


### Oracle
[http://www.oracle.com/technetwork/topics/cloud/faq-1963009.html](http://www.oracle.com/technetwork/topics/cloud/faq-1963009.html)

Oracle 的策略是为公有和私有云提供广泛的解决方案，同时针对如何在 Oracle 云以及其他云中部署 Oracle 软件方面给予客户选择权和灵活性。通过 Oracle 与 Microsoft 的合作关系，客户可以凭借可信的证书和 Oracle 支持在 Microsoft 公有和私有云中部署 Oracle 软件。Oracle 对 Oracle 公有和私有云的承诺和投资保持不变。


### SUSE
[http://www.suse.com/suse-linux-enterprise-server-on-azure](http://www.suse.com/suse-linux-enterprise-server-on-azure)

Azure 上的 SUSE Linux Enterprise Server 是一个已验证的平台，该平台为云计算提供了高级可靠性和安全性。SUSE 的通用 Linux 平台可与 Azure 云服务无缝集成，以便交付易于管理的云环境。借助 1,800 多个独立软件供应商提供的适用于 SUSE Linux Enterprise Server 的 9,200 多个认证应用程序，SUSE 可确保满怀信心地在 Azure 上部署数据中心内支持的运行负载。

<!---HONumber=60-->