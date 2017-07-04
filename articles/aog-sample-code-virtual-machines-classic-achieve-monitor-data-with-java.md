<properties
    pageTitle="使用 Java 获取经典模式虚拟机监视数据"
    description="如何通过 Java 获取经典模式虚拟机监视数据"
    service=""
    resource="virtualmachines"
    authors="Chen Rui"
    displayOrder=""
    selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Sample Code, Virtual Machines, Classic, Java, REST API, Monitor"
    cloudEnvironments="MoonCake" />
<tags
    ms.service="sample-code-aog"
    ms.date=""
    wacn.date="04/18/2017" />

# 使用 Java 获取经典模式虚拟机监视数据

在 .Net 中可以借助 Monitor Package 来获取 Azure 经典虚拟机的监视数据，但是在 Java 的 SDK 中并没有封装该部分 API，我们可以使用 Azure 相关的 API 来获取该部分数据。

Azure 中提供两个可以获取监视数据的 API，一个是用于获取经典虚拟机支持的监视规则定义，另外一个是用于获取经典虚拟机的监视数据。以下分别介绍这两个 REST API。


## 获取经典虚拟机支持的监视规则定义

### Endpoint

https://management.core.chinacloudapi.cn/{subscription}/services/monitoring/metricdefinitions/query?resourceId={resource id}

### 请求头部参数

| 参数名称 | 参数值 |
|:------------:|:----------:|
| x-ms-version | 2013-10-01 |
| Content-Type | application/xml |

### 请求参数

| 参数名称 | 参数值 |
|:------------:|:----------:|
| resourceId | 资源 ID，示例：/hostedservices/{hostedservice name}/deployments/{deployment name}/roles/{role name} |

### 调用示例

    String filter =  String.format("resourceId=/hostedservices/%s/deployments/%s/roles/%s",
            "kevin-vs-never",
            "kevin-vs-never",
            "kevin-vs-never");
    URI uri = new URI("https", "management.core.chinacloudapi.cn",
            String.format("/%s/services/monitoring/metricdefinitions/query", AbstactTest.SUB_ID), filter, null);
    URL url = uri.toURL();
    Map<String, String> params = new HashMap<String, String>();
    params.put("x-ms-version", "2013-10-01");
    params.put("Content-Type", "application/xml");
    String result = AzureRestClient.processGetRequest(url, params);

### 示例代码

GetMetricDefineOldTest : [https://github.com/wacn/AOG-CodeSample/blob/master/AzureServiceManager/Java/azure-service-manager/src/test/java/com/vianet/azure/sdk/manage/monitor/TestVMMetric.java#L73](https://github.com/wacn/AOG-CodeSample/blob/master/AzureServiceManager/Java/azure-service-manager/src/test/java/com/vianet/azure/sdk/manage/monitor/TestVMMetric.java#L73)

## 获取经典虚拟机的监视数据

### Endpoint

https://management.core.chinacloudapi.cn/{subscription}/services/monitoring/metricdefinitions/query?resourceId={resource id}&names={names}/ &timeGrain={timeGrain}&startTime={startTime}&endTime={endTime}

### 请求头部参数

| 参数名称 | 参数值 |
|:------------:|:----------:|
| x-ms-version | 2013-10-01 |
| Content-Type | application/xml |

### 请求参数

| 参数名称 | 参数值 |
|:------------:|----------|
| resourceId | 资源 ID，示例：/hostedservices/{hostedservice}/deployments/{deployment}/roles/{role} |
| names | 监视规则，可以多个规则，以逗号分隔，示例：Network In,Network Out,Disk Read Bytes/sec,Disk Write Bytes/sec |
| timeGrain | 值为 PT1H, PT5M |
| startTime | 起始时间，示例：2017-04-02T00:00:00.0000000Z |
| endTime | 结束时间，示例：2017-04-03T00:00:00.0000000Z |

### 调用示例

    String filter =  String.format("resourceId=/hostedservices/%s/deployments/%s/roles/%s&namespace=%s&names=%s&timeGrain=%s&startTime=%s&endTime=%s",
            "kevin-vs-never",
            "kevin-vs-never",
            "kevin-vs-never",
            "",
            "Network In,Network Out,Disk Read Bytes/sec,Disk Write Bytes/sec",
            "PT1H",
            "2017-04-03T00:00:00.0000000Z",
            "2017-04-05T23:59:59.0000000Z");
    URI uri = new URI("https", "management.core.chinacloudapi.cn",
            String.format("/%s/services/monitoring/metricvalues/query", AbstactTest.SUB_ID), filter, null);
    URL url = uri.toURL();
    Map<String, String> params = new HashMap<String, String>();
    params.put("x-ms-version", "2013-10-01");
    params.put("Content-Type", "application/xml");
    String result = AzureRestClient.processGetRequest(url, params);

### 示例代码

GetMetricOldTest :[https://github.com/wacn/AOG-CodeSample/blob/master/AzureServiceManager/Java/azure-service-manager/src/test/java/com/vianet/azure/sdk/manage/monitor/TestVMMetric.java#L89](https://github.com/wacn/AOG-CodeSample/blob/master/AzureServiceManager/Java/azure-service-manager/src/test/java/com/vianet/azure/sdk/manage/monitor/TestVMMetric.java#L89)
