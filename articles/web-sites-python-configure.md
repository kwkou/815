<properties 
	pageTitle="使用 Azure Web Apps 配置 Python" 
	description="本教程介绍用于在 Azure Web Apps 上创作并配置符合基本 Web 服务器网关接口 (WSGI) 的 Python 应用程序的选项。" 
	services="app-service" 
	documentationCenter="python" 
	tags="python"
	authors="huguesv" 
	manager="wpickett" 
	editor=""/>

<tags
	ms.service="app-service"
	ms.date="02/26/2016"
	wacn.date="04/26/2016"/>




# 使用 Azure Web Apps 配置 Python

本教程介绍用于在 [Azure Web Apps](/documentation/services/web-sites/) 上创作并配置符合基本 Web 服务器网关接口 (WSGI) 的 Python 应用程序的选项。

其中描述了 Git 部署的其他功能，如使用 requirements.txt 安装虚拟环境和包。

##<a name="bottle-django-flask"></a> Bottle、Django 还是 Flask？

Azure 应用商店包含用于 Bottle、Django 和 Flask 框架的模板。如果你正在 Azure 中开发第一个 Web 应用，或者你不熟悉 Git，我们建议你遵循以下教程之一，其中包括用于从 Windows 或 Mac 使用 Git 部署从库构建工作应用程序的分步说明：

- [使用 Bottle 创建 Web 应用](/documentation/articles/web-sites-python-create-deploy-bottle-app/)
- [使用 Django 创建 Web 应用](/documentation/articles/web-sites-python-create-deploy-django-app/)
- [使用 Flask 创建 Web 应用](/documentation/articles/web-sites-python-create-deploy-flask-app/)


##<a name="website-creation-on-portal"></a> 在 Azure 经典管理门户中创建 Web 应用

本教程使用现有 Azure 订阅以及对 Azure 经典管理门户的访问权限。

