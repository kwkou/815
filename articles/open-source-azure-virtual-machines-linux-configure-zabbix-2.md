<properties
	pageTitle="使用 Zabbix 监控 Nginx"
	description="本文介绍在 Azure Linux 虚拟机上使用 Zabbix 监控 Nginx"
	services="open-source"
	documentationCenter=""
	authors=""
	manager=""
	editor=""/>

<tags
	ms.service="open-source-website"
	ms.date=""
	wacn.date="05/09/2017"/>
 
#使用 Zabbix 监控 Nginx

1.	Nginx server 必须安装 zabbix agent 软件包，以及打开10050,10051和它的服务端口，比如80. 然后启动 zabbix agent 进程，添加到监控列表。详细操作流程请参照 [zabbix agent 安装指南](/documentation/articles/open-source-azure-virtual-machines-linux-configure-zabbix-1#install-zabbix-agent)。

2.	查看 nginx server 是否安装了 http_stub_status 模块. 使用 nginx -V 命令检查. 如果没有，重新编译。下面是一个简单示例 (CentOS 7 为例)

3.	连接到 nginx server , 执行 `$ sudo yum install zlib zlib-devel pcre pcre-devel openssl openssl-devel -y`

4.	下载，安装 nginx

        $ sudo wget http://nginx.org/download/nginx-1.9.8.tar.gz
        $sudo tar xzvf nginx-1.9.8.tar.gz
        $cd nginx-1.9.8
        $sudo ./configure --prefix=/opt/nginx --with-http_stub_status_module --with-http_ssl_module --with-threads
        $sudo make
        $sudo make install
    
5.	编辑 /opt/nginx/conf/nginx.conf, 添加如下红色部分
    
        server {
            listen       80;
            server_name  localhost;
    
            #charset koi8-r;
    
            #access_log  logs/host.access.log  main;
    
            location / {
                root   html;
                index  index.html index.htm;
            }
            location /nginx_status {
                stub_status on;	
            }
            #error_page  404              /404.html;
    
            # redirect server error pages to the static page /50x.html
            #
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
        }
6.	启动 nginx 进程

        $sudo /opt/nginx/sbin/nginx
    
7.	检查 nginx 连接状态. 打开 http://nginx server ip/nginx_status

    ![](./media/open-source-azure-virtual-machines-linux-configure-zabbix-2/31.png)
 
8.	Zabbix agent 设置。创建目录

        $ sudo mkdir -p /usr/local/zabbix/scripts

9.	编辑 /usr/local/zabbix/etc/zabbix_agentd.conf, 添加如下 

        #nginx
        UserParameter=nginx.accepts,/usr/local/zabbix/scripts/nginx_status.sh accepts
        UserParameter=nginx.handled,/usr/local/zabbix/scripts/nginx_status.sh handled
        UserParameter=nginx.requests,/usr/local/zabbix/scripts/nginx_status.sh requests
        UserParameter=nginx.connections.active,/usr/local/zabbix/scripts/nginx_status.sh active
        UserParameter=nginx.connections.reading,/usr/local/zabbix/scripts/nginx_status.sh reading
        UserParameter=nginx.connections.writing,/usr/local/zabbix/scripts/nginx_status.sh writing
        UserParameter=nginx.connections.waiting,/usr/local/zabbix/scripts/nginx_status.sh waiting

10.	编辑 /usr/local/zabbix/scripts/nginx_status.sh, 内容如下

        #!/bin/bash
        
        # Set Variables
        BKUP_DATE=`/bin/date +%Y%m%d`
        LOG="/var/log/nginx_status.log"
        #HOST=`/sbin/ifconfig eth0 | sed -n '/inet /{s/.*addr://;s/ .*//;p}'`
        HOST=`/sbin/ifconfig eth0 |sed -n '/inet /{s/.*inet \([0-9.]\+\).*/\1/p}'`
        #HOST=`curl ifconfig.me 2>/dev/null`
        PORT="80"
        #PORT="443"
        
        # Functions to return nginx stats
        
        function active {
                /usr/bin/curl -k "http://$HOST:$PORT/nginx_status" 2>/dev/null| grep 'Active' | awk '{print $NF}' 
                } 
        
        function reading {
                /usr/bin/curl -k "http://$HOST:$PORT/nginx_status" 2>/dev/null| grep 'Reading' | awk '{print $2}' 
                } 
        
        function writing {
                /usr/bin/curl -k "http://$HOST:$PORT/nginx_status" 2>/dev/null| grep 'Writing' | awk '{print $4}' 
                } 
        
        function waiting {
                /usr/bin/curl -k "http://$HOST:$PORT/nginx_status" 2>/dev/null| grep 'Waiting' | awk '{print $6}' 
                } 
        
        function accepts {
                /usr/bin/curl -k "http://$HOST:$PORT/nginx_status" 2>/dev/null| awk NR==3 | awk '{print $1}'
                } 
        
        function handled {
                /usr/bin/curl -k "http://$HOST:$PORT/nginx_status" 2>/dev/null| awk NR==3 | awk '{print $2}'
                } 
        
        function requests {
                /usr/bin/curl -k "http://$HOST:$PORT/nginx_status" 2>/dev/null| awk NR==3 | awk '{print $3}'
                }
        
        # Run the requested function
        $1

11.	设置执行权限

        $sudo chmod o+x /usr/local/zabbix/scripts/nginx_status.sh

12.	重启 zabbix agent 进程

        $sudo /etc/init.d/zabbix_agentd restart
        
13.	可以去到 [zabbix 模板官网](https://www.zabbix.org/wiki/Zabbix_Templates#Official_templates)下载相应的 nginx 模板并依据相关的指导完成 nginx 模板的导入和设置。步骤14-19是用从 zabbix community 社区找到的 nginx 模板所做的相关测试。模板放置在[链接](https://github.com/joey100/ZabbixTemplates)。下载此 nginx 模板 并关联到 nginx server。（注：此模板非官方提供，由社区贡献，如有顾虑，建议从官网下载 nginx 模板。）

14.	点击“Configuration” -- > “Templates” -- > “Import”
 
    ![](./media/open-source-azure-virtual-machines-linux-configure-zabbix-2/32.png)

15.	点击 “Browse” -- >选择模板文件, 这里我们选择下载下来的 “nginx_status 2.0.xml”, 点击 “Import”

    ![](./media/open-source-azure-virtual-machines-linux-configure-zabbix-2/33.png)
 
16.	点击 ‘Configuration’ -- > ‘Hosts’ -- > 选择 nginx server
 
    ![](./media/open-source-azure-virtual-machines-linux-configure-zabbix-2/34.png)

17.	点击 “Templates” -- > “Select” -- > 选择 “Nginx Status” 模板 , 点击“Select”, 然后点击 “Add” -- > “Save”

    ![](./media/open-source-azure-virtual-machines-linux-configure-zabbix-2/35.png)
 
18.	然后我们发现 nginx server 的 nginx 状态已经被监控。去到 “Monitoring” -- > “Graphs” -- > 选择 nginx server, 选择“Nginx Socket Status” 图, 会看到类似下图

    ![](./media/open-source-azure-virtual-machines-linux-configure-zabbix-2/36.png)
 
19.	您也可以选择 “Nginx Clients Status” 图。

    ![](./media/open-source-azure-virtual-machines-linux-configure-zabbix-2/37.png)
 


























