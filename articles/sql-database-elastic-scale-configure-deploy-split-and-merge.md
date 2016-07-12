<properties
	pageTitle="弹性数据库拆分/合并工具教程 | Azure"
	description="使用弹性数据库工具进行拆分与合并"
	services="sql-database"  
	documentationCenter=""
	authors="sidneyh"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="02/23/2016"
	wacn.date="06/14/2016" />

# 弹性数据库拆分/合并工具教程

## 下载拆分/合并包
1. 从 [NuGet](http://docs.nuget.org/docs/start-here/installing-nuget) 下载最新的 NuGet 版本。
2. 打开命令提示符，然后导航到您下载 nuget.exe 的目录。
3. 使用以下命令将最新的拆分/合并包下载到当前目录中：`nuget install Microsoft.Azure.SqlDatabase.ElasticScale.Service.SplitMerge`  

上面的步骤可将拆分/合并文件下载到当前目录。文件放置在名为 **Microsoft.Azure.SqlDatabase.ElasticScale.Service.SplitMerge.x.x.xxx.x** 的目录中，其中 *x.x.xxx.x* 表示版本号。拆分/合并服务文件可在 **content\\splitmerge\\service** 子目录中找到；拆分/合并 PowerShell 脚本（和所需的客户端 .dll）可在 **content\\splitmerge\\powershell** 子目录中找到。

## 先决条件

1. 创建将用作拆分/合并状态数据库的 Azure SQL DB。转到 [Azure 管理门户](https://manage.windowsazure.cn)。创建新的 **SQL 数据库**。填写数据库名称并创建新的用户和密码。确保记录该名称和密码以供日后使用。

2. 确保你的 Azure SQL DB 服务器允许 Azure 服务与其连接。在门户上的“防火墙设置”中，确保“允许访问 Azure 服务”设置设为“打开”。单击“保存”图标。

    ![允许的服务][1]

3. 创建将用于诊断输出的 Azure 存储帐户。转到 Azure 门户。在左侧栏中，依次单击“新建”、“数据 + 存储”、“存储”。

4. 创建将包含拆分/合并服务的 Azure 云服务。转到 Azure 门户。在左侧栏中，依次单击“新建”、“计算”、“云服务”、“创建”。


## 配置您的拆分/合并服务

### 拆分/合并服务配置

1. 在你下载拆分/合并程序集的文件夹中，创建 **SplitMergeService.cspkg** 随附的 **ServiceConfiguration.Template.cscfg** 文件的副本，并将其重命名为 **ServiceConfiguration.cscfg**。

2. 在文本编辑器（如 Visual Studio）中打开 **ServiceConfiguration.cscfg**，它会验证输入内容（例如，证书指纹的格式）。

3. 创建新的数据库或选择现有的数据库，以将其用作拆分/合并操作的状态数据库并检索该数据库的连接字符串。

	**重要说明：**目前，状态数据库必须使用拉丁语排序规则 (SQL\_Latin1\_General\_CP1\_CI\_AS)。有关详细信息，请参阅 [Windows 排序规则名称 (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms188046.aspx)。

	在 Azure SQL DB 中，连接字符串通常采用以下形式：

        "Server=myservername.database.chinacloudapi.cn; Database=mydatabasename;User ID=myuserID; Password=mypassword; Encrypt=True; Connection Timeout=30" .
4.    同时在 ElasticScaleMetadata 设置的 **SplitMergeWeb** 和 **SplitMergeWorker** 角色部分中，在 cscfg 文件内输入此连接字符串。

5.    对于 **SplitMergeWorker** 角色，在 **WorkerRoleSynchronizationStorageAccountConnectionString** 设置中输入有效的连接字符串用于连接到 Azure 存储空间。
        
### 配置安全性
有关配置服务安全性的详细说明，请参阅[拆分/合并安全配置](/documentation/articles/sql-database-elastic-scale-split-merge-security-configuration/)。

为了针对本教程创建一个简单的测试部署，我们将执行少量的配置步骤来使服务正常运行。仅一个计算机/帐户可以执行这些步骤，以便与服务进行通信。

### 创建自签名证书

创建新的目录并使用 [Visual Studio 的开发人员命令提示符](http://msdn.microsoft.com/zh-cn/library/ms229859.aspx)窗口从该目录执行以下命令：

    makecert ^
    -n "CN=*.chinacloudapp.cn" ^
    -r -cy end -sky exchange -eku "1.3.6.1.5.5.7.3.1,1.3.6.1.5.5.7.3.2" ^
    -a sha1 -len 2048 ^
    -sr currentuser -ss root ^
    -sv MyCert.pvk MyCert.cer

将要求您提供密码以保护私钥。输入强密码并进行确认。之后，系统将提示您再次输入该密码。在完成后单击“是”，以将证书导入到“受信任的根证书颁发机构”存储中。

### 创建 PFX 文件

从执行 makecert 的相同窗口中执行以下命令；使用您用于创建证书的相同密码：

    pvk2pfx -pvk MyCert.pvk -spc MyCert.cer -pfx MyCert.pfx -pi <password>

### 将客户端证书导入到个人存储中
1. 在 Windows 资源管理器中，双击“MyCert.pfx”。
2. 在“证书导入向导”中，选择“当前用户”，然后单击“下一步”。
3. 确认文件路径，然后单击“下一步”。
4. 键入密码、保持选中“包括所有扩展属性”，然后单击“下一步”。
5. 保持选中“自动选择证书存储[…]”，然后单击“下一步”。
6. 依次单击“完成”和“确定”。

### 将 PFX 文件上载到云服务

转到 [Azure 管理门户](https://manage.windowsazure.cn)。

1. 选择“云服务”。
2. 选择您之前为拆分/合并服务创建的云服务。
3. 单击顶部菜单上的“证书”。
4. 单击底部栏中的“上载”。
5. 选择 PFX 文件并输入上面所述的相同密码。
6. 完成操作后，从列表中的新条目复制证书指纹。

### 更新服务配置文件

将之前复制的证书指纹粘贴到这些设置的指纹/值属性中。对于辅助角色：

    <Setting name="DataEncryptionPrimaryCertificateThumbprint" value="" />
    <Certificate name="DataEncryptionPrimary" thumbprint="" thumbprintAlgorithm="sha1" />

对于 Web 角色：

    <Setting name="AdditionalTrustedRootCertificationAuthorities" value="" />
    <Setting name="AllowedClientCertificateThumbprints" value="" />
    <Setting name="DataEncryptionPrimaryCertificateThumbprint" value="" />
    <Certificate name="SSL" thumbprint="" thumbprintAlgorithm="sha1" />
    <Certificate name="CA" thumbprint="" thumbprintAlgorithm="sha1" />
    <Certificate name="DataEncryptionPrimary" thumbprint="" thumbprintAlgorithm="sha1" />


请注意，对于生产部署，应针对用于加密的 CA 使用单独的证书（服务器证书和客户端证书）。有关此内容的详细说明，请参阅[安全配置](/documentation/articles/sql-database-elastic-scale-split-merge-security-configuration/)。

### 部署你的拆分/合并服务

1. 转到 [Azure 管理门户](https://manage.windowsazure.cn)。
2. 单击左侧的“云服务”选项卡，然后选择你之前创建的云服务。
3. 单击“仪表板”。
4. 选择过渡环境，然后单击“上载新的过渡部署”。

    ![过渡][3]

5. 在对话框中，输入一个部署标签。对于“程序包”和“配置”，单击“从本地”，然后选择 **SplitMergeService.cspkg** 文件和你之前配置的 .cscfg 文件。
6. 确保选中标记为“即使一个或多个角色包含单个实例也部署”的复选框。
7. 点击右下角的勾选按钮以开始部署。它预计需要几分钟的时间才能完成。

	![上载][4]


## 部署故障排除

如果您的 Web 角色无法联机，可能是安全配置出现了问题。检查 SSL 是否按照上面的描述进行了配置。

如果您的辅助角色无法联机，但是您的 Web 角色已成功，很可能是在连接到您之前创建的状态数据库时出现了问题。

* 确保您的 .cscfg 中的连接字符串正确。
* 检查服务器和数据库是否存在，以及用户 ID 和密码是否正确。
* 对于 Azure SQL DB，连接字符串应采用以下形式：

        "Server=myservername.database.chinacloudapi.cn; Database=mydatabasename;User ID=myuserID; Password=mypassword; Encrypt=True; Connection Timeout=30" .

* 确保服务器名称不以 **https://** 开头。
* 确保你的 Azure SQL DB 服务器允许 Azure 服务与其连接。若要执行此操作，请打开 https://manage.windowsazure.cn，依次单击左侧的“SQL 数据库s”和顶部的“服务器”，然后选择你的服务器。在顶部单击“配置”并确保将“Azure 服务”设置为“是”。（请参阅此文章顶部的“先决条件”部分）。

## 测试你的拆分/合并服务部署

### 使用 Web 浏览器建立连接

确定您的拆分/合并服务的 Web 终结点。通过转到云服务的“仪表板”并在右侧的“站点 URL”下查找，你可以在 Azure 管理门户中找到此内容。由于默认的安全设置将禁用 HTTP 终结点，因此请将 **http://** 替换为 **https://**。将此 URL 的页面加载到您的浏览器中。

### 使用 PowerShell 脚本进行测试

可以通过运行包含的示例 PowerShell 脚本测试部署和您的环境。

包含的脚本文件为：

1. **SetupSampleSplitMergeEnvironment.ps1** - 为拆分/合并设置测试数据层（有关详细说明，请参见下表）
2. **ExecuteSampleSplitMerge.ps1** - 在测试数据层上执行测试操作（有关详细说明，请参见下表）
3. **GetMappings.ps1** – 可输出分片映射的当前状态的顶级示例脚本。
4. **ShardManagement.psm1** – 可包装 ShardManagement API 的帮助程序脚本
5. **SqlDatabaseHelpers.psm1** – 用于创建和管理 SQL 数据库的帮助程序脚本

<table style="width:100%">
  <tr>
    <th>PowerShell 文件</th>
    <th>步骤</th>
  </tr>
  <tr>
    <th rowspan="5">SetupSampleSplitMergeEnvironment.ps1</th>
    <td>1.创建分片映射管理器数据库</td>
  </tr>
  <tr>
    <td>2.创建 2 个分片数据库。
  </tr>
  <tr>
    <td>3.为这些数据库创建一个分片映射（删除这些数据库上的任何现有的分片映射）。</td>
  </tr>
  <tr>
    <td>4.在这两个分片中创建一个小的示例表，然后使用一个分片填充该表。</td>
  </tr>
  <tr>
    <td>5.声明分片表的 SchemaInfo。</td>
  </tr>

</table>

<table style="width:100%">
  <tr>
    <th>PowerShell 文件</th>
    <th>步骤</th>
  </tr>
<tr>
    <th rowspan="4">ExecuteSampleSplitMerge.ps1 </th>
    <td>1.将拆分请求发送到拆分/合并服务 Web 前端，以将数据从第一个分片到第二个分片拆分为两半。</td>
  </tr>
  <tr>
    <td>2.轮询拆分请求状态的 Web 前端并等待该请求完成。</td>
  </tr>
  <tr>
    <td>3.将合并请求发送到拆分/合并服务 Web 前端，以将数据从第二个分片移回到第一个分片。</td>
  </tr>
  <tr>
    <td>4.轮询合并请求状态的 Web 前端并等待该请求完成。</td>
  </tr>
</table>

##使用 PowerShell 验证你的部署

1.    打开新的 PowerShell 窗口并导航到您下载拆分/合并包的目录，然后导航到“powershell”目录中。
2.    创建将在其中创建分片映射管理器和分片的 Azure SQL 数据库服务器（或选择现有服务器）。

    注意：默认情况下，SetupSampleSplitMergeEnvironment.ps1 脚本将在相同的服务器上创建所有这些数据库以简化脚本。这并不表示拆分/合并服务本身存在限制。

    拆分/合并服务将需要具有数据库读/写访问权限的 SQL 身份验证登录，才能移动数据并更新分片映射。由于拆分/合并服务在云中运行，因此它当前不支持集成的身份验证。

    确保 Azure SQL 服务器已配置为允许从运行这些脚本的计算机的 IP 地址进行访问。您可以在“Azure SQL 服务器”/“配置”/“允许的 IP 地址”下找到此设置。

3.    执行 SetupSampleSplitMergeEnvironment.ps1 脚本以创建示例环境。

    运行此脚本将擦除分片映射管理器数据库和分片上任何现有的分片映射管理数据结构。如果您想要重新初始化分片映射或分片，重新运行脚本可能会很有用。

    示例命令行：

        .\SetupSampleSplitMergeEnvironment.ps1 `
            -UserName 'mysqluser' `
            -Password 'MySqlPassw0rd' `
            -ShardMapManagerServerName 'abcdefghij.database.chinacloudapi.cn'

4.    执行 Getmappings.ps1 脚本以查看示例环境中当前存在的映射。

        .\GetMappings.ps1 `
            -UserName 'mysqluser' `
            -Password 'MySqlPassw0rd' `
            -ShardMapManagerServerName 'abcdefghij.database.chinacloudapi.cn'

5.    执行 ExecuteSampleSplitMerge.ps1 脚本以执行拆分操作（将第一个分片上一半的数据移至第二个分片），然后执行合并操作（将数据移回第一个分片）。如果您已配置 SSL 并且已将 http 终结点保留为禁用，请确保改为使用 https:// 终结点。

    示例命令行：

        .\ExecuteSampleSplitMerge.ps1 `
            -UserName 'mysqluser' `
            -Password 'MySqlPassw0rd' `
            -ShardMapManagerServerName 'abcdefghij.database.chinacloudapi.cn' `
            -SplitMergeServiceEndpoint 'https://mysplitmergeservice.chinacloudapp.cn' `
            -CertificateThumbprint '0123456789abcdef0123456789abcdef01234567'

    如果您收到以下错误，很有可能是您的 Web 终结点证书出现了问题。尝试使用您最喜欢的 Web 浏览器连接到 Web 终结点并检查是否存在证书错误。

        Invoke-WebRequest : The underlying connection was closed: Could not establish trust relationship for the SSL/TLSsecure channel.

    如果成功，则输出应如下所示：

        > .\ExecuteSampleSplitMerge.ps1 -UserName 'mysqluser' -Password 'MySqlPassw0rd' -ShardMapManagerServerName 'abcdefghij.database.chinacloudapi.cn' -SplitMergeServiceEndpoint 'http://mysplitmergeservice.chinacloudapp.cn' –CertificateThumbprint 0123456789abcdef0123456789abcdef01234567
        Sending split request
        Began split operation with id dc68dfa0-e22b-4823-886a-9bdc903c80f3
        Polling split-merge request status. Press Ctrl-C to end
        Progress: 0% | Status: Queued | Details: [Informational] Queued request
        Progress: 5% | Status: Starting | Details: [Informational] Starting split-merge state machine for request.
        Progress: 5% | Status: Starting | Details: [Informational] Performing data consistency checks on target     shards.
        Progress: 20% | Status: CopyingReferenceTables | Details: [Informational] Moving reference tables from     source to target shard.
        Progress: 20% | Status: CopyingReferenceTables | Details: [Informational] Waiting for reference tables copy     completion.
        Progress: 20% | Status: CopyingReferenceTables | Details: [Informational] Moving reference tables from     source to target shard.
        Progress: 44% | Status: CopyingShardedTables | Details: [Informational] Moving key range [100:110) of     Sharded tables
        Progress: 44% | Status: CopyingShardedTables | Details: [Informational] Successfully copied key range     [100:110) for table [dbo].[MyShardedTable]
        ...
        ...
        Progress: 90% | Status: Completing | Details: [Informational] Successfully deleted shardlets in table     [dbo].[MyShardedTable].
        Progress: 90% | Status: Completing | Details: [Informational] Deleting any temp tables that were created     while processing the request.
        Progress: 100% | Status: Succeeded | Details: [Informational] Successfully processed request.
        Sending merge request
        Began merge operation with id 6ffc308f-d006-466b-b24e-857242ec5f66
        Polling request status. Press Ctrl-C to end
        Progress: 0% | Status: Queued | Details: [Informational] Queued request
        Progress: 5% | Status: Starting | Details: [Informational] Starting split-merge state machine for request.
        Progress: 5% | Status: Starting | Details: [Informational] Performing data consistency checks on target     shards.
        Progress: 20% | Status: CopyingReferenceTables | Details: [Informational] Moving reference tables from     source to target shard.
        Progress: 44% | Status: CopyingShardedTables | Details: [Informational] Moving key range [100:110) of     Sharded tables
        Progress: 44% | Status: CopyingShardedTables | Details: [Informational] Successfully copied key range     [100:110) for table [dbo].[MyShardedTable]
        ...
        ...
        Progress: 90% | Status: Completing | Details: [Informational] Successfully deleted shardlets in table     [dbo].[MyShardedTable].
        Progress: 90% | Status: Completing | Details: [Informational] Deleting any temp tables that were created     while processing the request.
        Progress: 100% | Status: Succeeded | Details: [Informational] Successfully processed request.

6.    试用其他数据类型！ 所有这些脚本均采取可选的 -ShardKeyType 参数，该参数允许您指定密钥类型。默认值为 Int32，但您也可以指定 Int64、Guid 或 Binary。

## 创建你自己的请求

可以通过 Web UI 或通过导入并使用 SplitMerge.psm1 PowerShell 模块（该模块将通过 Web 角色提交你的请求）使用该服务。

拆分/合并服务可以将数据同时移入分片表和引用表。分片表具有一个分片密钥列并在不同的分片上具有不同的行数据。引用表未进行分片，因此它在每个分片上都包含相同的行数据。引用表适用于不经常更改的数据，并且可用于接联查询中的分片表。

为了执行拆分/合并操作，您必须声明要移动的分片表和引用表。将使用 **SchemaInfo** API 完成此操作。此 API 位于 **Microsoft.Azure.SqlDatabase.ElasticScale.ShardManagement.Schema** 命名空间中。

1.    对于每个分片表，请创建一个 **ShardedTableInfo** 对象，该对象在包含分片密钥的表中描述了此表的父架构名称（可选，默认为“dbo”）、表名称以及列名称。
2.    对于每个引用表，请创建一个 **ShardedTableInfo** 对象，该对象描述了此表的父架构名称（可选，默认为“dbo”）和表名称。
3.    将上面的 TableInfo 对象添加到新的 **SchemaInfo** 对象。
4.    获取对 **ShardMapManager** 对象的引用，然后调用 **GetSchemaInfoCollection**。
5.    将 **SchemaInfo** 添加到 **SchemaInfoCollection**，从而提供分片映射名称。

可在 SetupSampleSplitMergeEnvironment.ps1 脚本中看到此操作的示例。

请注意，拆分/合并服务不会为您创建目标数据库（或在数据库中为任何表创建架构）。在将请求发送到服务之前，必须预先创建它们。


## 故障排除
在运行示例 powershell 脚本时，您可能会看到下面的消息：

    Invoke-WebRequest : The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel.

此错误表示您的 SSL 证书未正确配置。请按照“与 Web 浏览器连接”部分中的说明进行操作。

如果无法提交请求，你可能会看到:

 [异常] System.Data.SqlClient.SqlException (0x80131904): 找不到存储过程 'dbo.InsertRequest'。

在这种情况下，请检查你的配置文件，尤其是 **WorkerRoleSynchronizationStorageAccountConnectionString** 的设置。此错误通常表示辅助角色无法成功初始化首次使用的元数据数据库。

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/allowed-services.png
[2]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/manage.png
[3]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/staging.png
[4]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/upload.png
[5]: ./media/sql-database-elastic-scale-configure-deploy-split-and-merge/storage.png
 

<!---HONumber=Mooncake_0606_2016-->
