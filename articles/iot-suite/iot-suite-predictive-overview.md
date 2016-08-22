<properties
 pageTitle="预见性维护预配置解决方案 | Azure"
 description="介绍 Azure IoT 预见性维护预配置解决方案。"
 services=""
 suite="iot-suite"
 documentationCenter=""
 authors="stevehob"
 manager="timlt"
 editor=""/>  


<tags
 ms.service="iot-suite"
 ms.date="05/24/2016"
 wacn.date="08/22/2016"/>  


# 预见性维护预配置解决方案概述

预见性维护预配置解决方案是 [Azure IoT 套件][lnk_iot_suite]随附的[预配置解决方案][lnk_preconfigured_solutions]之一。此解决方案将实时设备遥测收集与使用 [Azure 机器学习][lnk_machine_learning]创建的预测模型相集成。


有了 Azure IoT 套件，企业可以又快又方便地连接和监视资产，并实时分析数据。预见性维护预配置解决方案会利用该数据及丰富的仪表板与可视化效果，为企业提供新的信息，以提升其效率及增加收入来源。

## 应用场景

Fabrikam 是一家区域性航空公司，致力于以优惠的价格为客户提供优质的体验。维护问题是造成航班延误的原因之一，而飞机引擎维护又是其中最为棘手的项目。因为必须严防飞行期间发生引擎故障，所以 Fabrikam 不仅会定期检查其引擎，而且会恪守所安排的维护计划。但因为飞机引擎的问题不一，所以有一些引擎维护工作并非必要。更重要的是，若在执行维护工作之前发生问题，可能会造成飞机停飞。此种延误会造成严重的损失，而飞机所在地点若正好缺少适当的技术人员或备用零件，将更为严重。

Fabrikam 飞机的引擎由各种传感器进行检测，这些传感器会监视飞行期间的引擎状况。Fabrikam 使用 Azure IoT 套件收集飞行期间所收集的传感器数据。经过多年累积引擎运行数据和故障数据之后，Fabrikam 的数据科学家制作出了一个模型，可以预测飞机引擎的剩余使用寿命 (RUL)。他们从四个引擎传感器的数据中，找出了数据之间的相关性，而其中一个引擎更潜藏了最终会导致引擎故障的问题。Fabrikam 现在除了继续执行定期检查来确保安全之外，还会在每次飞行后，将飞行期间从引擎收集而来的遥测数据应用于这些模型，以计算每个引擎的 RUL。Fabrikam 现在可以预测未来的故障点，并预先规划维护和维修工作。

> [AZURE.NOTE] 该解决方案模型使用实际的引擎损耗数据。

通过预测必要维护时间点，Fabrikam 可以优化各项作业，进而降低成本。维护专员与排班专员一起合作，根据飞机在特定地点停机的时间，规划维护的时间，以确保飞机能有足够的停飞时间，不会造成排班中断。Fabrikam 可以据此安排技术人员，让飞机无需浪费时间等待，就可获得有效率的维修。库存控制管理员会收到维护计划，因此可以优化其订单流程和备用零件库存。这一切不仅让 Fabrikam 可以将飞机停飞的时间降至最低，还可以降低运营成本，同时确保了乘客与乘务员的安全。

若要了解 [Azure IoT 套件][lnk_iot_suite]如何提供这些功能，客户需要先了解预见性维护，请查看此[信息图][lnk_infographic]。

## 如何生成预见性维护解决方案

该解决方案利用可作为模板的现有 Azure 机器学习模型，演示这些功能如何运用通过 IoT 套件服务收集而来的设备遥测数据。Microsoft 构建了一个飞机引擎的[回归模型][lnk_regression_model]，并发布了完整的模板、数据<sup>[1]</sup>以及有关如何使用该模型的分步指南。

Azure IoT 预见性维护预配置解决方案使用通过此模板创建的回归模型；此模型会部署到你的 Azure 订阅中，并通过自动生成的 API 加以公开。此解决方案包含代表 4 个（共 100 个）引擎和 4 个（共 21 个）传感器数据流的测试数据的子集，可通过定型模型提供精确的结果。

[1] A. Saxena and K. Goebel (2008)."Turbofan Engine Degradation Simulation Data Set", NASA Ames Prognostics Data Repository (http://ti.arc.nasa.gov/tech/dash/pcoe/prognostic-data-repository/), NASA Ames Research Center, Moffett Field, CA

## 后续步骤

若要了解有关 Azure IoT 如何实现预见性维护方案的详细信息，请阅读 [Capture value from the Internet of Things][lnk_capture_value]（捕获物联网的价值）。

[演练][lnk-predictive-walkthrough]预见性维护预配置解决方案。

[lnk-predictive-walkthrough]: /documentation/articles/iot-suite/iot-suite-predictive-walkthrough/
[lnk_preconfigured_solutions]: /documentation/articles/iot-suite/iot-suite-what-are-preconfigured-solutions/
[lnk_iot_suite]: /documentation/articles/iot-suite/iot-suite-overview/
[lnk_machine_learning]: /home/features/machine-learning/
[lnk_infographic]: https://www.microsoft.com/server-cloud/predictivemaintenance/Index.html
[lnk_regression_model]: http://gallery.cortanaanalytics.com/Collection/Predictive-Maintenance-Template-3
[lnk_capture_value]: http://download.microsoft.com/download/0/7/D/07D394CE-185D-4B96-AC3C-9B61179F7080/Capture_value_from_the_Internet%20of%20Things_with_Predictive_Maintenance.PDF

<!---HONumber=Mooncake_0815_2016-->