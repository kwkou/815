1. 在 **channels** 表的 insert.js 文件中，找到以下代码行，注释掉它们或将它们从文件中删除，然后保存所做的更改。

		sendNotifications(item.channelUri);

		function sendNotifications(uri) {
		    console.log("Uri: ", uri);
		    push.wns.sendToastText01(uri, {
		        text1: "Sample toast from sample insert"
		    }, {
		        success: function (pushResponse) {
		            console.log("Sent push:", pushResponse);
		        }
		    });
		}
		
	当您保存对 insert.js 文件的更改时，新版本的脚本将会上载到您的移动服务。

2. 在"服务器资源管理器"中，展开 TodoItem 表、打开 insert.js 文件并将当前的 insert 函数替换为以下代码，然后保存所做的更改： 

		function insert(item, user, request) {
			request.execute({
				success: function() {
					request.respond();
					sendNotifications();
				}
			});
		
			function sendNotifications() {
				var channelsTable = tables.getTable('channels');
				channelsTable.read({
					success: function(devices) {
						devices.forEach(function(device) {
							push.wns.sendToastText04(device.channelUri, {
								text1: item.text
							}, {
								success: function(pushResponse) {
									console.log("Sent push:", pushResponse);
								}
							});
						});
					}
				});
			}
		}
		
	现在，当您插入新的 TodoItem 时，将有一个推送通知发送到所有已注册设备。
