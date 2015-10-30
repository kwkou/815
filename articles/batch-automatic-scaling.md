
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
	ms.date="07/21/2015"
	wacn.date="10/30/2015"/>

# 自动缩放 Azure  批处理( Batch )池中的计算节点

自动缩放 Azure  批处理( Batch )池中的计算节点是指动态调整应用程序使用的处理能力。这种调整灵活性节省了时间和资金。若要了解有关计算节点和池的详细信息，请参阅 [Azure 批次技术概述](/documentation/articles/batch-technical-overview)。

如果对池启用了相应的功能并将某个公式关联到池，则会发生自动缩放。该公式用于确定处理应用程序所需的计算节点数。可以在创建池时设置自动缩放，以后也可以对现有的池设置该功能。在启用自动缩放的情况下，可在池上更新公式。

启用自动缩放后，将会根据该公式，每隔 15 分钟调整可用的计算节点数。该公式将对每隔 5 秒收集的样本执行运算，但是，每收集一个样本后并且该样本可供公式使用时，会存在 75 秒的延迟。在使用下述 GetSample 方法时，必须考虑这些时间因素。

在将公式分配到池之前，始终建议你评估该公式，并且你必须监视自动缩放的运行状态。

> [AZURE.NOTE]每个 Azure  批处理( Batch )帐户限制为可用于处理的计算节点的最大数目。系统最多会创建限制数目的节点，并且不可能会达到公式指定的目标数。

## 定义公式

使用定义的公式可以确定池中用于下一个处理间隔的可用节点数。该公式是一个字符串，其大小不能超过 8KB，最多可以包含 100 个以分号分隔的语句。

公式中的语句是自由格式的表达式。这些语句可以包含任何系统定义的变量、用户定义的变量、常量值，以及这些变量或常量支持的操作：

	VAR = Function(System defined variables, user-defined variables);

你可以组合变量来创建复杂的公式：

	VAR₀ = Function₀(system-defined variables);
	VAR₁ = Function₁(system-defined variables, VAR₀);

### 变量

可以在公式中使用系统定义的变量和用户定义的变量。可以设置这些系统定义变量的值，以管理池中的计算节点。

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
    <td>$TVMDeallocationOption</td>
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

你只能读取这些系统定义变量的值，以根据样本中计算节点的度量值进行调整。

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
    <td>$SampleTVMCount</td>
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
- timestamp
- timeinterval
	- TimeInterval_Zero
	- TimeInterval_100ns
	- TimeInterval_Microsecond
	- TimeInterval_Millisecond
	- TimeInterval_Second
	- TimeInterval_Minute
	- TimeInterval_Hour
	- TimeInterval_Day
	- TimeInterval_Week
	- TimeInterval_Year

### 操作

上面所列的类型允许的操作。

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

可以使用以下预定义函数来定义自动缩放公式。

