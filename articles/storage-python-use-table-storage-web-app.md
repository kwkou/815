<properties linkid="develop-python-web-app-with-blob-storage" urlDisplayName="Web App with Blob Storage" pageTitle="Python web app with table storage | Microsoft Azure" metaKeywords="Azure table storage Python, Azure Python application, Azure Python tutorial, Azure Python example" description="A tutorial that teaches you how to create a Python web application using the Azure Client Libraries. Django is used as the web framework." metaCanonical="" services="storage" documentationCenter="Python" title="Python Web Application using Table Storage" authors="" solutions="" videoId="" scriptId="" manager="" editor="mollybos" />

# 使用表存储创建 Python Web 应用程序

在本教程中，你将了解如何创建结合使用表存储和 Python 的 Azure 客户端库的应用程序。如果这是你的第一个 Python Azure 应用程序，则你可能希望首先查看 [Django Hello World Web 应用程序][]。

对于本指南，你将创建一个可部署到 Azure 的基于 Web 的任务列表应用程序。用户可以通过任务列表来检索任务、添加新任务以及将任务标记为已完成。我们将使用 Django 作为 Web 框架。

任务项存储在 Azure 存储空间中。Azure 存储空间提供了具有容错能力且可用性非常好的非结构化数据存储。Azure 存储空间包含一些可用来存储和访问数据的数据结构，你可以通过 Azure SDK for Python 中包含的 API 或通过 REST API 利用
存储服务。有关详细信息，请参阅[在 Azure 中存储和访问数据][]。

你将了解到以下内容：

-   如何使用 Azure 表存储服务

完成的应用程序的屏幕快照将类似于下图（添加的任务项将不同）：

![][]

[WACOM.INCLUDE [create-account-note][]]

## 设置开发环境

**注意：**如果你需要安装 Python 或客户端库，请参阅 [Python 安装指南][]。

*有关 Windows 的说明*：如果你使用的是 Windows WebPI 安装程序，那么你已安装了 Django 和客户端库。

## 在 Azure 中创建存储帐户

[WACOM.INCLUDE [create-storage-account][]]

## 创建 Django 项目

下面是创建应用程序的步骤：

-   创建一个名为“TableserviceSample”的默认 Django 项目
-   从命令行中，使用 cd 命令切换到要在其中存储你的代码的目录，然后运行以下命令：

        django-admin.py startproject TableserviceSample

-   将新的 Python 文件 **views.py** 添加到项目中
-   将以下代码添加到 **views.py** 中以导入所需的 Django 支持：

        from django.http import HttpResponse
        from django.template.loader import render_to_string
        from django.template import Context

-   在 **TableserviceSample/TableserviceSample** 文件夹下创建一个名为 **templates** 的新文件夹。
-   编辑应用程序设置，以便可以找到你的模板。打开 **settings.py** 并将以下条目添加到 INSTALLED\_APPS 中：

        'TableserviceSample',

-   将新的 Django 模板文件 **mytasks.html** 添加到 **templates** 文件夹中并将以下代码添加到文件中：

<!-- -->

    <html>
    <head><title></title></head>
    <body>
    <h2>My Tasks</h2> <br>
    <table border="1"> 
    <tr>
    <td>Name</td><td>Category</td><td>Date</td><td>Complete</td><td>Action</td></tr>
    {% for entity in entities %}
    <form action="update_task" method="GET">
    <tr><td>{{entity.name}} <input type="hidden" name='name' value="{{entity.name}}"></td>
    <td>{{entity.category}} <input type="hidden" name='category' value="{{entity.category}}"></td>
    <td>{{entity.date}} <input type="hidden" name='date' value="{{entity.date}}"></td>
    <td>{{entity.complete}} <input type="hidden" name='complete' value="{{entity.complete}}"></td>

    <td><input type="submit" value="Complete"></td>
    </tr>
    </form>
    {% endfor %}
    </table>
    <br>
    <hr>
    <table border="1">
    <form action="add_task" method="GET">
    <tr><td>Name:</td><td><input type="text" name="name"></input></td></tr>
    <tr><td>Category:</td><td><input type="text" name="category"></input></td></tr>
    <tr><td>Item Date:</td><td><input type="text" name="date"></input></td></tr>
    <tr><td><input type="submit" value="add task"></input></td></tr>
    </form>
    </table>
    </body>
    </html>    

