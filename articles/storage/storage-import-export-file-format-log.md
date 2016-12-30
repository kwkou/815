<properties
    pageTitle="导入/导出服务日志文件格式 | Azure"
    description="了解针对导入/导出服务作业执行步骤时创建的日志文件的格式"
    author="renashahmsft"
    manager="aungoo"
    editor="tysonn"
    services="storage"
    documentationcenter="" />  

<tags
    ms.assetid="38cc16bd-ad55-4625-9a85-e1726c35fd1b"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/25/2015"
    wacn.date="12/29/2016"
    ms.author="renash" />  


# 导入/导出服务日志文件格式
当 Azure 导入/导出服务在执行导入作业或导出作业的过程中针对驱动器执行某个操作时，会将日志写入到与该作业关联的存储帐户中的块 Blob 中。
  
导入/导出服务可能会写入两种日志：
  
-   发生错误时始终生成错误日志。
  
-   详细日志默认未启用，但可通过对[放置作业](https://docs.microsoft.com/zh-CN/rest/api/storageservices/importexport/Put-Job)或[更新作业属性](https://docs.microsoft.com/zh-CN/rest/api/storageservices/importexport/Update-Job-Properties)操作设置 `EnableVerboseLog` 属性来启用该日志。
  
## 日志文件的位置  
日志将写入到 `ImportExportStatesPath` 设置（可在“`Put Job`”操作中设置）指定的容器或虚拟目录中的块 Blob。日志写入到的位置取决于为该作业指定身份验证的方式，以及为 `ImportExportStatesPath` 指定的值。可通过存储帐户密钥或容器 SAS（共享访问签名）为作业指定身份验证。
  
容器或虚拟目录的名称可以是 `waimportexport` 的默认名称，也可以是指定的另一个容器或虚拟目录名称。
  
下表显示了可能的选项：
  
|身份验证方法|`ImportExportStatesPath` 元素的值|日志 Blob 的位置|  
|---------------------------|----------------------------------------------|---------------------------|  
|存储帐户密钥|默认值|<p>名为 `waimportexport` 的容器，这是默认容器。例如：</p><p> `https://myaccount.blob.core.chinacloudapi.cn/waimportexport`</p>|  
|存储帐户密钥|用户指定的值|<p>由用户命名的容器。例如：</p><p> `https://myaccount.blob.core.chinacloudapi.cn/mylogcontainer`</p>|  
|容器 SAS|默认值|<p>名为 `waimportexport` 的虚拟目录，这是默认名称，位于 SAS 中指定的容器下方。</p><p> 例如，如果为作业指定的 SAS 是 `https://myaccount.blob.core.chinacloudapi.cn/mylogcontainer?sv=2012-02-12&se=2015-05-22T06%3A54%3A55Z&sr=c&sp=wl&sig=sigvalue`，则日志位置应为 `https://myaccount.blob.core.chinacloudapi.cn/mylogcontainer/waimportexport`</p>|  
|容器 SAS|用户指定的值|<p>由用户命名的虚拟目录，位于 SAS 中指定的容器下方。</p><p> 例如，如果为作业指定的 SAS 是 `https://myaccount.blob.core.chinacloudapi.cn/mylogcontainer?sv=2012-02-12&se=2015-05-22T06%3A54%3A55Z&sr=c&sp=wl&sig=sigvalue`，指定的虚拟目录名为 `mylogblobs`，则日志位置应为 `https://myaccount.blob.core.chinacloudapi.cn/mylogcontainer/waimportexport/mylogblobs`。</p>|  
  
可以通过调用[获取作业](https://docs.microsoft.com/zh-CN/rest/api/storageservices/importexport/Get-Job3)操作来检索错误日志和详细日志的 URL。处理完驱动器后，将提供日志。
  
## 日志文件格式  
这两种日志的格式相同：它是一个 Blob，包含在硬盘驱动器与客户帐户之间复制 Blob 时发生的事件的 XML 说明。
  
详细日志包含有关每个 Blob（针对导入作业）或文件（针对导出作业）的复制操作状态的完整信息，错误日志仅包含执行导入作业或导出作业期间遇到错误的 Blob 或文件的信息。
  
详细日志格式如下所示。错误日志具有相同的结构，但可筛选出成功的操作。


	<DriveLog Version="2014-11-01">  
	  <DriveId>drive-id</DriveId>  
	  [<Blob Status="blob-status">  
	   <BlobPath>blob-path</BlobPath>  
	   <FilePath>file-path</FilePath>  
	   [<Snapshot>snapshot</Snapshot>]  
	   <Length>length</Length>  
	   [<LastModified>last-modified</LastModified>]  
	   [<ImportDisposition Status="import-disposition-status">import-disposition</ImportDisposition>]  
	   [page-range-list-or-block-list]  
	   [metadata-status]  
	   [properties-status]  
	  </Blob>]  
	  [<Blob>  
	    . . .  
	  </Blob>]  
	  <Status>drive-status</Status>  
	</DriveLog>  
  
	page-range-list-or-block-list ::= 
	  page-range-list | block-list  
  
	page-range-list ::=   
	<PageRangeList>  
	      [<PageRange Offset="page-range-offset" Length="page-range-length"   
	       [Hash="md5-hash"] Status="page-range-status"/>]  
	      [<PageRange Offset="page-range-offset" Length="page-range-length"   
	       [Hash="md5-hash"] Status="page-range-status"/>]  
	</PageRangeList>  
  
	block-list ::=  
	<BlockList>  
	      [<Block Offset="block-offset" Length="block-length" [Id="block-id"]  
	       [Hash="md5-hash"] Status="block-status"/>]  
	      [<Block Offset="block-offset" Length="block-length" [Id="block-id"]   
	       [Hash="md5-hash"] Status="block-status"/>]  
	</BlockList>  
  
	metadata-status ::=  
	<Metadata Status="metadata-status">  
	   [<GlobalPath Hash="md5-hash">global-metadata-file-path</GlobalPath>]  
	   [<Path Hash="md5-hash">metadata-file-path</Path>]  
	</Metadata>  
  
	properties-status ::=  
	<Properties Status="properties-status">  
	   [<GlobalPath Hash="md5-hash">global-properties-file-path</GlobalPath>]  
	   [<Path Hash="md5-hash">properties-file-path</Path>]  
	</Properties>  


下表介绍了日志文件的元素。
  
|XML 元素|类型|说明|  
|-----------------|----------|-----------------|  
|`DriveLog`|XML 元素|表示驱动器日志。|  
|`Version`|属性，字符串|日志格式的版本。|  
|`DriveId`|String|驱动器的硬件序列号。|  
|`Status`|String|驱动器处理状态。有关详细信息，请参阅下面的“`Drive Status Codes`”表。|  
|`Blob`|嵌套的 XML 元素|表示 Blob。|  
|`Blob/BlobPath`|String|Blob 的 URI。|  
|`Blob/FilePath`|String|驱动器上文件的相对路径。|  
|`Blob/Snapshot`|DateTime|Blob 的快照版本（仅适用于导出作业）。|  
|`Blob/Length`|Integer|Blob 的总长度（字节）。|  
|`Blob/LastModified`|DateTime|上次修改 Blob 的日期/时间（仅适用于导出作业）。|  
|`Blob/ImportDisposition`|String|Blob 的导入处置（仅适用于导入作业）。|  
|`Blob/ImportDisposition/@Status`|属性，字符串|导入处理状态。|  
|`PageRangeList`|嵌套的 XML 元素|表示页 Blob 的页面范围列表。|  
|`PageRange`|XML 元素|表示页面范围。|  
|`PageRange/@Offset`|属性，整数|Blob 中页面范围的起始偏移量。|  
|`PageRange/@Length`|属性，整数|页面范围的长度（字节）。|  
|`PageRange/@Hash`|属性，字符串|页面范围的 Base16 编码 MD5 哈希。|  
|`PageRange/@Status`|属性，字符串|页面范围的处理状态。|  
|`BlockList`|嵌套的 XML 元素|表示块 Blob 的块列表。|  
|`Block`|XML 元素|表示块。|  
|`Block/@Offset`|属性，整数|Blob 中块的起始偏移量。|  
|`Block/@Length`|属性，整数|块的长度（字节）。|  
|`Block/@Id`|属性，字符串|块 ID。|  
|`Block/@Hash`|属性，字符串|块的 Base16 编码 MD5 哈希。|  
|`Block/@Status`|属性，字符串|块的处理状态。|  
|`Metadata`|嵌套的 XML 元素|表示 Blob 的元数据。|  
|`Metadata/@Status`|属性，字符串|Blob 元数据的处理状态。|  
|`Metadata/GlobalPath`|String|全局元数据文件的相对路径。|  
|`Metadata/GlobalPath/@Hash`|属性，字符串|全局元数据文件的 Base16 编码 MD5 哈希。|  
|`Metadata/Path`|String|元数据文件的相对路径。|  
|`Metadata/Path/@Hash`|属性，字符串|元数据文件的 Base16 编码 MD5 哈希。|  
|`Properties`|嵌套的 XML 元素|表示 Blob 属性。|  
|`Properties/@Status`|属性，字符串|Blob 属性的处理状态，例如，找不到文件、已完成。|  
|`Properties/GlobalPath`|String|全局 properties 文件的相对路径。|  
|`Properties/GlobalPath/@Hash`|属性，字符串|全局 properties 文件的 Base16 编码 MD5 哈希。|  
|`Properties/Path`|String|properties 文件的相对路径。|  
|`Properties/Path/@Hash`|属性，字符串|properties 文件的 Base16 编码 MD5 哈希。|  
|`Blob/Status`|String|Blob 的处理状态。|  
  
### 驱动器状态代码  
下表列出了驱动器的处理状态代码。
  
|状态代码|说明|  
|-----------------|-----------------|  
|`Completed`|驱动器已完成处理，未出现任何错误。|  
|`CompletedWithWarnings`|驱动器已完成处理针对 Blob 指定的导入处置，但一个或多个 Blob 中出现警告。|  
|`CompletedWithErrors`|驱动器已完成处理，但一个或多个 Blob 或区块中出现错误。|  
|`DiskNotFound`|在驱动器上找不到磁盘。|  
|`VolumeNotNtfs`|磁盘上的第一个数据卷未采用 NTFS 格式。|  
|`DiskOperationFailed`|对驱动器执行操作时出现未知错误。|  
|`BitLockerVolumeNotFound`|找不到可通过 BitLocker 加密的卷。|  
|`BitLockerNotActivated`|未在卷上启用 BitLocker。|  
|`BitLockerProtectorNotFound`|卷上不存在数字密码密钥保护程序。|  
|`BitLockerKeyInvalid`|提供的数字密码无法解锁卷。|  
|`BitLockerUnlockVolumeFailed`|尝试解锁卷时出现未知错误。|  
|`BitLockerFailed`|执行 BitLocker 操作时出现未知错误。|  
|`ManifestNameInvalid`|清单文件名无效。|  
|`ManifestNameTooLong`|清单文件名太长。|  
|`ManifestNotFound`|找不到清单文件。|  
|`ManifestAccessDenied`|拒绝访问清单文件。|  
|`ManifestCorrupted`|清单文件已损坏（内容与其哈希不匹配）。|  
|`ManifestFormatInvalid`|清单内容不符合所需的格式。|  
|`ManifestDriveIdMismatch`|清单文件中的驱动器 ID 与从驱动器读取的 ID 不匹配。|  
|`ReadManifestFailed`|从清单进行读取时出现磁盘 I/O 故障。|  
|`BlobListFormatInvalid`|导出 Blob 列表 Blob 不符合所需的格式。|  
|`BlobRequestForbidden`|禁止访问存储帐户中的 Blob。这可能是因为存储帐户密钥或容器 SAS 无效。|  
|`InternalError`|处理驱动器时出现内部错误。|  
  
### Blob 状态代码  
下表列出了 Blob 的处理状态代码。
  
|状态代码|说明|  
|-----------------|-----------------|  
|`Completed`|Blob 已完成处理，未出现错误。|  
|`CompletedWithErrors`|Blob 已完成处理，但一个或多个页面范围或者块、元数据或属性中出现错误。|  
|`FileNameInvalid`|文件名无效。|  
|`FileNameTooLong`|文件名太长。|  
|`FileNotFound`|找不到该文件。|  
|`FileAccessDenied`|拒绝访问文件。|  
|`BlobRequestFailed`|访问 Blob 的 Blob 服务请求失败。|  
|`BlobRequestForbidden`|访问 Blob 的 Blob 服务请求被禁止。这可能是因为存储帐户密钥或容器 SAS 无效。|  
|`RenameFailed`|无法重命名 Blob（针对导入作业）或文件（针对导出作业）。|  
|`BlobUnexpectedChange`|Blob 出现意外更改（针对导出作业）。|  
|`LeasePresent`|Blob 上存在租约。|  
|`IOFailed`|处理 Blob 时出现磁盘或网络 I/O 故障。|  
|`Failed`|处理 Blob 时出现未知错误。|  
  
### 导入处置状态代码  
下表列出了导入处置的解决状态代码。
  
|状态代码|说明|  
|-----------------|-----------------|  
|`Created`|已创建 Blob。|  
|`Renamed`|已根据重命名导入处置将 Blob 重命名。`Blob/BlobPath` 元素包含已重命名的 Blob 的 URI。|  
|`Skipped`|已根据 `no-overwrite` 导入处置跳过 Blob。|  
|`Overwritten`|Blob 已根据 `overwrite` 导入处置覆盖现有 Blob。|  
|`Cancelled`|前一个错误导致停止进一步处理导入处置。|  
  
### 页面范围/块状态代码  
下表列出了页面范围或块的处理状态代码。
  
|状态代码|说明|  
|-----------------|-----------------|  
|`Completed`|页面范围或块已完成处理，未出现任何错误。|  
|`Committed`|块已提交，但不在完整块列表中，因为其他块已失败或放置完整块列表本身已失败。|  
|`Uncommitted`|块已上载但未提交。|  
|`Corrupted`|页面范围或块已损坏（内容与其哈希不匹配）。|  
|`FileUnexpectedEnd`|遇到了意外的文件结束。|  
|`BlobUnexpectedEnd`|遇到了意外的 Blob 结束。|  
|`BlobRequestFailed`|访问页面范围或块的 Blob 服务请求失败。|  
|`IOFailed`|处理页面范围或块时出现磁盘或网络 I/O 故障。|  
|`Failed`|处理页面范围或块时出现未知错误。|  
|`Cancelled`|前一个错误导致停止进一步处理页面范围或块。|  
  
### 元数据状态代码  
下表列出了 Blob 元数据的处理状态代码。
  
|状态代码|说明|  
|-----------------|-----------------|  
|`Completed`|元数据已完成处理，未出现错误。|  
|`FileNameInvalid`|元数据文件名无效。|  
|`FileNameTooLong`|元数据文件名太长。|  
|`FileNotFound`|找不到元数据文件。|  
|`FileAccessDenied`|拒绝访问元数据文件。|  
|`Corrupted`|元数据文件已损坏（内容与其哈希不匹配）。|  
|`XmlReadFailed`|元数据内容不符合所需的格式。|  
|`XmlWriteFailed`|写入元数据 XML 失败。|  
|`BlobRequestFailed`|访问元数据的 Blob 服务请求失败。|  
|`IOFailed`|处理元数据时出现磁盘或网络 I/O 故障。|  
|`Failed`|处理元数据时出现未知错误。|  
|`Cancelled`|前一个错误导致停止进一步处理元数据。|  
  
### 属性状态代码  
下表列出了 Blob 属性的处理状态代码。
  
|状态代码|说明|  
|-----------------|-----------------|  
|`Completed`|属性已完成处理，未出现任何错误。|  
|`FileNameInvalid`|properties 文件名无效。|  
|`FileNameTooLong`|properties 文件名太长。|  
|`FileNotFound`|找不到 properties 文件。|  
|`FileAccessDenied`|拒绝访问 properties 文件。|  
|`Corrupted`|properties 文件已损坏（内容与其哈希不匹配）。|  
|`XmlReadFailed`|属性内容不符合所需的格式。|  
|`XmlWriteFailed`|写入属性 XML 失败。|  
|`BlobRequestFailed`|访问属性的 Blob 服务请求失败。|  
|`IOFailed`|处理属性时出现磁盘或网络 I/O 故障。|  
|`Failed`|处理属性时出现未知错误。|  
|`Cancelled`|前一个错误导致停止进一步处理属性。|  
  
## 示例日志  
下面是一个详细日志示例。
  

	<?xml version="1.0" encoding="UTF-8"?>  
	<DriveLog Version="2014-11-01">  
	    <DriveId>WD-WMATV123456</DriveId>  
	    <Blob Status="Completed">  
	       <BlobPath>pictures/bob/wild/desert.jpg</BlobPath>  
	       <FilePath>\Users\bob\Pictures\wild\desert.jpg</FilePath>  
	       <Length>98304</Length>  
	       <ImportDisposition Status="Created">overwrite</ImportDisposition>  
	       <BlockList>  
	          <Block Offset="0" Length="65536" Id="AAAAAA==" Hash=" 9C8AE14A55241F98533C4D80D85CDC68" Status="Completed"/>  
	          <Block Offset="65536" Length="32768" Id="AQAAAA==" Hash=" DF54C531C9B3CA2570FDDDB3BCD0E27D" Status="Completed"/>  
	       </BlockList>  
	       <Metadata Status="Completed">  
	          <GlobalPath Hash=" E34F54B7086BCF4EC1601D056F4C7E37">\Users\bob\Pictures\wild\metadata.xml</GlobalPath>  
	       </Metadata>  
	    </Blob>  
	    <Blob Status="CompletedWithErrors">  
	       <BlobPath>pictures/bob/animals/koala.jpg</BlobPath>  
	       <FilePath>\Users\bob\Pictures\animals\koala.jpg</FilePath>  
	       <Length>163840</Length>  
	       <ImportDisposition Status="Overwritten">overwrite</ImportDisposition>  
	       <PageRangeList>  
	          <PageRange Offset="0" Length="65536" Hash="19701B8877418393CB3CB567F53EE225" Status="Completed"/>  
	          <PageRange Offset="65536" Length="65536" Hash="AA2585F6F6FD01C4AD4256E018240CD4" Status="Corrupted"/>  
	          <PageRange Offset="131072" Length="4096" Hash="9BA552E1C3EEAFFC91B42B979900A996" Status="Completed"/>  
	       </PageRangeList>  
	       <Properties Status="Completed">  
	          <Path Hash="38D7AE80653F47F63C0222FEE90EC4E7">\Users\bob\Pictures\animals\koala.jpg.properties</Path>  
	       </Properties>  
	    </Blob>  
	    <Status>CompletedWithErrors</Status>  
	</DriveLog>  

  
相应的错误日志如下所示。
  

	<?xml version="1.0" encoding="UTF-8"?>  
	<DriveLog Version="2014-11-01">  
	    <DriveId>WD-WMATV6965824</DriveId>  
	    <Blob Status="CompletedWithErrors">  
	       <BlobPath>pictures/bob/animals/koala.jpg</BlobPath>  
	       <FilePath>\Users\bob\Pictures\animals\koala.jpg</FilePath>  
	       <Length>163840</Length>  
	       <ImportDisposition Status="Overwritten">overwrite</ImportDisposition>  
	       <PageRangeList>  
	          <PageRange Offset="65536" Length="65536" Hash="AA2585F6F6FD01C4AD4256E018240CD4" Status="Corrupted"/>  
	       </PageRangeList>  
	    </Blob>  
	    <Status>CompletedWithErrors</Status>  
	</DriveLog>  


 以下导入作业的错误日志包含有关在导入驱动器上找不到的文件的错误。请注意，后续组件的状态为 `Cancelled`。
  

	<?xml version="1.0" encoding="utf-8"?>  
	<DriveLog Version="2014-11-01">  
	  <DriveId>9WM35C2V</DriveId>  
	  <Blob Status="FileNotFound">  
	    <BlobPath>pictures/animals/koala.jpg</BlobPath>  
	    <FilePath>\animals\koala.jpg</FilePath>  
	    <Length>30310</Length>  
	    <ImportDisposition Status="Cancelled">rename</ImportDisposition>  
	    <BlockList>  
	      <Block Offset="0" Length="6062" Id="MD5/cAzn4h7VVSWXf696qp5Uaw==" Hash="700CE7E21ED55525977FAF7AAA9E546B" Status="Cancelled" />  
	      <Block Offset="6062" Length="6062" Id="MD5/PEnGwYOI8LPLNYdfKr7kAg==" Hash="3C49C6C18388F0B3CB35875F2ABEE402" Status="Cancelled" />  
	      <Block Offset="12124" Length="6062" Id="MD5/FG4WxqfZKuUWZ2nGTU2qVA==" Hash="146E16C6A7D92AE5166769C64D4DAA54" Status="Cancelled" />  
	      <Block Offset="18186" Length="6062" Id="MD5/ZzibNDzr3IRBQENRyegeXQ==" Hash="67389B343CEBDC8441404351C9E81E5D" Status="Cancelled" />  
	      <Block Offset="24248" Length="6062" Id="MD5/ZzibNDzr3IRBQENRyegeXQ==" Hash="67389B343CEBDC8441404351C9E81E5D" Status="Cancelled" />  
	    </BlockList>  
	  </Blob>  
	  <Status>CompletedWithErrors</Status>  
	</DriveLog>  


以下导出作业的错误日志指示已将 Blob 内容成功写入驱动器，但在导出 Blob 的属性时出错。
  

	<?xml version="1.0" encoding="utf-8"?>  
	<DriveLog Version="2014-11-01">  
	  <DriveId>9WM35C3U</DriveId>  
	  <Blob Status="CompletedWithErrors">  
	    <BlobPath>pictures/wild/canyon.jpg</BlobPath>  
	    <FilePath>\pictures\wild\canyon.jpg</FilePath>  
	    <LastModified>2012-09-18T23:47:08Z</LastModified>  
	    <Length>163840</Length>  
	    <BlockList />  
	    <Properties Status="Failed" />  
	  </Blob>  
	  <Status>CompletedWithErrors</Status>  
	</DriveLog>  

  
## 另请参阅  
[存储导入/导出 REST](https://docs.microsoft.com/zh-CN/rest/api/storageservices/importexport/Storage-Import-Export-Service-REST-API-Reference)

<!---HONumber=Mooncake_1226_2016-->