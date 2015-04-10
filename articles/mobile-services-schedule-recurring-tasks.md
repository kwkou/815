<properties linkid="develop-mobile-tutorials-schedule-backend-tasks" urlDisplayName="计划后端任务" pageTitle="使用计划程序计划后端任务 - 移动服务" metaKeywords="" description="使用 Azure 移动服务计划程序来计划移动应用程序的作业。" metaCanonical="" services="" documentationCenter="Mobile" title="Schedule recurring jobs in Mobile Services" authors="glenga" solutions="" manager="" editor="" />

<tags ms.service="mobile-services" ms.workload="mobile" ms.tgt_pltfrm="mobile-multiple" ms.devlang="multiple" ms.topic="article" ms.date="09/26/2014" ms.author="glenga" />

# 在移动服务中计划定期作业 

<div class="dev-center-tutorial-subselector">
	<a href="/zh-cn/documentation/articles/mobile-services-dotnet-backend-schedule-recurring-tasks/" title=".NET backend">.NET 后端</a> | <a href="/zh-cn/documentation/articles/mobile-services-schedule-recurring-tasks/"  title="JavaScript backend" class="current">JavaScript 后端</a>
</div>
 
本主题说明如何使用管理门户中的作业计划程序功能来定义服务器脚本代码，该代码将基于你定义的计划执行。在此情况下，脚本将定期检查远程服务（在本主题中为 Twitter），并在新表中存储结果。可以计划的其他一些定期任务包括：

+ 存档旧的或重复的数据记录。
+ 请求和存储外部数据，例如推文、RSS 条目和地点信息。
+ 处理存储的图像或调整其大小。

本教程将引导你完成以下步骤，以使用作业计划程序创建一个计划的作业，该作业将从 Twitter 请求推文数据，并在新的 Updates 表中存储推文：

+ [注册以获取 Twitter 访问权限并存储凭据]
+ [创建新的 Updates 表]
+ [创建新的计划作业]

##<a name="get-oauth-credentials"></a>注册以获取 Twitter v1.1 API 访问权限并存储凭据

[WACOM.INCLUDE [mobile-services-register-twitter-access](../includes/mobile-services-register-twitter-access.md)]

##<a name="create-table"></a>创建新的 Updates 表

接下来，你需要创建一个用于存储推文的新表。

2. 在管理门户中，单击移动服务的**数据**选项卡，然后单击**+创建**。

   ![][2]

   此时将显示"创建新表"对话框。

3. 在**表名**中键入 _Updates_，然后单击勾选按钮。

   ![][3]

  	此时将会创建新的存储表**Updates**。 

##<a name="add-job"></a>创建新的计划作业  

现在，你可以创建计划的作业，用于访问 Twitter 并在新的 Updates 表中存储推文数据。

2. 单击**计划程序**选项卡，然后单击**+创建**。 

   ![][4]

    >[WACOM.NOTE]当您在免费级别 <em>免费</em> ，您只能一次运行一个计划作业。在付费版本级别中，一次最多可以运行 10 个计划的作业。

3. 在计划程序对话框中，为**作业名称**输入 _getUpdates_，设置计划间隔和单位，然后单击勾选按钮。 
   
   ![][5]

   此时将创建一个名为**getUpdates**的新作业。 

4. 单击刚刚创建的新作业，然后单击**脚本**选项卡。

   ![][6] 

