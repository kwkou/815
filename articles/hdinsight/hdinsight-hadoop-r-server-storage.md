<!-- not suitable for Mooncake -->

<properties
    pageTitle="适用于 HDInsight 上的 R Server 的 Azure 存储选项 | Azure"
    description="了解用户可用于 HDInsight 上的 R Server 的不同存储选项"
    services="HDInsight"
    documentationcenter=""
    author="jeffstokes72"
    manager="jhubbard"
    editor="cgronlun" />
<tags 
    ms.assetid="1cf30096-d3ca-45ea-b526-aa3954402f66"
    ms.service="HDInsight"
    ms.devlang="R"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="data-services"
    ms.date="01/09/2017"
    wacn.date="02/14/2017"
    ms.author="jeffstok" />

# Azure Storage options for R Server on HDInsight（适用于 HDInsight 上的 R Server 的 Azure 存储选项）
HDInsight 上的 Microsoft R Server 有权访问 Azure Blob，作为持久保存分析中的数据、代码、结果对象等的方式。

在 HDInsight 中创建 Hadoop 群集时，将指定 Azure 存储帐户或 Data Lake Store。该帐户中的一个特定 Blob 存储容器可为你所创建的群集保存文件系统（例如 Hadoop 分布式文件系统）。出于性能目的，HDInsight 群集将在与你指定的主存储帐户相同的数据中心内创建。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-use-blob-storage/ "将 Azure Blob 存储与 HDInsight 配合使用")。

## 使用多个 Azure Blob 存储帐户
如果需要，可以使用 HDI 群集访问多个 Azure 存储帐户或容器。为此，需要在创建群集时在 UI 中指定其他存储帐户，并按照下列步骤进行操作，以在 R 中使用它们。

1. 使用存储帐户名 **storage1** 和名为 **container1** 的默认容器创建 HDInsight 群集。
2. 指定名为 **storage2** 的另一个存储帐户。
3. 将 mycsv.csv 文件复制到 /share 目录，然后对该文件执行分析。

        hadoop fs -mkdir /share
        hadoop fs -copyFromLocal myscsv.scv /share  

4. 在 R 代码中，将名称节点设置为 **default**，并设置要处理的目录和文件。

        myNameNode <- "default"
        myPort <- 0

    数据的位置：

        bigDataDirRoot <- "/share"  

    定义 Spark 计算上下文：

        mySparkCluster <- RxSpark(consoleOutput=TRUE)

    设置计算上下文：

        rxSetComputeContext(mySparkCluster)

    定义 Hadoop 分布式文件系统 (HDFS)：

        hdfsFS <- RxHdfsFileSystem(hostName=myNameNode, port=myPort)

    指定 HDFS 中要分析的输入文件：

        inputFile <-file.path(bigDataDirRoot,"mycsv.csv")

所有目录和文件引用都指向存储帐户 wasbs://container1@storage1.blob.core.chinacloudapi.cn。这是与 HDInsight 群集关联的**默认存储帐户**。

现在，假设要处理名为 mySpecial.csv 的文件，其所在位置为 **storage2** 的 **container2** 中的 /private 目录。

在 R 代码中，将名称节点引用指向 **storage2** 存储帐户。

````
myNameNode <- "wasbs://container2@storage2.blob.core.chinacloudapi.cn"
myPort <- 0
````

数据的位置：

````
bigDataDirRoot <- "/private"
````

定义 Spark 计算上下文：

````
mySparkCluster <- RxSpark(consoleOutput=TRUE, nameNode=myNameNode, port=myPort)
````

设置计算上下文：

````
rxSetComputeContext(mySparkCluster)
````

定义 HDFS 文件系统：

````
hdfsFS <- RxHdfsFileSystem(hostName=myNameNode, port=myPort)
````

指定 HDFS 中要分析的输入文件：

````
inputFile <-file.path(bigDataDirRoot,"mySpecial.csv")
````

所有目录和文件引用现在都指向存储帐户 wasbs://container2@storage2.blob.core.chinacloudapi.cn。这是你指定的**名称节点**。

请注意，必须在 **storage2** 上配置 /user/RevoShare/<SSH 用户名> 目录，如下所示：

````
hadoop fs -mkdir wasbs://container2@storage2.blob.core.chinacloudapi.cn/user
hadoop fs -mkdir wasbs://container2@storage2.blob.core.chinacloudapi.cn/user/RevoShare
hadoop fs -mkdir wasbs://container2@storage2.blob.core.chinacloudapi.cn/user/RevoShare/<RDP username>
````

## 使用 Azure Data Lake Store
若要以 HDInsight 帐户使用 Data Lake Store，必须允许群集访问你要使用的每个 Azure Data Lake Store。在 R 脚本中使用该存储的方式与使用辅助存储帐户的方式（如上一过程中所述）非常类似。

