<properties
    pageTitle="使用 C 将事件发送到 Azure 事件中心 | Azure"
    description="使用 C 将事件发送到 Azure 事件中心"
    services="event-hubs"
    documentationcenter=""
    author="jtaubensee"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="event-hubs"
    ms.workload="na"
    ms.tgt_pltfrm="c"
    ms.devlang="csharp"
    ms.topic="article"
    ms.date="01/30/2017"
    wacn.date="04/17/2017"
    ms.author="jotaub;sethm"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="50bcf473d406b335726801b787d82c991603c425"
    ms.lasthandoff="04/07/2017" />

# <a name="send-events-to-azure-event-hubs-using-c"></a>使用 C 将事件发送到 Azure 事件中心

## <a name="introduction"></a>介绍
事件中心是一个具备高度可伸缩性的引入系统，每秒可收入大量事件，从而使应用程序能够处理和分析连接的设备和应用程序所产生的海量数据。 数据采集到事件中心后，可以使用任何实时分析提供程序或存储群集来转换和存储数据。

有关详细信息，请参阅[事件中心概述][Event Hubs overview]。

在本教程中，你将学习如何使用用 C 编写的控制台应用程序将事件发送到事件中心。若要接收事件，请单击左侧目录中的相应接收语言。

若要完成本教程，需要满足以下条件：

* C 语言开发环境。 对于本教程，我们将假定 gcc 堆栈在使用 Ubuntu 14.04 的 Azure Linux 虚拟机上。
* Microsoft Visual Studio 或 Visual Studio Community Edition
* 有效的 Azure 帐户。 如果没有帐户，可以创建一个试用帐户，只需几分钟即可完成。 有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial/)。

## <a name="send-messages-to-event-hubs"></a>将消息发送到事件中心
在本部分中，我们将编写用于将事件发送到事件中心的 C 应用。 我们将从 [Apache Qpid 项目](http://qpid.apache.org/)使用 Proton AMQP 库。 这类似于从 C 中将服务总线队列和主题与 AMQP 配合使用，如 [此处](https://code.msdn.microsoft.com/Using-Apache-Qpid-Proton-C-afd76504)所示。 有关详细信息，请参阅 [Qpid Proton 文档](http://qpid.apache.org/proton/index.html)。

<!-- [Qpid AMQP Messenger 页] actually is (http://qpid.apache.org/components/index.html) -->
1. 从 [Qpid AMQP Messenger 页](http://qpid.apache.org/components/index.html)中，单击“安装 Qpid Proton”链接，并根据你的环境，按照说明操作。
2. 若要编译 Proton 库，请安装以下程序包：

        sudo apt-get install build-essential cmake uuid-dev openssl libssl-dev

3. 下载 [Qpid Proton 库](http://qpid.apache.org/proton/index.html)并提取它，例如：

        wget http://archive.apache.org/dist/qpid/proton/0.7/qpid-proton-0.7.tar.gz
        tar xvfz qpid-proton-0.7.tar.gz

4. 创建生成目录、编译和安装：

        cd qpid-proton-0.7
        mkdir build
        cd build
        cmake -DCMAKE_INSTALL_PREFIX=/usr ..
        sudo make install

5. 在工作目录中，创建一个包含以下内容的名为 **sender.c** 的新文件。 请记得替换事件中心名称和命名空间名称（后者通常为 `{event hub name}-ns`）的值。 还必须用密钥的 URL 编码版本替换之前创建的 **SendRule**。 可以在 [此处](http://www.w3schools.com/tags/ref_urlencode.asp)对它进行 URL 编码。

        #include "proton/message.h"
        #include "proton/messenger.h"
        
        #include <getopt.h>
        #include <proton/util.h>
        #include <sys/time.h>
        #include <stddef.h>
        #include <stdio.h>
        #include <string.h>
        #include <unistd.h>
        #include <stdlib.h>
        
        #define check(messenger)                                                     
          {                                                                          
            if(pn_messenger_errno(messenger))                                        
            {                                                                        
              printf("check\n");                                                     
              die(__FILE__, __LINE__, pn_error_text(pn_messenger_error(messenger))); 
            }                                                                        
          }  
        
        pn_timestamp_t time_now(void)
        {
          struct timeval now;
          if (gettimeofday(&now, NULL)) pn_fatal("gettimeofday failed\n");
          return ((pn_timestamp_t)now.tv_sec) * 1000 + (now.tv_usec / 1000);
        }  
        
        void die(const char *file, int line, const char *message)
        {
          printf("Dead\n");
          fprintf(stderr, "%s:%i: %s\n", file, line, message);
          exit(1);
        }
        
        int sendMessage(pn_messenger_t * messenger) {
            char * address = (char *) "amqps://SendRule:{Send Rule key}@{namespace name}.servicebus.chinacloudapi.cn/{event hub name}";
            char * msgtext = (char *) "Hello from C!";
            
            pn_message_t * message;
            pn_data_t * body;
            message = pn_message();
            
            pn_message_set_address(message, address);
            pn_message_set_content_type(message, (char*) "application/octect-stream");
            pn_message_set_inferred(message, true);
            
            body = pn_message_body(message);
            pn_data_put_binary(body, pn_bytes(strlen(msgtext), msgtext));
            
            pn_messenger_put(messenger, message);
            check(messenger);
            pn_messenger_send(messenger, 1);
            check(messenger);
            
            pn_message_free(message);
        }
        
        int main(int argc, char** argv) {
            printf("Press Ctrl-C to stop the sender process\n");
            
            pn_messenger_t *messenger = pn_messenger(NULL);
            pn_messenger_set_outgoing_window(messenger, 1);
            pn_messenger_start(messenger);
            
            while(true) {
                sendMessage(messenger);
                printf("Sent message\n");
                sleep(1);
            }
            
            // release messenger resources
            pn_messenger_stop(messenger);
            pn_messenger_free(messenger);
            
            return 0;
        }

6. 使用 **gcc** 编译该文件：

        gcc sender.c -o sender -lqpid-proton

    > [AZURE.NOTE]
    > 在此代码中，我们使用传出窗口 1 以强制尽快发出消息。 通常，你的应用程序应尝试批处理消息，以提高吞吐量。 请参阅 [Qpid AMQP Messenger 页](http://qpid.apache.org/components/index.html)，详细了解如何在此环境及其他环境中以及从为其提供了绑定的平台（目前为 Perl、PHP、Python 和 Ruby）中使用 Qpid Proton 库。

## <a name="next-steps"></a>后续步骤
访问以下链接可以了解有关事件中心的详细信息：

* [事件中心概述](/documentation/articles/event-hubs-what-is-event-hubs/)
* [创建事件中心](/documentation/articles/event-hubs-create/)
* [事件中心常见问题](/documentation/articles/event-hubs-faq/)

<!-- Images. -->

[21]: ./media/event-hubs-c-ephcs-getstarted/run-csharp-ephcs1.png
[24]: ./media/event-hubs-c-ephcs-getstarted/receive-eph-c.png

<!-- Links -->

[Azure Classic Management Portal]: https://manage.windowsazure.cn/
[Event Processor Host]: https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost
[Event Hubs overview]: /documentation/articles/event-hubs-overview/
[sample application that uses Event Hubs]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097
[Scale out Event Processing with Event Hubs]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3


<!--Update_Description:wording update;update reference link-->