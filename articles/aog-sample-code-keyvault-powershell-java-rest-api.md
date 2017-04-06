<properties
    pageTitle="密钥保管库 Java 代码示例"
    description="密钥保管库 Java 代码示例"
    service=""
    resource=""
    authors="Allan Li"
    displayOrder=""
    selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="Keyvault, PowerShell, Java, Rest API"
    cloudEnvironments="MoonCake" />
<tags
    ms.service="sample-code-aog"
    ms.date=""
    wacn.date="03/31/2017" />

# 密钥保管库 Java 代码示例

Azure 密钥保管库使得用户能够保护和控制云应用程序和服务使用的加密密钥和其他机密，开发者也可以很方便安全的利用 .NET 和 Node.js SDK 来从密钥保管库里获取到机密信息。那么对于 Java，在 SDK 还没有发布的情况下，该如何使用密钥保管库呢？这片文章就提供一个简单示例。包括以下内容：

1. PowerShell 脚本创建密钥保管库并添加 Azure 存储连接字符串作为机密信息。
2. 利用 ADAL4J 库来获取访问密钥保管库的令牌（Token）。
3. 使用令牌去访问密钥保管库并获取 Azure 存储连接字符串，从而连接上 Azure 存储。

## 创建密钥保管库

此 PowerShell 脚本会去创建密钥保管库，然后将 Azure 存储连接字符串存入，最后注册当前示例应用并授予获取权限，最终打印出后面程序需要用到的信息。

    # TODO: update below settings based on your environment
    $subscriptionId = ""
    $keyVaultName = ""
    $keyVaultResourceGroupName = ""
    $location = "" # China East, China North
    $appName = ""
    $appKey = "" # password you want to set
    $appUri = "" # such as http://localhost:7880
    $secretName = "" # secret name you want to save
    $stConnStr = "" # secret you want to save

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud

    # In case the account has multiple subscription, make sure the expected subscription is selected
    $selectedSub = Select-AzureRmSubscription -SubscriptionId $subscriptionId
    $tenantId = $selectedSub.Tenant.TenantId

    New-AzureRmResourceGroup -Name $keyVaultResourceGroupName -Location $location

    # If you see the error The subscription is not registered to use namespace 'Microsoft.KeyVault' when you try to create your new key vault
    # Run Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.KeyVault, and then rerun your New-AzureRmKeyVault command
    New-AzureRmKeyVault -VaultName $keyVaultName -ResourceGroupName $keyVaultResourceGroupName -Location $location

    $secretValue = ConvertTo-SecureString $stConnStr -AsPlainText -Force
    $stConnStrSecret = Set-AzureKeyVaultSecret -VaultName $keyVaultName -Name $secretName -SecretValue $secretValue

    # Register your app to Azure AD now
    # And get its client id and client key
    $app = New-AzureRmADApplication -DisplayName $appName -HomePage $appUri -IdentifierUris $appUri -Password $appKey
    $appSP = New-AzureRmADServicePrincipal -ApplicationId $app.ApplicationId

    # Authorize the app to use the secret
    Set-AzureRmKeyVaultAccessPolicy -VaultName $keyVaultName -ServicePrincipalName $appSP.ApplicationId -PermissionsToSecrets get

    Write-Host ("Target tenant ID is: {0}" -f $tenantId)
    Write-Host ("Storage connection string key vault URI: {0}" -f $stConnStrSecret.Id)
    Write-Host ("App Client Id is: {0}" -f $appSP.ApplicationId)
    Write-Host ("App Client Key is: {0}" -f $appKey) 

## 获取密钥保管库令牌

利用 ADAL4J Java，基于上一步获取到的 `ClientId` 和 `ClientKey`，获取拥有 get 权限的访问令牌。

    <!-- https://mvnrepository.com/artifact/com.microsoft.azure/adal4j -->
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>adal4j</artifactId>
        <version>1.2.0</version>
    </dependency>

    private static AuthenticationResult getAccessTokenFromClientCredentials(String tenantId, String clientId, String clientKey) throws Exception {
            String authority = "https://login.chinacloudapi.cn/" + tenantId;
            AuthenticationContext context = null;
            AuthenticationResult result = null;
            ExecutorService service = null;
            ClientCredential clientCredential = null;
            try {
                service = Executors.newFixedThreadPool(1);
                context = new AuthenticationContext(authority, false, service);
                clientCredential = new ClientCredential(clientId, clientKey);
                Future<AuthenticationResult> future = context.acquireToken(RESOURCE, clientCredential, null);
                result = future.get();
            }
            catch (Exception ex) {
                System.out.println("Something wrong - " + ex.getMessage());
            }
            finally {
                service.shutdown();
            }

            if (result == null) {
                throw new ServiceUnavailableException("authentication result was null");
            }
            return result;
    }

## 通过 REST API 获取机密

调用密钥保管库的 REST API，传入上一步获取到的令牌，就可以拿到前面存入的 Azure 存储连接字符串。

    private static String GetSecretFromKeyVault(String keyVaultUrl, String accessToken)
    {
            HttpClient httpclient = HttpClients.createDefault();

            try {
                URIBuilder builder = new URIBuilder(keyVaultUrl);
                builder.setParameter("api-version", "2016-10-01");
                URI uri = builder.build();

                HttpGet request = new HttpGet(uri);
                request.setHeader("Content-Type", "application/json");
                request.setHeader("Authorization", "Bearer " + accessToken);

                HttpResponse response = httpclient.execute(request);
                HttpEntity entity = response.getEntity();

                if (entity != null) {
                    String secretInJson = EntityUtils.toString(entity);
                    JSONObject secretObj = new JSONObject(secretInJson);
                    return secretObj.getString("value");
                }

                return "";
            } catch (Exception e) {
                System.out.println(e.getMessage());
                return "";
            }
    }

完整代码示例：

- PowerShell 脚本：[https://github.com/wacn/AOG-CodeSample/blob/master/KeyVault/KeyVaultDemo.ps1](https://github.com/wacn/AOG-CodeSample/blob/master/KeyVault/KeyVaultDemo.ps1)
- Java 程序：[https://github.com/wacn/AOG-CodeSample/tree/master/KeyVault/Java/key-vault-native-demo-java](https://github.com/wacn/AOG-CodeSample/tree/master/KeyVault/Java/key-vault-native-demo-java)