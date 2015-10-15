<properties
   pageTitle="使用 PowerShell 和 Azure CLI 来浏览和选择 Azure 虚拟机映像"
   description="了解在使用资源管理器创建 Azure 虚拟机时如何确定映像的确定发布者、产品和 SKU。"
   services="virtual-machines"
   documentationCenter=""
   authors="squillace"
   manager="timlt"
   editor=""/>

<tags
   ms.service="virtual-machines"
   ms.date="05/29/2015"
   wacn.date="09/15/2015"/>

# 使用 PowerShell 和 Azure CLI 来浏览和选择 Azure 虚拟机映像

> [AZURE.NOTE]当你在本主题中搜索 VM 映像时，将要在 [Azure 资源管理模式](/documentation/articles/resource-group-overview)下，使用最近安装的适用于 Mac、Linux 和 Windows 的 Azure 命令行界面，或者使用 Windows PowerShell。使用 Azure CLI 时，键入 `azure config mode arm` 即可进入该模式。使用 PowerShell 时，请键入 `Switch-AzureMode AzureResourceManager`。有关完整的更新和配置详细信息，请参阅[将 Azure CLI 与资源管理配合使用](/documentation/articles/xplat-cli-azure-resource-manager)和[将 PowerShell 与 Azure 资源管理配合使用](/documentation/articles/powershell-azure-resource-manager)。

## 常用映像表


| PublisherName | 产品 | SKU |
|:---------------------------------|:-------------------------------------------|:---------------------------------|:--------------------|
| OpenLogic | CentOS | 7 |
| OpenLogic | CentOS | 7\.1 |
| CoreOS | CoreOS | Beta |
| CoreOS | CoreOS | Stable |
| MicrosoftDynamicsNAV | DynamicsNAV | 2015 |
| MicrosoftSharePoint | MicrosoftSharePointServer | 2013 |
| msopentech | Oracle-Database-12c-Weblogic-Server-12c | 标准 |
| msopentech | Oracle-Database-12c-Weblogic-Server-12c | Enterprise |
| MicrosoftSQLServer | SQL2014-WS2012R2 | Enterprise-Optimized-for-DW |
| MicrosoftSQLServer | SQL2014-WS2012R2 | Enterprise-Optimized-for-OLTP |
| Canonical | UbuntuServer | 12\.04.5-LTS |
| Canonical | UbuntuServer | 14\.04.2-LTS |
| MicrosoftWindowsServer | WindowsServer | 2012-Datacenter |
| MicrosoftWindowsServer | WindowsServer | 2012-R2-Datacenter |
| MicrosoftWindowsServer | WindowsServer | 2008-R2-SP1 |
| MicrosoftWindowsServer | WindowsServer | Windows-Server-Technical-Preview |
| MicrosoftWindowsServerEssentials | WindowsServerEssentials | WindowsServerEssentials |
| MicrosoftWindowsServerHPCPack | WindowsServerHPCPack | 2012R2 |


## Azure CLI

查找要与 `azure vm quick-create` 配合使用的映像或创建资源组模板文件的最简单且最快速方式是调用 `azure vm image list` 命令，并传递位置、发布者名称（不区分大小写！）和提供产品（如果你知道该产品）。例如，如果你知道“Canonical”是“UbuntuServer”产品的发布者，则以下列表只是简短的示例（许多列表相当长）。

    azure vm image list chinanorth canonical ubuntuserver
    info:    Executing command vm image list
    warn:    The parameter --sku if specified will be ignored
    + Getting virtual machine image skus (Publisher:"canonical" Offer:"ubuntuserver" Location:"chinanorth")
    data:    Publisher  Offer         Sku          Version          Location  Urn
    data:    ---------  ------------  -----------  ---------------  --------  --------------------------------------------------
    data:    canonical  ubuntuserver  12.04-DAILY  12.04.201504201  chinanorth    canonical:ubuntuserver:12.04-DAILY:12.04.201504201
    data:    canonical  ubuntuserver  12.04.2-LTS  12.04.201302250  chinanorth    canonical:ubuntuserver:12.04.2-LTS:12.04.201302250
    data:    canonical  ubuntuserver  12.04.2-LTS  12.04.201303250  chinanorth    canonical:ubuntuserver:12.04.2-LTS:12.04.201303250
    data:    canonical  ubuntuserver  12.04.2-LTS  12.04.201304150  chinanorth    canonical:ubuntuserver:12.04.2-LTS:12.04.201304150
    data:    canonical  ubuntuserver  12.04.2-LTS  12.04.201305160  chinanorth    canonical:ubuntuserver:12.04.2-LTS:12.04.201305160
    data:    canonical  ubuntuserver  12.04.2-LTS  12.04.201305270  chinanorth    canonical:ubuntuserver:12.04.2-LTS:12.04.201305270
    data:    canonical  ubuntuserver  12.04.2-LTS  12.04.201306030  chinanorth    canonical:ubuntuserver:12.04.2-LTS:12.04.201306030
    data:    canonical  ubuntuserver  12.04.2-LTS  12.04.201306240  chinanorth    canonical:ubuntuserver:12.04.2-LTS:12.04.201306240

