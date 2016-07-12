<properties 
   pageTitle="Azure 自动化中的 Runbook 输出和消息 | Azure"
   description="介绍如何创建和检索 Azure 自动化中 Runbook 的输出和错误消息。"
   services="automation"
   documentationCenter=""
   authors="mgoedtel"
   manager="stevenka"
   editor="tysonn" />
<tags 
   ms.service="automation"
   ms.date="03/03/2016"
   wacn.date="06/30/2016" />

# Azure 自动化中的 Runbook 输出和消息

大多数 Azure 自动化 Runbook 向用户或旨在由其他工作流使用的复杂对象提供某种形式的输出，例如错误消息。Windows PowerShell 提供[多个流](http://blogs.technet.com/heyscriptingguy/archive/2014/03/30/understanding-streams-redirection-and-write-host-in-powershell.aspx)，以便从脚本或工作流发送输出。Azure 自动化将以不同方式处理其中的每个流，当你在创建 Runbook 时，应该遵循有关如何使用每个流的最佳实践。

下表提供了每个流的简要描述，以及当你运行发布的 Runbook 和[测试 Runbook](/documentation/articles/automation-testing-runbook/) 时它们在 Azure 经典管理门户中的行为。后续部分将提供有关每个流的更多详细信息。

| Stream | 说明 | 已发布 | 测试|
|:---|:---|:---|:---|
|输出|对象旨在由其他 Runbook 使用。|写入作业历史记录。|显示在测试输出窗格中。|
|警告|面向用户的警告消息。|写入作业历史记录。|显示在测试输出窗格中。|
|错误|面向用户的错误消息。与发生异常时不同，默认情况下，在出现错误消息后，Runbook 会继续执行。|写入作业历史记录。|显示在测试输出窗格中。|
|详细|提供一般信息或调试信息的消息。|仅当为 Runbook 启用了详细日志记录时，才写入作业历史记录。|仅当在 Runbook 中将 $VerbosePreference 设置为 Continue 时，才显示在“测试输出”窗格中。|
|进度|完成 Runbook 中每个活动之前和之后自动生成的记录。Runbook 不应尝试创建自身的进度记录，因为这些记录面向交互式用户。|仅当为 Runbook 启用了进度日志记录时，才写入作业历史记录。|不显示在测试输出窗格中。|
|调试|面向交互式用户的消息。不应在 Runbook 中使用。|不会写入作业历史记录。|不会写入测试输出窗格。|

##<a id="output-stream" name="Output"></a> 输出流

输出流旨在输出脚本或工作流创建的对象（如果该脚本或工作流正常运行）。在 Azure 自动化中，此流主要用于旨在供[调用当前 Runbook 的父 Runbook](/documentation/articles/automation-child-runbooks/) 使用的对象。当你从父 Runbook [调用某个内嵌 Runbook](/documentation/articles/automation-child-runbooks/#InlineExecution) 时，被调用的 Runbook 会将输出流中的数据返回给父级。仅当你知道该 Runbook 永不被其他 Runbook 调用时，才应使用输出流将一般信息传回给用户。但是，作为最佳实践，你通常应该使用[详细流](#Verbose)向用户传递一般信息。

可以通过使用 [Write-Output](http://technet.microsoft.com/zh-cn/library/hh849921.aspx)，或者在 Runbook 中将对象放置在其对应行中，来向输出流写入数据。

	#The following lines both write an object to the output stream.
	Write-Output –InputObject $object
	$object

### 函数的输出

当你向 Runbook 包含的某个函数中的输出流写入数据时，输出将传回到 Runbook。如果 Runbook 将该输出分配给某个变量，则不会将数据写入到输出流。向函数内部的任何其他流写入数据会向 Runbook 的相应流写入数据。

请考虑以下示例 Runbook。

	Workflow Test-Runbook
	{
	   Write-Verbose "Verbose outside of function"
	   Write-Output "Output outside of function"
	   $functionOutput = Test-Function
	
	   Function Test-Function
	   {
	      Write-Verbose "Verbose inside of function"
	      Write-Output "Output inside of function"
	   }
	}

Runbook 作业的输出流将是：

	Output outside of function

Runbook 作业的详细流将是：

	Verbose outside of function
	Verbose inside of function

### 声明输出数据类型

工作流可以使用 [OutputType 属性](http://technet.microsoft.com/zh-cn/library/hh847785.aspx)指定其输出的数据类型。此属性在运行时不起作用，但在设计时，它可以向 Runbook 作者指明 Runbook 的预期输出。随着 Runbook 工具集的持续发展，在设计时声明输出数据类型的重要性也在不断提升。因此，最好是在你创建的所有 Runbook 中包含此声明。

以下示例 Runbook 将输出一个字符串对象，并包含其输出类型的声明。如果 Runbook 输出了特定类型的数组，则你仍应该指定相对于该类型数组的类型。

	Workflow Test-Runbook
	{
	   [OutputType([string])]
	
	   $output = "This is some string output."
	   Write-Output $output
	}

##<a id="message-streams" name="Message"></a> 消息流

与输出流不同，消息流旨在向用户传递信息。有多个消息流用于传递不同类型的信息，Azure 自动化将以不同的方式处理每个消息流。

###<a id="warning-and-error" name="WarningError"></a> 警告和错误流

警告和错误流旨在记录 Runbook 中出现的问题。在执行 Runbook 时，这些流将写入作业历史记录；在测试 Runbook 时，它们将包含在 Azure 经典管理门户的测试输出窗格中。默认情况下，在出现警告或错误后，Runbook 将继续执行。在创建消息之前，可以通过在 Runbook 中设置 [preference 变量](#PreferenceVariables)，指定 Runbook 应在出现警告或错误时挂起。例如，若要使 Runbook 在出现错误时挂起（就像发生异常时那样），请将 **$ErrorActionPreference** 设置为 Stop。

使用 [Write-Warning](https://technet.microsoft.com/zh-cn/library/hh849931.aspx) 或 [Write-Error](http://technet.microsoft.com/zh-cn/library/hh849962.aspx) cmdlet 创建警告或错误消息。活动可能也会向这些流写入数据。

	#The following lines create a warning message and then an error message that will suspend the runbook.
	
	$ErrorActionPreference = "Stop"
	Write-Warning –Message "This is a warning message."
	Write-Error –Message "This is an error message that will stop the runbook because of the preference variable."

###<a id="verbose-streams" name="Verbose"></a> 详细流

详细消息流用于传递有关 Runbook 操作的一般信息。由于[调试流](#Debug)在 Runbook 中不可用，因此，应该为调试信息使用详细消息。默认情况下，来自已发布 Runbook 的详细消息不会存储在作业历史记录中。若要存储详细消息，请在 Azure 经典管理门户中 Runbook 的“配置”选项卡上，将已发布 Runbook 配置为“记录详细记录”。在大多数情况下，出于性能方面的原因，你应该保留默认设置，即，不记录详细记录。启用此选项的目的只是为了排查 Runbook 的问题或对它进行调试。

在[测试 Runbook](/documentation/articles/automation-testing-runbook/) 时，将不会显示详细消息，即使已将该 Runbook 配置为记录详细记录，也是如此。若要在[测试 Runbook](/documentation/articles/automation-testing-runbook/) 时显示详细消息，必须将 $VerbosePreference 变量设置为 Continue。设置该变量后，Azure 经典管理门户的测试输出窗格中会显示详细消息。

使用 [Write-Verbose](http://technet.microsoft.com/zh-cn/library/hh849951.aspx) cmdlet 创建详细消息。

	#The following line creates a verbose message.
	
	Write-Verbose –Message "This is a verbose message."

###<a id="debug-streams" name="Debug"></a> 调试流

调试流旨在供交互式用户使用，不应在 Runbook 中使用。

##<a id="progress-record" name="Progress"></a> 进度记录

如果你将 Runbook 配置为记录进度记录（在 Azure 经典管理门户中 Runbook 的“配置”选项卡上），则在运行每个活动之前和之后，会向作业历史记录中写入一条记录。在大多数情况下，你应该保留默认设置，即，不记录 Runbook 的进度记录，以最大程度地提高性能。启用此选项的目的只是为了排查 Runbook 的问题或对它进行调试。在测试 Runbook 时，将不显示进度消息，即使已将该 Runbook 配置为记录进度记录。

[Write-Progress](http://technet.microsoft.com/zh-cn/library/hh849902.aspx) cmdlet 在 Runbook 中无效，因为此 cmdlet 旨在供交互式用户使用。

##<a id="preference-variables" name="PreferenceVariables"></a> Preference 变量

Windows PowerShell 使用 [preference 变量](http://technet.microsoft.com/zh-cn/library/hh847796.aspx)来确定如何响应发送到不同输出流的数据。你可以在 Runbook 中设置这些变量，以控制 Runbook 如何响应发送到不同流中的数据。

下表列出了可在 Runbook 中使用的 preference 变量及其有效值和默认值。请注意，此表格仅包含在 Runbook 中有效的值。preference 变量的其他值在 Azure 自动化外部的 Windows PowerShell 中使用时有效。

| 变量| 默认值| 有效值|
|:---|:---|:---|
|WarningPreference|继续|Stop<br>Continue<br>SilentlyContinue|
|ErrorActionPreference|继续|Stop<br>Continue<br>SilentlyContinue|
|VerbosePreference|SilentlyContinue|Stop<br>Continue<br>SilentlyContinue|

下表列出了在 Runbook 中有效的 preference 变量值的行为。

| 值| 行为|
|:---|:---|
|继续|继续记录消息并继续执行 Runbook。|
|SilentlyContinue|继续执行 Runbook 但不记录消息。这会导致忽略消息。|
|停止|记录消息并挂起 Runbook。|

## 检索 Runbook 输出和消息

### Azure 经典管理门户

可以从 Azure 经典管理门户中 Runbook 的“作业”选项卡查看 Runbook 作业的详细信息。作业的“摘要”将显示输入参数和[输出流](#Output)，此外，还显示有关作业的一般信息以及发生的任何异常。“历史记录”包含来自[输出流](#Output)以及[警告和错误流](#WarningError)的消息，此外，如果 Runbook 已配置为记录详细记录和进度记录，则该选项卡还包含[详细流](#Verbose)和[进度记录](#Progress)。

### Windows PowerShell

在 Windows PowerShell 中，可以使用 [Get-AzureAutomationJobOutput](http://msdn.microsoft.com/zh-cn/library/dn690268.aspx) cmdlet 检索 Runbook 的输出和消息。此 cmdlet 需要作业的 ID，如果你指定了要返回的流，则它还要使用一个名为 Stream 的参数。可以指定 Any 来返回作业的所有流。

以下示例将启动一个示例 Runbook，然后等待该 Runbook 完成。完成后，将从作业收集该 Runbook 的输出流。

	$job = Start-AzureAutomationRunbook –AutomationAccountName "MyAutomationAccount" –Name "Test-Runbook" 
	
	$doLoop = $true
	While ($doLoop) {
	   $job = Get-AzureAutomationJob –AutomationAccountName "MyAutomationAccount" -Id $job.Id
	   $status = $job.Status
	   $doLoop = (($status -ne "Completed") -and ($status -ne "Failed") -and ($status -ne "Suspended") -and ($status -ne "Stopped") 
	}
	
	Get-AzureAutomationJobOutput –AutomationAccountName "MyAutomationAccount" -Id $job.Id –Stream Output

## 相关文章

- [跟踪 Runbook 作业](/documentation/articles/automation-runbook-execution/)
- [子 Runbook](/documentation/articles/automation-child-runbooks/)

<!---HONumber=Mooncake_0215_2016-->
