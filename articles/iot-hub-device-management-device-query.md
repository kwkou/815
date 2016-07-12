<properties
	pageTitle="IoT 中心设备管理设备克隆查询 | Azure"
	description="Azure IoT 中心设备管理教程，描述如何使用查询查找设备克隆。"
	services="iot-hub"
	documentationCenter=".net"
	authors="ellenfosborne"
	manager="timlt"
	editor=""/>

<tags
 ms.service="iot-hub"
 ms.date="04/29/2016"
 wacn.date="05/30/2016"/>

# 教程：如何将查询用于 C# 以查找设备克隆（预览版）

[AZURE.INCLUDE [iot-hub-device-management-query-selector](../includes/iot-hub-device-management-query-selector.md)]

通过 Azure IoT 设备管理，可通过查询查找设备克隆，即物理设备的服务表示形式。可根据设备属性、服务属性或中设备克隆中的标记进行查询。可使用标记和属性进行查询：

-   若要使用标记查询设备克隆，请传递字符串数组，查询将返回一组标记了所传递全部字符串的设备。

-   若要使用服务属性或设备属性查询设备，请使用 JSON 查询表达式。

有关设备克隆和查询的详细信息，请参阅 [Azure IoT 中心设备管理概述][lnk-dm-overview]。

## 运行设备查询示例

下面的示例扩展了 [Azure IoT 中心设备管理入门][lnk-get-started]教程的功能。从使各模拟设备运行开始，它将使用查询来查找特定设备。

### 先决条件 

运行此示例之前，请务必完成 [Azure IoT 中心设备管理入门][lnk-get-started]中的步骤。这意味着必须运行模拟设备。如果你以前完成了该进程，现在请重启模拟设备。

### 启动示例

若要启动该示例，则需要运行 **Query.exe** 进程。这将执行几个不同的查询。按照以下步骤启动示例：

1.  让模拟设备运行至少 1 分钟。这可确保设备克隆中的设备属性与物理设备同步。请参阅[教程：如何使用设备克隆][lnk-twin-tutorial]，以了解有关同步的详细信息。

2.  从克隆 **azure-iot-sdks** 存储库的根文件夹，导航到 **azure-iot-sdks\\csharp\\service\\samples\\bin** 文件夹。

3.  运行 `Query.exe <IoT Hub Connection String>`

应可看到命令行窗口中的输出显示了使用标记、服务属性和设备属性的设备对象查询的结果。

## 查询结构 (JSON)

执行设备属性和服务属性的查询时使用 JSON 字符串表示查询本身。该 JSON 字符串由 4 个部分组成。下面是每个部分以及可以序列化为正确的 JSON 字符串的 C# 对象的说明。

- **Project**：用于指定设备对象中要包含在查询结果集中的字段的表达式（相当于 SQL 中的 SELECT）；

	```
	  var query = JsonConvert.SerializeObject(
		  new
		  {
			  project = new
			  {
				  all = false,
				  properties = new[]
				  {
					  new
					  {
					  name = "CustomerId",
					  type = "service"
					  },
					  new
					  {
					  name = "Weight",
					  type = "service"
					  }
				  }
			  }
		  }
	  );
	```

- **Filter**：用于限制查询结果集中包含的设备对象的表达式（相当于 SQL 中的 WHERE）；

  ```
  var query = JsonConvert.SerializeObject(
      new
      {
        filter = new
        {
          property = new
          {
            name = "CustomerId",
            type = "service"
          },
          value = "123456",
          comparisonOperator = "eq",
          type = "comparison"
        }
      }
  );
  ```

- **Aggregate**：用于确定如何对查询结果集进行分组的表达式（相当于 SQL 中的 GROUP BY）；

  ```
  var query = JsonConvert.SerializeObject(
      new
      {
      filter = new
        {
          property = new
          {
            name = "CustomerId",
            type = "service"
          },
          value = (string)null,
          comparisonOperator = "ne",
          type = "comparison"
        },
        aggregate = new
        {
          keys = new[]
          {
            new
            {
              name = "CustomerId",
              type = "service"
            }
          },
          properties = new[]
          {
            new
            {
              @operator = "avg",
              property = new
              {
                name = "Weight",
                type = "service"
              },
              columnName = "TotalWeight"
            }
          }
        }
      }
  );
  ```

- **Sort**：用于定义应用于对查询结果进行排序的属性的表达式（相当于 SQL 中的 ORDER BY）。如果 sort 为 null，则默认使用 **deviceID**：

  ```
  var query = JsonConvert.SerializeObject(
    new
    {
      sort = new[]
      {
        new
        {
          property = new
          {
            name = "QoS",
            type = "service"
          },
          order = "asc"
        }
      }
    }
  );
  ```

### 限制

查询的公共预览版实现中存在一些限制。

-   不会对查询表达式进行验证。

-   查询区分大小写。

-   使用查询表达式按服务或设备属性查询时，将仅返回 100 个设备。[我们的查询库][lnk-query-samples]中提供了有关如何实现分页的示例。

还[提供][lnk-query-expression-guide]了有关 JSON 的语法和可用字段更多详细信息。还可在我们的[查询表达式库][lnk-query-samples]中查看示例查询。

### 按设备和服务属性查询

拥有 JSON 查询表达式后，即可查询设备克隆。调用 **QueryDeviceJsonAsync** 并检查“AggregateResult”字段的聚合查询和“结果”字段的所有其他查询。“结果”包含表示与查询匹配的设备克隆的设备对象列表。“AggregateResult”包含大量字典，每个字典包含生成的行。

这些查询用于 **Query** 对象的 **Program.cs**。

```
var foundDevices = (await registryManager.QueryDevicesJsonAsync(query)).Result;

var results = (await registryManager.QueryDevicesJsonAsync(query)).AggregateResult;
```

### 按标记查询

通过按标记查询，可以不使用 JSON 查询表达式而查找设备对象。如果传递了多个标记，则将仅返回具有所有标记的设备对象。第二个参数是 **maxCount**，表示将返回的设备最大数量。**maxCount** 的最大值为 1000。

```
var foundDevices = await registryManager.QueryDevicesAsync(new[] { "bacon" }, 100);
```

### 设备实现

通过 Azure IoT 中心设备管理客户端库启用查询。只要同步了设备属性（如[教程：如何使用设备克隆][lnk-twin-tutorial]中所述），即可对其进行查询。仅当物理设备连接到 IoT 中心并提供初始值后，设备属性才可用。

## 后续步骤

若要了解有关 Azure IoT 中心设备管理功能的详细信息，可学习以下教程：

- [如何使用设备克隆][lnk-twin-tutorial]
- [如何使用设备作业更新设备固件][lnk-jobs-tutorial]



<!-- images and links -->

[lnk-dm-overview]: /documentation/articles/iot-hub-device-management-overview/
[lnk-get-started]: /documentation/articles/iot-hub-device-management-get-started/
[lnk-twin-tutorial]: /documentation/articles/iot-hub-device-management-device-twin/
[lnk-jobs-tutorial]: /documentation/articles/iot-hub-device-management-device-jobs/
[lnk-query-spec]: https://github.com/Azure/azure-iot-sdks/blob/dmpreview/node/service/devdoc/query_expression_requirements.md
[lnk-query-samples]: https://github.com/Azure/azure-iot-sdks/blob/dmpreview/doc/get_started/dm_queries/query-samples.md
[lnk-query-expression-guide]: https://github.com/Azure/azure-iot-sdks/blob/dmpreview/node/service/devdoc/query_expression_requirements.md

<!---HONumber=Mooncake_0523_2016-->