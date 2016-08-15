<properties 
	pageTitle="在 Azure 上选择 Linux 的用户名" 
	description="了解如何在 Azure 中选择 Linux 虚拟机的用户名。" 
	services="virtual-machines" 
	documentationCenter="" 
	authors="szarkos" 
	manager="timlt" 
	editor=""/>

<tags 
	ms.service="virtual-machines-linux" 
	ms.date="03/25/2016" 
	wacn.date="06/13/2016"/>



#在 Azure 上选择 Linux 的用户名#

当你在 Azure 上预配 Linux 虚拟机时，必须指定非根用户的名称，以便以后使用该用户登录到 VM。你可以选择新用户的名称，或者，如果是通过经典管理门户进行预配，则可接受默认名称“azureuser”。

大多数情况下，此用户不存在于基映像上，而是在预配过程中创建的。如果该用户存在于基 VM 映像上，则 Azure Linux 代理会直接根据你在创建 VM 时指定的信息为该用户配置密码（和/或 SSH 密钥）。

**不过**，Linux 定义了一组不应使用的用户名。如果你尝试使用现有的系统用户（按照定义，即 UID 为 0-99 的用户）来预配 Linux VM，预配过程将**失败**。一个典型的示例是 `root` 用户，其 UID 为 0。

 - 另请参见：[Linux 标准库 - 用户 ID 范围](http://refspecs.linuxfoundation.org/LSB_4.1.0/LSB-Core-generic/LSB-Core-generic/uidrange.html)

以下是在预配 Linux 虚拟机时应该避免使用的用户名。我们建议你**不要使用这些用户名**，否则 VM 预配过程可能会失败。


## openSUSE
- abrt
- adm
- audio
- bin
- bin
- cdrom
- cgred
- daemon
- dbus
- dialout
- dip
- disk
- floppy
- ftp
- games
- gopher
- haldaemon
- halt
- kmem
- lock
- lp
- mail
- man
- mem
- nfsnobody
- nobody
- ntp
- operator
- oprofile
- postdrop
- postfix
- qpidd
- root
- rpc
- rpcuser
- saslauth
- shutdown
- slocate
- sshd
- stapdev
- stapusr
- sync
- sys
- tape
- test
- tcpdump
- tty
- users
- utempter
- utmp
- uucp
- vcsa
- video
- wheel


## SLES
- audio
- bin
- cdrom
- console
- daemon
- dialout
- disk
- floppy
- ftp
- ftp
- games
- haldaemon
- kmem
- lp
- lp
- mail
- maildrop
- man
- messagebus
- modem
- news
- news
- nobody
- nogroup
- polkituser
- postfix
- public
- root
- shadow
- sshd
- sys
- test
- trusted
- tty
- users
- utmp
- uucp
- uuidd
- video
- wheel
- www
- wwwrun
- xok

 
## CentOS
- abrt
- adm
- audio
- bin
- cdrom
- cgred
- daemon
- dbus
- dialout
- dip
- disk
- floppy
- ftp
- ftp
- games
- gopher
- haldaemon
- halt
- kmem
- lock
- lp
- mail
- man
- mem
- nfsnobody
- nobody
- ntp
- operator
- oprofile
- postdrop
- postfix
- qpidd
- root
- rpc
- rpcuser
- saslauth
- shutdown
- slocate
- sshd
- stapdev
- stapusr
- sync
- sys
- tape
- test
- tcpdump
- tty
- users
- utempter
- utmp
- uucp
- vcsa
- video
- wheel


## Ubuntu
- adm
- admin
- audio
- backup
- bin
- cdrom
- crontab
- daemon
- dialout
- dip
- disk
- fax
- floppy
- fuse
- games
- gnats
- irc
- kmem
- landscape
- libuuid
- list
- lp
- mail
- man
- messagebus
- mlocate
- netdev
- news
- nobody
- nogroup
- operator
- plugdev
- proxy
- root
- sasl
- shadow
- src
- ssh
- sshd
- staff
- sudo
- sync
- sys
- syslog
- tape
- tty
- users
- utmp
- uucp
- video
- voice
- whoopsie
- www-data

 

<!---HONumber=70-->