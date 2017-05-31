<properties
    pageTitle="如何使用 Java 操作密钥保管库管理及客户端 API"
    description="如何使用 Java 操作密钥保管库管理及客户端 API"
    service=""
    resource="keyvault"
    authors="Chen Rui"
    displayOrder=""
    selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Key Vault, Java, PowerShell,  API"
    cloudEnvironments="MoonCake" />
<tags
    ms.service="key-vault-aog"
    ms.date=""
    wacn.date="05/25/2017" />

# 如何使用 Java 操作密钥保管库管理及客户端 API

## **问题描述**

在创建 KeyVaultClient 时不能通过 ApplicationTokenCredentials 来进行认证，如何通过 Java 来获取密钥保管库的密钥或机密。

## **问题分析**

需要基于 ADAL Library ，通过重写 doAuthenticate 方法来创建 KeyVaultClient 来做认证。

## **解决方法**

- PowerShell

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud
        Set-AzureRmContext -SubscriptionId "sub id"
        New-AzureRmKeyVault -VaultName 'geovault02' -ResourceGroupName 'geogroup' -Location 'China North'
        $securepfxpwd = ConvertTo-SecureString -String '1QAZxsw2' -AsPlainText -Force
        $key = Add-AzureKeyVaultKey -VaultName 'geovault02' -Name 'key1' -KeyFilePath 'E:\cer.pfx' -KeyFilePassword $securepfxpwd
        $Key.key.kid
        # 授权需要授予 get 权限，否则 Client API 读取时会报告权限问题
        Set-AzureRmKeyVaultAccessPolicy -VaultName 'geovault02' -ServicePrincipalName ‘Clinet ID’ -PermissionsToKeys decrypt,sign,get
        Set-AzureRmKeyVaultAccessPolicy -VaultName 'geovault02' -ServicePrincipalName ‘Clinet ID’ -PermissionsToSecrets Get

- Maven Dependency

        <dependency>
        <groupId>com.microsoft.azure</groupId>
            <artifactId>azure-keyvault</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>com.microsoft.azure</groupId>
            <artifactId>adal4j</artifactId>
            <version>1.2.0</version>
        </dependency>

- Test Class

    public class KeyVaultTest {
        private ApplicationTokenCredentials tokenCredentials;
        private KeyVaultClient keyVaultClient;
        private Azure azure;

        public KeyVaultTest(String clientId, String clientSecret) {
            try {
                keyVaultClient = new KeyVaultClient(new KeyVaultCredentials() {
                    @Override
                    public String doAuthenticate(String authorization, String resource, String scope) {
                        AuthenticationContext context;
                        try {
                            context = new AuthenticationContext(authorization, false, Executors.newFixedThreadPool(1));
                            ClientCredential credentials = new ClientCredential(clientId, clientSecret);
                            AuthenticationResult result = context.acquireToken(resource, credentials, null).get();
                            return result.getAccessToken();
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                        return "";
                    }
                });

                System.out.println("认证成功！");

            } catch (CloudException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        public void getKey(String keyIdentifier) {
            KeyBundle keyBundle = keyVaultClient.getKey(keyIdentifier);
            System.out.println(keyBundle.toString());
        }
    }

- Junit Test

        @org.junit.Test
            public void test(){
                    KeyVaultTest keyVault = new KeyVaultTest(
        "Client ID", "Client Secret");
                    keyVault.getKey(
        "https://geovault02.vault.azure.cn:443/keys/key1");
        }

        {"key":{"kid":"https://geovault02.vault.azure.cn/keys/key1/a0e2b77b58824f9c96f0f3b1e48e4235","kty":"RSA","key_ops":["encrypt","decrypt","sign","verify","wrapKey","unwrapKey"],"n":"n2eviMHVMcV-HBVxOiqXV1qbxBy9vEW3c_lTv3VwFaqseWIdaTq5UQvZH7QJ-lRJB_CgRpxMansLEc1YlObhcvxQ_ASeuZiHe0MaZz54Ucen-dzWNpdWHGJuSAbL7y1o-_Kqmp3oJ0SfhS8QMruiOIB8AZDE0-rZ-f7H9y0FZbxRh8CjgtfapgU2OL1O4E-RkHqSX7coHo9R1TAfGdQVe9zttFiZQ5NBwQ3G_NHN1x8342zy52VlDASy5xM-LafYQekjSVF1IHFAVlhQ6-LZs4sE70F_9QyNGS56mHgYYI-lHKo5OnfMWtBe4esaanqyMxwXPQMNTPm-sH1SYOyaYQ","e":"AQAB","d":null,"dp":null,"dq":null,"qi":null,"p":null,"q":null,"k":null,"key_hsm":null},"attributes":{"enabled":true,"nbf":null,"exp":null},"tags":null}
