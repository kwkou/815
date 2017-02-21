<properties
    pageTitle="将启动节点 API 和停止节点 API 替换为 Azure Service Fabric 节点转换 API | Azure"
    description="将启动节点 API 和停止节点 API 替换为 Azure Service Fabric 节点转换 API"
    services="service-fabric"
    documentationcenter=".net"
    author="LMWF"
    manager="rsinha"
    editor="" />
<tags
    ms.assetid="f4e70f6f-cad9-4a3e-9655-009b4db09c6d"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="12/19/2016"
    wacn.date="02/20/2017"
    ms.author="lemai" />  


# 将启动节点 API 和停止节点 API 替换为节点转换 API

## 停止节点 API 和启动节点 API 有什么作用？

停止节点 API（在托管环境中为 [StopNodeAsync\(\)][stopnode]，在 PowerShell 中为 [Stop-ServiceFabricNode][stopnodeps]）可停止 Service Fabric 节点。Service Fabric 节点是进程，而不是 VM 或计算机 – VM 或计算机仍会运行。在本文档的余下部分中，“节点”是指 Service Fabric 节点。停止某个节点会将其置于*已停止*状态，此时，该节点不再是群集的成员，也无法托管服务，因此相当于一个*关闭的*节点。这种做法有助于将故障注入系统，对应用程序进行测试。启动节点 API（在托管环境中为 [StartNodeAsync\(\)][startnode]，在 PowerShell 中为 [Start-ServiceFabricNode][startnodeps]\]）与停止节点 API 相反，可使节点恢复正常状态。

## 为什么要替换这些 API？

如前所述，*停止的* Service Fabric 节点是使用停止节点 API 有针对性地停止的节点。*关闭的*节点是出于其他任何原因（例如，VM 或计算机已关闭）而关闭的节点。使用停止节点 API 时，系统不会公开信息来区分*停止的*节点和*关闭的*节点。

此外，这些 API 返回的某些错误并不是那么浅显易懂。例如，在已*停止的*节点上调用停止节点 API 将返回错误 *InvalidAddress*。这种体验可以得到改善。

此外，在调用启动节点 API 之前，节点停止的持续时间为“无限期”。我们发现，这可能会导致问题且容易出错。例如，如果用户在节点上调用停止节点 API，但后来忘记了这件事，则就会出现问题。以后无法确定该节点的状态到底是*关闭*还是*停止*。


## 节点转换 API 简介

我们凭借一组新的 API 解决了上述问题。新的节点转换 API（在托管环境中为 [StartNodeTransitionAsync\(\)][snt]）可用于将 Service Fabric 节点转换为*停止*状态，或将其从*停止*状态转换为正常的启动状态。请注意，该 API 的名称中的“Start”不是指启动节点，而是指系统开始执行异步操作，将节点转换为*停止*或启动状态。

**使用情况**