如果你没有现成的 Web 应用，则可从 [Azure 经典管理门户](https://manage.windowsazure.cn)创建一个。单击左下角的“新建”按钮。将出现一个窗口。依次单击“计算”、“Web 应用”和“快速创建”。

![](./media/web-sites-python-configure/configure-python-create-website.png)

##<a name="git-publishing"></a> Git 发布

按照[在 Azure Web 应用中使用 GIT 进行连续部署](/documentation/articles/web-sites-publish-source-control/)中的说明为新创建的 Web 应用配置 Git 发布。本教程使用 Git 来创建、管理 Python Web 应用以及将其发布到 Azure Web 应用。

在设置 Git 发布之后，将创建 Git 存储库并使其与你的 Web 应用相关联。将显示该存储库的 URL，并且之后可将其用于将数据从本地开发环境推送到云。若要通过 Git 发布应用程序，请确保还安装了 Git 客户端，并使用提供的说明将你的 Web 应用内容推送到 Azure Web 应用。


##<a name="application-overview"></a> 应用程序概述

在接下来的各节中，将创建以下文件。这些文件应放在 Git 存储库的根目录中。

    app.py
    requirements.txt
    runtime.txt
    web.config
    ptvs_virtualenv_proxy.py


##<a name="wsgi-handler"></a> WSGI 处理程序

WSGI 是 [PEP 3333](http://www.python.org/dev/peps/pep-3333/) 描述的 Python 标准，用于定义 Web 服务器和 Python 之间的接口。它提供了用于使用 Python 编写各种 Web 应用程序和框架的标准化接口。当今常用的 Python Web 框架都使用 WSGI。Azure Web Apps 支持任何此类框架；此外，高级用户甚至可以创作自己的框架，只要自定义处理程序遵循 WSGI 规范准则即可。

下面是定义自定义处理程序的 `app.py` 的一个示例：

    def wsgi_app(environ, start_response):
        status = '200 OK'
        response_headers = [('Content-type', 'text/plain')]
        start_response(status, response_headers)
        response_body = 'Hello World'
        yield response_body.encode()

    if __name__ == '__main__':
        from wsgiref.simple_server import make_server

        httpd = make_server('localhost', 5555, wsgi_app)
        httpd.serve_forever()

可以使用 `python app.py` 在本地运行此应用程序，然后在 Web 浏览器中浏览到 `http://localhost:5555`。


## 虚拟环境

尽管上面的示例应用程序不需要任何外部包，则您的应用程序很可能需要一些外部包。

为了帮助管理外部包依赖项，Azure Git 部署支持创建虚拟环境。

当 Azure 在存储库的根目录中检测到 requirements.txt 文件时，将自动创建名为 `env` 的虚拟环境。仅在第一次部署进执行此操作，或者在所选的 Python 运行时发生更改后进行任何部署的过程中执行此操作。

您可能需要创建虚拟环境用于开发，但不将其包括在 Git 存储库中。


## 包管理

Requirements.txt 中列出的包将使用 pip 自动安装在虚拟环境中。在每次部署时都会发生这种情况，但如果已安装包，则 pip 将跳过安装。

示例 `requirements.txt`：

    azure==0.8.4


## Python 版本

[AZURE.INCLUDE [web-sites-python-customizing-runtime](../includes/web-sites-python-customizing-runtime.md)]

示例 `runtime.txt`：

    python-2.7


## Web.config

需要创建一个 web.config 文件以指定服务器应如何处理请求。

请注意，如果在存储库中有一个 Web.x.y 文件，其中 x.y 与所选的 Python 运行时匹配，则 Azure 会自动将相应的文件复制为 web.config。

以下 web.config 示例依赖于某个虚拟环境代理脚本下（下一节中介绍）。它们与上面的示例 `app.py` 中所用的 WSGI 处理程序配合使用。

Python 2.7 的示例 `web.config`：

    <?xml version="1.0"?>
    <configuration>
      <appSettings>
        <add key="WSGI_ALT_VIRTUALENV_HANDLER" value="app.wsgi_app" />
        <add key="WSGI_ALT_VIRTUALENV_ACTIVATE_THIS"
             value="D:\home\site\wwwroot\env\Scripts\activate_this.py" />
        <add key="WSGI_HANDLER"
             value="ptvs_virtualenv_proxy.get_virtualenv_handler()" />
        <add key="PYTHONPATH" value="D:\home\site\wwwroot" />
      </appSettings>
      <system.web>
        <compilation debug="true" targetFramework="4.0" />
      </system.web>
      <system.webServer>
        <modules runAllManagedModulesForAllRequests="true" />
        <handlers>
          <remove name="Python27_via_FastCGI" />
          <remove name="Python34_via_FastCGI" />
          <add name="Python FastCGI"
               path="handler.fcgi"
               verb="*"
               modules="FastCgiModule"
               scriptProcessor="D:\Python27\python.exe|D:\Python27\Scripts\wfastcgi.py"
               resourceType="Unspecified"
               requireAccess="Script" />
        </handlers>
        <rewrite>
          <rules>
            <rule name="Static Files" stopProcessing="true">
              <conditions>
                <add input="true" pattern="false" />
              </conditions>
            </rule>
            <rule name="Configure Python" stopProcessing="true">
              <match url="(.*)" ignoreCase="false" />
              <conditions>
                <add input="{REQUEST_URI}" pattern="^/static/.*" ignoreCase="true" negate="true" />
              </conditions>
              <action type="Rewrite"
                      url="handler.fcgi/{R:1}"
                      appendQueryString="true" />
            </rule>
          </rules>
        </rewrite>
      </system.webServer>
    </configuration>


Python 3.4 的示例 `web.config`：

    <?xml version="1.0"?>
    <configuration>
      <appSettings>
        <add key="WSGI_ALT_VIRTUALENV_HANDLER" value="app.wsgi_app" />
        <add key="WSGI_ALT_VIRTUALENV_ACTIVATE_THIS"
             value="D:\home\site\wwwroot\env\Scripts\python.exe" />
        <add key="WSGI_HANDLER"
             value="ptvs_virtualenv_proxy.get_venv_handler()" />
        <add key="PYTHONPATH" value="D:\home\site\wwwroot" />
      </appSettings>
      <system.web>
        <compilation debug="true" targetFramework="4.0" />
      </system.web>
      <system.webServer>
        <modules runAllManagedModulesForAllRequests="true" />
        <handlers>
          <remove name="Python27_via_FastCGI" />
          <remove name="Python34_via_FastCGI" />
          <add name="Python FastCGI"
               path="handler.fcgi"
               verb="*"
               modules="FastCgiModule"
               scriptProcessor="D:\Python34\python.exe|D:\Python34\Scripts\wfastcgi.py"
               resourceType="Unspecified"
               requireAccess="Script" />
        </handlers>
        <rewrite>
          <rules>
            <rule name="Static Files" stopProcessing="true">
              <conditions>
                <add input="true" pattern="false" />
              </conditions>
            </rule>
            <rule name="Configure Python" stopProcessing="true">
              <match url="(.*)" ignoreCase="false" />
              <conditions>
                <add input="{REQUEST_URI}" pattern="^/static/.*" ignoreCase="true" negate="true" />
              </conditions>
              <action type="Rewrite" url="handler.fcgi/{R:1}" appendQueryString="true" />
            </rule>
          </rules>
        </rewrite>
      </system.webServer>
    </configuration>


静态文件将由 Web 服务器直接处理，无需通过 Python 代码，从而可提高性能。

在上面的示例中，磁盘上的静态文件的位置应与 URL 中的位置匹配。也就是说，对 `http://pythonapp.chinacloudsites.cn/static/site.css` 的请求将为磁盘上 `\static\site.css` 处的文件服务。

`WSGI_ALT_VIRTUALENV_HANDLER` 是指定 WSGI 处理程序的位置。在上面的示例中，该位置为 `app.wsgi_app`，因为处理程序是根文件夹中的 `app.py` 中一个名为 `wsgi_app` 的函数。

可以自定义 `PYTHONPATH`，但是，如果通过在 requirements.txt 中指定所有依赖项将全部安装在虚拟环境中，则不需要对其更改。


## 虚拟环境代理

使用以下脚本可检索 WSGI 处理程序、激活虚拟环境以及记录错误。该脚本用于一般目的，无需修改即可使用。

`ptvs_virtualenv_proxy.py` 的内容：

     # ############################################################################
     #
     # Copyright (c) Microsoft Corporation. 
     #
     # This source code is subject to terms and conditions of the Apache License, Version 2.0. A 
     # copy of the license can be found in the License.html file at the root of this distribution. If 
     # you cannot locate the Apache License, Version 2.0, please send an email to 
     # vspython@microsoft.com. By using this source code in any fashion, you are agreeing to be bound 
     # by the terms of the Apache License, Version 2.0.
     #
     # You must not remove this notice, or any other, from this software.
     #
     # ###########################################################################

    import datetime
    import os
    import sys
    import traceback

    if sys.version_info[0] == 3:
        def to_str(value):
            return value.decode(sys.getfilesystemencoding())

        def execfile(path, global_dict):
            """Execute a file"""
            with open(path, 'r') as f:
                code = f.read()
            code = code.replace('\r\n', '\n') + '\n'
            exec(code, global_dict)
    else:
        def to_str(value):
            return value.encode(sys.getfilesystemencoding())

    def log(txt):
        """Logs fatal errors to a log file if WSGI_LOG env var is defined"""
        log_file = os.environ.get('WSGI_LOG')
        if log_file:
            f = open(log_file, 'a+')
            try:
                f.write('%s: %s' % (datetime.datetime.now(), txt))
            finally:
                f.close()

    ptvsd_secret = os.getenv('WSGI_PTVSD_SECRET')
    if ptvsd_secret:
        log('Enabling ptvsd ...\n')
        try:
            import ptvsd
            try:
                ptvsd.enable_attach(ptvsd_secret)
                log('ptvsd enabled.\n')
            except: 
                log('ptvsd.enable_attach failed\n')
        except ImportError:
            log('error importing ptvsd.\n');

    def get_wsgi_handler(handler_name):
        if not handler_name:
            raise Exception('WSGI_ALT_VIRTUALENV_HANDLER env var must be set')
    
        if not isinstance(handler_name, str):
            handler_name = to_str(handler_name)
    
        module_name, _, callable_name = handler_name.rpartition('.')
        should_call = callable_name.endswith('()')
        callable_name = callable_name[:-2] if should_call else callable_name
        name_list = [(callable_name, should_call)]
        handler = None
        last_tb = ''

        while module_name:
            try:
                handler = __import__(module_name, fromlist=[name_list[0][0]])
                last_tb = ''
                for name, should_call in name_list:
                    handler = getattr(handler, name)
                    if should_call:
                        handler = handler()
                break
            except ImportError:
                module_name, _, callable_name = module_name.rpartition('.')
                should_call = callable_name.endswith('()')
                callable_name = callable_name[:-2] if should_call else callable_name
                name_list.insert(0, (callable_name, should_call))
                handler = None
                last_tb = ': ' + traceback.format_exc()
    
        if handler is None:
            raise ValueError('"%s" could not be imported%s' % (handler_name, last_tb))
    
        return handler

    activate_this = os.getenv('WSGI_ALT_VIRTUALENV_ACTIVATE_THIS')
    if not activate_this:
        raise Exception('WSGI_ALT_VIRTUALENV_ACTIVATE_THIS is not set')

    def get_virtualenv_handler():
        log('Activating virtualenv with %s\n' % activate_this)
        execfile(activate_this, dict(__file__=activate_this))

        log('Getting handler %s\n' % os.getenv('WSGI_ALT_VIRTUALENV_HANDLER'))
        handler = get_wsgi_handler(os.getenv('WSGI_ALT_VIRTUALENV_HANDLER'))
        log('Got handler: %r\n' % handler)
        return handler

    def get_venv_handler():
        log('Activating venv with executable at %s\n' % activate_this)
        import site
        sys.executable = activate_this
        old_sys_path, sys.path = sys.path, []
    
        site.main()
    
        sys.path.insert(0, '')
        for item in old_sys_path:
            if item not in sys.path:
                sys.path.append(item)

        log('Getting handler %s\n' % os.getenv('WSGI_ALT_VIRTUALENV_HANDLER'))
        handler = get_wsgi_handler(os.getenv('WSGI_ALT_VIRTUALENV_HANDLER'))
        log('Got handler: %r\n' % handler)
        return handler


## 自定义 Git 部署

[AZURE.INCLUDE [web-sites-python-customizing-runtime](../includes/web-sites-python-customizing-deployment.md)]


## 故障排除 - 软件包安装

[AZURE.INCLUDE [web-sites-python-troubleshooting-package-installation](../includes/web-sites-python-troubleshooting-package-installation.md)]


## 故障排除 - 虚拟环境

[AZURE.INCLUDE [web-sites-python-troubleshooting-virtual-environment](../includes/web-sites-python-troubleshooting-virtual-environment.md)]

## 后续步骤

有关详细信息，请参阅 [Python 开发人员中心](/develop/python/)。

<!---HONumber=Mooncake_0215_2016-->