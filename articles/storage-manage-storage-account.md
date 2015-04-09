<properties linkid="manage-services-how-to-manage-a-storage-account" urlDisplayName="How to manage" pageTitle="如何管理存储帐户 | Microsoft Azure" metaKeywords="Azure manage storage accounts, storage account management portal, storage account geo-replication, Azure geo-replication, Azure access keys" description="了解如何使用管理门户在 Azure 中管理存储帐户。" metaCanonical="" services="storage" documentationCenter="" title="How To Manage Storage Accounts" authors="tamram" solutions="" manager="mbaldwin" editor="cgronlun" />




<h1><a id="managestorageaccounts"></a>如何管理存储帐户。</h1>

##目录##

* [如何：管理存储帐户复制](#georeplication)
* [如何：查看、复制和重新生成存储访问密钥](#regeneratestoragekeys)
* [如何：删除存储帐户](#deletestorageaccount)

<h2><a id="georeplication"></a>如何：复制存储帐户数据以实现耐用性和高可用性</h2>

[WACOM.INCLUDE [storage-replication-options](../includes/storage-replication-options.md)]

### 为存储帐户指定复制设置 ###

1. 在 [Azure 管理门户](https://manage.windowsazure.cn)单击"存储"中，然后单击你的存储帐户名称以显示仪表板。

2. 单击"配置"。

3. 在"复制"字段中，选择要用于存储帐户的复制类型。

4. 单击"保存"，并在出现提示时确认你的选择。


<h2><a id="regeneratestoragekeys"></a>如何：查看、复制和重新生成存储访问密钥</h2>
当你创建存储帐户时，Azure 将生成两个 512 位存储访问密钥，用于在用户访问该存储帐户时对其进行身份验证。通过提供两个存储访问密钥，Azure 使你能够在不中断存储服务的情况下重新生成用于访问该服务的密钥。

在[管理门户](http://manage.windowsazure.cn)中操作 在管理门户中，可使用仪表板或"存储"页上的"管理密钥"查看、复制和重新生成用于访问 Blob、表和队列服务的存储访问密钥。

### 复制存储访问密钥 ###

你可以使用"管理密钥"复制要在连接字符串中使用的存储访问密钥。连接字符串需要在进行身份验证时使用存储帐户名称和密钥。有关配置连接字符串以访问 Azure 存储服务的信息，请参阅[配置连接字符串](http://msdn.microsoft.com/zh-cn/library/ee758697.aspx)。

1. 在[管理门户](http://manage.windowsazure.cn)中操作单击"存储"，然后单击你的存储帐户名称以打开仪表板。

2. 单击"管理密钥"。

 	此时将打开"管理访问密钥"页面。

	![Managekeys](./media/storage-manage-storage-account/Storage_ManageKeys.png)

 
3. 若要复制存储访问密钥，请选择密钥文本。然后右键单击，并单击"复制"。

### 重新生成存储访问密钥 ###
你应定期更改你的存储帐户的访问密钥，使存储连接更安全。分配了两个访问密钥，以便在你重新生成其中一个访问密钥时，始终能够使用另一个访问密钥连接到存储帐户。 

<div class="dev-callout"> 
    <b>警告</b> 
    <p>重新生成访问密钥会影响虚拟机、媒体服务以及任何依赖于存储帐户的应用程序。
    </p> 
    </div>

虚拟机 - 如果你的存储帐户包含任何正在运行的虚拟机，则在重新生成访问密钥后必须重新部署所有虚拟机。若要避免重新部署，请在重新生成访问密钥之前关闭虚拟机。
 
Media Services - 如果你的 Media Services 依赖于存储帐户，则必须在重新生成密钥后将访问密钥与 Media Services 重新同步。
 
应用程序 - 如果你具有使用存储帐户的 Web 应用程序或云服务，则重新生成密钥将失去连接，除非你滚动使用密钥。过程如下：

1. 更新应用程序代码中的连接字符串以引用存储帐户的辅助访问密钥。 

2. 为你的存储帐户重新生成主访问密钥。在[管理门户](http://manage.windowsazure.cn)中操作, 从仪表板或"配置"页上，单击"管理密钥"。单击主访问密钥下的"重新生成"，然后单击"是"确认你要生成新密钥。

3. 更新代码中的连接字符串以引用新的主访问密钥。

4. 重新生成辅助访问密钥。


<h2><a id="deletestorageaccount"></a>如何：删除存储帐户</h2>

若要删除不再使用的存储帐户，请使用仪表板或"配置"页上的"删除"。"删除"操作将删除整个存储帐户，包括帐户中的所有 Blob、表和队列。 

<div class="dev-callout">
	<b>警告</b>
	<p>无法从已删除的存储帐户中还原内容。请确 
	保在删除帐户之前对要保存的所有内容进行备份。
	</p>
	<p>
	如果你的存储帐户包含用于 Azure 虚拟机的任何 VHD 文件或磁盘，则必须删除使用这些 VHD 文件的任何映像和磁盘，然后才能删除存储帐户。首先，如果虚拟机正在运行，则停止运行，然后将其删除。若要删除磁盘，请导航到"磁盘"选项卡并删除存储帐户中包含的所有磁盘。若要删除映像，请导航到"映像"选项卡并删除存储帐户中存储的任何映像。
	</p>
</div>


1. 在[管理门户](http://manage.windowsazure.cn)中操作, click **Storage**。

2. 单击存储帐户条目中除名称之外的任何位置，然后单击"删除"。

	 -或-

	单击存储帐户的名称以打开仪表板，然后单击"删除"。

3. 单击"是"确认你要删除存储帐户。
<!--HONumber=41-->
