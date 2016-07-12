资源|默认限制|最大限制
---|---|---
[每个部署的 Web/辅助角色数](/documentation/articles/cloud-services-choose-me/)<sup>1</sup>|25|25
每个部署的[实例输入终结点数](http://msdn.microsoft.com/zh-cn/library/gg557552.aspx#InstanceInputEndpoint)|25|25
每个部署的[输入终结点数](http://msdn.microsoft.com/zh-cn/library/gg557552.aspx#InputEndpoint)|25|25
每个部署的[内部终结点数](http://msdn.microsoft.com/zh-cn/library/gg557552.aspx#InternalEndpoint)|25|25

<sup>1</sup>包含 Web/辅助角色的每个云服务可以有两个部署，一个用于生产，一个用于过渡。另请注意，此限制与不同的角色数目（配置）相关，而不是与每个角色的实例数目（缩放）相关。

<!---HONumber=Mooncake_0530_2016-->