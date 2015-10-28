<properties 
	pageTitle="如何将 AzCopy 与 Windows Azure 存储一起使用" 
	description="了解如何使用 AzCopy 实用程序上载、下载以及复制 blob 和文件内容。" 
	services="storage" 
	documentationCenter="" 
	authors="tamram" 
	manager="adinah" 
	editor="cgronlun"/>

<tags 
	ms.service="storage" 
	ms.date="06/22/2015" 
	wacn.date="09/18/2015"/>


# AzCopy 命令行实用程序入门

## 概述

AzCopy 是一个高性能的命令行实用程序，用于将数据上载、复制到 Windows Azure二进制存储 (Blob)、文件存储 (File) 和表存储 (Table) 以及从其中下载和复制数据。本指南提供了有关使用 AzCopy 的概述。

> [AZURE.NOTE] 本指南假定你已安装了 AzCopy 3.2.0 或更高版本。AzCopy 3.x 现在为公开发行版本。 
> 本指南还包括如何使用 AzCopy 4.2.0，即 AzCopy 的预览版本。在本指南中，只有在预览版本中才提供的功能标注为*预览*。
> 注意，对于 AzCopy 4.x，命令行选项和功能在将来的版本中可能会改变。

## 下载并安装 AzCopy

1. 下载[最新版 AzCopy](http://aka.ms/downloadazcopy)，或[最新预览版本](http://aka.ms/downloadazcopypr)。
2. 运行安装。默认情况下，AzCopy 安装会在 `%ProgramFiles(x86)%\Microsoft SDKs\Azure`（运行 64 位 Windows 的计算机）或 `%ProgramFiles%\Microsoft SDKs\Azure`（运行 32 位 Windows 的计算机）下创建一个名为 `AzCopy` 的文件夹。不过，你可以通过安装向导更改安装路径。
3. 如果需要，你可以将 AzCopy 安装路径添加到系统路径。

## 了解 AzCopy 命令行语法

接下来，打开一个命令窗口，然后导航到计算机上的 AzCopy 安装目录，该位置存放着可执行的 `AzCopy.exe`。AzCopy 命令的基本语法是：

	AzCopy /Source:<source> /Dest:<destination> /Pattern:<filepattern> [Options]

> [WACOM.NOTE] 从 AzCopy 3.0.0 版开始，AzCopy 命令行语法要求在指定每个参数时都包括参数名称，例如 `/ParameterName:ParameterValue`。

## 编写你的第一条 AzCopy 命令

**将文件从文件系统上载到 Blob 存储：**
	
	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /Pattern:abc.txt

请注意，当复制单个文件，请用选项 /Pattern 指定文件名称。你可以在本文的后面部分中找到更多示例。

## 参数介绍

下表中描述了 AzCopy 的参数。你还可以从命令行键入下列命令之一来获得有关如何使用 AzCopy 的帮助信息：

- 若要获得 AzCopy 的详细命令行帮助信息，请键入：`AzCopy /?`
- 若要获得任何 AzCopy 参数的详细帮助信息，请键入： `AzCopy /?:SourceKey`
- 若要获得命令行示例，请键入： `AzCopy /?:Samples` 


<table>
  <tr>
    <th>选项名称</th>
    <th>说明</th>
    <th>适用于 Blob 存储 (Y/N)</th>
    <th>适用于文件存储 (Y/N)（仅限预览版本）</th>
    <th>适用于表存储 (Y/N)（仅限预览版本）</th>
  </tr>
  <tr>
    <td><b>/Source:&lt;source></b></td>
    <td>指定要从中复制数据的源。源可以是文件系统目录、blob 容器、blob 虚拟目录、文件共享 (File Share)、文件共享目录 (File Share Directory) 或 Azure 表。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><b>/Dest:&lt;destination></b></td>
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
          当 /Source 是 blob 容器或 blob 虚拟目录时，AzCopy 使用区分大小写匹配，并在所有其他情况下使用不区分大小写匹配。
          <br/>
          未指定文件模式时使用的默认文件模式为 *.*（对于文件系统位置）或空前缀（对于 Azure 存储位置）。不支持指定多个文件模式。</td>
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
    <td class="auto-style1">指定对目标具有读写权限的共享访问签名 (SAS)（如果适用）。请将 SAS 用双引号括起来，因为它可能包含特殊的命令行字符。<br />
        如果目标资源是 blob 容器、文件共享或表，你可以指定此选项，后跟 SAS 令牌，也可以将 SAS 指定为目标 blob 容器、文件共享或表 URI 的一部分，而不使用此选项。<br />
        如果源和目标都是 blob，则目标 blob 必须与源 blob 位于同一个存储帐户中。</td>
    <td class="auto-style1">Y</td>
    <td class="auto-style1">Y<br /> （仅限预览版本）</td>
    <td class="auto-style1">Y<br /> （仅限预览版本）</td>
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
    <td>指定对源具有读取和列表权限的共享访问签名（如果适用）。请将 SAS 用双引号括起来，因为它可能包含特殊的命令行字符。
        <br />
        如果源资源是 blob 容器，并且既未提供密钥又未提供 SAS，则将可以通过匿名访问读取 blob 容器。
        <br />
        如果源是文件共享或表，必须提供一个键或某一 SAS。</td>
    <td>Y</td>
    <td>Y<br /> （仅限预览版本）</td>
    <td>Y<br /> （仅限预览版本）</td>
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
    <td>指定目标 blob 是块 blob，页 blob 还是追加 blob。此选项仅在上载 blob 时适用，其他情况下会生成错误。如果目标是一个 blob 并且未指定此选项，则默认情况下 AzCopy 将创建块存储(block blob)。</td>
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
        传输的 blob 快照将按以下格式重命名：[blob-name](/documentation/articles/snapshot-time)[extension]。
        <br />
        默认情况下，不会复制快照。</td>
    <td>Y</td>
    <td>N</td>
    <td>N</td>
  </tr>
  <tr>
    <td><b>/V:[verbose 日志-文件]</b></td>
    <td>将详细的状态消息输出到日志文件中。默认情况下，详细的日志文件名为 <code>AzCopyVerbose.log</code> 位于 <code>%LocalAppData%\Microsoft\Azure\AzCopy</code>。如果你为此选项指定了现有的文件位置，则详细日志将追加到该文件中。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><b>/Z:[journal-file-folder]</b></td>
    <td>指定用于恢复某一操作的日志文件文件夹。<br />
        AzCopy 始终支持对被中断的操作进行恢复。<br />
        如果未指定此选项，或者未指定其文件夹路径，则 AzCopy 会在默认位置中创建日志文件，默认位置为<code>%LocalAppData%\Microsoft\Azure\AzCopy</code>。<br />
        每次向 AzCopy 发出命令时，它都会检查默认文件夹中是否存在恢复日志，或者你通过此选项指定的文件夹中是否存在恢复日志。如果这两个位置中都不存在恢复日志，则 AzCopy 会将操作视为新操作并生成一个新的恢复日志。
        <br />
		如果存在恢复日志，则 AzCopy 将检查你输入的命令行是否与该恢复日志中的命令行匹配。如果两个命令行匹配，则 AzCopy 将恢复未完成的操作。如果它们不匹配，则会提示你选择是覆盖该恢复日志以启动新操作，还是取消当前操作。 
        <br />
        成功完成操作后，将删除该恢复日志。
		<br />
		请注意，不支持通过由以前版本的 AzCopy 创建的恢复日志来恢复操作。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><b>/@:parameter-file</b></td>
    <td>指定包含参数的文件。AzCopy 会像处理在命令行上指定参数一样处理文件中的参数。<br /> 
		在响应文件中，可以在单个行上指定多个参数，也可以将每个参数指定在其单独的行上。请注意，单个参数不能跨多个行。
        <br />
		响应文件可包括以 <code>#</code> 标志开头的注释行。
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
    <td>仅指定列出操作；不复制数据。
    <br />
    AzCopy 对该选项的使用解释为对无选项 /L 运行命令行的模拟，并对复制的数量进行计数，你可以同时指定选项 /V 来检查哪些对象将复制到详细日志中。
    <br />
    该选项的行为也由源数据的位置、是否存在递归模式选项 /S 以及文件模式选项 /Pattern 决定。
    <br />
    使用此选项时，AzCopy 需要此源位置的列和读权限。</td>
    <td>Y</td>
    <td>Y<br />（仅限预览版本）</td>
    <td>N</td>
  </tr>
  <tr>
    <td><b>/MT</b></td>
    <td>将下载的文件的最后修改时间设置为与源 blob 或 file 的最后修改时间相同。</td>
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
        H&nbsp;&nbsp;&nbsp;隐藏文件
        <br />
        C&nbsp;&nbsp;&nbsp;压缩文件
        <br />
        N&nbsp;&nbsp;&nbsp;普通文件
        <br />
        E&nbsp;&nbsp;&nbsp;加密文件
        <br />
        T&nbsp;&nbsp;&nbsp;临时文件
        <br />
        O&nbsp;&nbsp;&nbsp;脱机文件
        <br />
        I&nbsp;&nbsp;&nbsp;未编制索引的文件</td>
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
        H&nbsp;&nbsp;&nbsp;隐藏文件  
        <br />
        C&nbsp;&nbsp;&nbsp;压缩文件  
        <br />
        N&nbsp;&nbsp;&nbsp;普通文件  
        <br />
        E&nbsp;&nbsp;&nbsp;加密文件  
        <br />
        T&nbsp;&nbsp;&nbsp;临时文件  
        <br />
        O&nbsp;&nbsp;&nbsp;脱机文件  
        <br />
        I&nbsp;&nbsp;&nbsp;未编制索引的文件</td>
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
    <td>Y<br /> （仅限预览版本）</td>
    <td>Y<br /> （仅限预览版本）</td>
  </tr>
  <tr>
    <td><b>/SourceType:Blob|Table</b></td>
    <td>指定<code>源</code>资源是本地开发环境中可用的一个 blob，在存储模拟器中运行。</td>
    <td>Y</td>
    <td>N</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><b>/DestType:Blob|Table</b></td>
    <td>指定<code>目标</code>资源是本地开发环境中可用的一个 blob，在存储模拟器中运行。</td>
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
        &#160;&#160;&#160;[&lt;第一个分区键>, aa) 
        <br />
        &#160;&#160;&#160;[aa, bb)
        <br />
        &#160;&#160;&#160;[bb, &lt;最后一个分区键>] </td>
    <td>N</td>
    <td>N</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><strong>/SplitSize:</strong><file-size><strong>&lt;file-size></strong></td>
    <td>指定导出的文件拆分大小（单位为 MB），允许的最小值为 32。
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
    <td><b>/Manifest:&lt;manifest-file></b></td>
    <td>指定表导出导入操作的清单文件。<br />
    此选项在导出操作过程是可选的，如果未指定此选项，AzCopy 将生成具有预定义名称的清单文件。
    <br />
    在用于定位数据文件的导入操作过程中，此选项是必要的。</td>
    <td>N</td>
    <td>N</td>
    <td>Y<br />（仅限预览版本）</td>
  </tr>
  <tr>
    <td><b>/SyncCopy</b></td>
    <td>指示是否要以同步方式复制两个 Azure 存储结点之间的 blob 或 file。<br />
		AzCopy 默认情况下使用服务器端的异步复制。指定此选项以执行同步复制，可将 blob 或文件下载到本地内存，然后将其上载到 Azure 存储空间。可以在以下情况使用该选项：在Blob 结点之间复制文件、file结点之间复制文件，从 blob 存储向file存储复制文件以及从file存储向blob存储复制文件。</td>
    <td>Y</td>
    <td>Y<br /> （仅限预览版本）</td>
    <td>N</td>
  </tr>
  <tr>
    <td><b>/SetContentType:&lt;content-type></b></td>
    <td>指定目标 blob 或 file 的 MIME 内容类型。<br />
		默认情况下，AzCopy 将 blob 或 file 的内容类型设置为<code>application/octet-stream</code>。通过显式指定此选项的值，可设置所有 blob 或 file 的内容类型。如果指定此选项不带值，AzCopy 将根据文件扩展名设置每个 blob 或 file 的内容类型。</td>
    <td>Y</td>
    <td>Y<br /> （仅限预览版本）</td>
    <td>N</td>
  </tr>
    <tr>
    <td><b>/PayloadFormat:&lt;JSON | CSV></b></td>
    <td>指定表导出数据文件的格式。<br />
    如果未指定此选项，则默认情况下，AzCopy 导出 JSON 格式的表数据文件。</td>
    <td>N</td>
    <td>N</td>
    <td>Y<br /> （仅限预览版本）</td>
  </tr>
</table>

<br/>

## 限制复制数据时的并发写入

在使用 AzCopy 复制 blob 或文件时，请记住，其他应用程序在你复制数据时可能正在修改该数据。如果可能，请确保你要复制的数据在执行复制操作期间不会被修改。例如，当复制与 Azure 虚拟机关联的 VHD 时，请确保当前没有其他应用程序正在向该 VHD 进行写入。另外，你还可以先创建 VHD 的快照，然后复制该快照。

如果你在复制 blob 或 file 时无法阻止其他应用程序向其进行写入，请记住，在作业完成时，复制的资源可能不再与源资源完全相同。

## 使用 AzCopy 复制 Azure blob

下面的示例演示了使用 AzCopy 复制 blob 的各种方案。

### 复制单个 blob

**将文件从文件系统上载到 Blob 存储：**
	
	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /Pattern:abc.txt

**将 blob 从 Blob 存储下载到文件系统：**

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:key /Pattern:abc.txt

有关使用存储访问密钥的详细信息，请参阅[查看、复制和重新生成存储访问密钥](/documentation/articles/storage-create-storage-account#regeneratestoragekeys)。

### 通过服务器端复制来复制 blob

当在 blob 存储帐户内或者跨存储帐户进行复制时，将执行服务器端复制操作。有关服务器端复制操作的详细信息，请参阅[异步跨帐户复制 Blob 简介](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-asynchronous-cross-account-copy-blob.aspx)。

**在存储帐户内复制 blob：**

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer1 /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer2 /SourceKey:key /DestKey:key /Pattern:abc.txt 

**跨存储帐户复制 blob：**

	AzCopy /Source:https://sourceaccount.blob.core.chinacloudapi.cn/mycontainer1 /Dest:https://destaccount.blob.core.chinacloudapi.cn/mycontainer2 /SourceKey:key1 /DestKey:key2 /Pattern:abc.txt
 
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

请注意，如果指定的虚拟目录不存在，则 AzCopy 将上载文件并在其名称中包括虚拟目录（*例如*上例中的 `vd/abc.txt`）

### 将 blob 下载到新文件夹

	AzCopy /Source:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /Dest:C:\myfolder /SourceKey:key /Pattern:abc.txt

如果文件夹 `C:\myfolder` 尚不存在，AzCopy 会在文件系统中创建并将 `abc.txt ` 下载到新文件夹中。

### 以递归方式将目录中的文件和子文件夹上载到容器

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /S

指定选项 `/S` 会以递归方式将指定目录的内容复制到 Blob 存储空间，这意味着还将复制所有子文件夹及其文件。例如，假定以下文件存在于文件夹 `C:\myfolder` 中：

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

如果未在命令行上指定选项 `/S`，AzCopy 将不会以递归方式进行复制。只会复制指定目录中的文件；不会复制任何子文件夹及其文件。例如，假定以下文件存在于文件夹 `C:\myfolder` 中：

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

假定以下文件存在于文件夹 `C:\myfolder` 中：

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

假定指定的容器中存在以下 blob。将复制所有以前缀 `a` 开头的 blob：

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

	AzCopy /Source:https://sourceaccount.blob.core.chinacloudapi.cn/mycontainer1 /Dest:https://destaccount.blob.core.chinacloudapi.cn/mycontainer2 /SourceKey:key1 /DestKey:key2 /Pattern:abc.txt /Snapshot

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

有一个名为 `dest.txt` 的响应文件在文件系统中指定了一个目标文件夹：

	/Dest:C:\myfolder

并且有一个名为  `options.txt` 的响应文件指定了 AzCopy 的选项：

	/S /Y

要使用这些响应文件（都位于目录 `C:\responsefiles` 中）调用 AzCopy，请使用以下命令：

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

请注意，每个 AzCopy 参数都必须指定在一个行上。例如，如果你将参数拆分到两行（如下面所示的 `/sourcekey` 参数），AzCopy 将会失败：

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

### 指定恢复日志文件夹

每次向 AzCopy 发出命令时，它都会检查默认文件夹中是否存在恢复日志，或者你通过此选项指定的文件夹中是否存在恢复日志。如果这两个位置中都不存在恢复日志，则 AzCopy 会将操作视为新操作并生成一个新的恢复日志。

如果存在恢复日志，则 AzCopy 将检查你输入的命令行是否与该日志文件中的命令行匹配。如果两个命令行匹配，则 AzCopy 将恢复未完成的操作。如果它们不匹配，则会提示你选择是覆盖该恢复日志以启动新操作，还是取消当前操作。 

**为恢复日志使用默认位置**

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /Z

如果省略了选项 `/Z`，或者指定了选项 `/Z` 但未指定文件夹路径（如上所示），则 AzCopy 在默认位置创建恢复日志，默认位置为 `%SystemDrive%\Users\%username%\AppData\Local\Microsoft\Azure\AzCopy`。如果恢复日志已存在，则 AzCopy 将继续根据恢复日志恢复操作。

**为恢复日志指定自定义位置**

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer /DestKey:key /Z:C:\journalfolder\

如果恢复日志尚不存在，此示例将创建恢复日志。如果它已存在，则 AzCopy 将根据该恢复日志恢复操作。

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

### 以同步方式复制两个 Azure 存储结点之间的 blob。

默认情况下，AzCopy 以异步方式复制两个存储终结点之间的数据。因此，复制操作使用空闲带宽容量运行在后台，没有规定 blob 复制速率的 SLA，AzCopy 将定期检查复制状态直到复制完成或失败。

`/SyncCopy` 选项确保复制操作的速度一致。AzCopy 通过下载 blob，将 blob 从指定的源复制到本地内存，然后上载该 blob 至 Blob 存储目标，以实现同步复制。

	AzCopy /Source:https://myaccount1.blob.core.chinacloudapi.cn/myContainer/ /Dest:https://myaccount2.blob.core.chinacloudapi.cn/myContainer/ /SourceKey:key1 /DestKey:key2 /Pattern:ab /SyncCopy

请注意，与异步复制相比，`/SyncCopy` 可能会产生额外流出成本，推荐方法是在与你的源存储账户处于同一区域的 Azure 虚拟机中使用该选项，以避免流出成本。

### 指定目标 blob 的 MIME 内容类型

默认情况下，AzCopy 将目标 blob 的内容类型设置为 `application/octet-stream`。从 3.1.0 版开始，可以明确通过选项 `/SetContentType:[content-type]` 指定内容类型。此语法在复制操作中设置所有 Blob 的内容类型。

	AzCopy /Source:C:\myfolder\ /Dest:https://myaccount.blob.core.chinacloudapi.cn/myContainer/ /DestKey:key /Pattern:ab /SetContentType:video/mp4

如果指定选项 `/SetContentType` 不带值，AzCopy 将根据文件扩展名设置每个 Blob 或文件的内容类型。

	AzCopy /Source:C:\myfolder\ /Dest:https://myaccount.blob.core.chinacloudapi.cn/myContainer/ /DestKey:key /Pattern:ab /SetContentType

## 使用 AzCopy 复制 Azure File 存储中的文件（仅限预览版本）

下面的示例演示了使用 AzCopy 复制 Azure 文件的各种方案。

### 将文件从 Azure File Share下载到文件系统

	AzCopy /Source:https://myaccount.file.core.chinacloudapi.cn/myfileshare/myfolder1/ /Dest:C:\myfolder /SourceKey:key /Pattern:abc.txt

请注意，如果指定的源是 Azure File Share，你必须指定确切的文件名（*例如*`abc.txt` ）来复制单个文件，或者指定选项 `/S` 来以递归方式复制该共享中的所有文件。尝试同时指定文件模式和选项 `/S` 将导致错误。

### 以递归方式将 Azure File Share中的文件和文件夹下载到文件系统中，指定共享访问签名

	AzCopy /Source:https://myaccount.file.core.chinacloudapi.cn/myfileshare/ /Dest:C:\myfolder /SourceSAS:SAS /S

请注意，不会复制任何空文件夹。


### 以递归方式将文件系统中的文件和文件夹上载到 Azure File Share

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.file.core.chinacloudapi.cn/myfileshare/ /DestKey:key /S

请注意，不会复制任何空文件夹。


### 以递归方式将与指定的文件模式匹配的文件上载到某个 Azure File Share

	AzCopy /Source:C:\myfolder /Dest:https://myaccount.file.core.chinacloudapi.cn/myfileshare/ /DestKey:key /Pattern:ab* /S

### 以异步方式复制 Azure File 存储中的文件

Azure File 存储支持服务器端异步复制。

以异步方式实现从文件存储到文件存储的复制：

	AzCopy /Source:https://myaccount1.file.core.chinacloudapi.cn/myfileshare1/ /Dest:https://myaccount2.file.core.chinacloudapi.cn/myfileshare2/ /SourceKey:key1 /DestKey:key2 /S

以异步方式实现从文件存储到块 Blob 的复制：
  
	AzCopy /Source:https://myaccount1.file.core.chinacloudapi.cn/myfileshare/ /Dest:https://myaccount2.blob.core.chinacloudapi.cn/mycontainer/ /SourceKey:key1 /DestKey:key2 /S

以异步方式实现从块/页 Blob 存储到文件存储的复制：

	AzCopy /Source:https://myaccount1.blob.core.chinacloudapi.cn/mycontainer/ /Dest:https://myaccount2.file.core.chinacloudapi.cn/myfileshare/ /SourceKey:key1 /DestKey:key2 /S

请注意，不支持从文件存储到页 Blob 的异步复制。

### 以同步方式复制 Azure File 存储中的文件

除了异步复制，用户还可以指定选项 `/SyncCopy`，以同步从文件存储复制数据到文件存储、从文件存储到 Blob 存储以及从 Blob 存储到文件存储，AzCopy 实现该同步操作是通过将源数据下载到本地内存并再次上传到目标。

	AzCopy /Source:https://myaccount1.file.core.chinacloudapi.cn/myfileshare1/ /Dest:https://myaccount2.file.core.chinacloudapi.cn/myfileshare2/ /SourceKey:key1 /DestKey:key2 /S /SyncCopy

	AzCopy /Source:https://myaccount1.file.core.chinacloudapi.cn/myfileshare/ /Dest:https://myaccount2.blob.core.chinacloudapi.cn/mycontainer/ /SourceKey:key1 /DestKey:key2 /S /SyncCopy
	
	AzCopy /Source:https://myaccount1.blob.core.chinacloudapi.cn/mycontainer/ /Dest:https://myaccount2.file.core.chinacloudapi.cn/myfileshare/ /SourceKey:key1 /DestKey:key2 /S /SyncCopy

当从文件存储复制数据到 Blob 存储时，默认的 blob 类型是块 blob，用户可以指定选项 `/BlobType:page` 以更改目标 blob 类型。

请注意，与异步复制相比，`/SyncCopy` 可能会产生额外出口成本，推荐方法是在与你的源存储账户处于同一区域的 Azure 虚拟机中使用该选项，以避免出口成本。


## 使用 AzCopy 复制 Azure 表中的文件（仅限预览版本）

下面的示例演示了使用 AzCopy 复制 Azure 表实体的各种方案。

### 将实体导出到本地文件系统

	AzCopy /Source:https://myaccount.table.core.chinacloudapi.cn/myTable/ /Dest:C:\myfolder\ /SourceKey:key

AzCopy 将一个清单文件写入到指定的目标文件夹或 blob 容器。导入进程在导入过程中使用该清单文件来定位必要的数据文件并执行数据验证。缺省情况下，清单文件使用以下命名约定：

	<account name>_<table name>_<timestamp>.manifest

用户还可以指定选项 `/Manifest:<manifest file name>` 以设置清单文件名。

	AzCopy /Source:https://myaccount.table.core.chinacloudapi.cn/myTable/ /Dest:C:\myfolder\ /SourceKey:key /Manifest:abc.manifest


### 将实体导出到 JSON 和 CSV 数据文件格式

默认情况下，AzCopy 将表实体导出到 JSON 文件，用户可以指定选项 `/PayloadFormat:JSON|CSV` 决定导出的数据文件类型。

	AzCopy /Source:https://myaccount.table.core.chinacloudapi.cn/myTable/ /Dest:C:\myfolder\ /SourceKey:key /PayloadFormat:CSV

除了扩展名为 `.csv`（其位置由参数 `/Dest` 指定）的数据文件，当指定 CSV 负载格式时，AzCopy 将生成每个数据文件扩展名为 `.schema.csv` 的方案文件。
请注意，AzCopy 不包括对导入 CSV 数据文件的支持，你可使用 JSON 格式导出和导入表数据。

### 将实体导出到 Azure blob

	AzCopy /Source:https://myaccount.table.core.chinacloudapi.cn/myTable/ /Dest:https://myaccount.blob.core.chinacloudapi.cn/mycontainer/ /SourceKey:key1 /Destkey:key2

AzCopy 将使用以下命名约定在本地文件夹或 blob 容器中生成一个 JSON 数据文件：

	<account name>_<table name>_<timestamp>_<volume index>_<CRC>.json

生成的 JSON 数据文件遵循最少元数据数据格式。有关此数据格式的详细信息，请参阅[表服务操作的数据格式](https://msdn.microsoft.com/zh-CN/library/azure/dn535600.aspx).

请注意，当导出存储表实体到存储 Blob 时，AzCopy 首先将其导出到本地临时数据文件，然后上载到 Blob，这些临时数据文件放入默认路径为`%LocalAppData%\\Microsoft\\Azure\\AzCopy` 的日志文件夹，你可以指定选项 /Z:[journal-file-folder]，以更改日志文件文件夹位置，从而此更改临时数据文件位置。临时数据文件的大小由你的表实体大小和你使用选项 /SplitSize 指定的大小决定，尽管本地磁盘中的临时数据文件上载到 Blob 后会被立即删除，请确保在删除之前，有足够的本地磁盘空间来存储这些临时数据文件。

### 拆分导出文件

	AzCopy /Source:https://myaccount.table.core.chinacloudapi.cn/mytable/ /Dest:C:\myfolder /SourceKey:key /S /SplitSize:100

AzCopy 在已拆分数据文件名称中使用*卷索引*来区分多个文件。卷索引由两个部分组成：*分区键范围索引*和*拆分文件索引*。两个索引都是从零开始的。

如果用户未指定选项 `/PKRS`（将在下节进行介绍），则分区键范围索引将是 0。

例如，假设 AzCopy 在用户指定选项 `/SplitSize` 后生成了两个数据文件。生成的数据文件名称可能是：

	myaccount_mytable_20140903T051850.8128447Z_0_0_C3040FE8.json
	myaccount_mytable_20140903T051850.8128447Z_0_1_0AB9AC20.json

请注意，选项 `/SplitSize` 的最小可能值为 32 MB。如果指定的目标是 Blob 存储，不管用户是否指定了选项 `/SplitSize`，AzCopy 将在数据文件的大小达到 blob 大小限制 (200GB) 时拆分数据文件。

### 并发导出实体

	AzCopy /Source:https://myaccount.table.core.chinacloudapi.cn/myTable/ /Dest:C:\myfolder\ /SourceKey:key /PKRS:"aa#bb"

当用户指定了选项 `/PKRS` 时，AzCopy 将启动并发操作来导出实体。每个操作导出一个分区键范围。

请注意，并发操作的数量还受选项 `/NC` 控制。当复制表实体时，AzCopy 使用核心处理器的数量作为 `/NC` 的默认值，即使未指定 `/NC` 也是如此。当用户指定了选项 `/PKRS` 时，AzCopy 将使用以下两个值中的较小者（分区键范围、隐式或显式指定的并发操作数量）来确定要启动的并发操作的数量。有关详细信息，请在命令行上键入 `AzCopy /?:NC`。

### 并发导入实体

	AzCopy /Source:C:\myfolder\ /Dest:https://myaccount.table.core.chinacloudapi.cn/mytable1/ /DestKey:key /Manifest:"myaccount_mytable_20140103T112020.manifest" /EntityOperation:InsertOrReplace 

选项 `/EntityOperation` 指示如何将实体插入表中。可能的值包括：

- `InsertOrSkip`：跳过现有实体，或者插入新实体（如果它不存在于表中）。
- `InsertOrMerge`：合并现有实体，或者插入新实体（如果它不存在于表中）。
- `InsertOrReplace`：替换现有实体，或者插入新实体（如果它不存在于表中）。

请注意，在导入方案中不能指定选项 `/PKRS`。与导出方案不同（在导出方案中必须指定 `/PKRS` 选项才会启动并发操作），在导入实体时，AzCopy 将默认启动并发操作。启动的并发操作的默认数量等于核心处理器的数量；不过，你可以通过选项 `/NC` 指定一个不同的并发数量。有关详细信息，请在命令行上键入 `AzCopy /?:NC`。


## 已知问题和最佳实践

#### 在一台计算机上运行一个 AzCopy 实例。
AzCopy 旨在最大限度地利用计算机资源来加快数据传输，如果需要更多的并发操作，我们建议在一台计算机上只运行一个 AzCopy 实例并指定选项 `/NC`。有关详细信息，请在命令行上键 入`AzCopy /?:NC`。

#### 当你激活“将 FIPS 兼容算法用于加密、哈希和签名”的本地策略时，请启用适用于 AzCopy的FIPS 兼容的 MD5 算法。
默认情况下，AzCopy 使用 .NET实现的 MD5 算法来计算 MD5，但出于安全原因有时候我们需要采用FIPS兼容的MD5算法。

可以创建属性为 `AzureStorageUseV1MD5` 的 app.config 文件 `AzCopy.exe.config`，将其放在 AzCopy.exe 所在的目录。

	<?xml version="1.0" encoding="utf-8" ?>
	<configuration>
	  <appSettings>
	    <add key="AzureStorageUseV1MD5" value="false"/>
	  </appSettings>
	</configuration>

至于属性 AzureStorageUseV1MD5
True -默认值，AzCopy 将使用 .NET MD5 实现。
False - AzCopy 将使用 FIPS 兼容的MD5 算法。

请注意，默认情况下，在 Windows 计算机上禁用 FIPS 兼容的算法，可以在你运行的窗口中键入 secpol.msc 并检查此开关在安全设置->本地策略->安全选项->系统加密：将 FIPS 兼容算法用于加密、哈希和签名。

## AzCopy 版本

| 版本 | 新增功能 |
|---------|-----------------------------------------------------------------------------------------------------------------|
| **V4.2.0** | **当前的预览版本。包括 V3.2.0 中的所有功能。此外支持 File 存储共享 SAS、File 存储异步复制，将表实体导出到 CSV 和在导出表实体时指定清单名称**
| **V3.2.0** | **当前的发行版本。支持附加的 Blob 和 FIPS 兼容的 MD5 设置**
| V4.1.0 | 包括 V3.1.0 中的所有功能。支持以同步方式复制 blob 和 File 并指定目标 blob 和 File 的内容类型
| V3.1.0 | 支持以同步方式复制 blob 并指定目标 blob 的内容类型
| V4.0.0 | 包括 V3.0.0 中的所有功能。还支持将文件复制到 Azure File 存储或从其中复制文件，以及将实体复制到 Azure 表存储或从其中复制实体。
| V3.0.0 | 将 AzCopy 命令行语法修改为需要参数名称，并且重新设计了命令行帮助。此版本仅支持复制到 Azure Blob 存储和从其中进行复制。	
| V2.5.1 | 使用选项 /xo 和 /xn 时优化性能。修复了与源文件名称中的特殊字符以及在用户输入错误的命令行语法后导致的恢复日志损坏相关的 bug。	
| V2.5.0 | 优化了大规模复制方案的性能，并执行了几处重要的可用性改进。
| V2.4.1 | 支持在安装向导中指定目标文件夹。                     			
| V2.4.0 | 支持为 Azure File 存储上载和下载文件。
| V2.3.0 | 支持读取访问异地冗余存储帐户。|
| V2.2.2 | 升级为使用 Azure 存储客户端库 3.0.3 版。
| V2.2.1 | 修复了在同一存储帐户内复制大量文件时的性能问题。
| V2.2 | 支持为 blob 名称设置虚拟目录分隔符。支持指定日志文件路径。|
| V2.1 | 提供了 20 多个选项来支持高效执行 blob 上载、下载和复制操作。|


## 后续步骤

有关 Azure 存储和 AzCopy 的更多信息，请参阅以下资源。

### Azure 存储文档：

- [Azure 存储简介](/zh-cn/documentation/articles/storage-introduction)
- [将文件存储在 Blob 存储中](/zh-cn/documentation/articles/storage-dotnet-how-to-use-blobs)
- [在具有文件存储的 Azure 中创建 SMB File Share](/zh-cn/documentation/articles/storage-dotnet-how-to-use-files)

### Azure 存储博客文章：
- [AzCopy：引入了同步复制和自定义内容类型](http://blogs.msdn.com/b/windowsazurestorage/archive/2015/01/13/azcopy-introducing-synchronous-copy-and-customized-content-type.aspx)
- [AzCopy：公开发行AzCopy 3.0，以及发布支持Azure 表（Table）和文件（File）的预览版本](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/10/29/azcopy-announcing-general-availability-of-azcopy-3-0-plus-preview-release-of-azcopy-4-0-with-table-and-file-support.aspx)
- [AzCopy：针对大规模复制方案进行了优化](http://go.microsoft.com/fwlink/?LinkId=507682)
- [Windows Azure 文件（File）存储服务简介](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/12/introducing-microsoft-azure-file-service.aspx)
- [AzCopy：支持读取访问异地冗余存储](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/04/07/azcopy-support-for-read-access-geo-redundant-account.aspx)
- [AzCopy：使用可重新启动的模式和 SAS 令牌传输数据](http://blogs.msdn.com/b/windowsazurestorage/archive/2013/09/07/azcopy-transfer-data-with-re-startable-mode-and-sas-token.aspx)
- [AzCopy：使用跨帐户复制 Blob](http://blogs.msdn.com/b/windowsazurestorage/archive/2013/04/01/azcopy-using-cross-account-copy-blob.aspx)
- [AzCopy：为 Windows Azure Blob 上载/下载文件](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/12/03/azcopy-uploading-downloading-files-for-windows-azure-blobs.aspx)

<!---HONumber=70-->