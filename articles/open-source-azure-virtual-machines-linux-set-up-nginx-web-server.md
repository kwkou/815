<properties
   pageTitle="在 Azure 中自己搭建 Nginx Web 服务器 | Azure"
   description="本文介绍如何微软的公有云平台 Azure 中搭建 Nginx Web 服务器。"
   services="open-source"
   documentationCenter=""
   authors=""
   manager=""
   editor=""/>

<tags
   ms.service="open-source-website"  
   ms.date=""
   wacn.date="06/14/2016"/>


#在 Azure 中自己搭建 Nginx Web 服务器

Nginx 是一款轻量级的 Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，并在一个 BSD-like 协议下发行。这篇文章介绍如何微软的公有云平台 Azure 中搭建 Nginx Web 服务器。   

大多数 Linux 发行版以及 BSD 版本已经在他们的安装库中包含了 Nginx，因此您可以通过任何该版本常用的安装方法来安装 Nginx (比如在Debian 和 Ubuntu 上运行 apt-get，在 FreeBSD 上运行 Ports 等)。有时候这些软件包已经过时。如果您想要最新的功能以及修正，您可以从nginx.org 下载包或从源码开始编译。  

下面我们就拿 CentOS 来做些例子。  

##创建一台虚拟机
您可以在 Azure上 选择您喜欢的 Linux 发行版并创建一台虚拟机，当然 Azure 也允许您自带自己的 Linux 虚拟机。  
![创建 Linux 虚拟机][1]
 
##安装 Nginx
1.在 CentOS 上安装自带 NGINX 软件包  

1.1  首先您需要添加NGINX yum安装源。  

创建 /etc/yum.repos.d/nginx.repo 并黏贴以下配置信息。  

	[nginx]
	name=nginx repo
	baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
	gpgcheck=0
	enabled=1  

或者添加 CentOS 7 EPEL 库  

	sudo yum install epel-release  

如果您想了解如何安装其他 Linux 发行版自带的 Nginx 软件包，可参考https://www.nginx.com/resources/wiki/start/topics/tutorials/install/#。 

1.2 安装  

	yum install nginx  

1.3 运行  

	systemctl start nginx  

您也可以运行一下命令来查看 nginx 服务的状态  

	systemctl status nginx
 
![nginx 服务的状态][2]  

2.在 CentOS 上安装来自 nginx.org 的软件包  

2.1 下载软件包，比如 NGINX 1.9.5。  

下面这个例子先安装了 wget 工具，这是因为 Azure 发布的 CentOS 7.1 中并没有默认安装 wget。  

	yum install wget
	wget nginx.org/download/nginx-1.9.5.tar.gz
	tar –xvzf nginx-1.9.5.tar.gz
	cd nginx-1.9.5  

