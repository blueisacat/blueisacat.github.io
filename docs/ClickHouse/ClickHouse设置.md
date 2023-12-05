---
layout: default
title: ClickHouse设置
parent: ClickHouse
nav_order: 5
---

# 全局服务器设置项

全局服务器设置项无法通过会话级别或者查询级别进行修改，仅可通过ClickHouse服务器上的`config.xml`文件进行修改。

***

## allow_use_jemalloc_memory

允许使用jemalloc内存。

**类型**：`Bool`

**默认值**：`1`

***

## asynchronous_heavy_metrics_update_period_s

更新异步指标的周期（以秒为单位）。

***

**类型**：`UInt32`

**默认值**：`120`

***

## asynchronous_metrics_update_period_s

更新异步指标的周期（以秒为单位）。

**类型**：`UInt32`

**默认值**：`1`

***

## background_buffer_flush_schedule_pool_size

用于在后台对Buffer引擎表执行刷新操作的最大线程数。

**类型**：`UInt64`

**默认值**：`16`

***

## background_common_pool_size

用于在后台对 *MergeTree 引擎表执行各种操作（主要是垃圾收集）的最大线程数。

**类型**：`UInt64`

**默认值**：`8`

***

## background_distributed_schedule_pool_size

用于执行分布式发送的最大线程数。

**类型**：`UInt64`

**默认值**：`16`

***

## background_fetches_pool_size

用于在后台从 *MergeTree 引擎表的另一个副本获取数据部分的最大线程数。

**类型**：`UInt64`

**默认值**：`16`

***

## background_merges_mutations_concurrency_ratio

设置线程数与可以同时执行的后台合并和突变数之间的比率。 例如，如果比率等于2并且`background_pool_size`设置为16，那么ClickHouse可以同时执行32个后台合并。 这是可能的，因为后台操作可以暂停和推迟。 这是为小型合并提供更多执行优先级所必需的。 您只能在运行时增加此比率。 要降低它，您必须重新启动服务器。 与`background_pool_size`设置相同的`background_merges_mutations_concurrency_ratio`可以从默认配置文件中应用以实现向后兼容性。

**类型**：`Float`

**默认值**：`2`

***

## background_merges_mutations_scheduling_policy

关于如何执行后台合并和突变调度的策略。可能的值为：`round_robin` 和`shortest_task_first`。

***

## background_merges_mutations_scheduling_policy

用于选择后台线程池执行的下一个合并或突变的算法。策略可以在运行时更改，无需重新启动服务器。可以从`默认`配置文件中应用以实现向后兼容性。

可能的值：

* “round_robin” — 每个并发合并和变异都按循环顺序执行，以确保无饥饿操作。较小的合并比较大的合并完成得更快，因为它们需要合并的块较少。

* “shortest_task_first” — 始终执行较小的合并或突变。 合并和突变根据其结果大小分配优先级。 较小尺寸的合并优先于较大尺寸的合并。 此策略可确保以最快的速度合并小部分，但可能会导致 INSERT 严重过载的分区中的大合并无限期匮乏。

**类型**：`String`

**默认值**：`round_robin`

***

## background_message_broker_schedule_pool_size

用于执行消息流后台操作的最大线程数。

**类型**：`UInt64`

**默认值**：`16`

***

## background_move_pool_size

用于在后台将 *MergeTree 引擎表的数据部分移动到另一个磁盘或卷的最大线程数。

**类型**：`UInt64`

**默认值**：`8`

***

## background_pool_size

设置使用 MergeTree 引擎对表执行后台合并和突变的线程数。 您只能在运行时增加线程数。 要减少线程数，您必须重新启动服务器。 通过调整此设置，您可以管理 CPU 和磁盘负载。 较小的池大小使用较少的 CPU 和磁盘资源，但后台进程进展较慢，最终可能会影响查询性能。
在更改之前，还请查看相关的 MergeTree 设置，例如 `number_of_free_entries_in_pool_to_lower_max_size_of_merge` 和 `number_of_free_entries_in_pool_to_execute_mutation`。

**类型**：`UInt64`

**默认值**：`16`

***

## background_schedule_pool_size

用于不断执行复制表、Kafka 流和 DNS 缓存更新的一些轻量级周期性操作的最大线程数。

**类型**：`UInt64`

**默认值**：`512`

***

## backup_threads

执行 BACKUP 请求的最大线程数。

**类型**：`UInt64`

**默认值**：`16`

***

## backups_io_thread_pool_queue_size
备份 IO 线程池上可以调度的最大作业数。由于当前的 S3 备份逻辑，建议保持此队列不受限制 (0)。

**类型**：`UInt64`

**默认值**：`0`

***

## cache_size_to_ram_max_ratio

将缓存大小设置为 RAM 最大比率。允许降低低内存系统上的缓存大小。

**类型**：`Double`

**默认值**：`0.5`

***

## concurrent_threads_soft_limit_num

允许运行所有查询的最大查询处理线程数（不包括用于从远程服务器检索数据的线程）。 这不是硬性限制。 万一达到限制，查询仍将至少有一个线程运行。 如果有更多线程可用，查询可以在执行期间扩展到所需的线程数。

零意味着无限。

**类型**：`UInt64`

**默认值**：`0`

***

## concurrent_threads_soft_limit_ratio_to_cores

与 **concurrent_threads_soft_limit_num** 相同，但具有与核心的比率。

**类型**：`UInt64`

**默认值**：`0`

***

## default_database

默认数据库名称。

**类型**：`String`

**默认值**：`default`

***

## disable_internal_dns_cache

禁用内部 DNS 缓存。建议在基础设施经常变化的系统（例如 Kubernetes）中运行 ClickHouse。

**类型**：`Bool`

**默认值**：`0`

***

## dns_cache_update_period

内部 DNS 缓存更新周期（以秒为单位）。

**类型**：`Int32`

**默认值**：`15`

***

## dns_max_consecutive_failures

从 ClickHouse DNS 缓存中删除主机之前的最大连续解析失败次数。

**类型**：`UInt32`

**默认值**：`10`

***

## index_mark_cache_policy

索引标记缓存策略名称。

**类型**：`String`

**默认值**：`SLRU`

***

## index_mark_cache_size

索引标记缓存的大小。零表示禁用。

> 该设置可以在运行时修改并立即生效。

**类型**：`UInt64`

**默认值**：`0`

***

## index_mark_cache_size_ratio

索引标记高速缓存中受保护队列的大小相对于高速缓存的总大小。

**类型**：`Double`

**默认值**：`0.5`

***

## index_uncompressed_cache_policy

索引未压缩缓存策略名称。

**类型**：`String`

**默认值**：`SLRU`

***

## index_uncompressed_cache_size

MergeTree 索引未压缩块的缓存大小。零表示禁用。

> 该设置可以在运行时修改并立即生效。

**类型**：`UInt64`

**默认值**：`0`

***

## index_uncompressed_cache_size_ratio

索引未压缩缓存中受保护队列的大小相对于缓存总大小的大小。

**类型**：`Double`

**默认值**：`0.5`

***

## io_thread_pool_queue_size

IO 线程池的队列大小。零意味着无限。

**类型**：`UInt64`

**默认值**：`10000`

***

## mark_cache_policy

标记缓存策略名称。

**类型**：`String`

**默认值**：`SLRU`

***

## mark_cache_size

标记缓存的大小（MergeTree 系列表的索引）。

> 该设置可以在运行时修改并立即生效。

**类型**：`UInt64`

**默认值**：`5368709120`

***

## mark_cache_size_ratio

标记缓存中受保护队列的大小相对于缓存总大小。

**类型**：`Double`

**默认值**：`0.5`

***

## max_backup_bandwidth_for_server

服务器上所有备份的最大读取速度（以字节/秒为单位）。零意味着无限。

**类型**：`UInt64`

**默认值**：`0`

***

## max_backups_io_thread_pool_free_size

如果Backups IO Thread池中的空闲线程数量超过`max_backup_io_thread_pool_free_size`，ClickHouse将释放空闲线程占用的资源并减小池大小。如果需要，可以再次创建线程。

**类型**：`UInt64`

**默认值**：`0`

***

## max_backups_io_thread_pool_size

用于 BACKUP 查询 IO 操作的最大线程数。

**类型**：`UInt64`

**默认值**：`1000`

***

## max_concurrent_queries

并发执行查询总数的限制。 零意味着无限。 请注意，还必须考虑对插入和选择查询以及用户最大查询数的限制。 另请参见 max_concurrent_insert_queries、max_concurrent_select_queries、max_concurrent_queries_for_all_users。 零意味着无限。

> 该设置可以在运行时修改并立即生效。已经运行的查询将保持不变。

**类型**：`UInt64`

**默认值**：`0`

***

## max_concurrent_insert_queries

并发插入查询总数的限制。零意味着无限。

> 该设置可以在运行时修改并立即生效。已经运行的查询将保持不变。

**类型**：`UInt64`

**默认值**：`0`

***

## max_concurrent_select_queries

并发选择查询总数的限制。零意味着无限。

> 该设置可以在运行时修改并立即生效。已经运行的查询将保持不变。

**类型**：`UInt64`

**默认值**：`0`

***

## max_connections

最大服务器连接数。

**类型**：`Int32`

**默认值**：`1024`

***

## max_io_thread_pool_free_size

IO 线程池的最大可用大小。

**类型**：`UInt64`

**默认值**：`0`

***

## max_io_thread_pool_size

用于 IO 操作的最大线程数。

**类型**：`UInt64`

**默认值**：`100`

***

## max_local_read_bandwidth_for_server

本地读取的最大速度（以字节/秒为单位）。零意味着无限。

**类型**：`UInt64`

**默认值**：`0`

***

## max_local_write_bandwidth_for_server

本地写入的最大速度（以字节/秒为单位）。零意味着无限。

**类型**：`UInt64`

**默认值**：`0`

***

## max_partition_size_to_drop

限制删除分区。

如果 MergeTree 表的大小超过 `max_partition_size_to_drop` （以字节为单位），则无法使用 DROP PARTITION 查询删除分区。此设置不需要重新启动 Clickhouse 服务器即可应用。禁用限制的另一种方法是创建 `<clickhouse-path>/flags/force_drop_table` 文件。**默认值**：`50 GB`。值 0 表示您可以不受任何限制地删除分区。

> 此限制不限制 drop table 和 truncate table，请参阅 **max_table_size_to_drop**

## max_remote_read_network_bandwidth_for_server

通过网络进行数据交换的最大读取速度（以字节/秒为单位）。零意味着无限。

**类型**：`UInt64`

**默认值**：`0`

***

## max_remote_write_network_bandwidth_for_server

通过网络进行写入的数据交换的最大速度（以字节/秒为单位）。零意味着无限。

**类型**：`UInt64`

**默认值**：`0`

***

## max_server_memory_usage

总内存使用量限制。零意味着无限。

默认 `max_server_memory_usage` 值的计算方式为：`memory_amount * max_server_memory_usage_to_ram_ratio`。

**类型**：`UInt64`

**默认值**：`0`

***

## max_server_memory_usage_to_ram_ratio

与 `max_server_memory_usage` 相同，但与物理 RAM 成比例。允许降低低内存系统上的内存使用量。零意味着无限。

在 RAM 和交换空间较低的主机上，您可能需要将 `max_server_memory_usage_to_ram_ratio` 设置为大于 1。

**类型**：`Double`

**默认值**：`0.9`

***

## max_table_size_to_drop

删除表的限制。

如果 MergeTree 表的大小超过 `max_table_size_to_drop` （以字节为单位），则无法使用 DROP 查询或 TRUNCATE 查询删除它。

此设置不需要重新启动 Clickhouse 服务器即可应用。禁用限制的另一种方法是创建 `<clickhouse-path>/flags/force_drop_table` 文件。

**默认值**：`50 GB`。值 0 表示您可以删除所有表，没有任何限制。

**例子**

```xml
<max_table_size_to_drop>0</max_table_size_to_drop>
```

## max_temporary_data_on_disk_size

可用于外部聚合、联接或排序的最大存储量。超过此限制的查询将失败并出现异常。零意味着无限。

