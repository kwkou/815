<properties
    pageTitle="在 Azure 门户预览中创建 Service Fabric 群集 | Azure"
    description="本文介绍如何使用 Azure 门户预览和 Azure 密钥保管库 在 Azure 中设置安全的 Service Fabric 群集。"
    services="service-fabric"
    documentationcenter=".net"
    author="chackdan"
    manager="timlt"
    editor="vturecek"
    translationtype="Human Translation" />
<tags
    ms.assetid="426c3d13-127a-49eb-a54c-6bde7c87a83b"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="02/21/2017"
    wacn.date="04/24/2017"
    ms.author="chackdan"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="a9be0c40e4d9e49cad60d7cc248b42f18120235d"
    ms.lasthandoff="04/14/2017" />

# <a name="create-a-service-fabric-cluster-in-azure-using-the-azure-portal-preview"></a>使用 Azure 门户预览在 Azure 中创建 Service Fabric 群集
> [AZURE.SELECTOR]
- [Azure 资源管理器](/documentation/articles/service-fabric-cluster-creation-via-arm/)
- [Azure 门户预览](/documentation/articles/service-fabric-cluster-creation-via-portal/)

本指南逐步介绍如何使用 Azure 门户预览在 Azure 中设置安全的 Service Fabric 群集， 其中包括以下步骤：

* 设置密钥保管库用于管理群集安全密钥。
* 通过 Azure 门户预览在 Azure 中创建安全群集。
* 使用证书对管理员进行身份验证。

> [AZURE.NOTE]
> 有关更高级的安全选项（例如使用 Azure Active Directory 进行用户身份验证和设置应用程序安全证书），请参阅 [create your cluster using Azure Resource Manager][create-cluster-arm]（使用 Azure Resource Manager 创建群集）。
> 
> 

安全的群集是防止未经授权访问管理操作的群集，这些操作包括部署、升级和删除应用程序、服务及其包含的数据。 不安全的群集是任何人都可以随时连接并执行管理操作的群集。 尽管可以创建不安全的群集，但 **强烈建议创建安全的群集**。 不安全的群集 **无法在事后受到保护** - 要保护群集，必须创建新群集。

