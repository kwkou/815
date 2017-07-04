<properties
    pageTitle="Power BI Embedded 故障排除"
    description="Power BI Embedded 故障排除"
    services="power-bi-embedded"
    documentationcenter=""
    author="guyinacube"
    manager="erikre"
    editor=""
    tags="" />
<tags
    ms.assetid="c8aee652-ed8b-4c66-9c63-f798b7a655b4"
    ms.service="power-bi-embedded"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="powerbi"
    ms.date="01/06/2017"
    wacn.date="02/22/2017"
    ms.author="asaxton" />  


# Power BI Embedded 故障排除
本文提供了有关如何解决 **Power BI Embedded** 问题的信息。



## 设置 SQL Server 连接字符串 <a name="connection-string"/>
若要设置 SQL Server 连接字符串，请遵循以下特定格式。下面列举了一个 SQL Server 连接字符串的示例。


	"Persist Security Info=False;Integrated Security=true;Initial Catalog=Northwind;server=(local)"


若要了解有关 SQL Server 连接字符串的详细信息，请参阅以下文章：

- [SQL Server Connection Strings（SQL Server 连接字符串）](https://msdn.microsoft.com/zh-cn/library/jj653752.aspx)
- [SqlConnection.ConnectionString](https://msdn.microsoft.com/zh-cn/library/system.data.sqlclient.sqlconnection.connectionstring.aspx)


## 设置凭据 <a name="credentials"/>
如果你具有用于开发或过渡环境的凭据（例如用户名和密码），建议更新此凭据，以匹配生产解决方案。

## 另请参阅
- [示例入门](/documentation/articles/power-bi-embedded-get-started-sample/)
- [什么是 Power BI Embedded](/documentation/articles/power-bi-embedded-what-is-power-bi-embedded/)

<!---HONumber=Mooncake_0213_2017-->
<!---Update_Description: wording update -->