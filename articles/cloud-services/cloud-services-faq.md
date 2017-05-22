<properties
    pageTitle="Azure 云服务角色常见问题解答 | Azure"
    description="有关 Azure 云服务的常见问题解答。 解答有关证书、Web 角色和辅助角色的一些常见问题。"
    services="cloud-services"
    documentationcenter=""
    author="Thraka"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="84985660-2cfd-483a-8378-50eef6a0151d"
    ms.service="cloud-services"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/16/2016"
    ms.author="adegeo"
    wacn.date="05/22/2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="e85c415dde18716f6433e505d81028c28e0b2da9"
    ms.lasthandoff="05/12/2017" />

# <a name="cloud-services-faq"></a>云服务常见问题
本文回答了一些关于 Microsoft Azure 云服务的常见问题。 你还可以访问 [Azure 支持 FAQ](http://go.microsoft.com/fwlink/?LinkID=185083) 了解一般的 Azure 定价和支持信息。 还可以参阅[云服务 VM 大小页面](/documentation/articles/cloud-services-sizes-specs/)，了解大小信息。

## <a name="certificates"></a>证书
### <a name="where-should-i-install-my-certificate"></a>应该在何处安装我的证书？
- **My**  
 具有私钥的应用程序证书（*.pfx、*.p12）。

- **CA**  
所有中间证书都会放入此存储（策略和子 CA）。

- **ROOT**  
根 CA 存储，因此应将主要的根 CA 证书放在此处。

### <a name="i-cant-remove-expired-certificate"></a>无法删除过期的证书
Azure 会阻止删除正在使用的证书。 需要删除使用该证书的部署，或使用其他证书或续订证书来更新部署。

### <a name="delete-an-expired-certificate"></a>删除过期的证书

只要未在使用证书，就可以使用 [Remove-AzureCertificate](https://msdn.microsoft.com/zh-cn/library/azure/mt589145.aspx) PowerShell cmdlet 删除该证书。

### <a name="i-have-expired-certificates-named-windows-azure-service-management-for-extensions"></a>我拥有名为 Microsoft Azure Service Management for Extensions 的已过期证书
每次将扩展（例如远程桌面扩展）添加到云服务中时，就会创建这些证书。 这些证书仅用于加密和解密扩展的专用配置。 这些证书是否过期并不重要。 系统不会检查过期日期。

### <a name="certificates-i-have-deleted-keep-reappearing"></a>已删除的证书一直重复出现
这些证书很可能是因为正在使用的某个工具而一直重复出现，例如 Visual Studio。 每次重新连接使用证书的工具时，证书将再次上传到 Azure。

### <a name="my-certificates-keep-disappearing"></a>我的证书一直消失
虚拟机实例回收时，所有本地更改都将丢失。 使用[启动任务](/documentation/articles/cloud-services-startup-tasks/)在每次角色启动时将证书安装到虚拟机。

### <a name="i-cannot-find-my-management-certificates-in-the-portal"></a>我在门户中找不到管理证书
[管理证书](/documentation/articles/azure-api-management-certs/)仅在 Azure 经典管理门户中提供。当前的 Azure 门户不使用管理证书。

### <a name="how-can-i-disable-a-management-certificate"></a>如何禁用管理证书？
[管理证书](/documentation/articles/azure-api-management-certs/)无法禁用。不想再使用它们时，可以通过 Azure 经典管理门户进行删除。

### 如何为特定 IP 地址创建 SSL 证书？
按照[创建证书教程](/documentation/articles/cloud-services-certs-create/)中的说明操作。使用 IP 地址作为 DNS 名称。

## <a name="security"></a>“安全”
### <a name="disable-ssl-30"></a>禁用 SSL 3.0
若要禁用 SSL 3.0 并使用 TLS 安全，可创建在以下博客文章中记录的启动任务：https://azure.microsoft.com/zh-cn/blog/how-to-disable-ssl-3-0-in-azure-websites-roles-and-virtual-machines/

### <a name="add-nosniff-to-your-website"></a>将 **nosniff** 添加到网站
若要防止客户端探查 MIME 类型，请在 *web.config* 文件中添加设置。

    <configuration>
       <system.webServer>
          <httpProtocol>
             <customHeaders>
                <add name="X-Content-Type-Options" value="nosniff" />
             </customHeaders>
          </httpProtocol>
       </system.webServer>
    </configuration>

也可以将此项作为 IIS 中的设置添加。 使用[常见启动任务](/documentation/articles/cloud-services-startup-tasks-common/#configure-iis-startup-with-appcmdexe)一文中的以下命令。

    %windir%\system32\inetsrv\appcmd set config /section:httpProtocol /+customHeaders.[name='X-Content-Type-Options',value='nosniff']

### <a name="customize-iis-for-a-web-role"></a>为 web 角色自定义 IIS
使用[常见启动任务](/documentation/articles/cloud-services-startup-tasks-common/#configure-iis-startup-with-appcmdexe)一文中的 IIS 启动脚本。

## <a name="scaling"></a>扩展
### <a name="i-cannot-scale-beyond-x-instances"></a>无法扩展到 X 个实例以上
Azure 订阅对可以使用的内核数存在限制。 如果已使用所有可用的内核，缩放无法执行。 例如，如果限制为 100 个核心，这意味着云服务可以有大小为 100 A1 的虚拟机实例或大小为 50 A2 的虚拟机实例。

## <a name="networking"></a>联网
### <a name="i-cant-reserve-an-ip-in-a-multi-vip-cloud-service"></a>无法在多 VIP 云服务中保留 IP

首先，请确保已打开想要为其保留 IP 的虚拟机实例。其次，请确保为过渡和生产部署使用保留 IP。**请勿**在部署升级过程中更改设置。

## <a name="remote-desktop"></a>远程桌面
### <a name="how-do-i-remote-desktop-when-i-have-an-nsg"></a>设置了 NSG 时，如何使用远程桌面？
将规则添加到 NSG，允许端口 **3389** 和 **20000** 上的流量。  远程桌面使用端口 **3389**。  云服务实例经过负载均衡，因此无法直接控制要连接到哪个实例。  RemoteForwarder 和 RemoteAccess 代理管理 RDP 流量，允许客户端发送 RDP cookie 和指定要连接到的单个实例。  RemoteForwarder 和 RemoteAccess 代理要求打开端口 **20000***（如果你具有 NSG，此端口可能已被阻止）。