###Druid 应用安装文档

**安装环境**

系统：CentOS 6.5

**vi 详解**  
[http://wenku.baidu.com/link?url=lwNWeoLEG0UnzoMhdw7X2Vek0sNkIqAaRVk0Uh7CkB51yAbXXD8ZarQuIZjuKfGRtEoRXBwn5Yq5zF74EcA7SHIVs84ocI57Vpnid7ruFQ7](http://wenku.baidu.com/link?url=lwNWeoLEG0UnzoMhdw7X2Vek0sNkIqAaRVk0Uh7CkB51yAbXXD8ZarQuIZjuKfGRtEoRXBwn5Yq5zF74EcA7SHIVs84ocI57Vpnid7ruFQ7)

**说明：** 
所有软件以及环境依据此文档为准，并且需要用 root 超级用户来完成，另外需要关闭防火墙。

**环境准备**

JDK： [http://www.oracle.com/technetwork/java/javase/downloads/index.html](http://www.oracle.com/technetwork/java/javase/downloads/index.html)  
Tomcat： [http://tomcat.apache.org/](http://tomcat.apache.org)

*安装 JDK*

1）解压 JDK

    tar xzvf jdk-8u5-linux-x64.gz
    
    mv jdk1.8.0_05/ /usr/local/java

2）配置环境变量
    
    vi ~/.bashrc
    
    export JAVA_HOME=/usr/local/java
    
    export JRE_HOME=/usr/local/java/jre
    
    export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
    
    export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
    
    source ~/.bashrc


执行以下命令

    update-alternatives --install /usr/bin/java java /usr/local/java/bin/java 300
       	
    update-alternatives --install /usr/bin/javac javac /usr/local/java/bin/javac 300
		
测试 Java	
		
    java –version

Tomcat 安装配置

    tar xzvf apache-tomcat-7.0.54.tar.gz
    
    mv apache-tomcat-7.0.54 /usr/local/tomcat

添加 Java 环境

    vi /usr/local/tomcat/bin/catalina.sh

添加在第一行

    export JAVA_HOME=/usr/local/java

启动 Tomcat

    sh /usr/local/tomcat/bin/startup.sh

访问： http://localhost:8080  
OK 没问题，Tomcat + JDK 完成。


*数据库准备*

使用 yum 安装 MySQL 数据库。

`yum -y install mysql-server mysql mysql-devel` 

命令将 mysql-server、 mysql、 mysql-devel 都安装好，当结果显示为 “Complete！” 即安装完毕。

进入 MySQL

    mysql –uroot –p
		
创建用户

    grant all on *.* to ‘myuser’@’%’ identified by ‘bjtest1234’;
    
    grant all on *.* to ‘myuser’@’localhost’ identified by ‘bjtest1234’;
    
    flush privileges;


*Druid 驱动*

Druid 驱动下载： http://central.maven.org/maven2/com/alibaba/druid

![](http://cn.msopentech.com/wp-content/uploads/druid01.png)
	 
将这4个驱动放入到 /usr/local/tomcat/lib 下面，必须重启tomcat服务

    /usr/local/tomcat/bin/shutdown.sh
    /usr/local/tomcat/bin/startup.sh


*数据库操作*

`mysql –uroot –p`（密码空）

创建数据库

    create database st;


创建表

    CREATE TABLE `users` (
      `id` int(10) DEFAULT NULL,
      `name` varchar(20) DEFAULT NULL,
      `password` varchar(30) DEFAULT NULL,
      `mail` varchar(50) DEFAULT NULL
    );


插入数据

    insert into users values(3,'li.e','pass_1','ss@132.com');

编辑测试文件（将 test.jsp 放到 /usr/local/tomcat/webapps/ROOT）

    cd /usr/local/tomcat/webapps
    
    mv ROOT ROOT_old
    
    mkdir ROOT

test.jsp 的代码

    <%@ page contentType="text/html;charset=utf-8"%>
    <%@ page import="java.sql.*"%>
    <%@ page import="com.alibaba.druid.pool.*"%>
    <html>
    <body>
    <%
    DruidDataSource dataSource = new DruidDataSource(); dataSource.setDriverClassName("com.mysql.jdbc.Driver"); dataSource.setUsername("myuser"); dataSource.setPassword("bjtest1234"); dataSource.setUrl("jdbc:mysql://localhost/st"); 
    Connection conn = dataSource.getConnection();
    Statement stmt=conn.createStatement(ResultSet.TYPE_SCROLL_SENSITIVE,ResultSet.CONCUR_UPDATABLE);
    
    String sql="select * from users";
    
    // 从 users 表读数据
    ResultSet rs=stmt.executeQuery(sql);
    while(rs.next()) {%>
    您的第一个字段内容为：<%=rs.getString(1)%>
    您的第二个字段内容为：<%=rs.getString(2)%>
    您的第三个字段内容为：<%=rs.getString(3)%>
    您的第四个字段内容为：<%=rs.getString(4)%>
    </p>

    <%}%>
    <%out.print("数据库操作成功，恭喜你");%>
    <%rs.close();
    stmt.close();
    conn.close();
    %>
    </body>
    </html>

测试 http://127.0.0.1:8080/test.jsp

![](http://cn.msopentech.com/wp-content/uploads/druid02.png)
 
配置成功！