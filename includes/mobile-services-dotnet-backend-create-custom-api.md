1.  在 Visual Studio 的解决方案资源管理器中，右键单击移动服务项目的 Controllers 文件夹，展开“添加”，然后单击“新基架项” 。

    这样将显示“添加基架”对话框。

2.  展开“Azure 移动服务” 并单击“Azure 移动服务自定义控制器” ，然后单击“添加”， 提供 `CompleteAllController` 的“控制器名称” ，然后再次单击“添加”。 

    ![][0]

    这样将创建新的名为 "CompleteAllController" 的空控制器类。

> [WACOM.NOTE] 如果你的对话框没有移动服务特定的基架，请创建新的“Web API 控制器 - 空” 。在这个新控制器类中，添加公共 "Services" 属性，它将返回 "ApiServices" 类型。此属性用于从控制器内部访问服务器特定的设置。

1.  在新的 CompleteAllController.cs 项目文件中，添加以下 "using" 语句：

        using System.Threading.Tasks;
        using todolistService.Models;

    在以上代码中，请将 `todolistService` 替换为你的移动服务项目的命名空间，它应该是在移动服务名称后追加 `Service`。

2.  将以下代码添加到新控制器：

        // POST api/completeall        
        public Task<int> Post()
        {
        using (todolistContext context = new todolistContext())
            {
        // Get the database from the context.
        var database = context.Database;

        // Create a SQL statement that sets all uncompleted items
        // to complete and execute the statement asynchronously.
        var sql = @"UPDATE TodoItems SET Complete = 1 " +
        @"WHERE Complete = 0; SELECT @@ROWCOUNT as count";
        var result = database.ExecuteSqlCommandAsync(sql);

        // Log the result.
        Services.Log.Info(string.Format("{0} items set to 'complete'.", 
        result.ToString()));

        return result;
            }
        }

    在以上代码中，将 `todolistContext` 替换为你的数据模型的 DbContext 名称，它应该是在移动服务名称后追加 `Context`。这段代码使用[数据库类][]来直接访问 "TodoItems" 表，在所有项上设置 completed 标志。此方法支持 POST 请求，已更改行的数量将以整数值形式返回至客户端。

    > [WACOM.NOTE] 设置了默认权限，这意味着应用程序的任何用户都能够调用自定义 API。但是，应用程序密钥无法安全地分发或存储，不能视为安全的凭据。出于这一原因，你应该考虑将访问权限仅限于经过身份验证的用户，仅允许这些用户执行修改数据或影响移动服务的操作。

接下来，你将修改快速启动应用程序，以添加新按钮和用于异步调用新的自定义 API 的代码。

  [0]: ./media/mobile-services-dotnet-backend-create-custom-api/add-custom-api-controller.png
  [数据库类]: http://msdn.microsoft.com/en-us/library/system.data.entity.database.aspx