另请参阅 `max_temporary_data_on_disk_size_for_user` 和 `max_temporary_data_on_disk_size_for_query`。

**类型**：`UInt64`

**默认值**：`0`

***

## max_thread_pool_free_size

如果全局线程池中的空闲线程数大于 `max_thread_pool_free_size`，则 ClickHouse 会释放部分线程占用的资源，并减小池大小。如果需要，可以再次创建线程。

**类型**：`UInt64`

**默认值**：`1000`

***

## max_thread_pool_size

可以从操作系统分配并用于查询执行和后台操作的最大线程数。

**类型**：`UInt64`

**默认值**：`10000`

***

## mmap_cache_size

设置映射文件的缓存大小（以字节为单位）。此设置可以避免频繁的打开/关闭调用（由于随之而来的页面错误，这非常昂贵），并可以重用来自多个线程和查询的映射。设置值为映射区域的数量（通常等于映射文件的数量）。可以使用 `MMappedFiles` 和 `MMappedFileBytes` 指标在表 system.metrics 和 system.metric_log 中监视映射文件中的数据量。此外，在 system.asynchronous_metrics 和 system.asynchronous_metrics_log 中通过 `MMapCacheCells` 指标，在 system.events、system.processes、system.query_log、system.query_thread_log、system.query_views_log 中通过 `CreatedReadBufferMMap`、`CreatedReadBufferMMapFailed`、`MMappedFileCacheHits`、`MMappedFileCacheMisses` 事件。

请注意，映射文件中的数据量不会直接消耗内存，并且不会计入查询或服务器内存使用量中，因为该内存可以像操作系统页面缓存一样被丢弃。在删除 MergeTree 系列表中的旧部分时，缓存会自动删除（文件被关闭），也可以通过 `SYSTEM DROP MMAP CACHE` 查询手动删除。

> 该设置可以在运行时修改并立即生效。

**类型**：`UInt64`

**默认值**：`1000`

***

## restore_threads

执行 RESTORE 请求的最大线程数。

**类型**：`UInt64`

**默认值**：`16`

***

## show_addresses_in_stack_traces

如果设置为 true 将在堆栈跟踪中显示地址。

**类型**：`Bool`

**默认值**：`1`

***

## shutdown_wait_unfinished_queries

如果设置为 true，ClickHouse 将在关闭之前等待正在运行的查询完成。

**类型**：`Bool`

**默认值**：`0`

***

## temporary_data_in_cache

使用此选项，临时数据将存储在特定磁盘的缓存中。在本节中，您应该指定具有类型缓存的磁盘名称。在这种情况下，缓存和临时数据将共享相同的空间，并且可以逐出磁盘缓存以创建临时数据。

> 只能使用一个选项来配置临时数据存储：`tmp_path`、`tmp_policy`、`temporary_data_in_cache`。

**例子**

`local_disk` 的缓存和临时数据都将存储在文件系统上的 `/tiny_local_cache` 中，由`tiny_local_cache` 管理。

```xml
<clickhouse>
    <storage_configuration>
        <disks>
            <local_disk>
                <type>local</type>
                <path>/local_disk/</path>
            </local_disk>

            <tiny_local_cache>
                <type>cache</type>
                <disk>local_disk</disk>
                <path>/tiny_local_cache/</path>
                <max_size_rows>10M</max_size_rows>
                <max_file_segment_size>1M</max_file_segment_size>
                <cache_on_write_operations>1</cache_on_write_operations>
            </tiny_local_cache>
        </disks>
    </storage_configuration>

    <temporary_data_in_cache>tiny_local_cache</temporary_data_in_cache>
</clickhouse>
```

**类型**：`String`

**默认值**：

***

## thread_pool_queue_size

全局线程池上可以调度的最大作业数。增加队列大小会导致更大的内存使用量。建议将此值保持等于 `max_thread_pool_size`。零意味着无限。

**类型**：`UInt64`

**默认值**：`10000`

***

## tmp_policy

临时数据的存储策略。另请参阅 MergeTree 表引擎文档。

> * 只能使用一个选项来配置临时数据存储：`tmp_path`、`tmp_policy`、`temporary_data_in_cache`。
> * `move_factor`、`keep_free_space_bytes`、`max_data_part_size_bytes` 被忽略。
> * 策略应该只有一个带有本地磁盘的卷。

**例子**

当 `/disk1` 已满时，临时数据将存储在 `/disk2` 上。

```xml
<clickhouse>
    <storage_configuration>
        <disks>
            <disk1>
                <path>/disk1/</path>
            </disk1>
            <disk2>
                <path>/disk2/</path>
            </disk2>
        </disks>

        <policies>
            <tmp_two_disks>
                <volumes>
                    <main>
                        <disk>disk1</disk>
                        <disk>disk2</disk>
                    </main>
                </volumes>
            </tmp_two_disks>
        </policies>
    </storage_configuration>

    <tmp_policy>tmp_two_disks</tmp_policy>
</clickhouse>
```

**类型**：`String`

**默认值**：

***

## uncompressed_cache_policy

未压缩的缓存策略名称。

**类型**：`String`

**默认值**：`SLRU`

***

## uncompressed_cache_size

MergeTree 系列表引擎使用的未压缩数据的缓存大小（以字节为单位）。零表示禁用。

服务器有一个共享缓存。内存是按需分配的。如果启用了 use_uncompressed_cache 选项，则使用缓存。

未压缩的缓存对于个别情况下非常短的查询是有利的。

> 该设置可以在运行时修改并立即生效。

**类型**：`UInt64`

**默认值**：`0`

***

## uncompressed_cache_size_ratio

未压缩缓存中受保护队列的大小相对于缓存总大小的比例。

**类型**：`Double`

**默认值**：`0.5`

***


## builtin_dictionaries_reload_interval

重新加载内置词典之前的时间间隔（以秒为单位）。

ClickHouse 每 x 秒重新加载内置词典。这使得无需重新启动服务器即可“即时”编辑词典。

例子

```xml
<builtin_dictionaries_reload_interval>3600</builtin_dictionaries_reload_interval>
```

**默认值**：`3600`

***

## compression

MergeTree 引擎表的数据压缩设置。

> 如果您刚刚开始使用 ClickHouse，请勿使用它。

配置模板：

```xml
<compression>
    <case>
      <min_part_size>...</min_part_size>
      <min_part_size_ratio>...</min_part_size_ratio>
      <method>...</method>
      <level>...</level>
    </case>
    ...
</compression>
```

`<case>` 字段：

* `min_part_size` – 数据部分的最小大小。

* `min_part_size_ratio` – 数据部分大小与表大小的比率。

* `method` – 压缩方法。可接受的值：`lz4`、`lz4hc`、`zstd`、`deflate_qpl`。

* `level` – 压缩级别。

您可以配置多个 `<case>` 部分。

满足条件时的操作：

* 如果数据部分符合条件集，ClickHouse 将使用指定的压缩方法。

* 如果一个数据部分匹配多个条件集，ClickHouse 将使用第一个匹配的条件集。

如果数据部分不满足条件，ClickHouse 将使用 `lz4` 压缩。

**例子**

```xml
<compression incl="clickhouse_compression">
    <case>
        <min_part_size>10000000000</min_part_size>
        <min_part_size_ratio>0.01</min_part_size_ratio>
        <method>zstd</method>
        <level>1</level>
    </case>
</compression>
```

***

## encryption

配置命令以获取加密编解码器使用的密钥。密钥（或多个密钥）应写入环境变量或在配置文件中设置。

密钥可以是十六进制或长度等于 16 字节的字符串。

**例子**

从配置加载：

```xml
<encryption_codecs>
    <aes_128_gcm_siv>
        <key>1234567812345678</key>
    </aes_128_gcm_siv>
</encryption_codecs>
```

> 不建议将密钥存储在配置文件中。它不安全。您可以将密钥移至安全磁盘上的单独配置文件中，并将指向该配置文件的符号链接放入 `config.d/` 文件夹中。

当密钥为十六进制时，从配置加载：

```xml
<encryption_codecs>
    <aes_128_gcm_siv>
        <key_hex>00112233445566778899aabbccddeeff</key_hex>
    </aes_128_gcm_siv>
</encryption_codecs>
```

从环境变量加载密钥：

```xml
<encryption_codecs>
    <aes_128_gcm_siv>
        <key_hex from_env="ENVVAR"></key_hex>
    </aes_128_gcm_siv>
</encryption_codecs>
```

这里`current_key_id`设置当前用于加密的密钥，所有指定的密钥都可以用于解密。

这些方法中的每一种都可以应用于多个键：

```xml
<encryption_codecs>
    <aes_128_gcm_siv>
        <key_hex id="0">00112233445566778899aabbccddeeff</key_hex>
        <key_hex id="1" from_env="ENVVAR"></key_hex>
        <current_key_id>1</current_key_id>
    </aes_128_gcm_siv>
</encryption_codecs>
```

这里 `current_key_id` 显示当前加密密钥。

此外，用户还可以添加长度必须为 12 字节的随机数（默认情况下，加密和解密过程使用由零字节组成的随机数）：

```xml
<encryption_codecs>
    <aes_128_gcm_siv>
        <nonce>012345678910</nonce>
    </aes_128_gcm_siv>
</encryption_codecs>
```

或者可以设置为十六进制：

```xml
<encryption_codecs>
    <aes_128_gcm_siv>
        <nonce_hex>abcdefabcdef</nonce_hex>
    </aes_128_gcm_siv>
</encryption_codecs>
```

上面提到的所有内容都可以应用于`aes_256_gcm_siv`（但密钥必须是32字节长）。

***

## custom_settings_prefixes

自定义设置的前缀列表。前缀必须用逗号分隔。

**例子**

```xml
<custom_settings_prefixes>custom_</custom_settings_prefixes>
```

***

## core_dump

配置核心转储文件大小的软限制。

**可能的值**：

* 正整数

**默认值**：`1073741824 (1 GB)`

> 硬限制通过系统工具配置

**例子**

```xml
<core_dump>
    <size_limit>1073741824</size_limit>
</core_dump>
```

***

## database_atomic_delay_before_drop_table_sec

设置删除表数据之前的延迟（以秒为单位）。如果查询具有 `SYNC` 修饰符，则忽略此设置。

**默认值**：`480 (8 minute)`

***

## database_catalog_unused_dir_hide_timeout_sec

从 `store/` 目录清理垃圾的任务的参数。如果某些子目录未被 clickhouse-server 使用，并且该目录在最后的 `database_catalog_unused_dir_hide_timeout_sec` 秒内没有被修改，则该任务将通过删除所有访问权限来“隐藏”该目录。它也适用于 clickhouse-server 不希望在 `store/` 中看到的目录。零意味着“立即”。

**默认值**：`3600 (1 hour)`

***

## database_catalog_unused_dir_rm_timeout_sec

从 `store/` 目录清理垃圾的任务的参数。如果某些子目录未被 clickhouse-server 使用并且之前已“隐藏”（请参阅`database_catalog_unused_dir_hide_timeout_sec`），并且在最后的`database_catalog_unused_dir_rm_timeout_sec` 秒内未修改此目录，则任务将删除此目录。它也适用于 clickhouse-server 不希望在 `store/` 中看到的目录。零意味着“从不”。

**默认值**：`2592000 (30 days)`

***

## database_catalog_unused_dir_cleanup_period_sec

从 `store/` 目录清理垃圾的任务的参数。设置任务的调度周期。零意味着“从不”。

**默认值**：`86400 (1 day)`

***

## default_profile

默认设置配置文件。

设置配置文件位于参数 `user_config` 中指定的文件中。

***

**例子**

```xml
<default_profile>default</default_profile>
```

***

## default_replica_path

ZooKeeper 中表的路径。

**例子**

```xml
<default_replica_path>/clickhouse/tables/{uuid}/{shard}</default_replica_path>
```

***

## default_replica_name

ZooKeeper 中的副本名称。

**例子**

```xml
<default_replica_name>{replica}</default_replica_name>
```

***

## dictionaries_config

字典配置文件的路径。

路径：

* 指定绝对路径或相对于服务器配置文件的路径。

* 该路径可以包含通配符 * 和 ?。

**例子**

