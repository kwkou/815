<properties
	pageTitle="将适用于 Mac、Linux 和 Windows 的 Windows Azure CLI 与 Azure 资源管理配合使用 | Windows Azure"
	description="将适用于 Mac、Linux 和 Windows 的 Windows Azure CLI 与 Azure 资源管理器配合使用。"
	editor="tysonn"
	manager="timlt"
	documentationCenter=""
	authors="dlepow"
	services="virtual-machines"/>

<tags
	ms.service="virtual-machines"
	ms.date="06/09/2015"
	wacn.date="09/18/2015"/>

# 将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure 资源管理器配合使用

> [AZURE.SELECTOR]
- [Azure PowerShell](/documentation/articles/powershell-azure-resource-manager)
- [Azure CLI](/documentation/articles/xplat-cli-azure-resource-manager)


本文介绍如何在 Azure 资源管理器模式下，使用适用于 Mac、Linux 和 Windows 的 Azure CLI 创建、管理和删除 Azure 资源与 VM。

>[AZURE.NOTE]若要使用命令行来创建和管理 Azure 资源，你必须有一个 Azure 帐户（[免费试用](/pricing/1rmb-trial/)）。你还需要[安装 Azure CLI](/documentation/articles/xplat-cli-install)，并[登录以使用与你帐户关联的 Azure 资源](/documentation/articles/xplat-cli-connect)。如果已做好了这些准备，你便可以开始了。

## Azure 资源

使用 Azure 资源管理器可将一组_资源_（用户管理的实体，例如虚拟机、数据库服务器、数据库或网站）作为单个逻辑单位进行管理，或作为_资源组_进行管理。你可以在命令行上强制创建、管理和删除这些资源，就如同在 asm 模式下一样。

使用 Azure 资源管理器模式，还可以通过描述可在 JSON *模板*中部署的资源组的结构与关系，以_声明性_方式管理 Azure 资源。模板描述可以在运行命令时填充任一内联的参数，或存储在单独 JSON azuredeploy-parameters.json 文件中的参数。这样您只需提供不同参数，即可使用同一模板轻松地创建新资源。例如，创建网站的模板将包含站点名称参数、网站所在区域的参数以及其他常见参数。

使用模板修改或创建组时，将创建_部署_，随后将其应用于该组。有关 Azure 资源管理器的详细信息，请参阅 [Azure 资源管理器概述](/documentation/articles/resource-group-overview)。

## 身份验证

若要通过 Azure CLI 使用 Azure 资源管理器，你需要使用公司或学校的帐户在 Microsoft Azure 上进行身份验证。无法使用通过发布设置文件安装的证书进行身份验证。

有关使用公司或学校帐户进行身份验证的详细信息，请参阅[从 Azure CLI 连接到 Azure 订阅](/documentation/articles/xplat-cli-connect)。

> [AZURE.NOTE]你使用的公司或学校帐户由 Azure Active Directory 管理，因此你也可以使用 Azure 基于角色的访问控制 (RBAC) 来管理 Azure 资源的访问与使用。有关详细信息，请参阅<!--[-->管理和审核对资源的访问权限<!--](/documentation/articles/resource-group-rbac)-->。

## 设置 Azure 资源管理器模式

因为默认情况下未启用 Azure 资源管理器模式，因此你必须使用以下命令来启用 Azure CLI 资源管理器命令。

	azure config mode arm

>[AZURE.NOTE]Azure 资源管理器模式与 Azure 服务管理模式互斥。即在一种模式下创建的资源不能从另一种模式进行管理。

## 查找位置

大多数 Azure 资源管理器命令需要有可用于创建或查找资源的有效位置。你可以使用以下命令来查找所有可用的位置。

	azure location list

该命令将列出区域特定的位置，例如“中国北部”、“中国东部”等。

## 创建资源组

资源组是网络、存储和其他资源的逻辑组。Azure 资源管理器模式中的几乎所有命令都需要资源组。例如，你可以使用以下命令创建名为 _testrg_ 的资源组。

	azure group create -n "testrg" -l "China North"

