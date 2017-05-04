<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="Azure China CDN API doc-signature" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-on, Live Streaming, 流媒体加速, CDN加速,CDN服务,主流CDN, 流媒体直播加速, 媒体服务, Azure Media Service, 缓存规则, HLS, CDN技术文档, CDN帮助文档, 视频直播加速, 直播加速" description="Learn How to create Live Streaming acceleration type CDN on Azure Management Portal and default caching rules for Live Streaming CDN" metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service="cdn"
    ms.date="5/4/2017"
    wacn.date="5/4/2017"
    wacn.lang="cn"
    />
 
# CDN API签名机制

Azure CDN服务会对每个访问请求进行身份验证，需要使用HTTPS协议提交请求，并在请求中包含签名信息。

Azure CDN通过对签名参数使用Key Value进行HMAC-SHA 256散列算法计算生成签名，从而进行身份验证。签名信息以Authorization请求头发送，格式为"AzureCDN {Key ID}:{token}"。

Key ID和Key Value可以到[Azure CDN新版管理门户](https://www.azure.cn/documentation/articles/cdn-management-v2-portal-how-to-use/)的密钥管理处申请和管理。**签名参数**部分包括：请求绝对路径，用 ", "连接起来的排序过的查询参数，请求的UTC时间戳，大写的请求方法。以上几部分用回车换行连接。

## Authorization请求头生成示例

### Python
```
def calculate_authorization_header(request_url, request_timestamp, key_id, key_value, http_method):
    urlparts = urllib.parse.urlparse(request_url)
    queries = urllib.parse.parse_qs(urlparts.query)
    ordered_queries = OrderedDict(queries)
    message = "%s\r\n%s\r\n%s\r\n%s" % (urlparts.path, ", ".join(['%s:%s' % (key, value[0]) for (key, value) in ordered_queries.items()]), request_timestamp, http_method)
    digest = hmac.new(bytearray(key_value, "utf-8"), bytearray(message, "utf-8"), hashlib.sha256).hexdigest().upper()
    return "AzureCDN %s:%s" % (key_id, digest)
```

### Java
```
public static String calculateAuthorizationHeader(String requestURL, String requestTimestamp, String keyID, String keyValue, String httpMethod) throws Exception {
              URL url = new URL(requestURL);
              String path = url.getPath();

              // Get query parameters
              String query = url.getQuery();      
              String[] params = query.split("&");
              Map<String, String> paramMap = new TreeMap<String, String>();
              for(String param: params) {
                    String[] paramterParts = param.split("=");
                    if(paramterParts.length != 2) {
                          continue;
                    }

                    paramMap.put(paramterParts[0], paramterParts[1]);
              }

              String orderedQueries = paramMap.entrySet()
                                              .stream()
                                              .map(entry -> entry.getKey() + ":" + entry.getValue())
                                              .collect(Collectors.joining(", "));

              String content = String.format("%s\r\n%s\r\n%s\r\n%s", path, orderedQueries, requestTimestamp, httpMethod);        
              Mac sha256HMAC = Mac.getInstance("HmacSHA256");
              SecretKeySpec secret_key = new SecretKeySpec(keyValue.getBytes(), "HmacSHA256");
              sha256HMAC.init(secret_key);
              byte[] bytes = sha256HMAC.doFinal(content.getBytes());

              StringBuffer hash = new StringBuffer();
              for (int i = 0; i < bytes.length; i++) {
                  String hex = Integer.toHexString(0xFF & bytes[i]);
                  if (hex.length() == 1) {
                    hash.append('0');
                  }
                  hash.append(hex);
              }

              return String.format("AzureCDN %s:%s", keyID, hash.toString().toUpperCase());
}
```
### Go
```
func calculateAuthorizationHeader(requestURL, requestTimeStamp, keyID, keyValue, httpMethod string) string {
            u, err := url.Parse(requestURL)
            if err != nil {
                panic(err)
            }

            var path = u.Path
            m, _ := url.ParseQuery(u.RawQuery)

            var keys []string
            for k := range m {
                            keys = append(keys, k)
            }
            sort.Strings(keys)
            var orderedQueries []string
            for _, k := range keys {
                            orderedQueries = append(orderedQueries, fmt.Sprintf("%s:%s", k, m[k][0]))
            }

            var queries = strings.Join(orderedQueries, ", ")
            content := fmt.Sprintf("%s\r\n%s\r\n%s\r\n%s", path, queries, requestTimeStamp, httpMethod)
            hash := hmac.New(sha256.New, []byte(keyValue))
            hash.Write([]byte(content))
            digest := strings.ToUpper(hex.EncodeToString(hash.Sum(nil)))
            return fmt.Sprintf("AzureCDN %s:%s", keyID, digest)
 }
```

### C Sharp
```
public static string CalculateAuthorizationHeader(string requestUrl, string requestDate, string key, string httpMethod)
{
          Uri requestUri = new Uri(requestUrl);

          StringBuilder hashContentBuilder = new StringBuilder();
          hashContentBuilder.Append(requestUri.AbsolutePath.ToLowerInvariant());
          hashContentBuilder.Append("\r\n");

          var queryStrings = HttpUtility.ParseQueryString(requestUri.Query);
          var sortedParameterNames = queryStrings.AllKeys.ToList();
          sortedParameterNames.Sort((q1, q2) => string.Compare(q1, q2));
          var result = string.Join(", ", sortedParameterNames.Select(p => string.Format("{0}:{1}", p, queryStrings[p])).ToArray());
          if (!string.IsNullOrEmpty(result))
          {
              hashContentBuilder.Append(result);
              hashContentBuilder.Append("\r\n");
          }

          hashContentBuilder.Append(requestDate);
          hashContentBuilder.Append("\r\n");
          hashContentBuilder.Append(httpMethod.ToUpper());
          string hashContent = hashContentBuilder.ToString();

          using (HMACSHA256 myhmacsha256 = new HMACSHA256(Encoding.UTF8.GetBytes(key)))
          {
              byte[] byteArray = Encoding.UTF8.GetBytes(hashContent);
              byte[] hashedValue = myhmacsha256.ComputeHash(byteArray);

              string sbinary = string.Empty;
              for (int i = 0; i < hashedValue.Length; i++)
              {
                  sbinary += hashedValue[i].ToString("X2");
              }

              return sbinary;
          }
}
```

