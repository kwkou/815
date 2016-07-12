<properties linkid="" urlDisplayName="" pageTitle="监视MySQL 数据库 on Azure数据库 - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,监视,性能指标,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="MySQL 数据库 on Azure 为用户提供核心性能指标的监控,您可以通过Azure管理门户的仪表盘进行查看。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="cn" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-operation-monitoring-metrics/)
- [English](/documentation/articles/mysql-database-enus-operation-monitoring-metrics/)

#监视MySQL 数据库 on Azure数据库

通过Azure管理门户-> MySQL 数据库 on Azure，在列表中选择所需的监控性能指标，点击“仪表盘”进行实时监控。

<table border="1" cellspacing="0" cellpadding="0">
  <tr>
    <td width="96" valign="top"><p align="center"><strong>性能指标 </strong></p></td>
    <td width="306" valign="top"><p align="center"><strong>描述 </strong></p></td>
  </tr>
    <tr>
    <td width="96" valign="top"><p align="center">CPU</p></td>
    <td width="306" valign="top"><p>CPU使用量的峰值 </p></td>
  </tr>
  <tr>
    <td width="96" valign="top"><p align="center">Query Per Second</p></td>
    <td width="306" valign="top"><p>每秒查询数量的平均值 </p></td>
  </tr>
    <tr>
    <td width="96" valign="top"><p align="center">Peak Query Per Second</p></td>
    <td width="306" valign="top"><p>每秒查询数量的峰值 </p></td>
  </tr>
  <tr>
    <td width="96" valign="top"><p align="center">Select Per Second</p></td>
    <td width="306" valign="top"><p>每秒Select查询数量的平均值 </p></td>
  </tr>
    <tr>
    <td width="96" valign="top"><p align="center">Peak Select Per Second</p></td>
    <td width="306" valign="top"><p>每秒Select查询数量的峰值 </p></td>
  </tr>
  <tr>
    <td width="96" valign="top"><p align="center">Update Per Second</p></td>
    <td width="306" valign="top"><p>每秒Update查询数量的平均值 </p></td>
  </tr>
    <tr>
    <td width="96" valign="top"><p align="center">Peak Update Per Second</p></td>
    <td width="306" valign="top"><p>每秒Update查询数量的峰值 </p></td>
  </tr>
  <tr>
    <td width="96" valign="top"><p align="center">Insert Per Seconds</p></td>
    <td width="306" valign="top"><p>每秒Insert 查询数量的平均值 </p></td>
  </tr>
   <tr>
    <td width="96" valign="top"><p align="center">Peak Insert Per Seconds</p></td>
    <td width="306" valign="top"><p>每秒Insert 查询数量的峰值 </p></td>
  </tr>
  <tr>
    <td width="96" valign="top"><p align="center">Delete Per Second</p></td>
    <td width="306" valign="top"><p>每秒Delete查询数量的平均值 </p></td>
  </tr>
    <tr>
    <td width="96" valign="top"><p align="center">Peak Delete Per Second</p></td>
    <td width="306" valign="top"><p>每秒Delete查询数量的峰值 </p></td>
  </tr>
  <tr>
    <td width="96" valign="top"><p align="center">Replace Per Second</p></td>
    <td width="306" valign="top"><p>每秒Replace查询数量的平均值 </p></td>
  </tr>
  <tr>
    <td width="96" valign="top"><p align="center">Peak Replace Per Second</p></td>
    <td width="306" valign="top"><p>每秒Replace查询数量的峰值 </p></td>
  </tr>
  <tr>
    <td width="96" valign="top"><p align="center">Slow Per Second</p></td>
    <td width="306" valign="top"><p>每秒慢查询的数量 </p></td>
  </tr>
  <tr>
    <td width="96" valign="top"><p align="center">Slow Queries</p></td>
    <td width="306" valign="top"><p>慢查询的累计数量 </p></td>
  </tr>
  <tr>
    <td width="96" valign="top"><p align="center">Successful Connections</p></td>
    <td width="306" valign="top"><p>成功链接的累计数量 </p></td>
  </tr>
  <tr>
    <td width="96" valign="top"><p align="center">Throttled Connections</p></td>
    <td width="306" valign="top"><p>被抑制链接的累计数量 </p></td>
  </tr>
  <tr>
    <td width="96" valign="top"><p align="center">Network in</p></td>
    <td width="306" valign="top"><p> 数据传入流量</p></td>
  </tr>
  <tr>
    <td width="96" valign="top"><p align="center">Network out</p></td>
    <td width="306" valign="top"><p> 数据传出流量</p></td>
  </tr>
  <tr>
    <td width="96" valign="top"><p align="center">Storage</p></td>
    <td width="306" valign="top"><p> 占用的存储空间</p></td>
  </tr>
</table>
