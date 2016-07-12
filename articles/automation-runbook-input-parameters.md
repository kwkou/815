<properties
   pageTitle="Runbook 输入参数 | Azure"
   description="Runbook 输入参数可让你将数据传递到启动的 Runbook，以增加 Runbook 的弹性。本文介绍在 Runbook 中使用输入参数的不同方案。"
   services="automation"
   documentationCenter=""
   authors="SnehaGunda"
   manager="stevenka"
   editor="tysonn" />
<tags
   ms.service="automation"
   ms.date="04/25/2016"
   wacn.date="06/30/2016"/>

# Runbook 输入参数

Runbook 输入参数可让你将数据传递到启动的 Runbook，以增加 Runbook 的弹性。这些参数可让 Runbook 操作以特定方案和环境为目标。在本文中，我们将逐步介绍在 Runbook 中使用输入参数的不同方案。

## 配置输入参数

可以在 PowerShell 工作流中配置输入参数。一个 Runbook 可以包含具有不同数据类型的多个参数，或者不包含任何参数。输入参数可以是必需的或可选的，你可以为可选参数分配默认值。你可以在通过某种可用方法启动 Runbook 时分配 Runbook 的输入参数值。这些方法包括使用 UI 或 Web 服务启动 Runbook。还可以启动一个 Runbook 作为另一个 Runbook 中内联调用的子 Runbook。

## 在 PowerShell 工作流 Runbook 中配置输入参数

Azure 自动化中的 [PowerShell 工作流 Runbook](/documentation/articles/automation-first-runbook-textual/) 支持通过以下属性定义的输入参数。

| **属性** | **说明** |
|:--- |:---|
| 类型 | 必需。参数值所需的数据类型。任何 .NET 类型均有效。 |
| Name | 必需。参数的名称。在 Runbook 中必须唯一，并且只能包含字母、数字或下划线字符。必须以字母开头。 |
| 必需 | 可选。指定是否必须为该参数提供值。如果将此项设置为 **true**，则在启动 Runbook 时必须提供一个值。如果将此项设置为 **false**，则值是可选的。 |
| 默认值 | 可选。指定在 Runbook 启动时未传递值的情况下要用于参数的值。可为任何参数设置默认值，此值将使参数自动成为可选，而不管 Mandatory 设置为何。 |

Windows PowerShell 支持的输入参数属性比此处所列的多，例如验证、别名和参数集。但是，Azure 自动化目前仅支持上面所列的输入参数。

PowerShell 工作流 Runbook 中的参数定义采用以下常规格式，其中，多个参数必须以逗号分隔。


     Param
     (
         [Parameter (Mandatory= $true/$false)]
         [Type] Name1 = <Default value>,

         [Parameter (Mandatory= $true/$false)]
         [Type] Name2 = <Default value>
     )


>[AZURE.NOTE] 定义参数时，如果未指定 **Mandatory** 属性，则会按默认将参数视为可选。此外，如果在 PowerShell 工作流 Runbook 中设置某个参数的默认值，则 PowerShell 会将其视为可选参数，而不管 **Mandatory** 属性值为何。

例如，让我们为输出有关虚拟机（可以是单个 VM 或服务中的所有 VM）的详细信息的 PowerShell 工作流 Runbook 配置输入参数。如以下屏幕截图中所示，此 Runbook 有两个参数：虚拟机的名称和服务的名称。


在此参数定义中，**VMName** 和 **ServiceName** 参数是字符串类型的简单参数。但是，PowerShell 和 PowerShell 工作流 Runbook 支持所有简单类型和复杂类型，例如输入参数的 **object** 或 **PSCredential**。

如果 Runbook 有 [object] 类型输入参数，请使用包含 (name,value) 对的 PowerShell 哈希表来传入值。例如，如果 Runbook 中有以下参数：

     [Parameter (Mandatory = $true)]
     [object] $FullName

则可将以下值传递到该参数：

    @{"FirstName"="Joe";"MiddleName"="Bob";"LastName"="Smith"}

## 为 Runbook 中的输入参数赋值

