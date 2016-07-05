## 为 VM 创建备份保管库

备份保管库是存储所有按时间创建的备份和恢复点的实体。备份保管库还包含将应用到要备份的虚拟机的备份策略。

下图显示了各种 Azure 备份实体之间的关系：
![Azure 备份实体和关系](./media/backup-create-vault-for-vms/vault-policy-vm.png)

创建备份保管库的步骤：

1. 登录到 [Azure 经典管理门户](http://manage.windowsazure.cn/)。

    >[AZURE.NOTE] 如果你的订阅上次是在经典管理门户中使用的，则你的订阅可能会在经典管理门户中打开。在此情况下，若要创建备份保管库，请单击“新建”>“数据服务”>“恢复服务”>“备份保管库”>“快速创建”（请参阅下图）。

    ![创建备份保管库](./media/backup-create-vault-for-vms/backup_vaultcreate.png)

3. 对于“名称”，输入一个友好名称以标识此保管库。名称对于 Azure 订阅需要是唯一的。键入包含 2 到 50 个字符的名称。名称必须以字母开头，只能包含字母、数字和连字符。

4. 在“区域”中，为保管库选择地理区域。保管库必须与你要保护的虚拟机位于同一区域中。如果你在多个区域中具有虚拟机，则必须在每个区域中创建备份保管库。无需指定存储帐户即可存储备份数据 — 备份保管库和 Azure 备份服务会自动处理这种情况。

5. 在“订阅”中，选择要与备份保管库关联的订阅。仅当组织帐户与多个 Azure 订阅关联时，才会有多个选项。

5. 单击“创建保管库”。创建备份保管库可能需要一段时间。可以在经典管理门户底部监视状态通知。

    ![创建保管库 toast 通知](./media/backup-create-vault-for-vms/creating-vault.png)

6. 一条消息将确认已成功创建保管库。并且将在“恢复服务”页上将保管库列出为“活动”保管库。确保在创建保管库后立即选择适当的存储冗余选项。阅读有关[在备份保管库中设置存储冗余选项](backup-configure-vault.md#azure-backup---storage-redundancy-options)的更多内容。

    ![备份保管库列表](./media/backup-create-vault-for-vms/backup_vaultslist.png)

7. 单击备份保管库将转到“快速启动”页，其中会显示 Azure 虚拟机的备份说明。

    ![“仪表板”页中的虚拟机备份说明](./media/backup-create-vault-for-vms/vmbackup-instructions.png)

<!---HONumber=Mooncake_0530_2016-->