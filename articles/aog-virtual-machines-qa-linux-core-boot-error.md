<properties 
	pageTitle="Linux 内核超时导致虚拟机无法正常启动" 
	description="Linux 内核超时导致虚拟机无法正常启动。" 
	services="virtual machine" 
	documentationCenter="" 
	authors=""
	manager="" 
	editor=""/>
<tags ms.service="virtual-machines-aog" ms.date="" wacn.date="12/05/2016"/>
# Linux 内核超时导致虚拟机无法正常启动 #

### 问题描述 ###

当 Linux 虚拟机启动时,通过串口输出或者启动日志, 观察到超时的报错.导致虚拟机无法正常启动和连接.

### 问题分析 ###

常见的超时报错范例如下:

    INFO: task swapper:1 blocked for more than 120 seconds.
      Not tainted 2.6.32-504.8.1.el6.x86_64 #1
    "echo 0 > /proc/sys/kernel/hung_task_timeout_secs" disables this message.
    swapper   D 0000000000000000 0 1  0 0x00000000
     ffff88010f64fde0 0000000000000046 ffff88010f64fd50 ffffffff81074f95
     0000000000005c2f ffffffff8100bb8e ffff88010f64fe50 0000000000100000
     0000000000000002 00000000fffb73e0 ffff88010f64dab8 ffff88010f64ffd8
    1.	Call Trace:
     [<ffffffff81074f95>] ? __call_console_drivers+0x75/0x90
     [<ffffffff8100bb8e>] ? apic_timer_interrupt+0xe/0x20
     [<ffffffff81075d51>] ? vprintk+0x251/0x560
     [<ffffffff8152a862>] schedule_timeout+0x192/0x2e0
     [<ffffffff810874f0>] ? process_timeout+0x0/0x10
     [<ffffffff8152a9ce>] schedule_timeout_uninterruptible+0x1e/0x20
     [<ffffffff81089650>] msleep+0x20/0x30
     [<ffffffff81c2a571>] prepare_namespace+0x30/0x1a9
     [<ffffffff81c2992a>] kernel_init+0x2e1/0x2f7ls
     [<ffffffff8100c20a>] child_rip+0xa/0x20
     [<ffffffff81c29649>] ? kernel_init+0x0/0x2f7
     [<ffffffff8100c200>] ? child_rip+0x0/0x20

上述报错描述了系统任务在等待 IO 超过 120 秒以后, 依旧没有得到响应,导致该任务被阻止. IO 超时未响应的原因, 有多种: 磁盘下线、 存储有严重的延迟、 磁盘阵列(RAID)工作异常, 或者 Linux 虚拟机本身的 CPU 和内存资源不足都会导致 IO 超时.

### 解决方案 ###

- 方案一:
 
	最快速的尝试是通过 Azure 门户重启该虚拟机,大部分问题可以得到解决.

- 方案二:
 
	如果方案一依然无法解决连接的问题, 建议您选择删除虚拟机保留磁盘, 然后基于该磁盘新建虚拟机. **修改内核参数:**

	>注: 以下方案仅适用于 CentOS 和 RHEL,其他版本 Linux 略有不同, 仅供参考.如果业务生产对内核参数严格要求的, 请参考相关参数的说明, 酌情进行修改配置.

	1. 编辑 `/etc/sysctl.conf` 增加(修改以下参数): 

			vm.dirty_background_ratio = 5
			vm.dirty_ratio = 10

	2. 保存并退出.
	3. 执行命令是上述改动立即生效:

			sysctl -p

如果上述两个方案依然无法解决问题, 请及时联系 Azure 技术支持中心,获取更深入的支持和帮助.