5. 将占位符函数**getUpdates**替换为以下代码：

		var updatesTable = tables.getTable('Updates');
		var request = require('request');
		var twitterUrl = "https://api.twitter.com/1.1/search/tweets.json?q=%23mobileservices&result_type=recent";

		// Get the service configuration module.
		var config = require('mobileservice-config');
		
		// Get the stored Twitter consumer key and secret. 
		var consumerKey = config.twitterConsumerKey,
		    consumerSecret = config.twitterConsumerSecret
		// Get the Twitter access token from app settings.    
		var accessToken= config.appSettings.TWITTER_ACCESS_TOKEN,
		    accessTokenSecret = config.appSettings.TWITTER_ACCESS_TOKEN_SECRET;
		
		function getUpdates() {   
		    // Check what is the last tweet we stored when the job last ran
		    // and ask Twitter to only give us more recent tweets
		    appendLastTweetId(
		        twitterUrl, 
		        function twitterUrlReady(url){            
		            // Create a new request with OAuth credentials.
		            request.get({
		                url: url,                
		                oauth: {
		                    consumer_key: consumerKey,
		                    consumer_secret: consumerSecret,
		                    token: accessToken,
		                    token_secret: accessTokenSecret
		                }},
		                function (error, response, body) {
		                if (!error && response.statusCode == 200) {
		                    var results = JSON.parse(body).statuses;
		                    if(results){
		                        console.log('Fetched ' + results.length + ' new results from Twitter');                       
		                        results.forEach(function (tweet){
		                            if(!filterOutTweet(tweet)){
		                                var update = {
		                                    twitterId: tweet.id,
		                                    text: tweet.text,
		                                    author: tweet.user.screen_name,
		                                    date: tweet.created_at
		                                };
		                                updatesTable.insert(update);
		                            }
		                        });
		                    }            
		                } else { 
		                    console.error('Could not contact Twitter');
		                }
		            });
		
		        });
		 }
		// Find the largest (most recent) tweet ID we have already stored
		// (if we have stored any) and ask Twitter to only return more
		// recent ones
		function appendLastTweetId(url, callback){
		    updatesTable
		    .orderByDescending('twitterId')
		    .read({success: function readUpdates(updates){
		        if(updates.length){
		            callback(url + '&since_id=' + (updates[0].twitterId + 1));           
		        } else {
		            callback(url);
		        }
		    }});
		}
		
		function filterOutTweet(tweet){
		    // Remove retweets and replies
		    return (tweet.text.indexOf('RT') === 0 || tweet.to_user_id);
		}


   此脚本将使用存储的凭据调用 Twitter 查询 API，以请求包含哈希标记 `#mobileservices`的最新推文。在表中存储结果之前，将从结果中删除重复的推文和回复。

   >[WACOM.NOTE]此示例假设每次按计划运行期间只会在表中插入少量的行。如果在一个循环中插入许多的行，那么，在免费版本级别上运行时可能会耗尽连接。在此情况下，你应该分批执行插入。有关详细信息，请参阅 <a href="/zh-cn/documentation/articles/mobile-services-how-to-use-server-scripts/#bulk-inserts">如何：执行批量插入。</a>。

6. 单击**运行一次**以测试脚本。 

  	![][7]

   此时将会保存并执行该作业，不过它在计划程序中保持为禁用状态。

7. 依次单击返回按钮、**数据**、**Updates**表和**浏览**，然后检查 Twitter 数据是否已插入表中。

   ![][8]

8. 单击返回按钮，单击**计划程序**，选择**getUpdates**，然后单击**启用**。

   ![][9]

   这样，作业将按指定的计划（在本例中为每隔一小时）运行。

祝贺你！你已成功地在移动服务中创建了一个新的计划作业。在你禁用或修改此作业之前，它将会按计划执行。

## <a name="nextsteps"> </a>后续步骤

* [移动服务服务器脚本参考]
  <br/>了解有关注册和使用服务器脚本的详细信息。

<!-- Anchors. -->
[注册以获取 Twitter 访问权限并存储凭据]: #get-oauth-credentials
[创建新的 Updates 表]: #create-table
[创建新的计划作业]: #add-job
[后续步骤]: #next-steps

<!-- Images. -->
[0]: ./media/mobile-services-schedule-recurring-tasks/mobile-twitter-my-apps.png
[1]: ./media/mobile-services-schedule-recurring-tasks/mobile-twitter-app-secrets.png
[2]: ./media/mobile-services-schedule-recurring-tasks/mobile-data-tab-empty-cli.png
[3]: ./media/mobile-services-schedule-recurring-tasks/mobile-create-updates-table.png
[4]: ./media/mobile-services-schedule-recurring-tasks/mobile-schedule-new-job-cli.png
[5]: ./media/mobile-services-schedule-recurring-tasks/mobile-create-job-dialog.png
[6]: ./media/mobile-services-schedule-recurring-tasks/mobile-schedule-job-script-new.png
[7]: ./media/mobile-services-schedule-recurring-tasks/mobile-schedule-job-script.png
[8]: ./media/mobile-services-schedule-recurring-tasks/mobile-browse-updates-table.png
[9]: ./media/mobile-services-schedule-recurring-tasks/mobile-schedule-job-enabled.png
[10]: ./media/mobile-services-schedule-recurring-tasks/mobile-schedule-job-app-settings.png
[11]: ./media/mobile-services-schedule-recurring-tasks/mobile-identity-tab-twitter-only.png

<!-- URLs. -->
[移动服务服务器脚本参考]: /zh-cn/documentation/articles/mobile-services-how-to-use-server-scripts/
[windowsazure.cn]: http://www.windowsazure.cn/
[Azure 管理门户]: https://manage.windowsazure.cn/
[注册应用以便在移动服务中进行 Twitter 登录]: /zh-cn/documentation/articles/mobile-services-how-to-register-twitter-authentication
[Twitter 开发人员]: http://go.microsoft.com/fwlink/p/?LinkId=268300
[应用程序设置]: http://msdn.microsoft.com/library/windowsazure/b6bb7d2d-35ae-47eb-a03f-6ee393e170f7
