<properties
    pageTitle="SQL Server 如何实现大批量数据的自动导入和导出"
    description="SQL Server 如何实现大批量数据的自动导入和导出"
    service=""
    resource=""
    authors="Yu Tao"
    displayOrder=""
    selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="SQL Server, Import/Export, BCP, Bat"
    cloudEnvironments="MoonCake" />
<tags
    ms.service="sql-server-aog"
    ms.date=""
    wacn.date="05/16/2017" />

# SQL Server 如何实现大批量数据的自动导入和导出

通常情况下，SQL Server 需要导入或导出大批量数据时，我们会借助相关工具，但此类工具普遍存在速度慢、容易中断、报错或中途退出等情况。此处介绍一个快速、实用的命令 BCP。

BCP 命令可以直接在 DOS 提示符下运行，也可以写在批处理文件中。将 BCP 命令写入 Bat 脚本文件中，然后将该文件放入计划任务中，定时执行，从而实现自动导入与导出大批量数据的要求。

此处列举了 BCP 命令的导出与导入的例子：

## 导出

    C:\>bcp "select lc1,lc2,lc3 from t1" queryout f:\databasebackup\abc5.txt -SCIE-08\CIESQL -Usa –P[password] –d[database name] -w

    Starting copy...

    2 rows copied.
    Network packet size (bytes): 4096
    Clock Time (ms.) Total     : 1      Average : (2000.00 rows per sec.)

该命令将数据导出并转换为 Unicode.

## 导入

    C:\>bcp t1 in f:\databasebackup\abc5.txt -S[servername].database.chinacloudapi.cn -d[database name] –U[user name]@[server name] –P[password] -w

    Starting copy...

    2 rows copied.
    Network packet size (bytes): 4096
    Clock Time (ms.) Total     : 31     Average : (64.52 rows per sec.)

[AZURE.NOTE] -w 选择项目是必须要加的，它支持汉字导入和导出。

需要注意，BCP 命令是添加数据操作，不能进行数据的更新。



