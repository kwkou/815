<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="How to use CDN - HTTPS" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure HTTPS, CDN HTTPS, Azure caching, Azure add-ons, CDN, CDN加速, CDN服务, 主流CDN, 多场景加速, 免费CDN, CDN网站加速, 网站加速, 网页加速, 静态加速, 下载加速, VOD加速, 流媒体直播加速, 云服务,  存储账户,缓存刷新, 回源, 云加速, 加速效果, 节点, 流量, CNAME, 带宽, 网速, 防盗链,https加速, 低成本带宽, 访问加速, CDN缓存, 存储账户, 云服务, 网站, 媒体服务, ICP备案号, ICP编号, ICP, 缓存刷新, 内容预取, 日志下载, CDN帮助文档, CDN技术文档" description="Learn how to create HTTPS CDN acceleration type." metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service="cdn"
    ms.date="7/7/2016"
    wacn.date="7/7/2016"
    wacn.lang="cn"
    />
> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-https-how-to/)
- [English](/documentation/articles/cdn-enus-https-how-to/) 
# Azure CDN HTTPS 加速服务


## 开通流程
目前CDN HTTPS加速服务仅对Azure付费用户开放。

1. **开通申请** 请联系[Azure 技术支持团队](https://www.azure.cn/support/contact/)进行开通申请。需要客户提供需要开通HTTPS加速服务的Azure订阅ID。


2. **自助式创建** Azure CDN团队收到开通请求并确认之后，会对用户提供的Azure订阅开通HTTPS加速服务。之后，用户可以自行登陆到Azure管理门户完成自助式的创建操作。请参考自助式创建流程。

    ![][1]


3. **SSL证书申请和配置**
收到配置请求后，Azure CDN会代为申请SSL证书（详细的证书类型请见附注）。
    > **注意**

    > 此证书申请配置时间大约需要***五个工作日***。同时在申请过程中，需要客户帮忙配合证书签发机构确认域名所有权。具体确认方式，请参见本文后面的详细流程。

4. **配置完成生效**
全部配置完成后，用户可以像创建其他加速类型服务一样，做最后的CNAME记录设置。之后，通过统一的Azure CDN管理门户来完成其他相应的管理配置任务。


## 自助式创建流程
1. 当用户在Azure管理门户完成HTTPS加速类型的创建之后，通过点击下图中的“管理”按钮，跳转到Azure CDN管理门户进行后续的开通操作。

	![][2]

	![][3]

2. 在Azure CDN管理门户中，通过“域名管理”选中要开通配置的HTTPS加速域名，然后点击“查看HTTPS配置状态”。

	![][4]

3. 这时用户可以看到完成整个的HTTPS开通配置需要经过五个步骤。在目前的第二步“提交证书申请”，需要用户按界面的提示补充下列信息：

	**预估带宽**：预计可能需要使用的峰值带宽值。

	**预估平均文件大小**：预计的需要被缓存加速的平均文件大小。

	**加速需求时间**：确认该HTTPS加速需求是长期的还是短期的。

	**客户端到CDN节点的访问端口**：确认需要开通的客户端到CDN节点的访问方式。

	1) 只开通HTTPS访问（HTTP访问会被禁止）

	2) 同时开通HTTP和HTTPS访问

	3) 将HTTP访问统一跳转至HTTPS访问

	**CDN节点到源站的回源访问端口**：确认CDN节点可以回源访问的方式。

	1) 只能使用HTTP回源

	2) 只能使用HTTPS回源

	3) 可以同时使用HTTP和HTTPS回源

	**检测URL**：输入一个后续可以用来检查访问的URL。要确保源站上的该URL可以被访问。

4. 将上一步当中的所有相关信息填写完毕后，点击“确定”按钮完成操作。之后再次查看HTTPS配置状态如下：

	![][5]

	当Azure CDN服务商后台确认了所有提交的相关信息，并正式向证书签发机构提交了证书申请之后，用户会来到步骤三如下这个界面：

	![][6]

