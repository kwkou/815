<properties 
	pageTitle="如何管理媒体服务中的资产" 
	description="了解如何在媒体服务上管理资产。你还可以管理作业、任务、访问策略、定位符，等等。代码示例用 C# 编写且使用 Media Services SDK for .NET。" 
	services="media-services" 
	documentationCenter="" 
	authors="juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.date="09/07/2015" 
	wacn.date="11/02/2015"/>


#如何：管理存储中的资产


创建媒体资产后，可以在服务器上访问并管理这些资产。还可以在服务器上管理属于媒体服务的其他对象，包括作业、任务、访问策略、定位器，等等。

以下示例演示如何按 assetId 查询资产。

	static IAsset GetAsset(string assetId)
	{
	    // Use a LINQ Select query to get an asset.
	    var assetInstance =
	        from a in _context.Assets
	        where a.Id == assetId
	        select a;
	    // Reference the asset as an IAsset.
	    IAsset asset = assetInstance.FirstOrDefault();
	
	    return asset;
	}

若要列出服务器上的所有可用资产，你可以使用以下方法，该方法将循环访问资产集合并显示有关每个资产的详细信息。
	
	static void ListAssets()
	{
	    string waitMessage = "Building the list. This may take a few "
	        + "seconds to a few minutes depending on how many assets "
	        + "you have."
	        + Environment.NewLine + Environment.NewLine
	        + "Please wait..."
	        + Environment.NewLine;
	    Console.Write(waitMessage);
	
	    // Create a Stringbuilder to store the list that we build. 
	    StringBuilder builder = new StringBuilder();
	
	    foreach (IAsset asset in _context.Assets)
	    {
	        // Display the collection of assets.
	        builder.AppendLine("");
	        builder.AppendLine("******ASSET******");
	        builder.AppendLine("Asset ID: " + asset.Id);
	        builder.AppendLine("Name: " + asset.Name);
	        builder.AppendLine("==============");
	        builder.AppendLine("******ASSET FILES******");
	
	        // Display the files associated with each asset. 
	        foreach (IAssetFile fileItem in asset.AssetFiles)
	        {
	            builder.AppendLine("Name: " + fileItem.Name);
	            builder.AppendLine("Size: " + fileItem.ContentFileSize);
	            builder.AppendLine("==============");
	        }
	    }
	
	    // Display output in console.
	    Console.Write(builder.ToString());
	}

以下代码段将从媒体服务帐户中删除所有资产。请注意，如果一个资产与一个程序相关联，则首先必须删除该程序。

	foreach (IAsset asset in _context.Assets)
	{
	    asset.Delete();
	}

 

<!---HONumber=76-->