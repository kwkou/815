<properties
    pageTitle="自动缩放 Azure Batch 池中的计算节点 | Azure"
    description="对云池启用自动缩放功能可以动态调整池中计算节点的数目。"
    services="batch"
    documentationcenter=""
    author="tamram"
    manager="timlt"
    editor="tysonn" />

<tags
    ms.assetid="c624cdfc-c5f2-4d13-a7d7-ae080833b779"
    ms.service="batch"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="multiple"
    ms.date="01/23/2017"
    wacn.date="03/14/2017"
    ms.author="tamram" />  


# 自动缩放 Azure Batch 池中的计算节点
使用自动缩放，Azure Batch 可以根据所定义的参数在池中动态添加或删除计算节点。还可以通过自动调整你的应用程序使用的计算能力（当工作的任务要求提高时增加节点，当任务要求降低时删除节点）来节省时间和金钱。

你可以通过将定义的*自动缩放公式*与计算节点池相关联（例如，使用 [Batch .NET](/documentation/articles/batch-dotnet-get-started/) 库中的 [PoolOperations.EnableAutoScale][net_enableautoscale] 方法），对计算节点池启用自动缩放。然后，Batch 服务将使用此公式来确定执行工作负荷所需的计算节点数目。Batch 将会响应定期收集的服务指标数据样本，并根据公式按可配置的间隔调整池中的计算节点数。

可以在创建池时启用自动缩放，也可以对现有池启用该功能。你还可以更改已启用“自动缩放”功能的池的现有公式。Batch 可让你在将公式分配给池之前先评估公式，以及监视自动缩放运行的状态。

## <a name="automatic-scaling-formulas"></a>自动缩放公式

自动缩放公式是你定义的字符串值，其中包含分配给池的 [autoScaleFormula][rest_autoscaleformula] 元素 (Batch REST) 或 [CloudPool.AutoScaleFormula][net_cloudpool_autoscaleformula] 属性 (Batch .NET) 的一个或多个语句。将公式分配到池后，Batch 服务将使用公式来确定池中可供下一个处理间隔使用的目标计算节点数（稍后将详细说明间隔）。公式是一个字符串，其大小不能超过 8 KB，最多可以包含 100 个以分号分隔的语句，可以包括换行符和注释。

可以将自动缩放公式视为使用 Batch 自动缩放“语言”。 公式语句是自由形式的表达式，可以包括服务定义的变量（由 Batch 服务定义的变量）和用户定义的变量（你定义的变量）。公式语句可以通过内置类型、运算符和函数对这些值执行各种操作。例如，语句可以采用以下格式：


	$myNewVariable = function($ServiceDefinedVariable, $myCustomVariable);


公式通常包含多个语句，这些语句对先前语句中获取的值执行操作。例如，首先获取 `variable1` 的值，然后将其传递给一个函数来填充 `variable2`：

	$variable1 = function1($ServiceDefinedVariable);
	$variable2 = function2($OtherServiceDefinedVariable, $variable1);

