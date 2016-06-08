
1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn/)，单击“移动服务”，然后单击你的移动服务。

2. 单击“推送”选项卡、为“权限”选择“仅经过身份验证的用户”，单击“保存”，然后单击“编辑脚本”。
	
	这样，你便可以自定义推送通知注册回调函数。如果你使用 Git 来编辑源代码，则在 `.\service\extensions\push.js` 中找到这一相同注册函数。

3. 将现有**注册**函数替换为以下代码，然后单击“保存”：

		exports.register = function (registration, registrationContext, done) {   
		    // Get the ID of the logged-in user.
			var userId = registrationContext.user.userId;    
		    
			// Perform a check here for any disallowed tags.
			if (!validateTags(registration))
			{
				// Return a service error when the client tries 
		        // to set a user ID tag, which is not allowed.		
				done("You cannot supply a tag that is a user ID");		
			}
			else{
				// Add a new tag that is the user ID.
				registration.tags.push(userId);
				
				// Complete the callback as normal.
				done();
			}
		};
		
		function validateTags(registration){
		    for(var i = 0; i < registration.tags.length; i++) { 
		        console.log(registration.tags[i]);           
				if (registration.tags[i]
				.search(/microsoft:/i) !== -1){
					return false;
				}
				return true;
			}
		}

	这会向注册添加一个标记（已登录用户的 ID）。将验证提供的标记以防止用户注册其他用户的 ID。通知发送给该用户后，用户可在此设备和用户所注册的其他任何设备上接收通知。

4. 依次单击向后箭头、“数据”选项卡、“TodoItem”、“脚本”，然后选择“插入”。

<!---HONumber=Mooncake_0118_2016-->