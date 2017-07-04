<properties
    pageTitle="如何使用密钥管理库导入 PEM 格式的证书"
    description="如何使用密钥管理库导入 PEM 格式的证书"
    service=""
    resource="keyvault"
    authors="Miley Chen"
    displayOrder=""
    selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Key Vault, PEM"
    cloudEnvironments="MoonCake" />
<tags
    ms.service="key-vault-aog"
    ms.date=""
    wacn.date="05/22/2017" />

# 如何使用密钥管理库导入 PEM 格式的证书

通过 Azure 密钥保管库，Azure 用户能够保护和控制云应用程序和服务使用的加密密钥和其他机密。密钥保管库在设计上确保了微软无法看到或提取用户的密钥，通过加密密钥存储在云中，能提高云应用程序的性能并减少延迟。

## **问题描述**

我们在使用密钥管理库上传证书时，根据不同的情况需要上传不同格式的证书，例如 PFX、PEM 等等。有时用户开发应用在调用密钥管理时，上传 PEM 格式证书时，会返回错误：PEM is unexpected format，因此无法成功导入 PEM 格式的证书。

## **问题分析**

我们以 .net 应用开发为例，可以通过调用方法 `ImportCertificateAsync(IKeyVaultClient, String, String, String, String, CertificatePolicy, CertificateAttributes, IDictionary<String,String>, CancellationToken)` 来导入 PEM 格式的证书，更多的说明可以参考官方链接。
但是当我们调用上述方法上传 PEM 格式证书时会有一些特殊的要求（需要注意的是，这些要求同样适用于使用 CLI 以及各种其他语言的 SDK）。

具体的要求如下：

1. 在创建/导入 PEM 格式证书时，`CertificatePolicy` 中 `secret` 的 `ContentType` 的值需要设定为 `application/x-pem-file` ；如果是 PFX 的话，`ContentType` 的值为 `CertificateContentType.Pfx`。

    PEM 可以通过下面的格式定义 `CertificatePolicy`：

        var policy = new CertificatePolicy
        {
            KeyProperties = new KeyProperties
            {
                Exportable = true,
                KeyType = "RSA"
            },
            SecretProperties = new SecretProperties
            {
                ContentType = "application/x-pem-file" // PEM certificate 
                //ContentType = CertificateContentType.Pfx // PFX certificate
            }
        };

2. 需要导入的 PEM 格式的证书需要是下面格式之一，这意味着:

    - 传递给 ImportCertificateAsync 的参数也要包括 “-----” 等特殊字符。
    - 同时也要保证传递过来的字符串包含换行符从而保证是下面要求的格式。
    - 导入的证书最少是一个，最多整个字符串长度不可超过 25K。
    - 关于密钥管理库可以接受的 PEM 格式文件的说明，可以参考[链接](http://how2ssl.com/articles/working_with_pem_files/)。<br>

            Pem with unencrypted pkcs#8 Key

            -----BEGIN PRIVATE KEY-----
            …………………
            -----END PRIVATE KEY-----

            -----BEGIN CERTIFICATE-----
            ……….
            -----END CERTIFICATE-----

            -----BEGIN CERTIFICATE-----
            ……….
            -----END CERTIFICATE-----


            Pem with encrypted pkcs#8 Key

            -----BEGIN ENCRYPTED PRIVATE KEY-----
            …………………
            -----END ENCRYPTED PRIVATE KEY-----

            -----BEGIN CERTIFICATE-----
            ……….
            -----END CERTIFICATE-----

    .net 中的参数定义如下：

        base64X509 = "-----BEGIN PRIVATE KEY-----\nMIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDkZlh01m5vpwFvFaffSHzcJRl8mZtLpo4K4p8Hed+QH/………LyVcynibPKezupYJ1wg\n-----END PRIVATE KEY-----\n-----BEGIN CERTIFICATE-----\nMIIDNDCCAhygAwIBAgIQBdSVCj/ TnN09N6jDqgmOtpUUhWwrn5………7quTVwcUEAjwrPDx3s=\n-----END CERTIFICATE-----";

最后，关于导入 PEM 证书，您可以通过[链接](https://github.com/wacn/AOG-CodeSample/tree/master/KeyVault/DotNET/HelloKeyVault)下载 demo 程序，您需要替换 App.config 中的参数为您自己的密钥管理的值。这方面的信息，我们即将在官方链接中更新，请您关注我们的网站，了解更多详细的信息。