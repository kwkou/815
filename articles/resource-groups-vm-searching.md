<properties
   pageTitle="导航和选择 VM 映像 | Azure"
   description="了解在使用资源管理器部署模型创建 Azure 虚拟机时如何确定映像的确定发布者、产品和 SKU。"
   services="virtual-machines"
   documentationCenter=""
   authors="squillace"
   manager="timlt"
   editor=""
   tags="azure-resource-manager"
   />

<tags
   ms.service="virtual-machines"
   ms.date="12/08/2015"
   wacn.date="02/17/2016"/>

# 使用 Windows PowerShell 和 Azure CLI 来浏览和选择 Azure 虚拟机映像

> [AZURE.NOTE]当你在本主题中搜索虚拟机映像时，你在 Azure 服务管理器模式下，使用最近安装的适用于 Mac、Linux 和 Windows 的 Azure 命令行接口，或者使用 Windows PowerShell。使用 Azure CLI 时，键入 `azure config mode asm` 即可进入该模式。使用 PowerShell 0.9 或之前版本时，请键入 `Switch-AzureMode AzureServiceManager`。

## 常用映像表


| PublisherName | 产品 | Name |
|:---------------------------------|:-------------------------------------------|:---------------------------------|:--------------------|
| OpenLogic | CentOS | f1179221e23b4dbb89e39d70e5bc9e72__OpenLogic-CentOS-70-20150904 |
| OpenLogic | CentOS | f1179221e23b4dbb89e39d70e5bc9e72__OpenLogic-CentOS-71-20150731 |
| Microsoft SharePoint Group | MicrosoftSharePointServer | 9619bdbee1584b6f80d684565a6eeb74__SharePoint-2013-Trial-3-26-2014 |
| Microsoft SQL Server Group | SQL2014-WS2012R2 | 74bb2f0b8dcc47fbb2914b60ed940c35__SQL-Server-2014-RTM-12.0.2361.0-DataWarehousing-CHS-Win2012R2-cy14su05 |
| Microsoft SQL Server Group | SQL2014-WS2012R2 | 74bb2f0b8dcc47fbb2914b60ed940c35__SQL-Server-2014-RTM-12.0.2361.0-Web-CHS-Win2012R2-cy14su05 |
| Canonical | UbuntuServer | b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-12_04_5-LTS-amd64-server-20150204-en-us-30GB |
| Canonical | UbuntuServer | b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_04_3-LTS-amd64-server-20151019-en-us-30GB |
| Microsoft | WindowsServer | 55bc2b193643443bb879a78bda516fc8__Windows-Server-2012-Datacenter-20151021-zh.cn-127GB.vhd |
| Microsoft | WindowsServer | 55bc2b193643443bb879a78bda516fc8__Windows-Server-2012-R2-20151021-zh.cn-127GB.vhd |
| Microsoft | WindowsServer | 55bc2b193643443bb879a78bda516fc8__Win2K8R2SP1-Datacenter-20151021-zh.cn-127GB.vhd |
| Microsoft | WindowsServer | 55bc2b193643443bb879a78bda516fc8__Windows-Server-Technical-Preview-20151118-en.us-127GB.vhd |
| Microsoft Windows Server Essentials Group | WindowsServerEssentials | 0c5c79005aae478e8883bf950a861ce0__Windows-Server-2012-Essentials-20141204-zhcn |
| Microsoft Windows Server HPC Pack team | WindowsServerHPCPack | 86f94d8ddf9d4aad9b064efb793ff4c2__HPC-Pack-2012R2-Update1-4.3.4660.0-WS2012R2-CHN |


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

关于如何使用这些映像，请参阅[使用 Azure CLI 创建多 VM 部署](/documentation/articles/virtual-machines-create-multi-vm-deployment-xplat-cli)


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

<!---HONumber=Mooncake_0118_2016-->