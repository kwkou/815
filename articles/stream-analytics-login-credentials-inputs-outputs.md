<properties 
	pageTitle="流分析：轮转输入和输出的登录凭据 | Azure" 
	description="了解如何更新流分析输入和输出的凭据。"
	keywords="登录凭据"
	services="stream-analytics" 
	documentationCenter="" 
	authors="jeffstokes72" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="stream-analytics" 
	ms.date="05/03/2016" 
	wacn.date="06/20/2016"/>
	

#在流分析作业中轮转输入和输出的登录凭据

##摘要
Azure 流分析目前不允许在作业运行时替换输入/输出上的凭据。

虽然 Azure 流分析支持从上一次输出恢复作业，但我们仍希望分享整个操作过程，尽量缩短从停止作业到开始作业这段时间的延迟，并轮转登录凭据。

##第 1 部分 - 准备一组新的凭据：
此部分适用于以下输入/输出：

* Blob 存储
* 事件中心
* SQL 数据库
* 表存储

对于其他输入/输出，请执行第 2 部分。

###Blog 存储/表存储
1.  在 Azure 管理门户上，转到“存储”扩展：  

    ![graphic1][graphic1]

2.  找到你的作业使用的存储并进入它：  
    
    ![graphic2][graphic2]
    
3.  单击“管理访问密钥”命令：  
    
    ![graphic3][graphic3]

4.  从主访问密钥和辅助访问密钥之中**挑选你的作业不使用的那个密钥**。
5.  单击“重新生成”：  

    ![graphic4][graphic4]

6.  复制新生成的密钥：  
    
    ![graphic5][graphic5]

7.  继续完成第 2 部分。

###事件中心
1.  转到 Azure 管理门户的 Service Bus 扩展：  
    
    ![graphic6][graphic6]

2.  找到你的作业使用的 Service Bus 命名空间，然后进入该空间：  
    
    ![graphic7][graphic7]

3.  如果你的作业在 Service Bus 命名空间上使用共享访问策略，请跳到步骤 6  
4.  转到“事件中心”选项卡：  
    
    ![graphic8][graphic8]

5.  找到你的作业使用的事件中心并进入它：  
    
    ![graphic9][graphic9]

6.  转到“配置”选项卡：  
    
    ![graphic10][graphic10]

7.  在“策略名称”下拉列表中，找到你的作业所使用的共享访问策略：  

    ![graphic11][graphic11]

8.  从主密钥和辅助密钥之中**挑选你的作业不使用的那个密钥**。  
9.  单击“重新生成”：  
    
    ![graphic12][graphic12]

10. 复制新生成的密钥：  

    ![graphic13][graphic13]

11. 继续完成第 2 部分。  

###SQL 数据库

>[AZURE.NOTE] 注意：你将需要连接到 SQL 数据库服务。我们将根据 Azure 管理门户上的管理经验来演示如何这样做，不过，你也可以选择使用某些客户端工具，例如 SQL Server Management Studio。

1.  转到 Azure 管理门户的 SQL 数据库扩展：  
    
    ![graphic14][graphic14]

2.  找到你的作业所使用的 SQL 数据库，然后**单击服务器**链接（位于同一行）：  
    
    ![graphic15][graphic15]

3.  单击“管理”命令：  
    
    ![graphic16][graphic16]

4.  键入主数据库：  
    
    ![graphic17][graphic17]

5.  键入你的用户名、密码并单击“登录”：  

    ![graphic18][graphic18]

6.  单击“新建查询”：  
    
    ![graphic19][graphic19]

7.  键入以下查询，将 <login_name> 替换为你的用户名，将 <enterStrongPasswordHere> 替换为你的新密码：  
`CREATE LOGIN <login_name> WITH PASSWORD = '<enterStrongPasswordHere>'`
8.  单击“运行”：  
    
    ![graphic20][graphic20]

9.  回到步骤 2，此时请单击数据库：  
    
    ![graphic21][graphic21]

10. 单击“管理”命令：  
    
    ![graphic22][graphic22]

11. 键入你的用户名、密码并单击“登录”：  

    ![graphic23][graphic23]

12. 单击“新建查询”：  
    
    ![graphic24][graphic24]

13. 键入以下查询，将 <user_name> 替换为用于在该数据库的上下文中标识此登录名的名称（例如，你提供的值可以与提供给 <login_name> 的值相同），并将 <login_name> 替换为新的用户名：  
`CREATE USER <user_name> FROM LOGIN <login_name>`
14. 单击“运行”：  
    
    ![graphic25][graphic25]

15. 现在，你应该为新用户提供角色和权限，这些角色和权限是你原来的用户所有的。
16. 继续完成第 2 部分。