```xml
<dictionaries_config>*_dictionary.xml</dictionaries_config>
```

***

## user_defined_executable_functions_config

可执行用户定义函数的配置文件的路径。

路径：

* 指定绝对路径或相对于服务器配置文件的路径。

* 该路径可以包含通配符 * 和 ?。

```xml
<user_defined_executable_functions_config>*_function.xml</user_defined_executable_functions_config>
```

***

## dictionaries_lazy_load

字典的延迟加载。

如果为 `true`，则每个字典都会在第一次使用时加载。如果加载失败，使用字典的函数将引发异常。

如果为 `false`，则服务器在启动时加载所有字典。服务器将在启动时等待，直到所有词典完成加载后再接收任何连接（例外：如果 `wait_dictionaries_load_at_startup` 设置为 `false` - 见下文）。

**默认值**：`true`

**例子**

```xml
<dictionaries_lazy_load>true</dictionaries_lazy_load>
```

***

## format_schema_path

包含输入数据方案的目录路径，例如 CapnProto 格式的方案。

**例子**

```xml
<!-- Directory containing schema files for various input formats. -->
<format_schema_path>format_schemas/</format_schema_path>
```

***

## graphite

向 Graphite 发送数据。

设置：

* host – Graphite服务器。

* port – Graphite 服务器上的端口。

* interval – 发送的时间间隔，单位为秒。

* timeout – 发送数据的超时时间，以秒为单位。

* root_path – 键的前缀。

* metrics – 从 system.metrics 表发送数据。

* events – 发送 system.events 表中在该时间段内累积的增量数据。

* events_cumulative – 从 system.events 表发送累积数据。

* asynchronous_metrics – 从 system.asynchronous_metrics 表发送数据。

您可以配置多个 `<graphite>` 子句。例如，您可以使用它以不同的时间间隔发送不同的数据。

**例子**

```xml
<graphite>
    <host>localhost</host>
    <port>42000</port>
    <timeout>0.1</timeout>
    <interval>60</interval>
    <root_path>one_min</root_path>
    <metrics>true</metrics>
    <events>true</events>
    <events_cumulative>false</events_cumulative>
    <asynchronous_metrics>true</asynchronous_metrics>
</graphite>
```

***

## graphite_rollup

Graphite细化数据的设置。

有关更多详细信息，请参阅 GraphiteMergeTree。

**例子**

```xml
<graphite_rollup_example>
    <default>
        <function>max</function>
        <retention>
            <age>0</age>
            <precision>60</precision>
        </retention>
        <retention>
            <age>3600</age>
            <precision>300</precision>
        </retention>
        <retention>
            <age>86400</age>
            <precision>3600</precision>
        </retention>
    </default>
</graphite_rollup_example>
```

***

## http_port/https_port

用于通过 HTTP 连接到服务器的端口。

如果指定了 `https_port`，则必须配置 openSSL。

如果指定了 `http_port`，则即使设置了 OpenSSL 配置，也会被忽略。

**例子**

```xml
<https_port>9999</https_port>
```

***

## http_server_default_response

访问 ClickHouse HTTP(s) 服务器时默认显示的页面。默认值为`Ok`。 （末尾有换行符）

**例子**

访问 `http://localhost: http_port` 时打开 `https://tabix.io/`。

```xml
<http_server_default_response>
  <![CDATA[<html ng-app="SMI2"><head><base href="http://ui.tabix.io/"></head><body><div ui-view="" class="content-ui"></div><script src="http://loader.tabix.io/master.js"></script></body></html>]]>
</http_server_default_response>
```

***

## hsts_max_age

HSTS 的过期时间（以秒为单位）。默认值为 `0` 表示 clickhouse 禁用 HSTS。如果您设置一个正数，则 HSTS 将启用，并且 max-age 是您设置的数字。

**例子**

```xml
<hsts_max_age>600000</hsts_max_age>
```

***

## include_from

包含替换的文件的路径。

有关详细信息，请参阅“配置文件”部分。

**例子**

```xml
<include_from>/etc/metrica.xml</include_from>
```

***

## interserver_listen_host

对可以在 ClickHouse 服务器之间交换数据的主机的限制。如果使用Keeper，不同Keeper实例之间的通信也会受到同样的限制。默认值等于`listen_host`设置。

**例子**

```xml
<interserver_listen_host>::ffff:a00:1</interserver_listen_host>
<interserver_listen_host>10.0.0.1</interserver_listen_host>
```

***

## interserver_http_port

ClickHouse 服务器之间交换数据的端口。

**例子**

```xml
<interserver_http_port>9009</interserver_http_port>
```

***

## interserver_http_host

其他服务器可以用来访问该服务器的主机名。

如果省略，则其定义方式与 `hostname -f` 命令相同。

对于脱离特定的网络接口很有用。

**例子**

```xml
<interserver_http_host>example.clickhouse.com</interserver_http_host>
```

***

## interserver_https_port

用于通过 `HTTPS` 在 ClickHouse 服务器之间交换数据的端口。

**例子**

```xml
<interserver_https_port>9010</interserver_https_port>
```

***

## interserver_https_host

与 `interserver_http_host` 类似，不同之处在于该主机名可以被其他服务器用来通过 `HTTPS` 访问该服务器。

**例子**

```xml
<interserver_https_host>example.clickhouse.com</interserver_https_host>
```

***

## interserver_http_credentials

用于在复制期间连接到其他服务器的用户名和密码。服务器还使用这些凭据对其他副本进行身份验证。因此，集群中所有副本的 `interserver_http_credentials` 必须相同。

默认情况下，如果省略 `interserver_http_credentials` 部分，则在复制期间不使用身份验证。

> `interserver_http_credentials` 设置与 ClickHouse 客户端凭据配置无关。

> 这些凭据对于通过 `HTTP` 和 `HTTPS` 进行复制很常见。

该部分包含以下参数：

* `user` — 用户名

* `password` — 密码

* `allow_empty` — 如果为 `true`，则即使设置了凭据，也允许其他副本无需身份验证即可连接。如果为 false，则拒绝未经身份验证的连接。默认值：`false`。

* `old` — 包含凭证轮换期间使用的旧`用户`和`密码`。可以指定几个旧部分。

**凭证轮换**

ClickHouse 支持动态服务器间凭证轮换，无需同时停止所有副本来更新其配置。可以通过几个步骤更改凭据。

要启用身份验证，请将 `interserver_http_credentials.allow_empty` 设置为 `true` 并添加凭据。这允许有身份验证和无身份验证的连接。

```xml
<interserver_http_credentials>
    <user>admin</user>
    <password>111</password>
    <allow_empty>true</allow_empty>
</interserver_http_credentials>
```

配置所有副本后，将 `allow_empty` 设置为`false` 或删除此设置。它强制使用新凭据进行身份验证。

要更改现有凭据，请将用户名和密码移至 `interserver_http_credentials.old` 部分，并使用新值更新`用户`和`密码`。此时，服务器使用新凭据连接到其他副本，并接受使用新或旧凭据的连接。

```xml
<interserver_http_credentials>
    <user>admin</user>
    <password>222</password>
    <old>
        <user>admin</user>
        <password>111</password>
    </old>
    <old>
        <user>temp</user>
        <password>000</password>
    </old>
</interserver_http_credentials>
```

当新凭证应用于所有副本时，旧凭证可能会被删除。

***

## keep_alive_timeout

ClickHouse 在关闭连接之前等待传入请求的秒数。默认为 10 秒。

**例子**

```xml
<keep_alive_timeout>10</keep_alive_timeout>
```

***

## listen_host

对请求可以来自的主机的限制。如果您希望服务器回答所有问题，请指定 `::`。

**例子**

```xml
<listen_host>::1</listen_host>
<listen_host>127.0.0.1</listen_host>
```

***

## listen_backlog

侦听套接字的积压（待处理连接的队列大小）。

**默认值**：`4096 (与linux 5.4+一致)`

通常不需要更改该值，因为：

* 默认值足够大，

* 并且为了接受客户端的连接，服务器有单独的线程。

因此，即使您的 `TcpExtListenOverflows`（来自 `nstat`）非零并且该计数器随着 ClickHouse 服务器的增加而增加，也不意味着该值需要增加，因为：

* 通常，如果 4096 还不够，它会显示一些内部 ClickHouse 扩展问题，因此最好报告问题。

* 这并不意味着服务器以后可以处理更多连接（即使可以，到那时客户端可能会消失或断开连接）。

**例子**

```xml
<listen_backlog>4096</listen_backlog>
```

***

## logger

记录设置。

键：

`level` – 记录级别。可接受的值：`trace` `debug` `information` `warning` `error`。

`log` – 日志文件。包含按级别排列的所有条目。

`errorlog` – 错误日志文件。

`size` – 文件的大小。适用于日志和错误日志。一旦文件达到大小，ClickHouse 就会对其进行归档并重命名，并在其位置创建一个新的日志文件。

`count` – ClickHouse 存储的归档日志文件的数量。

`console` – 将日志和错误日志发送到控制台而不是文件。要启用，请设置为 `1` 或 `true`。

`stream_compress` – 使用 lz4 流压缩来压缩日志和错误日志。要启用，请设置为 `1` 或 `true`。

日志和错误日志文件名（仅文件名，不支持目录）都支持日期和时间格式说明符。

**格式说明符** 使用以下格式说明符，您可以定义结果文件名的模式。 “示例”列显示 `2023-07-06 18:32:07` 的可能结果。

|说明符|描述|例子|
|--|---|---|
|%%|字面量%|%|
|%n|换行符||
|%t|水平制表符||	
|%Y|年份为十进制数，例如2017年|2023|
|%y|年份的最后 2 位十进制数（范围 [00,99]）|23|
|%C|年份的前 2 位十进制数（范围 [00,99]）|20|
|%G|四位数 ISO 8601 基于周的年份，即包含指定周的年份。通常仅与 %V 一起使用|2023|
|%g|ISO 8601 基于周的年份的最后 2 位数字，即包含指定周的年份。|Jul|
|%b|月份名称缩写，例如十月（取决于区域设置）|Jul|
|%h|%b 的同义词|Jul|
|%B|完整的月份名称，例如十月（取决于区域设置）|July|
|%m|十进制数形式的月份（范围 [01,12]）|07|
|%U|一年中的第几周，以十进制数表示（星期日是一周的第一天）（范围 [00,53]）|27|
|%W|一年中的第几周，以十进制数表示（星期一是一周的第一天）（范围 [00,53]）|27|
|%V|ISO 8601 周数（范围 [01,53]）|27|
|%j|一年中的第几天，十进制数（范围 [001,366]）|187|
|%d|月份中的某一天，以零填充的十进制数表示（范围 [01,31]）。单个数字前面有零。|06|
|%e|以空格填充的十进制数表示的月份中的日期（范围 [1,31]）。单个数字前面有一个空格。|6|
|%a|工作日名称缩写，例如周五（取决于区域设置）|Thu|
|%A|工作日的完整名称，例如星期五（取决于区域设置）|Thursday|
|%w|工作日为整数，星期日为 0（范围 [0-6]）|4|
|%u|十进制数形式的工作日，其中星期一为 1（ISO 8601 格式）（范围 [1-7]）|4|
|%H|小时为十进制数，24 小时制（范围 [00-23]）|18|
|%I|小时为十进制数，12 小时制（范围 [01,12]）|06|
|%M|分钟为十进制数（范围 [00,59]）|32|
|%S|秒为十进制数（范围 [00,60]）|07|
|%c|标准日期和时间字符串，例如2010 年 10 月 17 日星期日 04:41:13（取决于区域设置）|Thu Jul 6 18:32:07 2023|
|%x|本地化日期表示（取决于区域设置）|07/06/23|
|%X|本地化时间表示，例如18:40:20 或 6:40:20 PM（取决于区域设置）|18:32:07|
|%D|短 MM/DD/YY 日期，相当于 %m/%d/%y|07/06/23|
|%F|短 YYYY-MM-DD 日期，相当于 %Y-%m-%d|2023-07-06|
|%r|本地化 12 小时制时间（取决于区域设置）|06:32:07 PM|
|%R|相当于“%H:%M”|18:32|
|%T|相当于“%H:%M:%S”（ISO 8601 时间格式）|18:32:07|
|%p|本地化上午或下午指定（取决于区域设置）|PM|
|%z|采用 ISO 8601 格式的 UTC 偏移量（例如 -0430），或者如果时区信息不可用则不包含字符|+0800|
|%Z|与区域设置相关的时区名称或缩写，或者如果时区信息不可用则不包含字符|Z AWST|

