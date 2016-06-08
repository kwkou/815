

1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn/)，单击“移动服务”，然后选择你的移动服务。

2. 单击“API”选项卡，然后单击“创建”。这将显示“创建新的自定义 API”对话框。

3. 在“API 名称”中键入 _completeall_，然后单击勾选按钮创建新 API。

	> [AZURE.TIP] 使用默认权限和应用密钥的任何人都可以调用自定义 API。但是，应用程序密钥不被视为安全的凭据，因为无法安全地分发或存储它。请考虑仅限经过身份验证的用户访问。

4. 在 API 表中单击 **completeall**。

5. 单击“脚本”选项卡、将现有代码替换为以下代码，然后单击“保存”。这段代码使用 [mssql 对象] 来直接访问 **todoitem** 表，以在所有项上设置 `complete` 标志。由于使用了 **exports.post** 函数，客户端发送 POST 请求以执行操作。已更改行的数量将以整数值形式返回至客户端。


		exports.post = function(request, response) {
			var mssql = request.service.mssql;
			var sql = "UPDATE todoitem SET complete = 1 " + 
                "WHERE complete = 0; SELECT @@ROWCOUNT as count";
			mssql.query(sql, {
				success: function(results) {			
					if(results.length == 1)							
						response.send(200, results[0]);			
				}
			})
		};


> [AZURE.NOTE] 提供给自定义 API 函数的 [request](http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554218.aspx) 和 [response](http://msdn.microsoft.com/zh-cn/library/windowsazure/dn303373.aspx) 对象由 [Express.js 库](http://go.microsoft.com/fwlink/p/?LinkId=309046)实现。

<!-- Anchors. -->

<!-- Images. -->

<!-- URLs. -->

[mssql 对象]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554212.aspx

<!---HONumber=Mooncake_0118_2016-->