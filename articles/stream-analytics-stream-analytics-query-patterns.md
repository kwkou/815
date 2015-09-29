<properties
	pageTitle="Azure 流分析查询模式 | Windows Azure"
	description="常见的 Azure 流分析查询模式"
	keywords="stream analytics, sample, query, language, guide, patterns"
	services="stream-analytics"
	documentationCenter=""
	authors="jeffstokes72"
	manager="paulettm"
	editor="cgronlun"/>

<tags
	ms.service="stream-analytics"
	ms.date="08/19/2015"
	wacn.date="09/15/2015"/>


# 常见的 Azure 流分析查询模式  #

## 介绍 ##
Azure 流分析中的查询采用类似 SQL 的查询语言来表述，该语言的文档说明见[此处](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)。本文档概述了针对多个常见查询模式的解决方案，依据的是真实的场景。此工作仍在进行中，将继续使用新的模式不断进行更新。

## 基础知识 ##

## 数据类型转换 ##
**说明**：定义输入流中的属性类型，例如，车重在输入流中是字符串格式的，需要转换为 INT 才能执行 SUM 操作。

**输入**：

| 请确 | 时间 | 重量 |
| --- | --- | --- |
| Honda | 2015-01-01T00:00:01.0000000Z | "1000" |
| Honda | 2015-01-01T00:00:02.0000000Z | "2000" |

**输出**：

| 请确 | 重量 |
| --- | --- |
| Honda | 3000 |

**解决方案**：

	SELECT
    	Make,
    	SUM(CAST(Weight AS BIGINT)) AS Weight
	FROM
    	Input TIMESTAMP BY Time
	GROUP BY
		Make,
    	TumblingWindow(second, 10)