如果调用节点转换 API 后未引发异常，则表示系统已接受异步操作，将执行该操作。成功的调用并不意味着操作已完成。若要获取有关当前操作状态的信息，请结合针对此操作调用节点转换 API 时使用的 guid 调用节点转换进度 API（在托管环境中为 [GetNodeTransitionProgressAsync\(\)][gntp]）。节点转换进度 API 返回 NodeTransitionProgress 对象。此对象的 State 属性指定当前操作状态。如果状态为“Running”，则表示该操作正在执行。如果状态为“Completed”，则表示该操作已完成且未出错。如果状态为“Faulted”，则表示执行该操作时出现问题。Result 属性的 Exception 属性将指示具体的问题。有关 State 属性的详细信息，请参阅 [TestCommandProgressState Enum](https://docs.microsoft.com/dotnet/api/system.fabric.testcommandprogressstate)，以及以下“示例用法”部分中的代码示例。


**区分停止的节点和关闭的节点**如果使用节点转换 API *停止*某个节点，节点查询（在托管环境中为 [GetNodeListAsync\(\)][nodequery]，在 PowerShell 中为 [Get-ServiceFabricNode][nodequeryps]）的输出将显示此节点的 *IsStopped* 属性值为 true。请注意，这与 *NodeStatus* 属性的值不同，后者显示 *Down*。如果 *NodeStatus* 属性的值为 *Down*，但 *IsStopped* 为 false，则表示未使用节点转换 API 停止该节点，而是出于其他原因而使该节点处于*关闭*状态。如果 *IsStopped* 属性为 true，*NodeStatus* 属性为 *Down*，则表示已使用节点转换 API 停止该节点。

使用节点转换 API 启动*停止的*节点可让它重新成为群集的正常成员。节点查询 API 的输出将显示 *IsStopped* 为 false，*NodeStatus* 为除 Down 以外的某个值（例如 Up）。


**有限持续时间**使用节点转换 API 停止某个节点时，必须使用 *stopNodeDurationInSeconds* 参数，表示将该节点保持*停止*状态的秒数。此值必须在允许的范围内，最小为 600，最大为 14400。经过这段时间后，该节点将自行重新启动，自动进入 Up（启动）状态。有关用法示例，请参阅下面的“示例 1”。

> [AZURE.WARNING]
避免将节点转换 API 与停止节点 API 和启动节点 API 混合使用。建议只使用节点转换 API。\> 如果已使用停止节点 API 停止某个节点，应先使用启动节点 API 将它启动，\> 然后再使用节点转换 API。

> [AZURE.WARNING]
不能同时针对同一个节点执行多次节点转换 API 调用。否则，节点转换 API 将 \> 引发 FabricException 并返回 ErrorCode 属性值 NodeTransitionInProgress。在特定的节点上启动 \> 节点转换后，应等到该操作进入终止状态（Completed、Faulted 或 ForceCancelled），\> 然后在同一节点上启动新的转换。允许同时针对不同的节点执行节点转换调用。


#### 示例用法


**示例 1** - 以下示例使用节点转换 API 停止节点。


        // Helper function to get information about a node
        static Node GetNodeInfo(FabricClient fc, string node)
        {
            NodeList n = null;
            while (n == null)
            {
                n = fc.QueryManager.GetNodeListAsync(node).GetAwaiter().GetResult();
                Task.Delay(TimeSpan.FromSeconds(1)).GetAwaiter();
            };

            return n.FirstOrDefault();
        }

        static async Task WaitForStateAsync(FabricClient fc, Guid operationId, TestCommandProgressState targetState)
        {
            NodeTransitionProgress progress = null;

            do
            {
                bool exceptionObserved = false;
                try
                {
                    progress = await fc.TestManager.GetNodeTransitionProgressAsync(operationId, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                }
                catch (OperationCanceledException oce)
                {
                    Console.WriteLine("Caught exception '{0}'", oce);
                    exceptionObserved = true;
                }
                catch (FabricTransientException fte)
                {
                    Console.WriteLine("Caught exception '{0}'", fte);
                    exceptionObserved = true;
                }

                if (!exceptionObserved)
                {
                    Console.WriteLine("Current state of operation '{0}': {1}", operationId, progress.State);

                    if (progress.State == TestCommandProgressState.Faulted)
                    {
                        // Inspect the progress object's Result.Exception.HResult to get the error code.
                        Console.WriteLine("'{0}' failed with: {1}, HResult: {2}", operationId, progress.Result.Exception, progress.Result.Exception.HResult);

                        // ...additional logic as required
                    }

                    if (progress.State == targetState)
                    {
                        Console.WriteLine("Target state '{0}' has been reached", targetState);
                        break;
                    }
                }

                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);
            }
            while (true);
        }

        static async Task StopNodeAsync(FabricClient fc, string nodeName, int durationInSeconds)
        {
            // Uses the GetNodeListAsync() API to get information about the target node
            Node n = GetNodeInfo(fc, nodeName);

            // Create a Guid
            Guid guid = Guid.NewGuid();

            // Create a NodeStopDescription object, which will be used as a parameter into StartNodeTransition
            NodeStopDescription description = new NodeStopDescription(guid, n.NodeName, n.NodeInstanceId, durationInSeconds);

            bool wasSuccessful = false;

            do
            {
                try
                {
                    // Invoke StartNodeTransitionAsync with the NodeStopDescription from above, which will stop the target node.  Retry transient errors.
                    await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                    wasSuccessful = true;
                }
                catch (OperationCanceledException oce)
                {
                    // This is retryable
                }
                catch (FabricTransientException fte)
                {
                    // This is retryable
                }

                // Backoff
                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);
            }
            while (!wasSuccessful);

            // Now call StartNodeTransitionProgressAsync() until hte desired state is reached.
            await WaitForStateAsync(fc, guid, TestCommandProgressState.Completed).ConfigureAwait(false);
        }


**示例 2** - 以下示例启动*已停止的*节点。该示例使用第一个示例中的某些帮助器方法。


        static async Task StartNodeAsync(FabricClient fc, string nodeName)
        {
            // Uses the GetNodeListAsync() API to get information about the target node
            Node n = GetNodeInfo(fc, nodeName);

            Guid guid = Guid.NewGuid();
            BigInteger nodeInstanceId = n.NodeInstanceId;

            // Create a NodeStartDescription object, which will be used as a parameter into StartNodeTransition
            NodeStartDescription description = new NodeStartDescription(guid, n.NodeName, nodeInstanceId);

            bool wasSuccessful = false;

            do
            {
                try
                {
                    // Invoke StartNodeTransitionAsync with the NodeStartDescription from above, which will start the target stopped node.  Retry transient errors.
                    await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                    wasSuccessful = true;
                }
                catch (OperationCanceledException oce)
                {
                    Console.WriteLine("Caught exception '{0}'", oce);
                }
                catch (FabricTransientException fte)
                {
                    Console.WriteLine("Caught exception '{0}'", fte);
                }

                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);

            }
            while (!wasSuccessful);

            // Now call StartNodeTransitionProgressAsync() until hte desired state is reached.
            await WaitForStateAsync(fc, guid, TestCommandProgressState.Completed).ConfigureAwait(false);
        }