无论群集是 Linux 群集还是 Windows 群集，创建安全群集的思路都一样。 有关创建安全 Linux 群集的详细信息和帮助器脚本，请参阅[在 Linux 上创建安全群集](/documentation/articles/service-fabric-cluster-creation-via-arm/)。 可以按[在 Azure 门户预览中创建群集](#create-cluster-portal)部分中所述，将所提供的帮助器脚本获取的参数直接输入到门户中。

## <a name="log-in-to-azure"></a>登录到 Azure
本指南使用 [Azure PowerShell][azure-powershell]。 开始新的 PowerShell 会话时，请登录到 Azure 帐户并选择订阅，然后执行 Azure 命令。

登录到 Azure 帐户：


	Login-AzureRmAccount -EnvironmentName AzureChinacloud


选择订阅：


	Get-AzureRmSubscription
	Set-AzureRmContext -SubscriptionId <guid>

## <a name="set-up-key-vault"></a>设置密钥保管库
本部分逐步介绍如何为 Azure 中的 Service Fabric 群集以及为 Service Fabric 应用程序创建密钥保管库。 有关密钥保管库的完整指南，请参阅 [Key Vault getting started guide][key-vault-get-started]（密钥保管库入门指南）。

Service Fabric 使用 X.509 证书保护群集。 Azure 密钥保管库用于管理 Azure 中 Service Fabric 群集的证书。 在 Azure 中部署群集时，负责创建 Service Fabric 群集的 Azure 资源提供程序将从密钥保管库提取证书，然后将其安装在群集 VM 上。

下图演示密钥保管库、Service Fabric 群集与 Azure 资源提供程序（在创建群集时使用密钥保管库中存储的证书）之间的关系：

![证书安装][cluster-security-cert-installation]

### <a name="create-a-resource-group"></a>创建资源组。
第一个步骤是专门针对密钥保管库创建资源组。 建议将密钥保管库放入其自身的资源组中，以便可以删除计算与存储资源组（例如包含 Service Fabric 群集的资源组），而不会丢失密钥和密码。 包含密钥保管库的资源组必须与正在使用它的群集位于同一区域。



		PS C:\Users\vturecek> New-AzureRmResourceGroup -Name mycluster-keyvault -Location 'China East'
		WARNING: The output object type of this cmdlet will be modified in a future release.
	
		ResourceGroupName : mycluster-keyvault
		Location          : chinaeast
		ProvisioningState : Succeeded
		Tags              :
		ResourceId        : /subscriptions/<guid>/resourceGroups/mycluster-keyvault


### <a name="create-key-vault"></a>创建密钥保管库
在新资源组中创建密钥保管库。 **必须针对部署启用** 密钥保管库，使 Service Fabric 资源提供程序能够从中获取证书并将其安装在群集节点上：



		PS C:\Users\vturecek> New-AzureRmKeyVault -VaultName 'myvault' -ResourceGroupName 'mycluster-keyvault' -Location 'China East' -EnabledForDeployment
	
	
		Vault Name                       : myvault
		Resource Group Name              : mycluster-keyvault
		Location                         : China East
		Resource ID                      : /subscriptions/<guid>/resourceGroups/mycluster-keyvault/providers/Microsoft.KeyVault/vaults/myvault
		Vault URI                        : https://myvault.vault.azure.cn
		Tenant ID                        : <guid>
		SKU                              : Standard
		Enabled For Deployment?          : False
		Enabled For Template Deployment? : False
		Enabled For Disk Encryption?     : False
		Access Policies                  :
		                                   Tenant ID                :    <guid>
		                                   Object ID                :    <guid>
		                                   Application ID           :
		                                   Display Name             :    
		                                   Permissions to Keys      :    get, create, delete, list, update, import, backup, restore
		                                   Permissions to Secrets   :    all
	
	
		Tags                             :


如果有现有的密钥保管库，可以使用 Azure CLI 针对部署启用该保管库：


	> azure login -e AzureChinaCloud
	> azure account set "your account"
	> azure config mode arm 
	> azure keyvault list
	> azure keyvault set-policy --vault-name "your vault name" --enabled-for-deployment true

## <a name="add-certificates-to-key-vault"></a>将证书添加到密钥保管库
证书在 Service Fabric 中用于提供身份验证和加密，为群集及其应用程序提供全方位的保护。 有关如何在 Service Fabric 中使用证书的详细信息，请参阅 [Service Fabric 群集安全方案][service-fabric-cluster-security]。

### <a name="cluster-and-server-certificate-required"></a>群集和服务器证书（必需）
需要使用此证书来保护群集以及防止未经授权访问群集。 此证书通过多种方式保护群集：

* **群集身份验证：** 在群集联合的情况下对节点间的通信进行身份验证。 只有可以使用此证书自我证明身份的节点才能加入群集。
* **服务器身份验证：** 在管理客户端上对群集管理终结点进行身份验证，使管理客户端知道它正在与真正的群集通信。 此证书还通过 HTTPS 为 HTTPS 管理 API 和 Service Fabric Explorer 提供 SSL。

为满足这些用途，该证书必须符合以下要求：

* 证书必须包含私钥。
* 必须为密钥交换创建证书，并且该证书可导出到个人信息交换 (.pfx) 文件。
* 证书的使用者名称必须与用于访问 Service Fabric 群集的域匹配。 只有符合此要求，才能为群集的 HTTPS 管理终结点和 Service Fabric Explorer 提供 SSL。 无法从证书颁发机构 (CA) 获取 `.cloudapp.chinacloudapi.cn` 域的 SSL 证书。 获取群集的自定义域名。 在从 CA 请求证书时，该证书的使用者名称必须与用于群集的自定义域名匹配。

### <a name="client-authentication-certificates"></a>客户端身份验证证书
其他客户端证书可对执行群集管理任务的管理员进行身份验证。 Service Fabric 有两个访问级别：**管理员**和**只读用户**。 至少应使用一个证书进行管理访问。 若要进行其他用户级别的访问，必须提供单独的证书。 有关访问角色的详细信息，请参阅 [role-based access control for Service Fabric clients][service-fabric-cluster-security-roles]（适用于 Service Fabric 客户端的基于角色的访问控制）。

无需将客户端身份验证证书上载到密钥保管库即可使用 Service Fabric。 只需将这些证书提供给有权管理群集的用户。 

> [AZURE.NOTE]
> 建议使用 Azure Active Directory 对执行群集管理操作的客户端进行身份验证。 若要使用 Azure Active Directory，必须[使用 Azure Resource Manager 创建群集][create-cluster-arm]。
> 
> 

### <a name="application-certificates-optional"></a>应用程序证书（可选）
可以出于应用程序安全目的，在群集上安装任意数量的附加证书。 在创建群集之前，请考虑需要在节点上安装证书的应用程序安全方案，例如：

* 加密和解密应用程序配置值
* 在复制期间跨节点加密数据 

通过 Azure 门户预览创建群集时，无法配置应用程序证书。 若要在设置群集时配置应用程序证书，必须 [使用 Azure Resource Manager 创建群集][create-cluster-arm]。 也可以在创建群集后将应用程序证书添加到群集。

### <a name="formatting-certificates-for-azure-resource-provider-use"></a>格式化证书以供 Azure 资源提供程序使用
可以直接通过密钥保管库添加和使用私钥文件 (.pfx)。 但是，Azure 资源提供程序要求以特殊 JSON 格式存储密钥，在密钥中包含 .pfx 作为 Base-64 编码字符串和私钥密码。 若要满足这些要求，必须将密钥放入 JSON 字符串，然后在密钥保管库中将其存储为 *机密* 。

为了简化此过程， [GitHub 上提供了][service-fabric-rp-helpers]一个 PowerShell 模块。 请遵循以下步骤使用该模块：

1. 将存储库的整个内容下载到本地目录。 
2. 在 PowerShell 窗口中导入该模块：


  	PS C:\Users\vturecek> Import-Module "C:\users\vturecek\Documents\ServiceFabricRPHelpers\ServiceFabricRPHelpers.psm1"

     
此 PowerShell 模块中的 `Invoke-AddCertToKeyVault` 命令自动将证书私钥的格式设置为 JSON 字符串，并将它上载到密钥保管库。使用该字符串可将群集证书与任何其他应用程序证书添加到密钥保管库。针对要在群集中安装的其他任何证书重复此步骤。


	PS C:\Users\vturecek> Invoke-AddCertToKeyVault -SubscriptionId <guid> -ResourceGroupName mycluster-keyvault -Location "China East" -VaultName myvault -CertificateName mycert -Password "<password>" -UseExistingCertificate -ExistingPfxFilePath "C:\path\to\mycertkey.pfx"
	
		Switching context to SubscriptionId <guid>
		Ensuring ResourceGroup mycluster-keyvault in China East
		WARNING: The output object type of this cmdlet will be modified in a future release.
		Using existing valut myvault in China East
		Reading pfx file from C:\path\to\key.pfx
		Writing secret to myvault in vault myvault
	
	
	Name  : CertificateThumbprint
	Value : <value>

	Name  : SourceVault
	Value : /subscriptions/<guid>/resourceGroups/mycluster-keyvault/providers/Microsoft.KeyVault/vaults/myvault

	Name  : CertificateURL
	Value : https://myvault.vault.azure.cn:443/secrets/mycert/4d087088df974e869f1c0978cb100e47



这就是配置 Service Fabric 群集 Resource Manager 模板时所要满足的所有密钥保管库先决条件。该模板可安装用于节点身份验证、管理终结点安全性与身份验证以及使用 X.509 证书的其他任何应用程序安全功能的证书。此时，应已在 Azure 中设置以下各项：

* 密钥保管库资源组
  * 密钥保管库
    * 群集服务器身份验证证书

</a "create-cluster-portal" ></a>

## <a name="create-cluster-in-the-azure-portal-preview"></a><a name="create-cluster-portal"></a>在 Azure 门户预览中创建群集
### <a name="search-for-the-service-fabric-cluster-resource"></a>搜索 Service Fabric 群集资源
![在 Azure 门户预览中搜索 Service Fabric 群集模板。][SearchforServiceFabricClusterTemplate]

 1. 登录到 [Azure 门户预览][azure-portal]。
2. 单击“**新建**”添加新的资源模板。 在“**全部**”下面的“**应用商店**”中搜索 Service Fabric 群集模板。
3. 从列表中选择“**Service Fabric 群集**”。
4. 导航到“**Service Fabric 群集**”边栏选项卡，然后单击“**创建**”。
5. “**创建 Service Fabric 群集**”边栏选项卡包含以下四个步骤。

#### <a name="1-basics"></a>1.基础知识
![创建新资源组的屏幕截图。][CreateRG]

在“基本信息”边栏选项卡中，需要提供群集的基本详细信息。

1. 输入群集的名称。
2. 输入 VM 远程桌面的**用户名**和**密码**。
3. 务必选择要将群集部署到的**订阅**，尤其是在拥有多个订阅时。
4. 创建 **新的资源组**。 最好让它与群集同名，这样稍后就可以轻松找到它们，在尝试更改部署或删除群集时非常有用。
   
   > [AZURE.NOTE]
   > 尽管你可以决定使用现有资源组，但最好还是创建新的资源组。 这样可以轻松删除不需要的群集。
   > 
   > 
5. 选择要在其中创建群集的 **区域** 。 必须使用密钥保管库所在的同一区域。

#### <a name="2-cluster-configuration"></a>2.群集配置
![创建节点类型][CreateNodeType]

配置群集节点。 节点类型定义 VM 大小、VM 数目及其属性。 群集可以有不只一个节点类型，但主节点类型（在门户定义的第一个节点类型）必须至少有 5 个 VM，因为这是 Service Fabric 系统服务放置到的节点类型。 不需要配置“**放置属性**”，因为系统会自动添加了“NodeTypeName”的默认放置属性。

> [AZURE.NOTE]
> 具有多个节点类型的常见情景是包含前端服务和后端服务的应用程序。 要将前端服务放在端口向 Internet 开放的较小型 VM（D2 等 VM 大小）上，同时要将后端服务放在没有向 Internet 开放端口的较大型 VM（D4、D6、D15 等 VM 大小）上。
> 
> 

1. 选择节点类型的名称（1 到 12 个字符，只能包含字母和数字）。
2. 主节点类型的 VM **大小**下限取决于为群集选择的**持久性**层。 持久性层的默认值为 bronze。 有关持久性的详细信息，请参阅 [how to choose the Service Fabric cluster reliability and durability][service-fabric-cluster-capacity]（如何选择 Service Fabric 群集可靠性和持久性）。
3. 选择 VM 大小和定价层。 D 系列 VM 具有 SSD 驱动器，强烈建议用于有状态应用程序。 不要使用任何具有部分核心或可用磁盘容量小于 7GB 的 VM SKU。 
4. 主节点类型的 VM **数目**下限取决于你选择的**可靠性**层。 可靠性层的默认值为 Silver。 有关可靠性的详细信息，请参阅 [how to choose the Service Fabric cluster reliability and durability][service-fabric-cluster-capacity]（如何选择 Service Fabric 群集可靠性和持久性）。
5. 选择节点类型的 VM 数目。 可在以后扩展或缩减节点类型中的 VM 数目，但数目下限取决于选择的可靠性层。 其他节点类型的下限可以是 1 个 VM。
6. 配置自定义终结点。 可在此字段中输入以逗号分隔的端口列表，可以通过 Azure 负载均衡器针对应用程序向公共 Internet 公开这些端口。 例如，如果计划在群集中部署 Web 应用程序，请在此处输入“80”，允许端口 80 的流量进入群集。 有关终结点的详细信息，请参阅 [communicating with applications][service-fabric-connect-and-communicate-with-services]
7. 配置群集 **诊断**。 默认情况下，已在群集上启用诊断，以帮助排查问题。 若要禁用诊断，请将其“**状态**”切换为“**关闭**”。 **不**建议关闭诊断。
8. 选择要将群集设置到的 Fabric 升级模式。 如果希望系统自动选取最新可用版本并尝试将你的群集升级到最新版本，则选择“**自动**”。 如果想要选择受支持的版本，则将模式设置为“**手动**”。

> [AZURE.NOTE]
> 我们仅支持那些运行受支持的 Service Fabric 版本的群集。 通过选择“**手动**”模式，由你负责将群集升级到受支持的版本。 有关结构升级模式的详细信息，请参阅 [service-fabric-cluster-upgrade 文档][service-fabric-cluster-upgrade]。
> 
> 

#### <a name="3-security"></a>3.“安全”
![Azure 门户预览上安全配置的屏幕截图。][SecurityConfigs]

最后一个步骤是使用前面创建的密钥保管库和证书信息，提供证书信息来保护群集。

* 在主证书字段中，填充使用 `Invoke-AddCertToKeyVault`PowerShell 命令将**群集证书**上载到密钥保管库后获得的输出。

	    Name  : CertificateThumbprint
	    Value : <value>

	    Name  : SourceVault
	    Value : /subscriptions/<guid>/resourceGroups/mycluster-keyvault/providers/Microsoft.KeyVault/vaults/myvault

	    Name  : CertificateURL
	    Value : https://myvault.vault.azure.cn:443/secrets/mycert/4d087088df974e869f1c0978cb100e47

* 选中“**配置高级设置**”复选框，输入**管理客户端**和**只读客户端**的客户端证书。 在这些字段中，输入管理客户端证书的指纹和只读用户客户端证书的指纹（如果适用）。 当管理员尝试连接群集时，仅当他们的证书指纹与此处输入的指纹值匹配时，才被授予访问权限。  

#### <a name="4-summary"></a><a name="step-4--complete-the-cluster-creation"></a>4.摘要
![显示“正在部署 Service Fabric 群集”的开始面板屏幕截图。 ][Notifications]

若要完成群集创建过程，请单击“**摘要**”查看提供的配置，或下载将用于部署群集的 Azure Resource Manager 模板。 在提供所有必需的设置后，“**确定**”按钮将会启用，只需单击它即可启动群集创建过程。

可以在通知栏中查看群集创建进度。 （单击屏幕右上角状态栏附近的铃铛图标）。如果在创建群集时曾经单击“**固定到启动板**”，现在会看到“**部署 Service Fabric 群集**”已固定到**开始**面板。

### <a name="view-your-cluster-status"></a>查看群集状态
![仪表板中群集详细信息的屏幕截图。][ClusterDashboard]

创建群集后，你可以在门户检查群集：

1. 转到“**浏览**”，然后单击“**Service Fabric 群集**”。
2. 找到该群集并单击它。
3. 现在，仪表板会显示群集的详细信息，包括群集的公共终结点和 Service Fabric Explorer 的链接。

群集仪表板边栏选项卡上的“**节点监视器**”部分显示运行正常和不正常的 VM 的数目。 若要了解有关群集运行状况的详细信息，请参阅 [Service Fabric 运行状况模型简介][service-fabric-health-introduction]。

> [AZURE.NOTE]
> Service Fabric 群集需要一定数量的节点始终处于开机状态，以维护可用性和保留状态（称为“维护仲裁”）。 因此，除非已事先执行 [状态的完整备份][service-fabric-reliable-services-backup-restore]，否则关闭群集中的所有计算机通常是不安全的做法。
> 
> 

## <a name="remote-connect-to-a-virtual-machine-scale-set-instance-or-a-cluster-node"></a>远程连接到虚拟机规模集实例或群集节点
每次在群集中指定 NodeTypes，都会设置 VM 规模集。 有关详细信息，请参阅 [Remote connect to a VM Scale Set instance][remote-connect-to-a-vm-scale-set] （远程连接到 VM 规模集实例）。

## <a name="next-steps"></a>后续步骤
此时，已创建一个使用证书进行管理身份验证的安全群集。 接下来，请[连接到该群集](/documentation/articles/service-fabric-connect-to-secure-cluster/)，了解如何[管理应用程序机密](/documentation/articles/service-fabric-application-secret-management/)。  此外，了解 [Service Fabric 支持选项](/documentation/articles/service-fabric-support/)。

<!-- Links -->
[azure-powershell]: /documentation/articles/powershell-install-configure/
[service-fabric-rp-helpers]: https://github.com/ChackDan/Service-Fabric/tree/master/Scripts/ServiceFabricRPHelpers
[azure-portal]: https://portal.azure.cn/
[key-vault-get-started]: /documentation/articles/key-vault-get-started/
[create-cluster-arm]: /documentation/articles/service-fabric-cluster-creation-via-arm/
[service-fabric-cluster-security]: /documentation/articles/service-fabric-cluster-security/
[service-fabric-cluster-security-roles]: /documentation/articles/service-fabric-cluster-security-roles/
[service-fabric-cluster-capacity]: /documentation/articles/service-fabric-cluster-capacity/
[service-fabric-connect-and-communicate-with-services]: /documentation/articles/service-fabric-connect-and-communicate-with-services/
[service-fabric-health-introduction]: /documentation/articles/service-fabric-health-introduction/
[service-fabric-reliable-services-backup-restore]: /documentation/articles/service-fabric-reliable-services-backup-restore/
[remote-connect-to-a-vm-scale-set]: /documentation/articles/service-fabric-cluster-nodetypes/#remote-connect-to-a-vm-scale-set-instance-or-a-cluster-node
[service-fabric-cluster-upgrade]: /documentation/articles/service-fabric-cluster-upgrade/

<!--Image references-->
[SearchforServiceFabricClusterTemplate]: ./media/service-fabric-cluster-creation-via-portal/SearchforServiceFabricClusterTemplate.png
[CreateRG]: ./media/service-fabric-cluster-creation-via-portal/CreateRG.png
[CreateNodeType]: ./media/service-fabric-cluster-creation-via-portal/NodeType.png
[SecurityConfigs]: ./media/service-fabric-cluster-creation-via-portal/SecurityConfigs.png
[Notifications]: ./media/service-fabric-cluster-creation-via-portal/notifications.png
[ClusterDashboard]: ./media/service-fabric-cluster-creation-via-portal/ClusterDashboard.png
[cluster-security-cert-installation]: ./media/service-fabric-cluster-creation-via-arm/cluster-security-cert-installation.png
<!--Update_Description:add linux content;wording update;add anchors to sub titles-->