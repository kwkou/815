<properties linkid="manage-services-how-to-create-a-storage-account" urlDisplayName="How to create" pageTitle="如何创建存储帐户 | Azure" metaKeywords="" description="了解如何在 Azure 管理门户中创建存储帐户。" metaCanonical="" services="storage" documentationCenter="" title="How To Create a Storage Account" solutions="" authors="tamram" manager="mbaldwin" editor="cgronlun" />



# 关于 Azure 存储帐户

Azure 存储帐户是一个安全的帐户，它向你授予对 Azure 存储空间中服务的访问权限。你的存储帐户为你的数据提供唯一命名空间，而在默认情况下，它仅提供给你这个帐户所有者。 

在中国有一种类型的存储帐户：

- 标准存储帐户包括 Blob、表和队列存储。文件存储可通过 [Azure 门户](/zh-cn/)请求提供。
<!--
- 高级存储帐户当前仅支持 Azure 虚拟机磁盘。Azure  高级存储可通过 [Azure 门户](/zh-cn/)请求提供。请参阅[高级存储：Azure 虚拟机工作负载的高性能存储](http://go.microsoft.com/fwlink/?LinkId=521898) for an in-depth overview of Premium Storage.
-->
我们将根据你的存储帐户，针对你的 Azure 存储空间使用情况收费。存储成本取决于四个因素：存储容量、复制方案、存储事务和数据流出量。 

- 存储容量指的是存储帐户中用来存储数据的配额。对数据进行简单存储时，其成本取决于存储的数据量和数据复制方式。 
- 复制决定了你某一次的数据副本的保留数量，以及保留位置。 
- 事务指的是对 Azure 存储空间的所有读取和写入操作。 
- 数据流出量指的是传出某个 Azure 区域的数据。当不在同一区域中的应用程序访问你的存储帐户中的数据时，无论该应用程序是云服务还是某个其他类型的应用程序，都将会针对数据流出量向你收费。（对于 Azure 服务，你可以采取措施将你的数据和服务通过分组分到相同的数据中心内，从而降低或避免数据流出量费用。）  

[存储定价详细信息](/zh-cn/pricing/details/#storage)页提供了针对存储容量、复制和事务的详细定价信息。[数据传输定价详细信息](/zh-cn/pricing/details/data-transfers/)提供了针对数据流出量的详细定价信息。

有关存储帐户容量和性能目标的详细信息，请参阅 [Azure 存储空间可伸缩性和性能目标](http://msdn.microsoft.com/library/windowsazure/dn249410.aspx).

> [WACOM.NOTE] 当你创建 Azure 虚拟机时，如果在部署位置中还没有存储帐户，则会在该位置自动创建一个存储帐户。因此，没有必要按照下面的步骤来创建虚拟机磁盘的存储帐户。存储帐户名称将基于虚拟机名称。请参阅 [Azure 虚拟机文档](/zh-cn/documentation/services/virtual-machines/)以了解更多详细信息。<br />

##目录##

本文介绍如何创建标准存储帐户，以及当你创建它时需要考虑的一些决策事项。它还介绍如何管理你的存储帐户访问密钥，以及如何删除存储帐户。


- [如何：创建存储帐户](#create)
- [如何：查看、复制和重新生成存储访问密钥](#regeneratestoragekeys)
- [如何：删除存储帐户](#deletestorageaccount)
* [后续步骤](#next)

## <a id="create"></a>如何：创建存储帐户 ##

1. 登录到[管理门户](https://manage.windowsazure.cn).

2. 依次单击"新建"、"存储"和"快速创建"。

	![NewStorageAccount](./media/storage-create-storage-account/storage_NewStorageAccount.png)

3. 在 **URL** 中，输入你的存储帐户的名称。请参阅下面的[存储帐户终结点](#account-endpoints)， 以便详细了解如何将此名称用于存储在 Azure 存储空间中的对象的地址设置。

4. 在"位置/地缘组"中，选择靠近你或你的客户的存储帐户的位置。如果其他 Azure 服务（例如 Azure 虚拟机或云服务）将要访问你存储帐户中的数据，你可能需要从列表中选择一个地缘组，以便将你的存储帐户与其他需要用来改进性能和降低成本的 Azure 服务组合到同一个数据中心。 

	> [WACOM.NOTE] 请注意，在创建你的存储帐户时，必须选择一个地缘组；不能将现有帐户移到地缘组中。

	有关地缘组的详细信息，请参阅下面的[服务与地缘组的归置](#affinity-group) 。

	
5. 如果你有多个 Azure 订阅，则会显示"订阅"字段。在"订阅"中，输入要使用存储帐户的 Azure 订阅。你最多可以为一个订阅创建五个存储帐户。

6. 在"复制"中，选择你的存储帐户的所需复制级别。建议的复制选项为地域冗余复制，可为你的数据提供最大耐用性。有关 Azure 存储复制选项的详细信息，请参阅下面的[存储帐户复制选项](#replication-options) 。

6. 单击"创建存储帐户"。

	创建存储帐户可能需要花费几分钟的时间。若要检查状态，你可以监视门户底部的通知。创建存储帐户后，你的新存储帐户将处于"联机"状态并且随时可供使用。 

![StoragePage](./media/storage-create-storage-account/Storage_StoragePage.png)


### <a id="account-endpoints"></a>存储帐户终结点 

在 Azure 存储空间中存储的每个对象都有唯一的 URL 地址；存储帐户名称构成该地址的子域。特定于每个服务的子域和域名构成你的存储帐户的 *endpoint*。 

例如，如果你的存储帐户名为  *mystorageaccount*，则你的存储帐户的默认终结点为： 

- Blob 服务: http://*mystorageaccount*.blob.core.chinacloudapi.cn

- 表服务: http://*mystorageaccount*.table.core.chinacloudapi.cn

- 队列服务: http://*mystorageaccount*.queue.core.chinacloudapi.cn

- 文件服务: http://*mystorageaccount*.file.core.chinacloudapi.cn

创建存储帐户后，你可以在 Azure 管理门户的存储仪表板上看到该帐户的终结点。

用于访问存储帐户中某个对象的 URL 是通过将存储帐户中对象的位置附加到终结点而构建的。例如，Blob 地址可能具有以下格式：: http://*mystorageaccount*.blob.core.chinacloudapi.cn/*mycontainer*/*myblob*.

此外还可以配置用于存储帐户的自定义域名称。请参阅[为存储帐户中的 Blob 数据配置自定义域名](../storage-custom-domain-name/) 以了解详细信息。

### <a id="affinity-group"></a>服务与地缘组的归置 

 *affinity group*是你的 Azure 服务和 VM 及 Azure 存储帐户的地理分组。通过定位同一数据中心或靠近目标用户受众的计算机工作负载，地缘组可提高服务性能。此外，当某个存储帐户中的数据被另一个服务访问，而该服务是同一个地缘组的一部分时，不会对出口流量收费。

> [WACOM.NOTE]  若要创建地缘组，请打开管理门户的<b>"设置"</b>区域，单击<b>"地缘组"</b>，然后单击<b>"添加地缘组"</b>或<b>"添加"</b>按钮。你也可以使用 Azure 服务管理 API 创建和管理地缘组。有关详细信息，请参阅<a href="http://msdn.microsoft.com/zh-cn/library/windowsazure/ee460798.aspx">对地缘组的操作</a>。


### <a id="replication-options"></a>存储帐户复制选项

[WACOM.INCLUDE [storage-replication-options](../includes/storage-replication-options.md)]


## <a id="regeneratestoragekeys"></a>如何：查看、复制和重新生成存储访问密钥

当你创建存储帐户时，Azure 将生成两个 512 位存储访问密钥，用于在用户访问该存储帐户时对其进行身份验证。通过提供两个存储访问密钥，Azure 使你能够在不中断存储服务的情况下重新生成用于访问该服务的密钥。

> [WACOM.NOTE] 我们建议你避免与其他人共享你的存储帐户访问密钥。若要在不提供你的访问密钥的情况下允许他人访问存储资源，你可以使用 *shared access signature*。共享访问签名可用于访问你帐户中的资源，访问时间间隔由你定义，访问权限由你指定。请参阅[共享访问签名教程](../storage-dotnet-shared-access-signature-part-1/)，以了解详细信息。

在[管理门户](http://manage.windowsazure.cn)中操作, use **Manage Keys** on the dashboard or the **Storage** page to view, copy, and regenerate the storage access keys that are used to access the Blob, Table, and Queue services. 

### 复制存储访问密钥 ###

你可以使用"管理密钥"复制要在连接字符串中使用的存储访问密钥。连接字符串需要在进行身份验证时使用存储帐户名称和密钥。有关配置连接字符串以访问 Azure 存储服务的信息，请参阅[配置连接字符串](http://msdn.microsoft.com/zh-cn/library/ee758697.aspx)。

1. 在[管理门户](http://manage.windowsazure.cn)中操作, click **Storage**, and then click the name of the storage account to open the dashboard.

2. 单击"管理密钥"。

 	此时将打开"管理访问密钥"页面。

	![Managekeys](./media/storage-create-storage-account/Storage_ManageKeys.png)

 
3. 若要复制存储访问密钥，请选择密钥文本。然后右键单击，并单击"复制"。

### 重新生成存储访问密钥 ###
你应定期更改你的存储帐户的访问密钥，使存储连接更安全。分配了两个访问密钥，以便在你重新生成其中一个访问密钥时，始终能够使用另一个访问密钥连接到存储帐户。 

<div class="dev-callout"> 
    <b>警告</b> 
    <p>重新生成访问密钥会影响虚拟机、媒体服务以及任何依赖于存储帐户的应用程序。必须更新使用访问密钥访问存储帐户的所有客户端，以使用新密钥。
    </p> 
    </div>

虚拟机 - 如果你的存储帐户包含任何正在运行的虚拟机，则在重新生成访问密钥后必须重新部署所有虚拟机。若要避免重新部署，请在重新生成访问密钥之前关闭虚拟机。
 
Media Services - 如果你的 Media Services 依赖于存储帐户，则必须在重新生成密钥后将访问密钥与 Media Services 重新同步。
 
应用程序 - 如果你具有使用存储帐户的 Web 应用程序或云服务，则重新生成密钥将失去连接，除非你滚动使用密钥。过程如下：

1. 更新应用程序代码中的连接字符串以引用存储帐户的辅助访问密钥。 

2. 为你的存储帐户重新生成主访问密钥。在[管理门户](http://manage.windowsazure.cn)中操作，从仪表板或"配置"页上，单击"管理密钥"。单击主访问密钥下的"重新生成"，然后单击"是"确认你要生成新密钥。

3. 更新代码中的连接字符串以引用新的主访问密钥。

4. 重新生成辅助访问密钥。

## <a id="deletestorageaccount"></a>如何：删除存储帐户

若要删除不再使用的存储帐户，请使用仪表板或"配置"页上的"删除"。"删除"操作将删除整个存储帐户，包括帐户中的所有 Blob、表和队列。 

<div class="dev-callout">
	<b>警告</b>
	<p>无法从已删除的存储帐户中还原内容。请确保在删除帐户之前对要保存的所有内容进行备份。
	</p>
	<p>
	如果你的存储帐户包含用于 Azure 虚拟机的任何 VHD 文件或磁盘，则必须删除使用这些 VHD 文件的任何映像和磁盘，然后才能删除存储帐户。首先，如果虚拟机正在运行，则停止运行，然后将其删除。若要删除磁盘，请导航到"磁盘"选项卡并删除存储帐户中包含的所有磁盘。若要删除映像，请导航到"映像"选项卡并删除存储帐户中存储的任何映像。
	</p>
</div>


1. 在[管理门户](http://manage.windowsazure.cn)中操作, click **Storage**.

2. 单击存储帐户条目中除名称之外的任何位置，然后单击"删除"。

	 -或-

	单击存储帐户的名称以打开仪表板，然后单击"删除"。

3. 单击"是"确认你要删除存储帐户。

## <a id="next"></a>后续步骤

- 若要详细了解 Azure 存储服务，请参阅[了解云存储](/zh-cn/documentation/articles/storage-introduction/)和 [Blob、队列和表](http://msdn.microsoft.com/zh-cn/library/gg433040.aspx)。

- 访问 [Azure 存储空间团队博客](http://blogs.msdn.com/b/windowsazurestorage/).



<!--HONumber=41-->
