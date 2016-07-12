<properties
   pageTitle="Linux 虚拟机远程连接安全贴士 | Azure"
   description="本文介绍如何安全的远程连接到 Linux 虚拟机"
   services="open-source"
   documentationCenter=""
   authors=""
   manager=""
   editor=""/>

<tags
   ms.service="open-source-website"  
   ms.date=""
   wacn.date="06/14/2016"/>


#Azure Linux 虚拟机远程连接安全贴士

在 Azure 上创建 Linux 虚拟机很容易。虚拟机创建完后 SSH 端口 22 是默认打开的，并不设任何限制，这样我们可以用 SSH 通过公共网络连到虚拟机。SSH 通常来讲还是安全的。我们连的时候也需要提供密码或密钥。但开着这样通用的端口总是安全隐患。而且，通过 “快速创建” 方式建出来的虚拟机默认 azureuser 为虚拟机用户名。如果密码设的又弱的话，虚拟机很容易会被侵入，造成更大的损失。要完全免去安全隐患，最好是不暴露任何 SSH 端口，但这有可能不太现实。 如果一定要暴露，建议采取一些保护措施。
 
创建虚拟机时如果选择“从库中”，创建的流程将稍为复杂，但选择更多。 
 
![从库中][1]

首先可以改掉默认的 azureuser 用户名。  

![azureuser 用户名][2]
 
做身份验证最好使用密钥。如何产生密钥可以参考[使用密钥](/manage/linux/how-to-guides/ssh-into-linux)。如果使用密码，请务必选择复杂的密码，使得蛮力攻击不会很容易猜出你的密码，这样可以大大的增强安全性。如需修改密码，可以使用 VMAccess 扩展。具体可以参考 [VMAcess](/documentation/articles/virtual-machines-linux-classic-reset-access/)。

其次是关闭标准端口，改用其他端口。这样黑客们起码需要扫遍很多端口才能开始尝试连接，增加了攻击的难度和时间。如果不设公用端口，系统默认为自动，会随机选一个端口。  

![端口][3]
 
最后是为 SSH 端口配置 ACL。 ACL 可以控制允许或不允许连接 SSH 端口的 IP 地址或 CIDR 子网地址。例如你可以只允许属于你们团队的服务器的 IP 地址去 SSH 到你的 Azure 虚拟机。这样 Azure 平台就会帮你阻止很多非法的连接尝试。 
 
![为 SSH 端口配置 ACL][4]
 
以上简单几步，就可以让你更安全的远程连接到 Azure Linux 虚拟机。如果你希望进一步加强安全性，可以参考 [Network tutorial](/documentation/services/networking) 来利用网络功能对虚拟机做更好的隔离，比如部署点到站点 VPN 或网络安全组。  

<!--image references-->
[1]: ./media/open-source-azure-virtual-machines-linux-rdp-security-guide/open-source-azure-virtual-machines-linux-rdp-security-guide-1.png  
[2]: ./media/open-source-azure-virtual-machines-linux-rdp-security-guide/open-source-azure-virtual-machines-linux-rdp-security-guide-2.png  
[3]: ./media/open-source-azure-virtual-machines-linux-rdp-security-guide/open-source-azure-virtual-machines-linux-rdp-security-guide-3.png  
[4]: ./media/open-source-azure-virtual-machines-linux-rdp-security-guide/open-source-azure-virtual-machines-linux-rdp-security-guide-4.png  