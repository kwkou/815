<properties linkid="" urlDisplayName="" pageTitle="MySQL服务问题 - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,常见问题,数据安全访问,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS,FAQ" description="针对用户在使用MySQL 数据库 on Azure中遇到的一些常见技术问题,提供快速解答。如果您仍存有疑问,欢迎联系技术支持。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="cn" />
#安全性咨询
> [AZURE.SELECTOR]
- [全部问题](/documentation/articles/mysql-database-tech-faq/)
- [服务咨询](/documentation/articles/mysql-database-serviceinquiry/)
- [连接问题](/documentation/articles/mysql-database-connectioninquiry/)
- [安全性咨询](/documentation/articles/mysql-database-securityinquiry/)
- [兼容性问题](/documentation/articles/mysql-database-compatibilityinquiry/)

### **我看到数据库服务器的地址是一个公共的endpoint?我的应用在同一个数据中心访问数据库时?访问请求是否会先经过Internet再到服务器地址?**

不会，Azure数据中心的网络路由会解析到这是一个自己的地址，会直接通过数据中心的内部网络路由到那个IP地址。而且这样的路由是安全的，不用担心查询请求和结果会被第三方监听到。但是如果您的应用和数据库不在同一个数据中心里，数据库查询请求和结果会经过Internet，在这种情况下建议用户用SSL来保证数据传递的私密性。关于SSL链接的详细信息参考[SSL安全访问MySQL Database on Azure](/documentation/articles/mysql-database-ssl-connection/)。