##第 2 部分：停止流分析作业
1.  转到 Azure 管理门户的流分析扩展：  
    
    ![graphic26][graphic26]

2.  找到你的作业并进入它：  
    
    ![graphic27][graphic27]

3.  转到“输入”选项卡或“输出”选项卡，具体取决于你是在输入上轮转凭据还是在输出上轮转凭据。  
    
    ![graphic28][graphic28]

4.  单击“停止”命令，确认作业已停止：  
    
    ![graphic29][graphic29]

等待作业停止。
5.  找到你要在其上轮转凭据的输入/输出，然后进入它：  
    
    ![graphic30][graphic30]

6.  转到第 3 部分。

##第 3 部分：编辑流分析作业的凭据

###Blob 存储/表存储
1.	找到“存储帐户密钥”字段，将新生成的密钥粘贴到其中：  
    
    ![graphic31][graphic31]

2.	单击“保存”命令，然后确认你将保存所做的更改：  
    
    ![graphic32][graphic32]

3.	当你保存所做的更改时，连接测试将自动启动，请确保连接测试成功通过。
4.	转到第 4 部分。

###事件中心
1.	找到“事件中心策略密钥”字段，将新生成的密钥粘贴到其中：  
    
    ![graphic33][graphic33]

2.	单击“保存”命令，然后确认你将保存所做的更改：  
    
    ![graphic34][graphic34]

3.	当你保存所做的更改时，连接测试将自动启动，请确保连接测试成功通过。
4.	转到第 4 部分。


###SQL 数据库
1.	找到“用户名”和“密码”字段，然后将新创建的一组凭据粘贴到其中：  
    
    ![graphic38][graphic38]

2.	单击“保存”命令，然后确认你将保存所做的更改：  
    
    ![graphic39][graphic39]

3.	当你保存所做的更改时，连接测试将自动启动，请确保连接测试成功通过。  
4.	转到第 4 部分。

##第 4 部分：启动上次停止时的作业
1.	通过导航离开“输入/输出”：  
    
    ![graphic40][graphic40]

2.	单击“开始”命令：  
    
    ![graphic41][graphic41]

3.	选取“上次停止时间”，然后单击“确定”：  
    
    ![graphic42][graphic42]

4.	转到第 5 部分。  

##第 5 部分：删除旧的凭据组
此部分适用于以下输入/输出：
* Blob 存储 
* 事件中心
* SQL 数据库
* 表存储

###Blob 存储/表存储
重复第 1 部分以获取你的作业以前使用过的访问密钥，以便续订现在不使用的访问密钥。

###事件中心
重复第 1 部分以获取你的作业以前使用过的密钥，以便续订现在不使用的密钥。

###SQL 数据库
1.	回到第 1 部分步骤 7 中的查询窗口，键入以下查询，将 <previous_login_name> 替换为你的作业以前使用过的用户名：  
`DROP LOGIN <previous_login_name>`  
2.	单击“运行”：  
    
    ![graphic43][graphic43]  

你会获得以下确认：

	Command(s) completed successfully.

## 获取帮助
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=AzureStreamAnalytics)

## 后续步骤

