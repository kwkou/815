<properties
    pageTitle="升级 Azure Service Fabric 群集 | Azure"
    description="升级运行 Service Fabric 群集的 Service Fabric 代码和/或配置，包括设置群集更新模式、升级证书、添加应用程序端口、执行操作系统修补，等等。 执行升级时你会预料到哪种结果？"
    services="service-fabric"
    documentationcenter=".net"
    author="ChackDan"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="15190ace-31ed-491f-a54b-b5ff61e718db"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="02/22/2017"
    wacn.date="04/24/2017"
    ms.author="chackdan"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="277a8a4d3472b3feff909886a513fe134e71feb6"
    ms.lasthandoff="04/14/2017" />

# <a name="upgrade-an-azure-service-fabric-cluster"></a>升级 Azure Service Fabric 群集
> [AZURE.SELECTOR]
- [Azure 群集](/documentation/articles/service-fabric-cluster-upgrade/)
- [独立群集](/documentation/articles/service-fabric-cluster-upgrade-windows-server/)

对于任何新式系统而言，为可升级性做好规划是实现产品长期成功的关键所在。 Azure Service Fabric 群集是你拥有的，但部分由 Azure 管理的资源。 本文说明自动管理的项目以及你可以自行配置的项目。

## <a name="controlling-the-fabric-version-that-runs-on-your-cluster"></a>控制群集中运行的结构版本
可以将群集设置为在 Azure 发布新版本时接收自动结构升级，或者选择要运行的受支持结构版本。

为此，请门户上设置“upgradeMode”群集配置，或者在创建时或稍后在实时群集上使用 Resource Manager 进行设置。 

