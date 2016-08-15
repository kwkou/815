<properties
   pageTitle="升级 Service Fabric 群集 | Azure"
   description="升级运行 Service Fabric 群集的 Service Fabric 代码和/或配置，包括升级证书、添加应用程序端口、执行操作系统修补，等等。执行升级时你会预料到哪种结果？"
   services="service-fabric"
   documentationCenter=".net"
   authors="ChackDan"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="05/02/2016"
   wacn.date="07/07/2016"/>


# 升级 Service Fabric 群集

Azure Service Fabric 群集是你拥有的，但部分由 Microsoft 管理的资源。本文说明自动管理的项目以及你可以自行配置的项目。

## 自动管理的群集配置

Microsoft 将维护群集中运行的结构代码和配置。我们将根据需要，对软件执行受监视的自动升级。升级的部分可能是代码和/或配置。为了确保应用程序不受这些升级的影响或者将影响降到最低，我们将分以下三个阶段执行升级。

### 阶段 1：使用所有群集运行状况策略执行升级

在此阶段，升级过程将每次升级一个升级域，已在群集中运行的应用程序继续运行，而不会造成任何停机时间。在升级期间，将遵守群集运行状况策略（节点运行状况和所有在群集中运行的应用程序的运行状况的组合）。

如果不符合现行的群集运行状况策略，则回滚升级。然后，系统会向订阅所有者发送一封电子邮件。电子邮件中包含以下信息：

- 有关必须回滚群集升级的通知。
- 建议的补救措施（如果有）。
- 距离执行阶段 2 的天数 (N)。

如果有任何升级因为基础结构方面的原因而失败，我们将尝试多次执行同一升级。自电子邮件发送日期的 n 天之后，我们将继续执行阶段 2。

如果符合群集运行状况策略，则升级被视为成功并标记为完成。在此阶段进行初始升级或重新运行任何升级期间，可能发生这种情形。如果运行成功，将不发送任何电子邮件确认。这是为了避免发送过多的电子邮件。收到电子邮件则表示出现异常。大多数群集升级预期都会成功，且不影响应用程序可用性。

### 阶段 2：仅使用默认运行状况策略执行升级

在此阶段设置好运行状况策略，以便在升级开始时运行正常的应用程序数目在升级程序期间保持不变。与阶段 1 一样，阶段 2 升级过程将每次升级一个升级域，已在群集中运行的应用程序继续运行，而不会造成任何停机时间。在升级期间，将遵守群集运行状况策略（节点运行状况和所有在群集中运行的应用程序的运行状况的组合）。

如果不符合现行的群集运行状况策略，则回滚升级。然后，系统会向订阅所有者发送一封电子邮件。电子邮件中包含以下信息：

- 有关必须回滚群集升级的通知。
- 建议的补救措施（如果有）。
- 距离执行阶段 3 的天数 (N)。

如果有任何升级因为基础结构方面的原因而失败，我们将尝试多次执行同一升级。将在 n 天结束前的几天发送提醒电子邮件。自电子邮件发送日期的 n 天之后，我们将继续执行阶段 3。必须认真看待阶段 2 发送的电子邮件并采取补救措施。

如果符合群集运行状况策略，则升级被视为成功并标记为完成。在此阶段进行初始升级或重新运行任何升级期间，可能发生这种情形。如果运行成功，将不发送任何电子邮件确认。

### 阶段 3：使用积极的群集运行状况策略执行升级

此阶段中的这些运行状况策略旨在升级完成，而不反映应用程序的运行状况。极少的群集升级会在此阶段结束。如果群集进入此阶段，则表示应用程序有可能变得不正常和/或失去可用性。

类似于另外两个阶段，阶段 3 每次升级一个升级域。

如果不符合现行的群集运行状况策略，则回滚升级。如果有任何升级因为基础结构方面的原因而失败，我们将尝试多次执行同一升级。之后，便会锁定群集，使它不再接收支持和/或升级。

系统会将包含此信息以及补救措施的电子邮件发送给订阅所有者。预期不会有任何群集遇到阶段 3 失败的状况。

如果符合群集运行状况策略，则升级被视为成功并标记为完成。在此阶段进行初始升级或重新运行任何升级期间，可能发生这种情形。如果运行成功，将不发送任何电子邮件确认。

## 你可以控制的群集配置

下面是你可以在实时群集上更改的配置。

### 证书

可以通过对 servicefabric.cluster 资源发出 PUT 命令，来轻松更新主证书或辅助证书。

>[AZURE.NOTE] 在标识用于群集资源的证书之前，必须先完成以下步骤，否则不会使用新的证书：
1. 将新证书上载到 Azure 密钥保管库。有关说明，请参阅 [Service Fabric 安全性](/documentation/articles/service-fabric-cluster-security/)。从该文档的步骤 2 开始。
2. 更新所有构成群集的虚拟机 (VM)，以便将证书部署到这些虚拟机上。为此，请参阅 [Azure 密钥保管库团队博客](http://blogs.technet.com/b/kv/archive/2015/07/14/vm_2d00_certificates.aspx)。

### 应用程序端口

可以通过更改与节点类型关联的负载平衡器资源属性来更改应用程序端口。可以使用使用 Resource Manager PowerShell。

若要在某个节点类型中的所有 VM 上打开新端口，请执行以下操作：

1. 将新探测添加到相应的负载平衡器。


2. 将新规则添加到负载平衡器。



### 放置属性

对于每个节点类型，可以添加要在应用程序中使用的自定义放置属性。NodeType 是无需显式添加即可使用的默认属性。

>[AZURE.NOTE] 有关使用放置约束、节点属性以及如何定义它们的详细信息，请参阅 Service Fabric 群集资源管理器文档[描述群集](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/)中的“放置约束和节点属性”部分。

### 容量度量值

对于每个节点类型，可以添加要在应用程序中用于报告负载的自定义容量度量值。有关使用容量指标来报告负载的详细信息，请参阅 Service Fabric 群集资源管理器文档[描述群集](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/)以及[指标和负载](/documentation/articles/service-fabric-cluster-resource-manager-metrics/)。

### 构成群集的 VM 上的操作系统修补

此功能即将作为自动化功能推出。但目前你需要负责修补 VM。必须每次修补一个 VM，以便不会同时关闭多个 VM。

### 构成群集的 VM 上的操作系统升级

如果你必须升级群集虚拟机上的操作系统映像，必须一次升级一个 VM。你需要自行负责这种升级 - 目前没有自动化的功能用于实现此目的。

## 后续步骤

- 了解如何[扩展和缩减群集](/documentation/articles/service-fabric-cluster-scale-up-down/)
- 了解[应用程序升级](/documentation/articles/service-fabric-application-upgrade/)

<!--Image references-->
[CertificateUpgrade]: ./media/service-fabric-cluster-upgrade/CertificateUpgrade.png
[AddingProbes]: ./media/service-fabric-cluster-upgrade/addingProbes.png
[AddingLBRules]: ./media/service-fabric-cluster-upgrade/addingLBRules.png

<!---HONumber=Mooncake_0523_2016-->