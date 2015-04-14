<properties linkid="ckan-vmdepot-tutorial" urlDisplayName="使用VMDepot镜像快速部署CKAN开放数据门户" pageTitle="使用VMDepot镜像快速部署CKAN开放数据门户" metaKeywords="使用VMDepot镜像快速部署CKAN开放数据门户" description="使用VMDepot镜像快速部署CKAN开放数据门户" metaCanonical="" services="" documentationCenter="develop"  title="使用VMDepot镜像快速部署CKAN开放数据门户" authors="" solutions="" manager="TK" editor="Haifeng Liu" />
<tags ms.service=""
    ms.date="11/26/2014"
    wacn.date="04/11/2015"
    />

#使用VMDepot镜像快速部署CKAN开放数据门户#

最新发布的CKAN VMDepot镜像针对中国用户强化了中文支持，提升了与MS Office办公软件的互操作性，并集成了常用插件和最佳实践配置参数。
使得CKAN原本十分复杂繁琐的部署流程变得非常简单。本指南展示了如何使用VMDepot镜像快速部署CKAN开放数据门户:

- [前提条件](#id_1)
- [使用VMDepot镜像部署CKAN](#id_2)
	- [1. 导入CKAN镜像到您的本地帐户](#id_2_1)
	- [2. 使用本地CKAN镜像创建虚机](#id_2_2)
	- [3. 安装后的配置（必须完成）](#id_2_3)
- [创建您的第一个数据集](#id_3)
- [定制您的CKAN](#id_4)


## <a name='id_1'></a>前提条件 ##
您需要一个可用的微软中国Azure公有云账户

## <a name='id_2'></a>使用VMDepot镜像部署CKAN ###
### <a name='id_2_1'></a>1. 导入CKAN镜像到您的本地帐户 ###

打开Azure控制台：[https://manage.windowsazure.cn](https://manage.windowsazure.cn)
选择**Virtual Machines** > **Images** > **Browse VM Depot**:

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/1.PNG)

在**Ubuntu**类别下找到**CKAN**镜像，这是已经发布在VM Depot上的一键部署镜像：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/2.PNG)

下面需要将此镜像拷贝到的用户存储账户，可以选择已有的存储帐户，也可新建：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/3.PNG)

拷贝过程将花费几分钟的时间：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/4.PNG)

拷贝完成后，本地CKAN镜像的状态是**Pending registration**，点击**Register**注册：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/6.PNG)

填写注册镜像名称：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/7.PNG)

镜像状态变为**Available**，至此，CKAN镜像已经准备完毕：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/8.PNG)

### <a name='id_2_2'></a>2. 使用本地CKAN镜像创建虚机 ###
在Azure管理控制台中，选择**Virtual Machines** > **Create a Virtual Machine**:

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/9.PNG)

选择**From Gallery**：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/10.PNG)

在**My Images**类别，找到我们刚刚注册的CKAN镜像，点击**下一步**：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/11.PNG)

填写虚机名称，用户名和认证方式，注意这里的默认用户名为**azureuser**,点击**下一步**：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/12.PNG)

创建Cloud Service，在本例中，服务地址为**mytestckan.chinacloudapp.cn**，
注意需要打开至少三个TCP端口，分别为**22，80，443**，点击**下一步**：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/13.PNG)

确认VM Agent已经安装，点击**下一步**：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/14.PNG)

等待直至虚拟机状态变为**Running**，至此CKAN镜像部署完毕：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/15.PNG)

在浏览器中输入网址：[http://mytestckan.chinacloudapp.cn](http://mytestckan.chinacloudapp.cn)，可以看到CKAN门户已经可以访问了：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/16.PNG)

### <a name='id_2_3'></a>3. 安装后的配置（必须完成） ###
由于CKAN的特殊要求，每一个新部署的镜像需要调整**ckan.site_url**参数才能正常工作，下面演示如何修改此参数：

Windows用户可通过安装ssh客户端，如PuTTY，连接到新建的CKAN虚机；Linux和Mac用户可直接通过ssh命令连接：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/18.PNG)

本例中我们采用密码认证方式登录mytestckan.chinacloudapp.cn

运行以下命令，运行前将*YOUR-CKAN-DOMAIN-NAME*替换为您实际的网站域名，在本例中为mytestckan.chinacloudapp.cn：

`$sudo sed -i 's/ckanimage.chinacloudapp.cn/YOUR-CKAN-DOMAIN-NAME/' /etc/ckan/default/production.ini`

> 注意：上述命令中的网站域名请勿加“**http://**”前缀。

检查命令是否生效：

`$cat /etc/ckan/default/production.ini | grep ckan.site_url`

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/19.PNG)

> 注意：您也许会为您的CKAN门户申请不同的域名，请将site_url替换为最终用户实际访问的域名。

重启apache和nginx服务：

`$sudo service apache2 restart && sudo service nginx restart`


![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/20.PNG)

至此，您的CKAN已经配置完成，可以正常使用了。


## <a name='id_3'></a>创建您的第一个数据集 ##
以**admin**身份登录CKAN门户网站，默认密码是**admin**，登录后请立即更改密码：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/16.PNG)

点击**数据集** > **增加数据集**

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/17.PNG)

输入数据集名称，点击**下一步**：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/21.PNG)

点击**上传**：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/22.PNG)

选择本地Excel文档：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/23.PNG)

格式选择为**xls**，点击**下一步**：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/24.PNG)

可以选择补充数据集的额外信息，点击**完成**：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/25.PNG)

至此，CKAN将自动导入Excel表格，并同时生成OData格式数据访问API供应用程序访问。

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/26.PNG)

选择**浏览**>**预览**可以查看导入的数据：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/27.PNG)

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/28.PNG)

数据导入完成后，可在CKAN门户首页看到新增的数据集：

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/29.PNG)


## <a name='id_4'></a>定制您的CKAN ##
您也许希望改变此镜像默认的配置如网站标题，介绍文字等，
可以用admin登录后，点击首页右上角**系统管理员设置**，
选择**配置**选项卡，在这里，您可以对网站风格和文字进行定制:

![](https://raw.githubusercontent.com/msopentechcn/docs/master/images/30.PNG)