2.2 编译安装，关于更多的编译选项，可以运行 “./configure –help” 或者参考[官网](http://nginx.org/en/docs/configure.html)。  

下面这个例子先安装了 gcc 编译器，这是因为 Azure 发布的 CentOS 7.1 中并没有默认安装 gcc。另外您也需要安装 pcre，pcre-devel，zlib，zlib-devel 和 openssl，否则您可能遇到再编译过程中遇到错误。比如，如果您不注明 “—without-http_rewrite_module” 您将会遇到 ”error: the HTTP rewrite module requires the PCRE library”。  

	yum install gcc
	yum install pcre pcre-devel zlib zlib-devel 
	./configure 
	make
	make install

2.3 运行  

	/usr/sbin/nginx –c /etc/nginx/nginx.conf  

2.4 查看状态  

	ps –ef | grep nginx
 
![查看状态][3]  

##在 Azure 端中配置可以从公网访问虚拟机中部署的网站

Azure 虚拟机默认只开放对应 VIP 地址的有限端口，用于远程连接或管理。新建网站如果需要通过虚拟 IP 地址某端口访问，则需要在该虚拟机上添加此服务终端节点（endpoint）。此操作可以通过 Azure 管理门户网站或通过 PowerShell 进行。  
![添加服务终端节点][4]
 

##验证

现在您可以在浏览器中访问您刚刚搭建好的服务器的公网 IP 来进行验证。  
![验证][5]
 

##小建议

如果您是自己从源码编译安装 Ngnix，建议把它变成一个守护进程，这样更容易来管理该服务。  

关于如何创建 Nginx init 脚本，可参考 [Nginx init script wiki](https://www.nginx.com/resources/wiki/start/topics/examples/initscripts/)。  

下面的步骤针对 CentOS。  

	vi /etc/ini.d/nginx  

粘贴以下内容，保存。  

	#!/bin/sh
	#
	# nginx - this script starts and stops the nginx daemon
	#
	# chkconfig:   - 85 15
	# description:  NGINX is an HTTP(S) server, HTTP(S) reverse \
	#               proxy and IMAP/POP3 proxy server
	# processname: nginx
	# config:      /etc/nginx/nginx.conf
	# config:      /etc/sysconfig/nginx
	# pidfile:     /var/run/nginx.pid

	# Source function library.
	. /etc/rc.d/init.d/functions

	# Source networking configuration.
	. /etc/sysconfig/network

	# Check that networking is up.
	[ "$NETWORKING" = "no" ] && exit 0

	nginx="/usr/sbin/nginx"
	prog=$(basename $nginx)

	NGINX_CONF_FILE="/etc/nginx/nginx.conf"

	[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx

	lockfile=/var/lock/subsys/nginx

	make_dirs() {
   	# make required directories
	user=`$nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`

   	if [ -z "`grep $user /etc/passwd`" ]; then
       useradd -M -s /bin/nologin $user
   	fi
   	options=`$nginx -V 2>&1 | grep 'configure arguments:'`
   	for opt in $options; do
       if [ `echo $opt | grep '.*-temp-path'` ]; then
           value=`echo $opt | cut -d "=" -f 2`
           if [ ! -d "$value" ]; then
               # echo "creating" $value
               mkdir -p $value && chown -R $user $value
           fi
       fi
   	done
	}

	start() {
    	[ -x $nginx ] || exit 5
    	[ -f $NGINX_CONF_FILE ] || exit 6
    	make_dirs
    	echo -n $"Starting $prog: "
    	daemon $nginx -c $NGINX_CONF_FILE
    	retval=$?
    	echo
    	[ $retval -eq 0 ] && touch $lockfile
    	return $retval
	}

	stop() {
    	echo -n $"Stopping $prog: "
    	killproc $prog -QUIT
    	retval=$?
    	echo
    	[ $retval -eq 0 ] && rm -f $lockfile
    	return $retval
	}

	restart() {
    	configtest || return $?
    	stop
    	sleep 1
    	start
	}

	reload() {
    	configtest || return $?
    	echo -n $"Reloading $prog: "
    	killproc $nginx -HUP
    	RETVAL=$?
    	echo
	}

	force_reload() {
    	restart
	}

	configtest() {
  		$nginx -t -c $NGINX_CONF_FILE
	}

	rh_status() {
    	status $prog
	}

	rh_status_q() {
    	rh_status >/dev/null 2>&1
	}

	case "$1" in
    	start)
        	rh_status_q && exit 0
        	$1
        	;;
    	stop)
        	rh_status_q || exit 0
        	$1
        	;;
    	restart|configtest)
        	$1
        	;;
    	reload)
        	rh_status_q || exit 7
        	$1
        	;;
    	force-reload)
        	force_reload
        	;;
    	status)
        	rh_status
        	;;
    	condrestart|try-restart)
        	rh_status_q || exit 0
            ;;
    	*)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
	esac  

赋予守护进程运行权限。  

	Chmod +x /etc/init.d/nginx  

接下来您就可以启动/停止/查看 Nginx 服务了  

	/etc/init.d/nginx start
	/etc/init.d/nginx stop
	/etc/init.d/nginx status  

如果你使用定制的 CentOS 并开启了防火墙，请运行以下命令来允许 HTTP 和 HTTPS 的流量:  

	sudo firewall-cmd --permanent --zone=public --add-service=http 
	sudo firewall-cmd --permanent --zone=public --add-service=https
	sudo firewall-cmd --reload



<!-- image references -->  
[1]: ./media/open-source-azure-virtual-machines-linux-set-up-nginx-web-server/open-source-set-up-nginx-web-server-in-azure-1.png 
[2]: ./media/open-source-azure-virtual-machines-linux-set-up-nginx-web-server/open-source-set-up-nginx-web-server-in-azure-2.png 
[3]: ./media/open-source-azure-virtual-machines-linux-set-up-nginx-web-server/open-source-set-up-nginx-web-server-in-azure-3.png 
[4]: ./media/open-source-azure-virtual-machines-linux-set-up-nginx-web-server/open-source-set-up-nginx-web-server-in-azure-4.png 
[5]: ./media/open-source-azure-virtual-machines-linux-set-up-nginx-web-server/open-source-set-up-nginx-web-server-in-azure-5.png 

