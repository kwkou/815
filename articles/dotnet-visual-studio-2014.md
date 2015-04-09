<properties urlDisplayName="Visual Studio 14 CTP" pageTitle="安装 Azure SDK 2.4 for Visual Studio 14 CTP2" metaKeywords="Visual Studio, Azure SDK" description="安装 Azure SDK 2.4 和 Visual Studio 14 CTP2" metaCanonical="" services="" documentationCenter="" title="Installing Azure SDK 2.4 for Visual Studio 14 CTP" authors="ghogen" solutions="" manager="douge" editor="" />

# 安装 Azure SDK 2.4 for Visual Studio "14" CTP

若要安装 Azure SDK 2.4 for .NET with Visual Studio "14" CTP，请执行以下步骤。此过程安装 SDK、基本工具和扩展工具，用于通过 Visual Studio "14" CTP 进行的 Azure 开发，不能用于任何其他版本的 Visual Studio。

**注意**：Azure SDK 2.4 不兼容 Visual Studio "14" CTP1。

若要安装 Azure SDK 2.4 for .NET，请执行以下步骤：

1. 安装最新版的 [Visual Studio "14" CTP](http://www.visualstudio.com/zh-cn/downloads/visual-studio-2015-downloads-vs)。

2. 安装 Azure SDK 的每个组件，使用以下列表中的链接，并遵循相应的顺序。选择以下每个组件的 x86 或 x64 版本。

<ul>
   <li>Azure 创作工具：<a href="http://go.microsoft.com/fwlink/p/?LinkId=400892">MicrosoftAzureAuthoringTools-x86.msi</a> 或 <a href="http://go.microsoft.com/fwlink/p/?LinkId=400893">MicrosoftAzureAuthoringTools-x64.msi</a>。</li>
   <li>Azure 计算模拟器：<a href="http://go.microsoft.com/fwlink/p/?LinkId=400894">MicrosoftAzureEmulator-x86.exe</a> 或 <a href="http://go.microsoft.com/fwlink/p/?LinkId=400895">MicrosoftAzureEmulator-x64.exe</a>。</li>
   <li>Azure 客户端库：<a href="http://go.microsoft.com/fwlink/p/?LinkId=400896">MicrosoftAzureLibsForNet-x86.msi</a> 或 <a href="http://go.microsoft.com/fwlink/p/?LinkId=400897">MicrosoftAzureLibsForNet-x64.msi</a>。</li>
   <li>存储模拟器：<a href="http://go.microsoft.com/fwlink/p/?LinkId=400904">MicrosoftAzureStorageEmulator.msi</a>。                            如果你收到有关本地 SQL 数据库的警告，请在安装 SQL Server LocalDB 11.0 时从<a href="http://go.microsoft.com/fwlink/p/?LinkId=400778">此位置</a>选择 x86 版本的，或从<a href="http://go.microsoft.com/fwlink/p/?LinkId=400779">此位置</a>选择 x64 版本的。</li>
   <li> Azure Tools for Visual Studio：<a href="http://go.microsoft.com/fwlink/p/?LinkId=400903">WindowsAzureTools.vs140.exe</a>。</li>
</ul>

## 已知问题

1. 如果你在安装了 Visual Studio 2013 的计算机上安装 Visual Studio "14" CTP2，你将无法启动 Visual Studio "14" CTP2 中的移动服务。为了解决这个问题，请在你的移动服务项目中添加对以下程序集的引用：

	* packages/Microsoft.Data.OData.5.6.0/lib/net40/Microsoft.Data.OData.dll
	* packages/Microsoft.Data.Edm.5.6.0/lib/net40/Microsoft.Data.Edm.dll

2. 针对 Azure 网站和移动服务的远程调试在 Visual Studio "14" CTP2 中无法执行。

## 发行说明

请阅读 Azure SDK 2.4 的 [发行说明](https://msdn.microsoft.com/zh-cn/library/azure/dn794167.aspx)。
<!--HONumber=43--> 