在此之后，你可以开始将资源添加到此组中，然后使用此组来配置新的虚拟机。

## 创建虚拟机

可通过两种方法在 Azure 资源管理器模式中创建虚拟机：

1. 使用单个 Azure CLI 命令。
2. 使用资源组模板

在开始使用任何一种方法之前，请务必先创建至少一个资源组。

### 使用单个 Azure CLI 命令

这是根据你的需求配置和创建虚拟机的基本方法。在 Azure 资源管理器模式中，需要先设置某些必需的资源（例如网络），然后才能使用 **vm create** 命令。

>[AZURE.NOTE]如果你是第一次在命令行上为订阅创建资源，系统可能会提示你注册特定的资源提供程序。如果出现这种情况，你可以轻松地注册提供程序，并再次尝试失败的命令，如以下示例中所示。
>
> `azure provider register Microsoft.Storage`
>
> 你可以通过执行以下命令，找出为订阅注册的提供程序列表。
>
> `azure provider list`


#### 创建公共 IP 资源

你必须先创建公共 IP，才能对新的虚拟机进行 SSH 访问，以进行任何有意义的工作。创建公共 IP 十分简单。命令需要资源组、公共 IP 资源的名称和位置（遵循此顺序）。

	azure network public-ip create "testrg" "testip" "chinanorth"

#### 创建网络接口卡资源

需要先为网络接口卡 (NIC) 创建子网和虚拟网络。使用 **network vnet create** 命令在特定的位置和资源组中创建虚拟网络。

	azure network vnet create "testrg" "testvnet" "chinanorth"

然后，可以使用 **network vnet subnet create** 命令在此虚拟网络中创建子网。

	azure network vnet subnet create "testrg" "testvnet" "testsubnet"

你应该能够通过 **network nic create** 命令使用这些资源创建 NIC。

	azure network nic create "testrg" "testnic" "chinanorth" -k "testsubnet" -m "testvnet" -p "testip"

>[AZURE.NOTE]可选但同样很重要的步骤是，将公共 IP 名称作为参数传递给 **network nic create** 命令，因为这可以将 NIC 绑定到此 IP，而稍后将要通过这种绑定对使用此 NIC 创建的虚拟机进行 SSH 访问。

有关 **network** 命令的详细信息，请参阅命令行帮助或[将 Azure CLI 与 Azure 资源管理配合使用](/documentation/articles/azure-cli-arm-commands)。

#### 查找操作系统映像

目前，你只能根据映像的发布者查找操作系统。换而言之，你必须运行以下命令才能在所需的位置中查找操作系统映像发布者列表。

	azure vm image list-publishers "chinanorth"

从列表中选择发布者，然后通过运行以下命令来按发布者查找映像列表。

	azure vm image list "chinanorth" "CoreOS"

最后，从列表中选择如以下示例中所示的操作系统映像。

	info:    Executing command **vm image list**
	warn:    The parameters --offer and --sku if specified will be ignored
	+ Getting virtual machine image offers (Publisher: "Canonical" Location: "chinanorth")
	data:    Publisher  Offer        Sku          Version          Location  Urn

	data:    ---------  -----------  -----------  ---------------  --------- ----------------------------------------
	data:    CoreOS     CoreOS       Alpha        475.1.0          chinanorth    CoreOS:CoreOS:Alpha:475.1.0
	data:    CoreOS     CoreOS       Alpha        490.0.0          chinanorth    CoreOS:CoreOS:Alpha:490.0.0

保存你要在虚拟机上加载的映像的 URN 名称。本文稍后将会用到此信息。

#### 创建虚拟机

