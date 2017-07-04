<properties
    pageTitle="有关 Power BI Embedded 会话计费的常见问题及回答"
    description="有关 Power BI Embedded 会话计费的常见问题及回答"
    service=""
    resource="powerbiembedded"
    authors="Allan Li"
    displayOrder=""
    selfHelpType=""
    supportTopicIds=""
    productPesIds=""
    resourceTags="PowerBI Embedded, Token, PowerBI-Cli"
    cloudEnvironments="MoonCake" />
<tags
    ms.service="power-bi-embedded-aog"
    ms.date=""
    wacn.date="05/31/2017" />

# 有关 Power BI Embedded 会话计费的常见问题及回答

根据官网中的 [Power BI Embedded](/pricing/details/power-bi-embedded/) 相关文档介绍，Power BI Embedded 的计费依据是每个月产生的会话数目。每个月前 100 个会话免费，而超出的部分则按照每 100 个会话 32.86 元/月的标准进行计费。而对会话的定义，官网给出的解释如下所示：

> “会话是指最终用户和 Power BI Embedded 报表之间的一组交互活动。每次向用户显示 Power BI Embedded 报表时，就会启动一个会话，并向订阅持有者收取使用会话的费用。会话按统一收费率计费，独立于报表中的视觉对象元素数量或报表内容的刷新频率。会话结束可以按用户关闭报表算，也可以按启动会话后一个小时算，以先发生者为准。”

许多客户对这段内容不是很理解，故通过工单来咨询详情，以下针对常见的一些问题进行回答：

问题 1 ：怎样理解会话的定义？

答：当用户通过 PowerBI-Cli 工具或是其他工具生成 Power BI Embedded token，并将其用于内嵌报表的展示时，Power BI Embedded 后台将生成一个会话，它的最长寿命固定为一个小时，即如果不生成新的会话，旧会话会在生成一个小时后过期。

问题 2 ：会话结束可以按用户关闭报表算，其中，用户关闭报表具体指的是什么？

答：相比 [demo](https://microsoft.github.io/PowerBI-JavaScript/demo/code-demo/index.html#) 页面而言，用户访问题的 Power BI Embedded 报表展示页面往往类似于 [展示页面](https://pbi.chinacloudsites.cn/) 中的内容。当每次用户访问 URL 时，网页的后台将创建一个新的 token 用于报表展示，因此当用户在展示报表一个小时内关闭页面时，该 Power BI Embedded 的会话结束，当客户重新登陆时将开启一个新的会话；而当用户访问该报表超过一个小时后，也将开启一个新的会话，这也就是[官网文档](/pricing/details/power-bi-embedded/) 中所说 “ 会话结束可以按用户关闭报表算，也可以按启动会话后一个小时算，以先发生者为准。”


问题 3 ：创建一个 token 之后，是否多个用户同时用这一个 token，在一小时内算是一个会话？

答：多个用户同时用一个 token 访问内嵌报表，与一个用户用 token 访问内嵌报表所产生的会话数相同，一个小时内都只用一个会话。


问题 4 ：根据文档[如何将 Power BI Embedded 与 REST 配合使用](/documentation/articles/power-bi-embedded-iframe/)用客户端程序生成了 token 时，有以下两个参数可以随意设置 token 有效时长：

    "\"nbf\":" . date("U") . "," .
    "\"exp\":" . date("U" , strtotime("+1 hour"))

问题 5 ：以上修改 token 有效时长和用 PowerBI-Cli 创建 token 时有效时间固定为 1 小时的说法是否有冲突？

答：会话寿命与 token 寿命并不相关。当某个会话产生的一个小时内不生成新的会话，那么旧会话将在生成的一个小时后过期；但 token 寿命可根据需要自己来确定。换句话说，如果一个 token 的寿命超过一个小时，那么若在首次使用该 token 后超过一个小时候后再次被使用时，因为旧的会话已经过期，它将启动一个新的会话。