5. 接下来证书申请过程进行到了最关键的一步，需要用户来确认域名所有权。因为证书是由Azure CDN服务商代用户向证书签发机构申请的，所以这个过程当中证书签发机构会向最终用户进行域名所有权确认。确认的方式是**域名所有者按照域名签发机构所发的邮件里提到的方式来完成最终确认**。

	收到邮件的方式有两种：

	1) **默认方式**：证书签发机构接收到请求后，默认会在第一时间向加速域名相关的邮箱（具体见上面的截图）发送域名确认邮件。如果客户选择这种方式来获取确认邮件，可以直接点击“确定”按钮前往下一步。
		
		

	2) **DNS TXT方式**：如果用户无法登陆到上述邮箱来完成域名所有权确认，可以使用如下图所示的**通过创建DNS TXT记录**的方式来完成。这种方式可能会额外增加一些完成时间。

	![][8]

	DNS TXT记录创建完成后，在Windows操作系统下可以使用如下命令来验证： nslookup -qt=txt www.cdn.test.com

	![][9]

	之后，和默认方式一样，用户登陆到DNS TXT记录中指定的邮箱来完成后续的域名所有权验证。

	当用户在本步骤确定已经成功创建了相应的DNS TXT记录后，点击本步骤中的“确定”按钮，前往下一步。

	![][10]

	

6. 用户前往邮箱完成域名所有权认证
	
	当Azure CDN后台确认了用户所选择的收取认证邮件的方式后，用户会看到如下界面：

	![][11]

	这时用户就可以前往响应的邮箱去完成域名所有权认证。

	邮件标题: Please validate ownership of your domain www.cdn.test.com -- DigiCert order 00123456

	邮件正文：

	![][7]

	用户需要点击邮件中的确认链接来完成域名所有权的确认。

	之后，用户需要在步骤四的界面里点击“完成”，来最终完成整个域名所有权的确认。	

7. 接下来整个配置流程会进入到最后一步，如下图：

	![][12]

	在这一步骤中，Azure CDN后台从证书签发机构获得相应的证书之后，需要一定的时间来完成最终的配置任务。之后，用户就会看到最终完成的界面：

	![][13]

	最后，用户可以像创建其他加速类型的加速域名一样，通过自己的域名服务商完成最后的CNAME配置操作，将自定义加速域名指向Azure CDN平台所提供的以**.mschcdn.com**结尾的CDN域名，从而让整个配置生效。




## 附录

### 关于所使用的SSL证书
SSL证书类型为SAN多域名证书（SAN/UCC SSL）: 

SAN 证书 – Subject Alternative Name certificates, 又称为 UCC 证书– Unified Communication Certificates。SANs SSL证书允许您在同一张证书中，添加多个需要保护的"域名"或"服务器"名。这种功能为客户提供了非常大的使用弹性，它使您可以创建一张易于使用和安装，却又比通配符SSL 证书更安全，完全适合您服务器安全需求的SSL证书。

证书签发机构： <https://www.digicert.com/>
	
该SSL证书由Azure CDN代为申请，安装和维护。

### 关于计费信息
CDN HTTPS加速节点的计费信息，会和其他CDN加速节点一样可以通过相应的[Azure帐户门户](https://account.windowsazure.cn)查看，并被汇总到相应的Azure订阅下面。


### 关于CDN HTTPS的价格
目前Azure CDN HTTPS加速服务包含在高级版当中，具体的计费方式请参见[Azure官方网站](https://www.azure.cn/home/features/cdn/pricing/)。


<!--Image references-->
[1]: ./media/cdn-https/h001.png
[2]: ./media/cdn-https/h002.png
[3]: ./media/cdn-https/h003.png
[4]: ./media/cdn-https/h004.png
[5]: ./media/cdn-https/h005.png
[6]: ./media/cdn-https/h006.png
[7]: ./media/cdn-https/h007.png
[8]: ./media/cdn-https/h008.png
[9]: ./media/cdn-https/h009.png
[10]: ./media/cdn-https/h010.png
[11]: ./media/cdn-https/h011.png
[12]: ./media/cdn-https/h012.png
[13]: ./media/cdn-https/h013.png
