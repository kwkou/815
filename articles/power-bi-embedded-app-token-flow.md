<properties
   pageTitle="关于 Power BI Embedded 中的应用令牌流"
   description="Power BI Embedded 中用于身份验证和授权的应用令牌"
   services="power-bi-embedded"
   documentationCenter=""
   authors="dvana"
   manager="NA"
   editor=""
   tags=""/>
<tags
   ms.service="power-bi-embedded"
   ms.date="03/29/2016"
   wacn.date="05/30/2016"/>

# 关于 Power BI Embedded 中的应用令牌流

**Power BI Embedded** 服务使用“应用令牌”进行身份验证和授权，而非显式最终用户身份验证。在“应用令牌”模型中，应用程序管理最终用户的身份验证和授权。如有必要，你的应用将创建并发送**应用令牌**，告诉我们的服务渲染请求的报表。尽管可以这样做，但这种设计其实并不要求应用使用 **Azure Active Directory** 进行用户身份验证和授权。

**以下是应用令牌密钥流的工作原理**

1. 将 API 密钥复制到你的应用程序中。可以在“Azure 门户”中获取该密钥。

    ![](./media/powerbi-embedded-get-started-sample/azure-portal.png)

2. 令牌将发布声明，并且有过期时间。

    ![](./media/powerbi-embedded-get-started-sample/power-bi-embedded-token-2.png)

3. 令牌使用 API 访问密钥获取签名。

    ![](./media/powerbi-embedded-get-started-sample/power-bi-embedded-token-3.png)

4. 用户请求查看报表。

    ![](./media/powerbi-embedded-get-started-sample/power-bi-embedded-token-4.png)

5.	使用 API 访问密钥验证令牌。

    ![](./media/powerbi-embedded-get-started-sample/power-bi-embedded-token-5.png)

6.	Power BI Embedded 向用户发送报表。

    ![](./media/powerbi-embedded-get-started-sample/power-bi-embedded-token-6.png)

**Power BI Embedded** 向用户发送报表后，用户可以在自定义应用中查看报表。例如，如果导入了 [Analyzing Sales Data PBIX sample（分析销售数据 PBIX 示例）](http://download.microsoft.com/download/1/4/E/14EDED28-6C58-4055-A65C-23B4DA81C4DE/Analyzing_Sales_Data.pbix)，该示例 Web 应用将如下所示：

![](./media/powerbi-embedded-get-started-sample/sample-web-app.png)

## 另请参阅
- [Microsoft Power BI Embedded 示例入门](/documentation/articles/power-bi-embedded-get-started-sample)
- [Microsoft Power BI Embedded 是什么](/documentation/articles/power-bi-embedded-what-is-power-bi-embedded)
- [常见 Microsoft Power BI Embedded 预览版方案](/documentation/articles/power-bi-embedded-scenarios)
- [Microsoft Power BI Embedded 预览版入门](/documentation/articles/power-bi-embedded-get-started)

<!---HONumber=Mooncake_0523_2016-->