**例子**

```xml
<logger>
    <level>trace</level>
    <log>/var/log/clickhouse-server/clickhouse-server-%F-%T.log</log>
    <errorlog>/var/log/clickhouse-server/clickhouse-server-%F-%T.err.log</errorlog>
    <size>1000M</size>
    <count>10</count>
    <stream_compress>true</stream_compress>
</logger>
```

可以配置写入控制台。配置示例：

```xml
<logger>
    <level>information</level>
    <console>1</console>
</logger>
```

还支持写入系统日志。配置示例：

```xml
<logger>
    <use_syslog>1</use_syslog>
    <syslog>
        <address>syslog.remote:10514</address>
        <hostname>myhost.local</hostname>
        <facility>LOG_LOCAL6</facility>
        <format>syslog</format>
    </syslog>
</logger>
```

系统日志的键：

* use_syslog — 如果要写入系统日志，则需要设置。

* address — syslogd 的主机[:端口]。如果省略，则使用本地守护程序。

* hostname — 选修的。发送日志的主机的名称。

* facility — syslog 工具关键字采用大写字母并带有“LOG_”前缀：（`LOG_USER`、`LOG_DAEMON`、`LOG_LOCAL3` 等）。默认值：如果指定了`地址`，则为 `LOG_USER`，否则为 `LOG_DAEMON`。

* format – 消息格式。可能的值：`bsd` 和 `syslog`。

***

## send_crash_reports

用于选择通过 Sentry  向 ClickHouse 核心开发团队发送崩溃报告的设置。启用它，特别是在预生产环境中，受到高度赞赏。

服务器需要通过 IPv4 访问公共互联网（在撰写本文时 Sentry  不支持 IPv6），才能使此功能正常运行。

键：

* `enabled` – 用于启用该功能的布尔标志，默认为 `false`。设置为 `true` 以允许发送崩溃报告。

* `endpoint` – 您可以覆盖 Sentry 端点 URL 以发送崩溃报告。它可以是单独的 Sentry 帐户或您的自托管 Sentry 实例。使用 Sentry DSN 语法。

* `anonymize` - 避免将服务器主机名附加到崩溃报告中。

* `http_proxy` - 配置 HTTP 代理以发送崩溃报告。

* `debug` - 将 Sentry 客户端设置为调试模式。

* `tmp_path` - 临时崩溃报告状态的文件系统路径。

* `environment` - ClickHouse 服务器运行的环境的任意名称。每个崩溃报告中都会提到它。默认值为 `test` 或 `prod`，具体取决于 ClickHouse 的版本。

**推荐使用方式**

```xml
<send_crash_reports>
    <enabled>true</enabled>
</send_crash_reports>
```

## macros

复制表的参数替换。

如果不使用复制表，则可以省略。

有关更多信息，请参阅创建复制表部分。

**例子**

```xml
<macros incl="macros" optional="true" />
```

## replica_group_name

数据库 Replicated 的副本组名称。

复制数据库创建的集群将由同一组中的副本组成。 DDL查询只会等待同一组中的副本。

默认为空。

**例子**

```xml
<replica_group_name>backups</replica_group_name>
```

***

**默认值**：

## max_open_files

打开文件的最大数量。

**默认值**：`最大`

我们建议在 macOS 中使用此选项，因为 `getrlimit` 函数返回不正确的值。

**例子**

```xml
<max_open_files>262144</max_open_files>
```

***

## max_table_size_to_drop

删除表的限制。

如果 MergeTree 表的大小超过 `max_table_size_to_drop` （以字节为单位），则无法使用 DROP 查询或 TRUNCATE 查询删除它。

此设置不需要重新启动 Clickhouse 服务器即可应用。禁用限制的另一种方法是创建 `<clickhouse-path>/flags/force_drop_table` 文件。

**默认值**：`50 GB`

值 0 表示您可以删除所有表，没有任何限制。

**例子**

```xml
<max_table_size_to_drop>0</max_table_size_to_drop>
```

***

## max_partition_size_to_drop

限制删除分区。

如果 MergeTree 表的大小超过 `max_partition_size_to_drop` （以字节为单位），则无法使用 DROP PARTITION 查询删除分区。

此设置不需要重新启动 Clickhouse 服务器即可应用。禁用限制的另一种方法是创建 `<clickhouse-path>/flags/force_drop_table` 文件。

**默认值**：`50 GB`

值 0 表示您可以不受任何限制地删除分区。

> 此限制不限制 drop table 和 truncate table，请参阅 max_table_size_to_drop

**例子**

```xml
<max_partition_size_to_drop>0</max_partition_size_to_drop>
```

***

## max_thread_pool_size

ClickHouse 使用全局线程池中的线程来处理查询。如果没有空闲线程来处理查询，则会在池中创建一个新线程。 `max_thread_pool_size` 限制池中的最大线程数。

**可能的值**：

* 正整数。

**默认值**：`10000`

**例子**

```xml
<max_thread_pool_size>12000</max_thread_pool_size>
```

***

## max_thread_pool_free_size

如果全局线程池中的空闲线程数大于 `max_thread_pool_free_size`，则 ClickHouse 会释放部分线程占用的资源，并减小池大小。如果需要，可以再次创建线程。

**可能的值**： 

* 正整数。 

**默认值**：`1000` 

**例子**

```xml
<max_thread_pool_free_size>1200</max_thread_pool_free_size>
```

***

## thread_pool_queue_size

全局线程池上可以调度的最大作业数。增加队列大小会导致更大的内存使用量。建议将此值保持等于 `max_thread_pool_size`。

**可能的值**： 

* 正整数。 

* 0 — 无限制。 

**默认值**：`10000`

**例子**

```xml
<thread_pool_queue_size>12000</thread_pool_queue_size>
```

***

## max_io_thread_pool_size

ClickHouse 使用 IO 线程池中的线程来执行一些 IO 操作（例如与 S3 交互）。 `max_io_thread_pool_size` 限制池中的最大线程数。

**可能的值**： 

* 正整数。 

**默认值**：`100`

***

## max_io_thread_pool_free_size

如果IO线程池中的空闲线程数量超过`max_io_thread_pool_free_size`，ClickHouse将释放空闲线程占用的资源并减小池大小。如果需要，可以再次创建线程。

**可能的值**： 

* 正整数。 

**默认值**：`0`

***

## io_thread_pool_queue_size

IO线程池上可以调度的最大作业数。 

**可能的值**： 

* 正整数。 

* 0 — 无限制。 

**默认值**：`10000`

***

## max_backups_io_thread_pool_size

ClickHouse 使用备份 IO 线程池中的线程来执行 S3 备份 IO 操作。 `max_backups_io_thread_pool_size` 限制池中的最大线程数。

**可能的值**： 

* 正整数。 

**默认值**：`1000`

***

## max_backups_io_thread_pool_free_size

如果Backups IO Thread池中的空闲线程数量超过`max_backup_io_thread_pool_free_size`，ClickHouse将释放空闲线程占用的资源并减小池大小。如果需要，可以再次创建线程。

**可能的值**： 

* 正整数。 

* 零。 

**默认值**：`0`

***

## backups_io_thread_pool_queue_size

备份 IO 线程池上可以调度的最大作业数。由于当前的 S3 备份逻辑，建议保持此队列不受限制。

**可能的值**： 

* 正整数。 

* 0 — 无限制。 

**默认值**：`0`

***

## background_pool_size

设置使用 MergeTree 引擎对表执行后台合并和突变的线程数。此设置也可以在服务器启动时从默认配置文件配置应用，以便在 ClickHouse 服务器启动时向后兼容。您只能在运行时增加线程数。要减少线程数，您必须重新启动服务器。通过调整此设置，您可以管理 CPU 和磁盘负载。较小的池大小使用较少的 CPU 和磁盘资源，但后台进程进展较慢，最终可能会影响查询性能。

在更改之前，还请查看相关的 MergeTree 设置，例如 `number_of_free_entries_in_pool_to_lower_max_size_of_merge` 和 `number_of_free_entries_in_pool_to_execute_mutation`。

**可能的值**： 

* 任何正整数。 

**默认值**：`16` 

**例子**

```xml
<background_pool_size>16</background_pool_size>
```

***

## background_merges_mutations_concurrency_ratio

设置线程数与可以同时执行的后台合并和突变数之间的比率。例如，如果比率等于2并且`background_pool_size`设置为16，那么ClickHouse可以同时执行32个后台合并。这是可能的，因为后台操作可以​​暂停和推迟。这是为小型合并提供更多执行优先级所必需的。您只能在运行时增加此比率。要降低它，您必须重新启动服务器。与`background_pool_size`设置相同的`background_merges_mutations_concurrency_ratio`可以从默认配置文件中应用以实现向后兼容性。

**可能的值**： 

* 任何正整数。 

**默认值**：`2` 

**例子**

```xml
<background_merges_mutations_concurrency_ratio>3</background_merges_mutations_concurrency_ratio>
```

***

## merges_mutations_memory_usage_soft_limit

设置允许使用多少 RAM 来执行合并和变异操作。零意味着无限。如果 ClickHouse 达到此限制，它不会安排任何新的后台合并或突变操作，但会继续执行已安排的任务。

**可能的值**： 

* 任何正整数。 

**例子**

```xml
<merges_mutations_memory_usage_soft_limit>0</merges_mutations_memory_usage_soft_limit>
```

***

## merges_mutations_memory_usage_to_ram_ratio

默认的 `merges_mutations_memory_usage_soft_limit` 值的计算方式为：`memory_amount * merges_mutations_memory_usage_to_ram_ratio`。

**默认值**：`0.5` 

**也可以看看** 

* max_memory_usage

* merges_mutations_memory_usage_soft_limit

***

## merge_tree

对 MergeTree 中的表进行微调。

有关详细信息，请参阅 MergeTreeSettings.h 头文件。

**例子**

```xml
<merge_tree>
    <max_suspicious_broken_parts>5</max_suspicious_broken_parts>
</merge_tree>
```

***

## metric_log

默认情况下它是启用的。如果不是，您可以手动执行此操作。

**启用**

要手动打开指标历史记录收集 `system.metric_log`，请使用以下内容创建 `/etc/clickhouse-server/config.d/metric_log.xml`：

```xml
<clickhouse>
    <metric_log>
        <database>system</database>
        <table>metric_log</table>
        <flush_interval_milliseconds>7500</flush_interval_milliseconds>
        <collect_interval_milliseconds>1000</collect_interval_milliseconds>
        <max_size_rows>1048576</max_size_rows>
        <reserved_size_rows>8192</reserved_size_rows>
        <buffer_size_rows_flush_threshold>524288</buffer_size_rows_flush_threshold>
        <flush_on_crash>false</flush_on_crash>
    </metric_log>
</clickhouse>
```

**禁用**

要禁用 `metric_log` 设置，您应该创建以下文件 `/etc/clickhouse-server/config.d/disable_metric_log.xml`，其中包含以下内容：

```xml
<clickhouse>
<metric_log remove="1" />
</clickhouse>
```

***

## replicated_merge_tree

对 ReplicatedMergeTree 中的表进行微调。

该设置具有更高的优先级。

有关详细信息，请参阅 MergeTreeSettings.h 头文件。 

**例子**

```xml
<replicated_merge_tree>
    <max_suspicious_broken_parts>5</max_suspicious_broken_parts>
</replicated_merge_tree>
```

***

## openSSL

SSL 客户端/服务器配置。

`libpoco` 库提供对 SSL 的支持。 SSLManager.h 中解释了可用的配置选项。默认值可以在 SSLManager.cpp 中找到。

服务器/客户端设置键：

* privateKeyFile – 包含 PEM 证书密钥的文件的路径。该文件可能同时包含密钥和证书。

