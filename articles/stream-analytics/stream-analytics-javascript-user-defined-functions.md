<properties
    pageTitle="Azure 流分析 JavaScript 用户定义的函数 | Azure"
    description="使用 JavaScript 用户定义的函数执行高级查询机制"
    keywords="javascript, 用户定义的函数, udf"
    services="stream-analytics"
    author="jeffstokes72"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid=""
    ms.service="stream-analytics"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="data-services"
    ms.date="02/01/2017"
    wacn.date="03/10/2017"
    ms.author="jeffstok" />  


# Azure 流分析 JavaScript 用户定义的函数
Azure 流分析支持以 JavaScript 编写的用户定义的函数。利用 JavaScript 提供的丰富 **String**、**RegExp**、**Math**、**Array** 和 **Date** 方法，可以更轻松地创建包含流分析作业的复杂数据转换。

## JavaScript 用户定义的函数
JavaScript 用户定义的函数支持仅用于计算的且不需要外部连接的无状态标量函数。函数的返回值只能是标量（单个）值。将某个 JavaScript 用户定义的函数添加到作业后，可在查询中的任意位置使用该函数，如同内置标量函数一样。

下面是 JavaScript 用户定义的函数可派上用场的一些情景：
* 使用 **Regexp\_Replace()** 和 **Regexp\_Extract()** 等正则表达式函数分析和处理字符串
* 解码和编码数据，例如，二进制到十六进制的转换
* 使用 JavaScript **Math** 函数执行数学计算
* 执行排序、联接、查找和填充等数组操作

使用流分析中的 JavaScript 用户定义的函数无法实现以下目的：
* 调用外部 REST 终结点，例如，执行反向 IP 查找，或者从外部源提取引用数据
* 针对输入/输出执行自定义事件格式序列化或反序列化
* 创建自定义聚合

尽管函数定义中并不禁止 **Date.GetDate()** 或 **Math.random()** 等函数，但应该避免使用这些函数。这些函数在每次被调用时**不会**返回相同的结果，并且 Azure 流分析服务不会保留函数调用和返回结果的日记。当你或流分析服务重新启动某个作业时，如果某个函数针对相同的事件返回不同的结果，将无法保证可重复性。

## 在 Azure 门户中添加 JavaScript 用户定义的函数
若要在现有的流分析作业中创建一个简单的 JavaScript 用户定义的函数，请执行以下步骤：

1. 在 Azure 门户中找到你的流分析作业。
2. 在“作业拓扑”下面选择你的函数。此时将显示一个空白的函数列表。
3. 若要新建用户的定义函数，请选择“添加”。
4. 在“新建函数”边栏选项卡中，为“函数类型”选择“JavaScript”。编辑器中将显示默认函数模板。
5. 为“UDF 别名”输入 **hex2Int**，然后按如下所示更改函数实现：

        // Convert Hex value to integer.
        function main(hexValue) {
            return parseInt(hexValue, 16);
        }

6. 选择“保存”。该函数随即显示在函数列表中。
7. 选择新的 **hex2Int** 函数并检查函数定义。所有函数的函数别名带有 **UDF** 前缀。在流分析查询中调用该函数时，需要*包含该前缀*。在本例中，调用的是 **UDF.hex2Int**。

## 在查询中调用 JavaScript 用户定义的函数

1. 在查询编辑器中的“作业拓扑”下面选择“查询”。
2. 编辑查询，然后调用该用户定义的函数，如下所示：

        SELECT
            time,
            UDF.hex2Int(offset) AS IntOffset
        INTO
            output
        FROM
            InputStream

3. 若要上载示例数据文件，请右键单击作业输入。
4. 若要测试查询，请选择“测试”。

## 支持的 JavaScript 对象
Azure 流分析 JavaScript 用户定义的函数支持标准的内置 JavaScript 对象。有关这些对象的列表，请参阅 [Global Objects](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects)（全局对象）。

### 流分析和 JavaScript 类型转换

流分析查询语言与 JavaScript 支持的类型有差别。下表列出了两者之间的转换映射：

流分析 | JavaScript
--- | ---
bigint | 数字（JavaScript 只能精确呈现最大 2^53 的整数）
DateTime | 日期（JavaScript 仅支持毫秒）
double | Number
nvarchar(MAX) | String
记录 | 对象
Array | Array
Null | Null

下面是 JavaScript 到流分析的转换：

JavaScript | 流分析
--- | ---
Number | 如果数字已舍入并介于 long.MinValue 和 long.MaxValue 之间，则为 Bigint；否则为 double日期
日期 | DateTime
String | nvarchar(MAX)
对象 | 记录
Array | Array
Null、Undefined | Null
其他任何类型（例如函数或错误） | 不支持（导致运行时错误）

## 故障排除
JavaScript 运行时错误被视为严重错误，可通过活动日志查看。若要检索日志，请在 Azure 门户中转到你的作业，然后选择“活动日志”。

## JavaScript 用户定义的函数的其他模式

### 编写要输出的嵌套 JSON
如果后续处理步骤需要使用流分析作业输出作为输入并且要求采用 JSON 格式，可以编写要输出的 JSON 字符串。以下示例调用 **JSON.stringify()** 函数封装输入的所有名称/值对，然后将其写入为输出中的单个字符串值。

**JavaScript 用户定义的函数定义：**

    function main(x) {
    return JSON.stringify(x);
    }

**示例查询：**

    SELECT
        DataString,
        DataValue,
        HexValue,
        UDF.json_stringify(input) As InputEvent
    INTO
        output
    FROM
        input PARTITION BY PARTITIONID

## 获取帮助
如需更多帮助，请访问我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/home?forum=AzureStreamAnalytics)。

## 后续步骤
* [Azure 流分析简介](/documentation/articles/stream-analytics-introduction/)
* [Azure 流分析入门](/documentation/articles/stream-analytics-get-started/)
* [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs/)
* [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
* [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description: new article about how to define user function with javascript-->