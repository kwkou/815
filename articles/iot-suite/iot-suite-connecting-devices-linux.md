<properties
   pageTitle="在 Linux 上使用 C 连接设备 | Azure"
   description="介绍如何使用在 Linux 上运行的以 C 编写的应用程序将设备连接到 Azure IoT 套件预配置远程监视解决方案"
   services=""
   suite="iot-suite"
   documentationCenter="na"
   authors="dominicbetts"
   manager="timlt"
   editor=""/>

<tags
   ms.service="iot-suite"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="01/17/2017"
   wacn.date="03/28/2017"
   ms.author="dobett"/>


# 将设备连接到远程监视预配置解决方案 (Linux)

[AZURE.INCLUDE [iot-suite-selector-connecting](../../includes/iot-suite-selector-connecting.md)]

## 生成并运行示例 C 客户端 Linux
以下步骤说明如何创建一个客户端应用程序来与远程监视预配置解决方案通信。此应用程序以 C 编写，在 Ubuntu Linux 上生成和运行。
若要完成这些步骤，你需要一个运行 Ubuntu 版本 15.04 或 15.10 的设备。继续操作之前，请使用以下命令在 Ubuntu 设备上安装必备组件包：

    sudo apt-get install cmake gcc g++


## 在设备上安装客户端库
Azure IoT 中心客户端库以包的形式提供，你可以使用 **apt get** 命令在 Ubuntu 设备上安装该包。完成以下步骤，在 Ubuntu 计算机上安装包含 IoT 中心客户端库和标头文件的包：

1. 在外壳程序中，向计算机添加 AzureIoT 存储库：

    
		sudo add-apt-repository ppa:aziotsdklinux/ppa-azureiot
		sudo apt-get update
    

2. 安装 azure-iot-sdk-c-dev 包

    
		sudo apt-get install -y azure-iot-sdk-c-dev
    
## 安装 Parson JSON 分析器
IoT 中心客户端库使用 Parson JSON 分析器分析消息有效负载。在计算机上的适当文件夹中，使用以下命令克隆 Parson GitHub 存储库：


	git clone https://github.com/kgabis/parson.git


## 准备项目
在 Ubuntu 计算机上，创建名为 **remote\_monitoring** 的文件夹。在 **remote\_monitoring** 文件夹中：

- 创建四个文件：**main.c**、**remote\_monitoring.c**、**remote\_monitoring.h** 和 **CMakeLists.txt**。
- 创建名为 **parson** 的文件夹。

将 **parson.c** 和 **parson.h** 文件从 Parson 存储库的本地副本复制到 **remote\_monitoring/parson** 文件夹。

1. 在文本编辑器中，打开 **remote\_monitoring.c** 文件。添加以下 `#include` 语句：

    
            #include "iothubtransportmqtt.h"
	    #include "schemalib.h"
	    #include "iothub_client.h"
	    #include "serializer_devicetwin.h"
	    #include "schemaserializer.h"
	    #include "azure_c_shared_utility/threadapi.h"
	    #include "azure_c_shared_utility/platform.h"
    
	    #include "parson.h"


[AZURE.INCLUDE [iot-suite-connecting-code](../../includes/iot-suite-connecting-code.md)]

## 调用 remote\_monitoring\_run 函数
在文本编辑器中，打开 **remote\_monitoring.h** 文件。添加以下代码：


	void remote_monitoring_run(void);


在文本编辑器中，打开 **main.c** 文件。添加以下代码：


	#include "remote_monitoring.h"

	int main(void)
	{
	    remote_monitoring_run();

	    return 0;
	}


## 生成并运行应用程序
以下步骤描述如何使用 *CMake* 生成客户端应用程序。

1. 在文本编辑器中，打开 **remote\_monitoring** 文件夹中的 **CMakeLists.txt** 文件。

2. 添加以下指令，以定义如何生成客户端应用程序：
   

	    macro(compileAsC99)
	      if (CMAKE_VERSION VERSION_LESS "3.1")
	        if (CMAKE_C_COMPILER_ID STREQUAL "GNU")
	          set (CMAKE_C_FLAGS "--std=c99 ${CMAKE_C_FLAGS}")
	          set (CMAKE_CXX_FLAGS "--std=c++11 ${CMAKE_CXX_FLAGS}")
	        endif()
	      else()
	        set (CMAKE_C_STANDARD 99)
	        set (CMAKE_CXX_STANDARD 11)
	      endif()
	    endmacro(compileAsC99)

	    cmake_minimum_required(VERSION 2.8.11)
	    compileAsC99()

	    set(AZUREIOT_INC_FOLDER ".." "../parson" "/usr/include/azureiot" "/usr/include/azureiot/inc")

	    include_directories(${AZUREIOT_INC_FOLDER})

	    set(sample_application_c_files
	        ./parson/parson.c
	        ./remote_monitoring.c
	        ./main.c
	    )

	    set(sample_application_h_files
	        ./parson/parson.h
	        ./remote_monitoring.h
	    )

	    add_executable(sample_app ${sample_application_c_files} ${sample_application_h
	    _files})

	    target_link_libraries(sample_app
	        serializer
	        iothub_client
	        iothub_client_mqtt_transport
	        aziotsharedutil
	        umqtt
	        pthread
	        curl
	        ssl
	        crypto
	        m
	    )

1. 在 **remote\_monitoring** 文件夹中，创建一个文件夹用于存储 CMake 生成的 *make* 文件，然后运行 **cmake** 和 **make** 命令，如下所示：
   

	    mkdir cmake
	    cd cmake
	    cmake ../
	    make


4. 运行客户端应用程序，并将遥测数据发送到 IoT 中心：

    
        ./sample_app
    

[AZURE.INCLUDE [iot-suite-visualize-connecting](../../includes/iot-suite-visualize-connecting.md)]

<!---HONumber=Mooncake_0406_2017-->
<!--Update_Description:replace with chinese menu-->