<table>
  <tr>
    <th>函数</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>double avg(doubleVecList)</td>
    <td>DoubleVecList 中所有值的平均值。</td>
  </tr>
  <tr>
    <td>double len(doubleVecList)</td>
    <td>从 doubleVecList 创建的向量的长度。</td>
  <tr>
    <td>double lg(double)</td>
    <td>对数底数为 2。</td>
  </tr>
  <tr>
    <td>doubleVec lg(doubleVecList)</td>
    <td>分量对数底数 2。必须为单个 double 参数显式传递 vec(double)，否则将采用 double lg(double) 版本。</td>
  </tr>
  <tr>
    <td>double ln(double)</td>
    <td>自然对数。</td>
  </tr>
  <tr>
    <td>doubleVec ln(doubleVecList)</td>
    <td>分量对数底数 2。必须为单个 double 参数显式传递 vec(double)，否则将采用 double lg(double) 版本。</td>
  </tr>
  <tr>
    <td>double log(double)</td>
    <td>对数底数为 10。</td>
  </tr>
  <tr>
    <td>doubleVec log(doubleVecList)</td>
    <td>分量对数底数 10。必须为单个 double 参数显式传递 vec(double)，否则将采用 double log(double) 版本。</td>
  </tr>
  <tr>
    <td>double max(doubleVecList)</td>
    <td>doubleVecList 中的最大值。</td>
  </tr>
  <tr>
    <td>double min(doubleVecList)</td>
    <td>doubleVecList 中的最小值。</td>
  </tr>
  <tr>
    <td>double norm(doubleVecList)</td>
    <td>从 doubleVecList 创建的向量的二范数。
  </tr>
  <tr>
    <td>double percentile(doubleVec v, double p)</td>
    <td>向量 v 百分位元素。</td>
  </tr>
  <tr>
    <td>double rand()</td>
    <td>介于 0.0 和 1.0 之间的随机值。</td>
  </tr>
  <tr>
    <td>double range(doubleVecList)</td>
    <td>doubleVecList 中最小值和最大值之间的差。</td>
  </tr>
  <tr>
    <td>double std(doubleVecList)</td>
    <td>doubleVecList 中值的样本标准偏差。</td>
  </tr>
  <tr>
    <td>stop()</td>
    <td>停止自动缩放表达式计算。</td>
  </tr>
  <tr>
    <td>double sum(doubleVecList)</td>
    <td>doubleVecList 的所有组成部分之和。</td>
  </tr>
  <tr>
    <td>timestamp time(string dateTime="")</td>
    <td>如果未传递参数，则为当前时间的时间戳；如果传递了参数，则为 dateTime 字符串的时间戳。支持的 dateTime 格式为 W3CDTF 和 RFC1123。</td>
  </tr>
  <tr>
    <td>double val(doubleVec v, double i)</td>
    <td>在起始索引为零的向量 v 中，位置 i 处的元素的值。</td>
  </tr>
  <tr>
    <td>doubleVec vec(doubleVecList)</td>
    <td>从 doubleVecList 显式创建单个 doubleVec。</td>
  </tr>
</table>

表中描述的某些函数可以接受列表作为参数。逗号分隔列表为 double 和 doubleVec 的任意组合。例如：

	doubleVecList := ( (double | doubleVec)+(, (double | doubleVec) )* )?

DoubleVecList 值在计算之前将转换为单个 doubleVec。例如，如果 v = [1,2,3]，则调用 avg(v) 等效于调用 avg(1,2,3)，调用 avg(v, 7) 等效于调用 avg(1,2,3,7)。

### 样本数据

系统定义的变量是提供用于访问关联数据的方法的对象。例如，以下表达式显示了一个用于获取过去五分钟 CPU 使用率的请求：

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
    <td><p>返回数据样本的向量。例如：</p>
        <ul>
          <li><p><b>doubleVec GetSample(double count)</b> - 在最近的样本中指定所需的样本数。</p>
				  <p>一个样本最好包含 5 秒钟的度量值数据。GetSample(1) 返回最后一个可用样本，但对于像 $CPUPercent 这样的度量值，你不应使用此方法，因为无法知道样本是何时收集的。它可能是最近收集的，也可能由于系统问题而变得很旧。更好使用如下所示的时间间隔。</p></li>
          <li><p><b>doubleVec GetSample((timestamp | timeinterval) startTime [, double samplePercent])</b> – 指定收集样本数据的时间范围，并选择性地指定必须在请求范围内收集的样本百分比。</p>
          <p>如果 CPUPercent 历史记录中存在过去十分钟的所在样本，$CPUPercent.GetSample(TimeInterval_Minute*10) 应返回 200 个样本。如果最后一分钟的历史记录仍不存在，则只返回 180 个样本。</p>
					<p>$CPUPercent.GetSample(TimeInterval_Minute*10, 80) 将会成功，$CPUPercent.GetSample(TimeInterval_Minute*10,95) 将会失败。</p></li>
          <li><p><b>doubleVec GetSample((timestamp | timeinterval) startTime, (timestamp | timeinterval) endTime [, double samplePercent])</b> – 指定收集数据的时间范围，包括开始时间和结束时间。</p></li></ul></td>
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
    <p><b>doubleVec GetSamplePercent( (timestamp | timeinterval) startTime [, (timestamp | timeinterval) endTime] )</b> - 由于当返回的样本百分比小于指定的 samplePercent 时 GetSample 方法会失败，因此，你可以使用 GetSamplePercent 方法执行初始检查，然后在没有足够样本的情况下，不暂停样本自动缩放评估并执行备选操作。</p></td>
  </tr>
</table>

### 度量值

可以在公式中定义以下度量值。