* certificateFile – PEM 格式的客户端/服务器证书文件的路径。如果 `privateKeyFile` 包含证书，则可以省略它。

* caConfig（默认值：无） – 包含受信任 CA 证书的文件或目录的路径。如果它指向一个文件，则它必须是 PEM 格式并且可以包含多个 CA 证书。如果它指向一个目录，则每个 CA 证书必须包含一个 .pem 文件。文件名通过 CA 主题名称哈希值查找。详细信息可以在 SSL_CTX_load_verify_locations 的手册页中找到。

* verifyMode （默认值：relaxed） – 检查节点证书的方法。详细信息在 Context 类的描述中。可能的值：`none`, `relaxed`, `strict`, `once`。

* verifyDepth（默认值：9）– 验证链的最大长度。如果证书链长度超过设定值，验证将失败。

* loadDefaultCAFile（默认值：true）– 是否使用 OpenSSL 的内置 CA 证书。 ClickHouse 假定内置 CA 证书位于文件 `/etc/ssl/cert.pem`（分别是目录 `/etc/ssl/certs`）中或环境变量 `SSL_CERT_FILE`（分别是 `SSL_CERT_DIR`）指定的文件（分别是目录）中。

* cipherList（默认值：`ALL:!ADH:!LOW:!EXP:!MD5:!3DES:@STRENGTH`） - 支持的 OpenSSL 加密。

* cacheSessions（默认值： false） – 启用或禁用缓存会话。必须与sessionIdContext结合使用。可接受的值：`true`、`false`。

* sessionIdContext（默认值：`${application.name}`）– 服务器附加到每个生成的标识符的一组唯一的随机字符。 字符串的长度不得超过 `SSL_MAX_SSL_SESSION_ID_LENGTH`。 始终建议使用此参数，因为它有助于避免服务器缓存会话和客户端请求缓存时出现的问题。 默认值：`${application.name}`。

* sessionCacheSize（默认值：1024*20） – 服务器缓存的最大会话数。值 0 表示无限制会话。

* sessionTimeout（默认值：2h） – 在服务器上缓存会话的时间。

* ExtendedVerification（默认值： false） – 如果启用，则验证证书 CN 或 SAN 是否与对等主机名匹配。

* requireTLSv1（默认值： false） – 需要 TLSv1 连接。可接受的值：`true`、`false`。

* requireTLSv1_1（默认值： false） – 需要 TLSv1.1 连接。可接受的值：`true`、`false`。

* requireTLSv1_2 (default: false) – Require a TLSv1.2 connection. Acceptable values: `true`, `false`.

* fips（默认值： false） – 激活 OpenSSL FIPS 模式。如果库的 OpenSSL 版本支持 FIPS，则受支持。

* privateKeyPassphraseHandler（默认值：`KeyConsoleHandler`）– 请求用于访问私钥的密码的类（PrivateKeyPassphraseHandler 子类）。 例如：`<privateKeyPassphraseHandler>`, `<name>KeyFileHandler</name>`, `<options><password>test</password></options>`, `</privateKeyPassphraseHandler>`。

* invalidCertificateHandler（默认值：`RejectCertificateHandler`） – 用于验证无效证书的类（CertificateHandler 的子类）。 例如： `<invalidCertificateHandler> <name>RejectCertificateHandler</name> </invalidCertificateHandler>` 。

* disableProtocols（默认值：“”） – 不允许使用的协议。

* preferServerCiphers（默认值： false） – 客户端上的首选服务器密码。

**设置示例**：

```xml
<openSSL>
    <server>
        <!-- openssl req -subj "/CN=localhost" -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout /etc/clickhouse-server/server.key -out /etc/clickhouse-server/server.crt -->
        <certificateFile>/etc/clickhouse-server/server.crt</certificateFile>
        <privateKeyFile>/etc/clickhouse-server/server.key</privateKeyFile>
        <!-- openssl dhparam -out /etc/clickhouse-server/dhparam.pem 4096 -->
        <dhParamsFile>/etc/clickhouse-server/dhparam.pem</dhParamsFile>
        <verificationMode>none</verificationMode>
        <loadDefaultCAFile>true</loadDefaultCAFile>
        <cacheSessions>true</cacheSessions>
        <disableProtocols>sslv2,sslv3</disableProtocols>
        <preferServerCiphers>true</preferServerCiphers>
    </server>
    <client>
        <loadDefaultCAFile>true</loadDefaultCAFile>
        <cacheSessions>true</cacheSessions>
        <disableProtocols>sslv2,sslv3</disableProtocols>
        <preferServerCiphers>true</preferServerCiphers>
        <!-- Use for self-signed: <verificationMode>none</verificationMode> -->
        <invalidCertificateHandler>
            <!-- Use for self-signed: <name>AcceptCertificateHandler</name> -->
            <name>RejectCertificateHandler</name>
        </invalidCertificateHandler>
    </client>
</openSSL>
```

***

## part_log

记录与 MergeTree 关联的事件。例如，添加或合并数据。您可以使用日志来模拟合并算法并比较它们的特性。您可以可视化合并过程。

查询记录在 system.part_log 表中，而不是单独的文件中。您可以在`表`参数中配置该表的名称（见下文）。

使用以下参数来配置日志记录：

* `database` – 数据库的名称。

* `table` – 系统表的名称。

* `partition_by` – 系统表的自定义分区键。如果引擎已定义则无法使用。

* `order_by` – 系统表的自定义排序键。如果引擎已定义则无法使用。

* `engine` – 系统表的 MergeTree 引擎定义。如果定义了`partition_by`或`order_by`则不能使用。

* `flush_interval_milliseconds` – 将数据从内存缓冲区刷新到表的时间间隔。

* `max_size_rows` – 日志的最大大小（以行为单位）。当未刷新的日志量达到max_size时，日志转储到磁盘。默认值：`1048576`。

* `reserved_size_rows` – 为日志预先分配的内存大小（以行为单位）。默认值：`8192`。

* `buffer_size_rows_flush_threshold` – 行数阈值，达到该阈值会在后台将日志刷新到磁盘。默认值：`max_size_rows / 2`。

* `flush_on_crash` – 指示在发生崩溃时是否应将日志转储到磁盘。默认值：`false`。

* `storage_policy` – 用于表的存储策略名称（可选）

* `settings` – 控制 MergeTree 行为的附加参数（可选）。

**例子**

```xml
<part_log>
    <database>system</database>
    <table>part_log</table>
    <partition_by>toMonday(event_date)</partition_by>
    <flush_interval_milliseconds>7500</flush_interval_milliseconds>
    <max_size_rows>1048576</max_size_rows>
    <reserved_size_rows>8192</reserved_size_rows>
    <buffer_size_rows_flush_threshold>524288</buffer_size_rows_flush_threshold>
    <flush_on_crash>false</flush_on_crash>
</part_log>
```

***

## path

包含数据的目录的路径。

> 尾部斜杠是强制性的。

**例子**

```xml
<path>/var/lib/clickhouse/</path>
```

***

## Prometheus

> ClickHouse Cloud 目前不支持连接 Prometheus。要在支持此功能时收到通知，请联系 support@clickhouse.com。

公开指标数据以从 Prometheus 中抓取。

设置：

* `endpoint` – 用于通过 prometheus 服务器抓取指标的 HTTP 端点。从...开始 '/'。

* `port` – 端点的端口。

* `metrics` – 公开 system.metrics 表中的指标。

* `events` – 公开 system.events 表中的指标。

* `asynchronous_metrics ` – 从 system.asynchronous_metrics 表中公开当前指标值。

* `errors` – 通过错误代码显示自上次服务器重新启动以来发生的错误数。该信息也可以从 system.errors 中获取。

**例子**

```xml
<clickhouse>
    <listen_host>0.0.0.0</listen_host>
    <http_port>8123</http_port>
    <tcp_port>9000</tcp_port>
    <prometheus>
        <endpoint>/metrics</endpoint>
        <port>9363</port>
        <metrics>true</metrics>
        <events>true</events>
        <asynchronous_metrics>true</asynchronous_metrics>
        <errors>true</errors>
    </prometheus>
</clickhouse>
```

检查（将 `127.0.0.1` 替换为 ClickHouse 服务器的 IP 地址或主机名）：

```bash
curl 127.0.0.1:9363/metrics
```

***

## query_log

用于记录通过 log_queries=1 设置接收到的查询的设置。

查询记录在 system.query_log 表中，而不是单独的文件中。您可以在表参数中更改表的名称（见下文）。

使用以下参数来配置日志记录：

* `database` – 数据库的名称。

* `table` – 查询将登录的系统表的名称。

* `partition_by` – 系统表的自定义分区键。如果引擎已定义则无法使用。

* `order_by` – 系统表的自定义排序键。如果引擎已定义则无法使用。

* `engine` – 系统表的 MergeTree 引擎定义。如果定义了partition_by或order_by则不能使用。

* `flush_interval_milliseconds` – 将数据从内存缓冲区刷新到表的时间间隔。

* `max_size_rows` – 日志的最大大小（以行为单位）。当未刷新的日志量达到max_size时，日志转储到磁盘。默认值：`1048576`。

* `reserved_size_rows` – 为日志预先分配的内存大小（以行为单位）。默认值：`8192`。

* `buffer_size_rows_flush_threshold` – 行数阈值，达到该阈值会在后台将日志刷新到磁盘。默认值：`max_size_rows / 2`。

* `flush_on_crash` – 指示在发生崩溃时是否应将日志转储到磁盘。默认值：`false`。

* `storage_policy` – 用于表的存储策略名称（可选）

* `settings` – 控制 MergeTree 行为的附加参数（可选）。

如果该表不存在，ClickHouse 将创建它。如果ClickHouse服务器更新时查询日志的结构发生变化，旧结构的表将被重命名，并自动创建新表。

**例子**

```xml
<query_log>
    <database>system</database>
    <table>query_log</table>
    <engine>Engine = MergeTree PARTITION BY event_date ORDER BY event_time TTL event_date + INTERVAL 30 day</engine>
    <flush_interval_milliseconds>7500</flush_interval_milliseconds>
    <max_size_rows>1048576</max_size_rows>
    <reserved_size_rows>8192</reserved_size_rows>
    <buffer_size_rows_flush_threshold>524288</buffer_size_rows_flush_threshold>
    <flush_on_crash>false</flush_on_crash>
</query_log>
```

***

## query_cache

查询缓存配置。

可以使用以下设置：

* `max_size_in_bytes`: 最大缓存大小（以字节为单位）。 0 表示查询缓存已禁用。默认值：`1073741824` (1 GiB)。

* `max_entries`: 缓存中存储的 `SELECT` 查询结果的最大数量。默认值：`1024`。

* `max_entry_size_in_bytes`: `SELECT` 查询结果可能必须保存在缓存中的最大大小（以字节为单位）。默认值：`1048576` (1 MiB)。

* `max_entry_size_in_rows`: `SELECT` 查询结果可能必须保存在缓存中的最大行数。默认值：`30000000`（3000 万）。

更改的设置立即生效。

> 查询缓存的数据分配在 DRAM 中。如果内存不足，请确保为 `max_size_in_bytes` 设置一个较小的值或完全禁用查询缓存。

**例子**

```xml
<query_cache>
    <max_size_in_bytes>1073741824</max_size_in_bytes>
    <max_entries>1024</max_entries>
    <max_entry_size_in_bytes>1048576</max_entry_size_in_bytes>
    <max_entry_size_in_rows>30000000</max_entry_size_in_rows>
</query_cache>
```

***

## query_thread_log

用于记录通过 `log_query_threads=1` 设置接收的查询线程的设置。

查询记录在 `system.query_thread_log` 表中，而不是单独的文件中。您可以在表参数中更改表的名称（见下文）。

使用以下参数来配置日志记录：

* `database` – 数据库的名称。

* `table` – 查询将登录的系统表的名称。

* `partition_by` – 系统表的自定义分区键。如果引擎已定义则无法使用。

* `order_by` – 系统表的自定义排序键。如果引擎已定义则无法使用。

* `engine` – 系统表的 MergeTree 引擎定义。如果定义了`partition_by`或`order_by`则不能使用。

