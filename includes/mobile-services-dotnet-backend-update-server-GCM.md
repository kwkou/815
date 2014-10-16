1.  在 Visual Studio 的解决方案资源管理器中，展开移动服务项目中的 "Controllers" 文件夹。打开 TodoItemController.cs。在文件顶部，添加以下 `using` 语句：

         using System;
        using System.Collections.Generic;

2.  使用以下代码更新 `PostTodoItem` 方法定义：

        public async Task<IHttpActionResult> PostTodoItem(TodoItem item)
        {
        TodoItem current = await InsertAsync(item);

        Dictionary<string, string> data = new Dictionary<string, string>()
            {
        { "message", "Hello from Mobile Services"}
            };
        GooglePushMessage message = new GooglePushMessage(data, TimeSpan.FromHours(1));

        try
            {
        var result = await Services.Push.SendAsync(message);
        Services.Log.Info(result.State.ToString());
            }
        catch (System.Exception ex)
            {
        Services.Log.Error(ex.Message, null, "Push.SendAsync Error");
            }
        return CreatedAtRoute("Tables", new { id = current.Id }, current);
        }

    这段代码可在插入 Todo 项之后发送推送通知（包含所插入项的文本）。在发生错误的情况下，这段代码将添加一个错误日志条目，该条目可在管理门户中的移动服务的“日志”选项卡上查看。 


