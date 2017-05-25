1. 在[门户](http://portal.azure.cn)中，导航到要为其创建虚拟网关的 Resource Manager 虚拟网络。
2. 在 VNet 边栏选项卡的“设置”部分中，单击“子网”以展开“子网”边栏选项卡。
3. 在“子网”边栏选项卡中，单击“+网关子网”打开“添加子网”边栏选项卡。 

    ![添加网关子网](./media/vpn-gateway-add-gwsubnet-rm-portal-include/addgwsubnet.png "添加网关子网")
4. 子网的“名称”自动填充为值“GatewaySubnet”。 Azure 需要此值才能识别作为网关子网的子网。 调整自动填充的**地址范围**值，使其匹配配置要求。

    ![添加子网](./media/vpn-gateway-add-gwsubnet-rm-portal-include/addsubnetgw.png "添加子网")
5. 若要创建子网，请单击边栏选项卡底部的“确定”。