现在，你可以通过运行 **vm create** 命令并传递所需的信息来创建虚拟机。在此阶段，传递公共 IP 的操作是可选的，因为 NIC 已有此信息。你的命令可能如以下示例中所示，其中 _testvm_ 是在 _testrg_ 资源组中创建的虚拟机的名称。

	azure-cli@0.8.0:/# azure vm create "testrg" "testvm" "chinanorth" "Linux" -Q "CoreOS:CoreOS:Alpha:660.0.0" -u "azureuser" -p "Pass1234!" -N "testnic"
	info:    Executing command vm create
	+ Looking up the VM "testvm"
	info:    Using the VM Size "Standard_A1"
	info:    The [OS, Data] Disk or image configuration requires storage account
	+ Retrieving storage accounts
	info:    Could not find any storage accounts in the region "chinanorth", trying to create new one
	+ Creating storage account "cli02f696bbfda6d83414300" in "chinanorth"
	+ Looking up the storage account cli02f696bbfda6d83414300
	+ Looking up the NIC "testnic"
	+ Creating VM "testvm"
	info:    vm create command OK

你应该能通过运行以下命令来启动此虚拟机。

	azure vm start "testrg" "testvm"

然后，使用 **ssh username@ipaddress** 命令对它进行 SSH 访问。若要快速查找公共 IP 资源的 IP 地址，请使用以下命令。

	azure network public-ip show "testrg" "testip"

使用 **vm** 命令可以轻松管理此虚拟机。有关详细信息，请访问[将 Azure CLI 与 Azure 资源管理配合使用](/documentation/articles/azure-cli-arm-commands)。

### vm quick-create 快捷方式

新的 **vm quick-create** 快捷方式能够减少强制创建 VM 的大多数步骤。当你想要尝试创建简单的虚拟机，或者你不注重网络配置，此快捷方式将十分方便。它是交互式命令，你只需先找出操作系统映像 URN，然后再运行它。

	azure-cli@0.8.0:/# azure vm quick-create
	info:  Executing command vm quick-create
	Resource group name: CLIRG
	Virtual machine name: myqvm
	Location name: chinanorth
	Operating system Type [Windows, Linux]: /documentation/articles/Linux
	ImageURN (format: "publisherName:offer:skus:version"): CoreOS:CoreOS:Alpha:660.0.0
	User name: azureuser
	Password: ********
	Confirm password: ********

Azure CLI 将使用默认的 VM 大小创建虚拟机。它还将创建存储帐户、NIC、虚拟网络和子网，以及公共 IP。启动后，你可以使用公共 IP 对虚拟机进行 SSH 访问。

### 使用资源组模板

#### 查找和配置资源组模板