**说明**：对“重量”字段使用 CAST 语句，以便指定其类型（请在[此处](https://msdn.microsoft.com/zh-cn/library/azure/dn835065.aspx)查看受支持数据类型的列表）。

## 使用 Like/Not like 进行模式匹配 ##
**说明**：查看事件中的某个字段值是否符合特定模式，例如，返回以 A 开头，以 9 结尾的牌照

**输入**：

| 请确 | LicensePlate | 时间 |
| --- | --- | --- |
| Honda | ABC-123 | 2015-01-01T00:00:01.0000000Z |
| Toyota | AAA-999 | 2015-01-01T00:00:02.0000000Z |
| Nissan | ABC-369 | 2015-01-01T00:00:03.0000000Z |

**输出**：

| 请确 | LicensePlate | 时间 |
| --- | --- | --- |
| Toyota | AAA-999 | 2015-01-01T00:00:02.0000000Z |
| Nissan | ABC-369 | 2015-01-01T00:00:03.0000000Z |

**解决方案**：

	SELECT
    	*
	FROM
    	Input TIMESTAMP BY Time
	WHERE
    	LicensePlate LIKE 'A%9'

**说明**：使用 LIKE 语句来查看 LicensePlate 字段值是否以 A 开头、后面所跟的字符串是否包含 0 个或 0 个以上字符，以及是否以 9 结尾。

## 指定不同案例/值的逻辑（CASE 语句） ##
**说明**：根据某些标准为字段提供不同的计算操作。例如，提供一个字符串说明，说明有多少辆制造商相同的车通过，特殊情况下该数目为 1。

**输入**：

| 请确 | 时间 |
| --- | --- |
| Honda | 2015-01-01T00:00:01.0000000Z |
| Toyota | 2015-01-01T00:00:02.0000000Z |
| Toyota | 2015-01-01T00:00:03.0000000Z |

**输出**：

| CarsPassed | 时间 |
| --- | --- | --- |
| 1 辆 Honda | 2015-01-01T00:00:10.0000000Z |
| 2 辆 Toyota | 2015-01-01T00:00:10.0000000Z |

**解决方案**：

    SELECT
    	CASE
			WHEN COUNT(*) = 1 THEN CONCAT('1 ', Make)
			ELSE CONCAT(CAST(COUNT(*) AS NVARCHAR(MAX)), ' ', Make, 's')
		END AS CarsPassed,
		System.TimeStamp AS Time
	FROM
		Input TIMESTAMP BY Time
	GROUP BY
		Make,
		TumblingWindow(second, 10)

**说明**：可以通过 CASE 子句根据某些标准（在我们这个示例中为聚合窗口中车辆的计数）来提供不同的计算操作。

## 将数据发送到多个输出 ##
**说明**：将数据从单个作业发送到多个输出目标。例如，对数据进行分析，根据阈值发出警报，并将所有事件存档到 blob 存储

**输入**：

| 请确 | 时间 |
| --- | --- |
| Honda | 2015-01-01T00:00:01.0000000Z |
| Honda | 2015-01-01T00:00:02.0000000Z |
| Toyota | 2015-01-01T00:00:01.0000000Z |
| Toyota | 2015-01-01T00:00:02.0000000Z |
| Toyota | 2015-01-01T00:00:03.0000000Z |

**输出 1**：

| 请确 | 时间 |
| --- | --- |
| Honda | 2015-01-01T00:00:01.0000000Z |
| Honda | 2015-01-01T00:00:02.0000000Z |
| Toyota | 2015-01-01T00:00:01.0000000Z |
| Toyota | 2015-01-01T00:00:02.0000000Z |
| Toyota | 2015-01-01T00:00:03.0000000Z |

**输出 2**：

| 请确 | 时间 | 计数 |
| --- | --- | --- |
| Toyota | 2015-01-01T00:00:10.0000000Z | 3 |

**解决方案**：

	SELECT
		*
	INTO
		ArchiveOutput
	FROM
		Input TIMESTAMP BY Time

	SELECT
		Make,
		System.TimeStamp AS Time,
		COUNT(*) AS [Count]
	INTO
		AlertOutput
	FROM
		Input TIMESTAMP BY Time
	GROUP BY
		Make,
		TumblingWindow(second, 10)
	HAVING
		[Count] >= 3

**说明**：INTO 子句告诉流分析哪一个输出可以通过此语句写入数据。第一个查询将我们接收到的数据传递到名为 ArchiveOutput 的输出。第二个查询进行了一些简单的聚合和筛选操作，然后将结果发送到下游的报警系统。*注意*：你可以重复使用多个输出语句中的 CTE（即 WITH 语句）结果 – 这有一些额外的好处，例如，在读取输入源时，可以打开较少的读取器。

	WITH AllRedCars AS (
		SELECT
			*
		FROM
			Input TIMESTAMP BY Time
		WHERE
			Color = 'red'
	)
	SELECT * INTO HondaOutput FROM AllRedCars WHERE Make = 'Honda'
	SELECT * INTO ToyotaOutput FROM AllRedCars WHERE Make = 'Toyota'

## 模式 ##

## 对唯一值进行计数
**说明**：计算某个时段内出现在流中的唯一字段值的数目。例如，在 2 秒内，有多少由某个制造商制造的车通过了收费亭？

**输入**：

| 请确 | 时间 |
| --- | --- |
| Honda | 2015-01-01T00:00:01.0000000Z |
| Honda | 2015-01-01T00:00:02.0000000Z |
| Toyota | 2015-01-01T00:00:01.0000000Z |
| Toyota | 2015-01-01T00:00:02.0000000Z |
| Toyota | 2015-01-01T00:00:03.0000000Z |

**输出：**

| 计数 | 时间 |
| --- | --- |
| 2 | 2015-01-01T00:00:02.000Z |
| 1 | 2015-01-01T00:00:04.000Z |

**解决方案：**

	WITH Makes AS (
	    SELECT
	        Make,
	        COUNT(*) AS CountMake
	    FROM
	        Input TIMESTAMP BY Time
	    GROUP BY
	          Make,
	          TumblingWindow(second, 2)
	)
	SELECT
	    COUNT(*) AS Count,
	    System.TimeStamp AS Time
	FROM
	    Makes
	GROUP BY
	    TumblingWindow(second, 1)


**说明：**我们会进行一个初始的聚合来获得该时段内通过的车辆所属的不同制造商以及车辆的计数。然后，我们会进行一个聚合来计算有多少个制造商 – 在提供了某个时段内所有唯一值的情况下，我们可以获得相同的时间戳，然后，第二个聚合窗口需要尽可能小，不能聚合第一步中的 2 个窗口。

## 确定某个值是否已更改 ##
**说明**：查看以前的值，确定其是否不同于当前值。例如，收费公路上前一辆车的制造商是否与目前这辆车的制造商相同？

**输入**：

| 请确 | 时间 |
| --- | --- |
| Honda | 2015-01-01T00:00:01.0000000Z |
| Toyota | 2015-01-01T00:00:02.0000000Z |

**输出**：

| 请确 | 时间 |
| --- | --- |
| Toyota | 2015-01-01T00:00:02.0000000Z |

**解决方案**：

	SELECT
		Make,
		Time
	FROM
		Input TIMESTAMP BY Time
	WHERE
		LAG(Make, 1) OVER (LIMIT DURATION(minute, 1)) <> Make

**说明**：使用 LAG 来查看后退一个事件之后的输入流，然后即可获得“制造商”字段的值。然后，将其与当前事件的“制造商”字段进行比较，如果二者不同，则输出该事件。

## 在窗口中查找第一个事件 ##
**说明**：查找每个 10 分钟时间间隔内的第一辆车？

**输入**：

| 牌照 | 请确 | 时间 |
| --- | --- | --- |
| DXE 5291 | Honda | 2015-07-27T07:00:00:05.0000000Z |
| YZK 5704 | Ford | 2015-07-27T07:00:02:17.0000000Z |
| RMV 8282 | Honda | 2015-07-27T07:00:05:01.0000000Z |
| YHN 6970 | Toyota | 2015-07-27T07:00:06:00.0000000Z |
| VFE 1616 | Toyota | 2015-07-27T07:00:09:31.0000000Z |
| QYF 9358 | Honda | 2015-07-27T07:00:12:02.0000000Z |
| MDR 6128 | BMW | 2015-07-27T07:00:13:45.0000000Z |

**输出**：

| 牌照 | 请确 | 时间 |
| --- | --- | --- |
| DXE 5291 | Honda | 2015-07-27T07:00:05.000Z |
| QYF 9358 | Honda | 2015-07-27T07:12:02.000Z |

**解决方案**：

	SELECT 
		LicensePlate,
		Make,
		Time
	FROM 
		Input TIMESTAMP BY Time
	WHERE 
		IsFirst(minute, 10) = 1

现在，我们来变一下这个问题，查找每 10 分钟时间间隔内特定制造商的第一辆车。

| 牌照 | 请确 | 时间 |
| --- | --- | --- |
| DXE 5291 | Honda | 2015-07-27T07:00:05.000Z |
| YZK 5704 | Ford | 2015-07-27T07:02:17.000Z |
| YHN 6970 | Toyota | 2015-07-27T07:06:00.000Z |
| QYF 9358 | Honda | 2015-07-27T07:12:02.000Z |
| MDR 6128 | BMW | 2015-07-27T07:13:45.000Z |

**解决方案**：

	SELECT 
		LicensePlate,
		Make,
		Time
	FROM 
		Input TIMESTAMP BY Time
	WHERE 
		IsFirst(minute, 10) OVER (PARTITION BY Make) = 1

## 在窗口中查找最后一个事件 ##
**说明**：在每 10 分钟时间间隔内查找最后一辆车。

**输入**：

| 牌照 | 请确 | 时间 |
| --- | --- | --- |
| DXE 5291 | Honda | 2015-07-27T07:00:00:05.0000000Z |
| YZK 5704 | Ford | 2015-07-27T07:00:02:17.0000000Z |
| RMV 8282 | Honda | 2015-07-27T07:00:05:01.0000000Z |
| YHN 6970 | Toyota | 2015-07-27T07:00:06:00.0000000Z |
| VFE 1616 | Toyota | 2015-07-27T07:00:09:31.0000000Z |
| QYF 9358 | Honda | 2015-07-27T07:00:12:02.0000000Z |
| MDR 6128 | BMW | 2015-07-27T07:00:13:45.0000000Z |

**输出**：

| 牌照 | 请确 | 时间 |
| --- | --- | --- |
| VFE 1616 | Toyota | 2015-07-27T07:09:31.000Z |
| MDR 6128 | BMW | 2015-07-27T07:13:45.000Z |

**解决方案**：

	WITH LastInWindow AS
	(
		SELECT 
			MAX(Time) AS LastEventTime
		FROM 
			Input TIMESTAMP BY Time
		GROUP BY 
			TumblingWindow(minute, 10)
	)
	SELECT 
		Input.LicensePlate,
		Input.Make,
		Input.Time
	FROM
		Input TIMESTAMP BY Time 
		INNER JOIN LastInWindow
		ON DATEDIFF(minute, Input, LastInWindow) BETWEEN 0 AND 10
		AND Input.Time = LastInWindow.LastEventTime

**说明**：此查询中有两个步骤 – 第一个步骤是查找 10 分钟时间窗口内最后一个时间戳。第二个步骤需要将第一个查询的结果与初始流联接起来，查找每个窗口中与最后一个时间戳匹配的事件。

## 检测缺少的事件 ##
**说明**：查看某个流是否没有值符合特定的标准。例如，在 90 秒内，是否有 2 辆一起进入收费公路的车来自同一制造商？

**输入**：

| 请确 | 牌照 | 时间 |
| --- | --- | --- |
| Honda | ABC-123 | 2015-01-01T00:00:01.0000000Z |
| Honda | AAA-999 | 2015-01-01T00:00:02.0000000Z |
| Toyota | DEF-987 | 2015-01-01T00:00:03.0000000Z |
| Honda | GHI-345 | 2015-01-01T00:00:04.0000000Z |

**输出**：

| 请确 | 时间 | CurrentCarLicensePlate | FirstCarLicensePlate | FirstCarTime |
| --- | --- | --- | --- | --- |
| Honda | 2015-01-01T00:00:02.0000000Z | AAA-999 | ABC-123 | 2015-01-01T00:00:01.0000000Z |

**解决方案**：

	SELECT
	    Make,
	    Time,
	    LicensePlate AS CurrentCarLicensePlate,
	    LAG(LicensePlate, 1) OVER (LIMIT DURATION(second, 90)) AS FirstCarLicensePlate,
	    LAG(Time, 1) OVER (LIMIT DURATION(second, 90)) AS FirstCarTime
	FROM
	    Input TIMESTAMP BY Time
	WHERE
	    LAG(Make, 1) OVER (LIMIT DURATION(second, 90)) = Make

**说明**：使用 LAG 来查看后退一个事件之后的输入流，然后即可获得“制造商”字段的值。然后，将其与当前事件的“制造商”字段进行比较，如果二者相同，则输出该事件，并使用 LAG 来获取有关前一辆车的数据。

## 检测某个条件的持续时间 ##
**说明**：查看某个条件的持续时间。例如，假设某个 Bug 导致所有车的重量不正确（超出 20,000 磅），于是我们需要计算该 Bug 的持续时间。

**输入**：

| 请确 | 时间 | 重量 |
| --- | --- | --- |
| Honda | 2015-01-01T00:00:01.0000000Z | 2000 |
| Toyota | 2015-01-01T00:00:02.0000000Z | 25000 |
| Honda | 2015-01-01T00:00:03.0000000Z | 26000 |
| Toyota | 2015-01-01T00:00:04.0000000Z | 25000 |
| Honda | 2015-01-01T00:00:05.0000000Z | 26000 |
| Toyota | 2015-01-01T00:00:06.0000000Z | 25000 |
| Honda | 2015-01-01T00:00:07.0000000Z | 26000 |
| Toyota | 2015-01-01T00:00:08.0000000Z | 2000 |

**输出**：

| StartFault | EndFault | FaultDurationSeconds |
| --- | --- | --- |
| 2015-01-01T00:00:01.0000000Z | 2015-01-01T00:00:08.0000000Z | 7 |
| 2015-01-01T00:00:01.0000000Z | 2015-01-01T00:00:08.0000000Z | 7 |
| 2015-01-01T00:00:01.0000000Z | 2015-01-01T00:00:08.0000000Z | 7 |
| 2015-01-01T00:00:01.0000000Z | 2015-01-01T00:00:08.0000000Z | 7 |
| 2015-01-01T00:00:01.0000000Z | 2015-01-01T00:00:08.0000000Z | 7 |
| 2015-01-01T00:00:01.0000000Z | 2015-01-01T00:00:08.0000000Z | 7 |

**解决方案**：

	SELECT
	    PrevGood.Time AS StartFault,
	    ThisGood.Time AS Endfault,
	    DATEDIFF(second, PrevGood.Time, ThisGood.Time) AS FaultDuraitonSeconds
	FROM
	    Input AS ThisGood TIMESTAMP BY Time
	    INNER JOIN Input AS PrevGood TIMESTAMP BY Time
	    ON DATEDIFF(second, PrevGood, ThisGood) BETWEEN 1 AND 3600
	    AND PrevGood.Weight < 20000
	    INNER JOIN Input AS Bad TIMESTAMP BY Time
	    ON DATEDIFF(second, PrevGood, Bad) BETWEEN 1 AND 3600
	    AND DATEDIFF(second, Bad, ThisGood) BETWEEN 1 AND 3600
	    AND Bad.Weight >= 20000
	    LEFT JOIN Input AS MidGood TIMESTAMP BY Time
	    ON DATEDIFF(second, PrevGood, MidGood) BETWEEN 1 AND 3600
	    AND DATEDIFF(second, MidGood, ThisGood) BETWEEN 1 AND 3600
	    AND MidGood.Weight < 20000
	WHERE
	    ThisGood.Weight < 20000
	    AND MidGood.Weight IS NULL

**说明**：我们寻找的是 2 个正常事件中夹杂 1 个错误事件且这之中没有正常事件的情况。换句话说，这 2 个事件是位于至少 1 个错误事件之前以及之后的首批事件。获取 2 个在其中夹杂有 1 个错误事件的正常事件很简单，只需使用 2 个 JOIN 操作，然后通过检查重量并比较时间戳来验证我们获得的事件的状态是否为正常 -> 错误 -> 正常即可。

在学习“通过 LEFT 外部联接来包括 NULL 或检查事件是否缺失”后，我们知道了如何检查在选取的 2 个正常事件之间是否没有正常事件的发生。

将这些组合在一起，我们就可以得到“正常 -> 错误 -> 正常”这样的状态，二者之间没有其他正常事件。现在，我们可以计算开头的正常事件与末尾的正常事件之间的持续时间，从而得出 Bug 的持续时间。

## 获取帮助
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=AzureStreamAnalytics)

## 后续步骤

- [Azure 流分析简介](/documentation/articles/stream-analytics-introduction)
- [Azure 流分析入门](/documentation/articles/stream-analytics-get-started)
- [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs)
- [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
- [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)
 

<!---HONumber=69-->