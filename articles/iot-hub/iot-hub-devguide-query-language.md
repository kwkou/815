<properties
    pageTitle="了解 IoT 中心查询语言 | Azure"
    description="开发人员指南 - 介绍类似 SQL 的 IoT 中心查询语言，该语言用于在 IoT 中心检索设备孪生和作业的相关信息。"
    services="iot-hub"
    documentationcenter=".net"
    author="fsautomata"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="851a9ed3-b69e-422e-8a5d-1d79f91ddf15"
    ms.service="iot-hub"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="09/30/2016"
    wacn.date="02/10/2017"
    ms.author="elioda" />  


# 参考 - 设备孪生和作业的 IoT 中心查询语言
## 概述
IoT 中心提供类似于 SQL 的强大语言用于检索有关[设备孪生][lnk-twins]和[作业][lnk-jobs]的信息。本文内容：

* IoT 中心查询语言的主要功能简介，以及
* 语言的详细说明。

## <a name="getting-started-with-device-twin-queries"></a> 设备孪生查询入门
[设备孪生][lnk-twins]可以包含标记和属性形式的任意 JSON 对象。通过 IoT 中心，可将设备孪生作为包含所有设备孪生信息的 JSON 文档进行查询。例如，假设 IoT 中心设备孪生采用以下结构：

        {                                                                      
            "deviceId": "myDeviceId",                                            
            "etag": "AAAAAAAAAAc=",                                              
            "tags": {                                                            
                "location": {                                                      
                    "region": "CN",                                                  
                    "plant": "Redmond43"                                             
                }                                                                  
            },                                                                   
            "properties": {                                                      
                "desired": {                                                       
                    "telemetryConfig": {                                             
                        "configId": "db00ebf5-eeeb-42be-86a1-458cccb69e57",            
                        "sendFrequencyInSecs": 300                                          
                    },                                                               
                    "$metadata": {                                                   
                    ...                                                     
                    },                                                               
                    "$version": 4                                                    
                },                                                                 
                "reported": {                                                      
                    "connectivity": {                                                
                        "type": "cellular"                            
                    },                                                               
                    "telemetryConfig": {                                             
                        "configId": "db00ebf5-eeeb-42be-86a1-458cccb69e57",            
                        "sendFrequencyInSecs": 300,                                         
                        "status": "Success"                                            
                    },                                                               
                    "$metadata": {                                                   
                    ...                                                
                    },                                                               
                    "$version": 7                                                    
                }                                                                  
            }                                                                    
        }

IoT 中心将设备孪生公开为名为**设备**的文档集合。因此，以下查询将检索设备孪生的整个集：

        SELECT * FROM devices

> [AZURE.NOTE]
> [Azure IoT SDKs][lnk-hub-sdks] 支持将大量结果分页：
>
>

IoT 中心允许使用任意条件检索设备孪生筛选结果。例如，

        SELECT * FROM devices
        WHERE tags.location.region = 'CN'

检索 **location.region** 标记设置为 **CN** 的设备孪生。也支持布尔运算符和算术比较，例如

        SELECT * FROM devices
        WHERE tags.location.region = 'CN'
            AND properties.reported.telemetryConfig.sendFrequencyInSecs >= 60

检索位于中国、配置为以小于一分钟的频率发送遥测数据的所有设备孪生。方便起见，还可将数组常量与 **IN** 和 **NIN**（不包含）运算符结合使用。例如，

        SELECT * FROM devices
        WHERE property.reported.connectivity IN ['wired', 'wifi']

检索报告了 WiFi 或有线连接的所有设备孪生。通常需要它才能识别包含特定属性的所有设备孪生。为此，IoT 中心支持函数 `is_defined()`。例如，

        SELECT * FROM devices
        WHERE is_defined(property.reported.connectivity)

检索定义 `connectivity` 报告属性的所有设备孪生。有关筛选功能的完整参考，请参阅 [WHERE 子句][lnk-query-where]部分。

此外还支持分组与聚合。例如，

        SELECT properties.reported.telemetryConfig.status AS status,
            COUNT() AS numberOfDevices
        FROM devices
        GROUP BY properties.reported.telemetryConfig.status