* `flush_interval_milliseconds` – 将数据从内存缓冲区刷新到表的时间间隔。

* `max_size_rows` – 日志的最大大小（以行为单位）。当未刷新的日志数量达到max_size_rows时，日志转储到磁盘。默认值：`1048576`。

* `reserved_size_rows` – 为日志预先分配的内存大小（以行为单位）。默认值：`8192`。

* `buffer_size_rows_flush_threshold` – 行数阈值，达到该阈值会在后台将日志刷新到磁盘。默认值：`max_size_rows / 2`。

* `flush_on_crash` – 指示在发生崩溃时是否应将日志转储到磁盘。默认值：`false`。

* `storage_policy` – 用于表的存储策略名称（可选）

* `settings` – 控制 MergeTree 行为的附加参数（可选）。

如果该表不存在，ClickHouse 将创建它。如果ClickHouse服务器更新时查询线程日志的结构发生变化，旧结构的表将被重命名，并自动创建新表。

**例子**

```xml
<query_thread_log>
    <database>system</database>
    <table>query_thread_log</table>
    <partition_by>toMonday(event_date)</partition_by>
    <flush_interval_milliseconds>7500</flush_interval_milliseconds>
    <max_size_rows>1048576</max_size_rows>
    <reserved_size_rows>8192</reserved_size_rows>
    <buffer_size_rows_flush_threshold>524288</buffer_size_rows_flush_threshold>
    <flush_on_crash>false</flush_on_crash>
</query_thread_log>
```

***

## query_views_log

记录视图（实时、具体化等）的设置取决于使用 `log_query_views=1` 设置接收到的查询。

查询记录在 system.query_views_log 表中，而不是单独的文件中。您可以在表参数中更改表的名称（见下文）。

使用以下参数来配置日志记录：

* `database` – 数据库的名称。

* `table` – 查询将登录的系统表的名称。

* `partition_by` – 系统表的自定义分区键。如果引擎已定义则无法使用。

* `order_by` – 系统表的自定义排序键。如果引擎已定义则无法使用。

* `engine` – 系统表的 MergeTree 引擎定义。如果定义了`partition_by`或`order_by`则不能使用。

* `flush_interval_milliseconds` – 将数据从内存缓冲区刷新到表的时间间隔。

* `max_size_rows` – 日志的最大大小（以行为单位）。当未刷新的日志量达到max_size时，日志转储到磁盘。默认值：`1048576`。

* `reserved_size_rows` – 为日志预先分配的内存大小（以行为单位）。默认值：`8192`。

* `buffer_size_rows_flush_threshold` – 行数阈值，达到该阈值会在后台将日志刷新到磁盘。默认值：`max_size_rows / 2`。

* `flush_on_crash` – 指示在发生崩溃时是否应将日志转储到磁盘。默认值：`false`。

* `storage_policy` – 用于表的存储策略名称（可选）

* `settings` – 控制 MergeTree 行为的附加参数（可选）。

如果该表不存在，ClickHouse 将创建它。如果更新 ClickHouse 服务器时查询视图日志的结构发生变化，则旧结构的表将被重命名，并自动创建新表。

**例子**

```xml
<query_views_log>
    <database>system</database>
    <table>query_views_log</table>
    <partition_by>toYYYYMM(event_date)</partition_by>
    <flush_interval_milliseconds>7500</flush_interval_milliseconds>
    <max_size_rows>1048576</max_size_rows>
    <reserved_size_rows>8192</reserved_size_rows>
    <buffer_size_rows_flush_threshold>524288</buffer_size_rows_flush_threshold>
    <flush_on_crash>false</flush_on_crash>
</query_views_log>
```

***

## text_log

用于记录文本消息的text_log 系统表的设置。

参数：

* `level` – 将存储在表中的最大消息级别（默认为`Trace`）。

* `database` – 数据库名称。

* `table` – 表名。

* `partition_by` – 系统表的自定义分区键。如果引擎已定义则无法使用。

* `order_by` – 系统表的自定义排序键。如果引擎已定义则无法使用。

* `engine` – 系统表的 MergeTree 引擎定义。如果定义了`partition_by`或`order_by`则不能使用。

* `flush_interval_milliseconds` – 将数据从内存缓冲区刷新到表的时间间隔。

* `max_size_rows` – 日志的最大大小（以行为单位）。当未刷新的日志量达到max_size时，日志转储到磁盘。默认值：`1048576`。

* `reserved_size_rows` – 为日志预先分配的内存大小（以行为单位）。默认值：`8192`。

* `buffer_size_rows_flush_threshold` – 行数阈值，达到该阈值会在后台将日志刷新到磁盘。默认值：`max_size_rows / 2`。

* `flush_on_crash` – 指示在发生崩溃时是否应将日志转储到磁盘。默认值：`false`。

* `storage_policy` – 用于表的存储策略名称（可选）

* `settings` – 控制 MergeTree 行为的附加参数（可选）。

**例子**

```xml
<clickhouse>
    <text_log>
        <level>notice</level>
        <database>system</database>
        <table>text_log</table>
        <flush_interval_milliseconds>7500</flush_interval_milliseconds>
        <max_size_rows>1048576</max_size_rows>
        <reserved_size_rows>8192</reserved_size_rows>
        <buffer_size_rows_flush_threshold>524288</buffer_size_rows_flush_threshold>
        <flush_on_crash>false</flush_on_crash>
        <!-- <partition_by>event_date</partition_by> -->
        <engine>Engine = MergeTree PARTITION BY event_date ORDER BY event_time TTL event_date + INTERVAL 30 day</engine>
    </text_log>
</clickhouse>
```

***

## trace_log

Trace_log系统表操作的设置。

参数：

* `database` – 用于存储表的数据库。

* `table` – 表名。

* `partition_by` – 系统表的自定义分区键。如果引擎已定义则无法使用。

* `order_by` – 系统表的自定义排序键。如果引擎已定义则无法使用。

* `engine` – 系统表的 MergeTree 引擎定义。如果定义了`partition_by`或`order_by`则不能使用。

* `flush_interval_milliseconds` – 将数据从内存缓冲区刷新到表的时间间隔。

* `max_size_rows` – 日志的最大大小（以行为单位）。当未刷新的日志量达到max_size时，日志转储到磁盘。默认值：`1048576`。

* `reserved_size_rows` – 为日志预先分配的内存大小（以行为单位）。默认值：`8192`。

* `buffer_size_rows_flush_threshold` – 行数阈值，达到该阈值会在后台将日志刷新到磁盘。默认值：`max_size_rows / 2`。

* `storage_policy` – 用于表的存储策略名称（可选）

* `settings` – 控制 MergeTree 行为的附加参数（可选）。

默认服务器配置文件 config.xml 包含以下设置部分：

```xml
<trace_log>
    <database>system</database>
    <table>trace_log</table>
    <partition_by>toYYYYMM(event_date)</partition_by>
    <flush_interval_milliseconds>7500</flush_interval_milliseconds>
    <max_size_rows>1048576</max_size_rows>
    <reserved_size_rows>8192</reserved_size_rows>
    <buffer_size_rows_flush_threshold>524288</buffer_size_rows_flush_threshold>
    <flush_on_crash>false</flush_on_crash>
</trace_log>
```

***

## asynchronous_insert_log

用于记录异步插入的 asynchronous_insert_log 系统表的设置。

参数：

* `database` – 数据库名称。

* `table` – 表名。

* `partition_by` – 系统表的自定义分区键。如果引擎已定义则无法使用。

* `engine` – 系统表的 MergeTree 引擎定义。如果定义了`partition_by`则不能使用。

* `flush_interval_milliseconds` – 将数据从内存缓冲区刷新到表的时间间隔。

* `max_size_rows` – 日志的最大大小（以行为单位）。当未刷新的日志量达到max_size时，日志转储到磁盘。默认值：`1048576`。

* `reserved_size_rows` – 为日志预先分配的内存大小（以行为单位）。默认值：`8192`。

* `buffer_size_rows_flush_threshold` – 行数阈值，达到该阈值会在后台将日志刷新到磁盘。默认值：`max_size_rows / 2`。

* `flush_on_crash` – 指示在发生崩溃时是否应将日志转储到磁盘。默认值：`false`。

* `storage_policy` – 用于表的存储策略名称（可选）

**例子**

```xml
<clickhouse>
    <asynchronous_insert_log>
        <database>system</database>
        <table>asynchronous_insert_log</table>
        <flush_interval_milliseconds>7500</flush_interval_milliseconds>
        <partition_by>toYYYYMM(event_date)</partition_by>
        <max_size_rows>1048576</max_size_rows>
        <reserved_size_rows>8192</reserved_size_rows>
        <buffer_size_rows_flush_threshold>524288</buffer_size_rows_flush_threshold>
        <flush_on_crash>false</flush_on_crash>
        <!-- <engine>Engine = MergeTree PARTITION BY event_date ORDER BY event_time TTL event_date + INTERVAL 30 day</engine> -->
    </asynchronous_insert_log>
</clickhouse>
```

***

## crash_log

crash_log系统表操作的设置。

参数：

* `database` – 用于存储表的数据库。

* `table` – 表名。

* `partition_by` – 系统表的自定义分区键。如果引擎已定义则无法使用。

* `order_by` – 系统表的自定义排序键。如果引擎已定义则无法使用。

* `engine` – 系统表的 MergeTree 引擎定义。如果定义了`partition_by`或`order_by`则不能使用。

* `flush_interval_milliseconds` – 将数据从内存缓冲区刷新到表的时间间隔。

* `max_size_rows` – 日志的最大大小（以行为单位）。当未刷新的日志量达到max_size时，日志转储到磁盘。默认值：`1048576`。

* `reserved_size_rows` – 为日志预先分配的内存大小（以行为单位）。默认值：`8192`。

* `buffer_size_rows_flush_threshold` – 行数阈值，达到该阈值会在后台将日志刷新到磁盘。默认值：`max_size_rows / 2`。

* `flush_on_crash` – 指示在发生崩溃时是否应将日志转储到磁盘。默认值：`false`。

* `storage_policy` – 用于表的存储策略名称（可选）

* `settings` – 控制 MergeTree 行为的附加参数（可选）。

默认服务器配置文件 `config.xml` 包含以下设置部分：

```xml
<crash_log>
    <database>system</database>
    <table>crash_log</table>
    <partition_by>toYYYYMM(event_date)</partition_by>
    <flush_interval_milliseconds>7500</flush_interval_milliseconds>
    <max_size_rows>1024</max_size_rows>
    <reserved_size_rows>1024</reserved_size_rows>
    <buffer_size_rows_flush_threshold>512</buffer_size_rows_flush_threshold>
    <flush_on_crash>false</flush_on_crash>
</crash_log>
```

***

## backup_log

用于记录 `BACKUP` 和 `RESTORE` 操作的 backup_log 系统表的设置。

参数：

* `database` – 数据库名称。

* `table` – 表名。

* `partition_by` – 系统表的自定义分区键。如果定义了引擎则无法使用。

* `order_by` – 系统表的自定义排序键。如果定义了引擎则无法使用。

* `engine` – 系统表的 MergeTree 引擎定义。如果定义了`partition_by`或`order_by`则不能使用。

* `flush_interval_milliseconds` – 将数据从内存缓冲区刷新到表的时间间隔。

* `max_size_rows` – 日志的最大大小（以行为单位）。当未刷新的日志量达到max_size时，日志转储到磁盘。默认值：`1048576`。

* `reserved_size_rows` – 为日志预先分配的内存大小（以行为单位）。默认值：`8192`。

* `buffer_size_rows_flush_threshold` – 行数阈值，达到该阈值会在后台将日志刷新到磁盘。默认值：`max_size_rows / 2`。

* `flush_on_crash` – 指示在发生崩溃时是否应将日志转储到磁盘。默认值：`false`。

* `storage_policy` – 用于表的存储策略的名称（可选）。

* `settings` – 控制 MergeTree 行为的附加参数（可选）。

**例子**

