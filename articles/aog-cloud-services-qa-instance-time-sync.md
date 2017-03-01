<properties
    pageTitle="云服务实例时间同步问题"
    description="云服务在运行较长时间后不同实例时间差异较大"
    service=""
    resource="cloudservices"
    authors=""
    displayOrder=""
    selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Cloud Services, TimeSync"
    cloudEnvironments="MoonCake" />
<tags
    ms.service="cloud-services-aog"
    ms.date=""
    wacn.date="02/21/2017" />

# 云服务实例时间同步问题

## **问题描述**

云服务在运行较长时间后不同实例时间差异较大。

## **问题现象**

因为云服务后台启动的 windows 虚拟机默认同步时间的周期为 7 天，相对较长，当不同实例之间时间同步要求较高时往往无法满足需求。

## **解决方法**

1. 登录到云服务实例，修改时间同步周期参数：

    `[ HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W32Time\TimeProviders\NtpClient ]` 分支，并双击 `SpecialPollInterval` 键值，将对话框中的“ 基数栏 ”选择到“ 十进制 ”上，将此值的数值数据 `604800` 改为 `3600` 并选为“ 十进制（D）”键值意思是时间同步的间隔，单位为秒。原来 7 天就是 `7 * 24 * 3600 = 604800 ` 秒，1 小时就是 `60 * 60 = 3600 秒`（建议设置为 15 分钟或 1 个小时）。

2. 使用上面的方法在实例重启的情况下会恢复默认的设置，为避免该情况发生可以在部署云服务的 Startup task 中加入：`reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W32Time\TimeProviders\NtpClient" /v SpecialPollInterval /d 10800 /f`（修改 SpecialPollInterval 值为：`10800`）。
