
<properties
	pageTitle="自动缩放 Azure  批处理( Batch )池中的计算节点"
	description="可以通过对池启用自动缩放，并将一个公式（用于计算处理应用程序所需的计算节点数）关联到该池，来实现自动缩放。"
	services="batch"
	documentationCenter=""
	authors="davidmu1"
	manager="timlt"
	editor="tysonn"/>

<tags
	ms.service="batch"
	ms.date="08/26/2015"
	wacn.date="12/31/2015"/>

# 自动缩放 Azure 批处理( Batch )池中的计算节点

自动缩放 Azure 批处理( Batch )池中的计算节点是指动态调整应用程序使用的处理能力。这种调整灵活性节省了时间和资金。若要了解有关计算节点和池的详细信息，请参阅 [Azure 批次技术概述](/documentation/articles/batch-technical-overview)。

如果对池启用了相应的功能并将某个公式与池相关联，则会发生自动缩放。该公式用于确定处理应用程序所需的计算节点数。对定期收集的样本进行操作时，池中可用计算节点的数目会根据关联的公式每 15 分钟调整一次。

可以在创建池时设置自动缩放，也可以在以后对现有池启用该功能。在此前已启用自动缩放的情况下，也可在池上更新公式。在将公式分配到池之前，始终建议你评估该公式，并且你必须监视自动缩放的运行状态；我们将在下面逐一讨论这些主题。

> [AZURE.NOTE]每个 Azure 批处理( Batch )
> 帐户限制为可用于处理的计算节点的最大数目。系统最多会创建限制数目的节点，因此不可能会达到公式指定的目标数。

## 自动缩放计算资源

