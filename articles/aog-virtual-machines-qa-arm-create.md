<properties 
	pageTitle="订阅无法在 ARM 模式下创建虚拟机，只能在 ASM 模式下创建 Azure VM 部署" 
	description="订阅无法在 ARM 模式下创建虚拟机，只能在 ASM 模式下创建 Azure VM 部署。" 
	services="virtual machine" 
	documentationCenter="" 
	authors=""
	manager="" 
	editor=""/>
<tags ms.service="virtual-machines-aog" ms.date="" wacn.date="12/05/2016"/>
# 订阅无法在 ARM 模式下创建虚拟机，只能在 ASM 模式下创建 Azure VM 部署 #

### 问题描述 ###

资源组所有者可以在新版 portal 创建经典模式的虚拟机，但是无法创建 ARM 模式的虚拟机。

### 问题现象 ###

环境中有个相对权限比较高的账户，比如 account admin （以下简称为 AA），这个账号创建一个资源组和对应的 owner。

如果用这个资源组的 owner 登陆 Azure，会出现这个问题：只能创建经典模式的虚拟机，但无法创建 ARM 模式的虚拟机。

### 问题分析 ###

ARM 下很多资源提供程序没有注册，包括 ARM 下的 Microsoft.Compute：
<table><tr><th> NameSpace </th><th> RegistrationState </th></tr>
<tr><td> Microsoft.Batch </td><td> Registered</td></tr>
<tr><td> Microsoft.ClassicCompute </td><td> Registered </td></tr>
<tr><td> Microsoft.ClassicNetwork </td><td> Registered </td></tr>
<tr><td> Microsoft.ClassicStorage </td><td> Registered </td></tr>
<tr><td> microsoft.insights </td><td> Registered </td></tr>
<tr><td> Microsoft.Sql </td><td> Registered </td></tr>
<tr><td> Microsoft.StreamAnalytics </td><td> Registered </td></tr>
<tr><td> Microsoft.Web </td><td> Registered </td></tr>
<tr><td> Microsoft.ApiManagement </td><td style="background:red"> NotRegistered </td></tr>
<tr><td> Microsoft.Authorization </td><td> Registered </td></tr>
<tr><td> Microsoft.Cache </td><td style="background:red"> NotRegistered </td></tr>
<tr><td> Microsoft.ClassicInfrastructureMigrate </td><td style="background:red"> NotRegistered </td></tr>
<tr><td> Microsoft.CognitiveServices </td><td style="background:red"> NotRegistered </td></tr>
<tr><td> Microsoft.Compute </td><td style="background:red"> NotRegistered </td></tr>
<tr><td> Microsoft.Devices </td><td style="background:red"> NotRegistered </td></tr>
<tr><td> Microsoft.DocumentDB </td><td style="background:red"> NotRegistered </td></tr>
<tr><td> Microsoft.EventHub </td><td style="background:red"> NotRegistered </td></tr>
<tr><td> Microsoft.Features </td><td> Registered </td></tr>
<tr><td> Microsoft.HDInsight </td><td style="background:red"> NotRegistered </td></tr>
<tr><td> Microsoft.KeyVault </td><td style="background:red"> NotRegistered </td></tr>
<tr><td> Microsoft.Media </td><td style="background:red"> NotRegistered </td></tr>
<tr><td> Microsoft.Network </td><td style="background:red"> NotRegistered </td></tr>
<tr><td> Microsoft.Portal </td><td style="background:red"> NotRegistered </td></tr>
<tr><td> Microsoft.Resources </td><td> Registered </td></tr>
<tr><td> Microsoft.Scheduler </td><td> Registered </td></tr>
<tr><td> Microsoft.ServiceBus </td><td style="background:red"> NotRegistered </td></tr>
<tr><td> Microsoft.ServiceFabric </td><td style="background:red"> NotRegistered </td></tr>
<tr><td> Microsoft.Storage </td><td style="background:red"> NotRegistered </td></tr></table>
AA 创建的资源组的 owner 之所以没有自动注册资源提供程序是由于 AA 的权限导致的。如果 AA 没有创建过 Azure 虚拟机，那么就没有自动注册过资源提供程序，所以 AA 所创建的资源组的 owner 也没有这个权限。

### 解决方法 ###

基于以上理论，解决方案有两个：

1.	手动运行 PowerShell 命令注册资源提供程序；
2.	用创建资源组 owner 的那个账号（AA） 先创建一个 ARM 的 Azure 虚拟机。



