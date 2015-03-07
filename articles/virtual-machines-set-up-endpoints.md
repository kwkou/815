<properties linkid="manage-windows-howto-setup-endpoints" urlDisplayName="Set up endpoints" pageTitle="设置在 Azure 中的虚拟机上的终结点" metaKeywords="Azure config setup, configuring vm connection" description="了解如何设置与在 Azure 中虚拟机的通信。" metaCanonical="" services="virtual-machines" documentationCenter="" title="" authors="" solutions="" manager="" editor="" />


#如何设置虚拟机的终结点

**请注意**：如果您想要通过主机名或设置跨界连接直接连接到虚拟机，请参阅[Azure 虚拟网络概述](http://msdn.microsoft.com/library/azure/jj156007.aspx)。

在 Azure 中创建的所有虚拟机可以自动进行都通信通过专用网络通道与同一云服务或虚拟网络中其他虚拟机。但是，Internet 上的其他资源或其他虚拟网络需要终结点才能处理虚拟机的入站网络流量。 

当您在管理门户中创建虚拟机时，可创建用于远程桌面、Windows PowerShell 远程处理或安全 Shell (SSH) 等的终结点。创建虚拟机后，您可根据需要创建其他终结点。还可通过为终结点的访问控制列表 (ACL) 配置规则来管理公用端口的传入流量。本文向您演示如何执行这两项任务。

每个终结点都有一个公用端口和一个专用端口：

- 专用端口是由内部使用虚拟机来侦听该终结点上的流量。

- 公用端口由 Azure 负载平衡器用于从外部资源的虚拟机进行通信。创建终结点后，您可使用网络访问控制列表 (ACL) 定义帮助隔离和控制公用端口上的传入流量的规则。有关详细信息，请参阅[关于网络访问控制列表](http://msdn.microsoft.com/library/azure/dn376541.aspx)。

通过管理门户创建终结点时，将提供这些终结点的端口和协议的默认值。对于所有其他终结点，将在创建终结点时指定端口和协议。可使用 TCP 或 UDP 协议将资源连接到终结点。TCP 协议包括 HTTP 和 HTTPS 通信。  

**重要**：防火墙配置为端口关联的 Windows PowerShell 远程处理的使用远程桌面和安全外壳 (SSH)，并且在大多数情况下，自动完成。对于为所有其他终结点指定的端口，将不会为来宾操作系统的防火墙中自动进行任何配置。创建终结点时，您将需要在防火墙中配置相应的端口才能允许您计划通过终结点路由的流量。

###创建终结点 # # #

1. 如果尚未这样做，登录到[Azure 管理门户](http://manage.windowsazure.cn)。

2. 单击**虚拟机**，然后选择你想要配置的虚拟机。

3. 单击**终结点**。"终结点"页将列出虚拟机的所有终结点。

	![Endpoints](./media/virtual-machines-set-up-endpoints/endpointswindows.png)

4.	单击**添加**。

	**添加终结点**对话框随即出现。选择终结点的类型。如果您正在创建新的负载平衡集，选择独立若要在集中创建第一个终结点。
	
5. 在**名称**，键入终结点的名称。

6. 在协议中，指定 **TCP** 或 **UDP**。

7. 在**公用端口**和**专用端口**，键入您想要使用的端口号。这些端口号可以是不同的。公用端口是用于从在 Azure 外部的通信的入口点，并由 Azure 负载平衡器。可以使用专用端口和防火墙规则在虚拟机上重定向以一种适合于您的应用程序的通信。

8. 单击**创建负载平衡集**如果此终结点将在负载平衡集中的第一个。然后，在**配置负载平衡集**页上，指定名称、 协议和探测详细信息。负载平衡集需要探测，这样才能监视此集的运行状况。有关详细信息，请参阅[虚拟机负载平衡](/zh-cn/documentation/articles/virtual-machines-load-balance/).  

9.	单击复选标记以创建终结点。

	现在，您将看到终结点上列出**终结点**页。

	![Endpoint creation successful](./media/virtual-machines-set-up-endpoints/endpointwindowsnew.png)

###管理终结点 # # # 上的 ACL

按照下列步骤在终结点上添加、修改或删除 ACL。

**请注意**：如果终结点是负载平衡集的一部分，对一个终结点上的 ACL 进行任何更改都应用到集中的行的所有终结点。

1. 如果尚未这样做，登录到[Azure 管理门户](http://manage.windowsazure.cn)。

2. 单击**虚拟机**，然后选择你想要配置的虚拟机。

3. 单击**终结点**。"终结点"页将列出虚拟机的所有终结点。

    ![ACL list](./media/virtual-machines-set-up-endpoints/EndpointsShowsDefaultEndpointsForVM.png)

4. 从列表中选择适当的终结点。 

5. 单击**管理 ACL**。

    **指定 ACL 详细信息**对话框随即出现。

    ![Specify ACL details](./media/virtual-machines-set-up-endpoints/EndpointACLdetails.png)

6. 使用列表中的行为 ACL 添加、删除或编辑规则。远程子网值对应于您可作为规则接受或拒绝的 IP 地址范围。将按以第一条规则开始、以最后一条规则结束的顺序来计算规则。这意味着规则应按限制从少到多的顺序列出。有关示例和详细信息，请参阅[关于网络访问控制列表](http://msdn.microsoft.com/library/azure/dn376541.aspx)。
