<properties
    pageTitle="池调整大小开始事件 - Azure | Azure"
    caps.latest.revision="6"
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
    ms.assetid="8bd33e8b-6390-4a34-95dc-2e63d8bce941"
    ms.author="tamram"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="633134ee3186d09c8aec5cf4ce4ee9f447d1fb9f"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="pool-resize-start-event"></a>池调整大小开始事件
池调整大小开始事件日志正文

## <a name="remarks"></a>备注
 当池调整大小开始时，会发出此事件。 由于池调整大小是异步事件，预计在调整大小操作完成后会发出池调整大小完成事件。

 以下示例显示了池调整大小开始事件（即通过手动调整大小将池的大小从 0 调整为 2 个节点）的正文。

    {
        "poolId": "myPool1",
        "nodeDeallocationOption": "invalid",
        "currentDedicated": 0,
        "targetDedicated": 2,
        "enableAutoScale": false,
        "isAutoPool": false
    }

|元素|类型|说明|
|-------------|----------|-----------|
|poolId|String|池的 id。|
|nodeDeallocationOption|String|指定何时从池中删除节点（如果池的大小正在减小）。<br /><br /> 可能的值包括：<br /><br /> **requeue** - 终止正在运行的任务并将其重新排队。 当作业启用时，任务将再次运行。 一旦任务终止，便会立即删除节点。<br /><br /> **terminate** - 终止正在运行的任务。 任务将不会再次运行。 一旦任务终止，便会立即删除节点。<br /><br /> **taskcompletion** - 允许完成当前正在运行的任务。 等待时不计划任何新任务。 在所有任务完成时，删除节点。<br /><br /> **Retaineddata** - 允许完成当前正在运行的任务，然后等待所有任务数据保留期到期。 等待时不计划任何新任务。 在所有任务保留期都已过期时，删除节点。<br /><br /> 默认值为 requeue。<br /><br /> 如果池的大小正在增加，该值将设置为**无效**。|
|currentDedicated|Int32|当前分配到池的计算节点数。|
|targetDedicated|Int32|池请求的计算节点数。|
|enableAutoScale|Bool|指定池大小是否随时间自动调整。|
|isAutoPool|Bool|指定是否已通过作业的 AutoPool 机制创建池。|

