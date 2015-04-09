<properties urlDisplayName="AzCopy" pageTitle="如何将 AzCopy 与 Microsoft Azure 存储一起使用" metaKeywords="Get started Azure AzCopy   Azure unstructured data   Azure unstructured storage   Azure blob   Azure blob storage   Azure file   Azure file storage   Azure file share   AzCopy" description="了解如何使用 AzCopy 实用程序上载、下载以及复制 blob 和文件内容。" metaCanonical="" disqusComments="1" umbracoNaviHide="1" services="storage" documentationCenter="" title="How to use AzCopy with Windows Azure Storage" authors="tamram" manager="adinah" editor="cgronlun" />


# AzCopy 命令行实用程序入门

AzCopy 是一个高性能的命令行实用程序，用于将数据上载、复制到 Microsoft Azure Blob、文件和表存储以及从其中下载和复制数据。本指南提供了有关使用 AzCopy 的概述。

> [WACOM.NOTE] 本指南假定你已安装了 AzCopy 3.0.0 或更高版本。AzCopy 3.x 现在为公开发行版本。<br /> 
> 本指南还包括了如何使用 AzCopy 4.0.0，这是 AzCopy 的预览版本。在本指南中，只在预览版本中提供的功能标注有"预览版本"( *preview*) 字样。<br />
> 注意，对于 AzCopy 4.x，命令行选项和功能在将来的版本中可能会改变。

##目录

