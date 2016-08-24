<properties
	pageTitle="Azure AD Java 入门 | Azure"
	description="如何生成使用户登录以访问 API 的 Java 命令行应用。"
	services="active-directory"
	documentationCenter="java"
	authors="brandwe"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="05/31/2016"
	wacn.date="07/26/2016"/>


# 通过 Azure AD 使用 Java 命令行应用访问 API

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

若要开始，请[下载应用程序框架](https://github.com/Azure-Samples/active-directory-java-webapp-openidconnect/archive/skeleton.zip)或[下载已完成的示例](https://github.com/Azure-Samples/active-directory-java-webapp-openidconnect\/archive/complete.zip)。你还需要一个用于注册应用程序的 Azure AD 租户。如果你没有此租户，请[了解如何获取租户](/documentation/articles/active-directory-howto-tenant/)。

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
		<artifactId>public-client-adal4j-sample</artifactId>
		<packaging>jar</packaging>
		<version>0.0.1-SNAPSHOT</version>
		<name>public-client-adal4j-sample</name>
		<url>http://maven.apache.org</url>
		<properties>
			<spring.version>3.0.5.RELEASE</spring.version>
		</properties>
		
		<dependencies>
			<dependency>
				<groupId>com.microsoft.azure</groupId>
				<artifactId>adal4j</artifactId>
				<version>1.1.2</version>
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
		</dependencies>
		<build>
			<finalName>public-client-adal4j-sample</finalName>
			<plugins>
			<plugin>
		         <groupId>org.codehaus.mojo</groupId>
		          <artifactId>exec-maven-plugin</artifactId>
		           <version>1.2.1</version>
		           <configuration>
		            <mainClass>PublicClient</mainClass>
		            </configuration>
		     </plugin>
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
					<plugin>
		        <artifactId>maven-assembly-plugin</artifactId>
		        <executions>
		          <execution>
		            <phase>package</phase>
		            <goals>
		              <goal>single</goal>
		            </goals>
		          </execution>
		        </executions>
		        <configuration>
		          <descriptorRefs>
		            <descriptorRef>jar-with-dependencies</descriptorRef>
		          </descriptorRefs>
		        </configuration>
		      </plugin>
		      <plugin>
		  <groupId>org.apache.maven.plugins</groupId>
		  <artifactId>maven-assembly-plugin</artifactId>
		  <configuration>
		    <archive>
		      <manifest>
		        <mainClass>PublicClient</mainClass>
		      </manifest>
		    </archive>
		  </configuration>
		</plugin>
		</plugins>
		</build>
		
	</project>






## 3\.创建 java PublicClient 文件

如上所述，我们将使用图形 API 来获取有关已登录的用户的数据。为了顺利进行，我们应该创建一个表示“目录对象”的文件以及一个表示“用户”的单独文件，如此便可以使用 Java 的 OO 模式。

1. 创建一个名为 `DirectoryObject.java` 的文件，我们将用它来存储有关任何 DirectoryObject 的基本数据（你稍后可以随意使用它来执行任何其他图形查询）。可以从以下内容中剪切/粘贴此信息：

Java

	import java.io.BufferedReader;
	import java.io.InputStreamReader;
	import java.util.concurrent.ExecutorService;
	import java.util.concurrent.Executors;
	import java.util.concurrent.Future;
	
	import javax.naming.ServiceUnavailableException;
	
	import com.microsoft.aad.adal4j.AuthenticationContext;
	import com.microsoft.aad.adal4j.AuthenticationResult;
	
	public class PublicClient {
	
	    private final static String AUTHORITY = "https://login.microsoftonline.com/common/";
	    private final static String CLIENT_ID = "2a4da06c-ff07-410d-af8a-542a512f5092";
	
	    public static void main(String args[]) throws Exception {
	
	        try (BufferedReader br = new BufferedReader(new InputStreamReader(
	                System.in))) {
	            System.out.print("Enter username: ");
	            String username = br.readLine();
	            System.out.print("Enter password: ");
	            String password = br.readLine();
	
	            AuthenticationResult result = getAccessTokenFromUserCredentials(
	                    username, password);
	            System.out.println("Access Token - " + result.getAccessToken());
	            System.out.println("Refresh Token - " + result.getRefreshToken());
	            System.out.println("ID Token - " + result.getIdToken());
	        }
	    }
	
	    private static AuthenticationResult getAccessTokenFromUserCredentials(
	            String username, String password) throws Exception {
	        AuthenticationContext context = null;
	        AuthenticationResult result = null;
	        ExecutorService service = null;
	        try {
	            service = Executors.newFixedThreadPool(1);
	            context = new AuthenticationContext(AUTHORITY, false, service);
	            Future<AuthenticationResult> future = context.acquireToken(
	                    "https://graph.chinacloudapi.cn", CLIENT_ID, username, password,
	                    null);
	            result = future.get();
	        } finally {
	            service.shutdown();
	        }
	
	        if (result == null) {
	            throw new ServiceUnavailableException(
	                    "authentication result was null");
	        }
	        return result;
	    }
	}


##编译并运行示例

更改回根目录，并运行下列命令来生成你刚刚使用 `maven` 组成的示例。这会使用你针对依赖项编写的 `pom.xml` 文件。

`$ mvn package`

你的 `/targets` 目录中现在应包含 `adal4jsample.war` 文件。你可以在 Tomcat 容器中部署该文件并访问 URL

`http://localhost:8080/adal4jsample/`


> [AZURE.NOTE] 
使用最新的 Tomcat 服务器部署 WAR 非常容易。只要导航到 `http://localhost:8080/manager/` 并遵循有关上载 ``adal4jsample.war` 文件的说明即可。它会为你自动部署正确的终结点。

##后续步骤

祝贺你！ 现在，你已创建一个有效的 Java 应用程序，它可以对用户进行身份验证，使用 OAuth 2.0 安全调用 Web API，并获取有关用户的基本信息。如果你尚未这样做，可以在租户中填充一些用户。

[此处以 .zip 格式提供了](https://github.com/Azure-Samples/active-directory-java-webapp-openidconnect/archive/complete.zip)完整示例（不包括配置值），你也可以从 GitHub 克隆该示例：

	git clone --branch complete https://github.com/Azure-Samples/active-directory-java-webapp-openidconnect.git

<!---HONumber=Mooncake_0808_2016-->