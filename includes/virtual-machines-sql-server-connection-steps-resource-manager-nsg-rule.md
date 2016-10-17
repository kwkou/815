### 为 VM 配置网络安全组入站规则

如果你希望能够通过 Internet 连接到 SQL Server，则需为 SQL Server 实例所侦听的端口配置网络安全组入站规则。默认情况下，该端口为 TCP 端口 1433。

1. 在门户中选择“虚拟机”，然后选择 SQL Server VM。

3. 再选择“网络接口”。

	![网络接口](./media/virtual-machines-sql-server-connection-steps/rm-network-interface.png)  


4. 然后选择 VM 的网络接口。

4. 单击“网络安全组”链接。

	![网络接口](./media/virtual-machines-sql-server-connection-steps/rm-network-security-group.png)  


6. 在网络安全组的属性中，展开“入站安全规则”。

5. 单击“添加”按钮。

6. 提供“SQLServerPublicTraffic”作为“名称”。

7. 将“协议”更改为“TCP”。

8. 将“目标端口范围”指定为 1433（或 SQL Server 实例正在侦听的端口）。

9. 确认已将“操作”设置为“允许”。安全规则对话框应类似于以下屏幕快照。

	![网络安全规则](./media/virtual-machines-sql-server-connection-steps/rm-network-security-rule.png)  


9. 单击“确定”保存 VM 的规则。

>[AZURE.NOTE] 可以将另一个网络安全组与子网相关联（独立于 VM 上的网络安全组）。默认情况下不执行此操作；但是，如果在子网上创建了网络安全组，则必须子网和 VM 的网络安全组上的端口 1433。

<!---HONumber=Mooncake_1010_2016-->