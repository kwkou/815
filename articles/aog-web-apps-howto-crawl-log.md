<properties
    pageTitle="如何在 Web 应用实例上住抓取网络日志"
    description="如何在 Web 应用实例上住抓取网络日志"
    service=""
    resource="webapps"
    authors="Zhang Yannan"
    displayOrder=""
    selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Web Apps, PowerShell, Log"
    cloudEnvironments="MoonCake" />
<tags
    ms.service="app-service-web-aog"
    ms.date=""
    wacn.date="04/29/2017" />

# 如何在 Web 应用实例上住抓取网络日志

用户部署在 Azure 上的 Web 应用，根据业务需求可能会调用第三方服务，当调用第三方服务出现问题时，如果开发人员定位到是网络问题，并需要抓取网络包进行分析时，可以使用下面的步骤：

1. 安装 ArmClient 工具

    1. 以管理员权限打开 PowerShell

    2. 运行如下命令安装 chocolatey

            PS C:\WINDOWS\system32> iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))


2. 通过 chocolatey 安装 armclient 工具

        PS C:\WINDOWS\system32> choco install armclient


3. 使用 Azure 账号登陆 ARMClient

        PS C:\WINDOWS\system32> armclient.exe login Mooncake

4. 运行下面的命令进行网络日志的抓取（保证您的网站在运行状态下）

        armclient.exe POST "/subscriptions/<sub>/resourceGroups/<RG>/providers/Microsoft.Web/sites/<site>/networkTrace/start?duration=<seconds>&api-version=2015-06-01"

    [AZURE.NOTE] 对 SubID、资源组名称 RG、网站名称 site、duration(最大设置为 300，目前只允许抓取最长 5 分钟的网络包) 进行替换，这些信息可以在新 portal 中网站预览页面看到。

5. 抓包完成以后，日志会出现在 \home\logfiles\networktrace 目录下。