在以下情况下，可以将值传递到 Runbook 中的输入参数。

### 启动 Runbook 并分配参数

Runbook 有多种启动方式：通过 Azure 经典管理门户 UI、PowerShell cmdlet、REST API 或 SDK。下面介绍了启动 Runbook 和分配参数的不同方法。

- **使用 Azure 经典管理门户启动已发布的 Runbook 并分配参数**

当你[启动 Runbook](/documentation/articles/automation-starting-a-runbook/#starting-a-runbook-with-the-azure-portal) 时，“启动 Runbook”边栏选项卡将会打开，你可以为刚刚创建的参数配置值。

 
在输入框下面的标签中，可以查看为参数设置的属性。属性包括必需或可选状态、类型和默认值。在参数名称旁边的帮助气球中，可以查看做出参数输入值相关决策时所需的所有关键信息。此信息包括参数是必需还是可选的。此外还包括类型和默认值（如果有）及其他有用的说明。

 

>[AZURE.NOTE] 字符串类型参数支持**空**字符串值。在输入参数框中输入 **[EmptyString]** 将向参数传递空字符串。同时，字符串类型参数不支持传递 **Null** 值。如果未向字符串参数传递任何值，PowerShell 会将值解释为 Null。

- **使用 PowerShell cmdlet 启动已发布的 Runbook 并分配参数**

    - **Azure 服务管理 cmdlet：**可以使用 [Start-AzureAutomationRunbook](https://msdn.microsoft.com/zh-cn/library/dn690259.aspx) 启动在默认资源组中创建的自动化 Runbook。

    **示例：**

        $params = @{“VMName”=”WSVMClassic”; ”ServiceName”=”WSVMClassicSG”}

        Start-AzureAutomationRunbook -AutomationAccountName “TestAutomation” -Name “Get-AzureVMGraphical” -Parameters $params


    - **Azure 资源管理器 cmdlet：**可以使用 [Start-AzureRMAutomationRunbook](https://msdn.microsoft.com/zh-cn/library/mt603661.aspx) 启动在资源组中创建的自动化 Runbook。


    **示例：**

        $params = @{“VMName”=”WSVMClassic”;”ServiceName”=”WSVMClassicSG”}

        Start-AzureRMAutomationRunbook -AutomationAccountName “TestAutomationRG” -Name “Get-AzureVMGraphical” –ResourceGroupName “RG1” -Parameters $params


>[AZURE.NOTE] 使用 PowerShell cmdlet 启动 Runbook 时，将创建值为 **PowerShell** 的默认参数 **MicrosoftApplicationManagementStartedBy**。可以在“作业详细信息”边栏选项卡中查看此参数。

- **使用 SDK 启动 Runbook 并分配参数**

    - **Azure 服务管理方法：**可以使用编程语言 SDK 来启动 Runbook。以下 C# 代码段用于在自动化帐户中启动 Runbook。可以在 [GitHub 存储库](https://github.com/Azure/azure-sdk-for-net/blob/master/src/ServiceManagement/Automation/Automation.Tests/TestSupport/AutomationTestBase.cs)中查看完整代码。  
   
	        public Job StartRunbook(string runbookName, IDictionary<string, string> parameters = null)
	        {
	            var response = AutomationClient.Jobs.Create(automationAccount, new JobCreateParameters
	            {
	                Properties = new JobCreateProperties
	                {
	                    Runbook = new RunbookAssociationProperty
	                    {
	                        Name = runbookName
	                    },
	                        Parameters = parameters
	                }
	            });
	            return response.Job;
	        }

    - **Azure 资源管理器方法：**可以使用编程 SDK 来启动 Runbook。以下 C# 代码段用于在自动化帐户中启动 Runbook。可以在 [GitHub 存储库](https://github.com/Azure/azure-sdk-for-net/blob/master/src/ResourceManagement/Automation/Automation.Tests/TestSupport/AutomationTestBase.cs)中查看完整代码。

	        public Job StartRunbook(string runbookName, IDictionary<string, string> parameters = null)
	        {
	           var response = AutomationClient.Jobs.Create(resourceGroup, automationAccount, new JobCreateParameters
	           {
	               Properties = new JobCreateProperties
	               {
	                   Runbook = new RunbookAssociationProperty
	                   {
	                       Name = runbookName
	                   },
	                       Parameters = parameters
	               }
	           });
	        return response.Job;
	        }

若要启动此方法，请创建一个字典来存储 Runbook 参数、**VMName** 和 **ServiceName** 及其值。然后启动 Runbook。以下 C# 代码段用于调用上面定义的方法。


    IDictionary<string, string> RunbookParameters = new Dictionary<string, string>();

    // Add parameters to the dictionary.
    RunbookParameters.Add("VMName", "WSVMClassic");
    RunbookParameters.Add("ServiceName", "WSVMClassicSG");

    //Call the StartRunbook method with parameters
    StartRunbook(“Get-AzureVMGraphical”, RunbookParameters);


- **使用 REST API 启动 Runbook 并分配参数**

可以通过 Azure 自动化 REST API 并配合 **PUT** 方法及以下请求 URI 来创建和启动 Runbook 作业。

    https://management.core.chinacloudapi.cn/<subscription-id>/cloudServices/<cloud-service-name>/resources/automation/~/automationAccounts/<automation-account-name>/jobs/<job-id>?api-version=2014-12-08

在请求 URI 中替换以下参数：

* **subscription-id：**你的 Azure 订阅 ID。  
* **cloud-service-name：**请求所要发送到的云服务的名称。  
* **automation-account-name：**托管在指定云服务中的自动化帐户名称。  
* **job-id：**作业的 GUID。使用 **[GUID]::NewGuid().ToString()** cmdlet 可以创建 PowerShell 中的 GUID。

若要将参数传递到 Runbook 作业，请使用请求正文。它采用两个以 JSON 格式提供的属性：

* **Runbook 名称** -- 必需。作业要启动的 Runbook 的名称。  
* **Runbook 参数** -- 可选。参数列表的字典；列表必须采用 (名称, 值) 格式，其中的名称应为字符串类型，值可以是任何有效的 JSON 值。

如果想要启动先前以 **VMName** 和 **ServiceName** 作为参数创建的 **Get-AzureVMTextual** Runbook，请使用以下 JSON 格式的请求正文。


        {
           "properties":{
           "runbook":{
               "name":"Get-AzureVMTextual"
           },
           "parameters":{
               "VMName":"WSVMClassic",
               "ServiceName":”WSCS1”
           }Add-AzureAccount
          }
       }


如果成功创建了作业，将返回 HTTP 状态代码 201。有关响应标头和响应正文的详细信息，请参阅有关如何[使用 REST API 创建 Runbook 作业](https://msdn.microsoft.com/zh-cn/library/azure/mt163849.aspx)的文章。

### 测试 Runbook 并分配参数

使用测试选项[测试 Runbook 的草稿版本](/documentation/articles/automation-testing-runbook/)时，将打开“测试”边栏选项卡，你可以在其中为刚刚创建的参数设置值。

 
### 将计划链接到 Runbook 并分配参数

可以将[计划链接](/documentation/articles/automation-scheduling-a-runbook/)到 Runbook，以便在特定的时间启动 Runbook。可以在创建计划时分配输入参数，Runbook 在按计划启动时，将使用这些值。只有在提供所有必需参数值之后，才可以保存计划。

 

## 后续步骤

- 有关 Runbook 输入和输出的详细信息，请参阅 [Azure 自动化：Runbook 输入、输出和嵌套 Runbook](https://azure.microsoft.com/blog/azure-automation-runbook-input-output-and-nested-runbooks)。
- 有关以不同方式启动 Runbook 的详细信息，请参阅[启动 Runbook](/documentation/articles/automation-starting-a-runbook/)。
- 若要编辑文本 Runbook，请参阅[编辑文本 Runbook](/documentation/articles/automation-edit-textual-runbook/)。

<!---HONumber=Mooncake_0215_2016-->