<table>
  <tr>
    <th>度量值</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>资源</td>
    <td><p>根据 CPU 使用率、带宽使用率、内存使用率和计算节点的数目。可在公式中使用上述系统变量来管理池中的计算节点：</p>
    <p><ul>
      <li>$TargetDedicated</li>
      <li>$TVMDeallocationOption</li>
    </ul></p>
    <p>这些系统变量用于根据节点度量值进行调整：</p>
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
    <p>如果过去 10 分钟的最小平均 CPU 使用率高于 70%，此示例中显示的公式可将池中的计算节点数设置为当前目标节点数的 110%：</p>
    <p><b>totalTVMs = (min($CPUPercent.GetSample(TimeInterval_Minute*10)) > 0.7) ? ($CurrentDedicated * 1.1) : $CurrentDedicated;</b></p>
    <p>如果过去 60 分钟的平均 CPU 使用率低于 20%，此示例中显示的公式可将池中的计算节点数设置为当前目标节点数的 90%：</p>
    <p><b>totalTVMs = (avg($CPUPercent.GetSample(TimeInterval_Minute*60)) &lt; 0.2) ? ($CurrentDedicated * 0.9) : totalTVMs;</b></p>
    <p>此示例将专用计算节点的目标数目设置为最大值 400：</p>
    <p><b>$TargetDedicated = min(400, totalTVMs);</b></p></td>
  </tr>
  <tr>
    <td>任务</td>
    <td><p>根据任务的状态，例如“活动”、“挂起”和“已完成”。</p>
    <p>这些系统变量用于根据任务度量值进行调整：</p>
    <p><ul>
      <li>$ActiveTasks</li>
      <li>$RunningTasks</li>
      <li>$SucceededTasks</li>
      <li>$FailedTasks</li>
      <li>$CurrentDedicated</li></ul></p>
    <p>此示例中显示的公式将检测过去 15 分钟内是否已记录 70% 的样本。如果没有，则使用最后一个样本。该公式将尝试增大计算节点数，以匹配活动任务数（最大为 3）。因为池的 MaxTasksPerVM 属性设置为 4，该公式将节点数设置为活动任务数的四分之一。它还将 Deallocation 选项设置为“taskcompletion”，使计算机等到任务完成。</p>
    <p><b>$Samples = $ActiveTasks.GetSamplePercent(TimeInterval_Minute * 15); $Tasks = $Samples &lt; 70 ? max(0,$ActiveTasks.GetSample(1)) : max( $ActiveTasks.GetSample(1),avg($ActiveTasks.GetSample(TimeInterval_Minute * 15))); $Cores = $TargetDedicated * 4; $ExtraVMs = ($Tasks - $Cores) / 4; $TargetVMs = ($TargetDedicated+$ExtraVMs);$TargetDedicated = max(0,min($TargetVMs,3)); $TVMDeallocationOption = taskcompletion;</b></p></td>
  </tr>
</table>

## 评估自动缩放公式

在应用程序中使用公式之前，最好先对它进行评估。可以通过对现有池执行测试来评估公式。使用下列方法之一进行这种测试：

