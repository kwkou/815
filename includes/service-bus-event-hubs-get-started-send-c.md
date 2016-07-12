## 将消息发送到事件中心
在本部分中，我们将编写用于将事件发送到事件中心的 C 应用。我们将从 [Apache Qpid 项目](http://qpid.apache.org/)使用 Proton AMQP 库。这类似于从 C 中将服务总线队列和主题与 AMQP 配合使用，如[此处](https://code.msdn.microsoft.com/WindowsAzure/Using-Apache-Qpid-Proton-C-afd76504)所示。有关详细信息，请参阅 [Qpid Proton 文档](http://qpid.apache.org/proton/index.html)。

1. 从 [Qpid AMQP Messenger 页](http://qpid.apache.org/components/messenger/index.html)中，单击“安装 Qpid Proton”链接，并根据你的环境，按照说明操作。我们将采用一个 Linux 环境；例如，装有 Ubuntu 14.04 的 [Azure Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli/)。

2. 若要编译 Proton 库，请安装以下程序包：

	```
	sudo apt-get install build-essential cmake uuid-dev openssl libssl-dev
	```

3. 下载 [Qpid Proton 库](http://qpid.apache.org/proton/index.html)并提取它，例如：

	```
	wget http://archive.apache.org/dist/qpid/proton/0.7/qpid-proton-0.7.tar.gz
	tar xvfz qpid-proton-0.7.tar.gz
	```

4. 创建生成目录、编译和安装：

	```
	cd qpid-proton-0.7
	mkdir build
	cd build
	cmake -DCMAKE_INSTALL_PREFIX=/usr ..
	sudo make install
	```

5. 在工作目录中，创建一个包含以下内容的名为 **sender.c** 的新文件。请记得替换事件中心名称和命名空间名称（后者通常为 `{event hub name}-ns`）的值。还必须为之前创建的 **SendRule** 替换密钥的 URL 编码版本。可以在[此处](http://www.w3schools.com/tags/ref_urlencode.asp)对它进行 URL 编码。

	
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
    	
    	#define check(messenger)                                                     \
    	  {                                                                          \
    	    if(pn_messenger_errno(messenger))                                        \
    	    {                                                                        \
    	      printf("check\n");													 \
    	      die(__FILE__, __LINE__, pn_error_text(pn_messenger_error(messenger))); \
    	    }                                                                        \
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

	```
	gcc sender.c -o sender -lqpid-proton
	```

> [AZURE.NOTE]在此代码中，我们使用传出窗口 1 以强制尽快发出消息。通常，你的应用程序应尝试批处理消息，以提高吞吐量。请参阅 [Qpid AMQP Messenger 页](http://qpid.apache.org/components/messenger/index.html)，以详细了解如何在此环境及其他环境中以及从为其提供了绑定的平台（目前为 Perl、PHP、Python 和 Ruby）中使用 Qpid Proton 库。

<!---HONumber=Mooncake_0104_2016-->