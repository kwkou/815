<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="Azure CDN HTTPS customer self cert - Azure feature guide" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-ons, CDN FAQ, CDN常见问题, 回源流量, ICP备案号, CDN默认缓存规则, 回源域名, 订阅, CNAME, 下载加速, Web加速, 网站加速, 流媒体直播加速, VOD加速, 视频点播加速, CDN价格, CDN收费, 技术文档, 帮助文档" description="Find answers to common service consulting or inquiries related to Azure CDN" metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service="cdn"
    ms.date="6/14/2016"
    wacn.date="6/14/2016"
    wacn.lang="cn"
    />
> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-faq-service-inquiry/)
- [English](/documentation/articles/cdn-enus-faq-service-inquiry/) 

# Azure CDN HTTPS加速服务-用户自有证书

Azure CDN提供HTTPS安全加速服务，支持用户上传自有证书，也支持Azure CDN代为申请证书的自动化配置，均只开放给付费用户。本文档针对用户自己上传证书自助化配置的情况以及证书说明，Azure CDN代为申请证书的配置操作请参考[Azure CDN HTTPS加速服务-Azure CDN代申请证书](https://www.azure.cn/documentation/articles/cdn-https-how-to/)。二者的区别请参考[常见问题-服务咨询](https://www.azure.cn/documentation/articles/cdn-faq-service-inquiry/)。


## 配置说明

- HTTPS加速只开放给付费用户。
- 支持泛域名的HTTPS加速。
- 支持的加速类型为标准版CDN加速类型，如网页加速，下载加速，点播加速和直播加速。
     >**注意** 图片处理加速类型暂不支持HTTPS加速。
- 需要在**新版Azure CDN管理界面**启用HTTPS服务，并上传PEM格式的证书和秘钥。详见后文的“自助式启用HTTPS加速步骤”。
- 开启HTTPS加速后，默认同时支持HTTP和HTTPS方式的请求。如需强制将HTTP请求跳转成HTTPS请求，请联系世纪互联进行配置。我们后续会在管理界面实现自动化选择。
- 回源协议默认跟随用户发起的请求协议，即HTTP请求通过HTTP协议回源，HTTPS请求通过HTTPS协议回源。如需指定只有HTTP回源或者只有HTTPS回源，请联系世纪互联进行配置。我们后续会在管理界面实现自动化选择。


## 证书说明

- 用户自有证书的HTTPS加速是基于SNI技术实现的，SNI证书可以多个HTTPS 客户共享同一个IP 地址。
    >**注意** SNI证书不支持Windwos XP系统下所有的IE版本，浏览器会提示不受信任。

- 开启HTTPS加速后，需要上传加速域名的证书和私钥，证书和域名、证书和私钥需要匹配，否则会校验出错。

- 支持看证书信息，但不支持证书下载，也不支持秘钥查看，请保管好证书相关信息。
- 暂不不支持中间证书或证书链，用户只需上传加速域名对应的具体证书。

### 证书格式说明

- 证书格式为PEM格式，不支持其他格式的证书，需将其他格式的证书转换成PEM格式，可以通过openssl工具进行转换。请参考下文“常见证书格式转换”。
- 证书以 `-----BEGIN CERTIFICATE-----`，`-----END CERTIFICATE-----` 开头结尾。
- 秘钥目前只支持RSA PKCS8编码格式，PKCS8编码的私钥是以 `-----BEGIN PRIVATE KEY-----`，`-----END PRIVATE KEY-----` 开头结尾。
    >**注意**
    >可以使用openssl工具对秘钥编码格式进行转换，以RSA PKCS#1转PKCS8为例：
    >
    >openssl.exe pkcs8 -topk8 -inform PEM -outform PEM -in .\keyfile.key -out converted.key -nocrypt


### 常见证书格式转换

#### DER格式转换为PEM

- 证书转换

    ```shell
    openssl x509 -in cert.der -inform DER -out cert.pem -outform PEM
    ```
- 私钥转换

    ```shell
    openssl rsa -in privagekey.der -inform DER -out privatekey.pem -outform PEM
    ```

#### PFX格式转换为PEM

- 证书转换

    ```shell
    openssl pkcs12 -in cert.pfx -nokeys -out cert.pem
    ```

- 私钥转换

    ```shell
    openssl pkcs12 -in cert.pfx -nocerts -out key.pem -nodes
    ```

## 价格说明

自有证书的HTTPS加速，将于2017年7月1日起，使用标准版服务价格，2017年7月1日前，该服务属于高级服务，使用高级服务价格。具体计费方式请参见[Azure官方网站](https://www.azure.cn/pricing/details/cdn/)。


## 自助式启用HTTPS加速步骤

可以为所有符合条件的CDN域名开启自有证书的HTTPS加速（付费账号下所有的标准版加速类型的CDN域名，图片加速节点除外）。

>**注意**如果需要在“Azure门户预览”中新建CDN Profile和节点，并且启用自有证书的HTTPS加速，Pricing Tier请选择“S1 Standard”。"P1 Premium"中的HTTPS是指Azure CDN代为申请证书。
  
  ![][15]

1. 以新版Azure门户预览为例，点击“Manage”进入老版Azure CDN管理门户。点击“访问新站点”，到新版Azure CDN管理门户。
     >**注意**需要跳转到新版Azure CDN管理门户上传HTTPS自有证书。
    
    **新版Azure门户预览中的CDN profile界面：**

    ![][1]

    **老版Azure CDN管理门户界面：**

    ![][2]

    **新版Azure CDN管理门户界面：**
    ![][3]

2. **上传证书**：点击“证书管理”，点击“添加一张SSL证书”，输入证书名称，便于分辨证书。上传需要启用HTTPS服务的域名证书，注意证书必须是PEM格式，秘钥目前只支持RSA PKCS8编码格式。具体证书格式转换请参考上文的“证书说明”。
     >**注意**证书上传后，需要到“域名管理”界面，将证书和域名进行绑定,证书才会进行部署。
     
    ![][4]

3. **域名绑定证书**：可以在“证书管理”中绑定，也可以在“域名管理”中绑定.
    
    - 可以直接在“证书管理”处上传证书时选择要绑定的域名；
    ![][9]
    - 也可以上传证书后，在证书管理页面，选择证书，点击“编辑绑定”-> “添加绑定域名”；
     ![][5]
     ![][6]
    - 也可以点击“域名管理”，选择需要启用HTTPS服务的域名，在右侧界面选择“HTTPS（客户提供证书）”，点击“启用”绑定一张证书，从上传证书下拉列表框中选择已上传的证书，点击确定。如果没有符合的证书，点击“证书管理”，参考步骤3上传证书。
    ![][8]
4. **证书部署**：绑定域名后，系统会提示“证书正在部署中，生效时间一般为2-4小时，如果超过24小时部署仍未完成，请联系工作人员。 ”

    ![][13]
5. **域名绑定成功**后，系统会提示“证书绑定成功，可以通过HTTPS访问该加速域名”，并看到证书详情。同时该域名的“HTTPS状态（客户提供证书）”变为“活跃”。

    
    - 点击该域名，可以查看证书信息。
    ![][10]
    - 域名“HTTPS状态（客户提供证书）”变为“活动”。
    ![][11]
    - “证书管理”处，改证书绑定域名数也会变化。
    ![][12]

5. **查看证书详情**：点击证书管理处任一证书，查看证书详情，和所绑定域名信息。

    ![][7]
6. **查看是否生效**：通过HTTPS访问域名，可以看到带小锁的标志，表示HTTPS加速启用成功。
    ![][14]   

<!--Image references-->
[1]: ./httpsimage/manage.png
[2]: ./httpsimage/oldportal.png
[3]: ./httpsimage/newportaloverview.png
[4]: ./httpsimage/uploadcert.png
[5]: ./httpsimage/bindcert1.png
[6]: ./httpsimage/bindcert1.1.png
[7]: ./httpsimage/certdetail.png
[8]: ./httpsimage/bindcert2.png
[9]: ./httpsimage/bindcert3.png
[10]: ./httpsimage/success.png
[11]: ./httpsimage/successdomainstatuspng.png
[12]: ./httpsimage/cert4.png
[13]: ./httpsimage/deploying.png
[14]: ./httpsimage/finalaccess.png
[15]: ./httpsimage/ibizapricingtier.png