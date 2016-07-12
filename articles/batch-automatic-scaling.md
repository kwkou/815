<properties
	pageTitle="自动缩放 Azure Batch 池中的计算节点 | Azure"
	description="对云池启用自动缩放功能可以动态调整池中计算节点的数目。"
	services="batch"
	documentationCenter=""
	authors="mmacy"
	manager="timlt"
	editor="tysonn"/>

<tags
	ms.service="batch"
	ms.date="04/18/2016"
	wacn.date="06/06/2016"/>

# 自动缩放 Azure 批处理 ( Batch )池中的计算节点

使用自动缩放，Azure Batch 可以根据定义的参数在池中动态添加或删除计算节点。这样，你便可以调整应用程序使用的计算资源量，从而节省时间和资金。

你可以通过将定义的自动缩放公式与计算节点池相关联（例如，使用 [Batch .NET](/documentation/articles/batch-dotnet-get-started/) 库中的 [PoolOperations.EnableAutoScale][net_enableautoscale] 方法），对计算节点池启用自动缩放。然后，Batch 服务将使用此公式来确定执行工作负荷所需的计算节点数目。Batch 将会响应定期收集的服务指标数据样本，并根据公式按可配置的间隔调整池中的计算节点数。

可以在创建池时启用自动缩放，也可以对现有池启用该功能。你还可以更改已启用“自动缩放”功能的池的现有公式。Batch 可让你在将公式分配给池之前先评估公式，以及监视自动缩放运行的状态。

## 自动缩放公式

自动缩放公式是你定义的字符串值，其中包含分配给池的 [autoScaleFormula][rest_autoscaleformula] 元素 (Batch REST) 或 [CloudPool.AutoScaleFormula][net_cloudpool_autoscaleformula] 属性 (Batch .NET) 的一个或多个语句。将公式分配到池后，Batch 服务将使用公式来确定池中可供下一个处理间隔使用的目标计算节点数（稍后将详细说明间隔）。公式是一个字符串，其大小不能超过 8 KB，最多可以包含 100 个以分号分隔的语句，可以包括换行符和注释。

可以将自动缩放公式视为使用 Batch 自动缩放“语言”。 公式语句是自由格式的表达式，可能包括系统定义的和用户定义的变量和常量。公式语句可以通过内置类型、运算符和函数对这些值执行各种操作。例如，语句可以采用以下格式：

`VAR = Expression(system-defined variables, user-defined variables);`

公式通常包含多个语句，这些语句对先前语句中获取的值执行操作：

		
		VAR₀ = Expression₀(system-defined variables);
		VAR₁ = Expression₁(system-defined variables, VAR₀);
		

