
1. 登录到 [Azure 管理门户]、单击"移动服务"，然后单击您的移动服务。

   	![](./media/mobile-services-javascript-backend-push-notifications-app-users/mobile-services-selection.png)

2. 单击"推送"选项卡、为"权限"选择"仅经过身份验证的用户"，然后单击"编辑脚本"。

   	![](./media/mobile-services-javascript-backend-push-notifications-app-users/mobile-services-push-registration-endpoint.png)
	
	这样，您便可以自定义推送通知注册回调函数。如果使用 Git 编辑源代码，可以在 `.\service\extensions\push.js` 中找到与此相同的注册函数。

3. 将现有的 **register** 函数替换为以下代码：

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

4. 依次单击向后箭头、"数据"选项卡、"TodoItem"、"脚本"，然后选择"插入"。 
<!--HONumber=41-->
