<properties
    pageTitle="上传超过 2M 的文件到 Azure PHP 网站失败"
    description="上传超过 2M 的文件到 Azure PHP 网站失败"
    service=""
    resource="webapps"
    authors="高手"
    displayOrder=""
    selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Web Apps, PHP, Upload File"
    cloudEnvironments="MoonCake" />
<tags
    ms.service="web-apps-aog"
    ms.date=""
    wacn.date="03/16/2017" />
# 上传超过 2M 的文件到 Azure PHP 网站失败

## **问题描述**

上传超过 2M 的文件到 Azure PHP 网站失败。

## **问题分析**

由于 PHP 本身默认上传文件的上限是 2M，所以当上传超过2M的文件时会报错。

## **解决方法**

根据以下步骤进行配置：

1. 在 site -> wwwroot 目录下增加一个 `.user.ini` 文件，设置如下：

        upload_max_filesize = 40M
        post_max_size = 40M

    >[AZURE.NOTE] 以上两个参数的值可以根据需求自行修改。

    为了检查配置是否成功也可以在 `.user.ini` 文件中开启 PHP log ,配置如下：

        display_errors = on
        log_errors = on 
        error_reporting = E_ALL
        error_log = D:\home\site\wwwroot\php.log
        file_uploads = on

2. 由于 IIS 默认请求体的大小为 28.6M，若上传的文件超过这个大小，还需要在 `web.config` 文件中修改 IIS 请求体的大小,具体修改可以参照下面的例子：

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

3. 做完上述的配置之后，建议在管理门户重启网站。

## **参考链接**

[Windows Azure Web Sites: File upload limit for PHP sites hosted on WAWS](https://blogs.msdn.microsoft.com/kaushal/2014/01/01/windows-azure-web-sites-file-upload-limit-for-php-sites-hosted-on-waws/)<br>
[Request Limits](https://www.iis.net/configreference/system.webserver/security/requestfiltering/requestlimits#001) 

