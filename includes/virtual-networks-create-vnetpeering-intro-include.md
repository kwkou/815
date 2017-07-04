<!-- not suitable for Mooncake -->

VNet 对等互连是一种通过 Azure 主干网络在同一区域连接两个虚拟网络的机制。一旦对等后，两个虚拟网络将类似于用于所有连接的单一虚拟网络。若不熟悉 VNet 对等互连，请参阅 [VNet 对等互连概述](/documentation/articles/virtual-network-peering-overview/)。

VNet 对等互连提供公共预览版，须用以下命令注册后才可使用：

    Register-AzureRmProviderFeature -FeatureName AllowVnetPeering -ProviderNamespace Microsoft.Network

    Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Network

<!---HONumber=Mooncake_0919_2016-->