<properties
    pageTitle="在 Azure Service Fabric 中帮助保护服务的通信 | Azure"
    description="概述如何帮助保护在 Azure Service Fabric 群集中运行的 Reliable Services 的通信。"
    services="service-fabric"
    documentationcenter="java"
    author="PavanKunapareddyMSFT"
    manager="timlt" />
<tags
    ms.assetid=""
    ms.service="service-fabric"
    ms.devlang="java"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="required"
    ms.date="03/09/2017"
    wacn.date="05/15/2017"
    ms.author="pakunapa"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="e33d62ec8570780dac5e5e3831c78f27e94bb3cb"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="help-secure-communication-for-services-in-azure-service-fabric"></a>在 Azure Service Fabric 中帮助保护服务的通信
> [AZURE.SELECTOR]
- [Windows 上的 C#](/documentation/articles/service-fabric-reliable-services-secure-communication/)
- [Linux 上的 Java](/documentation/articles/service-fabric-reliable-services-secure-communication-java/)

## <a name="help-secure-a-service-when-youre-using-service-remoting"></a>使用服务远程处理时帮助保护服务
我们将使用一个现有[示例](/documentation/articles/service-fabric-reliable-services-communication-remoting-java/)来解释如何为 Reliable Services 设置远程处理。 若要在使用服务远程处理时帮助保护服务，请遵循以下步骤：

1. 创建接口 `HelloWorldStateless`，用于定义可供服务的远程过程调用使用的方法。 服务将使用 `microsoft.serviceFabric.services.remoting.fabricTransport.runtime` 包中声明的 `FabricTransportServiceRemotingListener`。 这是可以提供远程处理功能的 `CommunicationListener` 实现。

        public interface HelloWorldStateless extends Service {
            CompletableFuture<String> getHelloWorld();
        }

        class HelloWorldStatelessImpl extends StatelessService implements HelloWorldStateless {
            @Override
            protected List<ServiceInstanceListener> createServiceInstanceListeners() {
                ArrayList<ServiceInstanceListener> listeners = new ArrayList<>();
                listeners.add(new ServiceInstanceListener((context) -> {
                    return new FabricTransportServiceRemotingListener(context,this);
                }));
            return listeners;
            }

            public CompletableFuture<String> getHelloWorld() {
                return CompletableFuture.completedFuture("Hello World!");
            }
        }

2. 添加侦听器设置和安全凭据。

    确保你要用来帮助保护服务通信的证书安装在群集中的所有节点上。 有两种方式可用于提供侦听器设置和安全凭据：

   1. 使用[配置包](/documentation/articles/service-fabric-application-model/)提供：

       在 settings.xml 文件中添加 `TransportSettings` 节。

           <!--Section name should always end with "TransportSettings".-->
           <!--Here we are using a prefix "HelloWorldStateless".-->
            <Section Name="HelloWorldStatelessTransportSettings">
                <Parameter Name="MaxMessageSize" Value="10000000" />
                <Parameter Name="SecurityCredentialsType" Value="X509_2" />
                <Parameter Name="CertificatePath" Value="/path/to/cert/BD1C71E248B8C6834C151174DECDBDC02DE1D954.crt" />
                <Parameter Name="CertificateProtectionLevel" Value="EncryptandSign" />
                <Parameter Name="CertificateRemoteThumbprints" Value="BD1C71E248B8C6834C151174DECDBDC02DE1D954" />
            </Section>


       在这种情况下，`createServiceInstanceListeners` 方法如下所示：

            protected List<ServiceInstanceListener> createServiceInstanceListeners() {
                ArrayList<ServiceInstanceListener> listeners = new ArrayList<>();
                listeners.add(new ServiceInstanceListener((context) -> {
                    return new FabricTransportServiceRemotingListener(context,this, FabricTransportRemotingListenerSettings.loadFrom(HelloWorldStatelessTransportSettings));
                }));
                return listeners;
            }

        如果将在 settings.xml 中添加 `TransportSettings` 节而不添加任何前缀，`FabricTransportListenerSettings` 将默认加载此节中的所有设置。

            <!--"TransportSettings" section without any prefix.-->
            <Section Name="TransportSettings">
                ...
            </Section>

        在这种情况下，`CreateServiceReplicaListeners` 方法如下所示：

            protected List<ServiceInstanceListener> createServiceInstanceListeners() {
                ArrayList<ServiceInstanceListener> listeners = new ArrayList<>();
                listeners.add(new ServiceInstanceListener((context) -> {
                    return new FabricTransportServiceRemotingListener(context,this);
                }));
                return listeners;
            }

3. 在安全服务上使用远程堆栈（而不是使用 `microsoft.serviceFabric.services.remoting.client.ServiceProxyBase` 类）调用方法来创建服务代理时，请使用 `microsoft.serviceFabric.services.remoting.client.FabricServiceProxyFactory`。

    如果客户端代码正在作为服务的一部分运行，则可以从 settings.xml 文件中加载 `FabricTransportSettings`。 创建与服务代码类似的 TransportSettings 节，如上所示。 对客户端代码进行以下更改。

        FabricServiceProxyFactory serviceProxyFactory = new FabricServiceProxyFactory(c -> {
                return new FabricTransportServiceRemotingClientFactory(FabricTransportRemotingSettings.loadFrom("TransportPrefixTransportSettings"), null, null, null, null);
            }, null)

        HelloWorldStateless client = serviceProxyFactory.createServiceProxy(HelloWorldStateless.class,
            new URI("fabric:/MyApplication/MyHelloWorldService"));

        CompletableFuture<String> message = client.getHelloWorld();