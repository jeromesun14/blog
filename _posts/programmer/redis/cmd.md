title: Redis 命令集
date: 2016-05-04 20:33:29
toc: true
tags: [redis]
categories: redis
keywords: [Redis, command]
description: Redis 使用过的命令备忘。
---

* 查看所有 key：KEYS *
* 查看数据库大小：DBSIZE
* 查看数据库基本信息：INFO
```
redis /tmp/redis.sock> info
# Server
redis_version:3.2.1
redis_git_sha1:ce18ff58
redis_git_dirty:0
redis_build_id:2de158ec05fb50c4
redis_mode:standalone
os:Linux 2.6.32.13-Cavium-Octeon mips64
arch_bits:64
multiplexing_api:epoll
gcc_version:4.3.3
process_id:2877
run_id:91d28ac8ed2ab0f4ceb592d472a79ffcb10a1049
tcp_port:16904192
uptime_in_seconds:17846
uptime_in_days:0
hz:10
lru_clock:11220343
executable:/redis-server
config_file:/etc/redis/redis_ce_global.conf

# Clients
connected_clients:25
client_longest_output_list:0
client_biggest_input_buf:0
blocked_clients:0

# Memory
used_memory:2230112
used_memory_human:2.13M
used_memory_rss:3670016
used_memory_rss_human:3.50M
used_memory_peak:2296768
used_memory_peak_human:2.19M
total_system_memory:4111892480
total_system_memory_human:3.83G
used_memory_lua:37888
used_memory_lua_human:37.00K
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
mem_fragmentation_ratio:1.65
mem_allocator:libc

# Persistence
loading:0
rdb_changes_since_last_save:0
rdb_bgsave_in_progress:0
rdb_last_save_time:1470820590
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:0
rdb_current_bgsave_time_sec:-1
aof_enabled:1
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok
aof_current_size:623151
aof_base_size:0
aof_pending_rewrite:0
aof_buffer_length:0
aof_rewrite_buffer_length:0
aof_pending_bio_fsync:0
aof_delayed_fsync:0
aof_file_cmd_num:3936
aof_buf_cmd_num:0
aof_is_shadowfile:0

# Stats
total_connections_received:25
total_commands_processed:4483
instantaneous_ops_per_sec:0
total_net_input_bytes:655148
total_net_output_bytes:6071473
instantaneous_input_kbps:0.00
instantaneous_output_kbps:0.00
rejected_connections:0
sync_full:0
sync_partial_ok:0
sync_partial_err:0
expired_keys:0
evicted_keys:0
keyspace_hits:528
keyspace_misses:2
pubsub_channels:0
pubsub_patterns:11
latest_fork_usec:494
migrate_cached_sockets:0

# Replication
role:master
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

# CPU
used_cpu_sys:0.73
used_cpu_user:0.61
used_cpu_sys_children:0.01
used_cpu_user_children:0.05

# Cluster
cluster_enabled:0

# Keyspace
db0:keys=3073,expires=0,avg_ttl=0
```

