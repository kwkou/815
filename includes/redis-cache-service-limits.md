| 资源 | 限制 |
| --- | --- |
| 缓存大小 |530 GB |
| 数据库 |64 |
| 连接的最大客户端数 |40,000 |
| Redis 缓存副本（用于高可用性） |1 |
| 启用群集的高级缓存中的分片数 |10 |

每个定价层的 Azure Redis 缓存限制和大小都不相同。 若要查看定价层及其关联的大小，请参阅 [Azure Redis 缓存定价](/pricing/details/redis-cache/)。

有关 Azure Redis 缓存配置限制的详细信息，请参阅[默认 Redis 服务器配置](/documentation/articles/cache-configure/#default-redis-server-configuration)。

由于 Azure Redis 缓存实例的配置和管理是由 Microsoft 执行的，因此 Azure Redis 缓存不一定支持所有的 Redis 命令。 有关详细信息，请参阅 [Azure Redis 缓存中不支持的 Redis 命令](/documentation/articles/cache-configure/#redis-commands-not-supported-in-azure-redis-cache)。