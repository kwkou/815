<properties
   pageTitle="使用 DSC 将凭据传递到 Azure | Azure"
   description="概述如何使用 PowerShell Desired State Configuration 安全地将凭据传递给 Azure 虚拟机"
   services="virtual-machines-windows"
   documentationCenter=""
   authors="zjalexander"
   manager="timlt"
   editor=""
   tags="azure-service-management,azure-resource-manager"
   keywords=""/>  


<tags
   ms.service="virtual-machines-windows"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="vm-windows"
   ms.workload="na"
   ms.date="08/24/2016"
   wacn.date=""
   ms.author="zachal"/>  


# 将凭据传递到 Azure DSC 扩展处理程序 #

[AZURE.INCLUDE [了解部署模型](../../includes/learn-about-deployment-models-both-include.md)]

本文介绍 Azure 的 Desired State Configuration 扩展。有关 DSC 扩展处理程序的概述，请参阅 [Introduction to the Azure Desired State Configuration extension handler](/documentation/articles/virtual-machines-windows-extensions-dsc-overview/)（Azure Desired State Configuration 扩展处理程序简介）。

在配置过程中，你可能需要在用户上下文中设置用户帐户、访问服务或安装程序。若要执行这些操作，需提供凭据。

DSC 允许使用参数化配置，其中的凭据将传入配置并安全地存储在 MOF 文件中。Azure 扩展处理程序提供证书的自动管理功能，以此简化凭据管理。

请考虑使用以下 DSC 配置脚本，该脚本可创建具有指定密码的本地用户帐户：

*user_configuration.ps1*

	configuration Main
	{
	    param(
	        [Parameter(Mandatory=$true)]
	        [ValidateNotNullorEmpty()]
	        [PSCredential]
	        $Credential
	    )    
	    Node localhost {       
	        User LocalUserAccount
	        {
	            Username = $Credential.UserName
	            Password = $Credential
	            Disabled = $false
	            Ensure = "Present"
	            FullName = "Local User Account"
	            Description = "Local User Account"
	            PasswordNeverExpires = $true
	        } 
	    }  
	} 

必须将 *node localhost* 包含为配置的一部分。如果此语句缺失，则以下语句将无法运行，因为扩展处理程序会专门查找节点 localhost 语句。另外，必须包含 typecast *[PsCredential]*，因为此特定类型将触发扩展来加密凭据。

将此脚本发布到 Blob 存储：

`Publish-AzureVMDscConfiguration -ConfigurationPath .\user_configuration.ps1`

设置 Azure DSC 扩展并提供凭据：

	$configurationName = "Main"
	$configurationArguments = @{ Credential = Get-Credential }
	$configurationArchive = "user_configuration.ps1.zip"
	$vm = Get-AzureVM "example-1"
 
	$vm = Set-AzureVMDSCExtension -VM $vm -ConfigurationArchive $configurationArchive 
	-ConfigurationName $configurationName -ConfigurationArgument @configurationArguments
 
	$vm | Update-AzureVM

运行此代码时会出现输入凭据的提示。提供的凭据随即会存储在内存中。使用 `Set-AzureVmDscExtension` cmdlet 发布凭据时，凭据将通过 HTTPS 传输到 VM，Azure 将使用本地 VM 证书以加密形式将该凭据存储在 VM 的磁盘上。然后，凭据将即时在内存中解密再重新加密，以便传递给 DSC。

此行为不同于[使用不带扩展处理程序的安全配置](https://msdn.microsoft.com/powershell/dsc/securemof)。使用 Azure 环境可以通过证书安全地传输配置数据。使用 DSC 扩展处理程序时，无需在 ConfigurationData 中提供 $CertificatePath 或 $CertificateID / $Thumbprint 条目。


## 后续步骤 ##

有关 Azure DSC 扩展处理程序的详细信息，请参阅 [Introduction to the Azure Desired State Configuration extension handler](/documentation/articles/virtual-machines-windows-extensions-dsc-overview/)（Azure Desired State Configuration 扩展处理程序简介）。

有关 PowerShell DSC 的详细信息，请[访问 PowerShell 文档中心](https://msdn.microsoft.com/powershell/dsc/overview)。

若要查找可以使用 PowerShell DSC 管理的其他功能，请[浏览 PowerShell 库](https://www.powershellgallery.com/packages?q=DscResource&x=0&y=0)以获取更多 DSC 资源。

<!---HONumber=Mooncake_1017_2016-->