## 为群集添加 Azure Data Lake Store 访问权限
可以使用与 HDInsight 群集关联的 Azure Active Directory (Azure AD) 服务主体来访问 Data Lake Store。

### 添加服务主体
1\.创建 HDInsight 群集时，请在“数据源”选项卡中选择“群集 AAD 标识”。

2\.在“群集 AAD 标识”对话框中的“选择 AD 服务主体”下面，选择“新建”。

为服务主体命名并创建密码后，单击“管理 ADLS 访问”将该服务主体与 Data Lake Store 相关联。

还可以在执行群集创建过程中，打开 Data Lake Store 的 Azure 门户条目并转到“数据资源管理器”>“访问权限”>“添加”将群集访问权限添加到一个或多个 Data Lake Store。

有关将 HDI 群集访问权限添加到 Data Lake Store 的更多详细信息，请参阅 [Create an HDInsight cluster with Data Lake Store using Azure Portal](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-hdinsight-hadoop-use-portal#create-an-hdinsight-cluster-with-access-to-azure-data-lake-store)（使用 Azure 门户创建包含 Data Lake Store 的 HDInsight 群集）一文

## 在 R Server 中使用 Data Lake Store
获取 Data Lake Store 访问权限后，便可以在 HDInsight 上的 R Server 中使用 Data Lake Store，其使用方式与使用辅助 Azure 存储帐户类似。唯一的差别在于，前缀 **wasb://** 需更改为 **adl://**，如下所示：

````
# Point to the ADL store (e.g. ADLtest)
myNameNode <- "adl://rkadl1.azuredatalakestore.net"
myPort <- 0

# Location of the data (assumes a /share directory on the ADL account)
bigDataDirRoot <- "/share"  

# Define Spark compute context
mySparkCluster <- RxSpark(consoleOutput=TRUE, nameNode=myNameNode, port=myPort)

# Set compute context
rxSetComputeContext(mySparkCluster)

# Define HDFS file system
hdfsFS <- RxHdfsFileSystem(hostName=myNameNode, port=myPort)

# Specify the input file in HDFS to analyze
inputFile <-file.path(bigDataDirRoot,"AirlineDemoSmall.csv")

# Create factors for days of the week
colInfo <- list(DayOfWeek = list(type = "factor",
               levels = c("Monday", "Tuesday", "Wednesday", "Thursday",
                          "Friday", "Saturday", "Sunday")))

# Define the data source
airDS <- RxTextData(file = inputFile, missingValueString = "M",
                    colInfo  = colInfo, fileSystem = hdfsFS)

# Run a linear regression
model <- rxLinMod(ArrDelay~CRSDepTime+DayOfWeek, data = airDS)
````

以下命令用于结合 RevoShare 目录配置 Data Lake 存储帐户，并添加上述示例所述的示例 .csv 文件：

````
hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/user
hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/user/RevoShare
hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/user/RevoShare/<user>

hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/share

hadoop fs -copyFromLocal /usr/lib64/R Server-7.4.1/library/RevoScaleR/SampleData/AirlineDemoSmall.csv adl://rkadl1.azuredatalakestore.net/share

hadoop fs -ls adl://rkadl1.azuredatalakestore.net/share
````

## 在边缘节点上使用 Azure 文件
还有一个可在边缘节点上使用的便利数据存储选项，我们称之为 [Azure 文件](/documentation/articles/storage-how-to-use-files-linux/ "Azure 文件")。使用该选项可将 Azure 存储空间的文件共享装载到 Linux 文件系统。对比 HDFS，如果可以在边缘节点上使用本机文件系统，则存储数据文件、R 脚本以及随后可能需要的结果对象将更方便。

使用 Azure 文件的主要好处之一是，装有受支持 OS（例如 Windows 或 Linux）的系统都可以装载和使用文件共享。例如，你自己或者团队成员拥有的另一个 HDInsight 群集、Azure VM 甚至本地系统均可使用 Azure 文件。

## 后续步骤
现在，你已了解 Azure 存储选项，请使用以下链接来发现使用 HDInsight 上的 R Server 的其他方法。

* [Overview of R Server on HDInsight（HDInsight 上的 R Server 概述）](/documentation/articles/hdinsight-hadoop-r-server-overview/)
* [Get started with R server on Hadoop（Hadoop 上的 R Server 入门）](/documentation/articles/hdinsight-hadoop-r-server-get-started/)
* [将 RStudio Server 添加到 HDInsight（如果未在群集创建过程中添加）](/documentation/articles/hdinsight-hadoop-r-server-install-r-studio/)
* [Compute context options for R Server on HDInsight（适用于 HDInsight 上的 R Server 的计算上下文选项）](/documentation/articles/hdinsight-hadoop-r-server-compute-contexts/)

<!---HONumber=Mooncake_1205_2016-->