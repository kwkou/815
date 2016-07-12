<properties 
pageTitle="使用 PowerShell 为 Azure 云服务中的角色启用远程桌面连接" 
description="如何使用 PowerShell 配置 Azure 云服务应用程序以允许远程桌面连接" 
services="cloud-services" 
documentationCenter="" 
authors="sbtron" 
manager="timlt" 
editor=""/>

<tags 
ms.service="cloud-services" 
ms.date="09/17/2015" 
wacn.date="10/17/2015"/>

# 使用 PowerShell 为 Azure 云服务中的角色启用远程桌面连接

>[AZURE.SELECTOR]
- [Azure 管理门户](/documentation/articles/cloud-services-role-enable-remote-desktop/)
- [PowerShell](/documentation/articles/cloud-services-role-enable-remote-desktop-powershell/)
- [Visual Studio](https://msdn.microsoft.com/zh-cn/library/gg443832.aspx)


你可以通过远程桌面访问在 Azure 中运行的角色的桌面。你可以使用远程桌面连接，在应用程序正在运行时排查和诊断其问题。

本文介绍如何使用 PowerShell 在云服务角色上启用远程桌面。有关本文所需的先决条件，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。PowerShell 使用远程桌面扩展方法，以便即使在部署应用程序之后，也能启用远程桌面。


## 从 PowerShell 配置远程桌面

使用 [Set-AzureServiceRemoteDesktopExtension](https://msdn.microsoft.com/zh-cn/library/azure/dn495117.aspx) cmdlet 可以在云服务部署的指定角色或所有角色上启用远程桌面。该 cmdlet 允许你通过接受 PSCredential 对象的 *Credential* 参数为远程桌面用户指定用户名和密码。

如果你以交互方式使用 PowerShell，则可以通过调用 [Get-Credentials](https://technet.microsoft.com/zh-cn/library/hh849815.aspx) cmdlet 来轻松设置 PSCredential 对象。

```
	$remoteusercredentials = Get-Credential
```

这将显示一个对话框，允许你以安全方式为远程用户输入用户名和密码。

由于 PowerShell 将主要用于自动化方案，因此还可以通过无需用户交互的方式设置 PSCredential 对象。为此，你首先需要设置一个安全密码。首先指定纯文本密码，然后使用 [ConvertTo-SecureString](https://technet.microsoft.com/zh-cn/library/hh849818.aspx) 将其转换为安全字符串。接下来，需要使用 [ConvertFrom-SecureString](https://technet.microsoft.com/zh-cn/library/hh849814.aspx) 将此安全字符串转换为加密的标准字符串。现在可以使用 [Set-Content](https://technet.microsoft.com/zh-cn/library/ee176959.aspx) 将此加密的标准字符串保存到文件。创建 PSCredential 对象时，可以从此文件读取以便以安全方式设置密码，而无需在提示中指定密码或以纯文本方式存储密码。

使用以下 PowerShell 创建安全的密码文件：

```
	ConvertTo-SecureString -String "Password123" -AsPlainText -Force | ConvertFrom-SecureString | Set-Content "password.txt"
``` 

创建密码文件 (password.txt) 后，你将仅使用此文件并将不需要以纯文本格式指定密码。如果你需要更新密码，则可以使用新密码再次运行上面的 powershell 以生成新的 password.txt 文件。

>[AZURE.IMPORTANT]设置密码时请确保满足[复杂性要求](https://technet.microsoft.com/zh-cn/library/cc786468.aspx)。

若要从安全密码文件创建凭据对象，必须读取该文件的内容并使用 [ConvertTo-SecureString](https://technet.microsoft.com/zh-cn/library/hh849818.aspx) 将其转换回为安全字符串。除了凭据外，[Set-AzureServiceRemoteDesktopExtension](https://msdn.microsoft.com/zh-cn/library/azure/dn495117.aspx) cmdlet 还接受 *Expiration* 参数，该参数指定用户帐户将过期的日期时间。你可以通过指定静态的日期和时间或可以简单地选择从当前日期经过几天后帐户过期来设置该参数。

以下 PowerShell 示例显示如何在云服务上设置远程桌面扩展：

```
	$servicename = "cloudservice"
	$username = "RemoteDesktopUser"
	$securepassword = Get-Content -Path "password.txt" | ConvertTo-SecureString
	$expiry = $(Get-Date).AddDays(1)
	$credential = New-Object System.Management.Automation.PSCredential $username,$securepassword
	Set-AzureServiceRemoteDesktopExtension -ServiceName $servicename -Credential $credential -Expiration $expiry 
```
你还可以选择指定部署槽以及要在其上启用远程桌面的角色。如果未指定这些参数，则该 cmdlet 将默认为使用生产部署槽，并在生产部署中的所有角色上启用远程桌面。

远程桌面扩展与部署相关联。如果你打算为服务创建新部署，则将需要在新部署上重新启用远程桌面。如果你要始终在部署上启用远程桌面，则应考虑将用于启用远程桌面的 PowerShell 脚本集成到部署工作流中。


## 通过远程桌面连接到角色实例
可使用 [Get-AzureRemoteDesktopFile](https://msdn.microsoft.com/zh-cn/library/azure/dn495261.aspx) cmdlet 通过远程桌面连接到云服务的特定角色实例。可以使用该 cmdlet 上的 *LocalPath* 参数本地下载 RDP 文件，也可以使用 *Launch* 参数直接启动“远程桌面连接”对话框来访问云服务角色实例。

```
	Get-AzureRemoteDesktopFile -ServiceName $servicename -Name "WorkerRole1_IN_0" -Launch
```


## 检查是否在服务上启用了远程桌面扩展 
[Get-AzureServiceRemoteDesktopExtension](https://msdn.microsoft.com/zh-cn/library/azure/dn495261.aspx) cmdlet 显示是否在服务部署上启用了远程桌面。该 cmdlet 将返回启用了远程桌面扩展的远程桌面用户和角色的用户名。你可以选择指定默认用于生产的部署槽。

```
	Get-AzureServiceRemoteDesktopExtension -ServiceName $servicename
```

## 从服务中删除远程桌面扩展 
如果已在部署上启用了远程桌面扩展并且需要更新远程桌面设置，则必须先删除远程桌面扩展，然后使用新设置重新启用它。例如，如果你需要为远程用户帐户设置新密码，或者如果用户帐户已过期，则需要删除该扩展，然后使用新密码或过期时间重新添加它。仅在已启用远程桌面扩展的现有部署上才需要这么做。对于新部署，你只需直接应用扩展。

若要从服务部署中删除远程桌面扩展，可以使用 [Remove-AzureServiceRemoteDesktopExtension](https://msdn.microsoft.com/zh-cn/library/azure/dn495280.aspx) cmdlet。你还可以选择指定要从中删除远程桌面扩展的部署槽和角色。

```
Remove-AzureServiceRemoteDesktopExtension -ServiceName $servicename -UninstallConfiguration

```  

>[AZURE.NOTE]*UninstallConfiguration* 参数将卸载任何已应用于服务的扩展配置。所有扩展配置均与服务配置相关联，以使用部署激活扩展，部署必须与该扩展配置相关联。如果在未指定 *UninstallConfiguration* 的情况下调用 Remove cmdlet，则将解除部署与扩展配置的关联，从而实际上从部署中删除扩展。但是，扩展配置仍将保持与服务相关联。若要完全删除扩展配置，应使用 *UninstallConfiguration*参数调用 Remove cmdlet。



## 其他资源

[如何配置云服务](/documentation/articles/cloud-services-how-to-configure/)

<!---HONumber=74-->