在公式中使用这些语句的目标是实现池要缩放到的计算节点数目--**专用节点**的**目标**数目。此数目可以大于、小于或等于池中当前的节点数目。Batch 按特定的时间间隔（下文讨论了[自动缩放时间间隔](#automatic-scaling-interval)）对池的自动缩放公式求值。然后，它会将池中节点的目标数目调整成在求值时自动缩放公式所指定的数目。

举个简单的例子，以下两行自动缩放公式根据活动任务数目指定应该调整的节点数目（最多 10 个计算节点）：

	$averageActiveTaskCount = avg($ActiveTasks.GetSample(TimeInterval_Minute * 15));
	$TargetDedicated = min(10, $averageActiveTaskCount);

本文的后续部分将介绍构成自动缩放公式的各个实体，包括变量、运算符、操作和函数。你会了解如何在 Batch 中获取各种计算资源和任务度量值。你可以使用这些度量值，根据资源使用情况和任务状态对池的节点计数进行智能化调整。然后，你将了解如何使用 Batch REST 和 .NET API 构建公式以及对池启用自动缩放。最后，我们将讨论几个示例公式。

> [AZURE.IMPORTANT]
每个 Azure Batch 帐户都受限于可以用于处理的最大核心数（以及由此确定的计算节点数）。Batch 服务最多将创建达到该核心数限制的节点数。因此，它可能达不到公式所指定的目标计算节点数。请参阅 [Azure Batch 服务的配额和限制](/documentation/articles/batch-quota-limit/)了解有关查看和提高帐户配额的信息。
> 
> 

## <a name="variables"></a>变量
可以在自动缩放公式中使用**服务定义的**变量和**用户定义的**变量。服务定义的变量内置在 Batch 服务中，有些是可读写的，有些是只读的。用户定义的变量指的是*你*定义的变量。在上面的两行示例公式中，`$TargetDedicated` 是服务定义的变量，而 `$averageActiveTaskCount` 是用户定义的变量。

下表显示了 Batch 服务定义的读写和只读变量。

可以**获取**和**设置**这些服务定义的变量的值，以管理池中计算节点的数目：

| 读写服务定义的变量 | 说明 |
| --- | --- |
| $TargetDedicated |池的**专用计算节点**的**目标**数。这是池应该缩放到的计算节点数目。它是一个“目标”数目，因为池可能达不到此目标节点数目。如果在池达到初始目标之前节点的目标数目被后续的自动缩放评估再次修改，则可能会发生这种情况。如果在达到节点的目标数目之前达到了 Batch 帐户的节点或核心配额，则也可能会发生这种情况。 |
| $NodeDeallocationOption |从池中删除计算节点时发生的操作。可能的值为：<ul><li>**requeue**-- 立即终止任务并将其放回作业队列，以便对其进行重新安排。<li>**terminate**-- 立即终止任务并从作业队列中删除它们。<li>**taskcompletion**-- 等待当前运行的任务完成，然后从池中删除节点。<li>**retaineddata**-- 从池中删除节点之前等待清理节点上的所有本地保留的任务的数据。</ul> |

可以**获取**这些服务定义的变量的值，以根据 Batch 服务中的指标进行调整：

| 只读服务定义的变量 | 说明 |
| --- | --- |
| $CPUPercent |CPU 使用率的平均百分比。 |
| $WallClockSeconds |使用的秒数。 |
| $MemoryBytes |使用的平均 MB 数。 |
| $DiskBytes |本地磁盘上使用的平均 GB 数。 |
| $DiskReadBytes |读取的字节数。 |
| $DiskWriteBytes |写入的字节数。 |
| $DiskReadOps |执行的读取磁盘操作数。 |
| $DiskWriteOps |执行的写入磁盘操作数。 |
| $NetworkInBytes |入站字节数。 |
| $NetworkOutBytes |出站字节数。 |
| $SampleNodeCount |计算节点数。 |
| $ActiveTasks |处于活动状态的任务数。 |
| $RunningTasks |处于运行状态的任务数。 |
| $PendingTasks |$ActiveTasks 和 $RunningTasks 的总和。 |
| $SucceededTasks |成功完成的任务数。 |
| $FailedTasks |失败的任务数。 |
| $CurrentDedicated |当前的专用计算节点数。 |

> [AZURE.TIP]
上面所示的服务定义的只读变量是一些*对象*，它们提供了各种方法来访问与其相关的数据。有关详细信息，请参阅下面的[获取样本数据](#getsampledata)。
> 
> 

## 类型
公式支持以下**类型**：

- double
- doubleVec
- doubleVecList
- 字符串
- timestamp--timestamp 是包含以下成员的复合结构：
  
  - year
  - month (1-12)
  - day (1-31)
  - weekday（采用数字格式，例如 1 表示星期一）
  - hour（采用 24 时制数字格式，例如 13 表示下午 1 点）
  - minute (00-59)
  - second (00-59)
- timeinterval
  
  - TimeInterval\_Zero
  - TimeInterval\_100ns
  - TimeInterval\_Microsecond
  - TimeInterval\_Millisecond
  - TimeInterval\_Second
  - TimeInterval\_Minute
  - TimeInterval\_Hour
  - TimeInterval\_Day
  - TimeInterval\_Week
  - TimeInterval\_Year

## 操作
上面列出的类型允许以下**操作**。

| 操作 | 支持的运算符 | 结果类型 |
| --- | --- | --- |
| double *operator* double |+, -, *, / |double |
| double *operator* timeinterval |* |timeinterval |
| doubleVec *operator* double |+, -, *, / |doubleVec |
| doubleVec *operator* doubleVec |+, -, *, / |doubleVec |
| timeinterval *operator* double |*, / |timeinterval |
| timeinterval *operator* timeinterval |+, - |timeinterval |
| timeinterval *operator* timestamp |+ |timestamp |
| timestamp *operator* timeinterval |+ |timestamp |
| timestamp *operator* timestamp | - | timeinterval | 
| *operator*double | -, ! | double | 
| *operator*timeinterval | - | timeinterval | 
| double *operator* double | <, <=, ==, >=, >, != | double | 
| string *operator* string | <, <=, ==, >=, >, != | double | 
| timestamp *operator* timestamp | <, <=, ==, >=, >, != | double | 
| timeinterval *operator* timeinterval | <, <=, ==, >=, >, != | double | 
| double *operator* double | &&, || | double |


使用三元运算符 (`double ? statement1 : statement2`) 测试双精度值时，非零值为 **true**，零值为 **false**。

## 函数
可以使用以下预定义**函数**来定义自动缩放公式。

| 函数 | 返回类型 | 说明 |
| --- | --- | --- |
| avg(doubleVecList) |double |返回 doubleVecList 中所有值的平均值。 |
| len(doubleVecList) |double |返回从 doubleVecList 创建的向量的长度。 |
| lg(double) |double |返回 double 的对数底数 2。 |
| lg(doubleVecList) |doubleVec |返回 doubleVecList 的分量对数底数 2。必须为参数显式传递 vec(double)。否则会采用 double lg(double) 版本。 |
| ln(double) |double |返回 double 的自然对数。 |
| ln(doubleVecList) |doubleVec |返回 doubleVecList 的分量对数底数 2。必须为参数显式传递 vec(double)。否则会采用 double lg(double) 版本。 |
| log(double) |double |返回 double 的对数底数 10。 |
| log(doubleVecList) |doubleVec |返回 doubleVecList 的分量对数底数 10。对于单一的 double 参数，必须显式传递 vec(double)。否则会采用 double log(double) 版本。 |
| max(doubleVecList) |double |返回 doubleVecList 中的最大值。 |
| min(doubleVecList) |double |返回 doubleVecList 中的最小值。 |
| norm(doubleVecList) |double |返回从 doubleVecList 创建的向量的二范数。 |
| percentile(doubleVec v, double p) |double |返回向量 v 的百分位元素。 |
| rand() |double |返回介于 0.0 和 1.0 之间的随机值。 |
| range(doubleVecList) |double |返回 doubleVecList 中最小值和最大值之间的差。 |
| std(doubleVecList) |double |返回 doubleVecList 中值的样本标准偏差。 |
| stop() | |停止对自动缩放表达式求值。 |
| sum(doubleVecList) |double |返回 doubleVecList 的所有组成部分之和。 |
| time(string dateTime="") |timestamp |如果未传递参数，则返回当前时间的时间戳；如果传递了参数，则返回 dateTime 字符串的时间戳。支持的 dateTime 格式为 W3C-DTF 和 RFC 1123。 |
| val(doubleVec v, double i) |double |返回在起始索引为零的向量 v 中，位置 i 处的元素的值。 |

上表中描述的某些函数可以接受列表作为参数。逗号分隔列表为 *double* 和 *doubleVec* 的任意组合。例如：

`doubleVecList := ( (double | doubleVec)+(, (double | doubleVec) )* )?`  


*doubleVecList* 值在计算之前将转换为单个 *doubleVec*。例如，如果 `v = [1,2,3]`，则调用 `avg(v)` 相当于调用 `avg(1,2,3)`。调用 `avg(v, 7)` 相当于调用 `avg(1,2,3,7)`。

## <a name="getsampledata"></a>获取样本数据
自动缩放公式使用 Batch 服务提供的度量值数据（样本）。公式根据服务所提供的值来扩大或缩小池的大小。上述服务定义的变量是可提供各种方法来访问与该对象关联的数据的对象。例如，以下表达式显示了一个用于获取过去五分钟 CPU 使用率的请求：


	$CPUPercent.GetSample(TimeInterval_Minute * 5)


| 方法 | 说明 |
| --- | --- |
| GetSample() |`GetSample()` 方法返回数据样本的向量。<br/><br/>一个样本为 30 秒的度量数据。换而言之，将每隔 30 秒获取一次样本。但如下所示，样本在收集后需经历一定的延迟才能供公式使用。在此情况下，并不是给定时间段内的所有样本均可用于公式的计算。<ul> <li> `doubleVec GetSample(double count)`<br/>指定要从最新收集的样本中获取的样本数。<br/><br/> `GetSample(1)`返回最后一个可用样本。但对于像 `$CPUPercent` 这样的度量值，不应使用此方法，因为无法知道样本是*何时*收集的。它可能是最近收集的，也可能由于系统问题而变得很旧。在此情况下，最好使用如下所示的时间间隔。<li> `doubleVec GetSample((timestamp or timeinterval) startTime [, double samplePercent])`<br/>指定用于收集样本数据的时间范围。此外，它还指定必须在请求的时间范围内提供的样本的百分比。<br/><br/>`$CPUPercent.GetSample(TimeInterval_Minute * 10)` 返回 20 个样本（如果 CPUPercent 历史记录中显示过去十分钟的所有样本）。但如果最后一分钟的历史记录不可用，则只返回 18 个样本。如果这样：<br/><br/>`$CPUPercent.GetSample(TimeInterval_Minute * 10, 95)` 将失败，因为只有 90% 的样本可用。<br/><br/>`$CPUPercent.GetSample(TimeInterval_Minute * 10, 80)` 将成功。<li>`doubleVec GetSample((timestamp or timeinterval) startTime, (timestamp or timeinterval) endTime [, double samplePercent])`<br/>指定用于收集数据的时间范围，包括开始时间和结束时间。<br/><br/>如上所述，样本在收集后需经历一定的延迟才能供公式使用。在使用 `GetSample` 方法时，必须考虑这个因素。请参阅下面的 `GetSamplePercent`。 |
| GetSamplePeriod() |返回在历史样本数据集中采样的期间。 |
| Count() |返回度量值历史记录中的样本总数。 |
| HistoryBeginTime() |返回度量值最旧可用数据样本的时间戳。 |
| GetSamplePercent() |返回给定时间间隔的可用样本百分比。例如：<br/><br/>`doubleVec GetSamplePercent( (timestamp or timeinterval) startTime [, (timestamp or timeinterval) endTime] )`<br/><br/>由于 `GetSample` 方法失败，因此如果返回的样本的百分比低于指定的 `samplePercent`，则可先使用 `GetSamplePercent` 方法进行检查。然后，如果存在的样本数量不足，你可以执行其他操作，无需停止自动缩放评估。 |

### 样本、样本百分比和 *GetSample()* 方法
自动缩放公式的核心操作是获取任务和资源度量值数据，然后根据该数据调整池大小。因此，请务必明确知道自动缩放公式如何与度量值数据或“样本”交互。

**示例**

Batch 服务定期获取任务和资源指标的*样本*，使其可供自动缩放公式使用。这些样本每 30 秒由 Batch 服务记录一次。但是，通常有一些滞后，以致记录样本的时间与样本可供自动缩放公式使用（与读取）的时间之间存在延迟。此外，由于各种因素（例如网络或其他基础结构问题），可能无法记录特定间隔的样本，从而导致样本“遗漏”。

**样本百分比**

将 `samplePercent` 传递到 `GetSample()` 方法，或调用 `GetSamplePercent()` 方法时，“percent”是指 Batch 服务*可能*记录的样本总数与自动缩放公式实际*可用*的样本数目之间的比较。

让我们以 10 分钟的时间跨度为例。由于每隔 30 秒记录样本一次，因此在 10 分钟的时间跨度内，Batch 服务所记录的样本总数将达到 20 个（每分钟 2 个）。但是，由于报告机制固有的延迟，或者 Azure 基础结构出现的其他一些问题，可能只有 15 个样本可供自动缩放公式读取。这意味着，在这 10 分钟的期间内，记录的样本总数只有 **75%** 实际可供公式使用。

**GetSample() 和样本范围**

你的自动缩放公式将对池进行扩大和缩小操作--添加节点或删除节点。由于节点耗费资金，你需要确保公式所使用的智能分析方法采用了足够的数据。因此，建议你在公式中使用趋势类型的分析。此类型的分析将根据所收集样本的*范围*来扩大和缩小你的池。

为此，请使用 `GetSample(interval look-back start, interval look-back end)` 返回样本的**向量**：


	$runningTasksSample = $RunningTasks.GetSample(1 * TimeInterval_Minute, 6 * TimeInterval_Minute);


Batch 评估上述代码行后，会以值的向量形式返回样本范围。例如：


	$runningTasksSample=[1,1,1,1,1,1,1,1,1,1];


收集样本向量后，便可使用 `min()`、`max()` 和 `avg()` 等函数从所收集的范围派生有意义的值。

为了提高安全性，如果特定时间段可用的样本数小于特定百分比，你可以强制公式求值*失败*。强制公式求值失败会指示 Batch 在无法提供指定百分比的样本数时停止进一步的公式求值--不更改池大小。若要指定求值成功所需的样本百分比，请将其指定为 `GetSample()` 的第三个参数。下面指定要求 75% 的样本：


	$runningTasksSample = $RunningTasks.GetSample(60 * TimeInterval_Second, 120 * TimeInterval_Second, 75);


此外，由于先前提到的样本可用性延迟问题，请务必记得指定回查开始时间早于一分钟的时间范围。这是由于样本需要花大约一分钟的时间才能传播到整个系统，因此通常无法使用 `(0 * TimeInterval_Second, 60 * TimeInterval_Second)` 范围内的样本。同样地，可以使用 `GetSample()` 百分比参数来强制实施特定样本百分比要求。

> [AZURE.IMPORTANT]
**强烈建议****不要*只*依赖于自动缩放公式中的 `GetSample(1)`**。这是因为，`GetSample(1)` 基本上只是向 Batch 服务表明：“不论多久以前获取最后一个样本，请将它提供给我。” 由于它只是单个样本，而且可能是较旧的样本，因此可能无法代表最近任务或资源状态的全貌。如果使用 `GetSample(1)`，请确保它是更大语句的一部分，而不是公式所依赖的唯一数据点。
> 
> 

## 度量值
在定义公式时，可以同时使用**资源**和**任务**度量值。可根据获取和求值的度量值数据对池中专用节点的目标数目进行调整。有关每个度量值的详细信息，请参阅上面的[变量](#variables)部分。

<table>
  <tr>
    <th>度量值</th>
    <th>说明</th>
  </tr>
  <tr>
    <td><b>资源</b></td>
    <td><p><b>资源度量值</b>基于计算节点的 CPU、带宽和内存使用量，以及节点数目。</p>
        <p> 这些服务定义的变量可用于根据节点计数进行调整：</p>
    <p><ul>
      <li>$TargetDedicated</li>
            <li>$CurrentDedicated</li>
            <li>$SampleNodeCount</li>
    </ul></p>
    <p>这些服务定义的变量可用于根据节点资源使用量进行调整：</p>
    <p><ul>
      <li>$CPUPercent</li>
      <li>$WallClockSeconds</li>
      <li>$MemoryBytes</li>
      <li>$DiskBytes</li>
      <li>$DiskReadBytes</li>
      <li>$DiskWriteBytes</li>
      <li>$DiskReadOps</li>
      <li>$DiskWriteOps</li>
      <li>$NetworkInBytes</li>
      <li>$NetworkOutBytes</li></ul></p>
  </tr>
  <tr>
    <td><b>任务</b></td>
    <td><p><b>任务度量值</b>基于任务的状态，例如“活动”、“挂起”和“已完成”。以下服务定义的变量可用于根据任务度量值调整池大小：</p>
    <p><ul>
      <li>$ActiveTasks</li>
      <li>$RunningTasks</li>
      <li>$PendingTasks</li>
      <li>$SucceededTasks</li>
            <li>$FailedTasks</li></ul></p>
        </td>
  </tr>
</table>

## 编写自动缩放公式
构造自动缩放公式时，可以使用上述组件来生成语句，然后将这些语句组合成完整的公式即可。本节中将创建一个自动缩放公式示例，用来执行一些实际的缩放决策。

首先，定义新自动缩放公式的要求。该公式应可以：

1. 如果 CPU 使用率高，则**增加**池中计算节点的目标数。
2. 如果 CPU 使用率低，则**减少**池中计算节点的目标数。
3. 始终将**最大**节点数限制为 400。

为了在 CPU 使用率高时*增加*节点数，我们定义了一个语句，仅当过去 10 分钟内最小平均 CPU 使用率高于 70% 时，该语句向用户定义变量 (`$totalNodes`) 填充一个值，值的大小为节点当前目标数的 110%。否则，使用当前的专用值。


	$totalNodes =
	    (min($CPUPercent.GetSample(TimeInterval_Minute * 10)) > 0.7) ?
	    ($CurrentDedicated * 1.1) : $CurrentDedicated;


若要在 CPU 使用率低时*减少*节点数，则在过去 60 分钟的平均 CPU 使用率低于 20% 时，公式中的下一个语句将同一 `$totalNodes` 变量设置为节点当前目标数的 90%。否则，使用上述语句中填充的 `$totalNodes` 的当前值。


	$totalNodes =
	    (avg($CPUPercent.GetSample(TimeInterval_Minute * 60)) < 0.2) ?
	    ($CurrentDedicated * 0.9) : $totalNodes;


现在，将专用计算节点的目标数限制为**最大值** 400：


	$TargetDedicated = min(400, $totalNodes)


下面是完整公式：


	$totalNodes =
	    (min($CPUPercent.GetSample(TimeInterval_Minute * 10)) > 0.7) ?
	    ($CurrentDedicated * 1.1) : $CurrentDedicated;
	$totalNodes =
	    (avg($CPUPercent.GetSample(TimeInterval_Minute * 60)) < 0.2) ?
	    ($CurrentDedicated * 0.9) : $totalNodes;
	$TargetDedicated = min(400, $totalNodes)


## 创建已启用自动缩放功能的池
若要创建启用自动缩放功能的新池，可使用以下方法之一：

**Batch .NET**

1. 使用 [BatchClient.PoolOperations.CreatePool](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.pooloperations.createpool.aspx) 创建池。
2. 将 [CloudPool.AutoScaleEnabled](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudpool.autoscaleenabled.aspx) 属性设为 `true`。
3. 使用自动缩放公式设置 [CloudPool.AutoScaleFormula](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudpool.autoscaleformula.aspx) 属性。
4. （可选）设置 [CloudPool.AutoScaleEvaluationInterval](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudpool.autoScaleevaluationinterval.aspx) 属性（默认值为 15 分钟）。
5. 使用 [CloudPool.Commit](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudpool.commit.aspx) 或 [CommitAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudpool.commitasync.aspx) 提交池。

**Batch REST API**

- [将池添加到帐户](https://msdn.microsoft.com/zh-cn/library/azure/dn820174.aspx)：在 REST API 请求中指定 `enableAutoScale` 和 `autoScaleFormula` 元素，以便在创建池时为其配置自动缩放功能。

下面的代码片段通过 [Batch .NET][net_api] 库创建启用自动缩放的池。该池的自动缩放公式将星期一的节点的目标数设置为 5，将其他星期的目标数设置为 1。[自动缩放时间间隔](#automatic-scaling-interval)设置为 30 分钟。在本文的此部分与其他 C# 代码段中，“myBatchClient”是适当初始化的 [BatchClient][net_batchclient] 实例。

csharp

	CloudPool pool = myBatchClient.PoolOperations.CreatePool("mypool", "3", "small");
	pool.AutoScaleEnabled = true;
	pool.AutoScaleFormula = "$TargetDedicated = (time().weekday == 1 ? 5:1);";
	pool.AutoScaleEvaluationInterval = TimeSpan.FromMinutes(30);
	pool.Commit();


除了 Batch REST API 和 .NET SDK，还可以在自动缩放中使用任何其他 [Batch SDK](/documentation/articles/batch-technical-overview/#batch-development-apis/)、[Batch PowerShell cmdlet](/documentation/articles/batch-powershell-cmdlets-get-started/) 和 [Batch CLI](/documentation/articles/batch-cli-get-started/)。

> [AZURE.IMPORTANT]
当创建启用自动缩放功能的池时，**不可以**指定 `targetDedicated` 参数。另外，如果希望手动调整启用自动缩放功能的池的大小（例如，使用 [BatchClient.PoolOperations.ResizePool][net_poolops_resizepool] 来调整），则必须先**禁用**该池的自动缩放功能，然后再调整池的大小。
> 
> 

### <a name="interval" id="automatic-scaling-interval"></a>自动缩放间隔
默认情况下，Batch 服务根据其自动缩放公式每隔 **15 分钟**调整池大小。但是，可使用以下池属性配置此间隔：

- [CloudPool.AutoScaleEvaluationInterval][net_cloudpool_autoscaleevalinterval] (Batch .NET)
- [autoScaleEvaluationInterval][rest_autoscaleinterval] (REST API)

最小间隔为 5 分钟，最大间隔为 168 小时。如果指定的间隔超出此范围，Batch 服务将返回“错误的请求(400)”错误。

> [AZURE.NOTE]
自动缩放目前不能以低于一分钟的时间响应更改，而只能在你运行工作负荷时逐步调整池大小。
> 
> 

## 启用现有池的自动缩放功能
如果已使用 *targetDedicated* 参数创建具有一定数量的计算节点的池，仍可以启用池的自动缩放功能。每个 Batch SDK 提供“启用自动缩放”操作，例如：

- [BatchClient.PoolOperations.EnableAutoScale][net_enableautoscale] (Batch .NET)
- [启用池的自动缩放功能][rest_enableautoscale] (REST API)

启用现有池的自动缩放功能时，适用以下说明：

- 如果在发出“启用自动缩放”请求时已**禁用**池的自动缩放功能，那么在发出该请求时*必须*指定有效的自动缩放公式。你可以*选择性*地指定自动缩放评估时间间隔。如果未指定时间间隔，则使用默认值 15 分钟。
- 如果当前已**启用**池的自动缩放功能，则可以指定自动缩放公式和/或评估时间间隔。不能同时省略这两个属性。
  
  - 如果指定新的自动缩放评估时间间隔，那么将停止现有的评估计划，并启动新的计划。新计划的开始时间是发出“启用自动缩放”请求的时间。
  - 如果忽略自动缩放公式或评估时间间隔，那么 Batch 服务将继续使用该设置的当前值。

> [AZURE.NOTE]
如果某个值是在创建池时为 *targetDedicated* 参数指定的，则会在评估自动缩放公式时忽略该值。
> 
> 

此 C# 代码片段使用 [Batch .NET][net_api] 库启用现有池的自动缩放功能：

csharp

	// Define the autoscaling formula. This formula sets the target number of nodes
	// to 5 on Mondays, and 1 on every other day of the week
	string myAutoScaleFormula = "$TargetDedicated = (time().weekday == 1 ? 5:1);";

	// Set the autoscale formula on the existing pool
	myBatchClient.PoolOperations.EnableAutoScale(
		"myexistingpool",
		autoscaleFormula: myAutoScaleFormula);


### 更新自动缩放公式

使用相同的“启用自动缩放”请求*更新*现有的已启用自动缩放的池的公式（例如，使用 Batch.NET 中的 [EnableAutoScale][net_enableautoscale]）。未提供专门的“更新自动缩放”操作。例如，如果在执行以下代码时“myexistingpool”已启用自动缩放功能，那么该池的自动缩放公式将被替换为 `myNewFormula` 的内容。

csharp

	myBatchClient.PoolOperations.EnableAutoScale(
		"myexistingpool",
		autoscaleFormula: myNewFormula);



### 更新自动缩放的时间间隔

和更新自动缩放公式一样，使用相同的 [EnableAutoScale][net_enableautoscale] 方法更改现有的已启用自动缩放的池的自动缩放评估时间间隔。例如，将已启用自动缩放功能的池的自动缩放评估时间间隔设为 60 分钟：

csharp

	myBatchClient.PoolOperations.EnableAutoScale(
	    "myexistingpool",
	    autoscaleEvaluationInterval: TimeSpan.FromMinutes(60));


## 计算自动缩放公式

在将公式应用于池之前可以对公式进行计算。这样，就可以在将公式投入生产之前对公式执行“测试运行”以便查看公式的语句是如何进行计算的。

若要计算自动缩放公式，必须先通过**有效的公式****启用池的自动缩放功能**。如果想要测试还没有启用自动缩放功能的池的公式，则在首次启用自动缩放功能时可以使用单行公式 `$TargetDedicated = 0`。然后，使用以下任何一个方法来计算想要测试的公式：

- [BatchClient.PoolOperations.EvaluateAutoScale](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.pooloperations.evaluateautoscale.aspx) 或 [EvaluateAutoScaleAsync](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.pooloperations.evaluateautoscaleasync.aspx)
  
    这两种 Batch.NET 方法需要现有池的 ID 和包含要计算的自动缩放公式的字符串。计算结果包含在返回的 [AutoScaleEvaluation](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.autoscaleevaluation.aspx) 实例中。
- [计算自动缩放公式](https://msdn.microsoft.com/zh-cn/library/azure/dn820183.aspx)
  
    在此 REST API 请求中，指定 URI 中的池 ID，以及请求正文的 *autoScaleFormula* 元素中的自动缩放公式。操作的响应包含任何可能与该公式相关的错误信息。

在此 [Batch .NET][net_api] 代码片段中，我们先对公式进行计算，然后将其应用到 [CloudPool][net_cloudpool]。如果池尚未启用自动缩放功能，则首先启用它。

csharp

	// First obtain a reference to an existing pool
	CloudPool pool = batchClient.PoolOperations.GetPool("myExistingPool");

	// If autoscaling isn't already enabled on the pool, enable it.
	// You can't evaluate an autoscale formula on non-autoscale-enabled pool.
	if (pool.AutoScaleEnabled == false)
	{
	    // We need a valid autoscale formula to enable autoscaling on the
	    // pool. This formula is valid, but won't resize the pool:
	    pool.EnableAutoScale(
	        autoscaleFormula: $"$TargetDedicated = {pool.CurrentDedicated};",
	        autoscaleEvaluationInterval: TimeSpan.FromMinutes(5));

	    // Batch limits EnableAutoScale calls to once every 30 seconds.
	    // Because we want to apply our new autoscale formula below if it
	    // evaluates successfully, and we *just* enabled autoscaling on
	    // this pool, we pause here to ensure we pass that threshold.
	    Thread.Sleep(TimeSpan.FromSeconds(31));

	    // Refresh the properties of the pool so that we've got the
	    // latest value for AutoScaleEnabled
	    pool.Refresh();
	}

	// We must ensure that autoscaling is enabled on the pool prior to
	// evaluating a formula
	if (pool.AutoScaleEnabled == true)
	{
	    // The formula to evaluate - adjusts target number of nodes based on
	    // day of week and time of day
	    string myFormula = @"
	        $curTime = time();
	        $workHours = $curTime.hour >= 8 && $curTime.hour < 18;
	        $isWeekday = $curTime.weekday >= 1 && $curTime.weekday <= 5;
	        $isWorkingWeekdayHour = $workHours && $isWeekday;
	        $TargetDedicated = $isWorkingWeekdayHour ? 20:10;
	    ";

	    // Perform the autoscale formula evaluation. Note that this does not
	    // actually apply the formula to the pool.
	    AutoScaleRun eval =
	        batchClient.PoolOperations.EvaluateAutoScale(pool.Id, myFormula);

	    if (eval.Error == null)
	    {
	        // Evaluation success - print the results of the AutoScaleRun.
	        // This will display the values of each variable as evaluated by the
	        // autoscale formula.
	        Console.WriteLine("AutoScaleRun.Results: " +
	            eval.Results.Replace("$", "\n    $"));

	        // Apply the formula to the pool since it evaluated successfully
	        batchClient.PoolOperations.EnableAutoScale(pool.Id, myFormula);
	    }
	    else
	    {
	        // Evaluation failed, output the message associated with the error
	        Console.WriteLine("AutoScaleRun.Error.Message: " +
	            eval.Error.Message);
	    }
	}


成功对此代码段中的公式进行评估以后，将生成如下所示的输出：


	AutoScaleRun.Results:
	    $TargetDedicated=10;
	    $NodeDeallocationOption=requeue;
	    $curTime=2016-10-13T19:18:47.805Z;
	    $isWeekday=1;
	    $isWorkingWeekdayHour=0;
	    $workHours=0


## 获取有关自动缩放运行的信息
若要确保公式按预期方式执行，我们建议你定期检查 Batch 对池执行的自动缩放的“运行”结果。为此，请获取（或刷新）池的引用，并检查其最近的自动缩放运行的属性。

在 Batch .NET 中，[CloudPool.AutoScaleRun](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudpool.autoscalerun.aspx) 属性具有多个属性提供了有关 Batch 服务在池上执行的最新自动缩放运行的信息。

- [AutoScaleRun.Timestamp](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.autoscalerun.timestamp.aspx)
- [AutoScaleRun.Results](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.autoscalerun.results.aspx)
- [AutoScaleRun.Error](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.autoscalerun.error.aspx)

在 REST API 中，[获取池的信息](https://msdn.microsoft.com/zh-cn/library/dn820165.aspx)请求返回有关池的信息，在 [autoScaleRun](https://msdn.microsoft.com/zh-cn/library/dn820165.aspx#bk_autrun) 中包括最近的自动缩放运行信息。

以下 C# 代码片段使用 Batch .NET 库来打印有关池“myPool”的最新自动缩放运行的信息：

csharp

	Cloud pool = myBatchClient.PoolOperations.GetPool("myPool");
	Console.WriteLine("Last execution: " + pool.AutoScaleRun.Timestamp);
	Console.WriteLine("Result:" + pool.AutoScaleRun.Results.Replace("$", "\n  $"));
	Console.WriteLine("Error: " + pool.AutoScaleRun.Error);


前面的代码段的示例输出：


	Last execution: 10/14/2016 18:36:43
	Result:
	  $TargetDedicated=10;
	  $NodeDeallocationOption=requeue;
	  $curTime=2016-10-14T18:36:43.282Z;
	  $isWeekday=1;
	  $isWorkingWeekdayHour=0;
	  $workHours=0
	Error:


## 自动缩放公式示例
让我们看看几个使用不同的方法来调整池中的计算资源量的公式。

### 示例 1：基于时间的调整
也许，你希望能够根据星期几和一天的具体时间来调整池的大小，以便相应地增加或减少池中节点的数目。

此公式首先获取当前时间。如果日期是工作日（周一到周五）且时间是工作时间（早 8 点到晚 6 点），则会将目标池大小设置为 20 个节点。否则，设为 10 个节点。


	$curTime = time();
	$workHours = $curTime.hour >= 8 && $curTime.hour < 18;
	$isWeekday = $curTime.weekday >= 1 && $curTime.weekday <= 5;
	$isWorkingWeekdayHour = $workHours && $isWeekday;
	$TargetDedicated = $isWorkingWeekdayHour ? 20:10;


### 示例 2：基于任务的调整

在此示例中，池大小是根据队列中的任务数来调整的。请注意，在公式字符串中，注释和分行符都是可以接受的。

csharp

	// Get pending tasks for the past 15 minutes.
	$samples = $ActiveTasks.GetSamplePercent(TimeInterval_Minute * 15);
	// If we have fewer than 70 percent data points, we use the last sample point,
	// otherwise we use the maximum of last sample point and the history average.
	$tasks = $samples < 70 ? max(0,$ActiveTasks.GetSample(1)) : max( $ActiveTasks.GetSample(1), avg($ActiveTasks.GetSample(TimeInterval_Minute * 15)));
	// If number of pending tasks is not 0, set targetVM to pending tasks, otherwise
	// half of current dedicated.
	$targetVMs = $tasks > 0? $tasks:max(0, $TargetDedicated/2);
	// The pool size is capped at 20, if target VM value is more than that, set it
	// to 20. This value should be adjusted according to your use case.
	$TargetDedicated = max(0, min($targetVMs, 20));
	// Set node deallocation mode - keep nodes active only until tasks finish
	$NodeDeallocationOption = taskcompletion;


### 示例 3：考虑并行任务
这是另一个示例，可根据任务数调整池大小。此公式还考虑为池设置的 [MaxTasksPerComputeNode][net_maxtasks] 值。在对池启用了[并行任务执行](/documentation/articles/batch-parallel-node-tasks/)的情况下，此公式特别有用。

csharp

	// Determine whether 70 percent of the samples have been recorded in the past
	// 15 minutes; if not, use last sample
	$samples = $ActiveTasks.GetSamplePercent(TimeInterval_Minute * 15);
	$tasks = $samples < 70 ? max(0,$ActiveTasks.GetSample(1)) : max( $ActiveTasks.GetSample(1),avg($ActiveTasks.GetSample(TimeInterval_Minute * 15)));
	// Set the number of nodes to add to one-fourth the number of active tasks (the
	// MaxTasksPerComputeNode property on this pool is set to 4, adjust this number
	// for your use case)
	$cores = $TargetDedicated * 4;
	$extraVMs = (($tasks - $cores) + 3) / 4;
	$targetVMs = ($TargetDedicated + $extraVMs);
	// Attempt to grow the number of compute nodes to match the number of active
	// tasks, with a maximum of 3
	$TargetDedicated = max(0,min($targetVMs,3));
	// Keep the nodes active until the tasks finish
	$NodeDeallocationOption = taskcompletion;


### 示例 4：设置初始池大小
此示例演示了一个 C# 代码段，其中的自动缩放公式可在初始时间段内将池大小设置为特定的节点数。然后，在初始时间段过后，该公式会根据正在运行和处于活动状态的任务的数目调整池大小。

以下代码片段中的公式：

- 将初始池大小设置为 4 个节点。
- 在池生命周期的最初 10 分钟内不调整池大小。
- 10 分钟后，获取过去 60 分钟内正在运行和处于活动状态的任务数目的最大值。
  - 如果这两个值均为 0（表示过去 60 分钟没有正在运行或处于活动状态的任务），则池大小将设置为 0。
  - 如果其中一个值大于零，则不进行任何更改。

csharp

	string now = DateTime.UtcNow.ToString("r");
	string formula = string.Format(@"
		$TargetDedicated = {1};
		lifespan         = time() - time(""{0}"");
		span             = TimeInterval_Minute * 60;
		startup          = TimeInterval_Minute * 10;
		ratio            = 50;

		$TargetDedicated = (lifespan > startup ? (max($RunningTasks.GetSample(span, ratio), $ActiveTasks.GetSample(span, ratio)) == 0 ? 0 : $TargetDedicated) : {1});
		", now, 4);


## 后续步骤
- [Maximize Azure Batch compute resource usage with concurrent node tasks](/documentation/articles/batch-parallel-node-tasks/)（通过并发节点任务最大限度地提高 Azure Batch 计算资源的利用率）详细说明了如何在池中的计算节点上同时执行多个任务。除了自动缩放以外，此功能还可帮助降低某些工作负荷的作业持续时间，从而节省资金。
- 为了进一步提升效率，请确保 Batch 应用程序以最佳的方式查询 Batch 服务。在 [Query the Azure Batch service efficiently](/documentation/articles/batch-efficient-list-queries/)（有效地查询 Azure Batch 服务）中，可以了解在查询数千个计算节点或任务的状态时，如何限制跨线数据量。

[net_api]: https://msdn.microsoft.com/zh-cn/library/azure/mt348682.aspx
[net_batchclient]: http://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.batchclient.aspx
[net_cloudpool]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudpool.aspx
[net_cloudpool_autoscaleformula]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudpool.autoscaleformula.aspx
[net_cloudpool_autoscaleevalinterval]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudpool.autoscaleevaluationinterval.aspx
[net_enableautoscale]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.pooloperations.enableautoscale.aspx
[net_maxtasks]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudpool.maxtaskspercomputenode.aspx
[net_poolops_resizepool]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.pooloperations.resizepool.aspx

[rest_api]: https://msdn.microsoft.com/zh-cn/library/azure/dn820158.aspx
[rest_autoscaleformula]: https://msdn.microsoft.com/zh-cn/library/azure/dn820173.aspx
[rest_autoscaleinterval]: https://msdn.microsoft.com/zh-cn/library/azure/dn820173.aspx
[rest_enableautoscale]: https://msdn.microsoft.com/zh-cn/library/azure/dn820173.aspx

<!---HONumber=Mooncake_0306_2017-->
<!---Update_Description: wording update -->