- [IPoolManager.EvaluateAutoScale Method](https://msdn.microsoft.com/zh-cn/library/azure/dn931617.aspx) 或 [IPoolManager.EvaluateAutoScaleAsync 方法](https://msdn.microsoft.com/zh-cn/library/azure/dn931545.aspx) 这些 .NET 方法需要现有池的名称，并需要有包含自动缩放公式的字符串。调用的结果将包含在 [AutoScaleEvaluation 类](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.autoscaleevaluation.aspx)的实例中。
- [评估自动缩放公式](https://msdn.microsoft.com/zh-cn/library/azure/dn820183.aspx) – 在这个 REST 操作中，池名称已在 URI 中指定，自动缩放公式已在请求正文的 autoScaleFormula 元素中指定。操作的响应包含任何可能与该公式相关的错误信息。

## 在启用自动缩放的情况下创建池

使用以下方法之一创建池:

- [New-AzureBatchPool](https://msdn.microsoft.com/zh-cn/library/azure/mt125936.aspx) – 此 cmdlet 使用 AutoScaleFormula 参数来指定自动缩放公式。
- [IPoolManager.CreatePool 方法](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.ipoolmanager.createpool.aspx) – 在调用此 .NET 方法创建池后，将对池设置 [ICloudPool.AutoScaleEnabled 属性](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.icloudpool.autoscaleenabled.aspx)和 [ICloudPool.AutoScaleFormula 情况](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.icloudpool.autoscaleformula.aspx)，以启用自动缩放。
- [将池添加到帐户](https://msdn.microsoft.com/zh-cn/library/azure/dn820174.aspx) – 创建池后，此 REST API 中使用的 enableAutoScale 和 autoScaleFormula 元素将为池设置自动缩放。

> [AZURE.NOTE]如果池是使用上述资源创建的，则当你设置自动缩放时，将不使用该池的 targetDedicated 参数。

## 创建池后启用自动缩放

如果你使用 targetDedicated 参数设置了包含指定计算节点数的池，则以后可以更新现有池以自动缩放。可以通过以下方法之一实现此目的：

- [IPoolManager.EnableAutoScale 方法](https://msdn.microsoft.com/zh-cn/library/azure/dn931709.aspx) – 此 .NET 方法需要现有池的名称和自动缩放公式。
- [启用/禁用自动缩放](https://msdn.microsoft.com/zh-cn/library/azure/dn820173.aspx) – 此 REST API 需要 URI 中现有池的名称，以及请求正文中的自动缩放公式。

> [AZURE.NOTE]在评估自动缩放公式时，将忽略创建池时为 targetDedicated 参数指定的值。

## 获取有关自动缩放运行的信息

应定期检查自动缩放运行的结果。通过以下方法之一执行这种检查：

- [ICloudPool.AutoScaleRun 属性](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.icloudpool.autoscalerun.aspx) – 在使用 .NET 库时，池的这个属性将提供 [AutoScaleRun 类](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.autoscalerun.aspx)的实例，该实例提供 [AutoScaleRun.Error 属性](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.autoscalerun.error.aspx)、[AutoScaleRun.Results 属性](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.autoscalerun.results.aspx)和 [AutoScaleRun.Timestamp 属性](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.autoscalerun.timestamp.aspx)。
- [获取有关池的信息](https://msdn.microsoft.com/zh-cn/library/dn820165.aspx) – 此 REST API 返回有关池的信息，包括最近的自动缩放运行结果。

## 后续步骤

1.	你可能需要访问计算节点才能完全评估应用程序的效率。若要利用远程访问，必须将一个用户帐户添加到你要访问的计算节点，并且必须从该节点检索 RDP 文件。通过以下方法之一添加用户帐户：

	- [New-AzureBatchVMUser](https://msdn.microsoft.com/zh-cn/library/mt149846.aspx) – 此 cmdlet 使用池名称、计算节点名称、帐户名和密码作为参数。
	- [IVM.CreateUser Method](https://msdn.microsoft.com/zh-cn/library/microsoft.azure.batch.ivm.createuser.aspx) – 此 .NET 方法创建 [IUser 接口](https://msdn.microsoft.com/zh-cn/library/microsoft.azure.batch.iuser.aspx)的实例，可在该实例上设置计算节点的帐户名和密码。
	- [将用户帐户添加到节点](https://msdn.microsoft.com/zh-cn/library/dn820137.aspx) – 池和计算节点的名称在 URI 中指定，帐户名和密码将发送到此 REST API 的请求正文中的节点。

		获取 RDP 文件：

	- [IVM.GetRDPFile 方法](https://msdn.microsoft.com/zh-cn/library/microsoft.azure.batch.ivm.getrdpfile.aspx) – 此 .NET 方法需使用要创建的 RDP 文件的名称。
	- [从节点获取远程桌面协议文件](https://msdn.microsoft.com/zh-cn/library/dn820120.aspx) – 此 REST API 需要池的名称以及计算节点的名称。响应包含 RDP 文件的内容。
	- [Get-AzureBatchRDPFile](https://msdn.microsoft.com/zh-cn/library/mt149851.aspx) – 此 cmdlet 从指定的计算节点获取 RDP 文件，并将其保存到指定的文件位置或流。
2.	某些应用程序会生成大量难以处理的数据。解决此问题的方法之一是进行[有效的列表查询](/documentation/articles/batch-efficient-list-queries)。

<!---HONumber=66-->