<properties
    pageTitle="任务完成事件 - Azure | Azure"
    caps.latest.revision="4"
    author="tamram"
    manager="timlt" />
<tags
    ms.custom=""
    ms.date="2017-02-01"
    wacn.date="05/15/2017"
    ms.prod="azure"
    ms.reviewer=""
    ms.service="batch"
    ms.suite=""
    ms.tgt_pltfrm=""
    ms.topic="reference"
    ms.assetid="9dcc468b-e0a7-4b80-bec8-ffd466afdc8a"
    ms.author="tamram"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="bded25681d42bf79850eb0831f8a889f38f98dbf"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="task-complete-event"></a>任务完成事件
任务完成事件日志正文

## <a name="remarks"></a>备注
 无论退出代码如何，任务完成后都会发出此事件。 此事件可用于确定任务的持续时间、运行位置以及是否重试过。


 以下示例显示了任务完成事件的正文。

    {
        "jobId": "job-0000000001",
        "id": "task-5",
        "taskType": "User",
        "systemTaskVersion": 0,
        "nodeInfo": {
            "poolId": "pool-001",
            "nodeId": "tvm-257509324_1-20160908t162728z"
        },
        "multiInstanceSettings": {
            "numberOfInstances": 1
        },
        "constraints": {
            "maxTaskRetryCount": 2
        },
        "executionInfo": {
            "startTime": "2016-09-08T16:32:23.799Z",
            "endTime": "2016-09-08T16:34:00.666Z",
            "exitCode": 0,
            "retryCount": 0,
            "requeueCount": 0
        }
    }

|元素名称|类型|说明|
|------------------|----------|-----------|
|jobId|String|包含任务的作业的 id。|
|id|String|任务的 id。|
|taskType|String|任务的类型。 它可以是“JobManager”（指示它是作业管理器任务），也可以是“User”（指示它并非作业管理器任务）。 对于作业准备任务、作业释放任务或开始任务，不会发出此事件。|
|systemTaskVersion|Int32|这是任务上的内部重试计数器。 批处理服务可能会在内部重试任务来解决暂时性问题。 这些问题可能包括内部计划错误或尝试恢复处于错误状态的计算节点。|
|[nodeInfo](#nodeInfo)|复杂类型|包含有关运行任务的计算节点的信息。|
|[multiInstanceSettings](#multiInstanceSettings)|复杂类型|指定任务是需要多个计算节点的多实例任务。  有关详细信息，请参阅 [multiInstanceSettings](https://docs.microsoft.com/zh-cn/rest/api/batchservice/get-information-about-a-task)。|
|[constraints](#constraints)|复杂类型|应用到此任务的执行约束。|
|[executionInfo](#executionInfo)|复杂类型|包含有关任务执行的信息。|

###  <a name="nodeInfo"></a> nodeInfo

|元素名称|类型|说明|
|------------------|----------|-----------|
|poolId|String|运行任务的池的 id。|
|nodeId|String|运行任务的节点的 id。|

###  <a name="multiInstanceSettings"></a> multiInstanceSettings

|元素名称|类型|说明|
|------------------|----------|-----------|
|numberOfInstances|Int32|任务所需的计算节点数。|

###  <a name="constraints"></a> constraints

|元素名称|类型|说明|
|------------------|----------|-----------|
|maxTaskRetryCount|Int32|可以重试任务的最大次数。 批处理服务在其退出代码非零时重试任务。<br /><br /> 请注意，此值专门用于控制重试的次数。 批处理服务将尝试任务一次，然后重试，直至达到此上限为止。 例如，如果最大重试计数为 3，则批处理任务最多尝试任务 4 次（一次是初始尝试，其余 3 次是重试）。<br /><br /> 如果最大重试计数为 0，则批处理服务不会重试任务。<br /><br /> 如果最大重试计数为 -1，则批处理服务会无限制地重试任务。<br /><br /> 默认值为 0（不重试）。|

###  <a name="executionInfo"></a> executionInfo

|元素名称|类型|说明|
|------------------|----------|-----------|
|startTime|DateTime|任务开始运行的时间。 “Running”对应于**正在运行**状态，因此如果任务指定资源文件或应用程序包，则开始时间反映了任务开始下载或部署这些内容的时间。  如果任务已重启或重试，该时间是任务开始运行的最近时间。|
|endTime|DateTime|任务完成的时间。|
|exitCode|Int32|任务的退出代码。|
|retryCount|Int32|批处理服务重试任务的次数。 如果任务使用非零退出代码退出，该任务会重试，直至达到指定的 MaxTaskRetryCount。|
|requeueCount|Int32|批处理服务因用户请求而对任务进行重新排队的次数。<br /><br /> 当用户从池中删除节点（通过调整池的大小或缩小池）或作业已禁用时，用户可以指定节点上运行的任务重新排队等待执行。 此计数跟踪由于这些原因而重新排队任务的次数。|

