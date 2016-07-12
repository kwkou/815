<properties
   pageTitle="Azure 自动化中常见错误的疑难解答提示 | Azure"
   description="本文提供的基本疑难解答步骤适用于解决你在使用 Azure 自动化时会碰到的常见错误。"
   services="automation"
   documentationCenter=""
   authors="mgoedtel"
   manager="stevenka"
   editor="tysonn"
   tags="top-support-issue"/>
<tags
   ms.service="automation"
   ms.date="03/21/2016"
   wacn.date="06/30/2016"/>

# Azure 自动化中常见错误的疑难解答提示

本文介绍了你在使用 Azure 自动化时可能会遇到的部分常见错误，并提供了可能的补救步骤建议。

## 解决使用 Azure 自动化 Runbook 时遇到的身份验证错误  

### 场景：登录 Azure 帐户失败

**错误：** 
你在使用 Add-AzureAccount -Environment AzureChinaCloud 或 Login-AzureRmAccount -EnvironmentName AzureChinaCloud 命令时收到“Unknown\_user\_type: 用户类型未知”错误。

**错误原因：**
如果凭据资产名称无效或者用于设置自动化凭据资产的用户名和密码无效，则会出现此错误。

**疑难解答提示：**
为了确定具体错误，请执行以下步骤：

1. 确保在用于连接到 Azure 的自动化凭据资产名称中没有任何特殊字符，包括 **@** 字符。  

2. 查看你是否能够在本地 PowerShell ISE 编辑器中使用存储在 Azure 自动化凭据中的用户名和密码。为此，你可以在 PowerShell ISE 中运行以下 cmdlet：

        $Cred = Get-Credential  
        #Using Azure Service Management   
        Add-AzureAccount -Environment AzureChinaCloud –Credential $Cred  
        #Using Azure Resource Manager  
        Login-AzureRmAccount -EnvironmentName AzureChinaCloud –Credential $Cred

