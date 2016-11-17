<properties
	pageTitle="在 Azure 应用服务中生成和部署 Java API 应用"
	description="了解如何创建 Java API 应用包并将其部署到 Azure 应用服务。"
	services="app-service\api"
	documentationCenter="java"
	authors="bradygaster"
	manager="mohisri"
	editor="tdykstra"/>

<tags
	ms.service="app-service-api"
	ms.workload="web"
	ms.tgt_pltfrm="na"
	ms.devlang="java"
	ms.topic="get-started-article"
	ms.date="08/31/2016"
	wacn.date="09/26/2016"
	ms.author="rachelap"/>

# 在 Azure 应用服务中生成和部署 Java API 应用

[AZURE.INCLUDE [app-service-api-get-started-selector](../../includes/app-service-api-get-started-selector.md)]

本教程介绍如何创建 Java 应用程序，并使用 [Git] 将其部署到 Azure 应用服务 API 应用。本教程中的说明适用于任何能够运行 Java 的操作系统。本教程中的代码是使用 [Maven] 生成的。将使用 [Jax-RS] 创建 RESTful 服务，这些 Jax-RS 是根据 [Swagger] 元数据规范使用[Swagger 编辑器]生成的。

## 先决条件

1. [Java 开发人员工具包 8]（或更高版本）
1. 已在开发计算机上安装 [Maven]
1. 已在开发计算机上安装 [Git]
1. [Azure] 订阅付费版或[试用]版
1. HTTP 测试应用程序，如 [Postman]

## 使用 Swagger.IO 创建 API 基架

使用 swagger.io 在线编辑器可以输入表示 API 结构的 Swagger JSON 或 YAML 代码。设计 API 外围应用后，可以针对各种不同的平台和框架导出代码。在下一部分，我们将修改基架代码，包含模拟功能。

本演示从粘贴到 swagger.io 编辑器中的 Swagger JSON 正文开始，接着使用该正文来生成利用 JAX-RS 访问 REST API 终结点的代码。然后，将编辑基架代码来返回模拟数据，以便模拟一个构建在数据持久性机制基础上的 REST API。

1. 将以下 Swagger JSON 代码复制到剪贴板：

        {
            "swagger": "2.0",
            "info": {
                "version": "v1",
                "title": "Contact List",
                "description": "A Contact list API based on Swagger and built using Java"
            },
            "host": "localhost",
            "schemes": [
                "http",
                "https"
            ],
            "basePath": "/api",
            "paths": {
                "/contacts": {
                    "get": {
                        "tags": [
                            "Contact"
                        ],
                        "operationId": "contacts_get",
                        "consumes": [],
                        "produces": [
                            "application/json",
                            "text/json"
                        ],
                        "responses": {
                            "200": {
                                "description": "OK",
                                "schema": {
                                    "type": "array",
                                    "items": {
                                        "$ref": "#/definitions/Contact"
                                    }
                                }
                            }
                        },
                        "deprecated": false
                    }
                },
                "/contacts/{id}": {
                    "get": {
                        "tags": [
                            "Contact"
                        ],
                        "operationId": "contacts_getById",
                        "consumes": [],
                        "produces": [
                            "application/json",
                            "text/json"
                        ],
                        "parameters": [
                            {
                                "name": "id",
                                "in": "path",
                                "required": true,
                                "type": "integer",
                                "format": "int32"
                            }
                        ],
                        "responses": {
                            "200": {
                                "description": "OK",
                                "schema": {
                                    "type": "array",
                                    "items": {
                                        "$ref": "#/definitions/Contact"
                                    }
                                }
                            }
                        },
                        "deprecated": false
                    }
                }
            },
            "definitions": {
                "Contact": {
                    "type": "object",
                    "properties": {
                        "Id": {
                            "format": "int32",
                            "type": "integer"
                        },
                        "Name": {
                            "type": "string"
                        },
                        "EmailAddress": {
                            "type": "string"
                        }
                    }
                }
            }
        }

1. 导航到[在线 Swagger 编辑器]。在该位置，单击“文件”->“粘贴 JSON”菜单项。

    ![“粘贴 JSON”菜单项][paste-json]

1. 粘贴前面复制的联系人列表 API Swagger JSON。

    ![将 JSON 代码粘贴到 Swagger][pasted-swagger]

1. 查看编辑器中显示的文档页和 API 摘要。

    ![查看 Swagger 生成的文档][view-swagger-generated-docs]

1. 选择“生成服务器”->“JAX RS”菜单选项，创建服务器端代码的基架，稍后要编辑该代码来添加模拟实现。

    ![“生成代码”菜单项][generate-code-menu-item]

    生成代码后，系统会提供要下载的 ZIP 文件。此文件包含 Swagger 代码生成器创建了基架的代码，以及所有关联的生成脚本。将整个库解压缩到开发工作站上的某个目录。

## 编辑代码以添加 API 实现

