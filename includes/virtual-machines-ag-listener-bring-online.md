1. 导航回故障转移群集管理器。展开“角色”，然后突出显示可用性组。在“资源”选项卡上，右键单击侦听器名称，然后单击“属性”。

1. 选择“依赖项”选项卡。如果有多个资源列出，请验证 IP 地址具有 OR 而不是 AND 依赖项。单击**“确定”**。

1. 右键单击侦听器名称，然后单击“联机”。

1. 侦听器处于联机状态后，在“资源”选项卡上，右键单击可用性组，然后单击“属性”。

	![配置可用性组资源](./media/virtual-machines-sql-server-configure-alwayson-availability-group-listener/IC678772.gif)

1. 在侦听器名称资源（而不是 IP 地址资源名称）上创建依赖项。单击“确定”。

	![在侦听器名称上添加依赖项](./media/virtual-machines-sql-server-configure-alwayson-availability-group-listener/IC678773.gif)

1. 启动 SQL Server Management Studio 并连接到主副本。

1. 导航到“AlwaysOn 高可用性”|“可用性组”|“<AvailabilityGroupName>”|“可用性组侦听器”。

3. 你现在应看到在故障转移群集管理器中创建的侦听器名称。右键单击侦听器名称，然后单击“属性”。

1. 在“端口”框中，通过使用先前使用过的 $EndpointPort 为可用性组侦听器指定端口号（在本教程中，默认值是 1433），然后单击“确定”。

<!---HONumber=70-->