**示例 3** - 以下示例显示错误的用法。这种用法之所以不正确，是因为提供的 *stopDurationInSeconds* 超出了允许的范围。由于 StartNodeTransitionAsync\(\) 将会失败并返回严重错误，因此该操作未被接受。不应调用进度 API。此示例使用第一个示例中的某些帮助器方法。


        static async Task StopNodeWithOutOfRangeDurationAsync(FabricClient fc, string nodeName)
        {
            Node n = GetNodeInfo(fc, nodeName);

            Guid guid = Guid.NewGuid();

            // Use an out of range value for stopDurationInSeconds to demonstrate error
            NodeStopDescription description = new NodeStopDescription(guid, n.NodeName, n.NodeInstanceId, 99999);

            try
            {
                await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
            }

            catch (FabricException e)
            {
                Console.WriteLine("Caught {0}", e);
                Console.WriteLine("ErrorCode {0}", e.ErrorCode);
                // Output:
                // Caught System.Fabric.FabricException: System.Runtime.InteropServices.COMException (-2147017629)
                // StopDurationInSeconds is out of range ---> System.Runtime.InteropServices.COMException: Exception from HRESULT: 0x80071C63
                // << Parts of exception omitted>>
                //
                // ErrorCode InvalidDuration
            }
        }


**示例 4** - 以下示例演示当节点转换 API 发起的操作被接受，但随后在执行过程中失败时，节点转换进度 API 将返回的错误信息。在本例中，失败的原因是节点转换 API 尝试启动一个不存在的节点。此示例使用第一个示例中的某些帮助器方法。


        static async Task StartNodeWithNonexistentNodeAsync(FabricClient fc)
        {
            Guid guid = Guid.NewGuid();
            BigInteger nodeInstanceId = 12345;

            // Intentionally use a nonexistent node
            NodeStartDescription description = new NodeStartDescription(guid, "NonexistentNode", nodeInstanceId);

            bool wasSuccessful = false;

            do
            {
                try
                {
                    // Invoke StartNodeTransitionAsync with the NodeStartDescription from above, which will start the target stopped node.  Retry transient errors.
                    await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                    wasSuccessful = true;
                }
                catch (OperationCanceledException oce)
                {
                    Console.WriteLine("Caught exception '{0}'", oce);
                }
                catch (FabricTransientException fte)
                {
                    Console.WriteLine("Caught exception '{0}'", fte);
                }

                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);

            }
            while (!wasSuccessful);

            // Now call StartNodeTransitionProgressAsync() until the desired state is reached.  In this case, it will end up in the Faulted state since the node does not exist.
            // When StartNodeTransitionProgressAsync()'s returned progress object has a State if Faulted, inspect the progress object's Result.Exception.HResult to get the error code.
            // In this case, it will be NodeNotFound.
            await WaitForStateAsync(fc, guid, TestCommandProgressState.Faulted).ConfigureAwait(false);
        }


[stopnode]: https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.faultmanagementclient?redirectedfrom=MSDN#System_Fabric_FabricClient_FaultManagementClient_StopNodeAsync_System_String_System_Numerics_BigInteger_System_Fabric_CompletionMode_
[stopnodeps]: https://msdn.microsoft.com/zh-cn/library/mt125982.aspx
[startnode]: https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.faultmanagementclient?redirectedfrom=MSDN#System_Fabric_FabricClient_FaultManagementClient_StartNodeAsync_System_String_System_Numerics_BigInteger_System_String_System_Int32_System_Fabric_CompletionMode_System_Threading_CancellationToken_
[startnodeps]: https://msdn.microsoft.com/zh-cn/library/mt163520.aspx
[nodequery]: https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.queryclient#System_Fabric_FabricClient_QueryClient_GetNodeListAsync_System_String_
[nodequeryps]: https://docs.microsoft.com/powershell/servicefabric/vlatest/Get-ServiceFabricNode?redirectedfrom=msdn
[snt]: https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.testmanagementclient#System_Fabric_FabricClient_TestManagementClient_StartNodeTransitionAsync_System_Fabric_Description_NodeTransitionDescription_System_TimeSpan_System_Threading_CancellationToken_
[gntp]: https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.testmanagementclient#System_Fabric_FabricClient_TestManagementClient_GetNodeTransitionProgressAsync_System_Guid_System_TimeSpan_System_Threading_CancellationToken_

<!---HONumber=Mooncake_0213_2017-->