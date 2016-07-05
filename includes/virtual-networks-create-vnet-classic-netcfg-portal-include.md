##<a name="how-to-create-a-vnet-using-a-network-config-file-in-the-azure-portal"></a> 如何在 Azure 经典管理门户中使用网络配置文件创建 VNet

Azure 使用 xml 文件定义可用于订阅的所有 VNet。可以下载此文件，然后编辑它以修改或删除现有 VNet 并创建新 VNet。在本文档中，你将了解如何下载此文件（称为网络配置（或 netcgf）文件），并编辑它以创建新 VNet。查看 [Azure 虚拟网络配置架构](https://msdn.microsoft.com/zh-cn/library/azure/jj157100.aspx)以了解有关网络配置文件的详细信息。

若要通过 Azure 经典管理门户使用 netcfg 文件创建 VNet，请执行下面的步骤。

1. 从浏览器导航到 http://manage.windowsazure.cn，如有必要，请使用 Azure 帐户登录。
2. 向下滚动服务列表，并单击**“网络”**，如下所示。

	![Azure 虚拟网络](./media/virtual-networks-create-vnet-classic-portal-xml-include/vnet-create-portal-netcfg-figure1.gif)

3. 在页面底部，单击**“导出”**按钮，如下所示。

	![“导出”按钮](./media/virtual-networks-create-vnet-classic-portal-xml-include/vnet-create-portal-netcfg-figure2.png)

4. 在**“导出网络配置”**页上，选择要从中导出虚拟网络配置的订阅，然后单击页面左下角的复选标记按钮。
5. 按照浏览器说明进行操作以保存 **NetworkConfig.xml** 文件。请确保记下保存该文件的位置。
6. 使用任何 XML 或文本编辑器应用程序打开前面步骤 5 中保存的文件，并查找 **<VirtualNetworkSites>** 元素。如果你已创建任何网络，每个网络将显示为其自身的 **<VirtualNetworkSite>** 元素。
7. 若要创建此方案中所述的虚拟网络，请在 **<VirtualNetworkSites>** 元素的正下方添加以下 XML：

		<VirtualNetworkSite name="TestVNet" Location="China North">
		  <AddressSpace>
		    <AddressPrefix>192.168.0.0/16</AddressPrefix>
		  </AddressSpace>
		  <Subnets>
		    <Subnet name="FrontEnd">
		      <AddressPrefix>192.168.1.0/24</AddressPrefix>
		    </Subnet>
		    <Subnet name="BackEnd">
		      <AddressPrefix>192.168.2.0/24</AddressPrefix>
		    </Subnet>
		  </Subnets>
		</VirtualNetworkSite>

8.  保存网络配置文件。
9.  在 Azure 经典管理门户中的页面左下角单击**“新建”**，然后依次单击**“网络服务”**、**“虚拟网络”**、**“导入配置”**，如下图所示。

	![导入配置](./media/virtual-networks-create-vnet-classic-portal-xml-include/vnet-create-portal-netcfg-figure3.gif)

10.  在**“导入网络配置文件”**页上，单击**“浏览文件...”**，然后导航到你在前面步骤 8 中保存文件的文件夹，选择该文件，然后单击**“打开”**。网页应如下图所示。在页面右下角，单击箭头按钮以转到下一步。

	![“导入网络配置文件”页](./media/virtual-networks-create-vnet-classic-portal-xml-include/vnet-create-portal-netcfg-figure4.png)

11.   在**“构建网络”**页上，可看到新 VNet 的条目，如下图所示。

	![“构建网络”页](./media/virtual-networks-create-vnet-classic-portal-xml-include/vnet-create-portal-netcfg-figure5.png)

12.   单击页面右下角的复选标记按钮以创建 VNet。几秒钟后，你的 VNet 将显示在可用 VNet 列表中，如下图所示。

	![新建虚拟网络](./media/virtual-networks-create-vnet-classic-portal-xml-include/vnet-create-portal-netcfg-figure6.png)

<!---HONumber=69-->