在本部分，将 Swagger 所生成代码的服务器端实现替换为自定义代码。新代码将 Contact 实体的 ArrayList 返回给调用方客户端。

1. 使用 [Visual Studio Code] 或偏好的文本编辑器，打开位于 *src/gen/java/io/swagger/model* 文件夹中的 *Contact.java* 模型文件。

    ![打开联系人模型文件][open-contact-model-file]

1. 将以下构造函数添加到 **Contact** 类。

        public Contact(Integer id, String name, String email) 
        {
            this.id = id;
            this.name = name;
            this.emailAddress = email;
        }

1. 使用 [Visual Studio Code] 或偏好的文本编辑器，打开位于 *src/main/java/io/swagger/api/impl* 文件夹中的 *ContactsApiServiceImpl.java* 服务实现文件。

    ![打开联系人服务代码文件][open-contact-service-code-file]

1. 使用新代码覆盖文件中的代码，将模拟实现添加到服务代码。

        package io.swagger.api.impl;

        import io.swagger.api.*;
        import io.swagger.model.*;
        import com.sun.jersey.multipart.FormDataParam;
        import io.swagger.model.Contact;
        import java.util.*;
        import io.swagger.api.NotFoundException;
        import java.io.InputStream;
        import com.sun.jersey.core.header.FormDataContentDisposition;
        import com.sun.jersey.multipart.FormDataParam;
        import javax.ws.rs.core.Response;
        import javax.ws.rs.core.SecurityContext;

        @javax.annotation.Generated(value = "class io.swagger.codegen.languages.JaxRSServerCodegen", date = "2015-11-24T21:54:11.648Z")
        public class ContactsApiServiceImpl extends ContactsApiService {
  
            private ArrayList<Contact> loadContacts()
            {
                ArrayList<Contact> list = new ArrayList<Contact>();
                list.add(new Contact(1, "Barney Poland", "barney@contoso.com"));
                list.add(new Contact(2, "Lacy Barrera", "lacy@contoso.com"));
                list.add(new Contact(3, "Lora Riggs", "lora@contoso.com"));
                return list;
            }
  
            @Override
            public Response contactsGet(SecurityContext securityContext)
            throws NotFoundException {
                ArrayList<Contact> list = loadContacts();
                return Response.ok().entity(list).build();
                }
  
            @Override
            public Response contactsGetById(Integer id, SecurityContext securityContext)
            throws NotFoundException {
                ArrayList<Contact> list = loadContacts();
                Contact ret = null;
            
                for(int i=0; i<list.size(); i++)
                {
                    if(list.get(i).getId() == id)
                        {
                            ret = list.get(i);
                        }
                }
                return Response.ok().entity(ret).build();
            }
        }

1. 打开命令提示符，将目录切换到应用程序的根文件夹。

1. 执行以下 Maven 命令生成代码，然后在本地使用 Jetty 应用服务器运行该代码。

        mvn package jetty:run

1. 应会在命令窗口中看到 Jetty 已经在端口 8080 上启动代码。

    ![打开联系人服务代码文件][run-jetty-war]

1. 使用 [Postman] 对 http://localhost:8080/api/contacts 中的“get all contacts”API 方法发出请求。

    ![调用联系人 API][calling-contacts-api]

1. 使用 [Postman] 对 http://localhost:8080/api/contacts/2 中的“get specific contact”API 方法发出请求。

    ![调用联系人 API][calling-specific-contact-api]

1. 最后，在控制台中执行以下 Maven 命令来生成 Java WAR（Web 存档）文件。

        mvn package war:war

1. 生成的 WAR 文件将放入 **target** 文件夹。导航到 **target** 文件夹，然后将 WAR 文件重命名为 **ROOT.war**。（请确保大小写符合此格式）。

         rename swagger-jaxrs-server-1.0.0.war ROOT.war

1. 最后，从应用程序的根文件夹执行以下命令创建 **deploy** 文件夹，用于将 WAR 文件部署到 Azure。

         mkdir deploy
         mkdir deploy\webapps
         copy target\ROOT.war deploy\webapps
         cd deploy

## 将输出发布到 Azure 应用服务

本部分介绍如何使用 Azure 门户预览创建新的 API 应用、准备该 API 应用以托管 Java 应用程序，然后将新建的 WAR 文件部署到 Azure 应用服务以运行新 API 应用。

1. 在 [Azure 门户预览]中创建新的 API 应用：单击“新建”->“Web + 移动”->“API 应用”菜单项，输入应用详细信息，然后单击“创建”。

    ![创建新的 API 应用][create-api-app]

1. 创建 API 应用后，打开应用的“设置”边栏选项卡，然后单击“应用程序设置”菜单项。在可用选项中选择最新的 Java 版本，在“Web 容器”菜单中选择最新的 Tomcat，然后单击“保存”。

    ![在 API 应用边栏选项卡中设置 Java][set-up-java]

