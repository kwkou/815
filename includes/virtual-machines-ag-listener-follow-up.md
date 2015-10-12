创建可用性组侦听器之后，可能需要调整侦听器资源的 RegisterAllProvidersIP 和 HostRecordTTL 群集参数。这些参数可以减少故障转移后的重新连接时间，这样可以防止连接超时。有关这些参数以及示例代码的详细信息，请参阅[创建或配置可用性组侦听器](https://msdn.microsoft.com/zh-cn/library/hh213080.aspx#MultiSubnetFailover)。

<!---HONumber=70-->