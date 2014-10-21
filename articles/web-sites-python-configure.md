<properties linkid="develop-python-tutorials-web-sites-configuration" urlDisplayName="Configuring Python with Azure Web Sites" pageTitle="Configuring Python with Azure Web Sites" metaKeywords="" description="This tutorial describes options for authoring and configuring a basic Web server Gateway Interface (WSGI) compliant Python application on Azure Web Sites." metaCanonical="" services="web-sites" documentationCenter="Python" title="Configuring Python with Azure Web Sites" authors="" solutions="" manager="" editor="" />

# 使用 Azure 网站配置 Python

本教程介绍用于在 Azure 网站上创作并配置符合基本 Web 服务器网关接口 (WSGI) 的 Python 应用程序的各种方法。Azure 网站的使用很简单，并且你的 Python 应用程序将有缩放和扩展到其他 Azure 服务的空间。Azure 网站平台包括 Python 2.7 和 Python 的常规 wfastcgi.py FastCGI 处理程序。你只需将网站配置为使用 Python 处理程序即可。

有关在 Azure 网站上配置 Django 框架的更复杂示例，请参阅以下教程：
[][]<http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-python-create-deploy-django-app/></a>。

## WSGI 支持

WSGI 是 [PEP 3333][PEP 3333] 描述的 Python 标准，用于定义 Web 服务器和 Python 之间的接口。它提供了用于使用 Python 编写各种 Web 应用程序和框架的标准化接口。当今常用的 Python Web 框架都使用 WSGI。Azure 网站支持任何此类框架；另外，高级用户甚至可以创作自己的框架，只要自定义处理程序遵循 WSGI 规范指南即可。

## 网站创建

本教程使用现有 Azure 订阅以及对 Azure 管理门户的访问权限。[][1]<http://www.windowsazure.cn/zh-cn/manage/services/web-sites/how-to-create-websites/></a> 上提供了有关创建网站的详细指南。

简言之，如果你没有现成的网站，则可以从 Azure 管理门户创建一个。选择“网站”功能并使用“快速创建”选项，然后为你的网站指定 URL。

![][]

## Git 发布

使用你新创建的网站的“快速启动”或“仪表板”选项卡来配置 Git 发布。本教程使用 Git 来创建和管理我们的 Python 网站并将其发布到 Azure 网站。

![][2]

在设置 Git 发布之后，将创建 Git 存储库并使其与你的网站相关联。将显示该存储库的 URL，并且之后可将其用于将数据从本地开发环境推送到云。若要通过 Git 发布应用程序，请确保还安装了 Git 客户端，并使用提供的说明将你的网站内容推送到 Azure 网站。

## 网站内容

例如，我们使用基本 Python 应用程序，其中的基本 WSGI 处理程序指明利用 Azure 网站中的 Python 支持所需的最少工作量。此框架性 Python 应用程序稍后可用于开始创作各种解决方案，其复杂性范围从以下示例一直到完备的 Web 框架。

下面是基本 WSGI 处理程序的代码。它类似于 [PEP 3333][PEP 3333] 规范建议作为符合 WSGI 的应用程序的起点的代码。我们将此内容保存到了在网站根下的 ConfigurePython 文件夹中创建的名为 ConfigurePython.py 的文件中：

    def application(environ, start_response):
        status = '200 OK'
        response_headers = [('Content-type', 'text/plain')]
        start_response(status, response_headers)
        yield 'Hello from Azure Websites\n'

*application* 是可调用的 Python，它将充当符合 WSGI 的服务器调用的入口点。此可调用对象接受 2 个位置参数：

-   *environ*：具有各种环境变量的字典
-   *start\_response*：Web 服务器提供的用于传送 HTTP 状态和响应标头的可调用参数

此处理程序将为对它发出的每个请求返回纯文本“Hello from Azure Websites”。

## 配置选项

有 2 种用于使用 Azure 网站配置你的 Python 应用程序的不同方法。

### 方法 1：门户

1.1. 通过门户中的“配置”选项卡注册 FastCGI 处理程序。
在本示例中，我们使用随 Azure 网站提供的 Python 的 FastCGI 处理程序。若要执行相同操作，请对你的脚本处理器和 FastCGI 处理程序参数使用以下路径：

-   Python 脚本处理器路径：D:\\python27\\python.exe
-   Python FastCGI 处理程序路径：D:\\python27\\scripts\\wfastcgi.py

![][3]

1.2. 通过门户中的同一“配置”选项卡配置应用程序设置。
应用程序设置将转换为环境变量。这是可用于 Python 应用程序所需的配置值的机制。对于此基本示例应用程序，我们配置了以下值：

