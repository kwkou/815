可以使用通过企业解决方案生成的根证书（推荐），也可以生成自签名证书。 如果使用自签名证书，请确保参阅[为点到站点连接创建自签名根证书](/documentation/articles/vpn-gateway-certificates-point-to-site/#rootcert)一文。 本文包含的特定设置是生成 P2S 兼容证书所必需的。

创建根证书以后，请将公共证书数据（不是密钥）作为 Base-64 编码的 X.509 .cer 文件导出。 然后，将公共证书数据从根证书上传到 Azure。

* **企业证书：**如果要使用企业级解决方案，可以使用现有的证书链。 获取要使用的根证书的 .cer 文件。
* **自签名根证书：**如果使用的不是企业证书解决方案，则需要创建自签名根证书。 若要使用点到站点连接，根证书必须包含特定值。 有关说明，请参阅以下文章：
    * 若要创建自签名根证书，请参阅[为点到站点连接创建自签名根证书](/documentation/articles/vpn-gateway-certificates-point-to-site/#rootcert)。
    * 若要导出公钥（.cer 文件），请参阅[为点到站点连接创建自签名根证书](/documentation/articles/vpn-gateway-certificates-point-to-site/#cer)。