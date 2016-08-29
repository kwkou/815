可使用负载平衡器为 Azure 中的工作负荷提供可用性。Azure Load Balancer 是第 4 层（TCP、UDP）类型的负载平衡器，可在云服务或在负载平衡器集中定义的虚拟机中运行状况良好的服务实例之间分发传入流量。
 
可配置负载平衡器为。

- 对传入到虚拟机 (VM) 的 Internet 流量进行平衡负载。我们将此方案中的负载平衡器作为一个[面向 Internet 的负载平衡器](/documentation/articles/load-balancer-internet-overview/)。
- 对虚拟网络 (VNet) 和云服务中 VM 之间的流量或本地计算机和跨界虚拟网络中 VM 之间的流量进行平衡负载。我们将此方案中的负载平衡器作为一个[内部负载平衡器 (ILB)](/documentation/articles/load-balancer-internal-overview/)。
- 	将外部流量转发到特定的 VM 实例。

<!---HONumber=Mooncake_0822_2016-->