1. 登录到 [Azure 经典管理门户](https://manage.windowsazure.cn)。在“Web 应用”页上，选择要为其安装连续部署的 Web 应用，然后选择“仪表板”选项卡。

1. 在“速览”部分中，单击“重置部署凭据”设置菜单项，提供用于将文件发布到 API 应用的用户名和密码。

3. 选择“从源控件设置部署”。在“设置部署”对话框中，选择“本地 Git 存储库”选项，然后单击“确定”。随后将创建在 Azure 中运行的、与 API 应用关联的 Git 存储库。每次将代码提交到 Git 存储库的 *master* 分支时，代码就会发布到实时运行的 API 应用实例。

1. 回到 [Azure 门户预览]。将新的 Git 存储库 URL 复制到剪贴板。请保存此 URL，因为稍后它很重要。

    ![为应用设置新的 Git 存储库][copy-git-repo-url]

1. Git 会将 WAR 文件推送到联机存储库。为此，请导航到前面创建的 **deploy** 文件夹，以便轻松地将代码提交到应用服务中运行的存储库。进入控制台窗口并导航到 webapps 文件夹所在的文件夹后，请发出以下 Git 命令启动过程并开始部署。

         git init
         git add .
         git commit -m "initial commit"
         git remote add azure [YOUR GIT URL]
         git push azure master

    发出**推送**请求后，系统会要求提供前面为部署凭据创建的密码。输入凭据后，门户会显示已部署更新。

1. 如果再次使用 Postman 来调用刚刚部署的、在 Azure 应用服务中运行的新 API 应用，将看到行为是一致的，现在它会按预期返回联系人数据，并且会使用对 Swagger.io 创建了基架的 Java 代码所做的更改。

    ![在 Azure 中实时使用 Java 联系人 REST API][postman-calling-azure-contacts]

## 后续步骤

本文介绍了通过 Swagger.io 编辑器开始处理 Swagger JSON 文件，以及获得的一些 Java 基架代码。在该编辑器中，可以进行简单的更改，并使用 Git 部署过程以 Java 编写正常运行的 API 应用。下一篇教程说明如何[借助 CORS 从 JavaScript 客户端使用 API 应用][App Service API CORS]。本系列教程的后续文章说明如何实现身份验证和授权。

若要基于本示例生成应用，可以详细了解如何使用[用于 Java 的存储 SDK] 来持久保存 JSON Blob。或者，可以使用 [Document DB Java SDK] 将联系人数据保存到 Azure Document DB。

有关在 Azure 中使用 Java 的详细信息，请参阅 [Java Developer Center]（Java 开发人员中心）。

<!-- URL List -->

[App Service API CORS]: /documentation/articles/app-service-api-cors-consume-javascript/
[Azure 门户预览]: https://portal.azure.cn/
[Document DB Java SDK]: /documentation/articles/documentdb-java-application/
[试用]: /pricing/1rmb-trial/
[Git]: http://www.git-scm.com/
[Java Developer Center]: /develop/java/
[Java 开发人员工具包 8]: http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
[Jax-RS]: https://jax-rs-spec.java.net/
[Maven]: https://maven.apache.org/
[Azure]: https://www.azure.cn/
[在线 Swagger 编辑器]: http://editor.swagger.io/
[Postman]: https://www.getpostman.com/
[用于 Java 的存储 SDK]: /documentation/articles/storage-java-how-to-use-blob-storage/
[Swagger]: http://swagger.io/
[Swagger 编辑器]: http://editor.swagger.io/
[Visual Studio Code]: https://code.visualstudio.com

<!-- IMG List -->

[paste-json]: ./media/app-service-api-java-api-app/paste-json.png
[pasted-swagger]: ./media/app-service-api-java-api-app/pasted-swagger.png
[view-swagger-generated-docs]: ./media/app-service-api-java-api-app/view-swagger-generated-docs.png
[generate-code-menu-item]: ./media/app-service-api-java-api-app/generate-code-menu-item.png
[open-contact-model-file]: ./media/app-service-api-java-api-app/open-contact-model-file.png
[open-contact-service-code-file]: ./media/app-service-api-java-api-app/open-contact-service-code-file.png
[run-jetty-war]: ./media/app-service-api-java-api-app/run-jetty-war.png
[calling-contacts-api]: ./media/app-service-api-java-api-app/calling-contacts-api.png
[calling-specific-contact-api]: ./media/app-service-api-java-api-app/calling-specific-contact-api.png
[create-api-app]: ./media/app-service-api-java-api-app/create-api-app.png
[set-up-java]: ./media/app-service-api-java-api-app/set-up-java.png
[deployment-credentials]: ./media/app-service-api-java-api-app/deployment-credentials.png
[select-git-repo]: ./media/app-service-api-java-api-app/select-git-repo.png
[copy-git-repo-url]: ./media/app-service-api-java-api-app/copy-git-repo-url.png
[postman-calling-azure-contacts]: ./media/app-service-api-java-api-app/postman-calling-azure-contacts.png

<!---HONumber=Mooncake_0919_2016-->