> [AZURE.NOTE]
> 请确保群集始终运行受支持的结构版本。 当我们公布新版本的 Service Fabric 发布后，则标志着自该日期起至少 60 天以后结束对旧版本的支持。 新版本[在 Service Fabric 团队博客](https://blogs.msdn.microsoft.com/azureservicefabric/)上公布。 之后新版本则可供选择。 
> 
> 

群集运行的版本过期前 14 天，系统会生成运行状况事件，使群集进入警告运行状况状态。 在升级到支持的结构版本之前，群集将保持警告状态。

### <a name="setting-the-upgrade-mode-via-portal"></a>通过门户设置升级模式
创建群集时可以将群集设置为自动或手动模式。

![Create_Manualmode][Create_Manualmode]

在实时群集上可以利用管理经验将群集设置为自动或手动模式。 

#### <a name="upgrading-to-a-new-version-on-a-cluster-that-is-set-to-manual-mode-via-portal"></a>在设置为手动模式的群集上，通过门户升级到新版本。
若要升级到新版本，只需从下拉列表中选择可用的版本并保存即可。 结构升级将自动开始。 在升级过程中，将遵守群集运行状况策略（节点运行状况和所有在群集中运行的应用程序的运行状况的组合）。

如果不符合现行的群集运行状况策略，则回滚升级。 请在本文档中向下滚动，详细了解如何设置这些自定义运行状况策略。 

修复造成回滚的问题后，需要按照与之前完全相同的步骤重新启动升级。

![Manage_Automaticmode][Manage_Automaticmode]

### <a name="setting-the-upgrade-mode-via-a-resource-manager-template"></a>通过 Resource Manager 模板设置升级模式
将“upgradeMode”配置添加到 Microsoft.ServiceFabric/clusters 资源定义，将“clusterCodeVersion”设置为支持的结构版本之一，如下所示，然后部署模板。 “upgradeMode”的有效值为“Manual”或“Automatic”。

![ARMUpgradeMode][ARMUpgradeMode]

#### <a name="upgrading-to-a-new-version-on-a-cluster-that-is-set-to-manual-mode-via-a-resource-manager-template"></a>在设置为手动模式的群集上，通过 Resource Manager 模板升级到新版本。
当群集处于手动模式时，若要升级到新版本，请将“clusterCodeVersion”更改为支持的版本，然后部署该版本。 部署模板时，将自动开始结构升级。 在升级过程中，将遵守群集运行状况策略（节点运行状况和所有在群集中运行的应用程序的运行状况的组合）。

如果不符合现行的群集运行状况策略，则回滚升级。 请在本文档中向下滚动，详细了解如何设置这些自定义运行状况策略。 

解决导致回滚的问题后，需要遵循前面相同的步骤再次启动升级。

### <a name="get-list-of-all-available-version-for-all-environments-for-a-given-subscription"></a>获取给定订阅的所有环境的所有可用版本列表
运行以下命令，然后应会看到类似于下面的输出。

“supportExpiryUtc”告知给定的版本即将过期或已过期。 最新版本没有有效日期 - 它的值为“9999-12-31T23:59:59.9999999”，这只是表示尚未设置过期日期。

    	GET https://<endpoint>/subscriptions/{{subscriptionId}}/providers/Microsoft.ServiceFabric/locations/{{location}}/clusterVersions?api-version=2016-09-01

    	Example: https://management.chinacloudapi.cn/subscriptions/1857f442-3bce-4b96-ad95-627f76437a67/providers/Microsoft.ServiceFabric/locations/eastus/clusterVersions?api-version=2016-09-01

	Output:
	{
	                  "value": [
	                    {
	                      "id": "subscriptions/35349203-a0b3-405e-8a23-9f1450984307/providers/Microsoft.ServiceFabric/environments/Windows/clusterVersions/5.0.1427.9490",
	                      "name": "5.0.1427.9490",
	                      "type": "Microsoft.ServiceFabric/environments/clusterVersions",
	                      "properties": {
	                        "codeVersion": "5.0.1427.9490",
	                        "supportExpiryUtc": "2016-11-26T23:59:59.9999999",
	                        "environment": "Windows"
	                      }
	                    },
	                    {
	                      "id": "subscriptions/35349203-a0b3-405e-8a23-9f1450984307/providers/Microsoft.ServiceFabric/environments/Windows/clusterVersions/4.0.1427.9490",
	                      "name": "5.1.1427.9490",
	                      "type": " Microsoft.ServiceFabric/environments/clusterVersions",
	                      "properties": {
	                        "codeVersion": "5.1.1427.9490",
	                        "supportExpiryUtc": "9999-12-31T23:59:59.9999999",
	                        "environment": "Windows"
	                      }
	                    },
	                    {
	                      "id": "subscriptions/35349203-a0b3-405e-8a23-9f1450984307/providers/Microsoft.ServiceFabric/environments/Windows/clusterVersions/4.4.1427.9490",
	                      "name": "4.4.1427.9490",
	                      "type": " Microsoft.ServiceFabric/environments/clusterVersions",
	                      "properties": {
	                        "codeVersion": "4.4.1427.9490",
	                        "supportExpiryUtc": "9999-12-31T23:59:59.9999999",
	                        "environment": "Linux"
	                      }
	                    }
	                  ]
	                }



## <a name="fabric-upgrade-behavior-when-the-cluster-upgrade-mode-is-automatic"></a>群集升级模式为“自动”时的结构升级行为
我们将维护 Azure 群集中运行的结构代码和配置。我们将根据需要，对软件执行受监视的自动升级。升级的部分可能是代码和/或配置。为了确保应用程序不受这些升级的影响或者将影响降到最低，我们将分以下阶段执行升级：

### <a name="phase-1-an-upgrade-is-performed-by-using-all-cluster-health-policies"></a>阶段 1：使用所有群集运行状况策略执行升级
在此阶段，升级过程将每次升级一个升级域，已在群集中运行的应用程序继续运行，而不会造成任何停机时间。 在升级过程中，将遵守群集运行状况策略（节点运行状况和所有在群集中运行的应用程序的运行状况的组合）。

如果不符合现行的群集运行状况策略，则回滚升级。 然后，系统会向订阅所有者发送一封电子邮件。 电子邮件中包含以下信息：

* 有关必须回滚群集升级的通知。
* 建议的补救措施（如果有）。
* 距离执行阶段 2 的天数 (n)。

如果有任何升级因为基础结构方面的原因而失败，我们将尝试多次执行同一升级。 自电子邮件发送日期的 n 天之后，我们将继续执行阶段 2。

如果符合群集运行状况策略，则升级被视为成功并标记为完成。 在此阶段进行初始升级或重新运行任何升级期间，可能发生这种情形。 如果运行成功，将不发送任何电子邮件确认。 这是为了避免发送过多的电子邮件。收到电子邮件则表示出现异常。 大多数群集升级预期都会成功，且不影响应用程序可用性。

### <a name="phase-2-an-upgrade-is-performed-by-using-default-health-policies-only"></a>阶段 2：仅使用默认运行状况策略执行升级
在此阶段设置好运行状况策略，以便在升级开始时运行正常的应用程序数目在升级程序期间保持不变。 与阶段 1 一样，阶段 2 升级过程将每次升级一个升级域，已在群集中运行的应用程序继续运行，而不会造成任何停机时间。 在升级期间，将遵守群集运行状况策略（节点运行状况和所有在群集中运行的应用程序的运行状况的组合）。

如果不符合现行的群集运行状况策略，则回滚升级。 然后，系统会向订阅所有者发送一封电子邮件。 电子邮件中包含以下信息：

* 有关必须回滚群集升级的通知。
* 建议的补救措施（如果有）。
* 距离执行阶段 3 的天数 (n)。

如果有任何升级因为基础结构方面的原因而失败，我们将尝试多次执行同一升级。 将在 n 天结束前的几天发送提醒电子邮件。 自电子邮件发送日期的 n 天之后，我们将继续执行阶段 3。 必须认真看待阶段 2 发送的电子邮件并采取补救措施。

如果符合群集运行状况策略，则升级被视为成功并标记为完成。 在此阶段进行初始升级或重新运行任何升级期间，可能发生这种情形。 如果运行成功，将不发送任何电子邮件确认。

### <a name="phase-3-an-upgrade-is-performed-by-using-aggressive-health-policies"></a>阶段 3：使用积极的群集运行状况策略执行升级
此阶段中的这些运行状况策略旨在升级完成，而不反映应用程序的运行状况。 极少的群集升级会在此阶段结束。 如果群集进入此阶段，则表示应用程序有可能变得不正常和/或失去可用性。

类似于另外两个阶段，阶段 3 每次升级一个升级域。

如果不符合现行的群集运行状况策略，则回滚升级。 如果有任何升级因为基础结构方面的原因而失败，我们将尝试多次执行同一升级。 之后，便会锁定群集，使它不再接收支持和/或升级。

系统会将包含此信息以及补救措施的电子邮件发送给订阅所有者。 预期不会有任何群集遇到阶段 3 失败的状况。

如果符合群集运行状况策略，则升级被视为成功并标记为完成。 在此阶段进行初始升级或重新运行任何升级期间，可能发生这种情形。 如果运行成功，将不发送任何电子邮件确认。

## <a name="cluster-configurations-that-you-control"></a>你可以控制的群集配置
除了设置群集升级模式外，还可以在实时群集上更改以下配置。

### <a name="certificates"></a>证书
通过门户可以轻松为群集新增或删除证书。 请参阅[此文档中的详细说明](/documentation/articles/service-fabric-cluster-security-update-certs-azure/)

![显示 Azure 门户中证书指纹的屏幕截图。][CertificateUpgrade]

### <a name="application-ports"></a>应用程序端口
可以通过更改与节点类型关联的负载均衡器资源属性来更改应用程序端口。 可以使用门户或者直接使用 Resource Manager PowerShell。

若要在某个节点类型中的所有 VM 上打开新端口，请执行以下操作：

1. 将新探测添加到相应的负载均衡器。

    如果群集是使用门户部署的，则负载均衡器将命名为“资源组 NodeTypename 的 LB 名称”，每个节点类型各有一个负载均衡器。由于负载均衡器名称只是在资源组中唯一，因此最好在特定资源组下搜索名称。

    ![显示如何在门户中向负载均衡器添加探测的屏幕截图。][AddingProbes]  


2. 将新规则添加到负载均衡器。

    使用在上一个步骤中创建的探测，向同一负载均衡器添加新规则。

    ![在门户中向负载均衡器添加新规则。][AddingLBRules]  

### <a name="placement-properties"></a>放置属性
对于每个节点类型，可以添加要在应用程序中使用的自定义放置属性。 NodeType 是无需显式添加即可使用的默认属性。

> [AZURE.NOTE]
> 有关使用放置约束、节点属性以及如何定义它们的详细信息，请参阅 Service Fabric 群集资源管理器文档[描述群集](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/)中的“放置约束和节点属性”部分。
> 
> 

### <a name="capacity-metrics"></a>容量度量值
对于每个节点类型，可以添加要在应用程序中用于报告负载的自定义容量度量值。 有关使用容量指标来报告负载的详细信息，请参阅 Service Fabric 群集资源管理器文档[描述群集](/documentation/articles/service-fabric-cluster-resource-manager-cluster-description/)，以及[指标和负载](/documentation/articles/service-fabric-cluster-resource-manager-metrics/)。

### <a name="fabric-upgrade-settings---health-polices"></a>Fabric 升级设置 - 运行状况策略
可为结构升级指定自定义运行状况策略。 如果已将群集设置为“自动”结构升级，这些策略将应用到自动结构升级的阶段 1。
如果已将群集设置为“手动”结构升级，则每当选择新版本，因而触发系统开始在群集中进行结构升级时，就会应用这些策略。 如果未重写这些策略，则使用默认值。

在“结构升级”边栏选项卡下面，可以选择高级升级设置来指定自定义运行状况策略，或者查看当前设置。 请参考下图了解操作方法。 

![管理自定义运行状况策略][HealthPolices]

### <a name="customize-fabric-settings-for-your-cluster"></a>自定义群集的结构设置
请参阅 [Service Fabric 群集结构设置](/documentation/articles/service-fabric-cluster-fabric-settings/)了解可以如何自定义这些设置。

### <a name="os-patches-on-the-vms-that-make-up-the-cluster"></a>构成群集的 VM 上的操作系统修补
此功能即将作为自动化功能推出。 但目前你需要负责修补 VM。 必须每次修补一个 VM，以便不会同时关闭多个 VM。

### <a name="os-upgrades-on-the-vms-that-make-up-the-cluster"></a>构成群集的 VM 上的操作系统升级
如果你必须升级群集虚拟机上的操作系统映像，必须一次升级一个 VM。 你需要自行负责这种升级 - 目前没有自动化的功能用于实现此目的。

## <a name="next-steps"></a>后续步骤
* 了解如何自定义 [Service Fabric 群集结构设置](/documentation/articles/service-fabric-cluster-fabric-settings/)的部分内容
* 了解[应用程序升级](/documentation/articles/service-fabric-application-upgrade/)

<!--Image references-->
[CertificateUpgrade]: ./media/service-fabric-cluster-upgrade/CertificateUpgrade2.png
[AddingProbes]: ./media/service-fabric-cluster-upgrade/addingProbes2.PNG
[AddingLBRules]: ./media/service-fabric-cluster-upgrade/addingLBRules.png
[HealthPolices]: ./media/service-fabric-cluster-upgrade/Manage_AutomodeWadvSettings.PNG
[ARMUpgradeMode]: ./media/service-fabric-cluster-upgrade/ARMUpgradeMode.PNG
[Create_Manualmode]: ./media/service-fabric-cluster-upgrade/Create_Manualmode.PNG
[Manage_Automaticmode]: ./media/service-fabric-cluster-upgrade/Manage_Automaticmode.PNG
<!--Update_Description:wording update;add anchors to sub titles-->