<properties
	pageTitle="Azure AD Java 入门 | Azure"
	description="如何生成一个可让用户使用工作或学校帐户登录的 Java Web 应用。"
	services="active-directory"
	documentationCenter="java"
	authors="brandwe"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
        ms.tgt_pltfrm="na"
	ms.devlang="java"
	ms.topic="article"
	ms.date="09/16/2016"
	ms.author="brandwe"
	wacn.date="10/17/2016"/>  



# 使用 Azure AD 执行 Java Web 应用登录和注销

[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide.md)]

使用 Azure AD，只需编写几行代码，就能简单直接地外包 Web 应用的标识管理，提供单一登录和注销。在 Java Web Apps 中，你可以使用社区驱动 ADAL4J 的 Microsoft 实现来达到此目的。

  现在，我们将使用 ADAL4J 来执行以下操作：
- 使用 Azure AD 作为标识提供者将用户登录到应用。
- 显示有关用户的一些信息。
- 从应用中注销用户。

为此，你需要：

1. 将一个应用程序注册到 Azure AD
2. 将应用设置为使用 ADAL4J 库。
3. 使用 ADAL4J 库向 Azure AD 发出登录和注销请求。
4. 列显有关用户的数据。

若要开始，请[下载应用程序框架](https://github.com/Azure-Samples/active-directory-java-webapp-openidconnect/archive/skeleton.zip)或[下载已完成的示例](https://github.com/Azure-Samples/active-directory-java-webapp-openidconnect/archive/complete.zip)。你还需要一个用于注册应用程序的 Azure AD 租户。如果你没有此租户，请[了解如何获取租户](/documentation/articles/active-directory-howto-tenant/)。

## 1\.将应用程序注册到 Azure AD
若要使应用程序对用户进行身份验证，你首先需要在租户中注册新的应用程序。

- 登录到 Azure 管理门户。
- 在左侧的导航栏中单击“Active Directory”。
- 选择你要在其中注册应用程序的租户。
- 单击“应用程序”选项卡，然后在底部抽屉中单击“添加”。
- 根据提示创建一个新的 **Web 应用程序和/或 WebAPI**。
    - 应用程序的**名称**向最终用户描述你的应用程序
    - “登录 URL”是应用程序的基本 URL。框架的默认值为 `http://localhost:8080/adal4jsample/`。
    - “应用程序 ID URI”是应用程序的唯一标识符。约定是使用 `https://<tenant-domain>/<app-name>`，例如 `http://localhost:8080/adal4jsample/`
- 完成注册后，AAD 将为应用程序分配唯一的客户端标识符。在后面的部分中将会用到此值，因此，请从“配置”选项卡复制此值。

进入门户后，为你的应用创建一个**应用程序机密**并复制该机密。稍后您将需要它。


## 2\.使用 Maven 将应用设置为使用 ADAL4J 库和必备组件
在这里，我们要将 ADAL4J 配置为使用 OpenID Connect 身份验证协议。ADAL4J 将用于发出登录和注销请求、管理用户的会话、获取有关用户的信息，等等。

-	在项目的根目录中，打开/创建 `pom.xml`，找到 `// TODO: provide dependencies for Maven` 并替换为以下代码：

Java

	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
		<modelVersion>4.0.0</modelVersion>
		<groupId>com.microsoft.azure</groupId>
		<artifactId>adal4jsample</artifactId>
		<packaging>war</packaging>
		<version>0.0.1-SNAPSHOT</version>
		<name>adal4jsample</name>
		<url>http://maven.apache.org</url>
		<properties>
			<spring.version>3.0.5.RELEASE</spring.version>
		</properties>
	
		<dependencies>
			<dependency>
				<groupId>com.microsoft.azure</groupId>
				<artifactId>adal4j</artifactId>
				<version>1.1.1</version>
			</dependency>
			<dependency>
				<groupId>com.nimbusds</groupId>
				<artifactId>oauth2-oidc-sdk</artifactId>
				<version>4.5</version>
			</dependency>
			<dependency>
				<groupId>org.json</groupId>
				<artifactId>json</artifactId>
				<version>20090211</version>
			</dependency>
			<dependency>
				<groupId>javax.servlet</groupId>
				<artifactId>javax.servlet-api</artifactId>
				<version>3.0.1</version>
				<scope>provided</scope>
			</dependency>
			<dependency>
				<groupId>org.slf4j</groupId>
				<artifactId>slf4j-log4j12</artifactId>
				<version>1.7.5</version>
			</dependency>
			<!-- Spring 3 dependencies -->
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-core</artifactId>
				<version>${spring.version}</version>
			</dependency>
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-web</artifactId>
				<version>${spring.version}</version>
			</dependency>
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-webmvc</artifactId>
				<version>${spring.version}</version>
			</dependency>
		</dependencies>
	
		<build>
			<finalName>sample-for-adal4j</finalName>
			<plugins>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-compiler-plugin</artifactId>
					<configuration>
						<source>1.7</source>
						<target>1.7</target>
						<encoding>UTF-8</encoding>
					</configuration>
				</plugin>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-war-plugin</artifactId>
					<version>2.4</version>
					<configuration>
						<warName>${project.artifactId}</warName>
						<source>${project.basedir}\src</source>
						<target>${maven.compiler.target}</target>
						<encoding>utf-8</encoding>
					</configuration>
				</plugin>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-dependency-plugin</artifactId>
					<executions>
						<execution>
							<id>install</id>
							<phase>install</phase>
							<goals>
								<goal>sources</goal>
							</goals>
						</execution>
					</executions>
				</plugin>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-resources-plugin</artifactId>
					<version>2.5</version>
					<configuration>
						<encoding>UTF-8</encoding>
					</configuration>
				</plugin>
			</plugins>
		</build>
	
	</project>

## 3\.创建 Java Web 应用程序文件 (WEB-INF)

在这里，我们要将 Java Web 应用配置为使用 OpenID Connect 身份验证协议。ADAL4J 库将用于发出登录和注销请求、管理用户的会话、获取有关用户的信息，等等。

-	首先，打开位于 `\webapp\WEB-INF` 中的 `web.xml` 文件，并在 xml 中输入应用的配置值。

该文件应如下所示：

xml

	<?xml version="1.0"?>
	<web-app id="WebApp_ID" version="2.4"
		xmlns="http://java.sun.com/xml/ns/j2ee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee 
	    http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
		<display-name>Archetype Created Web Application</display-name>
		<context-param>
			<param-name>authority</param-name>
			<param-value>https://login.chinacloudapi.cn/</param-value>
		</context-param>
		<context-param>
			<param-name>tenant</param-name>
			<param-value>YOUR_TENANT_NAME</param-value>
		</context-param>
	
		<filter>
			<filter-name>BasicFilter</filter-name>
			<filter-class>com.microsoft.aad.adal4jsample.BasicFilter</filter-class>
			<init-param>
				<param-name>client_id</param-name>
				<param-value>YOUR_CLIENT_ID</param-value>
			</init-param>
			<init-param>
				<param-name>secret_key</param-name>
				<param-value>YOUR_CLIENT_SECRET</param-value>
			</init-param>
		</filter>
		<filter-mapping>
			<filter-name>BasicFilter</filter-name>
			<url-pattern>/secure/*</url-pattern>
		</filter-mapping>
	
		<servlet>
			<servlet-name>mvc-dispatcher</servlet-name>
			<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
			<load-on-startup>1</load-on-startup>
		</servlet>
	
		<servlet-mapping>
			<servlet-name>mvc-dispatcher</servlet-name>
			<url-pattern>/</url-pattern>
		</servlet-mapping>
	
		<context-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/mvc-dispatcher-servlet.xml</param-value>
		</context-param>
	
		<listener>
			<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
		</listener>
	</web-app>




    -	The `YOUR_CLIENT_ID` is the **Application Id** assigned to your app in the registration portal.
    -	The `YOUR_CLIENT_SECRET` is the **Application Secret**  you created in the portal.
    -   The `YOUR_TENANT_NAME` is the **tenant name** of your app, e.g. contoso.partner.onmschina.cn

将其余的配置参数保持不变。

> [AZURE.NOTE]
从 XML 文件中可以看到，我们要编写一个名为 `mvc-dispatcher` 的 JSP/Servlet Web 应用，每当我们访问 /secure URL 时，该应用就会使用 `BasicFilter`。在编写的代码的其余部分中，可以看到我们将使用 /secure 作为受保护内容的所在位置，并强制向 Azure Active Directory 进行身份验证。

-	接下来，在 `\webapp\WEB-INF` 中创建 `mvc-dispatcher-servlet.xml` 文件并输入以下内容：

xml
	
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:context="http://www.springframework.org/schema/context"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="
	        http://www.springframework.org/schema/beans     
	        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
	        http://www.springframework.org/schema/context 
	        http://www.springframework.org/schema/context/spring-context-3.0.xsd">
	
		<context:component-scan base-package="com.microsoft.aad.adal4jsample" />
	
		<bean
			class="org.springframework.web.servlet.view.InternalResourceViewResolver">
			<property name="prefix">
				<value>/</value>
			</property>
			<property name="suffix">
				<value>.jsp</value>
			</property>
		</bean>
	
	</beans>


这将告知 Web 应用要使用 Spring 以及在哪里查找我们在下面编写的 .jsp 文件。

## 4\.创建 Java JSP 视图文件（适用于 BasicFilter MVC）

在 WEB-INF 中设置 Web 应用的过程目前只完成了一半。接下来，需要创建 Web 应用将要执行的实际 Java Server 页面文件（已经在配置中提示）。

如果你记得的话，我们已在 XML 配置文件中告知 Java 某个 `/` 资源应该加载 .jsp 文件，并且某个 `/secure` 资源应该通过名为 `BasicFilter` 的筛选器。

现在让我们完成这些步骤。

-	首先，请在 `\webapp` 中创建 `index.jsp` 文件，并剪切/粘贴以下内容：

jsp

	<html>
	<body>
		<h2>Hello World!</h2>
		<ul>
		<li><a href="secure/aad">Secure Page</a></li>
		</ul>
	</body>
	</html>



这只会重定向到筛选器保护的安全页。

- 接下来，在同一个目录中创建 `error.jsp` 文件，用于捕获可能发生的任何错误：

jsp

	<html>
	<body>
		<h2>ERROR PAGE!</h2>
		<p>
			Exception -
			<%=request.getAttribute("error")%></p>
		<ul>
			<li><a href="<%=request.getContextPath()%>/index.jsp">Go Home</a></li>
		</ul>
	</body>
	</html>


- 最后，在 `\webapp` 中创建名为 `\secure` 的文件夹，以生成所需的安全网页，因此，目录现在为 `\webapp\secure`。

- 在此目录下，随后创建 `aad.jsp` 文件，并剪切/粘贴以下内容：
jsp

		<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
		<html>
		<head>
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
		<title>AAD Secure Page</title>
		</head>
		<body>			
			<h1>Directory - Users List</h1>
			<p>${users}</p>
			<ul>
				<li><a href="<%=request.getContextPath()%>/secure/aad?cc=1">Get
					new Access Token via Client Credentials</a></li>
			</ul>
			<ul>
				<li><a href="<%=request.getContextPath()%>/secure/aad?refresh=1">Get
					new Access Token via Refresh Token</a></li>
			</ul>
			<ul>
				<li><a href="<%=request.getContextPath()%>/index.jsp">Go Home</a></li>
			</ul>
		</body>
		</html>


你会看到，此页将重定向到 BasicFilter Servlet 将要读取的特定请求，然后使用 `ADAJ4J` 库执行请求。很简单，是吧？

当然，现在需要设置 Java 文件，以便 Servlet 可以执行其工作。

## 5\.创建一些 Java 帮助器文件（适用于 BasicFilter MVC）

我们的目标是创建一些 Java 文件，用于：

1. 允许用户登录和注销
2. 获取有关用户的一些数据。

> [AZURE.NOTE] 
为了获取有关用户的数据，我们必须使用 Azure Active Directory 中的图形 API。图形 API 是安全的 Web 服务，可用于检索有关组织（包括个人用户）的数据。这比预先在令牌中填充敏感数据要好，因为这可以确保请求数据的用户已获授权，并且意外获得令牌（从越狱的手机或台式机上的 Web 浏览器缓存）的任何用户不会获取大量有关用户或组织的重要详细信息。

让我们编写一些 Java 文件来执行此工作：

1. 在名为“adal4jsample”的根目录中创建一个文件夹用于存储所有 Java 文件。

我们将在 Java 文件中使用命名空间 `com.microsoft.aad.adal4jsample`。大多数 IDE 将为此创建嵌套的文件夹结构（例如 `/com/microsoft/aad/adal4jsample`）。你可以根据需要创建这种结构，但不一定要这样做。

2. 在此文件夹中创建名为 `JSONHelper.java` 的文件，我们将用它来根据令牌中的 JSON 数据帮助进行分析。可以从以下内容中剪切/粘贴此信息：

Java
		
		package com.microsoft.aad.adal4jsample;
		
		import java.lang.reflect.Field;
		import java.util.Arrays;
		import java.util.Enumeration;
		import java.util.List;
		
		import javax.servlet.http.HttpServletRequest;
		
		import org.apache.commons.lang3.text.WordUtils;
		import org.apache.log4j.Logger;
		import org.json.JSONArray;
		import org.json.JSONException;
		import org.json.JSONObject;
		
		/**
		 * This class provides the methods to parse JSON Data from a JSON Formatted
		 * String.
		 * 
		 * @author Azure Active Directory Contributor
		 * 
		 */
		public class JSONHelper {
		
		    private static Logger logger = Logger.getLogger(JSONHelper.class);
		
		    JSONHelper() {
		        // PropertyConfigurator.configure("log4j.properties");
		    }
		
		    /**
		     * This method parses an JSON Array out of a collection of JSON Objects
		     * within a string.
		     * 
		     * @param jSonData
		     *            The JSON String that holds the collection.
		     * @return An JSON Array that would contains all the collection object.
		     * @throws Exception
		     */
		    public static JSONArray fetchDirectoryObjectJSONArray(JSONObject jsonObject) throws Exception {
		        JSONArray jsonArray = new JSONArray();
		        jsonArray = jsonObject.optJSONObject("responseMsg").optJSONArray("value");
		        return jsonArray;
		    }
		
		    /**
		     * This method parses an JSON Object out of a collection of JSON Objects
		     * within a string
		     * 
		     * @param jsonObject
		     * @return An JSON Object that would contains the DirectoryObject.
		     * @throws Exception
		     */
		    public static JSONObject fetchDirectoryObjectJSONObject(JSONObject jsonObject) throws Exception {
		        JSONObject jObj = new JSONObject();
		        jObj = jsonObject.optJSONObject("responseMsg");
		        return jObj;
		    }
		
		    /**
		     * This method parses the skip token from a json formatted string.
		     * 
		     * @param jsonData
		     *            The JSON Formatted String.
		     * @return The skipToken.
		     * @throws Exception
		     */
		    public static String fetchNextSkiptoken(JSONObject jsonObject) throws Exception {
		        String skipToken = "";
		        // Parse the skip token out of the string.
		        skipToken = jsonObject.optJSONObject("responseMsg").optString("odata.nextLink");
		
		        if (!skipToken.equalsIgnoreCase("")) {
		            // Remove the unnecessary prefix from the skip token.
		            int index = skipToken.indexOf("$skiptoken=") + (new String("$skiptoken=")).length();
		            skipToken = skipToken.substring(index);
		        }
		        return skipToken;
		    }
		
		    /**
		     * @param jsonObject
		     * @return
		     * @throws Exception
		     */
		    public static String fetchDeltaLink(JSONObject jsonObject) throws Exception {
		        String deltaLink = "";
		        // Parse the skip token out of the string.
		        deltaLink = jsonObject.optJSONObject("responseMsg").optString("aad.deltaLink");
		        if (deltaLink == null || deltaLink.length() == 0) {
		            deltaLink = jsonObject.optJSONObject("responseMsg").optString("aad.nextLink");
		            logger.info("deltaLink empty, nextLink ->" + deltaLink);
		
		        }
		        if (!deltaLink.equalsIgnoreCase("")) {
		            // Remove the unnecessary prefix from the skip token.
		            int index = deltaLink.indexOf("deltaLink=") + (new String("deltaLink=")).length();
		            deltaLink = deltaLink.substring(index);
		        }
		        return deltaLink;
		    }
		
		    /**
		     * This method would create a string consisting of a JSON document with all
		     * the necessary elements set from the HttpServletRequest request.
		     * 
		     * @param request
		     *            The HttpServletRequest
		     * @return the string containing the JSON document.
		     * @throws Exception
		     *             If there is any error processing the request.
		     */
		    public static String createJSONString(HttpServletRequest request, String controller) throws Exception {
		        JSONObject obj = new JSONObject();
		        try {
		            Field[] allFields = Class.forName(
		                    "com.microsoft.windowsazure.activedirectory.sdk.graph.models." + controller).getDeclaredFields();
		            String[] allFieldStr = new String[allFields.length];
		            for (int i = 0; i < allFields.length; i++) {
		                allFieldStr[i] = allFields[i].getName();
		            }
		            List<String> allFieldStringList = Arrays.asList(allFieldStr);
		            Enumeration<String> fields = request.getParameterNames();
		
		            while (fields.hasMoreElements()) {
		
		                String fieldName = fields.nextElement();
		                String param = request.getParameter(fieldName);
		                if (allFieldStringList.contains(fieldName)) {
		                    if (param == null || param.length() == 0) {
		                        if (!fieldName.equalsIgnoreCase("password")) {
		                            obj.put(fieldName, JSONObject.NULL);
		                        }
		                    } else {
		                        if (fieldName.equalsIgnoreCase("password")) {
		                            obj.put("passwordProfile", new JSONObject("{\"password\": \"" + param + "\"}"));
		                        } else {
		                            obj.put(fieldName, param);
		
		                        }
		                    }
		                }
		            }
		        } catch (JSONException e) {
		            e.printStackTrace();
		        } catch (SecurityException e) {
		            e.printStackTrace();
		        } catch (ClassNotFoundException e) {
		            e.printStackTrace();
		        }
		        return obj.toString();
		    }
		
		    /**
		     * 
		     * @param key
		     * @param value
		     * @return string format of this JSON obje
		     * @throws Exception
		     */
		    public static String createJSONString(String key, String value) throws Exception {
		
		        JSONObject obj = new JSONObject();
		        try {
		            obj.put(key, value);
		        } catch (JSONException e) {
		            e.printStackTrace();
		        }
		
		        return obj.toString();
		    }
		
		    /**
		     * This is a generic method that copies the simple attribute values from an
		     * argument jsonObject to an argument generic object.
		     * 
		     * @param jsonObject
		     *            The jsonObject from where the attributes are to be copied.
		     * @param destObject
		     *            The object where the attributes should be copied into.
		     * @throws Exception
		     *             Throws a Exception when the operation are unsuccessful.
		     */
		    public static <T> void convertJSONObjectToDirectoryObject(JSONObject jsonObject, T destObject) throws Exception {
		
		        // Get the list of all the field names.
		        Field[] fieldList = destObject.getClass().getDeclaredFields();
		
		        // For all the declared field.
		        for (int i = 0; i < fieldList.length; i++) {
		            // If the field is of type String, that is
		            // if it is a simple attribute.
		            if (fieldList[i].getType().equals(String.class)) {
		                // Invoke the corresponding set method of the destObject using
		                // the argument taken from the jsonObject.
		                destObject
		                        .getClass()
		                        .getMethod(String.format("set%s", WordUtils.capitalize(fieldList[i].getName())),
		                                new Class[] { String.class })
		                        .invoke(destObject, new Object[] { jsonObject.optString(fieldList[i].getName()) });
		            }
		        }
		    }
		
		    public static JSONArray joinJSONArrays(JSONArray a, JSONArray b) {
		        JSONArray comb = new JSONArray();
		        for (int i = 0; i < a.length(); i++) {
		            comb.put(a.optJSONObject(i));
		        }
		        for (int i = 0; i < b.length(); i++) {
		            comb.put(b.optJSONObject(i));
		        }
		        return comb;
		    }
		
		}



3. 接下来，创建名为 `HttpClientHelper.java` 的文件，我们将用它来根据 AAD 终结点中的 HTTP 数据帮助进行分析。可以从以下内容中剪切/粘贴此信息：

Java
		
		package com.microsoft.aad.adal4jsample;
		
		import java.io.BufferedReader;
		import java.io.ByteArrayOutputStream;
		import java.io.IOException;
		import java.io.InputStream;
		import java.io.InputStreamReader;
		import java.io.OutputStreamWriter;
		import java.net.HttpURLConnection;
		
		import org.json.JSONException;
		import org.json.JSONObject;
		
		/**
		 * This is Helper class for all RestClient class.
		 * 
		 * @author Azure Active Directory Contributor
		 * 
		 */
		public class HttpClientHelper {
		
		    public HttpClientHelper() {
		        super();
		    }
		
		    public static String getResponseStringFromConn(HttpURLConnection conn, boolean isSuccess) throws IOException {
		
		        BufferedReader reader = null;
		        if (isSuccess) {
		            reader = new BufferedReader(new InputStreamReader(conn.getInputStream()));
		        } else {
		            reader = new BufferedReader(new InputStreamReader(conn.getErrorStream()));
		        }
		        StringBuffer stringBuffer = new StringBuffer();
		        String line = "";
		        while ((line = reader.readLine()) != null) {
		            stringBuffer.append(line);
		        }
		
		        return stringBuffer.toString();
		    }
		
		    public static String getResponseStringFromConn(HttpURLConnection conn, String payLoad) throws IOException {
		
		        // Send the http message payload to the server.
		        if (payLoad != null) {
		            conn.setDoOutput(true);
		            OutputStreamWriter osw = new OutputStreamWriter(conn.getOutputStream());
		            osw.write(payLoad);
		            osw.flush();
		            osw.close();
		        }
		
		        // Get the message response from the server.
		        BufferedReader br = new BufferedReader(new InputStreamReader(conn.getInputStream()));
		        String line = "";
		        StringBuffer stringBuffer = new StringBuffer();
		        while ((line = br.readLine()) != null) {
		            stringBuffer.append(line);
		        }
		
		        br.close();
		
		        return stringBuffer.toString();
		    }
		
		    public static byte[] getByteaArrayFromConn(HttpURLConnection conn, boolean isSuccess) throws IOException {
		
		        InputStream is = conn.getInputStream();
		        ByteArrayOutputStream baos = new ByteArrayOutputStream();
		        byte[] buff = new byte[1024];
		        int bytesRead = 0;
		
		        while ((bytesRead = is.read(buff, 0, buff.length)) != -1) {
		            baos.write(buff, 0, bytesRead);
		        }
		
		        byte[] bytes = baos.toByteArray();
		        baos.close();
		        return bytes;
		    }
		
		    /**
		     * for bad response, whose responseCode is not 200 level
		     * 
		     * @param responseCode
		     * @param errorCode
		     * @param errorMsg
		     * @return
		     * @throws JSONException
		     */
		    public static JSONObject processResponse(int responseCode, String errorCode, String errorMsg) throws JSONException {
		        JSONObject response = new JSONObject();
		        response.put("responseCode", responseCode);
		        response.put("errorCode", errorCode);
		        response.put("errorMsg", errorMsg);
		
		        return response;
		    }
		
		    /**
		     * for bad response, whose responseCode is not 200 level
		     * 
		     * @param responseCode
		     * @param errorCode
		     * @param errorMsg
		     * @return
		     * @throws JSONException
		     */
		    public static JSONObject processGoodRespStr(int responseCode, String goodRespStr) throws JSONException {
		        JSONObject response = new JSONObject();
		        response.put("responseCode", responseCode);
		        if (goodRespStr.equalsIgnoreCase("")) {
		            response.put("responseMsg", "");
		        } else {
		            response.put("responseMsg", new JSONObject(goodRespStr));
		        }
		
		        return response;
		    }
		
		    /**
		     * for good response
		     * 
		     * @param responseCode
		     * @param responseMsg
		     * @return
		     * @throws JSONException
		     */
		    public static JSONObject processBadRespStr(int responseCode, String responseMsg) throws JSONException {
		
		        JSONObject response = new JSONObject();
		        response.put("responseCode", responseCode);
		        if (responseMsg.equalsIgnoreCase("")) { // good response is empty string
		            response.put("responseMsg", "");
		        } else { // bad response is json string
		            JSONObject errorObject = new JSONObject(responseMsg).optJSONObject("odata.error");
		
		            String errorCode = errorObject.optString("code");
		            String errorMsg = errorObject.optJSONObject("message").optString("value");
		            response.put("responseCode", responseCode);
		            response.put("errorCode", errorCode);
		            response.put("errorMsg", errorMsg);
		        }
		
		        return response;
		    }
		
		}



## 6\.创建 Java 图形 API 模型文件（适用于 BasicFilter MVC）

如上所述，我们将使用图形 API 来获取有关已登录的用户的数据。为了顺利进行，我们应该创建一个表示**目录对象**的文件以及一个表示**用户**的单独文件，如此便可以使用 Java 的 OO 模式。

1. 创建一个名为 `DirectoryObject.java` 的文件，我们将用它来存储有关任何 DirectoryObject 的基本数据（你稍后可以随意使用它来执行任何其他图形查询）。可以从以下内容中剪切/粘贴此信息：

Java
		
		package com.microsoft.aad.adal4jsample;
		
		/**
		 * @author Azure Active Directory Contributor
		 *
		 */
		public abstract class DirectoryObject {
			
			public DirectoryObject() {
				super();
			}
			
			/**
			 * 
			 * @return
			 */
			public abstract String getObjectId();
			
			/**
			 * @param objectId
			 */
			public abstract void setObjectId(String objectId);
		
			/**
			 * 
			 * @return
			 */
			public abstract String getObjectType();
		
			/**
			 * 
			 * @param objectType
			 */
			public abstract void setObjectType(String objectType);
			
			/**
			 * 
			 * @return
			 */
			public abstract String getDisplayName();
		
			/**
			 * 
			 * @param displayName
			 */
			public abstract void setDisplayName(String displayName);
		
		}



2. 创建名为 `User.java` 的文件，我们将用它来存储有关目录中任何用户的基本数据。同样，这是针对目录数据的基本 getter/setter，因此可以从以下内容中剪切/粘贴此信息：

Java
		
		package com.microsoft.aad.adal4jsample;
		
		import java.security.acl.Group;
		import java.util.ArrayList;
		
		import javax.xml.bind.annotation.XmlRootElement;
		
		import org.json.JSONObject;
		
		/**
		 *  The User Class holds together all the members of a WAAD User entity and all the access methods and set methods
		 *  @author Azure Active Directory Contributor
		 */
		@XmlRootElement
		public class User extends DirectoryObject{
			
			// The following are the individual private members of a User object that holds
			// a particular simple attribute of an User object.
			protected String objectId;
			protected String objectType;
			protected String accountEnabled;
			protected String city;
			protected String country;
			protected String department;
			protected String dirSyncEnabled;
			protected String displayName;
			protected String facsimileTelephoneNumber;
			protected String givenName;
			protected String jobTitle;
			protected String lastDirSyncTime;
			protected String mail;
			protected String mailNickname;
			protected String mobile;
			protected String password;
			protected String passwordPolicies;
			protected String physicalDeliveryOfficeName;
			protected String postalCode;
			protected String preferredLanguage;
			protected String state;
			protected String streetAddress;
			protected String surname;
			protected String telephoneNumber;
			protected String usageLocation;
			protected String userPrincipalName;
			protected boolean isDeleted;  // this will move to dto
		
			/**
			 * below 4 properties are for future use
			 */
			// managerDisplayname of this user
			protected String managerDisplayname;
			
			// The directReports holds a list of directReports
			private ArrayList<User> directReports;
			
			// The groups holds a list of group entity this user belongs to. 
			private ArrayList<Group> groups;
			
			// The roles holds a list of role entity this user belongs to. 
			private ArrayList<Group> roles;
			
			
			/**
			 * The constructor for the User class. Initializes the dynamic lists and managerDisplayname variables.
			 */
			public User(){
				directReports = null;
				groups = new ArrayList<Group>();
				roles = new ArrayList<Group>();
				managerDisplayname = null;
			}
		//	
		//	public User(String displayName, String objectId){
		//		setDisplayName(displayName);
		//		setObjectId(objectId);
		//	}
		//	
		//	public User(String displayName, String objectId, String userPrincipalName, String accountEnabled){
		//		setDisplayName(displayName);
		//		setObjectId(objectId);
		//		setUserPrincipalName(userPrincipalName);
		//		setAccountEnabled(accountEnabled);
		//	}
		//	
		
			/**
			 * @return The objectId of this user.
			 */
			public String getObjectId() {
				return objectId;
			}
			
			/**
			 * @param objectId The objectId to set to this User object.
			 */
			public void setObjectId(String objectId) {
				this.objectId = objectId;
			}
		
		
			/**
			 * @return The objectType of this User.
			 */
			public String getObjectType() {
				return objectType;
			}
		
			/**
			 * @param objectType The objectType to set to this User object.
			 */
			public void setObjectType(String objectType) {
				this.objectType = objectType;
			}
		
			/**
			 * @return The userPrincipalName of this User.
			 */
			public String getUserPrincipalName() {
				return userPrincipalName;
			}
		
			/**
			 * @param userPrincipalName The userPrincipalName to set to this User object.
			 */
			public void setUserPrincipalName(String userPrincipalName) {
				this.userPrincipalName = userPrincipalName;
			}
		
			
			/**
			 * @return The usageLocation of this User.
			 */
			public String getUsageLocation() {
				return usageLocation;
			}
		
			/**
			 * @param usageLocation The usageLocation to set to this User object.
			 */
			public void setUsageLocation(String usageLocation) {
				this.usageLocation = usageLocation;
			}
		
			/**
			 * @return The telephoneNumber of this User.
			 */
			public String getTelephoneNumber() {
				return telephoneNumber;
			}
		
			/**
			 * @param telephoneNumber The telephoneNumber to set to this User object.
			 */
			public void setTelephoneNumber(String telephoneNumber) {
				this.telephoneNumber = telephoneNumber;
			}
		
			/**
			 * @return The surname of this User.
			 */
			public String getSurname() {
				return surname;
			}
		
			/**
			 * @param surname The surname to set to this User Object.
			 */
			public void setSurname(String surname) {
				this.surname = surname;
			}
		
			/**
			 * @return The streetAddress of this User.
			 */
			public String getStreetAddress() {
				return streetAddress;
			}
		
			/**
			 * @param streetAddress The streetAddress to set to this User.
			 */
			public void setStreetAddress(String streetAddress) {
				this.streetAddress = streetAddress;
			}
		
			/**
			 * @return The state of this User.
			 */
			public String getState() {
				return state;
			}
		
			/**
			 * @param state The state to set to this User object.
			 */
			public void setState(String state) {
				this.state = state;
			}
		
			/**
			 * @return The preferredLanguage of this User.
			 */
			public String getPreferredLanguage() {
				return preferredLanguage;
			}
		
			/**
			 * @param preferredLanguage The preferredLanguage to set to this User.
			 */
			public void setPreferredLanguage(String preferredLanguage) {
				this.preferredLanguage = preferredLanguage;
			}
		
			/**
			 * @return The postalCode of this User.
			 */
			public String getPostalCode() {
				return postalCode;
			}
		
			/**
			 * @param postalCode The postalCode to set to this User.
			 */
			public void setPostalCode(String postalCode) {
				this.postalCode = postalCode;
			}
		
			/**
			 * @return The physicalDeliveryOfficeName of this User.
			 */
			public String getPhysicalDeliveryOfficeName() {
				return physicalDeliveryOfficeName;
			}
		
			/**
			 * @param physicalDeliveryOfficeName The physicalDeliveryOfficeName to set to this User Object.
			 */
			public void setPhysicalDeliveryOfficeName(String physicalDeliveryOfficeName) {
				this.physicalDeliveryOfficeName = physicalDeliveryOfficeName;
			}
		
			/**
			 * @return The passwordPolicies of this User.
			 */
			public String getPasswordPolicies() {
				return passwordPolicies;
			}
		
			/**
			 * @param passwordPolicies The passwordPolicies to set to this User object.
			 */
			public void setPasswordPolicies(String passwordPolicies) {
				this.passwordPolicies = passwordPolicies;
			}
		
			/**
			 * @return The mobile of this User.
			 */
			public String getMobile() {
				return mobile;
			}
		
			/**
			 * @param mobile The mobile to set to this User object.
			 */
			public void setMobile(String mobile) {
				this.mobile = mobile;
			}
			
			/**
			 * @return The Password of this User.
			 */
			public String getPassword() {
				return password;
			}
		
			/**
			 * @param password The mobile to set to this User object.
			 */
			public void setPassword(String password) {
				this.password = password;
			}
		
			/**
			 * @return The mail of this User.
			 */
			public String getMail() {
				return mail;
			}
		
			/**
			 * @param mail The mail to set to this User object.
			 */
			public void setMail(String mail) {
				this.mail = mail;
			}
			
			/**
			 * @return The MailNickname of this User.
			 */
			public String getMailNickname() {
				return mailNickname;
			}
		
			/**
			 * @param mail The MailNickname to set to this User object.
			 */
			public void setMailNickname(String mailNickname) {
				this.mailNickname = mailNickname;
			}
		
		
			/**
			 * @return The jobTitle of this User.
			 */
			public String getJobTitle() {
				return jobTitle;
			}
		
			/**
			 * @param jobTitle The jobTitle to set to this User Object.
			 */
			public void setJobTitle(String jobTitle) {
				this.jobTitle = jobTitle;
			}
		
			/**
			 * @return The givenName of this User.
			 */
			public String getGivenName() {
				return givenName;
			}
		
			/**
			 * @param givenName The givenName to set to this User.
			 */
			public void setGivenName(String givenName) {
				this.givenName = givenName;
			}
		
			/**
			 * @return The facsimileTelephoneNumber of this User.
			 */
			public String getFacsimileTelephoneNumber() {
				return facsimileTelephoneNumber;
			}
		
			/**
			 * @param facsimileTelephoneNumber The facsimileTelephoneNumber to set to this User Object.
			 */
			public void setFacsimileTelephoneNumber(String facsimileTelephoneNumber) {
				this.facsimileTelephoneNumber = facsimileTelephoneNumber;
			}
		
			/**
			 * @return The displayName of this User.
			 */
			public String getDisplayName() {
				return displayName;
			}
		
			/**
			 * @param displayName The displayName to set to this User Object.
			 */
			public void setDisplayName(String displayName) {
				this.displayName = displayName;
			}
		
			/**
			 * @return The dirSyncEnabled of this User.
			 */
			public String getDirSyncEnabled() {
				return dirSyncEnabled;
			}
		
			/**
			 * @param dirSyncEnabled The dirSyncEnabled to set to this User.
			 */
			public void setDirSyncEnabled(String dirSyncEnabled) {
				this.dirSyncEnabled = dirSyncEnabled;
			}
		
			/**
			 * @return The department of this User.
			 */
			public String getDepartment() {
				return department;
			}
		
			/**
			 * @param department The department to set to this User.
			 */
			public void setDepartment(String department) {
				this.department = department;
			}
		
			/**
			 * @return The lastDirSyncTime of this User.
			 */
			public String getLastDirSyncTime() {
				return lastDirSyncTime;
			}
		
			/**
			 * @param lastDirSyncTime The lastDirSyncTime to set to this User.
			 */
			public void setLastDirSyncTime(String lastDirSyncTime) {
				this.lastDirSyncTime = lastDirSyncTime;
			}
		
			/**
			 * @return The country of this User.
			 */
			public String getCountry() {
				return country;
			}
		
			/**
			 * @param country The country to set to this User.
			 */
			public void setCountry(String country) {
				this.country = country;
			}
		
			/**
			 * @return The city of this User.
			 */
			public String getCity() {
				return city;
			}
		
			/**
			 * @param city The city to set to this User.
			 */
			public void setCity(String city) {
				this.city = city;
			}
		
			/**
			 * @return The accountEnabled attribute of this User.
			 */
			public String getAccountEnabled() {
				return accountEnabled;
			}
		
			/**
			 * @param accountEnabled The accountEnabled to set to this User.
			 */
			public void setAccountEnabled(String accountEnabled) {
				this.accountEnabled = accountEnabled;
			}
			
			public boolean isIsDeleted() {
				return this.isDeleted;
			}
		
			public void setIsDeleted(boolean isDeleted) {
				this.isDeleted = isDeleted;
			}
		
			@Override
			public String toString() {
				return new JSONObject(this).toString();
			}
			
			public String getManagerDisplayname(){
				return managerDisplayname;
			}
			
			public void setManagerDisplayname(String managerDisplayname){
				this.managerDisplayname = managerDisplayname;
			}
		}
		
		
		/**
		 * The Class DirectReports Holds the essential data for a single DirectReport entry. Namely,
		 * it holds the displayName and the objectId of the direct entry. Furthermore, it provides the
		 * access methods to set or get the displayName and the ObjectId of this entry.
		 */
		//class DirectReport extends User{
		//
		//	private String displayName;
		//	private String objectId;
		//	 
		//	/**
		//	 * Two arguments Constructor for the DirectReport Class.
		//	 * @param displayName
		//	 * @param objectId
		//	 */
		//	public DirectReport(String displayName, String objectId){
		//		this.displayName = displayName;
		//		this.objectId = objectId;
		//	}
		//
		//	/**
		//	 * @return The diaplayName of this direct report entry.
		//	 */
		//	public String getDisplayName() {
		//		return displayName;
		//	}
		//
		//	
		//	/**
		//	 *  @return The objectId of this direct report entry. 
		//	 */
		//	public String getObjectId() {
		//		return objectId;
		//	}
		//
		//}



## 7\.创建身份验证模型/控制器文件（适用于 BasicFilter）

没错，Java 确实相当冗长，但我们很快就要完成了。在完成最后一个步骤之前，让我们再编写一些 `ADAL4J` 库所需的帮助器文件，然后编写 BasicFilter Servlet 来处理请求。

1. 创建名为 `AuthHelper.java` 的文件，这可以提供用于确定已登录用户状态的方法。其中包括：

- `isAuthenticated()` 方法，用于返回用户是否已登录的结果
- `containsAuthenticationData()`，告知令牌是否包含数据
- `isAuthenticationSuccessful()`，告知用户身份验证是否成功。

剪切/粘贴以下代码：

Java

	package com.microsoft.aad.adal4jsample;
	
	import java.util.Map;
	
	import javax.servlet.http.HttpServletRequest;
	
	import com.microsoft.aad.adal4j.AuthenticationResult;
	import com.nimbusds.openid.connect.sdk.AuthenticationResponse;
	import com.nimbusds.openid.connect.sdk.AuthenticationResponseParser;
	import com.nimbusds.openid.connect.sdk.AuthenticationSuccessResponse;
	
	public final class AuthHelper {
	
	    public static final String PRINCIPAL_SESSION_NAME = "principal";
	
	    private AuthHelper() {
	    }
	
	    public static boolean isAuthenticated(HttpServletRequest request) {
	        return request.getSession().getAttribute(PRINCIPAL_SESSION_NAME) != null;
	    }
	
	    public static AuthenticationResult getAuthSessionObject(
	            HttpServletRequest request) {
	        return (AuthenticationResult) request.getSession().getAttribute(
	                PRINCIPAL_SESSION_NAME);
	    }
	
	    public static boolean containsAuthenticationData(
	            HttpServletRequest httpRequest) {
	        Map<String, String[]> map = httpRequest.getParameterMap();
	        return httpRequest.getMethod().equalsIgnoreCase("POST") && (httpRequest.getParameterMap().containsKey(
	                        AuthParameterNames.ERROR)
	                        || httpRequest.getParameterMap().containsKey(
	                                AuthParameterNames.ID_TOKEN) || httpRequest
	                        .getParameterMap().containsKey(AuthParameterNames.CODE));
	    }
	
	    public static boolean isAuthenticationSuccessful(
	            AuthenticationResponse authResponse) {
	        return authResponse instanceof AuthenticationSuccessResponse;
	    }
	}


2. 创建名为 `AuthParameterNames.java` 的文件，用于提供 `ADAL4J` 所需的一些不可变变量。剪切/粘贴以下内容：

Java

	package com.microsoft.aad.adal4jsample;
	
	public final class AuthParameterNames {
	
	    private AuthParameterNames() {
	    }
	
	    public static String ERROR = "error";
	    public static String ERROR_DESCRIPTION = "error_description";
	    public static String ERROR_URI = "error_uri";
	    public static String ID_TOKEN = "id_token";
	    public static String CODE = "code";
	}


3. 最后，创建名为 `AadController.java` 的文件，这是 MVC 模式的控制器，可提供 JSP 控制器，并公开应用的 `secure/aad` URL 终结点。此外，我们还要将图形查询放在此文件中。

剪切/粘贴以下内容：

Java

	package com.microsoft.aad.adal4jsample;
	
	import java.net.HttpURLConnection;
	import java.net.URL;
	
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpSession;
	
	import org.json.JSONArray;
	import org.json.JSONObject;
	import org.springframework.stereotype.Controller;
	import org.springframework.ui.ModelMap;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RequestMethod;
	
	import com.microsoft.aad.adal4j.AuthenticationResult;
	
	@Controller
	@RequestMapping("/secure/aad")
	public class AadController {
	
	    @RequestMapping(method = { RequestMethod.GET, RequestMethod.POST })
	    public String getDirectoryObjects(ModelMap model, HttpServletRequest httpRequest) {
	        HttpSession session = httpRequest.getSession();
	        AuthenticationResult result = (AuthenticationResult) session.getAttribute(AuthHelper.PRINCIPAL_SESSION_NAME);
	        if (result == null) {
	            model.addAttribute("error", new Exception("AuthenticationResult not found in session."));
	            return "/error";
	        } else {
	            String data;
	            try {
	                data = this.getUsernamesFromGraph(result.getAccessToken(), session.getServletContext()
	                        .getInitParameter("tenant"));
	                model.addAttribute("users", data);
	            } catch (Exception e) {
	                model.addAttribute("error", e);
	                return "/error";
	            }
	        }
	        return "/secure/aad";
	    }
	
	    private String getUsernamesFromGraph(String accessToken, String tenant) throws Exception {
	        URL url = new URL(String.format("https://graph.chinacloudapi.cn/%s/users?api-version=2013-04-05", tenant,
	                accessToken));
	
	        HttpURLConnection conn = (HttpURLConnection) url.openConnection();
	        // Set the appropriate header fields in the request header.
	        conn.setRequestProperty("api-version", "2013-04-05");
	        conn.setRequestProperty("Authorization", accessToken);
	        conn.setRequestProperty("Accept", "application/json;odata=minimalmetadata");
	        String goodRespStr = HttpClientHelper.getResponseStringFromConn(conn, true);
	        // logger.info("goodRespStr ->" + goodRespStr);
	        int responseCode = conn.getResponseCode();
	        JSONObject response = HttpClientHelper.processGoodRespStr(responseCode, goodRespStr);
	        JSONArray users = new JSONArray();
	
	        users = JSONHelper.fetchDirectoryObjectJSONArray(response);
	
	        StringBuilder builder = new StringBuilder();
	        User user = null;
	        for (int i = 0; i < users.length(); i++) {
	            JSONObject thisUserJSONObject = users.optJSONObject(i);
	            user = new User();
	            JSONHelper.convertJSONObjectToDirectoryObject(thisUserJSONObject, user);
	            builder.append(user.getUserPrincipalName() + "<br/>");
	        }
	        return builder.toString();
	    }
	
	}



## 8\.创建 BasicFilter 文件（适用于 BasicFilter MVC）

最后，我们可以创建 BasicFilter 文件，以便从视图（JSP 文件）处理请求。

创建名为 `BasicFilter.java` 的文件，其中包含以下内容：

Java
	
	package com.microsoft.aad.adal4jsample;
	
	import java.io.IOException;
	import java.io.UnsupportedEncodingException;
	import java.net.URI;
	import java.net.URLEncoder;
	import java.util.Date;
	import java.util.HashMap;
	import java.util.Map;
	import java.util.UUID;
	import java.util.concurrent.ExecutionException;
	import java.util.concurrent.ExecutorService;
	import java.util.concurrent.Executors;
	import java.util.concurrent.Future;
	
	import javax.naming.ServiceUnavailableException;
	import javax.servlet.Filter;
	import javax.servlet.FilterChain;
	import javax.servlet.FilterConfig;
	import javax.servlet.ServletException;
	import javax.servlet.ServletRequest;
	import javax.servlet.ServletResponse;
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	
	import com.microsoft.aad.adal4j.AuthenticationContext;
	import com.microsoft.aad.adal4j.AuthenticationResult;
	import com.microsoft.aad.adal4j.ClientCredential;
	import com.nimbusds.oauth2.sdk.AuthorizationCode;
	import com.nimbusds.openid.connect.sdk.AuthenticationErrorResponse;
	import com.nimbusds.openid.connect.sdk.AuthenticationResponse;
	import com.nimbusds.openid.connect.sdk.AuthenticationResponseParser;
	import com.nimbusds.openid.connect.sdk.AuthenticationSuccessResponse;
	
	public class BasicFilter implements Filter {
	
	    private String clientId = "";
	    private String clientSecret = "";
	    private String tenant = "";
	    private String authority;
	
	    public void destroy() {
	
	    }
	
	    public void doFilter(ServletRequest request, ServletResponse response,
	            FilterChain chain) throws IOException, ServletException {
	
	        if (request instanceof HttpServletRequest) {
	            HttpServletRequest httpRequest = (HttpServletRequest) request;
	            HttpServletResponse httpResponse = (HttpServletResponse) response;
	            try {
	
	                String currentUri = request.getScheme()
	                        + "://"
	                        + request.getServerName()
	                        + ("http".equals(request.getScheme())
	                                && request.getServerPort() == 80
	                                || "https".equals(request.getScheme())
	                                && request.getServerPort() == 443 ? "" : ":"
	                                + request.getServerPort())
	                        + httpRequest.getRequestURI();
	                String fullUrl = currentUri
	                        + (httpRequest.getQueryString() != null ? "?"
	                                + httpRequest.getQueryString() : "");
	                // check if user has a session
	                if (!AuthHelper.isAuthenticated(httpRequest)) {
	                    if (AuthHelper.containsAuthenticationData(httpRequest)) {
	                        Map<String, String> params = new HashMap<String, String>();
	                        for (String key : request.getParameterMap().keySet()) {
	                            params.put(key,
	                                    request.getParameterMap().get(key)[0]);
	                        }
	                        AuthenticationResponse authResponse = AuthenticationResponseParser
	                                .parse(new URI(fullUrl), params);
	                        if (AuthHelper.isAuthenticationSuccessful(authResponse)) {
	
	                            AuthenticationSuccessResponse oidcResponse = (AuthenticationSuccessResponse) authResponse;
	                            AuthenticationResult result = getAccessToken(
	                                    oidcResponse.getAuthorizationCode(),
	                                    currentUri);
	                            createSessionPrincipal(httpRequest, result);
	                        } else {
	                            AuthenticationErrorResponse oidcResponse = (AuthenticationErrorResponse) authResponse;
	                            throw new Exception(String.format(
	                                    "Request for auth code failed: %s - %s",
	                                    oidcResponse.getErrorObject().getCode(),
	                                    oidcResponse.getErrorObject()
	                                            .getDescription()));
	                        }
	                    } else {
	                            // not authenticated
	                            httpResponse.setStatus(302);
	                            httpResponse
	                                    .sendRedirect(getRedirectUrl(currentUri));
	                            return;
	                    }
	                } else {
	                    // if authenticated, how to check for valid session?
	                    AuthenticationResult result = AuthHelper
	                            .getAuthSessionObject(httpRequest);
	
	                    if (httpRequest.getParameter("refresh") != null) {
	                        result = getAccessTokenFromRefreshToken(
	                                result.getRefreshToken(), currentUri);
	                    } else {
	                        if (httpRequest.getParameter("cc") != null) {
	                            result = getAccessTokenFromClientCredentials();
	                        } else {
	                            if (result.getExpiresOnDate().before(new Date())) {
	                                result = getAccessTokenFromRefreshToken(
	                                        result.getRefreshToken(), currentUri);
	                            }
	                        }
	                    }
	                    createSessionPrincipal(httpRequest, result);
	                }
	            } catch (Throwable exc) {
	                httpResponse.setStatus(500);
	                request.setAttribute("error", exc.getMessage());
	                httpResponse.sendRedirect(((HttpServletRequest) request)
	                        .getContextPath() + "/error.jsp");
	            }
	        }
	        chain.doFilter(request, response);
	    }
	
	    private AuthenticationResult getAccessTokenFromClientCredentials()
	            throws Throwable {
	        AuthenticationContext context = null;
	        AuthenticationResult result = null;
	        ExecutorService service = null;
	        try {
	            service = Executors.newFixedThreadPool(1);
	            context = new AuthenticationContext(authority + tenant + "/", true,
	                    service);
	            Future<AuthenticationResult> future = context.acquireToken(
	                    "https://graph.chinacloudapi.cn", new ClientCredential(clientId,
	                            clientSecret), null);
	            result = future.get();
	        } catch (ExecutionException e) {
	            throw e.getCause();
	        } finally {
	            service.shutdown();
	        }
	
	        if (result == null) {
	            throw new ServiceUnavailableException(
	                    "authentication result was null");
	        }
	        return result;
	    }
	
	    private AuthenticationResult getAccessTokenFromRefreshToken(
	            String refreshToken, String currentUri) throws Throwable {
	        AuthenticationContext context = null;
	        AuthenticationResult result = null;
	        ExecutorService service = null;
	        try {
	            service = Executors.newFixedThreadPool(1);
	            context = new AuthenticationContext(authority + tenant + "/", true,
	                    service);
	            Future<AuthenticationResult> future = context
	                    .acquireTokenByRefreshToken(refreshToken,
	                            new ClientCredential(clientId, clientSecret), null,
	                            null);
	            result = future.get();
	        } catch (ExecutionException e) {
	            throw e.getCause();
	        } finally {
	            service.shutdown();
	        }
	
	        if (result == null) {
	            throw new ServiceUnavailableException(
	                    "authentication result was null");
	        }
	        return result;
	
	    }
	
	    private AuthenticationResult getAccessToken(
	            AuthorizationCode authorizationCode, String currentUri)
	            throws Throwable {
	        String authCode = authorizationCode.getValue();
	        ClientCredential credential = new ClientCredential(clientId,
	                clientSecret);
	        AuthenticationContext context = null;
	        AuthenticationResult result = null;
	        ExecutorService service = null;
	        try {
	            service = Executors.newFixedThreadPool(1);
	            context = new AuthenticationContext(authority + tenant + "/", true,
	                    service);
	            Future<AuthenticationResult> future = context
	                    .acquireTokenByAuthorizationCode(authCode, new URI(
	                            currentUri), credential, null);
	            result = future.get();
	        } catch (ExecutionException e) {
	            throw e.getCause();
	        } finally {
	            service.shutdown();
	        }
	
	        if (result == null) {
	            throw new ServiceUnavailableException(
	                    "authentication result was null");
	        }
	        return result;
	    }
	
	    private void createSessionPrincipal(HttpServletRequest httpRequest,
	            AuthenticationResult result) throws Exception {
	        httpRequest.getSession().setAttribute(
	                AuthHelper.PRINCIPAL_SESSION_NAME, result);
	    }
	
	    private String getRedirectUrl(String currentUri)
	            throws UnsupportedEncodingException {
	        String redirectUrl = authority
	                + this.tenant
	                + "/oauth2/authorize?response_type=code%20id_token&scope=openid&response_mode=form_post&redirect_uri="
	                + URLEncoder.encode(currentUri, "UTF-8") + "&client_id="
	                + clientId + "&resource=https%3a%2f%2fgraph.chinacloudapi.cn"
	                + "&nonce=" + UUID.randomUUID() + "&site_id=500879";
	        return redirectUrl;
	    }
	
	    public void init(FilterConfig config) throws ServletException {
	        clientId = config.getInitParameter("client_id");
	        authority = config.getServletContext().getInitParameter("authority");
	        tenant = config.getServletContext().getInitParameter("tenant");
	        clientSecret = config.getInitParameter("secret_key");
	    }
	
	}


此 Servlet 将公开 `ADAL4J` 应该从我们的应用程序运行的所有方法。这包括：

- `getAccessTokenFromClientCredentials()` - 从机密中获取访问令牌
- `getAccessTokenFromRefreshToken()` - 从刷新令牌中获取访问令牌
- `getAccessToken()` - 从我们使用的 OpenID Connect 流中获取访问令牌
- `createSessionPrincipal()` - 创建用于访问图形 API 的主体
- `getRedirectUrl()` - 获取重定向 URL，以将它与你在门户中输入的值进行比较。

##在 Tomcat 中编译并运行示例

切换回到根目录，并运行以下命令来生成你刚刚使用 `maven` 组成的示例。这会使用你针对依赖项编写的 `pom.xml` 文件。

`$ mvn package`

你的 `/targets` 目录中现在应包含 `adal4jsample.war` 文件。你可以在 Tomcat 容器中部署该文件并访问 URL

`http://localhost:8080/adal4jsample/`


> [AZURE.NOTE] 
使用最新的 Tomcat 服务器部署 WAR 非常容易。只要导航到 `http://localhost:8080/manager/` 并遵循有关上载 ``adal4jsample.war` 文件的说明即可。它会为你自动部署正确的终结点。

##后续步骤

祝贺你！ 现在，你已创建一个有效的 Java 应用程序，它可以对用户进行身份验证，使用 OAuth 2.0 安全调用 Web API，并获取有关用户的基本信息。如果你尚未这样做，可以在租户中填充一些用户。

[此处以 .zip 格式提供了](https://github.com/Azure-Samples/active-directory-java-webapp-openidconnect/archive/complete.zip)完整示例（不包括配置值），你也可以从 GitHub 克隆该示例：

	git clone --branch complete https://github.com/Azure-Samples/active-directory-java-webapp-openidconnect.git

<!---HONumber=Mooncake_1010_2016-->
