
<properties 
	pageTitle="Kudu管理和诊断azure web应用" 
	description="Kudu管理和诊断azure web应用。" 
	services="app-service-web" 
	documentationCenter="" 
	authors=""
	manager="" 
	editor=""/>
<tags ms.service="app-service-web-aog" ms.date="" wacn.date="08/31/2016"/>
# Kudu管理和诊断azure web应用


[Azure Kudu](https://github.com/projectkudu/kudu/wiki/Process-Threads-list-and-minidump-gcdump-diagsession)是GitHub上的一个开源项目，Kudu站点 (也称为网站控制管理 SCM) 提供了一系列的在线工具，可以帮助用户查看web应用的设置，诊断web应用，以及安装web应用扩展。本文包含以下内容：

- [登陆Azure Kudu站点](#login)
- [查看web应用环境](#env)
- [调试控制台](#console)
- [场景举例](#scenario)
- [进程管理器](#process)
- [其它在线工具](#tool)
- [Web应用扩展](#ext)

## <a id="login"></a>登陆Azure Kudu站点
在浏览器中输入[https://{sitename}.scm.chinacloudsites.cn](#),在弹出的认证窗口中输入web应用的部署账号和密码（关于如何访问KUDU请阅读[这篇文章](https://github.com/projectkudu/kudu/wiki/Accessing-the-kudu-service)），验证成功后进入Kudu首页，首页包含Kudu当前的版本信息以及Kudu REST API接口信息，如下图:

![Kudu REST API接口信息](media/aog-web-app-diagnostics-kudu/kudu-rest-api.png)

## <a id="env"></a>查看web应用环境

进入Environment页面，用户可以查看web应用的运行环境，包括系统信息，应用程序设置，连接设置，环境变量，运行时路径，服务器HTTP headers配置以及服务器变量。

![Environment](media/aog-web-app-diagnostics-kudu/environment.png)

## <a id="console"></a>调试控制台

提供文件管理器和远程终端两大核心功能。

* 用户可以通过文件管理器查看网站文件，修改，删除和下载文件，并支持拖拽方式上传文件。

	![Debug Console](media/aog-web-app-diagnostics-kudu/debug-console.png)
* 提供远程终端Power Shell和 cmd窗口，用户可以远程在web应用实例上执行命令。

## <a id="scenario"></a>场景举例

当web应用出现性能问题，可以通过控制台 抓取web 应用的工作进程（w3wp.exe）的hang dump用于后续分析。

1. 下载ProcDump并上传到/site/Dianostices/Procdump 位置。

	![](media/aog-web-app-diagnostics-kudu/sence-1.png)

2. 进入Process explore页面，获取网站工作进程w3wp.exe 的进程ID

	![](media/aog-web-app-diagnostics-kudu/sence-2.png)

3. 进入/site/Dianostices/Procdump文件夹，运行procdump -ma -accepteula pid (pid为具体的进程ID)

	![](media/aog-web-app-diagnostics-kudu/sence-3.png)

4. 下载生成的dump文件，用windows调试工具如windbg进行后续分析。

## <a id="process"></a>进程管理器

用户可以通过进程管理器查看web应用实例上运行的进程的信息：

![Process Manage](media/aog-web-app-diagnostics-kudu/process-manage.png)

点击Properties按钮可以查看进程的具体信息
点击Start Profiling 开始监控进程CPU使用情况，监控结束后自动生成报表。

## <a id="tool"></a>其它在线工具
Tools选项卡下面提供了一些其他在线工具：

* Diagnostic Dump：

	单击此按钮可以下载web应用的重要日志文件。
* Log stream：
   
	输出实时日志到页面, 此功能也可以通过curl 命令行 启动。

	``` 
	curl -u {username} https://{sitename}.scm.azurewebsites.net/logstream
	```
	页面打印如下格式的日志信息：

	![](media/aog-web-app-diagnostics-kudu/log-stream.png)

	>**注意**: 如果通过Kudu站点启动Log stream，由于浏览器会缓存响应数据，用户需要等到日志填满缓冲区后，才可以看到实时日志输出到页面。 Log stream会持续12小时开启应用程序文件日志 (错误模式) ,用户可以在Azure portal上手动关闭。
    
* WebJobs dashboard： 

	进入WebJobs dashboard页面后，单击WebJobs的NAME链接，可以显示web Job历史运行情况。
	
	![](media/aog-web-app-diagnostics-kudu/webjobs-deshboard.png)

* Download deployment script：
  
	如果web应用是通过Git部署，用户点击此按钮可以下载部署脚本。

## <a id="ext"></a>Web应用扩展

Kudu站点提供了web应用扩展界面，用户可以查看已经安装的扩展或者安装新的扩展。

![Web extension](media/aog-web-app-diagnostics-kudu/web-extension.png)