-   PYTHONPATH 通知 Python 要在其中搜索模块的目录。Azure 网站提供 D:\\home\\site\\wwwroot 作为指向你的网站的根的语法糖。
-   WSGI\_HANDLER 记录模块或包名称以及要使用的属性。

![][4]

### 方法 2：web.config

该配置替代方法是使用网站根下的 web.config 文件执行操作，如下所述。使用 web.config 方法为 Web 应用程序提供了更好的潜在可移植性。有 2 种方法可用于将请求路由到 Web 应用程序：设置处理 \* 路径的处理程序，以指示 IIS 通过 Python 路由每个传入请求；或设置 Python 将处理的特定路径，以便稍后使用 URL 重写将各种 URL 重定向到我们所选的路径。事实上，建议采用后一种方法 – 使用网站根下的空处理程序文件来充当请求目标（在我们的示例中为 handler.fcgi）– 以提高性能。在前一个方案中，包括对静态内容（例如，图像文件和样式表）的请求的所有请求将必须通过 Python，从而影响 Web 服务器为访问静态文件提供的优化功能。使用后一种方法允许高效提供静态内容并只在必要时调用 Python。

2.1. 指定 PYTHONPATH 变量。

> 这会通知 Python 在哪里查找应用程序代码。在这里，也使用 D:\\home\\site\\wwwroot 作为网站的绝对路径。

2.2. 设置 WSGI\_HANDLER 变量。

> Azure 网站使用此值来指示 Python 调用我们的 WSGI 处理程序。此变量的值是一个 Python 表达式，在执行时，应返回表示 WSGI 处理程序的可调用值。

2.3. 添加 Python 的处理程序。

> 这会通知 Azure 网站 Python 应处理对路径 handler.fcgi 发出的请求。处理程序语法与以下示例中的 \<handlers\> 标记内的内容完全相同很重要，除非你具有自己的 FastCGI 处理程序或 Python 开发堆栈。

2.4. 重写到 handler.fcgi 的 URL。

> 一直请求 handler.fcgi 可能不是最好的主意。为选择要由 Python 处理程序处理的文件的路径，我们使用了 URL 重写，以便所有 URL 都将由我们的 Python 处理程序进行处理。

    <configuration>
        <appSettings>
            <add key="pythonpath" value="D:\home\site\wwwroot\ConfigurePython" />
            <add key="WSGI_HANDLER" value="ConfigurePython.application" />
        </appSettings>
        <system.webServer>
            <handlers>
                <add name="PythonHandler" 
                verb="*" path="handler.fcgi" 
                modules="FastCgiModule" 
                scriptProcessor="D:\Python27\Python.exe|D:\Python27\Scripts\wfastcgi.py" 
                resourceType="Either" />
            </handlers>
            <rewrite>
                <rules>
                    <rule name="Configure Python" stopProcessing="true">
                        <match url="(.*)" ignoreCase="false" />
                        <conditions>
                            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
                        </conditions>
                        <action type="Rewrite" url="handler.fcgi/{R:1}" appendQueryString="true" />
                    </rule>
                </rules>
            </rewrite>
        </system.webServer>
    </configuration> 

网站根下的示例文件夹结构如下（Python 文件夹和文件名称的大小写很重要并反映在 web.config 中）：

-   ConfigurePython\\ConfigurePython.py
-   web.config
-   handler.fcgi

因为我们重写到 handler.fcgi 的所有 URL 并通过 FastCGI 将该路径传递给 Python，所以我们需要创建同名的占位符文件，以便 IIS 不会返回 HTTP 404 错误。这是由于 IIS FastCGI 模块的内部行为，它强制要请求的文件必须先存在，然后才能将该文件传递给指定的脚本处理器应用程序。

浏览到你的网站来测试配置是否正确。对于此示例，在访问时将显示“Hello from Azure Websites”消息。

![][5]

  []: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-python-create-deploy-django-app/
  [PEP 3333]: http://www.python.org/dev/peps/pep-3333/
  [1]: http://www.windowsazure.cn/zh-cn/manage/services/web-sites/how-to-create-websites/
  []: ./media/web-sites-python-configure/configure-python-create-website.png
  [2]: ./media/web-sites-python-configure/configure-python-git.png
  [3]: ./media/web-sites-python-configure/configure-python-handler-mapping.png
  [4]: ./media/web-sites-python-configure/configure-python-app-settings.png
  [5]: ./media/web-sites-python-configure/configure-python-result.png