返回处于每种遥测配置状态的设备计数。

        [
            {
                "numberOfDevices": 3,
                "status": "Success"
            },
            {
                "numberOfDevices": 2,
                "status": "Pending"
            },
            {
                "numberOfDevices": 1,
                "status": "Error"
            }
        ]

上述示例阐释了以下情景：3 个设备报告配置成功，2 个设备仍在应用配置，1 个设备报告错误。

### C# 示例
查询功能由 [C# 服务 SDK][lnk-hub-sdks] 在 **RegistryManager** 类中公开。下面是一个简单的查询示例：

        var query = registryManager.CreateQuery("SELECT * FROM devices", 100);
        while (query.HasMoreResults)
        {
            var page = await query.GetNextAsTwinAsync();
            foreach (var twin in page)
            {
                // do work on twin object
            }
        }

请注意如何使用页面大小（最大 1000）实例化 **query** 对象，然后通过调用 **GetNextAsTwinAsync** 方法多次来检索多个页面。请注意，查询对象会公开多个 **Next\***，具体取决于查询所需的反序列化选项（如设备孪生或作业对象）或者使用投影时要用的普通 JSON。

### Node.js 示例
查询功能由[适用于 Node.js 的 Azure IoT 服务 SDK][lnk-hub-sdks] 在 **Registry** 对象中公开。下面是一个简单的查询示例：

        var query = registry.createQuery('SELECT * FROM devices', 100);
        var onResults = function(err, results) {
            if (err) {
                console.error('Failed to fetch the results: ' + err.message);
            } else {
                // Do something with the results
                results.forEach(function(twin) {
                    console.log(twin.deviceId);
                });

                if (query.hasMoreResults) {
                    query.nextAsTwin(onResults);
                }
            }
        };
        query.nextAsTwin(onResults);

请注意如何使用页面大小（最大 1000）实例化 **query** 对象，然后通过调用 **nextAsTwin** 方法多次来检索多个页面。请注意，查询对象会公开多个 **Next\***，具体取决于查询所需的反序列化选项（如设备孪生或作业对象）或者使用投影时要用的普通 JSON。

### 限制
目前，仅支持在基元类型（无对象）之间进行比较，例如，仅在这些属性具有基元值时才支持 `... WHERE properties.desired.config = properties.reported.config`。

## 作业查询入门
使用[作业][lnk-jobs]可对一组设备执行操作。每个设备孪生包含名为**作业**的集合中该设备参与的作业的信息。从逻辑上讲，

        {                                                                      
            "deviceId": "myDeviceId",                                            
            "etag": "AAAAAAAAAAc=",                                              
            "tags": {                                                            
                ...                                                              
            },                                                                   
            "properties": {                                                      
                ...                                                                 
            },
            "jobs": [
                {
                    "deviceId": "myDeviceId",
                    "jobId": "myJobId",    
                    "jobType": "scheduleTwinUpdate",            
                    "status": "completed",                    
                    "startTimeUtc": "2016-09-29T18:18:52.7418462",
                    "endTimeUtc": "2016-09-29T18:20:52.7418462",
                    "createdDateTimeUtc": "2016-09-29T18:18:56.7787107Z",
                    "lastUpdatedDateTimeUtc": "2016-09-29T18:18:56.8894408Z",
                    "outcome": {
                        "deviceMethodResponse": null   
                    }                                         
                },
                ...
            ]                                                             
        }

目前，可以使用 IoT 中心查询语言以 **devices.jobs** 形式查询此集合。

> [AZURE.IMPORTANT]
> 目前，查询设备孪生（即包含“FROM devices”的查询）时不会返回作业属性。它仅可使用 `FROM devices.jobs` 通过查询直接访问。
>
>

例如，若要获取影响单个设备的所有作业（过去和计划的作业），可以使用以下查询：

        SELECT * FROM devices.jobs
        WHERE devices.jobs.deviceId = 'myDeviceId'

请注意此查询如何提供每个返回的作业的设备特定状态（可能还会提供直接方法响应）。还可针对 **devices.jobs** 集合中的所有对象属性，使用任意布尔条件进行筛选。例如，以下查询：

        SELECT * FROM devices.jobs
        WHERE devices.jobs.deviceId = 'myDeviceId'
            AND devices.jobs.jobType = 'scheduleTwinUpdate'
            AND devices.jobs.status = 'completed'
            AND devices.jobs.createdTimeUtc > '2016-09-01'

检索 2016 年 9 月后为设备 **myDeviceId** 创建的所有已完成设备孪生更新作业。

还可以检索单个作业在每个设备上的结果。

        SELECT * FROM devices.jobs
        WHERE devices.jobs.jobId = 'myJobId'

### 限制
目前，对 **devices.jobs** 的查询不支持：

* 投影，因此仅可使用 `SELECT *`。
* 引用设备孪生和作业属性的条件（参见上一节）。
* 执行聚合，例如 count、avg、group by。

## 设备到云的消息路由查询表达式入门

可通过[设备到云的路由][lnk-devguide-messaging-routes]配置 IoT 中心，根据按各消息计算的表达式将设备到云的消息分派给不同的终结点。

在克隆和作业查询中，路由[条件][lnk-query-expressions]使用相同的 IoT 中心查询语言作为条件。假设以下 JSON 表示形式，根据消息属性评估路由条件：

        {
            "userProperty1": "",
            "userProperty2": ""
        }

请记住，属性名称不区分大小写。

> [AZURE.NOTE]
> 消息属性均为字符串。如[开发人员指南][lnk-devguide-messaging-format]所述，目前不能在查询中使用系统属性。
>
>

例如，如果使用 `messageType` 属性，可能需要将所有遥测和所有警报路由到两个不同的终结点。可编写以下表达式进行遥测路由：

        messageType = 'telemetry'

编写以下表达式进行警报消息路由：

        messageType = 'alert'

此外，还支持布尔表达式和函数。通过此功能可区分严重性级别，例如：

        messageType = 'alerts' AND as_number(severity) <= 2

请参阅[表达式和条件][lnk-query-expressions]部分，了解受支持的运算符和函数的完整列表。

## IoT 中心查询基础知识
每个 IoT 中心查询包括 SELECT 和 FROM 子句，以及可选的 WHERE 和 GROUP BY 子句。每个查询针对 JSON 文档的集合（例如，设备孪生）运行。FROM 子句指示要迭代的文档集合（**devices** 或 **devices.jobs**）。然后，应用 WHERE 子句中的筛选器。使用聚合时，根据 GROUP BY 子句中的指定将此步骤的结果分组；对于每个组，将根据 SELECT 子句中的指定生成一行。

        SELECT <select_list>
        FROM <from_specification>
        [WHERE <filter_condition>]
        [GROUP BY <group_specification>]

## FROM 子句
**FROM <from\_specification>** 子句只能采用两个值：**FROM devices** - 查询设备孪生；**FROM devices.jobs** - 查询每个设备上的作业详细信息。

## <a name="where-clause"></a> WHERE 子句
**WHERE <filter\_condition>** 子句是可选的。它指定要将 FROM 集合中的 JSON 文档内含在结果中时需满足的一项或多项条件。任何 JSON 文档必须将指定的条件求值为“true”才能包含在结果中。

[表达式和条件][lnk-query-expressions]部分中介绍了允许的条件。

## SELECT 子句
SELECT 子句 (**SELECT <select_list>**) 是必需的，用于指定要从查询中检索的值。它指定用于生成新 JSON 对象的 JSON 值。对于 FROM 集合中已筛选子集（且可选择性分组）的每个元素，投影阶段将生成一个新 JSON 对象，其由 SELECT 子句中指定的值构造而成。

SELECT 子句的语法如下：

        SELECT [TOP <max number>] <projection list>

        <projection_list> ::=
            '*'
            | <projection_element> AS alias [, <projection_element> AS alias]+

        <projection_element> :==
            attribute_name
            | <projection_element> '.' attribute_name
            | <aggregate>

        <aggregate> :==
            count()
            | avg(<projection_element>)
            | sum(<projection_element>)
            | min(<projection_element>)
            | max(<projection_element>)

其中，**attribute\_name** 引用 FROM 集合中 JSON 文档的任一属性。在[设备孪生查询入门][lnk-query-getstarted]部分中可以找到 SELECT 子句的一些示例。

目前，仅支持在针对设备孪生执行的聚合查询中使用除 **SELECT *** 以外的选择子句。

## GROUP BY 子句
**GROUP BY <group\_specification>** 子句是可选步骤，可在 WHERE 子句中指定的筛选器后和在 SELECT 中指定的投影前执行该子句。它根据属性值来分组文档。这些组用于生成 SELECT 子句中指定的聚合值。

下面是使用 GROUP BY 的查询示例：

        SELECT properties.reported.telemetryConfig.status AS status,
            COUNT() AS numberOfDevices
        FROM devices
        GROUP BY properties.reported.telemetryConfig.status

GROUP BY 的正式语法为：

        GROUP BY <group_by_element>
        <group_by_element> :==
            attribute_name
            | < group_by_element > '.' attribute_name

其中，**attribute\_name** 引用 FROM 集合中 JSON 文档的任一属性。

目前，仅在查询设备孪生时才支持使用 GROUP BY 子句。

## <a name="expressions-and-conditions"></a> 表达式和条件
从较高层面讲，*表达式*可以：

* 求值为 JSON 类型的实例（例如布尔值、数字、字符串、数组或对象），以及
* 由设备 JSON 文档中的操作数据以及使用内置运算符和函数的常量定义。

*条件*是求值为布尔值的表达式。除布尔值 **true** 以外的任何常量均被视为 **false**（包括 **null**、**undefined**、任何对象或数组实例、任何字符串，当然还有布尔值 **false**）。

表达式的语法为：

        <expression> ::=
            <constant> |
            attribute_name |
            <function_call> |
            <expression> binary_operator <expression> |
            <create_array_expression> |
            '(' <expression> ')'

        <function_call> ::=
            <function_name> '(' expression ')'

        <constant> ::=
            <undefined_constant>
            | <null_constant> 
            | <number_constant> 
            | <string_constant> 
            | <array_constant> 

        <undefined_constant> ::= undefined
        <null_constant> ::= null
        <number_constant> ::= decimal_literal | hexadecimal_literal
        <string_constant> ::= string_literal
        <array_constant> ::= '[' <constant> [, <constant>]+ ']'

其中：

| 符号 | 定义 |
| --- | --- |
| attribute\_name | **FROM** 集合中 JSON 文档的任一属性。 |
| binary\_operator | [运算符](#operators)部分列出的任意二进制运算符。 |
| function\_name| [函数](#functions)部分列出的任一函数。 |
| decimal\_literal |以十进制表示法表示的浮点数。 |
| hexadecimal\_literal |以字符串“0x”后接十六进制数字符串表示的数字。 |
| string\_literal |字符串文本是以零个或多个 Unicode 字符序列或转义符序列表示的 Unicode 字符串。字符串文本括在单引号 (') 或双引号 (") 中。允许的转义符：`\'`、`"`、`\`、`\uXXXX`（由 4 个十六进制数字定义的 Unicode 字符）。 |


### <a name="operators"></a> 运算符
支持以下运算符：

| 系列 | 运算符 |
| --- | --- |
| 算术 |+,-,*,/,% |
| 逻辑 |AND、OR、NOT |
| 比较 |=、!=、<、>、<=、>=、<> |

### <a name="functions"></a> 函数
查询克隆和作业时唯一受支持的函数是：

| 函数 | 说明 |
| -------- | ----------- |
| IS\_DEFINED(property) | 返回一个布尔值，指示是否已向属性分配值（包括 `null`）。 |

在路由情况下，支持以下数学函数：

| 函数 | 说明 |
| -------- | ----------- |
| ABS(x) | 返回指定数值表达式的绝对（正）值。 |
| EXP(x) | 返回指定数值表达式 (e^x) 的指数值。 |
| POWER(x,y) | 返回指定表达式相对指定幂 (x^y) 的值。|
| SQUARE(x) | 返回指定数字值的平方。 |
| CEILING(x) | 返回大于或等于指定数值表达式的最小整数值。 |
| FLOOR(x) | 返回小于或等于指定数值表达式的最大整数。 |
| SIGN(x) | 返回指定数值表达式的正数 (+1)、零 (0) 或负数 (-1)。|
| SQRT(x) | 返回指定数字值的平方。 |

在路由情况下，支持以下检查和强制转换类型的函数：

| 函数 | 说明 |
| -------- | ----------- |
| AS\_NUMBER | 将输入字符串转换为数字。如果输入数字，则为 `noop`；如果字符串不表示数字，则为 `Undefined`。|
| IS\_ARRAY | 返回一个布尔值，指示指定表达式类型是否为数组。 |
| IS\_BOOL | 返回一个布尔值，指示指定表达式的类型是否为布尔表达式。 |
| IS\_DEFINED | 返回一个布尔，它指示属性是否已经分配了值。 |
| IS\_NULL | 返回一个布尔值，指示指定表达式的类型是否为 null。 |
| IS\_NUMBER | 返回一个布尔值，指示指定表达式的类型是否为数字。 |
| IS\_OBJECT | 返回一个布尔值，指示指定表达式的类型是否为 JSON 对象。 |
| IS\_PRIMITIVE | 返回一个布尔值，指示指定表达式的类型是否为基元（字符串、布尔值、数字或 `null`）。 |
| IS\_STRING | 返回一个布尔值，指示指定表达式的类型是否为字符串。 |

在路由情况下，支持以下字符串函数：

| 函数 | 说明 |
| -------- | ----------- |
| CONCAT(x, …) | 返回一个字符串，该字符串是连接两个或多个字符串值的结果。 |
| LENGTH(x) | 返回指定字符串表达式的字符数。|
| LOWER(x) | 返回在将大写字符数据转换为小写后的字符串表达式。 |
| UPPER(x) | 返回在将小写字符数据转换为大写后的字符串表达式。 |
| SUBSTRING(string, start [, length]) | 返回字符串表达式的部分内容，该内容起于指定字符从零开始的位置，继续到指定长度或字符串结尾。 |
| INDEX\_OF(string, fragment) | 返回第一个指定的字符串表达式中第一次出现第二个字符串表达式的起始位置，如果未找到字符串，则返回 -1。|
| STARTS\_WITH(x, y) | 返回一个布尔值，指示第一个字符串表达式是否以第二个字符串表达式开头。 |
| ENDS\_WITH(x, y) | 返回一个布尔值，指示第一个字符串表达式是否以第二个字符串表达式结尾。 |
| CONTAINS(x,y) | 返回一个布尔值，该值指示第一个字符串表达式是否包含第二个字符串表达式。 |

## 后续步骤
了解如何使用 [Azure IoT SDK][lnk-hub-sdks] 在应用中执行查询。

[lnk-query-where]: /documentation/articles/iot-hub-devguide-query-language/#where-clause
[lnk-query-expressions]: /documentation/articles/iot-hub-devguide-query-language/#expressions-and-conditions
[lnk-query-getstarted]: /documentation/articles/iot-hub-devguide-query-language/#getting-started-with-device-twin-queries

[lnk-twins]: /documentation/articles/iot-hub-devguide-device-twins/
[lnk-jobs]: /documentation/articles/iot-hub-devguide-jobs/
[lnk-devguide-endpoints]: /documentation/articles/iot-hub-devguide-endpoints/
[lnk-devguide-quotas]: /documentation/articles/iot-hub-devguide-quotas-throttling/
[lnk-devguide-mqtt]: /documentation/articles/iot-hub-mqtt-support/
[lnk-devguide-messaging-routes]: /documentation/articles/iot-hub-devguide-messaging/#device-to-cloud-configuration-options
[lnk-devguide-messaging-format]: /documentation/articles/iot-hub-devguide-messaging/#message-format

[lnk-hub-sdks]: /documentation/articles/iot-hub-devguide-sdks/

<!---HONumber=Mooncake_0206_2017-->
<!--Update_Description:update wording-->
