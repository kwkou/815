<properties
    pageTitle="池删除完成事件 - Azure | Azure"
    caps.latest.revision="5"
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
    ms.assetid="580a1278-5740-4143-826c-7d875ef127ff"
    ms.author="tamram"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="4e30609cd4adc2dc7875b5016c6ace48e8ad235a"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="pool-delete-complete-event"></a>池删除完成事件
池删除完成事件日志正文

## <a name="remarks"></a>备注
 当完成池删除操作时，会发出此事件。

 以下示例演示了池删除完成事件的正文。

    {
        "id": "myPool1",
        "startTime": "2016-09-09T22:13:48.579Z",
        "endTime": "2016-09-09T22:14:08.836Z"
    }

|元素|类型|说明|
|-------------|----------|-----------|
|id|String|池的 id。|
|startTime|DateTime|池删除开始的时间。|
|endTime|DateTime|池删除完成的时间。|

## <a name="remarks"></a>备注
有关池调整大小操作的状态和错误代码的详细信息，请参阅[从帐户中删除池](https://docs.microsoft.com/zh-cn/rest/api/batchservice/delete-a-pool-from-an-account)。

