
默认情况下，可匿名调用移动应用后端中的 API。接下来，需限制为仅可访问已验证的客户端。

+ **Node.js 后端（通过门户）**：
	
	在移动应用的“设置”中，单击“简易表”并选择相应表。单击“更改权限”，为所有权限选择“仅已验证的访问”，然后单击“保存”。

+ **.NET 后端 (C#)**：

	在服务器项目中，导航到“控制器”>“TodoItemController.cs”。将 `[Authorize]` 属性添加到“TodoItemController”类，如下所示。若要限制为仅可访问特定方法，还可只向这些方法应用此属性（而非类）。重新发布服务器项目。


        [Authorize]
        public class TodoItemController : TableController<TodoItem>

+ **Node.js 后端（通过 Node.js 代码）**：
	
	若要访问表时需验证身份，请向 Node.js 服务器脚本添加以下行：


        table.access = 'authenticated';

	有关更多详细信息，请参阅[如何：要求在访问表时进行身份验证](/documentation/articles/app-service-mobile-node-backend-how-to-use-server-sdk/#howto-tables-auth)。若要了解如何从网站下载快速入门代码项目，请参阅[如何：使用 Git 下载 Node.js 后端快速入门代码项目](/documentation/articles/app-service-mobile-node-backend-how-to-use-server-sdk/#download-quickstart)。

<!---HONumber=Mooncake_0919_2016-->