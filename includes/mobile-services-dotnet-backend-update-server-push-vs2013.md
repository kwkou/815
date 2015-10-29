这些步骤创建新的自定义 [ApiController](http://go.microsoft.com/fwlink/p/?LinkId=512673)，后者将推送通知发送到应用。你可以在 [TableController](http://msdn.microsoft.com/zh-cn/library/azure/dn643359.aspx) 中或者在后端服务内的其他任何位置实施这一相同代码。

1. 在 Visual Studio 的解决方案资源管理器中，右键单击移动服务项目的 Controllers 文件夹，展开“添加”，然后单击“新基架项”。

	这样将显示“添加基架”对话框。

2. 展开“Azure 移动服务”并单击“Azure 移动服务自定义控制器”，然后单击“添加”，提供 `NotifyAllUsersController` 的“控制器名称”，然后再次单击“添加”。

	![Web API“添加基架”对话框](./media/mobile-services-dotnet-backend-update-server-push-vs2013/add-custom-api-controller.png)

	这将创建新的名为 **NotifyAllUsersController** 的空控制器类。

3. 在新的 NotifyAllUsersController.cs 项目文件中，添加以下 **using** 语句：

        using Newtonsoft.Json.Linq;
        using System.Threading.Tasks;

4. 添加在控制器上实施 POST 方法的以下代码：

        public async Task<bool> Post(JObject data)
        {
            try
            {
                // Define the XML paylod for a WNS native toast notification 
				// that contains the value supplied in the POST request.
                string wnsToast = 
                    string.Format("<?xml version="1.0" encoding="utf-8"?>" +
                    "<toast><visual><binding template="ToastText01">" + 
                    "<text id="1">{0}</text></binding></visual></toast>", 
                    data.GetValue("toast").Value<string>());

                // Define the WNS native toast with the payload string.
                WindowsPushMessage message = new WindowsPushMessage();
                message.XmlPayload = wnsToast;

                // Send the toast notification.
                await Services.Push.SendAsync(message);
                return true;
            }
            catch (Exception e)
            {
                Services.Log.Error(e.ToString());
            }
            return false;
        }

	>[AZURE.NOTE]此 POST 方法可以由具有应用程序密钥的任何客户端调用，这样并不安全。若要保护终结点，请将属性 `[AuthorizeLevel(AuthorizationLevel.User)]` 应用于要求身份验证的方法或类。

<!---HONumber=74-->