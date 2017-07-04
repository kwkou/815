## <a name="Scenario"></a>方案
创建具有一个 NIC 的 VM，并连接到虚拟网络。 VM 需要三个不同的专用 IP 地址和两个公共 IP 地址。 IP 地址将分配到以下 IP 配置：

* **IPConfig-1：**分配一个静态专用 IP 地址和一个静态公共 IP 地址。
* **IPConfig-2：**分配一个静态专用 IP 地址和一个静态公共 IP 地址。
* **IPConfig-3：**分配一个静态专用 IP 地址，不分配公共 IP 地址。

    ![多个 IP 地址](./media/virtual-network-multiple-ip-addresses-scenario/multiple-ipconfigs.png)

IP 配置在创建 NIC 时关联到 NIC，NIC 在创建 VM 时附加到 VM。 本方案使用的 IP 地址类型用于演示目的。 可分配需要的任何 IP 地址和分配类型。

> [AZURE.NOTE]
> 尽管本文中的步骤将所有 IP 配置都分配给一个 NIC，但也可将多个 IP 配置分配给多 NIC VM 中的任何 NIC。 若要了解如何创建具有多个 NIC 的 VM，请阅读[创建具有多个 NIC 的 VM](/documentation/articles/virtual-network-deploy-multinic-arm-ps/)一文。