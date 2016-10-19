
### 基本弹性池限制

| | |
|---|:---:|
| 每个池的 eDTU 上限 | &nbsp;100 &nbsp;&nbsp;&nbsp; 200 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 400 &nbsp;&nbsp;&nbsp;&nbsp; 800 &nbsp;&nbsp;&nbsp;&nbsp; 1200 |
| 每个池的最大存储空间 (GB)*| &nbsp;&nbsp;&nbsp;&nbsp;10 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;20 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;39 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;73 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;117 |
| 每个池的数据库的最大数目 | &nbsp;&nbsp;&nbsp;200 &nbsp;&nbsp;&nbsp;&nbsp;400 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;400 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;400 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;400 |
| 每个池的最大内存 OLTP 存储空间 (GB)| 不适用 |
| 每个池的最大并发工作线程数 | &nbsp;&nbsp;&nbsp;200 &nbsp;&nbsp; 400 &nbsp;&nbsp;&nbsp;&nbsp; 800 &nbsp;&nbsp;&nbsp; 1600 &nbsp;&nbsp;&nbsp;&nbsp;2400 |
| 每个池的最大并发登录数 | &nbsp;&nbsp;&nbsp;200 &nbsp;&nbsp; 400 &nbsp;&nbsp;&nbsp;&nbsp; 800 &nbsp;&nbsp;&nbsp; 1600 &nbsp;&nbsp;&nbsp;&nbsp;2400 |
| 每个池的最大并发会话数 | 4800 &nbsp;9600 &nbsp; 19200 &nbsp; 28800 &nbsp; 28800 |
| 每个数据库的最大 eDTU 数* | 5 |
| 每个数据库的最小 eDTU 数* | 0、5 |
| 每个数据库的最大存储空间 (GB)** | 2 |
| Point-in-time-restore | 过去 7 天内的任一时间点 |
| 灾难恢复 | 活动异地复制 |
|||

* 只要选择的池 DTU 大小至少和每个数据库的最大 eDTU 数相同，就可以将每个数据库的最大和最小 eDTU 数设为任何一个列出的值

**弹性数据库共享池的存储空间，因此数据库存储空间限制为小于池的剩余存储空间和每个数据库的最大存储空间。


### 标准弹性池限制

| | |
|---|:---:|
| 每个池的 eDTU 上限 | &nbsp;100 &nbsp;&nbsp;&nbsp; 200 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 400 &nbsp;&nbsp;&nbsp;&nbsp; 800 &nbsp;&nbsp;&nbsp;&nbsp; 1200 |
| 每个池的最大存储空间 (GB)*| &nbsp;100 &nbsp;&nbsp;&nbsp; 200 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 400 &nbsp;&nbsp;&nbsp;&nbsp; 800 &nbsp;&nbsp;&nbsp;&nbsp; 1200 |
| 每个池的数据库的最大数目 | &nbsp;200 &nbsp;&nbsp;&nbsp;&nbsp;400 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;400 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;400 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;400 |
| 每个池的最大内存 OLTP 存储空间 (GB)| 不适用 |
| 每个池的最大并发工作线程数 | &nbsp;&nbsp;200 &nbsp;&nbsp;&nbsp; 400 &nbsp;&nbsp;&nbsp; 800 &nbsp;&nbsp; 1600 &nbsp;&nbsp;&nbsp; 2400 |
| 每个池的最大并发登录数 | &nbsp;&nbsp;200 &nbsp;&nbsp;&nbsp; 400 &nbsp;&nbsp;&nbsp; 800 &nbsp;&nbsp; 1600 &nbsp;&nbsp;&nbsp; 2400 |
| 每个池的最大并发会话数 | 4800 &nbsp; 9600 &nbsp;19200 &nbsp;28800 &nbsp;&nbsp; 28800 |
| 每个数据库的最大 eDTU 数* | 10、20、50、100 |
| 每个数据库的最小 eDTU 数* | 0、10、20、50、100 |
| 每个数据库的最大存储空间 (GB)** | 250 |
| Point-in-time-restore | 过去 35 天内的任一时间点 |
| 灾难恢复 | 活动异地复制 |
|||

* 只要选择的池 DTU 大小至少和每个数据库的最大 eDTU 数相同，就可以将每个数据库的最大和最小 eDTU 数设为任何一个列出的值

**弹性数据库共享池的存储空间，因此数据库存储空间限制为小于池的剩余存储空间和每个数据库的最大存储空间。

### 高级弹性池限制

| | |
|---|:---:|
| 每个池的 eDTU 上限 | 125 &nbsp;&nbsp;&nbsp; 250 &nbsp;&nbsp;&nbsp; 500 &nbsp;&nbsp;&nbsp; 1000 &nbsp;&nbsp;&nbsp; &nbsp;1500 |
| 每个池的最大存储空间 (GB)*| 250 &nbsp;&nbsp;&nbsp; 500 &nbsp;&nbsp;&nbsp; 750 &nbsp;&nbsp;&nbsp;&nbsp; 750 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 750 |
| 每个池的数据库的最大数目 | 50 |
| 每个池的最大内存 OLTP 存储空间 (GB)| 不适用 |
| 每个池的最大并发工作线程数 | &nbsp;&nbsp;200 &nbsp;&nbsp;&nbsp; 400 &nbsp;&nbsp;&nbsp; 800 &nbsp;&nbsp; 1600 &nbsp;&nbsp;&nbsp; 2400 |
| 每个池的最大并发登录数 | &nbsp;&nbsp;200 &nbsp;&nbsp;&nbsp; 400 &nbsp;&nbsp;&nbsp; 800 &nbsp;&nbsp; 1600 &nbsp;&nbsp;&nbsp; 2400 |
| 每个池的最大并发会话数 | 4800 &nbsp; 9600 &nbsp;19200 &nbsp;28800 &nbsp;&nbsp; 28800 |
| 每个数据库的最大 eDTU 数* | 125、250、500、1000 |
| 每个数据库的最小 eDTU 数* | 0、125、250、500、1000 |
| 每个数据库的最大存储空间 (GB)** | 500 |
| Point-in-time-restore | 过去 35 天内的任一时间点 |
| 灾难恢复 | 活动异地复制 |
|||

* 只要选择的池 DTU 大小至少和每个数据库的最大 eDTU 数相同，就可以将每个数据库的最大和最小 eDTU 数设为任何一个列出的值

**弹性数据库共享池的存储空间，因此数据库存储空间限制为小于池的剩余存储空间和每个数据库的最大存储空间。

<!---HONumber=Mooncake_1010_2016-->