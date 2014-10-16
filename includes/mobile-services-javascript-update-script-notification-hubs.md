最后，你必须更新注册到 TodoItem 表上的插入操作的脚本，以便发送通知。

1.  单击“TodoItem” ，再单击“脚本”并选择“插入”。 

    ![][0]

2.  将 insert 函数替换为以下代码，然后单击“保存”： 

        function insert(item, user, request) {
        // Define a payload for the Windows Store toast notification.
        var payload = '<?xml version="1.0" encoding="utf-8"?><toast><visual>' +    
        '<binding template="ToastText01">  <text id="1">' +
        item.text + '</text></binding></visual></toast>';

        request.execute({
        success:function() {
        // If the insert succeeds, send a notification.
        push.wns.send(null,payload, 'wns/toast', {
        success:function (pushResponse) {
        console.log("Sent push:", pushResponse);
        request.respond();
                        },              
        错误：function (pushResponse) {
        console.log("Error Sending push:", pushResponse);
        request.respond(500, { error:pushResponse });
                            }
                        });
                    }
                });
        }

    插入成功后，此插入脚本会向所有 Windows 应用商店应用程序注册发送推送通知（包含插入项的文本）。

  [0]: ./media/mobile-services-javascript-update-script-notification-hubs/mobile-insert-script-push2.png
