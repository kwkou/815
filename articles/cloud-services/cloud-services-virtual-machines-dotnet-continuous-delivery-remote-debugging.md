<properties
	pageTitle="使用持续交付启用远程调试 | Azure"
	description="了解在使用持续交付部署到 Azure 时如何启用远程调试"
	services="cloud-services"
	documentationCenter=".net"
	authors="TomArcher"
	manager="douge"
	editor=""/>

<tags
	ms.service="cloud-services"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-multiple"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="11/18/2016"
	wacn.date="01/03/2017"
	ms.author="tarcher"/>

# 使用持续交付功能发布到 Azure 时如何启用远程调试

使用[持续交付](/documentation/articles/cloud-services-dotnet-continuous-delivery/)发布到 Azure 时，可通过执行以下步骤，在 Azure 中为云服务或虚拟机启用远程调试。

## 为云服务启用远程调试

1. 在生成代理上，根据 [Azure 命令行生成](http://msdn.microsoft.com/zh-cn/library/hh535755.aspx)中所述设置 Azure 的初始环境。
2. 由于包需要远程调试运行时 (msvsmon.exe)，请安装 Remote Tools for Visual Studio 2015（如果使用的是 Visual Studio 2013，请安装 [Remote Tools for Microsoft Visual Studio 2013 Update 5](https://www.microsoft.com/zh-cn/download/details.aspx?id=48156)）。或者，也可以从装有 Visual Studio 的系统复制远程调试二进制文件。
3. 根据 [Azure 云服务的证书概述](/documentation/articles/cloud-services-certs-create/)中所述创建证书。保留 .pfx 和 RDP 证书指纹，并将证书上传到目标云服务。
4. 在 MSBuild 命令行中使用以下选项生成一个已启用远程调试的包。（将尖括号中的项替换为系统和项目文件的实际路径）。

		msbuild /TARGET:PUBLISH /PROPERTY:Configuration=Debug;EnableRemoteDebugger=true;VSX64RemoteDebuggerPath="<remote tools path>";RemoteDebuggerConnectorCertificateThumbprint="<thumbprint of the certificate added to the cloud service>";RemoteDebuggerConnectorVersion="2.7" "<path to your VS solution file>"

	`VSX64RemoteDebuggerPath` 是 Visual Studio 远程工具中 msvsmon.exe 所在的文件夹的路径。`RemoteDebuggerConnectorVersion` 是云服务中的 Azure SDK 版本。它也应与随 Visual Studio 一起安装的版本匹配。

5. 使用上一步中生成的包和 .cscfg 文件发布到目标云服务。
6. 将证书（.pfx 文件）导入到装有 Visual Studio 和用于 .NET 的 Azure SDK 的计算机。请确保导入到 `CurrentUser\My` 证书存储，否则附加到 Visual Studio 中的调试器会失败。

## 为虚拟机启用远程调试

1. 创建一个 Azure 虚拟机。请参阅[创建运行 Windows Server 的虚拟机](/documentation/articles/virtual-machines-windows-hero-tutorial/)或[在 Visual Studio 中创建和管理 Azure 虚拟机](/documentation/articles/virtual-machines-windows-classic-manage-visual-studio/)。
2. 在 [Azure 经典管理门户页](http://go.microsoft.com/fwlink/p/?LinkID=269851)上的虚拟机仪表板中，查看虚拟机的“RDP 证书指纹”。扩展配置中的 `ServerThumbprint` 值将使用此值。
3. 根据 [Azure 云服务的证书概述](/documentation/articles/cloud-services-certs-create/)中所述创建客户端证书（保留 .pfx 和 RDP 证书指纹）。
4. 根据[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/) 中所述安装 Azure Powershell（0.7.4 或更高版本）。
5. 运行以下脚本以启用 RemoteDebug 扩展。将路径和个人数据替换为自己的数据，例如，订阅名称、服务名称和指纹。

	>[AZURE.NOTE] 此脚本是针对 Visual Studio 2015 配置的。如果使用的是 Visual Studio 2013，请修改下面的 `$referenceName` 和 `$extensionName` 赋值以使用 `RemoteDebugVS2013`（而不是使用 `RemoteDebugVS2015`）。

	<pre>
	Add-AzureRmAccount -EnvironmentName AzureChinaCloud
	
	Select-AzureSubscription "My Microsoft Subscription"
	
	$vm = Get-AzureVM -ServiceName "mytestvm1" -Name "mytestvm1"
	
	$endpoints = @(
	,@{Name="RDConnVS2013"; PublicPort=30400; PrivatePort=30398}
	,@{Name="RDFwdrVS2013"; PublicPort=31400; PrivatePort=31398}
	)
	
	foreach($endpoint in $endpoints)
	{
	Add-AzureEndpoint -VM $vm -Name $endpoint.Name -Protocol tcp -PublicPort $endpoint.PublicPort -LocalPort $endpoint.PrivatePort
	}
	
	$referenceName = "Microsoft.VisualStudio.WindowsAzure.RemoteDebug.RemoteDebugVS2015"
	$publisher = "Microsoft.VisualStudio.WindowsAzure.RemoteDebug"
	$extensionName = "RemoteDebugVS2015"
	$version = "1.*"
	$publicConfiguration = "<PublicConfig><Connector.Enabled>true</Connector.Enabled><ClientThumbprint>56D7D1B25B472268E332F7FC0C87286458BFB6B2</ClientThumbprint><ServerThumbprint>E7DCB00CB916C468CC3228261D6E4EE45C8ED3C6</ServerThumbprint><ConnectorPort>30398</ConnectorPort><ForwarderPort>31398</ForwarderPort></PublicConfig>"
	
	$vm | Set-AzureVMExtension `
	-ReferenceName $referenceName `
	-Publisher $publisher `
	-ExtensionName $extensionName `
	-Version $version `
	-PublicConfiguration $publicConfiguration
	
	foreach($extension in $vm.VM.ResourceExtensionReferences)
	{
	if(($extension.ReferenceName -eq $referenceName) `
	-and ($extension.Publisher -eq $publisher) `
	-and ($extension.Name -eq $extensionName) `
	-and ($extension.Version -eq $version))
	{
	$extension.ResourceExtensionParameterValues[0].Key = 'config.txt'
	break
	}
	}
	
	$vm | Update-AzureVM
	</pre>

6. 将证书导入到装有 Visual Studio 和用于 .NET 的 Azure SDK 的计算机。

<!---HONumber=Mooncake_Quality_Review_1202_2016-->