<properties
    pageTitle="Azure 分析的公共数据集 | Azure"
    description="了解可用于设计 Azure 分析服务和解决方案原型并进行测试的公共数据集。"
    services="sql-database"
    documentationcenter=""
    author="douglaslMS"
    manager="jhubbard"
    editor=""
    tags="" />
<tags
    ms.custom=""
    ms.assetid=""
    ms.service="sql-database"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload=""
    ms.date="03/20/2017"
    wacn.date="05/22/2017"
    ms.author="douglasl"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="2f465881625b2d5abc74b393ce1c1fe9f0d08d46"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/12/2017" />

# <a name="public-data-sets-for-testing-and-prototyping"></a>用于测试和原型设计的公共数据集

浏览公共数据集的这个列表，其其中是否存在可用于设计存储和分析服务及解决方案的原型并进行测试的数据。

## <a name="us-government-and-agency-data"></a>美国政府和机构数据

| 数据源 | 关于数据 | 关于文件 |
|---|---|---|
| [美国政府数据](https://www.census.gov/data.html) | 超过 190,000 个数据集，涵盖了美国的农业、气候、消费者、生态系统、教育、能源、金融、保健、地方政府、制造业、海运、海洋、公共安全和科研方面的数据。 | 各种大小的文件，采用 HTML、XML、CSV、JSON、Excel 等格式。 可按文件格式筛选可用数据集。 |
| [美国人口普查数据](https://www.census.gov/data.html) | 美国人口的统计数据 | 数据集采用各种格式。 |
| [来自 NASA 的地球科学数据](https://earthdata.nasa.gov/) | 32,000 多个数据集，涵盖了农业、大气、生物圈、气候、低温层、人文领域、水圈、地表、海洋、太阳与地球相互作用等方面的数据。 | 数据集采用各种格式。 |
| [航班延迟和其他交通数据](https://www.transtats.bts.gov/OT_Delay/OT_DelayCause1.asp) | “美国运输部 (DOT) 运输统计局 (BTS) 对大型航空公司运营的国内航班的准时情况进行了跟踪。 可在此网站发布的汇总表中了解准时的、延迟的和取消的航班及转机航班数的汇总信息。” | 文件为 CSV 格式。 |
| [交通死亡事故 - 美国事故分析报告系统 (FARS)](http://www.nhtsa.gov/FARS) | “FARS 是全国性的普查，可提供 NHTSA、国会和美国公众就机动车辆交通事故造成的致命事故公开的年度数据。” | “使用 FARS 查询系统自己创建在线运行的死亡数据。 或从 FTP 站点下载自 1975 起的所有 FARS 数据。” |
| [有毒化学物质数据 - EPA 毒性预测 (ToxCast™) 数据](https://www.epa.gov/chemical-research/toxicity-forecaster-toxcasttm-data) | “EPA 可公开提供最近更新的数千种化学品的高通量毒性数据。 该数据由 EPA 的 ToxCast 研究得出。” | 存在各种格式的数据集，包括电子表格、R 包和 MySQL 数据库文件。 |
| [有毒化学物质数据 - NIH Tox21 数据挑战 2014](https://tripod.nih.gov/tox21/challenge/) | “2014 Tox21 数据挑战旨在帮助科学家了解通过 21 世纪毒理学进行测试的化学物质和化合物的潜力，以可能造成毒性反应的方法主动打破生物学路径。” | 数据集格式为 SMILES 和 SDF。 该数据可提供“Tox21 收集的约 10,000 种化合物 (Tox21 10K) 的测定活性数据和化学结构。” |
| [NCBI 提供的生物技术和基因组数据](https://www.ncbi.nlm.nih.gov/guide/data-software/) | 多个数据集，涵盖了基因、基因组和蛋白质的数据。 | 数据集为文本、XML、BLAST 等格式。 可使用 BLAST 应用。 |

## <a name="other-statistical-and-scientific-data"></a>其他统计和科学类数据

| 数据源 | 关于数据 | 关于文件 |
|---|---|---|
| [纽约市出租车数据](http://www.nyc.gov/html/tlc/html/about/trip_record_data.shtml) | “出租车行程记录，其中包括捕获出租车接客和下客的日期/时间、地点、行程距离、各项费用、费率类型、付款方式和驾驶员报告的乘客数量的字段。” | 数据集文件以 CSV 格式按月提供。 |
| [Microsoft Research 数据集 -“研究型数据科学”](https://www.microsoft.com/research/academic-program/data-science-microsoft-research/) | 多个数据集，涵盖了人机交互、音频/视频、数据挖掘/信息检索、地理空间/位置、自然语言处理和机器人/计算机视觉。 | 数据集有各种格式，可压缩后下载。 |
| [公共基因组数据](http://www.completegenomics.com/public-data/) | “公众可使用全人类基因组的不同数据集，加强基因组学研究……”提供商 Complete Genomics 是一家营利性私有公司。 | 经提取的数据集为 UNIX 文本格式。 还可使用分析工具。 |
| [Open Science Data Cloud 数据](https://www.opensciencedatacloud.org/) | “Open Science Data Cloud 为科学界提供了可存储、共享和分析 TB 级和 PB 级科学数据集的资源。”| 数据集采用各种格式。 |
| [全球气候数据 - WorldcLIM](http://worldclim.org/) | “WorldClim 是一组全球气候层（网格气候数据），空间分辨率约为 1 平方千米。 这些数据可用于映射和空间建模。” | 这些文件包含地理空间数据。 有关详细信息，请参阅[数据格式](http://worldclim.org/formats1)。 |
| [关于人类社会的数据 - GDELT 项目](http://www.gdeltproject.org/data.html) | “GDELT 项目是目前为止创建的有关人类社会的最大、最全面、分辨率最高的开放数据库。” | 原始数据文件为 CSV 格式。 |
| [来自 Criteo 的机器学习广告点击预测数据](http://labs.criteo.com/2013/12/download-terabyte-click-logs/) | “公开发布的最大的 ML 数据集。” 有关详细信息，请参阅 [Criteo's 1 TB Click Prediction Dataset](https://blogs.technet.microsoft.com/machinelearning/2015/04/01/now-available-on-azure-ml-criteos-1tb-click-prediction-dataset/)（Criteo 的 1 TB 点击预测数据集）。 | |
| [来自 The Lemur Project 的 ClueWeb09 文本挖掘数据集](http://www.lemurproject.org/clueweb09.php/) | “创建 ClueWeb09 数据集是为了支持与信息检索和相关人类语言技术相关的研究。 由 2009 年 1 月和 2 月收集的约 10 亿个 10 种语言的网页组成。” | 请参阅 [Dataset Information](http://www.lemurproject.org/clueweb09/datasetInformation.php)（数据集信息）。|

## <a name="online-service-data"></a>联机服务数据

| 数据源 | 关于数据 | 关于文件 |
|---|---|---|
| [GitHub Archive](https://www.githubarchive.org/) | “GitHub Archive 是一个用于记录事件公共 GitHub 时间轴，将其存档，并使其易于进行进一步分析的项目。” | 从 Web 客户端下载 .gz (Gzip) 格式的以 JSON 编码的事件存档。 |
| [来自 GHTorrent 项目的 GitHub 活动数据](http://ghtorrent.org/) | “GHTorrent 项目正努力创建一种通过 GitHub REST API 提供的、可缩放的、可查询的脱机镜像数据。 GHTorrent 可监视 GitHub 公共事件时间轴。 它会彻底检索每个事件的内容及其依赖项。” | MySQL 数据库转储采用 CSV 格式。 |
| [Stack Overflow 数据转储](https://archive.org/details/stackexchange) | “这是指将用户贡献的所有内容匿名转储在 Stack Exchange 网络（包括 Stack Overflow）上。” | “每个站点（例如 Stack Overflow）都被格式化为一个单独的存档，其中包含通过 7-zip 使用 bzip2 压缩的 XML 文件。 每个站点存档都包括帖子、用户、投票、评论、发布历史和发布链接。” |