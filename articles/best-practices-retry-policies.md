<properties
   pageTitle="NuGet 程序包 |Windows Azure"
   description="常规重试策略工作 NuGet 程序包指南。"
   services=""
   documentationCenter="na"
   authors="dragon119"
   manager="masimms"
   editor=""
   tags=""/>

<tags
   ms.service="best-practice"
   ms.date="04/09/2015"
   wacn.date="08/29/2015"/>

# NuGet 程序包

<p class="lead">随着更多的组件开始通信，必须巧妙地处理各种暂时性故障，这很重要。可通过重试策略来处理暂时性故障处理工作，而 NuGet 程序包可帮助处理单个实例中的重试。</p>

> 本文档基于一个可用作概念证明的草稿。它不是实际的已审阅的指南。

建议将模式与实践 `TransientFaultHandling` 规范用于常规的重试策略工作。

```
Install-Package EnterpriseLibrary.WindowsAzure.TransientFaultHandling
```

## 配置

此部分包括重试功能的配置信息：

参数 | 说明
-------------------- | ----------------------
MaximumExecutionTime | 请求的最长执行时间，包括所有可能的重试尝试。
ServerTimeOut | 请求的服务器超时间隔
RetryPolicy | 重试策略。请参阅下面的策略部分

```csharp
/// <summary>
/// An interface required for request option types.
/// </summary>
public interface IRequestOptions
{
    IRetryPolicy RetryPolicy { get; set; }

    TimeSpan? ServerTimeout { get; set; }

    TimeSpan? MaximumExecutionTime { get; set; }
}
```

以编程方式：

- 支持客户端上的设置。
- 允许在客户端所提供操作的基础上进行重写

配置文件：

```xml
<RetryPolicyConfiguration defaultRetryStrategy="Fixed Interval Retry Strategy">
    <linearInterval name="Fixed Interval Retry Strategy"
	retryInterval="00:00:01" maxRetryCount="10" />
    <exponentialBackoff name="Backoff Retry Strategy" minBackoff="00:00:01"
        maxBackoff="00:00:30" deltaBackoff="00:00:10" maxRetryCount="10"
        fastFirst="false"/>
</RetryPolicyConfiguration>
```

## 策略

### 指数

用于隔开重复的其数目呈指数增长的服务调用尝试，以免出现服务被遏制的情况。

__方法：__

在后续尝试之间让回退时间间隔呈指数增加。给回退时间间隔增加随机性 (+/- 20%)，以免所有客户端在同一时间进行重试

__配置:__

参数 | 说明
-------------------- | -------------------------------------------------------
maxAttempt | 重试的次数。
deltaBackoff | 不同重试之间的回退时间间隔。将针对后续的重试使用多个这样的时间跨度。
MinBackoff | 添加到从 deltaBackoff 计算的所有重试时间间隔。
FastFirst | 立即进行第一次重试
MaxBackoff | 如果计得的重试时间间隔大于 MaxBackoff，则使用 MaxBackoff。不能更改此值。

__实现逻辑：__

```csharp
if(!ExponentialRetry.FastFirst){
    Random r = new Random();
    double increment = (Math.Pow(2, currentRetryCount) - 1) * r.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8), (int)(this.deltaBackoff.TotalMilliseconds * 1.2));
    retryInterval = (increment < 0) ? ExponentialRetry.MaxBackoff :
    TimeSpan.FromMilliseconds(Math.Min(ExponentialRetry.MaxBackoff.TotalMilliseconds, ExponentialRetry.MinBackoff.TotalMilliseconds + increment));
} else {
    retryInterval = TimeSpan.Zero;
}
```

## 线性

用于隔开重复的其数目呈线性增长的服务调用尝试，以免出现服务被遏制的情况。

__方法：__

执行指定数目的重试，在不同重试之间使用所指定的固定时间间隔。给回退时间间隔增加随机性 (+/- 20%)，以免所有客户端在同一时间进行重试。

__配置:__

参数 | 说明
-------------------- | -------------------------------------------------------
maxAttempt | 重试的次数。
deltaBackoff | 不同重试之间的回退时间间隔。
FastFirst | 立即进行第一次重试

__实现逻辑：__

```csharp
if(!ExponentialRetry.FastFirst) {
    Random r = new Random();
    retryInterval = TimeSpan.FromMilliseconds(r.Next((int)(
    this.deltaBackoff.TotalMilliseconds * 0.8), (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
} else {
    retryInterval = TimeSpan.Zero;
}
```

## 自适应

用于根据服务在响应标头中传递的错误代码/元数据，隔开重复的服务调用尝试。

__方法：__

执行指定数目的重试，使用的回退时间间隔根据服务在响应标头中传递的错误代码/元数据来计算


__配置:__

不能配置

__实现逻辑：__

根据服务在响应标头中传递的错误代码/元数据

__线路中断：__

基于[断路器](http://msdn.microsoft.com/zh-cn/library/dn589784.aspx)

## 可扩展性

一种公共接口，可以在实现后提供自定义重试策略

```csharp
public interface IRetryPolicy
{
    /// <summary>
    /// Generates a new retry policy for the current request attempt.
    /// </summary>
    IRetryPolicy CreateInstance();

    /// <summary>
    /// Determines whether the operation should be retried and the interval until the next retry.
    /// </summary>
    /// <param name="currentRetryCount">An integer specifying the number of retries for the given operation. A value of zero signifies this is the first error encountered.</param>
    /// <param name="statusCode">An integer containing the status code for the last operation.</param>
    /// <param name="retryInterval">A <see cref="TimeSpan"/> indicating the interval to wait until the next retry.</param>
    /// <returns><c>true</c> if the operation should be retried; otherwise, <c>false</c>.</returns>
    bool ShouldRetry(int currentRetryCount, int statusCode, out TimeSpan retryInterval);
}
```

## 遥测

使用 EventSource 以 ETW 事件形式记录重试。以下是每次重试尝试时都应记录的字段

参数 | 说明
-------------------- | -------------------------------------------------------
requestId | ""
policyType | "RetryExponential"
operation | "Get:https://retry-guidance-tests.servicebus.chinacloudapi.cn/TestQueue/?api-version=2014-05"
operationStartTime | "9/5/2014 10:00:13 PM"
operationEndTime | "9/5/2014 10:00:14 PM"
iteration | "0"
iterationSleep | "00:00:00.1000000"
lastExceptionType | "Microsoft.ServiceBus.Messaging.MessagingCommunicationException"
exceptionMessage | "The remote name could not be resolved: 'retry-guidance-tests.servicebus.chinacloudapi.cn'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3,TimeStamp:9/5/2014 10:00:13 PM"

<!---HONumber=67-->