1. 使用模板时，你可以创建自己的模板、使用模板库中的模板，或是使用 [GitHub](https://github.com/azurermtemplates/azurermtemplates) 中的模板。首先，让我们使用资源库中名为 CoreOS.CoreOSStable.0.2.40-preview 的模板。要列出库中可用的模板，请使用以下命令。由于有数千个可用的模板，你可以将结果分页，或者使用 **grep** 或 **findstr**（在 Windows 上），或你偏好的字符串搜索命令来查找想要使用的模板。或者，你可以使用 **--json** 选项，并下载 JSON 格式的完整列表以方便搜索。

		azure group template list

	响应将列出发布者和模板名称，并显示类似于以下示例的结果（不过这只是一小部分内容）。

		data:    Publisher               Name
		data:    ----------------------------------------------------------------------------
		data:    CoreOS                  CoreOS.CoreOSStable.0.2.40-preview
		data:    CoreOS                  CoreOS.CoreOSAlpha.0.2.39-preview
		data:    SUSE                    SUSE.SUSELinuxEnterpriseServer12.2.0.36-preview
		data:    SUSE                    SUSE.SUSELinuxEnterpriseServer11SP3PremiumImage0.2.54-preview

2. 若要查看模板的详细信息，请使用以下命令。

		azure group template show CoreOS.CoreOSStable.0.2.40-preview

	这将返回有关模板的描述性信息。我们使用的模板将创建 Linux 虚拟机。

3. 选择了一个模板之后，您可以使用以下命令来下载该模板。

		azure group template download CoreOS.CoreOSStable.0.2.40-preview

	下载模板后，您可以对其进行自定义，以便更好地满足您的要求。例如，向模板中添加其他资源。

	>[AZURE.NOTE]如果修改了模板，请在使用该模板创建或修改现有资源组之前，使用 `azure group template validate` 命令来验证该模板。

4. 若要配置你要使用的资源组模板，请在文本编辑器中打开模板文件。请注意靠近顶部的 **parameters** JSON 集合。该集合包含此模板要用来创建模板所述资源的参数列表。部分参数可能具有默认值，而其他参数只指定了某个值或一系列允许值的类型。

	使用模板时，您可以提供参数使之作为命令行参数的一部分，或通过指定包含参数值的文件来提供参数。你还可以直接在模板的 **parameters** 节中编写 **value** 字段，不过，这会使模板紧密绑定到特定的部署，因而无法轻松地重复使用。无论如何，参数都必须采用 JSON 格式，并且你必须为没有默认值的键提供你自己的值。

	例如，若要创建一个包含 CoreOS.CoreOSStable.0.2.40-preview 模板参数的文件，请使用以下数据创建名为 params.json 的文件。将此示例中使用的值替换为你自己的值。**Location** 应指定你附近的 Azure 区域，如“欧洲北部”或“美国中南部”。（此示例使用“中国北部”）。

		{
		  "newStorageAccountName": {
			"value": "testStorage"
		  },
		  "newDomainName": {
			"value": "testDomain"
		  },
		  "newVirtualNetworkName": {
			"value": "testVNet"
		  },
		  "vnetAddressSpace": {
			"value": "10.0.0.0/11"
		  },
		  "hostName": {
			"value": "testHost"
		  },
		  "userName": {
			"value": "azureUser"
		  },
		  "password": {
			"value": "Pass1234!"
		  },
		  "location": {
			"value": "China North"
		  },
		  "hardwareSize": {
			"value": "Medium"
		  }
	    }

5. 保存 params.json 文件后，请使用以下命令基于模板创建新的资源组。`-e` 参数指定上一步中创建的 params.json 文件。将 **testRG** 替换为你要使用的组名称，将 **testDeploy** 替换为部署名称。位置应与你在 params.json 模板参数文件中指定的位置相同。

		azure group create "testRG" "China North" -f CoreOS.CoreOSStable.0.2.40-preview.json -d "testDeploy" -e params.json

	在上载部署之后、将部署应用于组中的资源之前，此命令将返回 OK。要检查部署的状态，请使用以下命令。

		azure group deployment show "testRG" "testDeploy"

	**ProvisioningState** 将显示部署的状态。

	如果部署成功，你将看到与以下示例类似的输出。

		azure-cli@0.8.0:/# azure group deployment show testRG testDeploy
		info:    Executing command group deployment show
		+ Getting deployments
		data:    DeploymentName     : testDeploy
		data:    ResourceGroupName  : testRG
		data:    ProvisioningState  : Running
		data:    Timestamp          : 2015-04-27T07:49:18.5237635Z
		data:    Mode               : Incremental
		data:    Name                   Type          Value
		data:    ---------------------  ------------  ----------------
		data:    newStorageAccountName  String        testStorage
		data:    newDomainName          String        testDomain
		data:    newVirtualNetworkName  String        testVNet
		data:    vnetAddressSpace       String        10.0.0.0/11
		data:    hostName               String        testHost
		data:    userName               String        azureUser
		data:    password               SecureString  undefined
		data:    location               String        China North
		data:    hardwareSize           String        Medium
		info:    group deployment show command OK

	>[AZURE.NOTE]如果您发现配置不正确，且需要停止长时间运行的部署，请使用以下命令。
	>
	> `azure group deployment stop "testRG" "testDeploy"`
	>
	> 如果您未提供部署名称，系统将根据模板文件的名称自动创建一个。返回的名称将作为 `azure group create` 命令输出的一部分。

6. 要查看组，请使用以下命令：

		azure group show "testRG"

	此命令将返回组中资源的相关信息。如果有多个组，可以使用 `azure group list` 命令来检索组名称的列表，然后使用 `azure group show` 来查看特定组的详细信息。

7. 你还可以直接使用 GitHub 中的最新模板，而无需从模板库下载模板。打开 [GitHub.com](http://www.github.com) 并搜索 AzureRmTemplates。选择 AzureRmTemplates 存储库，然后查找任何你要用的模板，例如 _101-simple-vm-from-image_。如果你单击该模板，将会看到它包含 azuredeploy.json 和其他文件。这就是你要使用 **--template-url** 选项在命令中使用的模板。以 _raw_ 模式打开模板，然后复制浏览器地址栏中的 URL。接下来，你可以使用类似于以下示例的命令，直接使用此 URL 创建部署，而无需从资源库下载。

		azure group deployment create "testDeploy" -g "testResourceGroup" --template-uri https://raw/githubusercontent.com/azurermtemplates/azurermtemplates/master/101-simple-vm-from-image/azuredeploy.json

	> [AZURE.NOTE]请务必以 _raw_ 模式打开 json 模板。浏览器地址栏中显示的 URL 与常规模式下显示的 URL 不同。在 GitHub 上查看文件时，若要以 _raw_ 模式打开文件，请单击右上角的“原始”。

#### 使用资源

尽管模板可用于声明组范围内的配置更改，但有时您只需要使用特定资源。你可以使用 `azure resource` 命令来实现此目的。

> [AZURE.NOTE]使用除 `list` 命令以外的 `azure resource` 命令时，你必须使用 `-o` 参数指定你使用的资源的 API 版本。如果你不确定要使用的 API 版本，请查阅模板文件并查找资源的 **apiVersion** 字段。

1. 要列出组中的所有资源，请使用以下命令。

		azure resource list "testRG"

2. 若要查看组中的单个资源，请使用以下命令。

		azure resource show "testRG" "testHost" Microsoft.ClassicCompute/virtualMachines -o "2014-06-01"

	注意 **Microsoft.ClassicCompute/virtualMachines** 参数。其表示您正在请求信息的资源的类型。如果你看一下之前下载的模板文件，你会注意到此相同的值用于定义模板中所述的虚拟机资源的类型。

	此命令将返回与虚拟机相关的信息。

3. 查看资源详细信息时，使用 `--json` 参数很有用，因为由于某些值为嵌套结构或集合，使用此参数可使输出更易于阅读。以下示例演示了将 **show** 命令的结果返回为 JSON 文档。

		azure resource show "testRG" "testHost" Microsoft.ClassicCompute/virtualMachines -o "2014-06-01" --json

	>[AZURE.NOTE]你可以通过使用 &gt; 字符将输出传输到文件，将 JSON 数据保存到文件。例如：
	>
	> `azure resource show "testRG" "testHost" Microsoft.ClassicCompute/virtualMachines -o "2014-06-01" --json > myfile.json`

4. 要删除现有资源，请使用以下命令。

		azure resource delete "testRG" "testHost" Microsoft.ClassicCompute/virtualMachines -o "2014-06-01"

## 日志记录

要查看对组执行的操作的日志记录信息，请使用 `azure group log show` 命令。默认情况下，此命令将列出对组执行的最后一个操作。要查看所有操作，请使用可选的 `--all` 参数。对于最后一个部署，请使用 `--last-deployment`。对于特定部署，请使用 `--deployment` 并指定部署名称。以下示例将返回对组 MyGroup 执行的所有操作的日志。

	azure group log show mygroup --all

## 后续步骤

* 有关使用 Azure 命令行界面 (Azure CLI) 的信息，请参阅[安装和配置 Azure CLI][clisetup]。
* 有关将 Azure 资源管理器与 Azure PowerShell 配合使用的信息，请参阅[将 Azure PowerShell 与 Azure 资源管理器配合使用](/documentation/articles/powershell-azure-resource-manager)。
* 有关从 Azure 门户使用 Azure 资源管理器的信息，请参阅[使用资源组管理 Azure 资源][psrm]。

[signuporg]: http://www.windowsazure.com/documentation/articles/sign-up-organization/
[adtenant]: http://technet.microsoft.com/zh-cn/library/jj573650#createAzureTenant
[portal]: https://manage.windowsazure.com/
[clisetup]: /documentation/articles/xplat-cli
[psrm]: http://go.microsoft.com/fwlink/?LinkId=394760

<!---HONumber=70-->