**Urn** 列是你传递给 `azure vm quick-create` 的表单。

不过，你通常还不知道什么可用。在此情况下，你可以浏览映像，方法是先使用 `azure vm image list-publishers` 发现发布者，并以你预期用于资源组的数据中心位置来响应位置提示符。例如，以下列表是中国北部位置的所有映像发布者（将位置设为小写并删除标准位置中的空格，以传递位置参数）。

    azure vm image list-publishers
    info:    Executing command vm image list-publishers
    Location: chinanorth
    + Getting virtual machine image publishers (Location: "chinanorth")
    data:    Publisher                                       Location
    data:    ----------------------------------------------  --------
    data:    a10networks                                     chinanorth  
    data:    aiscaler-cache-control-ddos-and-url-rewriting-  chinanorth  
    data:    alertlogic                                      chinanorth  
    data:    AlertLogic.Extension                            chinanorth  


这些列表可能相当长，因此上述示例列表只是程序代码段。假设我注意到 Canonical 事实上是中国北部位置的映像发布者。你现在可以调用 `azure vm image list-offers 来查找其产品，并在提示符下传递位置和发布者，如以下示例所示：

    azure vm image list-offers
    info:    Executing command vm image list-offers
    Location: chinanorth
    Publisher: canonical
    + Getting virtual machine image offers (Publisher: "canonical" Location:"chinanorth")
    data:    Publisher  Offer         Location
    data:    ---------  ------------  --------
    data:    canonical  UbuntuServer  chinanorth  
    info:    vm image list-offers command OK

现在，我们知道在中国北部区域中，Canonical 在 Azure 上发布 **UbuntuServer** 产品。但是，是什么 SKU？ 若要获取那些项目，请调用 `azure vm image list-skus`，并使用你发现的位置、发布者和产品来响应提示。

    azure vm image list-skus
    info:    Executing command vm image list-skus
    Location: chinanorth
    Publisher: canonical
    Offer: ubuntuserver
    + Getting virtual machine image skus (Publisher:"canonical" Offer:"ubuntuserver" Location:"chinanorth")
    data:    Publisher  Offer         sku          Location
    data:    ---------  ------------  -----------  --------
    data:    canonical  ubuntuserver  12.04-DAILY  chinanorth  
    data:    canonical  ubuntuserver  12.04.2-LTS  chinanorth  
    data:    canonical  ubuntuserver  12.04.3-LTS  chinanorth  
    data:    canonical  ubuntuserver  12.04.4-LTS  chinanorth  
    data:    canonical  ubuntuserver  12.04.5-LTS  chinanorth  
    data:    canonical  ubuntuserver  12.10        chinanorth  
    data:    canonical  ubuntuserver  14.04-beta   chinanorth  
    data:    canonical  ubuntuserver  14.04-DAILY  chinanorth  
    data:    canonical  ubuntuserver  14.04.0-LTS  chinanorth  
    data:    canonical  ubuntuserver  14.04.1-LTS  chinanorth  
    data:    canonical  ubuntuserver  14.04.2-LTS  chinanorth  
    data:    canonical  ubuntuserver  14.10        chinanorth  
    data:    canonical  ubuntuserver  14.10-beta   chinanorth  
    data:    canonical  ubuntuserver  14.10-DAILY  chinanorth  
    data:    canonical  ubuntuserver  15.04        chinanorth  
    data:    canonical  ubuntuserver  15.04-beta   chinanorth  
    data:    canonical  ubuntuserver  15.04-DAILY  chinanorth  
    info:    vm image list-skus command OK

现在，使用此信息在顶部调用原始调用，即可找到所需的映像。

    azure vm image list chinanorth canonical ubuntuserver 14.04.2-LTS
    info:    Executing command vm image list
    + Getting virtual machine images (Publisher:"canonical" Offer:"ubuntuserver" Sku: "14.04.2-LTS" Location:"chinanorth")
    data:    Publisher  Offer         Sku          Version          Location  Urn
    data:    ---------  ------------  -----------  ---------------  --------  --------------------------------------------------
    data:    canonical  ubuntuserver  14.04.2-LTS  14.04.201503090  chinanorth    canonical:ubuntuserver:14.04.2-LTS:14.04.201503090
    data:    canonical  ubuntuserver  14.04.2-LTS  14.04.20150422   chinanorth    canonical:ubuntuserver:14.04.2-LTS:14.04.20150422
    data:    canonical  ubuntuserver  14.04.2-LTS  14.04.201504270  chinanorth    canonical:ubuntuserver:14.04.2-LTS:14.04.201504270
    info:    vm image list command OK

现在，你可以确切地选择想要使用的映像。若要使用你刚刚找到的 URN 信息快速创建 VM，或要使用包含该 URN 信息的模板，请参阅[将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure 资源管理配合使用](/documentation/articles/xplat-cli-azure-resource-manager)。

### 视频演练

此视频演示如何使用 CLI 完成上述步骤。

[AZURE.VIDEO resource-groups-vm-searching-cli]


## PowerShell


使用 Azure 资源管理器创建新的虚拟机时，在某些情况下，你需要使用以下映像属性组合来指定映像：

- 发布者
- 产品
- SKU

例如，**Set-AzureVMSourceImage PowerShell** cmdlet 或资源组模板文件需要这些值，而在此文件中，你必须指定要创建的虚拟机类型。

如果你需要确定这些值，可以浏览映像以确定这些值，方法是：

1. 列出映像发布者。
2. 对于给定的发布者，列出其产品。
3. 对于给定的产品，列出其 SKU。

若要在 PowerShell 中执行此操作，请先切换到 Azure PowerShell 的资源管理器模式。

	Switch-AzureMode AzureResourceManager

在上面第一个步骤中，使用这些命令列出发布者。

	$locName="<Azure location, such as China North>"
	Get-AzureVMImagePublisher -Location $locName | Select PublisherName

填写选择的发布者名称，然后运行以下命令。

	$pubName="<publisher>"
	Get-AzureVMImageOffer -Location $locName -Publisher $pubName | Select Offer

填写选择的产品名称，然后运行以下命令。

	$offerName="<offer>"
	Get-AzureVMImageSku -Location $locName -Publisher $pubName -Offer $offerName | Select Skus

从 **Get-AzureVMImageSku** 命令的输出来看，你已获得了所需的所有信息，可以为新虚拟机指定映像。

下面是一个示例。

	PS C:\> $locName="China North"
	PS C:\> Get-AzureVMImagePublisher -Location $locName | Select PublisherName

	PublisherName
	-------------
	a10networks
	aiscaler-cache-control-ddos-and-url-rewriting-
	alertlogic
	AlertLogic.Extension
	Barracuda.Azure.ConnectivityAgent
	barracudanetworks
	basho
	boxless
	bssw
	Canonical
	...

对于“MicrosoftWindowsServer”发布者：

	PS C:\> $pubName="MicrosoftWindowsServer"
	PS C:\> Get-AzureVMImageOffer -Location $locName -Publisher $pubName | Select Offer

	Offer
	-----
	WindowsServer

对于“WindowsServer”产品：

	PS C:\> $offerName="WindowsServer"
	PS C:\> Get-AzureVMImageSku -Location $locName -Publisher $pubName -Offer $offerName | Select Skus

	Skus
	----
	2008-R2-SP1
	2012-Datacenter
	2012-R2-Datacenter
	Windows-Server-Technical-Preview

从此列表中复制选择的 SKU 名称，然后你将获得 **Set-AzureVMSourceImage** PowerShell cmdlet 或资源组模板文件的所有信息，而资源组模板文件需要你指定映像的发布者、产品和 SKU。

### 视频演练

此视频演示如何使用 PowerShell 完成上述步骤。

[AZURE.VIDEO resource-groups-vm-searching-posh]


<!--Image references-->
[5]: ./media/markdown-template-for-new-articles/octocats.png
[6]: ./media/markdown-template-for-new-articles/pretty49.png
[7]: ./media/markdown-template-for-new-articles/channel-9.png
[8]: ./media/markdown-template-for-new-articles/copytemplate.png

<!--Reference style links - using these makes the source content way more readable than using inline links-->
[gog]: http://google.com/
[yah]: http://search.yahoo.com/
[msn]: http://search.msn.com/

<!---HONumber=69-->