在公式中使用这些语句的目标是实现池要缩放到的计算节点数目--**专用节点**的**目标**数目。此数目可以大于、小于或等于池中当前的节点数目。Batch 按特定的时间间隔（下面讨论[自动缩放时间间隔](#automatic-scaling-interval)）对池的自动缩放公式求值。然后，它会将池中节点的目标数目调整成在求值时自动缩放公式所指定的数目。

举个简单的例子，以下两行自动缩放公式根据活动任务数目指定应该调整的节点数目（最多 10 个计算节点）：

		
		$averageActiveTaskCount = avg($ActiveTasks.GetSample(TimeInterval_Minute * 15));
		$TargetDedicated = min(10, $averageActiveTaskCount);


本文的后续部分将介绍构成自动缩放公式的各个实体，包括变量、运算符、操作和函数。你会了解如何在 Batch 中获取各种计算资源和任务度量值。你可以使用这些度量值，根据资源使用情况和任务状态对池的节点计数进行智能化调整。然后，你将了解如何使用 Batch REST 和 .NET API 构建公式以及对池启用自动缩放。最后，我们将讨论几个示例公式。

> [AZURE.IMPORTANT] 每个 Azure 批处理帐户限制为可用于处理的计算节点的最大数目。Batch 服务将会根据该限制来创建节点。因此，它可能达不到公式所指定的目标数目。请参阅 [Azure Batch 服务的配额和限制](/documentation/articles/batch-quota-limit/)了解有关查看和提高帐户配额的信息。

## <a name="variables"></a>变量

可以在自动缩放公式中使用系统定义的变量和用户定义的变量。在上述两行示例公式中，`$TargetDedicated` 是系统定义的变量，而 `$averageActiveTaskCount` 是用户定义的变量。下表显示了 Batch 服务定义的读写和只读变量。

可以“获取”和“设置”这些**系统定义变量**的值，以管理池中计算节点的数目：

<table>
  <tr>
    <th>变量（读写）</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>$TargetDedicated</td>
    <td>池的<b>专用计算节点</b>的<b>目标</b>数。这是池应该缩放到的计算节点数目。它是一个“目标”数目，因为池可能达不到此目标节点数目。如果在池达到初始目标之前节点的目标数目被后续的自动缩放评估再次修改，则可能会发生这种情况。如果在达到节点的目标数目之前达到了 Batch 帐户的节点或核心配额，则也可能会发生这种情况。</td>
  </tr>
  <tr>
    <td>$NodeDeallocationOption</td>
    <td>从池中删除计算节点时发生的操作。可能的值包括：
      <br/>
      <ul>
        <li><p><b>requeue</b>--立即终止任务并将其放回作业队列，以便重新计划这些任务。</p></li>
        <li><p><b>terminate</b>--立即终止任务并从作业队列中删除它们。</p></li>
        <li><p><b>taskcompletion</b>--等待当前运行的任务完成，然后从池中删除节点。</p></li>
        <li><p><b>retaineddata</b>--等待清理节点上的本地任务保留的所有数据，然后从池中删除节点。</p></li>
      </ul></td>
   </tr>
</table>

获取这些**系统定义变量**的值即可根据 Batch 服务中的度量值进行调整：

<table>
  <tr>
    <th>变量（只读）</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>$CPUPercent</td>
    <td>CPU 使用率的平均百分比。</td>
  </tr>
  <tr>
    <td>$WallClockSeconds</td>
    <td>使用的秒数。</td>
  </tr>
  <tr>
    <td>$MemoryBytes</td>
    <td>使用的平均 MB 数。</td>
  <tr>
    <td>$DiskBytes</td>
    <td>本地磁盘上使用的平均 GB 数。</td>
  </tr>
  <tr>
    <td>$DiskReadBytes</td>
    <td>读取的字节数。</td>
  </tr>
  <tr>
    <td>$DiskWriteBytes</td>
    <td>写入的字节数。</td>
  </tr>
  <tr>
    <td>$DiskReadOps</td>
    <td>执行的读取磁盘操作数。</td>
  </tr>
  <tr>
    <td>$DiskWriteOps</td>
    <td>执行的写入磁盘操作数。</td>
  </tr>
  <tr>
    <td>$NetworkInBytes</td>
    <td>入站字节数。</td>
  </tr>
  <tr>
    <td>$NetworkInBytes</td>
    <td>出站字节数。</td>
  </tr>
  <tr>
    <td>$SampleNodeCount</td>
    <td>计算节点数。</td>
  </tr>
  <tr>
    <td>$ActiveTasks</td>
    <td>处于活动状态的任务数。</td>
  </tr>
  <tr>
    <td>$RunningTasks</td>
    <td>处于运行状态的任务数。</td>
  </tr>
  <tr>
    <td>$SucceededTasks</td>
    <td>成功完成的任务数。</td>
  </tr>
  <tr>
    <td>$FailedTasks</td>
    <td>失败的任务数。</td>
  </tr>
  <tr>
    <td>$CurrentDedicated</td>
    <td>当前的专用计算节点数。</td>
  </tr>
</table>

> [AZURE.TIP] 上面所示的只读系统定义变量是可提供各种方法来访问相互关联数据的“对象”。有关详细信息，请参阅下面的[获取样本数据](#getsampledata)。

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

上面所列的类型允许的**操作**：

| 操作 | 支持的运算符 | 结果类型 |
| ------------------------------------- | --------------------- | ------------- |
| double “运算符” double | +, -, *, / | double |
| double “运算符” timeinterval | * | timeinterval |
| doubleVec “运算符” double | +, -, *, / | doubleVec |
| doubleVec “运算符” doubleVec | +, -, *, / | doubleVec |
| timeinterval “运算符” double | *, / | timeinterval |
| timeinterval “运算符” timeinterval | +, - | timeinterval |
| timeinterval “运算符” timestamp | + | timestamp |
| timestamp “运算符” timeinterval | + | timestamp |
| timestamp “运算符” timestamp | - | timeinterval |
| *operator*double | -, ! | double |
| *operator*timeinterval | - | timeinterval |
| double “运算符” double | <, <=, ==, >=, >, != | double |
| string “运算符” string | <, <=, ==, >=, >, != | double |
| timestamp “运算符” timestamp | <, <=, ==, >=, >, != | double |
| timeinterval “运算符” timeinterval | <, <=, ==, >=, >, != | double |
| double “运算符” double | &&, || | double |

测试包含三元运算符的双精度值 (`double ? statement1 : statement2`) 时，非零值为 **true**，零值为 **false**。

## 函数

可以使用以下预定义**函数**来定义自动缩放公式。

| 函数 | 返回类型 | 说明
| --------------------------------- | ------------- | --------- |
| avg(doubleVecList) | double | 返回 DoubleVecList 中所有值的平均值。
| len(doubleVecList) | double | 返回从 doubleVecList 创建的向量的长度。
| lg(double) | double | 返回 double 的对数底数 2。
| lg(doubleVecList) | doubleVec | 返回 doubleVecList 的分量对数底数 2。必须为参数显式传递 vec(double)。否则会采用 double lg(double) 版本。
| ln(double) | double | 返回 double 的自然对数。
| ln(doubleVecList) | doubleVec | 返回 doubleVecList 的分量对数底数 2。必须为参数显式传递 vec(double)。否则会采用 double lg(double) 版本。
| log(double) | double | 返回 double 的对数底数 10。
| log(doubleVecList) | doubleVec | 返回 doubleVecList 的分量对数底数 10。对于单一的 double 参数，必须显式传递 vec(double)。否则会采用 double log(double) 版本。
| max(doubleVecList) | double | 返回 doubleVecList 中的最大值。
| min(doubleVecList) | double | 返回 doubleVecList 中的最小值。
| norm(doubleVecList) | double | 返回从 doubleVecList 创建的向量的二范数。
| percentile(doubleVec v, double p) | double | 返回向量 v 的百分位元素。
| rand() | double | 返回介于 0.0 和 1.0 之间的随机值。
| range(doubleVecList) | double | 返回 doubleVecList 中最小值和最大值之间的差。
| std(doubleVecList) | double | 返回 doubleVecList 中值的样本标准偏差。
| stop() | | 停止对自动缩放表达式求值。
| sum(doubleVecList) | double | 返回 doubleVecList 的所有组成部分之和。
| time(string dateTime="") | timestamp | 如果未传递参数，则返回当前时间的时间戳；如果传递了参数，则返回 dateTime 字符串的时间戳。支持的 dateTime 格式为 W3C-DTF 和 RFC 1123。
| val(doubleVec v, double i) | double | 返回在起始索引为零的向量 v 中，位置 i 处的元素的值。

上表中描述的某些函数可以接受列表作为参数。逗号分隔列表为 *double* 和 *doubleVec* 的任意组合。例如：

`doubleVecList := ( (double | doubleVec)+(, (double | doubleVec) )* )?`

*doubleVecList* 值在计算之前将转换为单个 *doubleVec*。例如，如果 `v = [1,2,3]`，则调用 `avg(v)` 相当于调用 `avg(1,2,3)`。调用 `avg(v, 7)` 相当于调用 `avg(1,2,3,7)`。

## <a name="getsampledata"></a>获取样本数据

自动缩放公式使用 Batch 服务提供的度量值数据（样本）。公式根据服务所提供的值来扩大或缩小池的大小。上述系统定义的变量是可提供各种方法来访问与该对象关联的数据的对象。例如，以下表达式显示了一个用于获取过去五分钟 CPU 使用率的请求：

`$CPUPercent.GetSample(TimeInterval_Minute * 5)`

<table>
  <tr>
    <th>方法</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>GetSample()</td>
    <td><p><b>GetSample()</b> 方法返回数据样本的向量。
	<p>一个样本最好包含 30 秒钟的度量值数据。换而言之，将每隔 30 秒获取一次样本。但如下所示，样本在收集后需经历一定的延迟才能供公式使用。因此，并非一段指定时间内的所有样本都可用于公式求值。
        <ul>
          <li><p><b>doubleVec GetSample(double count)</b>--在最近收集的样本中指定要获取的样本数。</p>
				  <p>GetSample(1) 返回最后一个可用样本。但对于像 $CPUPercent 这样的度量值，你不应使用此方法，因为无法知道样本是“何时”收集的。它可能是最近收集的，也可能由于系统问题而变得很旧。最好使用如下所示的时间间隔。</p></li>
          <li><p><b>doubleVec GetSample((timestamp | timeinterval) startTime [, double samplePercent])</b>--指定收集样本数据的时间范围。（可选）它还指定必须在请求的时间范围内提供的样本的百分比。</p>
          <p>如果 CPUPercent 历史记录中存在过去十分钟的所在样本，<em>$CPUPercent.GetSample(TimeInterval_Minute * 10)</em> 将返回 20 个样本。但如果最后一分钟的历史记录不可用，则只返回 18 个样本。在本示例中：<br/>
		  &#160;&#160;&#160;&#160;<em>$CPUPercent.GetSample(TimeInterval_Minute * 10, 95)</em> 将会失败，因为只有 90% 的样本可用。<br/>
		  &#160;&#160;&#160;&#160;<em>$CPUPercent.GetSample(TimeInterval_Minute * 10, 80)</em> 将会成功。</p></li>
          <li><p><b>doubleVec GetSample((timestamp | timeinterval) startTime, (timestamp | timeinterval) endTime [, double samplePercent])</b>--指定收集数据的时间范围，包括开始时间和结束时间。</p></li></ul>
		  <p>如前所述，每收集一个样本后并且该样本可供公式使用时，会存在一定的延迟。在使用 <em>GetSample</em> 方法时，必须考虑这个因素。参见下面的 <em>GetSamplePercent</em>。</td>
  </tr>
  <tr>
    <td>GetSamplePeriod()</td>
    <td>返回在历史样本数据集中采样的期间。</td>
  </tr>
	<tr>
		<td>Count()</td>
		<td>返回度量值历史记录中的样本总数。</td>
	</tr>
  <tr>
    <td>HistoryBeginTime()</td>
    <td>返回度量值最旧可用数据样本的时间戳。</td>
  </tr>
  <tr>
    <td>GetSamplePercent()</td>
    <td><p>返回给定时间间隔的可用样本百分比。例如：</p>
    <p><b>doubleVec GetSamplePercent( (timestamp | timeinterval) startTime [, (timestamp | timeinterval) endTime] )</b>
	<p>由于 GetSample 方法失败，因此如果返回的样本的百分比低于指定的 samplePercent，则可先使用 GetSamplePercent 方法进行检查。然后，如果存在的样本数量不足，你可以执行其他操作，无需停止自动缩放评估。</p></td>
  </tr>
</table>

### 样本、样本百分比和 *GetSample()* 方法

自动缩放公式的核心操作是获取任务和资源度量值数据，然后根据该数据调整池大小。因此，请务必明确知道自动缩放公式如何与度量值数据或“样本”交互。

**示例**

Batch 服务定期获取任务和资源度量值的“样本”，使其可供自动缩放公式使用。这些样本每 30 秒由 Batch 服务记录一次。但是，通常有一些滞后，以致记录样本的时间与样本可供自动缩放公式使用（与读取）的时间之间存在延迟。此外，由于各种因素（例如网络或其他基础结构问题），可能无法记录特定间隔的样本，从而导致样本“遗漏”。

**样本百分比**

将 `samplePercent` 传递到 `GetSample()` 方法，或调用 `GetSamplePercent()` 方法时，“percent”是指 Batch 服务*可能*记录的样本总数与自动缩放公式实际“可用”的样本数目之间的比较。

让我们以 10 分钟的时间跨度为例。由于每隔 30 秒记录样本一次，因此在 10 分钟的时间跨度内，Batch 服务所记录的样本总数将达到 20 个（每分钟 2 个）。但是，由于报告机制固有的延迟，或者 Azure 基础结构出现的其他一些问题，可能只有 15 个样本可供自动缩放公式读取。这意味着，在这 10 分钟的期间内，记录的样本总数只有 **75%** 实际可供公式使用。

**GetSample() 和样本范围**

你的自动缩放公式将对池进行扩大和缩小操作--添加节点或删除节点。由于节点耗费资金，你需要确保公式所使用的智能分析方法采用了足够的数据。因此，建议你在公式中使用趋势类型的分析。此类型的分析将根据所收集样本的“范围”来扩大和缩小你的池。

为此，请使用 `GetSample(interval look-back start, interval look-back end)` 返回样本的**向量**：

`runningTasksSample = $RunningTasks.GetSample(1 * TimeInterval_Minute, 6 * TimeInterval_Minute);`

Batch 评估上述代码行后，会以值的向量形式返回样本范围。例如：

`runningTasksSample=[1,1,1,1,1,1,1,1,1,1];`

收集样本向量后，便可使用 `min()`、`max()` 和 `avg()` 等函数从所收集的范围派生有意义的值。

为了提高安全性，如果特定时间段可用的样本数小于特定百分比，你可以强制公式求值*失败*。强制公式求值失败会指示 Batch 在无法提供指定百分比的样本数时停止进一步的公式求值--不更改池大小。若要指定求值成功所需的样本百分比，请将其指定为 `GetSample()` 的第三个参数。下面指定要求 75% 的样本：

`runningTasksSample = $RunningTasks.GetSample(60 * TimeInterval_Second, 120 * TimeInterval_Second, 75);`

此外，由于先前提到的样本可用性延迟问题，请务必记得指定回查开始时间早于一分钟的时间范围。这是由于样本需要花大约一分钟的时间才能传播到整个系统，因此通常无法使用 `(0 * TimeInterval_Second, 60 * TimeInterval_Second)` 范围内的样本。同样地，可以使用 `GetSample()` 百分比参数来强制实施特定样本百分比要求。

> [AZURE.IMPORTANT] **强烈建议****不要*只*依赖于自动缩放公式中的 `GetSample(1)`**。这是因为，`GetSample(1)` 基本上只是向 Batch 服务表明：“不论多久以前获取最后一个样本，请将它提供给我。” 由于它只是单个样本，而且可能是较旧的样本，因此可能无法代表最近任务或资源状态的全貌。如果使用 `GetSample(1)`，请确保它是更大语句的一部分，而不是公式所依赖的唯一数据点。

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
		<p> 这些系统定义的变量可用于根据节点计数进行调整：</p>
    <p><ul>
      <li>$TargetDedicated</li>
			<li>$CurrentDedicated</li>
			<li>$SampleNodeCount</li>
    </ul></p>
    <p>这些系统定义的变量可用于根据节点资源使用量进行调整：</p>
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
      <li>$NetworkInBytes</li></ul></p>
  </tr>
  <tr>
    <td><b>任务</b></td>
    <td><p><b>任务度量值</b>基于任务的状态，例如“活动”、“挂起”和“已完成”。以下系统定义变量可用于根据任务度量值调整池大小：</p>
    <p><ul>
      <li>$ActiveTasks</li>
      <li>$RunningTasks</li>
      <li>$SucceededTasks</li>
			<li>$FailedTasks</li></ul></p>
		</td>
  </tr>
</table>

## 构建自动缩放公式

构造自动缩放公式时，可以使用上述组件来生成语句，然后将这些语句组合成完整的公式即可。例如，在这里，我们在构造公式时，会先定义对公式的要求：

1. 如果 CPU 使用率高，则增加池中计算节点的目标数。
2. 如果 CPU 使用率低，则降低池中计算节点的目标数。
3. 始终将最大节点数限制为 400。

为了在 CPU 使用率高时“增加” 节点数，我们定义了一个语句，如果在过去 10 分钟内最小平均 CPU 使用率高于 70%，该语句就会向用户定义变量 ($TotalNodes) 填充一个值，值的大小为节点当前目标数的 110%：

`$TotalNodes = (min($CPUPercent.GetSample(TimeInterval_Minute*10)) > 0.7) ? ($CurrentDedicated * 1.1) : $CurrentDedicated;`

如果过去 60 分钟的平均 CPU 使用率“低于” 20%，则下一个语句会将同一变量设置为节点当前目标数的 90%。这会降低 CPU 使用率低时的目标数。请注意，此语句还引用以上语句中的用户定义变量 *$TotalNodes*。

`$TotalNodes = (avg($CPUPercent.GetSample(TimeInterval_Minute * 60)) < 0.2) ? ($CurrentDedicated * 0.9) : $TotalNodes;`

现在，将专用计算节点的目标数限制为**最大值** 400：

`$TargetDedicated = min(400, $TotalNodes)`

下面是完整公式：

		
		$TotalNodes = (min($CPUPercent.GetSample(TimeInterval_Minute*10)) > 0.7) ? ($CurrentDedicated * 1.1) : $CurrentDedicated;
		$TotalNodes = (avg($CPUPercent.GetSample(TimeInterval_Minute*60)) < 0.2) ? ($CurrentDedicated * 0.9) : $TotalNodes;
		$TargetDedicated = min(400, $TotalNodes)


> [AZURE.NOTE] 自动缩放公式由 [Batch REST][rest_api] API 变量、类型、操作和函数组成。即使是在使用 [Batch .NET][net_api] 库的时候，也可在公式字符串中使用这些组成元素。

## 在启用自动缩放的情况下创建池

若要在创建池时启用自动缩放功能，请使用以下方法之一：

- [New-AzureBatchPool](https://msdn.microsoft.com/library/azure/mt125936.aspx)--此 Azure PowerShell cmdlet 使用 AutoScaleFormula 参数来指定自动缩放公式。
- [BatchClient.PoolOperations.CreatePool](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.createpool.aspx)--在调用此 .NET 方法创建池后，将设置池的 [CloudPool.AutoScaleEnabled](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.autoscaleenabled.aspx) 属性和 [CloudPool.AutoScaleFormula](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.autoscaleformula.aspx) 属性，以启用自动缩放。
- [将池添加到帐户](https://msdn.microsoft.com/library/azure/dn820174.aspx)--创建池后，此 REST API 请求中使用的 enableAutoScale 和 autoScaleFormula 元素将为池设置自动缩放。

> [AZURE.IMPORTANT] 如果你使用上述方法之一创建了支持自动缩放的池，则**不得**指定该池的 *targetDedicated* 参数。另请注意，如果你希望手动调整启用自动缩放功能的池的大小（例如，使用 [BatchClient.PoolOperations.ResizePool][net_poolops_resizepool] 来调整），则必须先**禁用**该池的自动缩放功能，然后再调整池的大小。

下面的代码段演示如何通过 [Batch .NET][net_api] 库创建启用自动缩放的池 ([CloudPool][net_cloudpool])。该池的自动缩放公式在星期一将节点的目标数设置为 5，在每星期的其他日子将该目标数设置为 1。此外，自动缩放间隔设置为 30 分钟（请参阅下面的[自动缩放间隔](#automatic-scaling-interval)）。在本文的此部分与其他 C# 代码段中，“myBatchClient”是适当初始化的 [BatchClient][net_batchclient] 实例。


		CloudPool pool = myBatchClient.PoolOperations.CreatePool("mypool", "3", "small");
		pool.AutoScaleEnabled = true;
		pool.AutoScaleFormula = "$TargetDedicated = (time().weekday==1?5:1);";
		pool.AutoScaleEvaluationInterval = TimeSpan.FromMinutes(30);
		pool.Commit();
		

### <a name="interval"></a>自动缩放间隔

默认情况下，Batch 服务根据其自动缩放公式每隔 **15 分钟**调整池大小。但是，可使用以下池属性配置此间隔：

- REST API--[autoScaleEvaluationInterval][rest_autoscaleinterval]
- .NET API--[CloudPool.AutoScaleEvaluationInterval][net_cloudpool_autoscaleevalinterval]

最小间隔为 5 分钟，最大间隔为 168 小时。如果指定的间隔超出此范围，Batch 服务将返回“错误的请求(400)”错误。

> [AZURE.NOTE] 自动缩放目前不能以低于一分钟的时间响应更改，而只能在你运行工作负荷时逐步调整池大小。

## 创建池后启用自动缩放

如果你使用 *targetDedicated* 参数设置了包含指定计算节点数的池，则以后可以更新现有池以自动缩放。可以通过以下方法之一实现此目的：

- [BatchClient.PoolOperations.EnableAutoScale][net_enableautoscale]--此 .NET 方法需要现有池的 ID 和自动缩放公式才能应用到池。
- [允许对池进行自动缩放][rest_enableautoscale]--此 REST API 请求要求 URI 中存在现有池的 ID，以及请求正文中存在自动缩放公式。

> [AZURE.NOTE] 如果某个值是在创建池时为 *targetDedicated* 参数指定的，则会在评估自动缩放公式时忽略该值。

此代码段演示了如何在现有池上通过 [Batch .NET][net_api] 库启用自动缩放功能。请注意，针对现有池启用公式和更新公式使用相同的方法。因此，如果已启用自动缩放功能，则此方法会针对指定池“更新” 公式。该代码段假设“mypool”是现有池 ([CloudPool][net_cloudpool]) 的 ID。

		 // Define the autoscaling formula. In this snippet, the  formula sets the target number of nodes to 5 on
		 // Mondays, and 1 on every other day of the week
		 string myAutoScaleFormula = "$TargetDedicated = (time().weekday==1?5:1);";

		 // Update the existing pool's autoscaling formula by calling the BatchClient.PoolOperations.EnableAutoScale
		 // method, passing in both the pool's ID and the new formula.
		 myBatchClient.PoolOperations.EnableAutoScale("mypool", myAutoScaleFormula);

## 评估自动缩放公式

在应用程序中使用公式之前，最好先对它进行评估。评估公式时，可以针对现有池对公式进行“测试性运行”。执行此操作时，可通过以下方式：

- [BatchClient.PoolOperations.EvaluateAutoScale](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.evaluateautoscale.aspx) 或 [BatchClient.PoolOperations.EvaluateAutoScaleAsync](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.evaluateautoscaleasync.aspx)--这些 .NET 方法需要现有池的 ID，并需要包含自动缩放公式的字符串。调用的结果将包含在 [AutoScaleEvaluation](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.autoscaleevaluation.aspx) 类的实例中。
- [对自动缩放公式求值](https://msdn.microsoft.com/library/azure/dn820183.aspx)--在此 REST API 请求中，池 ID 在 URI 中指定。自动缩放公式在请求正文的 *autoScaleFormula* 元素中指定。操作的响应包含任何可能与该公式相关的错误信息。

> [AZURE.NOTE] 若要评估某个自动缩放公式，必须先通过有效的公式对池启用了自动缩放功能。

在这个使用 [Batch .NET][net_api] 库的代码段中，我们先对公式求值，然后将其应用到池 ([CloudPool][net_cloudpool])。

		
		// First obtain a reference to the existing pool
		CloudPool pool = myBatchClient.PoolOperations.GetPool("mypool");
		// We must ensure that autoscaling is enabled on the pool prior to evaluating a formula
		if (pool.AutoScaleEnabled.HasValue && pool.AutoScaleEnabled.Value)
		{
			// The formula to evaluate - adjusts target number of nodes based on day of week and time of day
			string myFormula = @"
				$CurTime=time();
				$WorkHours=$CurTime.hour>=8 && $CurTime.hour<18;
				$IsWeekday=$CurTime.weekday>=1 && $CurTime.weekday<=5;
				$IsWorkingWeekdayHour=$WorkHours && $IsWeekday;
				$TargetDedicated=$IsWorkingWeekdayHour?20:10;
			";
			// Perform the autoscale formula evaluation. Note that this does not actually apply the formula to
			// the pool.
			AutoScaleEvaluation eval = client.PoolOperations.EvaluateAutoScale(pool.Id, myFormula);
			if (eval.AutoScaleRun.Error == null)
			{
				// Evaluation success - print the results of the AutoScaleRun. This will display the values of each
				// variable as evaluated by the autoscale formula.
				Console.WriteLine("AutoScaleRun.Results: " + eval.AutoScaleRun.Results);
				// Apply the formula to the pool since it evaluated successfully
				client.PoolOperations.EnableAutoScale(pool.Id, myFormula);
			}
			else
			{
				// Evaluation failed, output the message associated with the error
				Console.WriteLine("AutoScaleRun.Error.Message: " + eval.AutoScaleRun.Error.Message);
			}
		}


成功对此代码段中的公式进行评估以后，将生成如下所示的输出：

`AutoScaleRun.Results: $TargetDedicated = 10;$NodeDeallocationOption = requeue;$CurTime = 2015 - 08 - 25T20: 08:42.271Z;$IsWeekday = 1;$IsWorkingWeekdayHour = 0;$WorkHours = 0`

## 获取有关自动缩放运行的信息

定期检查自动缩放的运行结果，以确保公式按预期执行。

- [CloudPool.AutoScaleRun](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.autoscalerun.aspx)--使用 .NET 库时，池的此属性提供 [AutoScaleRun](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.autoscalerun.aspx) 类的实例。此类提供最新自动缩放运行的以下属性：
  - [AutoScaleRun.Error](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.autoscalerun.error.aspx)
  - [AutoScaleRun.Results](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.autoscalerun.results.aspx)
  - [AutoScaleRun.Timestamp](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.autoscalerun.timestamp.aspx)
- [获取有关池的信息](https://msdn.microsoft.com/library/dn820165.aspx)--此 REST API 请求返回有关池的信息，包括最近的自动缩放运行结果。

## <a name="examples"></a>示例公式

让我们看看一些示例，了解如何通过多种方式使用公式来自动缩放池中的计算资源。

### 示例 1：基于时间的调整

也许，你希望能够根据星期几和一天的具体时间来调整池的大小，相应地增加或减少池中节点的数目：
		
		
		$CurTime=time();
		$WorkHours=$CurTime.hour>=8 && $CurTime.hour<18;
		$IsWeekday=$CurTime.weekday>=1 && $CurTime.weekday<=5;
		$IsWorkingWeekdayHour=$WorkHours && $IsWeekday;
		$TargetDedicated=$IsWorkingWeekdayHour?20:10;
		

此公式首先获取当前时间。如果日期是工作日（周一到周五）且时间是工作时间（早 8 点到晚 6 点），则会将目标池大小设置为 20 个节点。否则，目标池大小将设置为 10 个节点。

### 示例 2：基于任务的调整

在此示例中，池大小是根据队列中的任务数来调整的。请注意，在公式字符串中，注释和分行符都是可以接受的。

		
		// Get pending tasks for the past 15 minutes.
		$Samples = $ActiveTasks.GetSamplePercent(TimeInterval_Minute * 15);
		// If we have fewer than 70 percent data points, we use the last sample point, otherwise we use the maximum of
		// last sample point and the history average.
		$Tasks = $Samples < 70 ? max(0,$ActiveTasks.GetSample(1)) : max( $ActiveTasks.GetSample(1), avg($ActiveTasks.GetSample(TimeInterval_Minute * 15)));
		// If number of pending tasks is not 0, set targetVM to pending tasks, otherwise half of current dedicated.
		$TargetVMs = $Tasks > 0? $Tasks:max(0, $TargetDedicated/2);
		// The pool size is capped at 20, if target VM value is more than that, set it to 20. This value
		// should be adjusted according to your use case.
		$TargetDedicated = max(0,min($TargetVMs,20));
		// Set node deallocation mode - keep nodes active only until tasks finish
		$NodeDeallocationOption = taskcompletion;


### 示例 3：考虑并行任务

这是另一个示例，可根据任务数调整池大小。此公式还考虑为池设置的 [MaxTasksPerComputeNode][net_maxtasks] 值。在对池启用了[并行任务执行](/documentation/articles/batch-parallel-node-tasks/)的情况下，此公式特别有用。


		// Determine whether 70 percent of the samples have been recorded in the past 15 minutes; if not, use last sample
		$Samples = $ActiveTasks.GetSamplePercent(TimeInterval_Minute * 15);
		$Tasks = $Samples < 70 ? max(0,$ActiveTasks.GetSample(1)) : max( $ActiveTasks.GetSample(1),avg($ActiveTasks.GetSample(TimeInterval_Minute * 15)));
		// Set the number of nodes to add to one-fourth the number of active tasks (the MaxTasksPerComputeNode
		// property on this pool is set to 4, adjust this number for your use case)
		$Cores = $TargetDedicated * 4;
		$ExtraVMs = (($Tasks - $Cores) + 3) / 4;
		$TargetVMs = ($TargetDedicated+$ExtraVMs);
		// Attempt to grow the number of compute nodes to match the number of active tasks, with a maximum of 3
		$TargetDedicated = max(0,min($TargetVMs,3));
		// Keep the nodes active until the tasks finish
		$NodeDeallocationOption = taskcompletion;
		

### 示例 4：设置初始池大小

此示例演示了一个 C# 代码段，其中的自动缩放公式可在初始时间段内将池大小设置为特定的节点数。然后，在初始时间段过后，该公式会根据正在运行和处于活动状态的任务的数目调整池大小。

		
		string now = DateTime.UtcNow.ToString("r");
		string formula = string.Format(@"
			$TargetDedicated = {1};
			lifespan         = time() - time(""{0}"");
			span             = TimeInterval_Minute * 60;
			startup          = TimeInterval_Minute * 10;
			ratio            = 50;
			$TargetDedicated = (lifespan > startup ? (max($RunningTasks.GetSample(span, ratio), $ActiveTasks.GetSample(span, ratio)) == 0 ? 0 : $TargetDedicated) : {1});
			", now, 4);

上面的代码段中的公式：

- 将初始池大小设置为 4 个节点。
- 在池生命周期的最初 10 分钟内不调整池大小。
- 10 分钟后，获取过去 60 分钟内正在运行和处于活动状态的任务数目的最大值。
  - 如果这两个值均为 0（表示过去 60 分钟没有正在运行或处于活动状态的任务），则池大小将设置为 0。
  - 如果其中一个值大于零，则不进行任何更改。

## 后续步骤

* [Maximize Azure Batch compute resource usage with concurrent node tasks（通过并发节点任务最大限度提高 Azure Batch 计算资源的使用量）](/documentation/articles/batch-parallel-node-tasks/)详细说明了如何在池中的计算节点上同时执行多个任务。除了自动缩放以外，此功能还可帮助降低某些工作负荷的作业持续时间，从而节省资金。

* 为了进一步提升效率，请确保 Batch 应用程序以最佳的方式查询 Batch 服务。在 [Query the Azure Batch service efficiently（有效地查询 Azure Batch 服务）](/documentation/articles/batch-efficient-list-queries/)中，可以了解在查询数千个计算节点或任务的状态时，如何限制跨线数据量。

[net_api]: https://msdn.microsoft.com/library/azure/mt348682.aspx
[net_batchclient]: http://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.aspx
[net_cloudpool]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx
[net_cloudpool_autoscaleformula]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.autoscaleformula.aspx
[net_cloudpool_autoscaleevalinterval]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.autoscaleevaluationinterval.aspx
[net_enableautoscale]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.enableautoscale.aspx
[net_maxtasks]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.maxtaskspercomputenode.aspx
[net_poolops_resizepool]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.resizepool.aspx

[rest_api]: https://msdn.microsoft.com/library/azure/dn820158.aspx
[rest_autoscaleformula]: https://msdn.microsoft.com/library/azure/dn820173.aspx
[rest_autoscaleinterval]: https://msdn.microsoft.com/en-us/library/azure/dn820173.aspx
[rest_enableautoscale]: https://msdn.microsoft.com/library/azure/dn820173.aspx

<!---HONumber=Mooncake_0530_2016-->