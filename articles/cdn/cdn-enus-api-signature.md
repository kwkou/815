<properties
    linkid="dev-net-common-tasks-cdn"
    urlDisplayName="CDN"
    pageTitle="Azure China CDN API doc-signature"
    metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-on, Live Streaming, Streaming media acceleration, CDN acceleration, CDN services, mainstream CDN, live streaming media acceleration, media services, Azure Media Service, cache rules, HLS, CDN technology files, CDN help files, live video acceleration, live broadcast acceleration"
    description="Learn How to Create Live Streaming Acceleration Type CDNs on Azure Management Portal and Default Caching Rules for Live Streaming CDNs"
    metaCanonical=""
    services="cdn"
    documentationCenter=".NET"
    authors="v-jijes"
    solutions=""
    manager=""
    editor="" />
<tags
    ms.service="cdn_en"
    ms.author="v-jijes"
    ms.topic="article"
    ms.date="6/6/2017"
    wacn.date="6/6/2017"
    wacn.lang="en" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-api-signature/)
- [English](/documentation/articles/cdn-enus-api-signature/)

# <a name="cdn-api"></a>CDN API signature mechanism


The Azure CDN will authenticate each request. Requests must be submitted using the HTTPS protocol, and the request must include the signature information.

The Azure CDN uses the key value with the signature parameters to generate the signature by means of the HMAC-SHA 256 hashing algorithm, thereby performing authentication. The signature information is sent as an authorization request header in the format “AzureCDN {Key ID}:{token}.”

You can use the key management section of the [new version of the Azure CDN Management Portal](/documentation/articles/cdn-enus-management-v2-portal-how-to-use/) to apply for and manage the key ID and key value. The **signature parameters** section includes: the absolute path of the request, query parameters sorted in order and joined using “, “, the UTC timestamp for the request, and the request method in upper case. The sections above are joined using carriage returns.

## <a name="authorization"></a>Authorization request header generation example

### <a name="python"></a>Python

    def calculate_authorization_header(request_url, request_timestamp, key_id, key_value, http_method):
        urlparts = urllib.parse.urlparse(request_url)
        queries = urllib.parse.parse_qs(urlparts.query)
        ordered_queries = OrderedDict(queries)
        message = "%s\r\n%s\r\n%s\r\n%s" % (urlparts.path, ", ".join(['%s:%s' % (key, value[0]) for (key, value) in ordered_queries.items()]), request_timestamp, http_method)
        digest = hmac.new(bytearray(key_value, "utf-8"), bytearray(message, "utf-8"), hashlib.sha256).hexdigest().upper()
        return "AzureCDN %s:%s" % (key_id, digest)

### <a name="java"></a>Java

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

### <a name="go"></a>Go

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

### <a name="c-sharp"></a>C Sharp

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

<!--HONumber=May17_HO3-->