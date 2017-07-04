<properties
    pageTitle="JAVA 网站上传大文件报 500 错误"
    description="JAVA 网站上传大文件报 500 错误"
    service=""
    resource="webapps"
    authors="Zhang Yannan"
    displayOrder=""
    selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Web Apps, Java, IIS"
    cloudEnvironments="MoonCake" />
<tags
    ms.service="app-service-web-aog"
    ms.date=""
    wacn.date="04/29/2017" />

# JAVA 网站上传大文件报 500 错误

## **问题描述**

当通过 JAVA 网站上传大文件，会报 500 错误。

## **问题分析**

因为 Azure 的 Java 网站都是基于 IIS 转发的，所以我们需要关注 IIS 的文件上传限制以及 requestTimeout。

## **解决方法**

1. 由于 IIS 默认请求体的大小为 28.6M，若上传的文件超过这个大小，还需要在 web.config 文件中修改 IIS 请求体的大小,具体修改可以参照下面的例子：

        <configuration>
        <system.webServer>
            <security>
                <requestFiltering>
                        <requestLimits maxAllowedContentLength="300000000"/>
                </requestFiltering>
            </security>
        </system.webServer>
        <system.web>
                        <httpRuntime maxRequestLength="300000000" appRequestQueueLimit="100" useFullyQualifiedRedirectUrl="true" executionTimeout="600"/>
        </system.web>
        </configuration>

2. 若上传大文件，建议配置 requestTimeout 为较长时间，默认是2分钟。

        <httpPlatform processPath="<Your_Project_Path>\bin\apache-tomcat-8.0.28\apache-tomcat-8.0.28\bin\startup.bat"     arguments="" stdoutLogEnabled="true" stdoutLogFile="<Your_stdout_Log_File_Path>"   requestTimeout="00:02:00">

3. 做完上述的配置之后，建议在管理门户重启网站。


## **参考链接**

- [Request Limits <requestLimits>](https://www.iis.net/configreference/system.webserver/security/requestfiltering/requestlimits#001)

- [HttpPlatformHandler Configuration Reference](http://www.iis.net/learn/extensions/httpplatformhandler/httpplatformhandler-configuration-reference)

