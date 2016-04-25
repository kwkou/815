

## Azure CLI

查找映像的最简单且最快速方式是调用 `azure vm image list` 命令，并传递位置、发布者名称）和提供产品。例如，如果你知道“Canonical”是“Ubuntu”产品的发布者，

你可以使用以下 bash 命令进行查询：

	azure vm image list | cat | grep -E "data:\s*Name|data:\s*-+|data:\s*[^\s]*Ubuntu.*\s*Canonical"
	data:    Name                                                                                                      Category  OS       Publisher
	data:    --------------------------------------------------------------------------------------------------------  --------  -------  -----------------------------------------
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-12_04_3-LTS-amd64-server-20140130-en-us-30GB                     Public    Linux    Canonical
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-12_04_4-LTS-amd64-server-20140529-en-us-30GB                     Public    Linux    Canonical
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-12_04_4-LTS-amd64-server-20140606-en-us-30GB                     Public    Linux    Canonical
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-12_04_4-LTS-amd64-server-20140619-en-us-30GB                     Public    Linux    Canonical
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-12_04_4-LTS-amd64-server-20140702-en-us-30GB                     Public    Linux    Canonical
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-12_04_5-LTS-amd64-server-20140829.2-en-us-30GB                   Public    Linux    Canonical
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-12_04_5-LTS-amd64-server-20140925.1-en-us-30GB                   Public    Linux    Canonical
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-12_04_5-LTS-amd64-server-20150204-en-us-30GB                     Public    Linux    Canonical
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04-LTS-amd64-server-20140226.1-beta1-en-us-30GB               Public    Linux    Canonical
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04-LTS-amd64-server-20140416.1-en-us-30GB                     Public    Linux    Canonical
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04-LTS-amd64-server-20140528-en-us-30GB                       Public    Linux    Canonical
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04-LTS-amd64-server-20140606.1-en-us-30GB                     Public    Linux    Canonical
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04_1-LTS-amd64-server-20140909-en-us-30GB                     Public    Linux    Canonical
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04_1-LTS-amd64-server-20140927-en-us-30GB                     Public    Linux    Canonical
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04_1-LTS-amd64-server-20141125-en-us-30GB                     Public    Linux    Canonical
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04_1-LTS-amd64-server-20150123-en-us-30GB                     Public    Linux    Canonical
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04_2-LTS-amd64-server-20150706-en-us-30GB                     Public    Linux    Canonical
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04_3-LTS-amd64-server-20151019-en-us-30GB                     Public    Linux    Canonical
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_10-amd64-server-20140826-beta1-en-us-30GB                     Public    Linux    Canonical
	data:    b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-15_04-amd64-server-20150707-en-us-30GB                           Public    Linux    Canonical


## PowerShell

使用 Azure PowerShell 创建新的虚拟机时，在某些情况下，你需要使用映像名字来指定映像：而映像名字可以通过 `Get-AzureVMImage` 得到。然而所得到的结果非常冗长，你可以通过以下的 PowerShell 脚本对 `Get-AzureVMImage` 的结果进行过滤。

以下的示例能查找到映像名字中含有 **Ubuntu**，操作系统为 **Linux**，发布者为 **Canonical**，标签中包含 **14.04.3** 的映像。

	$images = Get-AzureVMImage
	foreach ($image in $images){
		if (($image.ImageName -like '*Ubuntu*') -and `
			($image.OS -eq 'Linux') -and `
			($image.PublisherName -eq 'Canonical') -and `
			($image.Label -like '*14.04.3*')){
			$image
		}
	}

关于如何使用这些映像，请参阅[在 Azure 中创建 SQL Server 虚拟机 (PowerShell)](/documentation/articles/virtual-machines-sql-server-create-vm-with-powershell)


<!--Image references-->
[5]: ./media/markdown-template-for-new-articles/octocats.png
[6]: ./media/markdown-template-for-new-articles/pretty49.png
[7]: ./media/markdown-template-for-new-articles/channel-9.png
[8]: ./media/markdown-template-for-new-articles/copytemplate.png

<!--Reference style links - using these makes the source content way more readable than using inline links-->
[gog]: http://google.com/
[yah]: http://search.yahoo.com/
[msn]: http://search.msn.com/
