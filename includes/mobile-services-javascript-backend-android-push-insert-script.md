
1. 在 [Azure 经典门户](https://manage.windowsazure.cn/)中，单击“数据”选项卡，然后单击“TodoItem”表。 
 
2. 在 **todoitem** 中，单击“脚本”选项卡，然后选择“插入”。
   
   	将显示当 **TodoItem** 表中发生插入时所调用的函数。

3. 将 insert 函数替换为以下代码，然后单击“保存”：

		function insert(item, user, request) {
		// Define a simple payload for a GCM notification.
	    var payload = {
	        "data": {
	            "message": item.text
	        }
	    };		
		request.execute({
		    success: function() {
		        // If the insert succeeds, send a notification.
		        push.gcm.send(null, payload, {
		            success: function(pushResponse) {
		                console.log("Sent push:", pushResponse, payload);
		                request.respond();
		                },              
		            error: function (pushResponse) {
		                console.log("Error Sending push:", pushResponse);
		                request.respond(500, { error: pushResponse });
		                }
		            });
		        },
		    error: function(err) {
		        console.log("request.execute error", err)
		        request.respond();
		    }
		  });
		}

   	这将会注册一个新的插入脚本，插入成功后，该脚本将使用 [gcm 对象](https://msdn.microsoft.com/zh-CN/library/dn126137.aspx)向所有已注册的设备发送推送通知。 

<!---HONumber=Mooncake_0118_2016-->