<properties
	pageTitle="如何在 JAVA 中导入 Wosign 证书"
	description="介绍如何在 JAVA 中导入 Wosign 证书。"
	services="app-service-web-aog"
	documentationCenter=""
	authors=""
	manager=""
	editor=""
	tags="Java,Wosign 证书"/>

<tags
	ms.service="app-service-web-aog"
	ms.date="10/28/2016"
	wacn.date="10/28/2016"/>

# 如何在 JAVA 中导入 Wosign 证书 #

**问题：** JAVA 在调用 Azure 的 HTTPS 的 REST API 时，会经常报证书问题。错误如下：

`PKIX：unable to find valid certification path to requested target`

**原因：** JDK 有一套单独的证书库，Java 在访问 HTTPS 服务时，会使用自己的证书仓库中的信任根证书，对 HTTPS 的证书，进行校验是否可信。如果服务提供方的根证书不在 JDK 可信证书库中，就会报该证书不存在。而 Azure 使用 Wosign 根证书，默认是不包含在 JDK 的证书库中。所以就会出现该问题。


**解决方法：**安装 Wosign 根证书到 JDK 的证书库，详细步骤如下：

1. 从 [http://www.wosign.com/Root/index.htm#](http://www.wosign.com/Root/index.htm# "http://www.wosign.com/Root/index.htm#") 站点 下载 WoSign 根证书（Certification Authority of WoSign），将 .crt 文件后缀改为 .cer
2. 执行以下命令导入

		keytool -keystore "C:\Program Files\Java\jdk1.8.0_71\jre\lib\security\cacerts" -importcert -alias WoSign -file WS_CA1_NEW.cer

	接下来 会提示输入密码，默认密码为 changeit，输入之后，选择‘是’将其安装到 JDK 可信证书库中。

3. 如果看到以下结果，则导入成功。

	![certification-imoport-success](./media/aog-web-app-java-import-wosign-certification/certification-import-success.png "certification-import-success")