使用定义的缩放公式可以确定池中用于下一个处理间隔的可用计算节点数。自动缩放公式只不过是一个字符串值，该值分配给请求主体 (REST API) 中池的 [autoScaleFormula](https://msdn.microsoft.com/library/azure/dn820173.aspx) 元素或 [CloudPool.AutoScaleFormula](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.autoscaleformula.aspx) 属性 (.NET API)。该公式是一个字符串，其大小不能超过 8KB，最多可以包含 100 个以分号分隔的语句，可以包括换行符和注释。

公式中的语句是自由格式的表达式。这些语句可以包含任何系统定义的变量、用户定义的变量、常量值，以及这些变量或常量支持的操作：

	VAR = Expression(system-defined variables, user-defined variables);

复杂的公式是使用多个语句和变量创建的：

	VAR₀ = Expression₀(system-defined variables);
	VAR₁ = Expression₁(system-defined variables, VAR₀);

> [AZURE.NOTE]自动缩放公式由 [批处理( Batch ) REST](https://msdn.microsoft.com/library/azure/dn820158.aspx) API 变量、类型、操作和函数组成。即使是在使用 [批处理( Batch ) .NET](https://msdn.microsoft.com/library/azure/mt348682.aspx) 库的时候，也会在公式字符串中使用这些组成元素。

### 变量

可以在公式中使用系统定义的变量和用户定义的变量。

可以*获取*和*设置*这些**系统定义变量**的值，以管理池中计算节点的数目。

<table>
  <tr>
    <th>变量</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>$TargetDedicated</td>
    <td>池的专用计算节点的目标数。可以根据任务的实际用法更改该值。</td>
  </tr>
  <tr>
    <td>$NodeDeallocationOption</td>
    <td>从池中删除计算节点时发生的操作。可能的值包括：
      <br/>
      <ul>
        <li><p><b>requeue</b> – 立即终止任务并将其放回作业队列，以便重新计划这些任务。</p></li>
        <li><p><b>terminate</b> – 立即终止任务并从作业队列中删除它们。</p></li>
        <li><p><b>taskcompletion</b> – 等待当前运行的任务完成，然后从池中删除节点。</p></li>
        <li><p><b>retaineddata</b> - 等待清理节点上的本地任务保留的所有数据，然后从池中删除节点。</p></li>
      </ul></td>
   </tr>
</table>

*获取*这些**系统定义变量**的值即可根据样本中计算节点的度量值进行调整。这些变量是只读变量。

<table>
  <tr>
    <th>变量</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>$CPUPercent</td>
    <td>CPU 使用率的平均百分比</td>
  </tr>
  <tr>
    <td>$WallClockSeconds</td>
    <td>使用的秒数</td>
  </tr>
  <tr>
    <td>$MemoryBytes</td>
    <td>使用的平均 MB 数</td>
  <tr>
    <td>$DiskBytes</td>
    <td>本地磁盘上使用的平均 GB 数</td>
  </tr>
  <tr>
    <td>$DiskReadBytes</td>
    <td>读取的字节数</td>
  </tr>
  <tr>
    <td>$DiskWriteBytes</td>
    <td>写入的字节数</td>
  </tr>
  <tr>
    <td>$DiskReadOps</td>
    <td>执行的读取磁盘操作数</td>
  </tr>
  <tr>
    <td>$DiskWriteOps</td>
    <td>执行的写入磁盘操作数</td>
  </tr>
  <tr>
    <td>$NetworkInBytes</td>
    <td>入站字节数</td>
  </tr>
  <tr>
    <td>$NetworkInBytes</td>
    <td>出站字节数</td>
  </tr>
  <tr>
    <td>$SampleNodeCount</td>
    <td>计算节点数</td>
  </tr>
  <tr>
    <td>$ActiveTasks</td>
    <td>处于活动状态的任务数</td>
  </tr>
  <tr>
    <td>$RunningTasks</td>
    <td>处于运行状态的任务数</td>
  </tr>
  <tr>
    <td>$SucceededTasks</td>
    <td>成功完成的任务数</td>
  </tr>
  <tr>
    <td>$FailedTasks</td>
    <td>失败的任务数</td>
  </tr>
  <tr>
    <td>$CurrentDedicated</td>
    <td>当前的专用计算节点数</td>
  </tr>
</table>

### 类型

公式中支持以下类型：

- double
- doubleVec
- 字符串
- timestamp。timestamp 是包含以下成员的复合结构。
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

### 操作

上面所列的类型允许的**操作**：

<table>
  <tr>
    <th>操作</th>
    <th>允许的操作</th>
  </tr>
  <tr>
    <td>double &lt;运算符> double => double</td>
    <td>+, -, *, /</td>
  </tr>
  <tr>
    <td>double &lt;运算符> timeinterval => timeinterval</td>
    <td>*</td>
  </tr>
  <tr>
    <td>doubleVec &lt;运算符> double => doubleVec</td>
    <td>+, -, *, /</td>
  </tr>
  <tr>
    <td>doubleVec &lt;运算符> doubleVec => doubleVec</td>
    <td>+, -, *, /</td>
  </tr>
  <tr>
    <td>timeinterval &lt;运算符> double => timeinterval</td>
    <td>*, /</td>
  </tr>
  <tr>
    <td>timeinterval &lt;运算符> timeinterval => timeinterval</td>
    <td>+, -</td>
  </tr>
  <tr>
    <td>timeinterval &lt;运算符> timestamp => timestamp</td>
    <td>+</td>
  </tr>
  <tr>
    <td>timestamp &lt;运算符> timeinterval => timestamp</td>
    <td>+</td>
  </tr>
  <tr>
    <td>timestamp &lt;运算符> timestamp => timeinterval</td>
    <td>-</td>
  </tr>
  <tr>
    <td>&lt;运算符>double => double</td>
    <td>-, !</td>
  </tr>
  <tr>
    <td>&lt;运算符>timeinterval => timeinterval</td>
    <td>-</td>
  </tr>
  <tr>
    <td>double &lt;运算符> double => double</td>
    <td>&lt;, &lt;=, ==, >=, >, !=</td>
  </tr>
  <tr>
    <td>string &lt;运算符> string => double</td>
    <td>&lt;, &lt;=, ==, >=, >, !=</td>
  </tr>
  <tr>
    <td>timestamp &lt;运算符> timestamp => double</td>
    <td>&lt;, &lt;=, ==, >=, >, !=</td>
  </tr>
  <tr>
    <td>timeinterval &lt;运算符> timeinterval => double</td>
    <td>&lt;, &lt;=, ==, >=, >, !=</td>
  </tr>
  <tr>
    <td>double &lt;运算符> double => double</td>
    <td>&amp;&amp;, ||</td>
  </tr>
  <tr>
    <td>测试仅限 double（非零值为 true，零值为 false）</td>
    <td>? :</td>
  </tr>
</table>

### 函数

可以使用以下预定义**函数**来定义自动缩放公式。

<table>
  <tr>
    <th>函数</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>double <b>avg</b>(doubleVecList)</td>
    <td>DoubleVecList 中所有值的平均值。</td>
  </tr>
  <tr>
    <td>double <b>len</b>(doubleVecList)</td>
    <td>从 doubleVecList 创建的向量的长度。</td>
  <tr>
    <td>double <b>lg</b>(double)</td>
    <td>对数底数为 2。</td>
  </tr>
  <tr>
    <td>doubleVec <b>lg</b>(doubleVecList)</td>
    <td>分量对数底数 2。必须为单个 double 参数显式传递 vec(double)，否则将采用 double lg(double) 版本。</td>
  </tr>
  <tr>
    <td>double <b>ln</b>(double)</td>
    <td>自然对数。</td>
  </tr>
  <tr>
    <td>doubleVec <b>ln</b>(doubleVecList)</td>
    <td>分量对数底数 2。必须为单个 double 参数显式传递 vec(double)，否则将采用 double lg(double) 版本。</td>
  </tr>
  <tr>
    <td>double <b>log</b>(double)</td>
    <td>对数底数为 10。</td>
  </tr>
  <tr>
    <td>doubleVec <b>log</b>(doubleVecList)</td>
    <td>分量对数底数 10。必须为单个 double 参数显式传递 vec(double)，否则将采用 double log(double) 版本。</td>
  </tr>
  <tr>
    <td>double <b>max</b>(doubleVecList)</td>
    <td>doubleVecList 中的最大值。</td>
  </tr>
  <tr>
    <td>double <b>min</b>(doubleVecList)</td>
    <td>doubleVecList 中的最小值。</td>
  </tr>
  <tr>
    <td>double <b>norm</b>(doubleVecList)</td>
    <td>从 doubleVecList 创建的向量的二范数。
  </tr>
  <tr>
    <td>double <b>percentile</b>(doubleVec v, double p)</td>
    <td>向量 v 百分位元素。</td>
  </tr>
  <tr>
    <td>double <b>rand</b>()</td>
    <td>介于 0.0 和 1.0 之间的随机值。</td>
  </tr>
  <tr>
    <td>double <b>range</b>(doubleVecList)</td>
    <td>doubleVecList 中最小值和最大值之间的差。</td>
  </tr>
  <tr>
    <td>double <b>std</b>(doubleVecList)</td>
    <td>doubleVecList 中值的样本标准偏差。</td>
  </tr>
  <tr>
    <td><b>stop</b>()</td>
    <td>停止自动缩放表达式计算。</td>
  </tr>
  <tr>
    <td>double <b>sum</b>(doubleVecList)</td>
    <td>doubleVecList 的所有组成部分之和。</td>
  </tr>
  <tr>
    <td>timestamp <b>time</b>(string dateTime="")</td>
    <td>如果未传递参数，则为当前时间的时间戳；如果传递了参数，则为 dateTime 字符串的时间戳。支持的 dateTime 格式为 W3CDTF 和 RFC1123。</td>
  </tr>
  <tr>
    <td>double <b>val</b>(doubleVec v, double i)</td>
    <td>在起始索引为零的向量 v 中，位置 i 处的元素的值。</td>
  </tr>
</table>

上表中描述的某些函数可以接受列表作为参数。逗号分隔列表为 *double* 和 *doubleVec* 的任意组合。例如：

`doubleVecList := ( (double | doubleVec)+(, (double | doubleVec) )* )?`

*doubleVecList* 值在计算之前将转换为单个 *doubleVec*。例如，如果 `v = [1,2,3]`，则调用 `avg(v)` 等效于调用 `avg(1,2,3)`，调用 `avg(v, 7)` 等效于调用 `avg(1,2,3,7)`。

### 获取示例数据

上述系统定义的变量是提供用于访问关联数据的方法的对象。例如，以下表达式显示了一个用于获取过去五分钟 CPU 使用率的请求：

	$CPUPercent.GetSample(TimeInterval_Minute*5)

这些方法可用于获取样本数据。

<table>
  <tr>
    <th>方法</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>Count()</td>
    <td>返回度量值历史记录中的样本总数。</td>
  </tr>
  <tr>
    <td>GetSample()</td>
    <td><p>返回数据样本的向量。
	<p>一个样本最好包含 30 秒钟的度量值数据。换而言之，将每隔 30 秒获取样本一次，但如下所述，每收集一个样本后并且该样本可供公式使用时，会存在一定的延迟。因此，并非一段指定时间内的所有样本都可用于公式求值。
        <ul>
          <li><p><b>doubleVec GetSample(double count)</b> - 在最近的收集的样本中指定要获取的样本数。</p>
				  <p>GetSample(1) 返回最后一个可用样本。但对于像 $CPUPercent 这样的度量值，你不应使用此方法，因为无法知道样本是<em>何时</em>收集的 - 它可能是最近收集的，也可能由于系统问题而变得很旧。最好使用如下所示的时间间隔。</p></li>
          <li><p><b>doubleVec GetSample((timestamp | timeinterval) startTime [, double samplePercent])</b> – 指定收集样本数据的时间范围，并选择性地指定必须在请求时间范围内可用的样本百分比。</p>
          <p>如果 CPUPercent 历史记录中存在过去十分钟的所在样本，<em>$CPUPercent.GetSample(TimeInterval_Minute * 10)</em> 将返回 20 个样本。如果最后一分钟的历史记录不可用，则只返回 18 个样本，在这种情况下：<br/>
		  &#160;&#160;&#160;&#160;<em>$CPUPercent.GetSample(TimeInterval_Minute * 10, 95)</em> 将会失败，因为只有 90% 的样本可用；<br/>
		  &#160;&#160;&#160;&#160;<em>$CPUPercent.GetSample(TimeInterval_Minute * 10, 80)</em> 将会成功。</p></li>
          <li><p><b>doubleVec GetSample((timestamp | timeinterval) startTime, (timestamp | timeinterval) endTime [, double samplePercent])</b> – 指定收集数据的时间范围，包括开始时间和结束时间。</p></li></ul>
		  <p>如前所述，每收集一个样本后并且该样本可供公式使用时，会存在一定的延迟。在使用 <em>GetSample</em> 方法时，必须考虑这个因素 - 请参阅下面的 <em>GetSamplePercent</em>。</td>
  </tr>
  <tr>
    <td>GetSamplePeriod()</td>
    <td>返回在历史样本数据集中采样的期间。</td>
  </tr>
  <tr>
    <td>HistoryBeginTime()</td>
    <td>返回度量值最旧可用数据样本的时间戳。</td>
  </tr>
  <tr>
    <td>GetSamplePercent()</td>
    <td><p>返回历史记录当前针对给定时间间隔提供的样本百分比。例如：</p>
    <p><b>doubleVec GetSamplePercent( (timestamp | timeinterval) startTime [, (timestamp | timeinterval) endTime] )</b>
	<p>由于当返回的样本百分比小于指定的 samplePercent 时 GetSample 方法会失败，因此，你可以使用 GetSamplePercent 方法执行初始检查，然后在没有足够样本的情况下，不暂停样本自动缩放评估并执行备选操作。</p></td>
  </tr>
</table>

### 度量值

在定义公式时，你可以使用资源和任务**度量值**，这些度量值可用于管理池中的计算节点。

<table>
  <tr>
    <th>度量值</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>资源</td>
    <td><p>资源度量值基于 CPU 使用率、带宽使用率、内存使用率和计算节点的数目。可在公式中使用上述系统定义变量（在上面的**变量**中进行了说明）来管理池中的计算节点：</p>
    <p><ul>
      <li>$TargetDedicated</li>
      <li>$NodeDeallocationOption</li>
    </ul></p>
    <p>这些系统定义变量用于根据节点资源度量值进行调整：</p>
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
    <td>任务</td>
    <td><p>根据任务的状态，例如“活动”、“挂起”和“已完成”。</p>
    <p>这些系统定义变量用于根据任务度量值进行调整：</p>
    <p><ul>
      <li>$ActiveTasks</li>
      <li>$RunningTasks</li>
      <li>$SucceededTasks</li>
      <li>$FailedTasks</li>
      <li>$CurrentDedicated</li></ul></p></td>
  </tr>
</table>

## 构建自动缩放公式

构造自动缩放公式时，可以使用上述组件来生成语句，然后将这些语句组合成完整的公式即可。例如，在这里，我们在构造公式时，会先定义对公式的要求：

1. 如果 CPU 使用率高，则增加池中计算节点的目标数
2. 如果 CPU 使用率低，则降低池中计算节点的目标数
3. 始终将最大节点数限制为 400

为了在 CPU 使用率高时*增加* 节点数，我们定义了一个语句，如果在过去 10 分钟内最小平均 CPU 使用率高于 70%，该语句就会向用户定义变量 ($TotalNodes) 填充一个值，值的大小为节点当前目标数的 110%：

	$TotalNodes = (min($CPUPercent.GetSample(TimeInterval_Minute*10)) > 0.7) ? ($CurrentDedicated * 1.1) : $CurrentDedicated;

如果过去 60 分钟的平均 CPU 使用率*低于* 20%，则下一个语句会将同一变量设置为节点当前目标数的 90%，降低 CPU 使用率低时的目标数。请注意，此语句还引用以上语句中的用户定义变量 *$TotalNodes*。

	$TotalNodes = (avg($CPUPercent.GetSample(TimeInterval_Minute*60)) < 0.2) ? ($CurrentDedicated * 0.9) : $TotalNodes;

现在，将专用计算节点的目标数限制为**最大值** 400：

	$TargetDedicated = min(400, $TotalNodes)

下面是完整公式：

	$TotalNodes = (min($CPUPercent.GetSample(TimeInterval_Minute*10)) > 0.7) ? ($CurrentDedicated * 1.1) : $CurrentDedicated;
	$TotalNodes = (avg($CPUPercent.GetSample(TimeInterval_Minute*60)) < 0.2) ? ($CurrentDedicated * 0.9) : $TotalNodes;
	$TargetDedicated = min(400, $TotalNodes)

## 在启用自动缩放的情况下创建池

若要在创建池时启用自动缩放功能，请使用以下方法之一：

- [New-AzureBatchPool](https://msdn.microsoft.com/library/azure/mt125936.aspx) – 此 Azure PowerShell cmdlet 使用 AutoScaleFormula 参数来指定自动缩放公式。
- [BatchClient.PoolOperations.CreatePool](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.createpool.aspx) – 在调用此 .NET 方法创建池后，将设置池的 [CloudPool.AutoScaleEnabled](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.autoscaleenabled.aspx) 和 [CloudPool.AutoScaleFormula](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.autoscaleformula.aspx) 属性，以启用自动缩放。
- [将池添加到帐户](https://msdn.microsoft.com/library/azure/dn820174.aspx) – 创建池后，此 REST API 请求中使用的 enableAutoScale 和 autoScaleFormula 元素将为池设置自动缩放。

> [AZURE.NOTE]如果池是使用上述方法之一创建的，则当你在池创建后设置自动缩放时，将不指定该池的 *targetDedicated* 参数（不能指定）。另请注意，如果你希望手动调整启用自动缩放功能的池的大小（例如，使用 [BatchClient.PoolOperations.ResizePool](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.resizepool.aspx) 来调整），则必须先禁用该池的自动缩放功能，然后再调整池的大小。

以下代码段显示了如何创建启用自动缩放功能的 [CloudPool](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx)，创建时使用 [批处理( Batch ) .NET](https://msdn.microsoft.com/library/azure/mt348682.aspx) 库，其公式将节点的目标数设置为 5（周一）和 1（除周一外的其他时间）。在代码段中，“myBatchClient”是正确初始化的 [BatchClient](http://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.aspx) 实例。

		CloudPool pool myBatchClient.PoolOperations.CreatePool("mypool", "3", "small");
		pool.AutoScaleEnabled = true;
		pool.AutoScaleFormula = "$TargetDedicated = (time().weekday==1?5:1);";
		pool.Commit();

## 创建池后启用自动缩放

如果你使用 *targetDedicated* 参数设置了包含指定计算节点数的池，则以后可以更新现有池以自动缩放。通过以下方法之一执行这种检查：

- [BatchClient.PoolOperations.EnableAutoScale](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.enableautoscale.aspx) – 此 .NET 方法需要现有池的 ID 和自动缩放公式才能应用到池。
- [允许对池进行自动缩放](https://msdn.microsoft.com/library/azure/dn820173.aspx) – 此 REST API 请求要求 URI 中存在现有池的 ID，以及请求正文中存在自动缩放公式。

> [AZURE.NOTE]如果某个值是在创建池时为 *targetDedicated* 参数指定的，则会在评估自动缩放公式时忽略该值。

此代码段演示了如何在现有池上通过 [Batch .NET](https://msdn.microsoft.com/library/azure/mt348682.aspx) 库启用自动缩放功能。请注意，针对现有池启用公式和更新公式使用相同的方法。因此，如果已启用自动缩放功能，则此方法会针对指定池*更新* 公式。此代码段假定“myBatchClient”是正确初始化的 [BatchClient](http://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.aspx) 实例，而“mypool”则是现有 [CloudPool](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx) 的 ID。

		 // Define the autoscaling formula. In this snippet, the  formula sets the target number of nodes to 5 on
		 // Mondays, and 1 on every other day of the week
		 string myAutoScaleFormula = "$TargetDedicated = (time().weekday==1?5:1);";

		 // Update the existing pool's autoscaling formula by calling the BatchClient.PoolOperations.EnableAutoScale
		 // method, passing in both the pool's ID and the new formula.
		 myBatchClient.PoolOperations.EnableAutoScale("mypool", myAutoScaleFormula);

## 评估自动缩放公式

在应用程序中使用公式之前，最好先对它进行评估。评估公式时，可以针对现有池对公式进行“测试性运行”。执行此操作时，可通过以下方式：

- [BatchClient.PoolOperations.EvaluateAutoScale](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.evaluateautoscale.aspx) 或 [BatchClient.PoolOperations.EvaluateAutoScaleAsync](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.evaluateautoscaleasync.aspx) – 这些 .NET 方法需要现有池的 ID，并需要包含自动缩放公式的字符串。调用的结果将包含在 [AutoScaleEvaluation](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.autoscaleevaluation.aspx) 类的实例中。
- [评估自动缩放公式](https://msdn.microsoft.com/library/azure/dn820183.aspx) – 在这个 REST API 请求中，池 ID 已在 URI 中指定，自动缩放公式已在请求正文的 *autoScaleFormula* 元素中指定。操作的响应包含任何可能与该公式相关的错误信息。

> [AZURE.NOTE]若要评估某个自动缩放公式，你必须先通过有效的公式对池启用了自动缩放功能。

在这个使用 [批处理( Batch ) .NET](https://msdn.microsoft.com/library/azure/mt348682.aspx) 库的代码段中，我们先对公式进行评估，然后将其应用到 [CloudPool](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx)。

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
				// variable as evaluated by the the autoscaling formula.
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

		AutoScaleRun.Results: $TargetDedicated = 10;$NodeDeallocationOption = requeue;$CurTime = 2015 - 08 - 25T20: 08:42.271Z;$IsWeekday = 1;$IsWorkingWeekdayHour = 0;$WorkHours = 0

## 获取有关自动缩放运行的信息

如果公式正常执行，则应定期查看自动缩放的运行结果。通过以下方法之一执行这种检查：

- [CloudPool.AutoScaleRun](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.autoscalerun.aspx) – 使用 .NET 库时，池的此属性将提供 [AutoScaleRun](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.autoscalerun.aspx) 类的一个实例，该类提供最新自动缩放运行的以下属性：
  - [AutoScaleRun.Error](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.autoscalerun.error.aspx)
  - [AutoScaleRun.Results](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.autoscalerun.results.aspx)
  - [AutoScaleRun.Timestamp](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.autoscalerun.timestamp.aspx)
- [获取有关池的信息](https://msdn.microsoft.com/library/dn820165.aspx) – 此 REST API 请求返回有关池的信息，包括最近的自动缩放运行结果。

## 示例公式

让我们看看一些示例，了解如何通过多种方式使用公式来自动缩放池中的计算资源。

### 示例 1

也许，你希望能够根据星期几和一天的具体时间来调整池的大小，相应地增加或减少池中节点的数目：

		$CurTime=time();
		$WorkHours=$CurTime.hour>=8 && $CurTime.hour<18;
		$IsWeekday=$CurTime.weekday>=1 && $CurTime.weekday<=5;
		$IsWorkingWeekdayHour=$WorkHours && $IsWeekday;
		$TargetDedicated=$IsWorkingWeekdayHour?20:10;

此公式首先获取当前时间。如果日期是工作日（周一到周五）且时间是工作时间（早 8 点到晚 6 点），则会将目标池大小设置为 20 个节点。否则，目标池大小将设置为 10 个节点。

### 示例 2.

在此示例中，池大小是根据队列中的任务数来调整的。请注意，在公式字符串中，注释和分行符都是可以接受的。

	    // Get pending tasks for the past 15 minutes.
	    $Samples = $ActiveTasks.GetSamplePercent(TimeInterval_Minute * 15);
	    // If we have less than 70% data points, we use the last sample point, otherwise we use the maximum of
		// last sample point and the history average.
	    $Tasks = $Samples < 70 ? max(0,$ActiveTasks.GetSample(1)) : max( $ActiveTasks.GetSample(1), avg($ActiveTasks.GetSample(TimeInterval_Minute * 15)));
	    // If number of pending tasks is not 0, set targetVM to pending tasks, otherwise half of current dedicated.
	    $TargetVMs = $Tasks > 0? $Tasks:max(0, $TargetDedicated/2);
	    // The pool size is capped at 20, if target VM value is more than that, set it to 20. This value
		// should be adjusted according to your use case.
	    $TargetDedicated = max(0,min($TargetVMs,20));
	    // Set node deallocation mode - keep nodes active only until tasks finish
	    $NodeDeallocationOption = taskcompletion;

### 示例 3

另一个根据任务数来调整池大小的示例就是，此公式还会考虑为池设置的 [MaxTasksPerComputeNode](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.maxtaskspercomputenode.aspx) 值。在需要针对计算节点并行执行任务时，此方法特别有用。

		// Determine whether 70% of the samples have been recorded in the past 15 minutes; if not, use last sample
		$Samples = $ActiveTasks.GetSamplePercent(TimeInterval_Minute * 15);
		$Tasks = $Samples < 70 ? max(0,$ActiveTasks.GetSample(1)) : max( $ActiveTasks.GetSample(1),avg($ActiveTasks.GetSample(TimeInterval_Minute * 15)));
		// Set the number of nodes to add to one-fourth the number of active tasks (the MaxTasksPerComputeNode
		// property on this pool is set to 4, adjust this number for your use case)
		$Cores = $TargetDedicated * 4;
		$ExtraVMs = ($Tasks - $Cores) / 4;
		$TargetVMs = ($TargetDedicated+$ExtraVMs);
		// Attempt to grow the number of compute nodes to match the number of active tasks, with a maximum of 3
		$TargetDedicated = max(0,min($TargetVMs,3));
		// Keep the nodes active until the tasks finish
		$NodeDeallocationOption = taskcompletion;

### 示例 4

此示例演示自动缩放公式在初始时间段将池大小设置为一定的节点数目，然后在初始时间段过后，根据正在运行和处于活动状态的任务数目调整池大小。

```
string now = DateTime.UtcNow.ToString("r");
string formula = string.Format(@"

	$TargetDedicated = {1};
	lifespan         = time() - time(""{0}"");
	span             = TimeInterval_Minute * 60;
	startup          = TimeInterval_Minute * 10;
	ratio            = 50;

	$TargetDedicated = (lifespan > startup ? (max($RunningTasks.GetSample(span, ratio), $ActiveTasks.GetSample(span, ratio)) == 0 ? 0 : $TargetDedicated) : {1});
	", now, 4);
```

上述代码段中的公式具有以下特征：

- 将初始池大小设置为 4 个节点
- 在池生命周期的最初 10 分钟内不调整池大小
- 10 分钟后，获取过去 60 分钟内正在运行和处于活动状态的任务数目的最大值
  - 如果这两个值均为 0（表示过去 60 分钟没有正在运行或处于活动状态的任务），则池大小将设置为 0
  - 如果其中一个值大于零，则不进行任何更改

## 后续步骤

1. 若要完全评估应用程序的效率，你可能需要访问计算节点。若要利用远程访问，必须将一个用户帐户添加到你要访问的节点，并且必须为该节点检索 RDP 文件。
    - 通过以下方法之一添加用户帐户：
        * [New-AzureBatchVMUser](https://msdn.microsoft.com/library/mt149846.aspx) – 此 PowerShell cmdlet 使用池名称、计算节点名称、帐户名和密码作为参数。
        * [BatchClient.PoolOperations.CreateComputeNodeUser](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.createcomputenodeuser.aspx) – 此 .NET 方法会创建 [ComputeNodeUser](https://msdn.microsoft.com/library/microsoft.azure.batch.computenodeuser.aspx) 类的一个实例，你可以在该实例上针对计算节点设置帐户名和密码，然后再在该实例上调用 [ComputeNodeUser.Commit](https://msdn.microsoft.com/library/microsoft.azure.batch.computenodeuser.commit.aspx)，以便在该节点上创建用户。
        * [将用户帐户添加到节点](https://msdn.microsoft.com/library/dn820137.aspx) – 池和计算节点的名称在 URI 中指定，帐户名和密码将发送到此 REST API 请求的请求正文中的节点。
    - 获取 RDP 文件：
        * [BatchClient.PoolOperations.GetRDPFile](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.getrdpfile.aspx) – 此 .NET 方法需要池 ID、节点 ID 以及要创建的 RDP 文件的名称。
        * [从节点获取远程桌面协议文件](https://msdn.microsoft.com/library/dn820120.aspx) – 此 REST API 请求需要池的名称以及计算节点的名称。响应包含 RDP 文件的内容。
        * [Get-AzureBatchRDPFile](https://msdn.microsoft.com/library/mt149851.aspx) – 此 PowerShell cmdlet 从指定的计算节点获取 RDP 文件，并将其保存到指定的文件位置或流。
2.	某些应用程序会生成大量难以处理的数据。解决此问题的方法之一是进行[有效的列表查询](/documentation/articles/batch-efficient-list-queries)。

<!---HONumber=Mooncake_1221_2015-->