## <a name="x-sub"></a>跨订阅进行对等互连
在此方案中，将创建存在于不同订阅中的两个 VNet 间的对等互连。

![跨订阅方案](./media/virtual-networks-create-vnetpeering-scenario-crosssub-include/figure01.PNG)  


VNet 对等互连依赖基于角色的访问控制 (RBAC) 进行授权。对于跨订阅方案，需要先向将创建对等链接的用户授予充足权限。

> [AZURE.NOTE]
如果同一用户具有两个订阅的特权，可跳过下面的步骤 1-4。
> 
>

<!---HONumber=Mooncake_0320_2017-->