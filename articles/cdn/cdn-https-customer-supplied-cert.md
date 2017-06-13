<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="How to enable Azure CDN HTTPS with customer certificate - Azure feature guide" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-ons, CDN加速, CDN服务, 主流CDN, Web加速, Web, 网页加速, 静态加速, 缓存规则, 图片加速, CDN技术文档, CDN帮助文档, 门户网站加速" description="Learn How to create Web acceleration type CDN on Azure Management Portal and default caching rules for Web CDN" metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service="cdn"
    ms.date="6/13/2017"
    wacn.date="6/13/20167
    wacn.lang="cn"
    />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-how-to-create-Web-CDN-endpoint/)
- [English](/documentation/articles/cdn-enus-how-to-create-Web-CDN-endpoint/) 
# Azure CDN HTTPS加速服务-用户自有证书

Azure CDN提供HTTPS安全加速服务，支持用户上传自有证书，也支持Azure CDN代为申请证书的自动化配置，均只开放给付费用户。本文档针对用户自己上传证书自助化配置的情况以及证书说明，Azure CDN代为申请证书的配置操作请参考[Azure CDN HTTPS加速服务-Azure CDN代申请证书](https://www.azure.cn/documentation/articles/cdn-https-how-to/)。二者的区别请参考常见问题。


## 配置说明

- HTTPS加速只开放给付费用户。
- 支持泛域名的HTTPS加速。
- 支持的加速类型为网页加速，下载加速，点播加速和直播加速。
     >**注意** 图片处理加速类型暂不支持HTTPS加速。
- 需要在**新版Azure CDN管理界面**启用HTTPS服务，并上传PEM格式的证书和秘钥。详见后文的“自助式启用HTTPS加速步骤”。
- 开启HTTPS加速后，默认同时支持HTTP和HTTPS方式的请求。如需强制将HTTP请求跳转成HTTPS请求，请联系世纪互联进行配置。我们后续会在管理界面实现自动化选择。
- 回源协议默认跟随用户发起的请求协议，即HTTP请求通过HTTP协议回源，HTTPS请求通过HTTPS协议回源。如需指定只有HTTP回源或者只有HTTPS回源，请联系世纪互联尽心配置。我们后续会在管理界面实现自动化选择。

## 证书说明

- 用户自有证书的HTTPS加速是基于SNI技术实现的，SNI证书可以多个HTTPS 客户共享同一个IP 地址。
    >**注意** SNI证书不支持Windwos XP系统下所有的IE版本，浏览器会提示不受信任。

- 开启HTTPS加速后，需要上传加速域名的证书和私钥，证书和私钥需要匹配，否则会校验出错。

- 支持看证书信息，但不支持证书下载，也不支持秘钥查看，请保管好证书相关信息。

### 证书格式说明

- 证书格式为PEM格式，不支持其他格式的证书，需将其他格式的证书转换成PEM格式，可以通过openssl工具进行转换。请参考下文“常见证书格式转换”。
- 证书以 `-----BEGIN CERTIFICATE-----`，`-----END CERTIFICATE-----` 开头结尾。
- 秘钥目前只支持RSA PKCS8编码格式，PKCS8编码的私钥是以 `-----BEGIN PRIVATE KEY-----`，`-----END PRIVATE KEY-----` 开头结尾。
    >**注意**
    >可以使用openssl工具对秘钥编码格式进行转换，以RSA PKCS#1转PKCS8为例：
    >
    >openssl.exe pkcs8 -topk8 -inform PEM -outform PEM -in .\keyfile.key -out converted.key -nocrypt

- 暂不不支持中间证书或证书链，用户只需上传加速域名对应的具体证书。

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

自有证书的HTTPS加速价格，将于2017年7月1日起，享用标准版服务价格，2017年7月1日前，该服务属于高级服务，使用高级服务价格。具体计费方式请参见[Azure官方网站](https://www.azure.cn/pricing/details/cdn/)。

## 自助式启用HTTPS加速步骤

1. 可以为所有符合条件的已有域名开启HTTPS（付费账号，图片加速节点除外）。以新版Azure门户预览为例，点击“Manage”进入老版Azure CDN管理门户。点击“访问新站点”，到新版Azure CDN管理门户。
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
    >**注意**部署生效时间一般为2-4小时，如果超过24小时部署仍未完成，请联系工作人员。
 - 可以直接在“证书管理”处上传证书时选择要绑定的域名；
    ![][9]
 - 也可以上传证书后，在证书管理页面，选择证书，点击“编辑绑定”-> “添加绑定域名”；
     ![][5]
     ![][6]
 - 也可以点击“域名管理”，选择需要启用HTTPS服务的域名，在右侧界面选择“HTTPS（客户提供证书）”，点击“启用”绑定一张证书，从上传证书下拉列表框中选择已上传的证书，点击确定。如果没有符合的证书，点击“证书管理”，参考步骤3上传证书。
    ![][8]

4. **域名绑定成功**后，在“域名管理”处，可以看到该域名的“HTTPS状态（客户提供证书）”为活跃。

    ![][11]
     点击该域名，可以查看证书信息。
    ![][10]
     “证书管理”处，改证书绑定域名数也会变化。
    ![][12]

5. **查看证书详情**：点击证书管理处任一证书，查看证书详情，和所绑定域名信息。

    ![][7]
    
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
