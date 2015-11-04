在此步骤中，你使用同一网络上运行的客户端应用程序测试可用性组侦听器。

对客户端连接，请注意以下要求：

- 对于与侦听器的客户端连接，其源计算机所驻留的云服务必须不同于托管 AlwaysOn 可用性副本的云服务。

- 如果 AlwaysOn 副本位于不同子网中，客户端必须在连接字符串中指定“MultisubnetFailover=True”。这会导致尝试并行连接到不同子网中的副本。请注意，这种情况下包括跨区域 AlwaysOn 可用性组部署。

例如，从同一个 Azure VNet 的一个虚拟机（而不是托管副本的虚拟机）连接到侦听器。完成此测试一个简单的方法是尝试将 SSMS 连接到可用性组侦听器。另一种简单方法是运行 [SQLCMD.exe](https://technet.microsoft.com/zh-cn/library/ms162773.aspx)，如下所示：

		sqlcmd -S "<ListenerName>,<EndpointPort>" -d "<DatabaseName>" -Q "select @@servername, db_name()" -l 15

> [AZURE.NOTE]如果 EndpointPort 值为 1433，则不需要在调用时指定它。前面的调用还假定客户端计算机加入了同一个域，并且调用方已被授予使用 windows 身份验证访问数据库的权限。

在测试侦听器时，请务必对可用性组进行故障转移，以确保客户端可以在故障转移之间连接到侦听器。

<!---HONumber=70-->