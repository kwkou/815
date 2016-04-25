<properties
	pageTitle="将 VM 的 D 驱动器设为数据磁盘 | Azure"
	description="说明如何更改使用经典部署模型创建的 Windows VM 的驱动器号，以便可以使用 D: 驱动器作为数据驱动器。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="11/03/2015"
	wacn.date="12/31/2015"/>

# 使用 D 盘作为 Windows VM 上的数据驱动器 

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-classic-include.md)]


如果需要使用 D 盘存储数据，请遵循这些说明操作以使用其他驱动器号作为临时磁盘。切勿使用临时磁盘来存储需要保存的数据。

## 附加数据磁盘

首先，需要将数据磁盘附加到虚拟机。若要附加新磁盘，请参阅[如何将数据磁盘附加到 Windows 虚拟机][Attach]。

如果要使用现有数据磁盘，请确保将 VHD 也上载到存储帐户。有关说明，请参阅[创建 Windows Server VHD 并将其上载到 Azure][VHD] 中的步骤 3 和 4。


## 将 pagefile.sys 暂时移到 C 驱动器

1. 连接到虚拟机。 

2. 右键单击“开始”菜单，然后选择“系统”。

3. 在左侧菜单中，选择“高级系统设置”。

4. 在“性能”部分中，选择“设置”。

5. 选择“高级”选项卡。

5. 在“虚拟内存”部分中，选择“更改”。

6. 选择 **C** 驱动器，然后依次单击“系统管理的大小”、“设置”。

7. 选择 **D** 驱动器，然后依次单击“无分页文件”、“设置”。

8. 单击“应用”。你将收到警告，指出计算机需要重新启动才能使更改生效。

9. 重启虚拟机。




## 更改驱动器号 

1. VM 重新启动后，重新登录到 VM。

2. 单击“开始”菜单，键入 **diskmgmt.msc**，然后按 Enter。此时将启动“磁盘管理”。

3. 右键单击 **D**（临时存储驱动器），然后选择“更改驱动器号和路径”。

4. 在“驱动器号”下，选择驱动器 **G**，然后单击“确定”。

5. 右键单击数据磁盘，并选择“更改驱动器号和路径”。

6. 在“驱动器号”下，选择驱动器 **D**，然后单击“确定”。

7. 右键单击 **G**（临时存储驱动器），然后选择“更改驱动器号和路径”。

8. 在“驱动器号”下，选择驱动器 **E**，然后单击“确定”。

> [AZURE.NOTE]如果你的 VM 有其他磁盘或驱动器，可使用相同的方法来重新分配其他磁盘和驱动器的驱动器号。所需的磁盘配置为：
> - C: OS 磁盘  
> - D: 数据磁盘  
> - E: 临时磁盘



## 将 pagefile.sys 移回临时存储驱动器 

1. 右键单击“开始”菜单，然后选择“系统”

2. 在左侧菜单中，选择“高级系统设置”。

3. 在“性能”部分中，选择“设置”。

4. 选择“高级”选项卡。

5. 在“虚拟内存”部分中，选择“更改”。

6. 选择 OS 驱动器 **C**，然后依次单击“无分页文件”、“设置”。

7. 选择临时存储驱动器 **E**，然后依次单击“系统管理的大小”、“设置”。

8. 单击“应用”。你将收到警告，指出计算机需要重新启动才能使更改生效。

9. 重启虚拟机。




## 其他资源
[如何登录到运行 Windows Server 的虚拟机][Logon]

[如何从 Windows 虚拟机分离数据磁盘][Detach]

[关于 Azure 存储帐户][Storage]

<!--Link references-->
[Attach]: /documentation/articles/storage-windows-attach-disk
[VHD]: /documentation/articles/virtual-machines-create-upload-vhd-windows-server
[Logon]: /documentation/articles/virtual-machines-log-on-windows-server
[Detach]: /documentation/articles/storage-windows-detach-disk
[Storage]: /documentation/articles/storage-create-storage-account/

<!---HONumber=Mooncake_1221_2015-->