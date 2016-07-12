<properties
	pageTitle="确定 Azure 备份存储冗余选项 | Azure"
	description="了解异地冗余存储和本地冗余存储之间的差异，以确定要使用哪个 Azure 备份存储冗余选项。"
	services="backup"
	documentationCenter=""
	authors="Jim-Parker"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="backup"
	ms.date="02/05/2016"
	wacn.date="04/12/2016"/>


# Azure 备份的存储冗余选项

你的业务需求确定了 Azure 备份后端存储的相应存储冗余选项。如果你要使用 Azure 作为主要备份存储终结点（例如，你要从 Windows Server 备份到 Azure），应考虑选择（默认的）异地冗余存储选项。如果使用 Azure 作为第三级备份存储终结点（例如，你正在使用 SCDPM 在本地创建本地备份复制，使用 Azure 满足长期数据保留需求），应考虑选择本地冗余存储。

## 异地冗余存储 (GRS)

异地冗余存储保留数据的六个副本。使用 GRS 时，你的数据将在主区域内复制三次，并且还在离主区域数百英里的辅助区域中复制三次，从而提供最高级别的持久性。当主区域中发生故障时，Azure 备份会将数据存储在 GRS 中，从而确保你的数据在两个独立的区域中持久保存。

## 本地冗余存储 (LRS)

本地冗余存储保留数据的三个副本。LRS 将在单个区域中的单个设施内复制三次。LRS 可以保护你的数据免受普通的硬件故障损害，但无法保护你的数据免受整个 Azure 设施故障的损害。这可以降低在 Azure 中存储数据的成本，但提供的数据持久性更低，不过，对于第三级副本是可接受的。

你可以根据需要从备份保管库的“配置”选项中选择相应的选项。

## 后续步骤

- 确保你的环境[已准备好备份 Windows 服务器或客户端计算机](/documentation/articles/backup-configure-vault/)
- 如果仍有疑问，请查看 [Azure 备份常见问题](/documentation/articles/backup-azure-backup-faq/)。
- 访问 [Azure 备份论坛](http://go.microsoft.com/fwlink/p/?LinkId=290933)

<!---HONumber=Mooncake_0405_2016-->