3. 如果无法在本地进行身份验证，则意味着你尚未设置好 Azure Active Directory 凭据。请参阅[使用 Azure Active Directory 向 Azure 进行身份验证](https://azure.microsoft.com/blog/azure-automation-authenticating-to-azure-using-azure-active-directory)博客文章，了解如何正确设置 Azure Active Directory 帐户。


### 场景：找不到 Azure 订阅

**错误：**
使用 Select-AzureSubscription 或 Select-AzureRmSubscription cmdlet 时，你收到“找不到名为 `<subscription name>` 的订阅”错误。

**错误原因：**
如果订阅名称无效或者未将正尝试获取订阅详细信息的 Azure Active Directory 用户配置为订阅的管理员，则会出现此错误。

**疑难解答提示：**
若要确定你是否已向 Azure 进行过适当的身份验证并可访问所要选择的订阅，请执行以下步骤：

1. 确保先运行 **Add-AzureAccount -Environment AzureChinaCloud** cmdlet，然后运行 **Select-AzureSubscription** cmdlet。  

2. 如果仍显示此错误消息，可通过添加 **Get-AzureSubscription** cmdlet（此前已添加 **Add-AzureAccount -Environment AzureChinaCloud** cmdlet）来修改代码，然后执行代码。现在，请验证 Get-AzureSubscription 的输出是否包含你的订阅详细信息。
    * 如果你在输出中看不到任何订阅详细信息，则说明该订阅尚未初始化。  
    * 如果你在输出中看到了订阅详细信息，请确认你对 **Select-AzureSubscription** cmdlet 使用了正确的订阅名称或 ID。   


### 场景：无法向 Azure 进行身份验证，因为已启用多重身份验证

**错误：**使用 Azure 用户名和密码向 Azure 进行身份验证时，你收到“Add-AzureAccount: AADSTS50079: 需要进行强身份验证注册(验证)”错误。

**错误原因：**如果你对 Azure 帐户设置了多重身份验证，则不能使用 Azure Active Directory 用户向 Azure 进行身份验证，而只能使用证书或服务主体向 Azure 进行身份验证。

**疑难解答提示：**若要将证书用于 Azure 服务管理 cmdlet，请参阅[创建并添加管理 Azure 服务所需的证书](http://blogs.technet.com/b/orchestrator/archive/2014/04/11/managing-azure-services-with-the-microsoft-azure-automation-preview-service.aspx)。 若要将服务主体用于 Azure 资源管理器 cmdlet，请参阅[使用 Azure 经典管理门户创建服务主体](/documentation/articles/resource-group-create-service-principal-portal/)和[通过 Azure 资源管理器 对服务主体进行身份验证](/documentation/articles/resource-group-authenticate-service-principal/)。


## 解决使用 Runbook 时的常见错误
### 场景：Runbook 因反序列化的对象而失败

**错误：**Runbook 失败，出现错误“无法绑定参数 `<ParameterName>`。无法将反序列化 `<ParameterType>` 类型的 `<ParameterType>` 值转换成 `<ParameterType>` 类型”。

**错误原因：**如果你的 Runbook 为 PowerShell 工作流，则会将复杂对象以反序列化格式进行存储，以便在工作流暂停的情况下保留 Runbook 状态。

**疑难解答提示：**下述三种解决方案中的任何一种都可以解决此问题：

1. 如果你要将复杂对象从一个 cmdlet 传送到另一个 cmdlet，则可将这两个 cmdlet 包装在 InlineScript 中。  
2. 传递复杂对象中你所需要的名称或值，不必传递整个对象。  


### 场景：Runbook 作业失败，因为超过了分配的配额

**错误：**
Runbook 作业失败，出现“已达到此订阅的每月总作业运行时间配额”错误。

**错误原因：**
当作业执行时间超过你帐户的 500 分钟免费配额时，就会出现此错误。此配额适用于所有类型的作业执行任务，例如测试作业、从经典管理门户启动作业、使用 Webhook 执行作业，以及通过 Azure 经典管理门户或数据中心计划要执行的作业。若要详细了解自动化的定价，请参阅 [Automation pricing（自动化定价）](/home/features/automation/pricing/)。

**疑难解答提示：**
如果你想要每月使用 500 分钟以上的处理时间，则需将订阅从免费层改为基本层。你可以通过下述步骤升级到基本层：

1. 登录到 Azure 订阅  
2. 选择要升级的自动化帐户  
3. 单击“缩放”
4. 在“常规”中的“自动化计划”，选择“基本”    


### 场景：在执行 Runbook 时无法识别 Cmdlet

**错误：**
Runbook 作业失败，出现“`<cmdlet name>`: 无法将术语 `<cmdlet name>` 视为 cmdlet 名称、函数、脚本文件或可运行程序”错误。

**错误原因：**
当 PowerShell 引擎找不到你要在 Runbook 中使用的 cmdlet 时，则会导致此错误。这可能是因为，帐户中缺少包含该 cmdlet 的模块、与 Runbook 名称存在名称冲突，或者该 cmdlet 也存在于其他模块中，而自动化无法解析该名称。

**疑难解答提示：**
下述解决方案中的任何一种都可以解决此问题：

- 检查输入的 cmdlet 名称是否正确。  

- 确保 cmdlet 存在于你的自动化帐户中，且没有冲突。若要验证 cmdlet 是否存在，请在编辑模式下打开 Runbook，然后搜索你希望在库中找到的 cmdlet，或者运行 **`Get-Command <CommandName>`**。验证该 cmdlet 可供帐户使用且与其他 cmdlet 或 Runbook 不存在名称冲突以后，可将其添加到画布上，并确保你使用的是 Runbook 中的有效参数集。

- 如果存在名称冲突且 cmdlet 可在两个不同的模块中使用，则可使用 cmdlet 的完全限定名称来解决此问题。例如，你可以使用 **ModuleName\\CmdletName**。

- 如果你是在本地执行混合辅助角色组中的 Runbook，则请确保模块/cmdlet 已安装在托管混合辅助角色的计算机上。


### 场景：某个长时间运行的 Runbook 不断失败并出现异常：“该作业无法继续运行，因为它已反复从同一个检查点逐出”。

**错误原因：**
这是设计使然。Azure 自动化中对进程的“公平份额”监视会自动暂停执行时间超过 3 小时的 Runbook。但是，返回的错误消息不会提供“后续措施”选项。Runbook 可能会出于多种原因而暂停。发生暂停的主要原因是出错。例如，Runbook 中出现未捕获到的异常、网络故障、运行 Runbook 的 Runbook 辅助角色崩溃，都会导致 Runbook 暂停，并在恢复时从其最后一个检查点开始运行。

**疑难解答提示：**
避免此错误的有记录解决方法是在工作流中使用检查点。若要了解详细信息，请参阅 [Learning PowerShell Workflows](/documentation/articles/automation-powershell-workflow/#Checkpoints)（学习 PowerShell 工作流）。[Using Checkpoints in Runbooks](https://azure.microsoft.com/zh-cn/blog/azure-automation-reliable-fault-tolerant-runbook-execution-using-checkpoints)（在 Runbook 中使用检查点）博客文章中提供了有关“公平份额”和检查点的更全面说明。


## 解决导入模块时的常见错误

### 场景：模块无法导入，或者 cmdlet 在导入后无法执行

**错误：**
模块无法导入，或者虽然成功导入，但无法提取 cmdlet。

**错误原因：**
模块无法成功导入到 Azure 自动化中的部分常见原因是：

- 结构与自动化所需的模块结构不符。  

- 该模块依赖于其他模块，而后者尚未部署到你的自动化帐户。

- 该模块的文件夹中缺少依赖项。

- 使用了 **New-AzureAutomationModule** cmdlet 来上载该模块，但你尚未提供完整的存储路径，或者尚未使用可公开访问的 URL 来加载该模块。

**疑难解答提示：**
下述解决方案中的任何一种都可以解决此问题：

- 确保模块遵循以下格式：ModuleName.Zip **->** ModuleName 或版本号 **->** (ModuleName.psm1, ModuleName.psd1)

- 打开 .psd1 文件，看模块是否有任何依赖项。如果有，则将这些模块上载到自动化帐户。

- 确保任何引用的 .dll 都存在于模块文件夹中。

  <br/>

## 后续步骤

如果你在完成上述疑难解答步骤以后仍对本文中的内容存有疑问，你可以：

- 从 Azure 专家那里获取帮助。向 [MSDN Azure 或 CSDN](/support/forums)提交问题。

- 提出 Azure 支持事件。转到“[Azure 支持站点](/support/contact)”，单击“技术和帐单支持”下的“获得支持”。

- 将关于 Azure 自动化的反馈或功能请求发布在[用户之声](/product-feedback)。

<!---HONumber=Mooncake_0411_2016-->
