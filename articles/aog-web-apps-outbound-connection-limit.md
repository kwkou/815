<properties
                pageTitle="Azure Web 应用对外连接数上限分析"
                description="Azure Web 应用对外连接数存在上限，超出后会发生异常"
                services="app-service-web"
                documentationCenter=""
                authors=""
                manager=""
                editor=""
                tags="Web Apps,outbound connection"/>

<tags
                ms.service="app-service-web-aog"
                ms.date="12/23/2016"
                wacn.date="12/23/2016"/>

# Azure Web 应用对外连接数上限分析

在 Azure Web 应用中发起大量外部连接操作时，需要考虑已经建立了多少外部连接。当超过最大对外连接数时，Azure Web 应用将会产生套接字异常。Azure Web 应用对于各个级别的实例，对外连接数是固定的，如下所示：

| Limit name  |                Description             | Small(A1) | Medium(A2) | Large(A3) |
| ----------- | -------------------------------------- | --------- | ---------- | --------- |
| Connections | Number of connections across entire VM |    1920   |    3968    |    8064   |

>[AZURE.NOTE] 
>该限制是对于单个实例而言的。如果您一个实例上部署了多个应用和 web 作业，那么总连接数是这些应用和 web 作业所发连接之和。如果是 web 应用，是以定价层为计量单位的，如果一个定价层中包括多个 Web 应用，则这些 Web 应用及 Web 应用上所部署的 web 作业连接数之和为总连接数。

参考资料：[Azure Web 应用运行环境限制](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox)。