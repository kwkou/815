<properties
	pageTitle="访问 Azure Web 应用遇到“指定的 CGI 应用程序遇到错误，服务器终止了该进程”错误"
	description="访问 Azure Web 应用遇到“指定的 CGI 应用程序遇到错误，服务器终止了该进程”错误。"
	services="app-service-web"
	documentationCenter=""
	authors=""
	manager=""
	editor=""
	tags=""/>

<tags
	ms.service="app-service-web-aog"
	ms.date="10/27/2016"
	wacn.date="11/03/2016"/>


#访问 Azure Web 应用遇到“指定的 CGI 应用程序遇到错误，服务器终止了该进程”错误

###问题描述：

当 Java 服务器 (如 Tomcat ) 处理一个耗时比较长的请求，并在两分钟内没有响应 (比如上传大文件) ，IIS 服务器会抛出如下错误:

	“指定的CGI应用程序遇到错误，服务器终止了该进程“

###解决方法：

修改 web.config 中 HttpPlatformHandler 节点处配置的 requestTimeout 的数值，下面的例子是将requestTimeout 修改为10分钟:

	<httpPlatform processPath="%HOME%\site\wwwroot\bin\tomcat\bin\startup.bat" arguments="" requestTimeout="00:10:00"> 

>注意： requestTimeout 默认为2分钟，修改后的数值必须是60秒整数倍，否则将会返回 http502.3 错误。 

###其他资源：

requestTimeout 具体配置可以参考[这个链接](https://www.iis.net/learn/extensions/httpplatformhandler/httpplatformhandler-configuration-reference) 