- [下载并安装 AzCopy](#install)
- [了解 AzCopy 命令行语法](#syntax)
- [限制复制数据时的并发写入](#limit-writes)
- [使用 AzCopy 复制 Azure blob](#copy-blobs)
- [使用 AzCopy 复制 Azure 文件共享中的文件（仅限预览版本）](#copy-files)
- [使用 AzCopy 复制 Azure 表实体（仅限预览版本）](#copy-entities)
- [AzCopy 版本](#versions)
- [后续步骤](#next-steps)

##<a id="install"></a> 下载并安装 AzCopy

1. 下载[最新版本的 AzCopy](http://aka.ms/downloadazcopy), or the [latest preview version](http://aka.ms/downloadazcopypr).
2. 运行安装。默认情况下，AzCopy 安装会在 `%ProgramFiles(x86)%\Microsoft SDKs\Azure\`（在运行 64 位 Windows 的计算机上）或 `%ProgramFiles%\Microsoft SDKs\Azure\`（在运行 32 位 Windows 的计算机上）中创建一个名为  `AzCopy` 的文件夹。不过，你可以通过安装向导更改安装路径。
3. 如果需要，你可以将 AzCopy 安装路径添加到系统路径。

##<a id="syntax"></a> 了解 AzCopy 命令行语法

接下来，打开一个命令窗口，然后导航到你的计算机上的 AzCopy 安装目录， `AzCopy.exe` 可执行文件位于其中。AzCopy 命令的基本语法是：

	AzCopy /Source:<source> /Dest:<destination> /Pattern:<filepattern> [Options]

> [WACOM.NOTE] 从 AzCopy 3.0.0 版开始，AzCopy 命令行语法要求在指定每个参数时都包括参数名称，例如 `/ParameterName:ParameterValue`。

下表中描述了 AzCopy 的参数。你还可以从命令行键入下列命令之一来获得有关如何使用 AzCopy 的帮助信息：

- 若要获得 AzCopy 的详细命令行帮助信息，请键入：`AzCopy /?`
- 若要获得任何 AzCopy 参数的详细帮助信息，请键入： `AzCopy /?:SourceKey`
- 若要获得命令行示例，请键入： `AzCopy /?:Samples` 


<table>
  <tr>
    <th>选项名称</th>
    <th>描述</th>
    <th>适用于 Blob 存储 (Y/N)</th>
    <th>适用于文件存储 (Y/N)（仅限预览版本）</th>
    <th>适用于表存储 (Y/N)（仅限预览版本）</th>
  </tr>
  <tr>
    <td><b>/Source:&lt;source&gt;</b></td>
    <td>指定要从中复制数据的源。源可以是文件系统目录、blob 容器、blob 虚拟目录、存储文件共享、存储文件目录或 Azure 表。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><b>/Dest:&lt;destination&gt;</b></td>
    <td>指定要复制到的目标。目标可以是文件系统目录、blob 容器、blob 虚拟目录、存储文件共享、存储文件目录或 Azure 表。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><b>/Pattern:&lt;file-pattern&gt;</b></td>
      <td>
          指定文件模式，它指示要复制哪些文件。/Pattern 参数的行为是由源数据的位置以及是否存在递归模式选项决定的。递归模式是通过选项 /S 指定的。
          <br />
          如果指定的源是文件系统中的一个目录，则标准通配符将生效，并且会将该目录中的文件与提供的文件模式进行匹配。如果指定了选项 /S，则 AzCopy 还会将该目录下的任何子文件夹中的所有文件与指定的模式进行匹配。
          <br />
          如果指定的源是一个 blob 容器或虚拟目录，则不会应用通配符。如果指定了选项 &nbsp;/S&nbsp;，则 AzCopy 会将指定的文件模式解释为 blob 前缀。如果未指定选项 &nbsp;/S&nbsp;，则 AzCopy 会将确切的 blob 名称与文件模式进行匹配。
          <br />
          如果指定的源是一个 Azure 文件共享，则你必须指定确切的文件名（例如 &nbsp;abc.txt）来复制单个文件，或者指定选项 &nbsp;/S&nbsp; 来以递归方式复制该共享中的所有文件。尝试同时指定文件模式和选项 /S&nbsp; 将导致错误。
          <br />
          未指定文件模式时使用的默认文件模式为 *.*（对于文件系统位置）或空前缀（对于  Azure 存储位置）。不支持指定多个文件模式。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>N</td>
  </tr>
  <tr>
    <td><b>/DestKey:&lt;storage-key&gt;</b></td>
    <td>指定目标资源的存储帐户密钥。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td class="auto-style1"><b>/DestSAS:&lt;sas-token&gt;</b></td>
    <td class="auto-style1">指定目标容器的共享访问签名 (SAS)（如果适用）。请将 SAS 用双引号引起来，因为它可能包含特殊的命令行字符。<br />
        如果目标资源是 blob 容器或表，则你可以指定此选项，后跟 SAS 令牌，也可以将 SAS 指定为目标 blob 的 URI 的一部分，而不使用此选项。<br />
        如果源和目标都是 blob，则目标 blob 必须与源 blob 位于同一个存储帐户中。</td>
    <td class="auto-style1">Y</td>
    <td class="auto-style1">N</td>
    <td class="auto-style1">Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><b>/SourceKey:&lt;storage-key&gt;</b></td>
    <td>指定源资源的存储帐户密钥。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><b>/SourceSAS:&lt;sas-token&gt;</b></td>
    <td>指定源容器的共享访问签名（如果适用）。请将 SAS 用双引号括起来，因为它可能包含特殊的命令行字符。
        <br />
        如果源资源是 blob 容器，并且既未提供密钥又未提供 SAS，则将可以通过匿名访问读取 blob 容器。
        <br />
        如果源是一个表，则必须提供密钥或 SAS。</td>
    <td>Y</td>
    <td>N</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><b>/S</b></td>
    <td>指定复制操作的递归模式。在递归模式下，AzCopy 将复制与指定的文件模式匹配的所有 blob 或文件，包括子文件夹中的那些文件。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>N</td>
  </tr>
  <tr>
    <td><b>/BlobType:&lt;block | page&gt;</b></td>
    <td>指定目标 blob 是块 blob 还是页 blob。此选项仅在上载 blob 时适用，其他情况下会生成错误。如果目标是一个 blob 并且未指定此选项，则默认情况下 AzCopy 将创建块 blob。</td>
    <td>Y</td>
    <td>N</td>
    <td>N</td>
  </tr>
  <tr>
    <td><b>/CheckMD5</b></td>
    <td>计算已下载的数据的 MD5 哈希，并验证存储在 blob 或文件的 Content-MD5 属性中的 MD5 哈希是否与计算得到的哈希匹配。默认情况下，MD5 检查处于关闭状态，因此，你必须指定此选项以在下载数据时执行 MD5 检查。
	<br />
    请注意，Azure 存储不保证为 blob 或文件存储的 MD5 哈希是最新的。每当 blob 或文件被修改时，都要对 MD5 进行更新，这是客户自己的责任。
	<br />
    在将 Azure blob 或文件上载到服务后，AzCopy 始终会为其设置 Content-MD5 属性。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>N</td>
  </tr>
  <tr>
    <td><b>/Snapshot</b></td>
    <td>指示是否传输快照。只有当源是 blob 时，此选项才有效。 
        <br />
        传输的 blob 快照将按以下格式重命名：[blob-name] (snapshot-time)[extension]. 
        <br />
        默认情况下，不会复制快照。</td>
    <td>Y</td>
    <td>N</td>
    <td>N</td>
  </tr>
  <tr>
    <td><b>/V:[verbose log-file]</b></td>
    <td>将详细的状态消息输出到日志文件中。默认情况下，详细日志文件被命名为 <code>AzCopyVerbose.log，</code> 位于 <code>%LocalAppData%\Microsoft\Azure\AzCopy</code> 中。如果你为此选项指定了现有的文件位置，则详细日志将追加到该文件中。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><b>/Z:[journal-file-folder]</b></td>
    <td>指定用于恢复某一操作的日志文件文件夹。<br />
        AzCopy 始终支持对被中断的操作进行恢复。<br />
        如果未指定此选项，或者指定它时未指定文件夹路径，则 AzCopy 将在默认位置中创建日志文件，默认位置为 <code>%LocalAppData%\Microsoft\Azure\AzCopy</code>。<br />
        每次向 AzCopy 发出命令时，它都会检查默认文件夹中是否存在日志文件，或者你通过此选项指定的文件夹中是否存在日志文件。如果这两个位置中都不存在日志文件，则 AzCopy 会将操作视为新操作并生成一个新的日志文件。
        <br />
		如果存在日志文件，则 AzCopy 将检查你输入的命令行是否与该日志文件中的命令行匹配。如果两个命令行匹配，则 AzCopy 将恢复未完成的操作。如果它们不匹配，则会提示你选择是覆盖该日志文件以启动新操作，还是取消当前操作。 
        <br />
        成功完成操作后，将删除该日志文件。
		<br />
		请注意，不支持通过由以前版本的 AzCopy 创建的日志文件来恢复操作。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><b>/@:parameter-file</b></td>
    <td>指定包含参数的文件。AzCopy 会像处理在命令行上指定的参数一样处理该文件中的参数。<br /> 
		在响应文件中，可以在单个行上指定多个参数，也可以将每个参数指定在其单独的行上。请注意，单个参数不能跨多个行。 
        <br />
		响应文件可以包括注释行，注释行以 <code>#</code> 符号开头。 
        <br />
        可以指定多个响应文件。但请注意，AzCopy 不支持嵌套的响应文件。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><b>/Y</b></td>
    <td>抑制所有 AzCopy 确认提示。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><b>/L</b></td>
    <td>仅指定列出操作；不复制数据。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>N</td>
  </tr>
  <tr>
    <td><b>/MT</b></td>
    <td>将下载的文件的上次修改时间设置为与源 blob 或文件的上次修改时间相同。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>N</td>
  </tr>
  <tr>
    <td><b>/XN</b></td>
    <td>排除较新的源资源。如果源比目标新，将不会复制该资源。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>N</td>
  </tr>
  <tr>
    <td><b>/XO</b></td>
    <td>排除较旧的源资源。如果源资源比目标旧，将不会复制该资源。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>N</td>
  </tr>
  <tr>
    <td><b>/A</b></td>
    <td>仅上载设置了存档属性的文件。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>N</td>
  </tr>
  <tr>
    <td><b>/IA:[RASHCNETOI]</b></td>
    <td>仅上载设置了任何指定属性的文件。<br />
        可用的属性包括：  
        <br />
        R&nbsp;&nbsp;&nbsp;只读文件
        <br />
        A&nbsp;&nbsp;&nbsp;可用于存档的文件
        <br />
        S&nbsp;&nbsp;&nbsp;系统文件
        <br />
        H&nbsp;&nbsp;&nbsp;隐藏的文件
        <br />
        C&nbsp;&nbsp;&nbsp;压缩的文件
        <br />
        N&nbsp;&nbsp;&nbsp;普通文件
        <br />
        E&nbsp;&nbsp;&nbsp;加密的文件
        <br />
        T&nbsp;&nbsp;&nbsp;临时文件
        <br />
        O&nbsp;&nbsp;&nbsp;脱机文件
        <br />
        I&nbsp;&nbsp;&nbsp;无索引的文件</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>N</td>
  </tr>
  <tr>
    <td><b>/XA:[RASHCNETOI]</b></td>
    <td>排除设置了任何指定属性的文件。<br />
        可用的属性包括：  
        <br />
        R&nbsp;&nbsp;&nbsp;只读文件  
        <br />
        A&nbsp;&nbsp;&nbsp;可用于存档的文件  
        <br />
        S&nbsp;&nbsp;&nbsp;系统文件  
        <br />
        H&nbsp;&nbsp;&nbsp;隐藏的文件  
        <br />
        C&nbsp;&nbsp;&nbsp;压缩的文件  
        <br />
        N&nbsp;&nbsp;&nbsp;普通文件  
        <br />
        E&nbsp;&nbsp;&nbsp;加密的文件  
        <br />
        T&nbsp;&nbsp;&nbsp;临时文件  
        <br />
        O&nbsp;&nbsp;&nbsp;脱机文件  
        <br />
        I&nbsp;&nbsp;&nbsp;无索引的文件</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>N</td>
  </tr>
  <tr>
    <td><b>/Delimiter:&lt;delimiter&gt;</b></td>
    <td>指示用于分隔 blob 名称中的虚拟目录的分隔符字符。<br />
        默认情况下，AzCopy 使用 / 作为分隔符字符。不过，AzCopy 支持使用任何常见字符（例如 @、# 或 %）作为分隔符。如果你需要在命令行上包括这些特殊字符之一，请将文件名用双引号引起来。 
        <br />
        此选项仅适用于下载 blob。</td>
    <td>Y</td>
    <td>N</td>
    <td>N</td>
  </tr>
  <tr>
    <td><b>/NC:&lt;number-of-concurrents&gt;</b></td>
    <td>指定并发操作的数量。
        <br />
        默认情况下，AzCopy 会启动一定数量的并发操作以提高数据传输吞吐量。请注意，在低带宽环境中，大量的并发操作可能会压垮网络连接，并且会阻碍操作彻底完成。请根据实际的可用网络带宽限制并发操作。
        <br />
		并发操作的上限为 512。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>N</td>
  </tr>
  <tr>
    <td><b>/SourceType:Blob|Table</b></td>
    <td>指定 <code>源</code> 资源是本地开发环境中可用的一个 blob，在存储仿真程序中运行。</td>
    <td>Y</td>
    <td>N</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><b>/DestType:Blob|Table</b></td>
    <td>指定 <code>目标</code> 资源是本地开发环境中可用的一个 blob，在存储仿真程序中运行。</td>
    <td>Y</td>
    <td>N</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><strong>/PKRS:&lt;&quot;key1#key2#key3#...&quot;&gt;</strong></td>
    <td>对分区键范围进行拆分以便并行导出表数据，这可以提高导出操作的速度。
        <br />
        如果未指定此选项，AzCopy 将使用单个线程来导出表实体。例如，如果用户指定了 /PKRS:&quot;aa#bb&quot;，则 AzCopy 将启动三个并发操作。
        <br />
        每个操作将导出三个分区键范围中的一个，如下所示： 
        <br />
        &nbsp;&nbsp;&nbsp;&#91;&lt;first partition key&gt;, aa&#41; 
        <br />
        &nbsp;&nbsp;&nbsp;&#91;aa, bb&#41;
        <br />
        &nbsp;&nbsp;&nbsp;&#91;bb, &lt;last partition key&gt;&#93; </td>
    <td>N</td>
    <td>N</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><strong>/SplitSize:</strong><file-size><strong>&lt;file-size&gt;</strong></td>
    <td>指定导出的文件的拆分大小 (MB)。
        <br />
        如果未指定此选项，则 AzCopy 会将表数据导出到单个文件。
        <br />
        如果将表数据导出到一个 blob，并且导出的文件的大小达到了 200 GB 的 blob 大小限制，则 AzCopy 将拆分导出的文件，即使未指定此选项也是如此。 </td>
    <td>N</td>
    <td>N</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><b>/EntityOperation:&lt;InsertOrSkip | InsertOrMerge | InsertOrReplace&gt;
</b>
</td>
    <td>指定表数据导入行为。
        <br />
        InsertOrSkip - 跳过现有实体，或者插入新实体（如果它不存在于表中）。
        <br />
        InsertOrMerge - 合并现有实体，或者插入新实体（如果它不存在于表中）。
        <br />
        InsertOrReplace - 替换现有实体，或者插入新实体（如果它不存在于表中）。 </td>
    <td>N</td>
    <td>N</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><b>/Manifest:&lt;manifest-file&gt;</b></td>
    <td>指定导入操作的清单文件。<br />
        清单文件是在执行导出操作期间生成的。</td>
    <td>N</td>
    <td>N</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
</table>

<br/>

##<a id="limit-writes"></a> 限制复制数据时的并发写入

在使用 AzCopy 复制 blob 或文件时，请记住，其他应用程序在你复制数据时可能正在修改该数据。如果可能，请确保你要复制的数据在执行复制操作期间不会被修改。例如，当复制与 Azure 虚拟机关联的 VHD 时，请确保当前没有其他应用程序正在向该 VHD 进行写入。另外，你还可以先创建 VHD 的快照，然后复制该快照。

如果你在复制 blob 或文件时无法阻止其他应用程序向其进行写入，请记住，在作业完成时，复制的资源可能不再与源资源完全相同。

##<a id="copy-blobs"></a> 使用 AzCopy 复制 Azure blob

下面的示例演示了使用 AzCopy 复制 blob 的各种方案。

### 复制单个 blob

**将文件从文件系统上载到 Blob 存储：**
	
	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /Pattern:abc.txt

**将 blob 从 Blob 存储下载到文件系统：**

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:key /Pattern:abc.txt

### 通过服务器端复制来复制 blob

当在 blob 存储帐户内或者跨存储帐户进行复制时，将执行服务器端复制操作。有关服务器端复制操作的更多信息，请参阅[跨帐户异步复制 Blob 简介](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx).

**在存储帐户内复制 blob：**

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer/mycontainer1 /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer2 /SourceKey:key /DestKey:key /Pattern:abc.txt 

**跨存储帐户复制 blob：**

	AzCopy /Source:https://sourceaccount.blob.core.chinacloudapi.cn/mycontainer/mycontainer1 /Dest:https://destaccount.blob.core.chinacloudapi.cn/mycontainer2 /SourceKey:key1 /DestKey:key2 /Pattern:abc.txt
 
### 从辅助区域复制 blob 

如果你的存储帐户启用了读取访问地域冗余存储，则可以从辅助区域复制数据。 

**将 blob 从主帐户复制到辅助帐户：**

	AzCopy /Source:https://myaccount1-secondary.blob.core.chinacloudapi.cn/mynewcontainer1 /Dest:https://myaccount2.blob.core.chinacloudapi.cn/mynewcontainer2 /SourceKey:key1 /DestKey:key2 /Pattern:abc.txt

**将辅助帐户中的 blob 下载到文件系统中的文件：**

	AzCopy /Source:https://myaccount-secondary.blob.core.chinacloudapi.cn/mynewcontainer /Dest:C:\myfolder /SourceKey:key /Pattern:abc.txt

### 将文件上载到新的 blob 容器或虚拟目录

**将文件上载到新的 blob 容器**

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mynewcontainer /DestKey:key /Pattern:abc.txt

请注意，如果指定的目标容器不存在，则 AzCopy 将创建它并将文件上载到其中。

**将文件上载到新的 blob 虚拟目录**

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer/vd /DestKey:key /Pattern:abc.txt

请注意，如果指定的虚拟目录不存在，则 AzCopy 将上载文件并在其名称中包括虚拟目录（*例如*，以上示例中的  `vd/abc.txt`）。

### 将 blob 下载到新文件夹

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:key /Pattern:abc.txt

如果文件夹  `C:\myfolder` 尚不存在，则 AzCopy 将在文件系统中创建它并将 `abc.txt` 下载到新文件夹中。

### 以递归方式将目录中的文件和子文件夹上载到容器

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /S

指定选项 `/S` 会以递归方式将指定目录的内容复制到 Blob 存储，这意味着还将复制所有子文件夹及其文件。例如，假定文件夹  `C:\myfolder` 中存在以下文件：

	C:\myfolder\abc.txt
	C:\myfolder\abc1.txt
	C:\myfolder\abc2.txt
	C:\myfolder\subfolder\a.txt
	C:\myfolder\subfolder\abcd.txt

在复制操作完成后，容器中将包括以下文件：

    abc.txt
    abc1.txt
    abc2.txt
    subfolder\a.txt
    subfolder\abcd.txt

### 以非递归方式将目录中的文件上载到容器

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key

如果未在命令行上指定选项 `/S`，则 AzCopy 将不会以递归方式进行复制。只会复制指定目录中的文件；不会复制任何子文件夹及其文件。例如，假定文件夹  `C:\myfolder` 中存在以下文件：

	C:\myfolder\abc.txt
	C:\myfolder\abc1.txt
	C:\myfolder\abc2.txt
	C:\myfolder\subfolder\a.txt
	C:\myfolder\subfolder\abcd.txt

在复制操作完成后，容器中将包括以下文件：

	abc.txt
	abc1.txt
	abc2.txt

### 以递归方式将某个容器中的所有 blob 下载到文件系统中的某个目录

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:key /S

假定指定的容器中存在以下 blob：  

	abc.txt
	abc1.txt
	abc2.txt
	vd1\a.txt
	vd1\abcd.txt

在复制操作完成后，目录  `C:\myfolder` 将包括以下文件：

	C:\myfolder\abc.txt
	C:\myfolder\abc1.txt
	C:\myfolder\abc2.txt
	C:\myfolder\vd1\a.txt
	C:\myfolder\vd1\abcd.txt

### 以递归方式将某个虚拟 blob 目录中的 blob 下载到文件系统中的某个目录

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer/vd1/ /Dest:C:\myfolder /SourceKey:key /S

假定指定的容器中存在以下 blob：

	abc.txt
	abc1.txt
	abc2.txt
	vd1\a.txt
	vd1\abcd.txt

在复制操作完成后，目录  `C:\myfolder` 将包括以下文件。请注意，仅复制了虚拟目录中的 blob：

	C:\myfolder\a.txt
	C:\myfolder\abcd.txt

### 以递归方式将与指定的文件模式匹配的文件上载到某个容器 

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /Pattern:a* /S

假定文件夹  `C:\myfolder` 中存在以下文件：

	C:\myfolder\abc.txt
	C:\myfolder\abc1.txt
	C:\myfolder\abc2.txt
	C:\myfolder\xyz.txt
	C:\myfolder\subfolder\a.txt
	C:\myfolder\subfolder\abcd.txt

在复制操作完成后，容器中将包括以下文件：

	abc.txt
	abc1.txt
	abc2.txt
	subfolder\a.txt
	subfolder\abcd.txt
	
### 以递归方式将具有指定前缀的 blob 下载到文件系统中

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:key /Pattern:a /S

假定指定的容器中存在以下 blob。以前缀  `a` will be copied: 开头的所有 blob

	abc.txt
	abc1.txt
	abc2.txt
	xyz.txt
	vd1\a.txt
	vd1\abcd.txt

在复制操作完成后，文件夹  `C:\myfolder` 将包括以下文件：

	C:\myfolder\abc.txt
	C:\myfolder\abc1.txt
	C:\myfolder\abc2.txt

请注意，前缀适用于虚拟目录，后者构成了 blob 名称的第一个部分。在上面所示的示例中，虚拟目录与指定的前缀不匹配，因此不会复制它。


### 将 blob 及其快照复制到其他存储帐户

	AzCopy /Source:https://sourceaccount.blob.core.chinacloudapi.cn/mycontainer/mycontainer1 /Dest:https://destaccount.blob.core.chinacloudapi.cn/mycontainer2 /SourceKey:key1 /DestKey:key2 /Pattern:abc.txt /Snapshot

在复制操作完成后，目标容器中将包括 blob 及其快照。假定上面的示例中的 blob 具有两个快照，则容器将包括以下 blob 和快照：

	abc.txt
	abc (2013-02-25 080757).txt
	abc (2014-02-21 150331).txt


### 使用响应文件指定命令行参数

	AzCopy /@:"C:\myfolder\abc.txt"

可以在响应文件中包括任何 AzCopy 命令行参数。AzCopy 会像处理在命令行上指定的参数一样处理该文件中的参数，使用该文件的内容执行直接替换。

**指定一个或多个单行响应文件**

假定有一个名为  `source.txt` 的响应文件指定了一个源容器：

	/Source:http://myaccount.blob.core.chinacloudapi.cn/mycontainer

And 有一个名为  `dest.txt` 的响应文件指定了文件系统中的一个目标文件夹：

	/Dest:C:\myfolder

并且有一个名为  `options.txt` 的响应文件指定了 AzCopy 的选项：

	/S /Y

若要在使用这些响应文件（它们都位于目录  `C:\responsefiles` 中）的情况下调用 AzCopy，请使用以下命令：

	AzCopy /@:"C:\responsefiles\source.txt" /@:"C:\responsefiles\dest.txt" /SourceKey:<sourcekey> /@:"C:\responsefiles\options.txt"   

AzCopy 会像在命令行上包括了所有个体参数一样来处理此命令：

	AzCopy /Source:http://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:<sourcekey> /S /Y

**指定多行响应文件**

假定有一个名为  `copyoperation.txt` 的响应文件，包含以下行。每个 AzCopy 参数都是在其各自的行上指定的：

	/Source:http://myaccount.blob.core.chinacloudapi.cn/mycontainer
	/Dest:C:\myfolder
	/SourceKey:<sourcekey>
	/S 
	/Y

若要在使用此响应文件的情况下调用 AzCopy，请使用以下命令：

	AzCopy /@:"C:\responsefiles\copyoperation.txt"

AzCopy 会像在命令行上包括了所有个体参数一样来处理此命令：	

	AzCopy /Source:http://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:<sourcekey> /S /Y

请注意，每个 AzCopy 参数都必须指定在一个行上。例如，如果你将参数拆分到两个行上（如下面所示的 `/sourcekey`/ 参数），则 AzCopy 将会失败：

	http://myaccount.blob.core.chinacloudapi.cn/mycontainer
 	C:\myfolder
	/sourcekey:
	[sourcekey]
	/S 
	/Y

### 指定共享访问签名 (SAS)
	
**使用 /sourceSAS 选项指定源容器的 SAS**

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer1 /DestC:\myfolder /SourceSAS:SAS /S

**在源容器 URI 上指定源容器的 SAS**

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer1/?SourceSASToken /Dest:C:\myfolder /S

**使用 /destSAS 选项指定目标容器的 SAS**

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer1 /DestSAS:SAS /Pattern:abc.txt

**指定源容器和目标容器的 SAS**

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer1 /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer2 /SourceSAS:SAS1 /DestSAS:SAS2 /Pattern:abc.txt

### 指定日志文件文件夹

每次向 AzCopy 发出命令时，它都会检查默认文件夹中是否存在日志文件，或者你通过此选项指定的文件夹中是否存在日志文件。如果这两个位置中都不存在日志文件，则 AzCopy 会将操作视为新操作并生成一个新的日志文件。

如果存在日志文件，则 AzCopy 将检查你输入的命令行是否与该日志文件中的命令行匹配。如果两个命令行匹配，则 AzCopy 将恢复未完成的操作。如果它们不匹配，则会提示你选择是覆盖该日志文件以启动新操作，还是取消当前操作。 

**为日志文件使用默认位置**

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /Z

如果省略了选项 `/Z` 或者指定了选项 `/Z` 但未指定文件夹路径（如上所示），则 AzCopy 将在默认位置中创建日志文件，默认位置为 `%SystemDrive%\Users\%username%\AppData\Local\Microsoft\Azure\AzCopy`。如果日志文件已存在，则 AzCopy 将继续根据日志文件恢复操作。 

**为日志文件指定自定义位置**

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /Z:C:\journalfolder\

如果日志文件尚不存在，此示例将创建日志文件。如果它已存在，则 AzCopy 将根据该日志文件恢复操作。

**恢复 AzCopy 操作**

	AzCopy /Z:C:\journalfolder\

此示例将恢复可能没有完成的上一操作。


### 生成日志文件

**写入到默认位置中的详细日志文件**

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /V

如果指定了选项 `/V` 但未提供详细日志的文件路径，则 AzCopy 将在默认位置中创建日志文件，默认位置为 `%SystemDrive%\Users\%username%\AppData\Local\Microsoft\Azure\AzCopy`。

**写入到自定义位置中的详细日志文件**

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /V:C:\myfolder\azcopy1.log

请注意，如果在选项 `/V` 后指定了相对路径，例如 `/V:test/azcopy1.log`，则会在当前工作目录中名为  `test` 的子文件夹内创建详细日志。


### 将下载的文件的上次修改时间设置为与源 blob 相同

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:key /MT

### 根据 blob 的上次修改时间将其从复制操作中排除

指定 `/MT` 选项来比较源 blob 与目标文件的上次修改时间。

**排除比目标文件新的 blob**

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:key /MT /XN

**排除比目标文件旧的 blob**

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:key /MT /XO

### 指定要启动的并发操作的数量

选项 `/NC` 指定并发复制操作的数量。默认情况下，AzCopy 将启动八倍于你具有的核心处理器数量的并发操作。如果你正在低带宽网络中运行 AzCopy，可以为此选项指定较低的数量以避免由于资源争用导致的故障。


###	针对存储仿真程序中的 blob 资源运行 AzCopy

	AzCopy /Source:https://127.0.0.1:10004/myaccount/myfileshare/ /Dest:C:\myfolder /SourceKey:key /SourceType:Blob /S


##<a id="copy-files"></a> 使用 AzCopy 复制 Azure 文件存储中的文件（仅限预览版本）

下面的示例演示了使用 AzCopy 复制 Azure 文件的各种方案。

### 将文件从 Azure 文件共享下载到文件系统

	AzCopy /Source:https://myaccount.file.core.chinacloudapi.cn/myfileshare/myfolder1/ /Dest:C:\myfolder /SourceKey:key /Pattern:abc.txt

如果指定的源是一个 Azure 文件共享，则你必须指定确切的文件名（*例如*  `abc.txt`）来复制单个文件，或者指定选项 `/S` 来以递归方式复制该共享中的所有文件。尝试同时指定文件模式和选项 `/S` 将导致错误。

### 以递归方式将 Azure 文件共享中的文件和文件夹下载到文件系统

	AzCopy /Source:https://myaccount.file.core.chinacloudapi.cn/myfileshare/ /Dest:C:\myfolder /SourceKey:key /S

请注意，不会复制任何空文件夹。


### 以递归方式将文件系统中的文件和文件夹上载到 Azure 文件共享

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.file.core.chinacloudapi.cn/myfileshare/ /DestKey:key /S

请注意，不会复制任何空文件夹。


### 以递归方式将与指定的文件模式匹配的文件上载到某个 Azure 文件共享

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.file.core.chinacloudapi.cn/myfileshare/ /DestKey:key /Pattern:ab* /S

##<a id="copy-entities"></a> 使用 AzCopy 复制 Azure 表中的文件（仅限预览版本）

下面的示例演示了使用 AzCopy 复制 Azure 表实体的各种方案。

### 将实体导出到本地文件系统

	AzCopy /Source:https://myaccount.table.core.chinacloudapi.cn/myTable/ /Dest:D:\test\ /SourceKey:key

### 将实体导出到 Azure blob

	AzCopy /Source:https://myaccount.table.core.chinacloudapi.cn/myTable/ /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer/ /SourceKey:key1 /Destkey:key2

AzCopy 将使用以下命名约定在本地文件夹或 blob 容器中生成一个 JSON 数据文件：

	<account name>_<table name>_<timestamp>_<volume index>_<CRC>.json

生成的 JSON 数据文件遵循最少元数据负载格式。有关此负载格式的详细信息，请参阅[表服务操作的负载格式](http://msdn.microsoft.com/library/azure/dn535600.aspx).

### 拆分导出文件

	AzCopy /Source:https://myaccount.file.core.chinacloudapi.cn/myfileshare/ /Dest:C:\myfolder /SourceKey:key /S /SplitSize:100

AzCopy 在已拆分数据文件名称中使用卷索引 ( *volume index*) 来区分多个文件。卷索引包括两个部分：分区键范围索引 ( *partition key range index*) 和已拆分文件索引 ( *split file index*)。两个索引都是从零开始的。

如果用户未指定选项 `/PKRS`（将在下节进行介绍），则分区键范围索引将是 0。

例如，假设 AzCopy 在用户指定选项 `/SplitSize` 后生成了两个数据文件。生成的数据文件名称可能是：

	myaccount_mytable_20140903T051850.8128447Z_0_0_C3040FE8.json
	myaccount_mytable_20140903T051850.8128447Z_0_1_0AB9AC20.json

请注意，选项 `/SplitSize` 的最小可能值为 32 MB。如果指定的目标是 Blob 存储，不管用户是否指定了选项 `/SplitSize`，AzCopy 将在数据文件的大小达到 blob 大小限制 (200GB) 时拆分数据文件。

### 并发导出实体

	AzCopy /Source:https://myaccount.table.core.chinacloudapi.cn/myTable/ /Dest:D:\test\ /SourceKey:key /PKRS:"aa#bb"

当用户指定了选项 `/PKRS` 时，AzCopy 将启动并发操作来导出实体。每个操作导出一个分区键范围。

请注意，并发操作的数量还受选项 `/NC` 控制。当复制表实体时，AzCopy 使用核心处理器的数量作为 `/NC` 的默认值，即使未指定 `/NC` 也是如此。当用户指定了选项 `/PKRS` 时，AzCopy 将使用以下两个值中的较小者（分区键范围、隐式或显式指定的并发操作数量）来确定要启动的并发操作的数量。若要获得更多详细信息，请在命令行上键入  `AzCopy /?:NC`。

### 并发导入实体

	AzCopy /Source:D:\test\ /Dest:https://myaccount.table.core.chinacloudapi.cn/mytable1/ /DestKey:key /Manifest:"myaccount_mytable_20140103T112020.manifest" /EntityOperation:InsertOrReplace 

导出表实体时，AzCopy 会将一个清单文件写入到指定的目标文件夹或 blob 容器。导入进程在导入过程中使用该清单文件来定位必要的数据文件并执行数据验证。清单文件使用以下命名约定：

	<account name>_<table name>_<timestamp>.manifest

选项 `/EntityOperation` 指示如何将实体插入表中。可能的值包括：

- `InsertOrSkip`：跳过现有实体，或者插入新实体（如果它不存在于表中）。
- `InsertOrMerge`:合并现有实体，或者插入新实体（如果它不存在于表中）。
- `InsertOrReplace`：替换现有实体，或者插入新实体（如果它不存在于表中）。

请注意，在导入方案中不能指定选项 `/PKRS`。与导出方案不同（在导出方案中必须指定 `/PKRS` 选项才会启动并发操作），在导入实体时，AzCopy 将默认启动并发操作。启动的并发操作的默认数量等于核心处理器的数量；不过，你可以通过选项 `/NC` 指定一个不同的并发数量。若要获得更多详细信息，请在命令行上键入  `AzCopy /?:NC`。


##<a id="versions"></a> AzCopy 版本

| 版本 | 新增功能                                                                                      				|
|---------|-----------------------------------------------------------------------------------------------------------------|
| **V4.0.0**  | **当前的预览版本。包括 V3.0.0 中的所有功能。还支持将文件复制到 Azure 文件存储或从其中复制文件，以及将实体复制到 Azure 表存储或从其中复制实体。**	
| **V3.0.0**  | **当前的发行版本。将 AzCopy 命令行语法修改为需要参数名称，并且重新设计了命令行帮助。此版本仅支持复制到 Azure Blob 存储和从其中进行复制。**	
| V2.5.1  | 优化了使用选项 /xo 和 /xn 时的性能。修复了与源文件名称中的特殊字符以及在用户输入错误的命令行语法后导致的日志文件损坏相关的 bug。	
| V2.5.0  | 优化了大规模复制方案的性能，并执行了几处重要的可用性改进。	
| V2.4.1  | 支持在安装向导中指定目标文件夹。                     			
| V2.4.0  | 支持为 Azure 文件存储上载和下载文件。                       				                              
| V2.3.0  | 支持读取访问地域冗余存储帐户。                                                  			|
| V2.2.2  | 升级为使用 Azure 存储客户端库 3.0.3 版。                                            				                    
| V2.2.1  | 修复了在同一存储帐户内复制大量文件时的性能问题。            				                                                
| V2.2    | 支持为 blob 名称设置虚拟目录分隔符。支持指定日志文件路径。		|
| V2.1    | 提供了 20 多个选项来支持以高效的方式执行 blob 上载、下载和复制操作。		|


##<a id="next-steps"></a> 后续步骤

有关 Azure 存储和 AzCopy 的更多信息，请参阅以下资源。

### Azure 存储文档：

- [Azure 存储简介](/zh-cn/documentation/articles/storage-introduction/)
- [将文件存储在 Blob 存储中](/zh-cn/documentation/articles/storage-dotnet-how-to-use-blobs/)
- [在具有文件存储的 Azure 中创建 SMB 文件共享](/zh-cn/documentation/articles/storage-dotnet-how-to-use-files/)

### Azure 存储博客文章：

- [AzCopy 3.0：支持表和文件的 AzCopy 3.0 增强预览版本 AzCopy 4.0 宣布公开发行](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/10/29/azcopy-announcing-general-availability-of-azcopy-3-0-plus-preview-release-of-azcopy-4-0-with-table-and-file-support.aspx)
- [AzCopy 2.5：针对大规模复制方案进行了优化](http://go.microsoft.com/fwlink/?LinkId=507682)
- [Microsoft Azure 文件服务介绍](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/12/introducing-microsoft-azure-file-service.aspx)
- [AzCopy：对读取访问地域冗余存储的支持](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/04/07/azcopy-support-for-read-access-geo-redundant-account.aspx)
- [AzCopy：使用可重新启动的模式和 SAS 令牌传输数据](http://blogs.msdn.com/b/windowsazurestorage/archive/2013/09/07/azcopy-transfer-data-with-re-startable-mode-and-sas-token.aspx)
- [AzCopy：使用跨帐户复制 Blob](http://blogs.msdn.com/b/windowsazurestorage/archive/2013/04/01/azcopy-using-cross-account-copy-blob.aspx)
- [AzCopy：为 Microsoft Azure Blob 上载/下载文件](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/12/03/azcopy-uploading-downloading-files-for-windows-azure-blobs.aspx)

<!--HONumber=41-->