- [Azure 流分析简介](/documentation/articles/stream-analytics-introduction/)
- [Azure 流分析入门](/documentation/articles/stream-analytics-get-started/)
- [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs/)
- [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
- [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)


[graphic1]: ./media/stream-analytics-login-credentials-inputs-outputs/1-stream-analytics-login-credentials-inputs-outputs.png
[graphic2]: ./media/stream-analytics-login-credentials-inputs-outputs/2-stream-analytics-login-credentials-inputs-outputs.png
[graphic3]: ./media/stream-analytics-login-credentials-inputs-outputs/3-stream-analytics-login-credentials-inputs-outputs.png
[graphic4]: ./media/stream-analytics-login-credentials-inputs-outputs/4-stream-analytics-login-credentials-inputs-outputs.png
[graphic5]: ./media/stream-analytics-login-credentials-inputs-outputs/5-stream-analytics-login-credentials-inputs-outputs.png
[graphic6]: ./media/stream-analytics-login-credentials-inputs-outputs/6-stream-analytics-login-credentials-inputs-outputs.png
[graphic7]: ./media/stream-analytics-login-credentials-inputs-outputs/7-stream-analytics-login-credentials-inputs-outputs.png
[graphic8]: ./media/stream-analytics-login-credentials-inputs-outputs/8-stream-analytics-login-credentials-inputs-outputs.png
[graphic9]: ./media/stream-analytics-login-credentials-inputs-outputs/9-stream-analytics-login-credentials-inputs-outputs.png
[graphic10]: ./media/stream-analytics-login-credentials-inputs-outputs/10-stream-analytics-login-credentials-inputs-outputs.png
[graphic11]: ./media/stream-analytics-login-credentials-inputs-outputs/11-stream-analytics-login-credentials-inputs-outputs.png
[graphic12]: ./media/stream-analytics-login-credentials-inputs-outputs/12-stream-analytics-login-credentials-inputs-outputs.png
[graphic13]: ./media/stream-analytics-login-credentials-inputs-outputs/13-stream-analytics-login-credentials-inputs-outputs.png
[graphic14]: ./media/stream-analytics-login-credentials-inputs-outputs/14-stream-analytics-login-credentials-inputs-outputs.png
[graphic15]: ./media/stream-analytics-login-credentials-inputs-outputs/15-stream-analytics-login-credentials-inputs-outputs.png
[graphic16]: ./media/stream-analytics-login-credentials-inputs-outputs/16-stream-analytics-login-credentials-inputs-outputs.png
[graphic17]: ./media/stream-analytics-login-credentials-inputs-outputs/17-stream-analytics-login-credentials-inputs-outputs.png
[graphic18]: ./media/stream-analytics-login-credentials-inputs-outputs/18-stream-analytics-login-credentials-inputs-outputs.png
[graphic19]: ./media/stream-analytics-login-credentials-inputs-outputs/19-stream-analytics-login-credentials-inputs-outputs.png
[graphic20]: ./media/stream-analytics-login-credentials-inputs-outputs/20-stream-analytics-login-credentials-inputs-outputs.png
[graphic21]: ./media/stream-analytics-login-credentials-inputs-outputs/21-stream-analytics-login-credentials-inputs-outputs.png
[graphic22]: ./media/stream-analytics-login-credentials-inputs-outputs/22-stream-analytics-login-credentials-inputs-outputs.png
[graphic23]: ./media/stream-analytics-login-credentials-inputs-outputs/23-stream-analytics-login-credentials-inputs-outputs.png
[graphic24]: ./media/stream-analytics-login-credentials-inputs-outputs/24-stream-analytics-login-credentials-inputs-outputs.png
[graphic25]: ./media/stream-analytics-login-credentials-inputs-outputs/25-stream-analytics-login-credentials-inputs-outputs.png
[graphic26]: ./media/stream-analytics-login-credentials-inputs-outputs/26-stream-analytics-login-credentials-inputs-outputs.png
[graphic27]: ./media/stream-analytics-login-credentials-inputs-outputs/27-stream-analytics-login-credentials-inputs-outputs.png
[graphic28]: ./media/stream-analytics-login-credentials-inputs-outputs/28-stream-analytics-login-credentials-inputs-outputs.png
[graphic29]: ./media/stream-analytics-login-credentials-inputs-outputs/29-stream-analytics-login-credentials-inputs-outputs.png
[graphic30]: ./media/stream-analytics-login-credentials-inputs-outputs/30-stream-analytics-login-credentials-inputs-outputs.png
[graphic31]: ./media/stream-analytics-login-credentials-inputs-outputs/31-stream-analytics-login-credentials-inputs-outputs.png
[graphic32]: ./media/stream-analytics-login-credentials-inputs-outputs/32-stream-analytics-login-credentials-inputs-outputs.png
[graphic33]: ./media/stream-analytics-login-credentials-inputs-outputs/33-stream-analytics-login-credentials-inputs-outputs.png
[graphic34]: ./media/stream-analytics-login-credentials-inputs-outputs/34-stream-analytics-login-credentials-inputs-outputs.png
[graphic35]: ./media/stream-analytics-login-credentials-inputs-outputs/35-stream-analytics-login-credentials-inputs-outputs.png
[graphic36]: ./media/stream-analytics-login-credentials-inputs-outputs/36-stream-analytics-login-credentials-inputs-outputs.png
[graphic37]: ./media/stream-analytics-login-credentials-inputs-outputs/37-stream-analytics-login-credentials-inputs-outputs.png
[graphic38]: ./media/stream-analytics-login-credentials-inputs-outputs/38-stream-analytics-login-credentials-inputs-outputs.png
[graphic39]: ./media/stream-analytics-login-credentials-inputs-outputs/39-stream-analytics-login-credentials-inputs-outputs.png
[graphic40]: ./media/stream-analytics-login-credentials-inputs-outputs/40-stream-analytics-login-credentials-inputs-outputs.png
[graphic41]: ./media/stream-analytics-login-credentials-inputs-outputs/41-stream-analytics-login-credentials-inputs-outputs.png
[graphic42]: ./media/stream-analytics-login-credentials-inputs-outputs/42-stream-analytics-login-credentials-inputs-outputs.png
[graphic43]: ./media/stream-analytics-login-credentials-inputs-outputs/43-stream-analytics-login-credentials-inputs-outputs.png
 

<!---HONumber=Mooncake_0314_2016-->