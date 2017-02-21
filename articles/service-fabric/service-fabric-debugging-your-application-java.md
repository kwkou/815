<properties
    pageTitle="在 Eclipse 中调试 Azure Service Fabric 应用程序 | Azure"
    description="通过在本地开发群集上采用 Eclipse 进行开发和调试，来提高服务的可靠性和性能。"
    services="service-fabric"
    documentationcenter=".net"
    author="vturecek"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="cb888532-bcdb-4e47-95e4-bfbb1f644da4"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/01/2016"
    wacn.date="02/20/2017"
    ms.author="vturecek;mikhegn" />  


# 使用 Eclipse 调试 Java Service Fabric 应用程序
>[AZURE.SELECTOR]
- [Visual Studio/CSharp](/documentation/articles/service-fabric-debugging-your-application/)
- [Eclipse/Java](/documentation/articles/service-fabric-debugging-your-application-java/)

1. 按[设置 Service Fabric 开发环境](/documentation/articles/service-fabric-get-started-linux/)中的步骤创建本地开发群集。

2. 更新要调试的服务的 entryPoint.sh，让其使用远程调试参数启动 java 进程。可在以下位置找到此文件：``ApplicationName\ServiceNamePkg\Code\entrypoint.sh``。设置了端口 8001，以便在此示例中进行调试。

    
    	java -Xdebug -Xrunjdwp:transport=dt_socket,address=8001,server=y,suspend=y -Djava.library.path=$LD_LIBRARY_PATH -jar myapp.jar
    
3. 更新应用程序清单，将要调试服务的实例计数或副本计数设置为 1。此设置可避免用于调试的端口发生冲突。例如，对于无状态服务，设置 ``InstanceCount="1"``；对于有状态服务，将目标和最小副本集大小设置为 1，如下所示：`` TargetReplicaSetSize="1" MinReplicaSetSize="1"``。

4. 部署应用程序。

5. 在 Eclipse IDE 中，选择“运行”-\>“调试配置”-\>“远程 Java 应用程序和输入连接属性”，然后将属性设置如下：

   
	   主机：ipaddress 
	   端口：8001
   
6.  在所需点处设置断点并调试应用程序。

如果应用程序发生故障，则可能还需要启用 coredump。在 shell 中执行 ``ulimit -c``，如果返回 0，则 coredump 未启用。若要启用不受限制的 coredump，请执行以下命令：``ulimit -c unlimited``。还可以使用命令 ``ulimit -a`` 验证状态。若要更新 coredump 生成路径，请执行 ``echo '/tmp/core_%e.%p' | sudo tee /proc/sys/kernel/core_pattern``。

### 后续步骤

* [使用 Linux Azure 诊断收集日志](/documentation/articles/service-fabric-diagnostics-how-to-setup-lad/)。
* [在本地监视和诊断服务](/documentation/articles/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally-linux/)。

<!---HONumber=Mooncake_0213_2017-->