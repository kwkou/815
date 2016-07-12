<properties linkid="" urlDisplayName="" pageTitle="MySQL服务问题 - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,快照备份,增量备份,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS,FAQ" description="针对用户在使用MySQL 数据库 on Azure中遇到的一些常见技术问题,提供快速解答。如果您仍存有疑问,欢迎联系技术支持。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="v-chuw" solutions="" manager="RongYu" editor="" />

<tags ms.service="mysql" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="cn" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-point-in-time-restore/)
- [English](/documentation/articles/mysql-database-enus-point-in-time-restore/)

# MySQL Database on Azure备份和恢复

MySQL Database on Azure 支持完全备份（也叫快照备份）和增量备份，备份所用空间不占用数据库本身分配的数据存储限额。您可以轻松回溯数据库到过去7天的任意一个时间点或者保存在系统中的某一个特定的快照备份。

##**完全备份**

MySQL Database on Azure每天在您指定的时间自动备份您的MySQL数据库，也就是完全备份，完全备份保留30天。

此外您也可以自己做即时的手动完全备份，一共可以产生10个手动完全备份，手动完全备份不会被自动清除，达到限制个数时您需要删除手动完全备份才可以建新的手动完全备份。

手动即时备份的步骤：

1. 登陆[Azure管理门户](http://manage.windowsazure.cn/), 在左边服务列表里点击“MYSQL DATABASE ON AZURE"。
2.	在服务器列表里点击要恢复的数据库服务器进入服务器界面，点击“备份”页面。
3.	点击“立即备份”键并确认操作。

##**增量备份**

MySQL Database on Azure会自动备份过去七天的数据库增量部分（或者改动部分）以实现基于任意时间点的回滚。用户不需要做任何的操作。

##**还原数据库到任意时间点**

MySQL Database on Azure支持还原到过去七天中的某一时间点。

恢复到某一时间点的步骤：

1. 登陆[Azure管理门户](http://manage.windowsazure.cn/),在左边服务列表里点击“MYSQL DATABASE ON AZURE"。
2. 在服务器列表里点击要恢复的数据库服务器进入服务器界面。点击“操作台”页面。
![还原到任意时间点][1]
3. 在下面的任务菜单里点击“还原”键，弹出对话框。
4. “还原类型”缺省已选中“回退到一个时间点”，选择您想要还原的时间点，还原的目标数据库服务器实例，和版本。然后点击完成键。
![还原到任意时间点][2]
5. (可选)如果用户有多地区部署需求或者异地迁移需求，可以在弹出的对话框里，在“还原为”项中选择“新服务器”。并在“位置”下拉菜单中选择指定区域，点击确认。需要注意的是，异地还原由于是跨地域拷贝数据的过程，因而会引起数据传输。
6. 回到服务器列表页面，确认新的服务器创建成功（状态变成“正在运行”）。数据恢复的过程从几分钟到几小时不等，这与您选择的时间点和数据库服务器在当天的增量大小有关。

您可以用原服务器实例的账号访问新的服务器实例。但注意账号里的服务器名字前缀需要替换成新的服务器名字。

##**还原一个完全备份到新实例**

MySQL Database on Azure支持还原一个完全备份到一个新的实例。具体步骤如下：

1.	登陆[Azure管理门户](http://manage.windowsazure.cn/), 在左边服务列表里点击“MYSQL DATABASE ON AZURE"。
2.	在服务器列表里点击要恢复的数据库服务器进入服务器界面，点击“备份”页面。
3.	选中要恢复的完全备份，点击“还原”键。
![还原一个完全备份到新实例][3]
4.	在弹出的对话框里，“还原类型”缺省设为“回退到完全备份”，“选择的备份”自动设成选中的完全备份，“还原为” 缺省设为“新服务”。输入新的服务实例名称和版本。然后点击完成键。
![还原一个完全备份到新实例][4]
5. (可选)如果用户有多地区部署需求或者异地迁移需求，可以在弹出的对话框里，在“还原为”项中选择“新服务器”。并在“位置”下拉菜单中选择指定区域，点击确认。需要注意的是，异地还原由于是跨地域拷贝数据的过程，因而会引起数据传输。
6.	回到服务器列表，确认新的服务器创建成功（状态变成“正在运行”）。这个过程一般在2分钟内完成。您可以用原服务器实例的账号访问新的服务器实例。但注意账号里的服务器名字前缀需要替换成新的服务器名字。

##**还原一个完全备份到原实例**

MySQL Database on Azure支持还原一个完全备份到原实例。具体步骤如下：

1.	登陆[Azure管理门户](http://manage.windowsazure.cn/), 在左边服务列表里点击“MYSQL DATABASE ON AZURE"。
2.	在服务器列表里点击要恢复的数据库服务器进入服务器界面，点击“备份”页面。
3.	选中要恢复的完全备份，点击“还原”键。
4.	在弹出的对话框里，在“还原为”项中选择“当前服务”。同时确认“我理解还原服务器将需要几分钟，在此期间无法访问数据库服务器”。然后点击完成键。
![还原一个完全备份到原实例][5]
5.	回到服务器列表，确认该服务器状态变成“正在运行”，这表示服务器已经恢复成功。

<!--Image references-->

[1]: ./media/mysql-database-point-in-time-restore/Restore1.jpg
[2]: ./media/mysql-database-point-in-time-restore/Restore2.jpg
[3]: ./media/mysql-database-point-in-time-restore/Restore3.jpg
[4]: ./media/mysql-database-point-in-time-restore/Restore4.jpg
[5]: ./media/mysql-database-point-in-time-restore/Restore5.jpg