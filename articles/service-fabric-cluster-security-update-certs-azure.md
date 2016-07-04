<properties
   pageTitle="更新 Azure 群集的证书 | Azure"
   description="介绍如何将新证书上载到 Azure 密钥保管库，以及更新或删除现有 Service Fabric 群集的证书。"
   services="service-fabric"
   documentationCenter=".net"
   authors="ChackDan"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="05/27/2016"
   wacn.date="07/04/2016"/>

# 在 Azure 中添加或删除 Service Fabric 群集的证书

当你在创建群集期间配置证书安全性时，Service Fabric 可让你指定两个证书（主要证书和辅助证书）。默认情况下，在创建时所指定的证书是主证书。在创建群集后，可以添加一个新证书作为辅助证书，或者删除现有证书。有关 Service Fabric 如何使用 X.509 证书的详细信息，请阅读 [Cluster security scenarios](/documentation/articles/service-fabric-cluster-security)（群集安全方案）。

>[AZURE.NOTE] 对于安全群集，始终需要至少部署一个有效（未吊销或过期）的证书（主要或辅助），否则将无法访问群集。

## 添加辅助证书
若要添加另一个证书作为辅助证书，必须将该证书上载到 Azure 密钥保管库，然后将它部署到群集中的 VM。有关更多信息，请参阅 [Deploy certificates to VMs from a customer-managed key vault](http://blogs.technet.com/b/kv/archive/2015/07/14/vm_2d00_certificates.aspx)（将证书从客户管理的密钥保管库部署到 VM）。

1. 将 X.509 证书上载到密钥保管库  
2. 登录到 [Azure 门户预览](https://portal.azure.cn/)并浏览到要将此证书添加到的群集资源。
3. 在“设置”下面，单击证书配置并输入辅助证书指纹。
4. 单击“保存”。随即将开始部署，在部署成功完成后，你可以使用主证书或辅助证书在群集上执行管理操作。

![门户中证书指纹的屏幕截图][SecurityConfigurations_02]

## 删除证书
下面是删除旧证书，以使群集不使用它的过程：

1. 登录到 [Azure 门户预览](https://portal.azure.cn/)并导航到群集的安全设置。
2. 删除其中的一个证书。
3. 单击“保存”，随即将开始部署。部署完成后，删除的证书不再可用于连接到群集。

## 后续步骤
有关群集管理的详细信息，请阅读以下文章：

- [Service Fabric 群集升级过程与期望](/documentation/articles/service-fabric-cluster-upgrade)
- [Setup role-based access for clients](/documentation/articles/service-fabric-cluster-security-roles)（为客户端设置基于角色的访问）


<!--Image references-->
[SecurityConfigurations_02]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_02.png

<!---HONumber=Mooncake_0627_2016-->