```xml
<clickhouse>
    <backup_log>
        <database>system</database>
        <table>backup_log</table>
        <flush_interval_milliseconds>1000</flush_interval_milliseconds>
        <partition_by>toYYYYMM(event_date)</partition_by>
        <max_size_rows>1048576</max_size_rows>
        <reserved_size_rows>8192</reserved_size_rows>
        <buffer_size_rows_flush_threshold>524288</buffer_size_rows_flush_threshold>
        <flush_on_crash>false</flush_on_crash>
        <!-- <engine>Engine = MergeTree PARTITION BY event_date ORDER BY event_time TTL event_date + INTERVAL 30 day</engine> -->
    </backup_log>
</clickhouse>
```

## query_masking_rules

基于正则表达式的规则，将应用于查询以及所有日志消息，然后将它们存储在服务器日志、`system.query_log`、`system.text_log`、`system.processes` 表以及发送到客户端的日志中。这样可以防止敏感数据从 SQL 查询（如姓名、电子邮件、个人标识符或信用卡号）泄漏到日志中。

**例子**

```xml
<query_masking_rules>
    <rule>
        <name>hide SSN</name>
        <regexp>(^|\D)\d{3}-\d{2}-\d{4}($|\D)</regexp>
        <replace>000-00-0000</replace>
    </rule>
</query_masking_rules>
```

配置字段：

* `name` - 规则的名称（可选）
* `regexp` - RE2 兼容正则表达式（强制）
* `replace` - 敏感数据的替换字符串（可选，默认情况下 - 六个星号）

屏蔽规则应用于整个查询（以防止由于格式错误/不可解析的查询而泄漏敏感数据）。

`system.events` 表有计数器 `QueryMaskingRulesMatch`，它具有查询屏蔽规则匹配的总数。

对于分布式查询，每个服务器必须单独配置，否则，传递到其他节点的子查询将不加掩码地存储。

***

## remote_servers

分布式表引擎和簇表功能使用的簇的配置。

**例子**

```xml
<remote_servers incl="clickhouse_remote_servers" />
```

有关 `incl` 属性的值，请参阅“配置文件”部分。

***

## timezone

服务器的时区。

指定为 UTC 时区或地理位置（例如，非洲/阿比让）的 IANA 标识符。

当 DateTime 字段输出为文本格式（打印在屏幕上或文件中）以及从字符串获取 DateTime 时，时区对于 String 和 DateTime 格式之间的转换是必需的。此外，如果在输入参数中没有收到时区，则在处理时间和日期的函数中会使用时区。

**例子**

```xml
<timezone>Asia/Istanbul</timezone>
```

***

## tcp_port

通过 TCP 协议与客户端通信的端口。

**例子**

```xml
<tcp_port>9000</tcp_port>
```

***

## tcp_port_secure

用于与客户端安全通信的 TCP 端口。将其与 OpenSSL 设置一起使用。

**可能的值** 

* 正整数。

**默认值**

```xml
<tcp_port_secure>9440</tcp_port_secure>
```

***

## mysql_port

通过 MySQL 协议与客户端通信的端口。

**可能的值** 

* 正整数指定要侦听的端口号或空值以禁用。

**例子**

```xml
<mysql_port>9004</mysql_port>
```

***

## postgresql_port

用于通过 PostgreSQL 协议与客户端通信的端口。

**可能的值** 

* 正整数指定要侦听的端口号或空值以禁用。 

**例子**

```xml
<postgresql_port>9005</postgresql_port>
```

***

## tmp_path

本地文件系统上用于存储处理大型查询的临时数据的路径。

> * 只能使用一个选项来配置临时数据存储：`tmp_path`、`tmp_policy`、`temporary_data_in_cache`。
> * 尾部斜杠是强制性的。

**例子**

```xml
<tmp_path>/var/lib/clickhouse/tmp/</tmp_path>
```

***

## user_files_path

包含用户文件的目录。用于表函数file()、fileCluster()。

**例子**

```xml
<user_files_path>/var/lib/clickhouse/user_files/</user_files_path>
```

***

## user_scripts_path

包含用户脚本文件的目录。用于可执行用户定义函数 可执行用户定义函数。

**例子**

```xml
<user_scripts_path>/var/lib/clickhouse/user_scripts/</user_scripts_path>
```

***

## user_defined_path

包含用户定义文件的目录。用于 SQL 用户定义函数 SQL 用户定义函数。

**例子**

```xml
<user_defined_path>/var/lib/clickhouse/user_defined/</user_defined_path>
```

***

## users_config

包含以下内容的文件的路径： 

* 用户配置。 

* 访问权。 

* 设置配置文件。 

* 配额设置。

**例子**

```xml
<users_config>users.xml</users_config>
```

***

## wait_dictionaries_load_at_startup

此设置允许指定 `dictionaries_lazy_load` 为 `false` 时的行为。 （如果 `dictionaries_lazy_load` 为 `true`，则此设置不会影响任何内容。）

如果 `wait_dictionaries_load_at_startup` 为 `false`，则服务器将在启动时开始加载所有词典，并且在加载的同时它将接收连接。当第一次在查询中使用字典时，如果尚未加载字典，则查询将等待字典加载。将 `wait_dictionaries_load_at_startup` 设置为 `false` 可以使 ClickHouse 启动得更快，但是某些查询可能会执行得更慢（因为它们必须等待某些字典加载）。

如果 `wait_dictionaries_load_at_startup` 为 `true`，则服务器将在启动时等待，直到所有词典完成加载（成功或失败），然后再接收任何连接。

默认为 `true`。

**例子**

```xml
<wait_dictionaries_load_at_startup>true</wait_dictionaries_load_at_startup>
```

***

## zookeeper

包含允许 ClickHouse 与 ZooKeeper 集群交互的设置。

当使用复制表时，ClickHouse 使用 ZooKeeper 来存储副本的元数据。如果不使用复制表，这部分参数可以省略。

该部分包含以下参数：

* `node` — ZooKeeper 端点。您可以设置多个端点。

例如：

```xml
<node index="1">
    <host>example_host</host>
    <port>2181</port>
</node>
```

```
The `index` attribute specifies the node order when trying to connect to the ZooKeeper cluster.
```

* `session_timeout_ms` — 客户端会话的最大超时（以毫秒为单位）。

* `operation_timeout_ms` — 一项操作的最大超时（以毫秒为单位）。

* `root` — 用作 ClickHouse 服务器使用的 znode 的根的 znode。选修的。

* `fallback_session_lifetime.min` - 如果通过zookeeper_load_balancing策略解析的第一个zookeeper主机不可用，则将zookeeper会话的生命周期限制到后备节点。这样做是为了负载平衡的目的，以避免其中一台 Zookeeper 主机负载过重。此设置设置回退会话的最短持续时间。以秒为单位设置。选修的。默认为 3 小时。

* `fallback_session_lifetime.max` - 如果通过zookeeper_load_balancing策略解析的第一个zookeeper主机不可用，则将zookeeper会话的生命周期限制到后备节点。这样做是为了负载平衡的目的，以避免其中一台 Zookeeper 主机负载过重。此设置设置回退会话的最大持续时间。以秒为单位设置。选修的。默认值为 6 小时。

* `identity` — ZooKeeper 可能需要用户和密码来授予对所请求的 znode 的访问权限。选修的。

* `zookeeper_load_balancing` - 指定ZooKeeper节点选择的算法

    * random - 随机选择一个 ZooKeeper 节点。

    * in_order - 选择第一个 ZooKeeper 节点，如果不可用则选择第二个，依此类推。

    * nearest_hostname - 选择主机名与服务器主机名最相似的 ZooKeeper 节点，将主机名与名称前缀进行比较。

    * hostname_levenshtein_distance - 就像nearest_hostname一样，但它以编辑距离的方式比较主机名。

    * first_or_random - 选择第一个 ZooKeeper 节点，如果不可用，则随机选择剩余的 ZooKeeper 节点之一。

    * round_robin - 选择第一个 ZooKeeper 节点，如果发生重新连接，则选择下一个。

* `use_compression` — 如果设置为 true，则启用 Keeper 协议中的压缩。

**配置示例**

```xml
<zookeeper>
    <node>
        <host>example1</host>
        <port>2181</port>
    </node>
    <node>
        <host>example2</host>
        <port>2181</port>
    </node>
    <session_timeout_ms>30000</session_timeout_ms>
    <operation_timeout_ms>10000</operation_timeout_ms>
    <!-- Optional. Chroot suffix. Should exist. -->
    <root>/path/to/zookeeper/node</root>
    <!-- Optional. Zookeeper digest ACL string. -->
    <identity>user:password</identity>
    <!--<zookeeper_load_balancing>random / in_order / nearest_hostname / hostname_levenshtein_distance / first_or_random / round_robin</zookeeper_load_balancing>-->
    <zookeeper_load_balancing>random</zookeeper_load_balancing>
</zookeeper>
```

***

## use_minimalistic_part_header_in_zookeeper

ZooKeeper中数据部分头的存储方法。

此设置仅适用于 `MergeTree` 系列。可以指定：

* 全局位于 config.xml 文件的 merge_tree 部分。

    ClickHouse 使用服务器上所有表的设置。您可以随时更改设置。当设置更改时，现有表会更改其行为。

* 对于每张表。

    创建表时，指定相应的引擎设置。即使全局设置发生更改，具有此设置的现有表的行为也不会更改。

**可能的值**

* 0 — 功能已关闭。 

* 1 — 功能已打开。

如果 `use_minimalistic_part_header_in_zookeeper = 1`，则复制表使用单个 `znode` 紧凑地存储数据部分的标头。如果表包含很多列，这种存储方式会显着减少Zookeeper中存储的数据量。

> 应用 `use_minimalistic_part_header_in_zookeeper = 1` 后，您无法将 ClickHouse 服务器降级到不支持此设置的版本。 在集群中的服务器上升级 ClickHouse 时要小心。 不要一次升级所有服务器。 在测试环境中或仅在集群的几台服务器上测试 ClickHouse 的新版本会更安全。
> 
> 已使用此设置存储的数据部分标头无法恢复为其之前的（非紧凑）表示形式。

**默认值**：`0`

***

## distributed_ddl

管理在集群上执行分布式 ddl 查询（CREATE、DROP、ALTER、RENAME）。仅当启用 ZooKeeper 时才有效。

`<distributed_ddl>` 中的可配置设置包括：

* `path`: 用于 DDL 查询的 task_queue 在 Keeper 中的路径

* `profile`: 用于执行 DDL 查询的配置文件

* `pool_size`: 可以同时运行多少个 `ON CLUSTER` 查询

* `max_tasks_in_queue`: 队列中可以容纳的最大任务数。默认值为 1,000

* `task_max_lifetime`: 如果节点的年龄大于此值，则删除节点。默认为 `7 * 24 * 60 * 60`（一周以秒为单位）

* `cleanup_delay_period`: 如果最后一次清理没有早于 `cleanup_delay_period` 秒前进行，则在收到新节点事件后开始清理。默认值为 60 秒

**例子**

```xml
<distributed_ddl>
    <!-- Path in ZooKeeper to queue with DDL queries -->
    <path>/clickhouse/task_queue/ddl</path>

    <!-- Settings from this profile will be used to execute DDL queries -->
    <profile>default</profile>

    <!-- Controls how much ON CLUSTER queries can be run simultaneously. -->
    <pool_size>1</pool_size>

    <!--
         Cleanup settings (active tasks will not be removed)
    -->

    <!-- Controls task TTL (default 1 week) -->
    <task_max_lifetime>604800</task_max_lifetime>

    <!-- Controls how often cleanup should be performed (in seconds) -->
    <cleanup_delay_period>60</cleanup_delay_period>

    <!-- Controls how many tasks could be in the queue -->
    <max_tasks_in_queue>1000</max_tasks_in_queue>
</distributed_ddl>
```

***

## access_control_path

ClickHouse 服务器存储由 SQL 命令创建的用户和角色配置的文件夹的路径。

**默认值**：`/var/lib/clickhouse/access/`

***

## user_directories

包含设置的配置文件部分：

* 具有预定义用户的配置文件的路径。 

* 存储由 SQL 命令创建的用户的文件夹路径。 

* ZooKeeper 节点路径，其中存储和复制由 SQL 命令创建的用户（实验）。

如果指定此部分，则不会使用 users_config 和 access_control_path 中的路径。