## 导入 windowsazure 存储模块

将以下代码添加到 **views.py** 顶部，紧跟在 Django 导入之后

    from azure.storage import TableService

## 获取存储帐户名称和帐户密钥

将以下代码添加到 **views.py** 中，紧跟在 windowsazure 导入之后，并将“youraccount”和“yourkey”替换为你的真实帐户名称和密钥。你可以从 azure 管理门户获取帐户名称和密钥。

    account_name = 'youraccount'
    account_key = 'yourkey'

## 创建 TableService

将以下代码添加到 **account\_name** 之后：

    table_service = TableService(account_name=account_name, account_key=account_key)
    table_service.create_table('mytasks')

## 列表任务

将函数 list\_tasks 添加到 **views.py** 中：

    def list_tasks(request): 
    entities = table_service.query_entities('mytasks', '', 'name,category,date,complete')    
    html = render_to_string('mytasks.html', Context({'entities':entities}))
    return HttpResponse(html)

## 添加任务

将函数 add\_task 添加到 **views.py** 中：

    def add_task(request):
    name = request.GET['name']
    category = request.GET['category']
    date = request.GET['date']
    table_service.insert_entity('mytasks', {'PartitionKey':name+category, 'RowKey':date, 'name':name, 'category':category, 'date':date, 'complete':'No'}) 
    entities = table_service.query_entities('mytasks', '', 'name,category,date,complete')
    html = render_to_string('mytasks.html', Context({'entities':entities}))
    return HttpResponse(html)

## 更新任务状态

将函数 update\_task 添加到 **views.py** 中：

    def update_task(request):
    name = request.GET['name']
    category = request.GET['category']
    date = request.GET['date']
    partition_key = name + category
    row_key = date
    table_service.update_entity('mytasks', partition_key, row_key, {'PartitionKey':partition_key, 'RowKey':row_key, 'name':name, 'category':category, 'date':date, 'complete':'Yes'})
    entities = table_service.query_entities('mytasks', '', 'name,category,date,complete')    
    html = render_to_string('mytasks.html', Context({'entities':entities}))
    return HttpResponse(html)

## 映射 url

现在，你需要在 Django 应用程序中映射 URL。打开 **urls.py** 并将以下映射添加到 urlpatterns 中：

    url(r'^$', 'TableserviceSample.views.list_tasks'),
    url(r'^list_tasks$', 'TableserviceSample.views.list_tasks'),
    url(r'^add_task$', 'TableserviceSample.views.add_task'),
    url(r'^update_task$', 'TableserviceSample.views.update_task'),

## 运行应用程序

-   切换到 **TableserviceSample** 目录（如果你尚未这么做）并运行以下命令：

    python manage.py runserver

-   使你的浏览器指向：`http://127.0.0.1:8000/`。将 8000 替换为实际端口号。

现在，你可以单击 **Add Task**（添加任务）来创建一个任务，然后单击 **Complete**（完成）按钮来更新任务并将其状态设置为“Yes”（是）。

## 在计算模拟器中运行应用程序，发布和停止/删除你的应用程序

现在，你已在内置的 Django 服务器上成功运行了你的应用程序，你可以通过将它部署到 Azure 模拟器（仅限 Windows），然后发布到 Azure 来进一步对它进行测试。有关如何执行此操作的一般说明，请参阅 [Django Hello World Web 应用程序][]一文，其中详细讨论了这些步骤。

## 后续步骤

现在，你已了解了 Azure 表存储服务的基础知识，单击下面的链接可了解如何执行更复杂的存储任务。

-   查看 MSDN 参考：[在 Azure 中存储和访问数据][]
-   访问 Azure 存储空间团队博客：<http://blogs.msdn.com/b/windowsazurestorage/>

  [Django Hello World Web 应用程序]: http://www.windowsazure.cn/zh-cn/develop/python/tutorials/web-app-with-django/
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  []: ./media/storage-python-use-table-storage-web-app/web-app-with-storage-Finaloutput-mac.png
  [create-account-note]: ../includes/create-account-note.md
  [Python 安装指南]: http://windowsazure.com/en-us/documentation/articles/python-how-to-install
  [create-storage-account]: ../includes/create-storage-account.md
