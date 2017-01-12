Azure load balancer 是位于第 4 层 (TCP, UDP) 的负载均衡器。该负载均衡器可以在云服务或负载均衡器集的虚拟机中运行状况良好的服务实例之间分配传入流量，从而提供高可用性。Azure Load Balancer 还可以在多个端口和/或多个 IP 地址上显示这些服务。

可以将负载均衡器配置为用于：

* 对传入到虚拟机 (VM) 的 Internet 流量进行平衡负载。我们将此方案中的负载均衡器作为一个[面向 Internet 的负载均衡器](/documentation/articles/load-balancer-internet-overview/)。
* 对虚拟网络 (VNet) 和云服务中 VM 之间的流量或本地计算机和跨界虚拟网络中 VM 之间的流量进行平衡负载。我们将此方案中的负载均衡器作为一个[内部负载均衡器 (ILB)](/documentation/articles/load-balancer-internal-overview/)。
* 将外部流量转发到特定的 VM 实例。

<!---HONumber=Mooncake_0926_2016-->