<properties
	pageTitle="通过 Java 上传大文件到媒体服务账号报错"
	description="Java 通过分片的方式上传大文件到媒体服务账号中"
	services="media-services"
	documentationCenter=""
	authors=""
	manager=""
	editor=""
	tags="Azure,java,媒体服务,分片"/>

<tags
    ms.service="media-services-aog"
    ms.date="12/08/2016"
    wacn.date="12/08/2016"/>

# 通过 Java 上传大文件到媒体服务账号报错 #

### 问题现象 ###

通过 Java 代码上传文件到媒体服务账号中，若上传的文件过大，则会上传失败。

### 解决方法 ###

可以选择通过分片的方式进行上传，从而解决该问题，示例代码如下：


	// The name of the file as it will exist in your Media Services account.
	String fileName = "xxxxxxxx";  
	// The local file that will be uploaded to your Media Services account.
	InputStream input = new FileInputStream(new File("D:/Test" + fileName));
	String blobName = fileName;
	// Upload the local file to the asset.
	uploader.createBlockBlob(fileName, null);        
	String blockId;
	byte[] buffer = new byte[1024000];
	BlockList blockList = new BlockList();
	int bytesRead;        
	ByteArrayInputStream byteArrayInputStream;
	while ((bytesRead = input.read(buffer)) > 0) 
	{
	        blockId = UUID.randomUUID().toString();
	        byteArrayInputStream = new ByteArrayInputStream(buffer, 0, bytesRead);
	        uploader.createBlobBlock(blobName, blockId, byteArrayInputStream);
	        blockList.addUncommittedEntry(blockId);
	}
	uploader.commitBlobBlocks(blobName, blockList);