`user_directories` 部分可以包含任意数量的项目，项目的顺序意味着它们的优先级（项目越高优先级越高）。

**例子**

```xml
<user_directories>
    <users_xml>
        <path>/etc/clickhouse-server/users.xml</path>
    </users_xml>
    <local_directory>
        <path>/var/lib/clickhouse/access/</path>
    </local_directory>
</user_directories>
```

用户、角色、行策略、配额和配置文件也可以存储在 ZooKeeper 中：

```xml
<user_directories>
    <users_xml>
        <path>/etc/clickhouse-server/users.xml</path>
    </users_xml>
    <replicated>
        <zookeeper_path>/clickhouse/access/</zookeeper_path>
    </replicated>
</user_directories>
```

您还可以定义部分`内存` - 表示仅将信息存储在内存中，而不写入磁盘，而 `ldap` - 表示将信息存储在 LDAP 服务器上。

要将 LDAP 服务器添加为本地未定义的用户的远程用户目录，请使用以下参数定义单个 `ldap` 部分：

* `server` — ldap_servers 配置部分中定义的 LDAP 服务器名称之一。该参数为必填项，不能为空。

* `roles` — 部分包含本地定义的角色列表，这些角色将分配给从 LDAP 服务器检索到的每个用户。如果未指定角色，用户在身份验证后将无法执行任何操作。如果在身份验证时未在本地定义任何列出的角色，则身份验证尝试将失败，就像提供的密码不正确一样。

**例子**

```xml
<ldap>
    <server>my_ldap_server</server>
        <roles>
            <my_local_role1 />
            <my_local_role2 />
        </roles>
</ldap>
```

***

## total_memory_profiler_step

设置每个峰值分配步骤的堆栈跟踪的内存大小（以字节为单位）。数据存储在`system.trace_log`系统表中，`query_id`等于空字符串。

**可能的值**：

* 正整数。

**默认值**：`4194304`

***

## total_memory_tracker_sample_probability

允许收集随机分配和释放，并将它们写入 `system.trace_log` 系统表中，`trace_type` 等于具有指定概率的 `MemorySample`。概率针对每次分配或取消分配，无论分配的大小如何。请注意，仅当未跟踪内存量超过未跟踪内存限制（默认值为 `4` MiB）时才会进行采样。如果降低`total_memory_profiler_step`，则可以降低它。您可以将`total_memory_profiler_step`设置为等于`1`以进行更细粒度的采样。

**可能的值**：

* 正整数。

* 0 — 禁止在 system.trace_log 系统表中写入随机分配和释放。

**默认值**：`0`

***

compiled_expression_cache_size

设置编译表达式的缓存大小（以元素为单位）。

**可能的值**：

* 正整数。

**默认值**：`10000`

***

## display_secrets_in_show_and_select

启用或禁用在表、数据库、表函数和字典的 `SHOW` 和 `SELECT` 查询中显示机密。

希望查看机密的用户还必须打开 `format_display_secrets_in_show_and_select` 格式设置并具有 `displaySecretsInShowAndSelect` 权限。

**可能的值**： 

* 0 — 禁用。 

* 1 — 启用。 

**默认值**：`0`

***

## 代理

为 HTTP 和 HTTPS 请求定义代理服务器，目前由 S3 存储、S3 表函数和 URL 函数支持。

定义代理服务器有三种方法：环境变量、代理列表和远程代理解析器。

### 环境变量

`http_proxy` 和 `https_proxy` 环境变量允许您为给定协议指定代理服务器。如果您在系统上设置了它，它应该可以无缝运行。

如果给定协议只有一个代理服务器并且该代理服务器不会更改，则这是最简单的方法。

### 代理列表

这种方法允许您为协议指定一个或多个代理服务器。如果定义了多个代理服务器，ClickHouse 会循环使用不同的代理，平衡服务器之间的负载。如果某个协议有多个代理服务器并且代理服务器列表不会更改，则这是最简单的方法。

### 配置模板

```xml
<proxy>
    <http>
        <uri>http://proxy1</uri>
        <uri>http://proxy2:3128</uri>
    </http>
    <https>
        <uri>http://proxy1:3128</uri>
    </https>
</proxy>
```

`<proxy>`字段

* `<http>` — 一个或多个 HTTP 代理的列表

* `<https>` — 一个或多个 HTTPS 代理的列表

`<http>` 和 `<https>` 字段

* `<uri>` - 代理的 URI

### 远程代理解析器

代理服务器可能会动态更改。在这种情况下，您可以定义解析器的端点。ClickHouse 向该端点发送一个空的 GET 请求，远程解析器应返回代理主机。ClickHouse 将使用它来使用以下模板形成代理 URI：`{proxy_scheme}://{proxy_host}:{proxy_port}`

### 配置模板

```xml
<proxy>
    <http>
        <resolver>
            <endpoint>http://resolver:8080/hostname</endpoint>
            <proxy_scheme>http</proxy_scheme>
            <proxy_port>80</proxy_port>
            <proxy_cache_time>10</proxy_cache_time>
        </resolver>
    </http>

    <https>
        <resolver>
            <endpoint>http://resolver:8080/hostname</endpoint>
            <proxy_scheme>http</proxy_scheme>
            <proxy_port>3128</proxy_port>
            <proxy_cache_time>10</proxy_cache_time>
        </resolver>
    </https>

</proxy>
```

`<proxy>`字段

* `<http>` — 一个或多个解析器的列表*

* `<https>` — 一个或多个解析器的列表*

`<http>` 和 `<https>` 字段

* <resolver> — 解析器的端点和其他详细信息。您可以有多个 `<resolver>` 元素，但仅使用给定协议的第一个 `<resolver>` 元素。该协议的任何其他 `<resolver>` 元素都将被忽略。这意味着负载平衡（如果需要）应该由远程解析器实现。

`<resolver>`字段

* `<endpoint>` - 代理解析器的 URI

* `<proxy_scheme>` - 最终代理 URI 的协议。这可以是 `http` 或 `https`。

* `<proxy_port>` - 代理解析器的端口号

* `<proxy_cache_time>` - ClickHouse 应缓存来自解析器的值的时间（以秒为单位）。将此值设置为 0 会导致 ClickHouse 为每个 HTTP 或 HTTPS 请求联系解析器。

### 优先级

代理设置按以下顺序确定：

1. 远程代理解析器 

2. 代理列表 

3. 环境变量

ClickHouse 将检查请求协议的最高优先级解析器类型。如果未定义，它将检查下一个最高优先级的解析器类型，直到到达环境解析器。这也允许混合使用解析器类型。

### 禁用通过 http 代理的 https 请求的隧道

默认情况下，隧道（即 `HTTP CONNECT`）用于通过 `HTTP` 代理发出 `HTTPS` 请求。此设置可用于禁用它。

# 查询级别设置项

有多种方法可以设置 ClickHouse 查询级别设置。设置按层进行配置，每个后续层都会重新定义设置的先前值。

定义设置的优先级顺序是：

1. 直接或在设置配置文件中将设置应用于用户

    * SQL（推荐）

    * 将一个或多个 XML 或 YAML 文件添加到 `/etc/clickhouse-server/users.d`

2. 会话设置

    * 从 ClickHouse Cloud SQL 控制台或 `clickhouse 客户端`以交互模式发送 `SET setting=value`。同样，您可以在 HTTP 协议中使用 ClickHouse 会话。为此，您需要指定 `session_id` HTTP 参数。

3. 查询设置

    * 非交互方式启动`clickhouse客户端`时，设置启动参数`--setting=value`。

    * 使用 HTTP API 时，传递 CGI 参数（`URL?setting_1=value&setting_2=value...`）。

    * 在 SELECT 查询的 SETTINGS 子句中定义设置。设置值仅应用于该查询，并在执行查询后重置为默认值或之前的值。

*** 

## 例子

这些示例均将 `async_insert` 设置的值设置为 `1`，并展示如何检查正在运行的系统中的设置。

### 使用 SQL 将设置直接应用于用户

这将使用设置 `async_inset = 1` 创建用户摄取器：

```sql
CREATE USER ingester
IDENTIFIED WITH sha256_hash BY '7e099f39b84ea79559b3e85ea046804e63725fd1f46b37f281276aae20f86dc3'
SETTINGS async_insert = 1
```

### 检查设置配置文件和分配

```sql
SHOW ACCESS
```

```bash
┌─ACCESS─────────────────────────────────────────────────────────────────────────────┐
│ ...                                                                                │
│ CREATE USER ingester IDENTIFIED WITH sha256_password SETTINGS async_insert = true  │
│ ...                                                                                │
└────────────────────────────────────────────────────────────────────────────────────┘
```

### 使用 SQL 创建设置配置文件并分配给用户

这将使用设置 `async_inset = 1` 创建配置文件 `log_ingest`：

```sql
CREATE
SETTINGS PROFILE log_ingest SETTINGS async_insert = 1
```

这将创建用户 `ingest` 并为用户分配设置配置文件 `log_ingest`：

```sql
CREATE USER ingester
IDENTIFIED WITH sha256_hash BY '7e099f39b84ea79559b3e85ea046804e63725fd1f46b37f281276aae20f86dc3'
SETTINGS PROFILE log_ingest
```

使用 XML 创建设置配置文件和用户

```bash
/etc/clickhouse-server/users.d/users.xml
```

```xml
<clickhouse>
    <profiles>
        <log_ingest>
            <async_insert>1</async_insert>
        </log_ingest>
    </profiles>

    <users>
        <ingester>
            <password_sha256_hex>7e099f39b84ea79559b3e85ea046804e63725fd1f46b37f281276aae20f86dc3</password_sha256_hex>
            <profile>log_ingest</profile>
        </ingester>
        <default replace="true">
            <password_sha256_hex>7e099f39b84ea79559b3e85ea046804e63725fd1f46b37f281276aae20f86dc3</password_sha256_hex>
            <access_management>1</access_management>
            <named_collection_control>1</named_collection_control>
        </default>
    </users>
</clickhouse>
```

### 检查设置配置文件和分配

```sql
SHOW ACCESS
```

```bash
┌─ACCESS─────────────────────────────────────────────────────────────────────────────┐
│ CREATE USER default IDENTIFIED WITH sha256_password                                │
│ CREATE USER ingester IDENTIFIED WITH sha256_password SETTINGS PROFILE log_ingest   │
│ CREATE SETTINGS PROFILE default                                                    │
│ CREATE SETTINGS PROFILE log_ingest SETTINGS async_insert = true                    │
│ CREATE SETTINGS PROFILE readonly SETTINGS readonly = 1                             │
│ ...                                                                                │
└────────────────────────────────────────────────────────────────────────────────────┘
```

### 为会话分配设置

```sql
SET async_insert =1;
SELECT value FROM system.settings where name='async_insert';
```

```bash
┌─value──┐
│ 1      │
└────────┘
```

### 在查询期间分配设置

```sql
INSERT INTO YourTable
SETTINGS async_insert=1
VALUES (...)
```

***

## 将设置转换为其默认值

如果更改设置并希望将其恢复为默认值，请将该值设置为 `DEFAULT`。语法如下：

```sql
SET setting_name = DEFAULT
```

例如，`async_insert` 的默认值为 `0`。假设将其值更改为 `1`：

```sql
SET async_insert = 1;

SELECT value FROM system.settings where name='async_insert';
```

响应是：

```bash
┌─value──┐
│ 1      │
└────────┘
```

以下命令将其值设置回 0：

```sql
SET async_insert = DEFAULT;

SELECT value FROM system.settings where name='async_insert';
```

该设置现在恢复为默认值：

```bash
┌─value───┐
│ 0       │
└─────────┘
```

***

## 自定义设置

除了通用设置之外，用户还可以定义自定义设置。

自定义设置名称必须以预定义前缀之一开头。这些前缀的列表必须在服务器配置文件的 `custom_settings_prefixes` 参数中声明。

```xml
<custom_settings_prefixes>custom_</custom_settings_prefixes>
```

要定义自定义设置，请使用 `SET` 命令：

```sql
SET custom_a = 123;
```

要获取自定义设置的当前值，请使用 `getSetting()` 函数：












