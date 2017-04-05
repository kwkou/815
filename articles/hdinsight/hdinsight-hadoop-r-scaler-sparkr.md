<properties
    pageTitle="将 ScaleR 和 SparkR 与 Azure HDInsight 混合使用 | Azure"
    description="将 ScaleR 和 SparkR 与 R Server 和 HDInsight 配合使用"
    services="hdinsight"
    documentationcenter=""
    author="jeffstokes72"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="5a76f897-02e8-4437-8f2b-4fb12225854a"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/08/2017"
    wacn.date="03/31/2017"
    ms.author="jeffstok" />  


# 在 HDInsight 中混合使用 ScaleR 和 SparkR

了解如何将用于在 Spark 中处理数据的 SparkR 与用于分析的 Microsoft R Server 混合使用。尽管这两个包在 Hadoop 的 Spark 执行引擎的顶层运行以利用分布式处理中的最新功能，但它们需要自身的 Spark 会话，因此无法共享内存中的数据。在将来的 R Server 版本中解决此项限制之前，解决方法是保留非重叠的 Spark 会话，并通过中间文件交换数据。可以看到，这两项要求的实现都相当直截了当。

为了演示，我们将使用最初由 Mario Inchiosa 和 Roni Burd 在 Strata 2016 研讨会中分享的一个示例，网络研讨会 [Building a Scalable Data Science Platform with R](http://event.on24.com/eventRegistration/console/EventConsoleNG.jsp?uimode=nextgeneration&eventid=1160288&sessionid=1&key=8F8FB9E2EB1AEE867287CD6757D5BD40&contenttype=A&eventuserid=305999&playerwidth=1000&playerheight=650&caller=previewLobby&text_language_id=en&format=fhaudio)（使用 R 构建可缩放的数据科研平台）中也提供了此示例。该示例使用 SparkR 来联接已知的航班抵达延误数据集以及出发地与目的地机场的天气数据，并使用这些数据作为 ScaleR 逻辑回归模型的输入来预测航班抵达延误时间。

演练的代码原本是针对 Azure 中 HDInsight 群集上的 Spark 中运行的 R Server 编写的，但在一个脚本中混合使用 SparkR 和 ScaleR 的思路同样适用于本地环境。下面假设读者对 R 和 R Server 的 [ScaleR](https://msdn.microsoft.com/microsoft-r/scaler-user-guide-introduction) 库有一个中等的了解。本文将会讲解 [SparkR](https://spark.apache.org/docs/2.1.0/sparkr.html) 的整个用法。

## 航班和天气数据集

AirOnTime08to12CSV airlines 公用数据集包含美国境内从 1987 年 10 月到 2012 年 12 月所有商务航班的抵达和出发详细信息。这是一个大型数据集：总共有大约 1.5 亿条记录，解压缩后略小于 4 GB。该数据集是从[美国政府存档](http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236)获取的，为了方便使用，它以 zip 文件 (AirOnTimeCSV.zip) 的形式提供，其中包含 [Revolution Analytics 数据集存储库](http://packages.revolutionanalytics.com/datasets/AirOnTime87to12/)中的 303 个单独的每月 CSV 文件

为了查看天气对航班延误的影响，我们还需要获取每个机场的天气数据。可以从[美国海洋与大气管理存储库](http://www.ncdc.noaa.gov/orders/qclcd/)下载原始格式的每月天气数据 zip 文件。为了演示此示例，我们已提取 2007 年 5 月到 2012 年 12 月的天气数据，并使用了 68 个月的 zip 天气数据文件中，每个文件内的每小时数据文件。每月 zip 文件还包含气象台 ID (WBAN)、关联的机场 (CallSign) 与机场时区与 UTC 的偏差 (TimeZone) 之间的映射 (YYYYMMstation.txt) - 联接航班延误数据时需要用到所有这些信息。

## 设置 Spark 环境

我们首先设置 Spark 环境，再准备天气数据，然后将这些数据与航班数据合并，最后建模。首先，我们指向包含输入数据目录的目录，创建 Spark 计算上下文，然后创建一个日志记录函数用于在控制台上记录参考信息：

    workDir        <- '~'  
    myNameNode     <- 'default' 
    myPort         <- 0
    inputDataDir   <- 'wasb://hdfs@myAzureAcccount.blob.core.chinacloudapi.cn'
    hdfsFS         <- RxHdfsFileSystem(hostName=myNameNode, port=myPort)

    # create a persistent Spark session to reduce startup times 
    #   (remember to stop it later!)
 
    sparkCC        <- RxSpark(consoleOutput=TRUE, nameNode=myNameNode, port=myPort, persistentRun=TRUE)

    # create working directories 

    rxHadoopMakeDir('/user')
    rxHadoopMakeDir('user/RevoShare')
    rxHadoopMakeDir('user/RevoShare/remoteuser')

    (dataDir <- '/share')
    rxHadoopMakeDir(dataDir)
    rxHadoopListFiles(dataDir) 

    setwd(workDir)
    getwd()

    # version of rxRoc that runs in a local CC 
    rxRoc <- function(...){
      rxSetComputeContext(RxLocalSeq())
      roc <- RevoScaleR::rxRoc(...)
      rxSetComputeContext(sparkCC)
      return(roc)
    }

    logmsg <- function(msg) { cat(format(Sys.time(), "%Y-%m-%d %H:%M:%S"),':',msg,'\n') } 
    t0 <- proc.time() 

    #..start 

    logmsg('Start') 
    (trackers <- system("mapred job -list-active-trackers", intern = TRUE))
    logmsg(paste('Number of task nodes=',length(trackers)))

接下来，将“Spark\_Home”添加到 R 包的搜索路径，以便可以使用 SparkR 并初始化 SparkR 会话。

    #..setup for use of SparkR  

    logmsg('Initialize SparkR') 

    .libPaths(c(file.path(Sys.getenv("SPARK_HOME"), "R", "lib"), .libPaths()))
    library(SparkR)

    sparkEnvir <- list(spark.executor.instances = '10',
                       spark.yarn.executor.memoryOverhead = '8000')

    sc <- sparkR.init(
      sparkEnvir = sparkEnvir,
      sparkPackages = "com.databricks:spark-csv_2.10:1.3.0"
    )

    sqlContext <- sparkRSQL.init(sc)

## 准备天气数据

为了准备天气数据，我们将这些数据放入建模时所需的列：“Visibility”、“DryBulbCelsius”、“DewPointCelsius”、“RelativeHumidity”、“WindSpeed”和“Altimeter”，添加与气象站关联的机场代码，然后将测量值从从当地时间转换为 UTC。

先创建一个文件，用于将气象站 (WBAN) 信息映射到机场代码。在天气数据随附的映射文件中，通过将天气数据文件中的 CallSign（例如 LAX）字段映射到航班数据中的 Origin 即可实现此目的，但是，我们手头正好有另一个映射文件可将 WBAN 映射到 AirportID（例如，与 LAX 对应的 12892），其中包含已保存到我们要使用的、名为“wban-to-airport-id-tz.CSV”的 CSV 文件的 TimeZone

| AirportID | WBAN | TimeZone
|-----------|------|---------
| 10685 | 54831 | -6
| 14871 | 24232 | -8
| .. | .. | ..

以下代码将读取每小时原始天气数据文件、将文件放入所需的列、合并气象站映射文件、将测量值的日期时间调整为 UTC，然后写出文件的新版本。

    # Look up AirportID and Timezone for WBAN (weather station ID) and adjust time

    adjustTime <- function(dataList)
    {
      dataset0 <- as.data.frame(dataList)
  
      dataset1 <- base::merge(dataset0, wbanToAirIDAndTZDF1, by = "WBAN")

      if(nrow(dataset1) == 0) {
        dataset1 <- data.frame(
          Visibility = numeric(0),
          DryBulbCelsius = numeric(0),
          DewPointCelsius = numeric(0),
          RelativeHumidity = numeric(0),
          WindSpeed = numeric(0),
          Altimeter = numeric(0),
          AdjustedYear = numeric(0),
          AdjustedMonth = numeric(0),
          AdjustedDay = integer(0),
          AdjustedHour = integer(0),
          AirportID = integer(0)
        )
    
        return(dataset1)
      }
  
      Year <- as.integer(substr(dataset1$Date, 1, 4))
      Month <- as.integer(substr(dataset1$Date, 5, 6))
      Day <- as.integer(substr(dataset1$Date, 7, 8))
  
      Time <- dataset1$Time
      Hour <- ceiling(Time/100)
  
      Timezone <- as.integer(dataset1$TimeZone)
  
      adjustdate = as.POSIXlt(sprintf("%4d-%02d-%02d %02d:00:00", Year, Month, Day, Hour), tz = "UTC") + Timezone * 3600

      AdjustedYear = as.POSIXlt(adjustdate)$year + 1900
      AdjustedMonth = as.POSIXlt(adjustdate)$mon + 1
      AdjustedDay   = as.POSIXlt(adjustdate)$mday
      AdjustedHour  = as.POSIXlt(adjustdate)$hour
  
      AirportID = dataset1$AirportID
      Weather = dataset1[,c("Visibility", "DryBulbCelsius", "DewPointCelsius", "RelativeHumidity", "WindSpeed", "Altimeter")]
  
      data.set = data.frame(cbind(AdjustedYear, AdjustedMonth, AdjustedDay, AdjustedHour, AirportID, Weather))
  
      return(data.set)
    }

    wbanToAirIDAndTZDF <- read.csv("wban-to-airport-id-tz.csv")

    colInfo <- list(
      WBAN = list(type="integer"),
      Date = list(type="character"),
      Time = list(type="integer"),
      Visibility = list(type="numeric"),
      DryBulbCelsius = list(type="numeric"),
      DewPointCelsius = list(type="numeric"),
      RelativeHumidity = list(type="numeric"),
      WindSpeed = list(type="numeric"),
      Altimeter = list(type="numeric")
    )

    weatherDF <- RxTextData(file.path(inputDataDir, "WeatherRaw"), colInfo = colInfo)

    weatherDF1 <- RxTextData(file.path(inputDataDir, "Weather"), colInfo = colInfo,
                    filesystem=hdfsFS)

    rxSetComputeContext("localpar")
    rxDataStep(weatherDF, outFile = weatherDF1, rowsPerRead = 50000, overwrite = T,
               transformFunc = adjustTime,
               transformObjects = list(wbanToAirIDAndTZDF1 = wbanToAirIDAndTZDF))

## 将航班和天气数据导入 Spark DataFrames

现在，我们使用 SparkR [read.df()](https://docs.databricks.com/spark/latest/sparkr/functions/read.df.html) 函数将天气和航班数据导入 Spark DataFrames。请注意，与其他许多 Spark 方法一样，此函数是惰式执行的，也就是说，它会排入执行队列，但只在需要时才会实际执行。

    airPath     <- file.path(inputDataDir, "AirOnTime08to12CSV")
    weatherPath <- file.path(inputDataDir, "Weather") # pre-processed weather data
    rxHadoopListFiles(airPath) 
    rxHadoopListFiles(weatherPath) 

    # create a SparkR DataFrame for the airline data

    logmsg('create a SparkR DataFrame for the airline data') 
    airDF <- read.df(sqlContext, airPath, source = "com.databricks.spark.csv", 
                     header = "true", inferSchema = "true")

    # Create a SparkR DataFrame for the weather data

    logmsg('create a SparkR DataFrame for the weather data') 
    weatherDF <- read.df(sqlContext, weatherPath, source = "com.databricks.spark.csv", 
                         header = "true", inferSchema = "true")

## 数据清理和和转换

接下来，我们针对导入的航班数据执行一些清理工作，以便重命名列，仅保留所需的变量，将计划的出发时间向下舍入为最接近的小时，以便与出发前的最新天气数据合并。

    logmsg('clean the airline data') 
    airDF <- rename(airDF,
                    ArrDel15 = airDF$ARR_DEL15,
                    Year = airDF$YEAR,
                    Month = airDF$MONTH,
                    DayofMonth = airDF$DAY_OF_MONTH,
                    DayOfWeek = airDF$DAY_OF_WEEK,
                    Carrier = airDF$UNIQUE_CARRIER,
                    OriginAirportID = airDF$ORIGIN_AIRPORT_ID,
                    DestAirportID = airDF$DEST_AIRPORT_ID,
                    CRSDepTime = airDF$CRS_DEP_TIME,
                    CRSArrTime =  airDF$CRS_ARR_TIME
    )

    # Select desired columns from the flight data. 
    varsToKeep <- c("ArrDel15", "Year", "Month", "DayofMonth", "DayOfWeek", "Carrier", "OriginAirportID", "DestAirportID", "CRSDepTime", "CRSArrTime")
    airDF <- select(airDF, varsToKeep)

    # Round down scheduled departure time to full hour.
    airDF$CRSDepTime <- floor(airDF$CRSDepTime / 100)

现在，我们针对天气数据执行类似的操作：

    # Average weather readings by hour
    logmsg('clean the weather data') 
    weatherDF <- agg(groupBy(weatherDF, "AdjustedYear", "AdjustedMonth", "AdjustedDay", "AdjustedHour", "AirportID"), Visibility="avg",
                      DryBulbCelsius="avg", DewPointCelsius="avg", RelativeHumidity="avg", WindSpeed="avg", Altimeter="avg"
                      )

    weatherDF <- rename(weatherDF,
                        Visibility = weatherDF$'avg(Visibility)',
                        DryBulbCelsius = weatherDF$'avg(DryBulbCelsius)',
                        DewPointCelsius = weatherDF$'avg(DewPointCelsius)',
                        RelativeHumidity = weatherDF$'avg(RelativeHumidity)',
                        WindSpeed = weatherDF$'avg(WindSpeed)',
                        Altimeter = weatherDF$'avg(Altimeter)'
    )

## 联接天气和航班数据

现在，我们使用 SparkR [join()](https://docs.databricks.com/spark/latest/sparkr/functions/join.html) 函数根据出发地 AirportID 和日期时间，针对航班和天气数据执行左外部联接。使用外部联接可以保留所有航班数据记录，即使没有匹配的天气数据。联接后，删除一些多余的列，并重命名保留的列，以删除联接时传入的 DataFrame 前缀。

    logmsg('Join airline data with weather at Origin Airport')
    joinedDF <- SparkR::join(
      airDF,
      weatherDF,
      airDF$OriginAirportID == weatherDF$AirportID &
        airDF$Year == weatherDF$AdjustedYear &
        airDF$Month == weatherDF$AdjustedMonth &
        airDF$DayofMonth == weatherDF$AdjustedDay &
        airDF$CRSDepTime == weatherDF$AdjustedHour,
      joinType = "left_outer"
    )

    # Remove redundant columns
    vars <- names(joinedDF)
    varsToDrop <- c('AdjustedYear', 'AdjustedMonth', 'AdjustedDay', 'AdjustedHour', 'AirportID')
    varsToKeep <- vars[!(vars %in% varsToDrop)]
    joinedDF1 <- select(joinedDF, varsToKeep)

    joinedDF2 <- rename(joinedDF1,
                        VisibilityOrigin = joinedDF1$Visibility,
                        DryBulbCelsiusOrigin = joinedDF1$DryBulbCelsius,
                        DewPointCelsiusOrigin = joinedDF1$DewPointCelsius,
                        RelativeHumidityOrigin = joinedDF1$RelativeHumidity,
                        WindSpeedOrigin = joinedDF1$WindSpeed,
                        AltimeterOrigin = joinedDF1$Altimeter
    )

以类似的方式根据目的地 AirportID 和日期时间联接天气和航班数据。

    logmsg('Join airline data with weather at Destination Airport')
    joinedDF3 <- SparkR::join(
      joinedDF2,
      weatherDF,
      airDF$DestAirportID == weatherDF$AirportID &
        airDF$Year == weatherDF$AdjustedYear &
        airDF$Month == weatherDF$AdjustedMonth &
        airDF$DayofMonth == weatherDF$AdjustedDay &
        airDF$CRSDepTime == weatherDF$AdjustedHour,
      joinType = "left_outer"
    )

    # Remove redundant columns
    vars <- names(joinedDF3)
    varsToDrop <- c('AdjustedYear', 'AdjustedMonth', 'AdjustedDay', 'AdjustedHour', 'AirportID')
    varsToKeep <- vars[!(vars %in% varsToDrop)]
    joinedDF4 <- select(joinedDF3, varsToKeep)

    joinedDF5 <- rename(joinedDF4,
                        VisibilityDest = joinedDF4$Visibility,
                        DryBulbCelsiusDest = joinedDF4$DryBulbCelsius,
                        DewPointCelsiusDest = joinedDF4$DewPointCelsius,
                        RelativeHumidityDest = joinedDF4$RelativeHumidity,
                        WindSpeedDest = joinedDF4$WindSpeed,
                        AltimeterDest = joinedDF4$Altimeter
                        )

## 将结果保存到 CSV 以便与 ScaleR 交换

联接操作到此完成，因此 SparkR 中的操作也到此完成。将数据从最终的 Spark DataFrame“joinedDF5”保存到 CSV 以便输入到 ScaleR，然后关闭 SparkR 会话。需要明确告知 SparkR 要在 80 个不同的分区中保存生成的 CSV，以便在 ScaleR 处理过程中实现足够的并行度。

    logmsg('output the joined data from Spark to CSV') 
    joinedDF5 <- repartition(joinedDF5, 80) # write.df below will produce this many CSVs

    # write result to directory of CSVs
    write.df(joinedDF5, file.path(dataDir, "joined5Csv"), "com.databricks.spark.csv", "overwrite", header = "true")

    # We can shut down the SparkR Spark context now
    sparkR.stop()

    # remove non-data files
    rxHadoopRemove(file.path(dataDir, "joined5Csv/_SUCCESS"))

## 导入到 XDF 供 ScaleR 使用

可以通过 ScaleR 文本数据源按原样使用已联接航班和天气数据的 CSV 文件来建模，但此处我们要将该文件导入 XDF，因为在针对数据集运行多个操作时，XDF 的效率更高。

    logmsg('Import the CSV to compressed, binary XDF format') 

    # set the Spark compute context for R Server 
    rxSetComputeContext(sparkCC)
    rxGetComputeContext()

    colInfo <- list(
      ArrDel15 = list(type="numeric"),
      Year = list(type="factor"),
      Month = list(type="factor"),
      DayofMonth = list(type="factor"),
      DayOfWeek = list(type="factor"),
      Carrier = list(type="factor"),
      OriginAirportID = list(type="factor"),
      DestAirportID = list(type="factor"),
      RelativeHumidityOrigin = list(type="numeric"),
      AltimeterOrigin = list(type="numeric"),
      DryBulbCelsiusOrigin = list(type="numeric"),
      WindSpeedOrigin = list(type="numeric"),
      VisibilityOrigin = list(type="numeric"),
      DewPointCelsiusOrigin = list(type="numeric"),
      RelativeHumidityDest = list(type="numeric"),
      AltimeterDest = list(type="numeric"),
      DryBulbCelsiusDest = list(type="numeric"),
      WindSpeedDest = list(type="numeric"),
      VisibilityDest = list(type="numeric"),
      DewPointCelsiusDest = list(type="numeric")
    )

    joinedDF5Txt <- RxTextData(file.path(dataDir, "joined5Csv"),
                               colInfo = colInfo, fileSystem = hdfsFS)
    rxGetInfo(joinedDF5Txt) 

    destData <- RxXdfData(file.path(dataDir, "joined5XDF"), fileSystem = hdfsFS)

    rxImport(inData = joinedDF5Txt, destData, overwrite = TRUE)

    rxGetInfo(destData, getVarInfo = T)

    # File name: /user/RevoShare/dev/delayDataLarge/joined5XDF 
    # Number of composite data files: 80 
    # Number of observations: 148619655 
    # Number of variables: 22 
    # Number of blocks: 320 
    # Compression type: zlib 
    # Variable information: 
    #   Var 1: ArrDel15, Type: numeric, Low/High: (0.0000, 1.0000)
    # Var 2: Year
    # 26 factor levels: 1987 1988 1989 1990 1991 ... 2008 2009 2010 2011 2012
    # Var 3: Month
    # 12 factor levels: 10 11 12 1 2 ... 5 6 7 8 9
    # Var 4: DayofMonth
    # 31 factor levels: 1 3 4 5 7 ... 29 30 2 18 31
    # Var 5: DayOfWeek
    # 7 factor levels: 4 6 7 1 3 2 5
    # Var 6: Carrier
    # 30 factor levels: PI UA US AA DL ... HA F9 YV 9E VX
    # Var 7: OriginAirportID
    # 374 factor levels: 15249 12264 11042 15412 13930 ... 13341 10559 14314 11711 10558
    # Var 8: DestAirportID
    # 378 factor levels: 13303 14492 10721 11057 13198 ... 14802 11711 11931 12899 10559
    # Var 9: CRSDepTime, Type: integer, Low/High: (0, 24)
    # Var 10: CRSArrTime, Type: integer, Low/High: (0, 2400)
    # Var 11: RelativeHumidityOrigin, Type: numeric, Low/High: (0.0000, 100.0000)
    # Var 12: AltimeterOrigin, Type: numeric, Low/High: (28.1700, 31.1600)
    # Var 13: DryBulbCelsiusOrigin, Type: numeric, Low/High: (-46.1000, 47.8000)
    # Var 14: WindSpeedOrigin, Type: numeric, Low/High: (0.0000, 81.0000)
    # Var 15: VisibilityOrigin, Type: numeric, Low/High: (0.0000, 90.0000)
    # Var 16: DewPointCelsiusOrigin, Type: numeric, Low/High: (-41.7000, 29.0000)
    # Var 17: RelativeHumidityDest, Type: numeric, Low/High: (0.0000, 100.0000)
    # Var 18: AltimeterDest, Type: numeric, Low/High: (28.1700, 31.1600)
    # Var 19: DryBulbCelsiusDest, Type: numeric, Low/High: (-46.1000, 53.9000)
    # Var 20: WindSpeedDest, Type: numeric, Low/High: (0.0000, 136.0000)
    # Var 21: VisibilityDest, Type: numeric, Low/High: (0.0000, 88.0000)
    # Var 22: DewPointCelsiusDest, Type: numeric, Low/High: (-43.0000, 29.0000)

    finalData <- RxXdfData(file.path(dataDir, "joined5XDF"), fileSystem = hdfsFS)

## 拆分数据以供训练和测试

使用 rxDataStep 拆分 2012 年的数据以供测试，保留剩余的数据以供训练。

    # split out the training data

    logmsg('split out training data as all data except year 2012')
    trainDS <- RxXdfData( file.path(dataDir, "finalDataTrain" ),fileSystem = hdfsFS)

    rxDataStep( inData = finalData, outFile = trainDS,
                rowSelection = ( Year != 2012 ), overwrite = T )

    # split out the testing data

    logmsg('split out the test data for year 2012') 
    testDS <- RxXdfData( file.path(dataDir, "finalDataTest" ), fileSystem = hdfsFS)

    rxDataStep( inData = finalData, outFile = testDS,
                rowSelection = ( Year == 2012 ), overwrite = T )

    rxGetInfo(trainDS)
    rxGetInfo(testDS)

## 训练并测试逻辑回归模型

好了，现在可以开始构建模型！ 为了查看天气数据对延误抵达时间的影响，我们将使用 ScaleR 的逻辑回归例程来建模，确定超过 15 分钟的抵达延误是否受到了相对日期、出发地和目的地机场以及这些机场的天气等因素的影响。

    logmsg('train a logistic regression model for Arrival Delay > 15 minutes') 
    formula <- as.formula(ArrDel15 ~ Year + Month + DayofMonth + DayOfWeek + Carrier +
                         OriginAirportID + DestAirportID + CRSDepTime + CRSArrTime + 
                         RelativeHumidityOrigin + AltimeterOrigin + DryBulbCelsiusOrigin +
                         WindSpeedOrigin + VisibilityOrigin + DewPointCelsiusOrigin + 
                         RelativeHumidityDest + AltimeterDest + DryBulbCelsiusDest +
                         WindSpeedDest + VisibilityDest + DewPointCelsiusDest
                       )

    # Use the scalable rxLogit() function but set max iterations to 3 for the purposes of 
    # this exercise 

    logitModel <- rxLogit(formula, data = trainDS, maxIterations = 3)

    base::summary(logitModel)

现在，让我们做出一些预测并查看 ROC 和 AUC，了解模型如何处理测试数据。

    # Predict over test data (Logistic Regression).

    logmsg('predict over the test data') 
    logitPredict <- RxXdfData(file.path(dataDir, "logitPredict"), fileSystem = hdfsFS)

    # Use the scalable rxPredict() function

    rxPredict(logitModel, data = testDS, outData = logitPredict,
              extraVarsToWrite = c("ArrDel15"), 
              type = 'response', overwrite = TRUE)

    # Calculate ROC and Area Under the Curve (AUC).

    logmsg('calculate the roc and auc') 
    logitRoc <- rxRoc("ArrDel15", "ArrDel15_Pred", logitPredict)
    logitAuc <- rxAuc(logitRoc)
    head(logitAuc)
    logitAuc

    plot(logitRoc)

## 在其他位置评分

我们还可以使用该模型在另一个平台上为数据评分：将数据保存到 RDS 文件，然后将该 RDS 传输并导入到 SQL Server R Services 等目标评分环境。执行此操作时，必须确保要评分的数据的系数级别与构建的模型上的级别匹配。为此，可以通过 ScaleR 的 rxCreateColInfo() 函数来提取并保存与建模数据关联的列信息，然后将该列信息应用到输入数据源用于预测。下面我们保存了测试数据集的几个行，接下来将从此示例中提取列信息并在预测脚本中使用。

    # save the model and a sample of the test dataset 

    logmsg('save serialized version of the model and a sample of the test data')
    rxSetComputeContext('localpar') 
    saveRDS(logitModel, file = "logitModel.rds")
    testDF <- head(testDS, 1000)  
    saveRDS(testDF    , file = "testDF.rds"    )
    list.files()

    # stop the spark engine 
    rxStopEngine(sparkCC) 

    list.files() 
    rxHadoopListFiles(file.path(inputDataDir,''))
    rxHadoopListFiles(dataDir)
    logmsg('Done.')
    elapsed <- (proc.time() - t0)[3]
    logmsg(paste('Elapsed time=',sprintf('%6.2f',elapsed),'(sec)\n\n'))

## 摘要

就这么简单！ 本文说明了如何在 Hadoop Spark 中将用于数据处理的 SparkR 与用于模型开发的 ScaleR 结合使用，但首先必须记得要保留单独的 Spark 会话，一次只运行一个会话，并通过 CSV 文件交换数据。尽管此过程现已相当简单直接，但在将来的 R Server 版本中还会得到大幅简化，因为到时 SparkR 和 ScaleR 可以共享 Spark 会话，因而也能共享 Spark DataFrames。

## 后续步骤和详细信息

- 有关使用 Spark 上的 R Server 的详细信息，请参阅 [MSDN 上的入门指南](https://msdn.microsoft.com/microsoft-r/scaler-spark-getting-started)

- 有关 R Server 的一般信息，请参阅 [Get started with R](https://msdn.microsoft.com/microsoft-r/microsoft-r-get-started-node)（R 入门）一文。

有关 SparkR 用法的详细信息，请参阅以下资源：

- [Apache SparkR 文档](https://spark.apache.org/docs/2.1.0/sparkr.html)

- Databricks 提供的 [SparkR Overview](https://docs.databricks.com/spark/latest/sparkr/overview.html)（SparkR 概述）

<!---HONumber=Mooncake_0327_2017-->