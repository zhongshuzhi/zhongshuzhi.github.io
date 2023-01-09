---
title: 【速查表】一些检查Linux机器配置的命令
date: 2022-12-01 09:18:23
tags: Linux
---
## Linux
```shell
#机型
dmidecode | grep "Product Name"
#CPU
cat /proc/cpuinfo | grep name | uniq #查看cpu型号、主频
lscpu | grep -w 'CPU(s)' | grep -v "line\NUMA" #查看逻辑cpu个数
lscpu | grep -w 'Thread(s) per core' #查看超线程个数，1表示未开启超线程
grep 'processor' /proc/cpuinfo | sort -u | wc -l #查看逻辑cpu个数
cat /proc/cpuinfo | grep 'physical id' | sort | uniq | wc -l #查看物理CPU个数(物理机)
cat /proc/cpuinfo | grep 'cores' | uniq #查看物理CPU个数(虚拟机)
cat /proc/cpuinfo | grep core #查看结果中两个相邻的Core ID是否相同，若相同则代表打开
date #查看系统时区
cat /proc/meminfo | grep MemTotal #查看系统的物理内存大小（KB）
free -m #以MB为单位显示当前内存大小
df -k 查看可用存储大小，单位为KB
# 查看操作系统版本
cat /etc/issue | grep Linux
cat /etc/redhad-release
cat /proc/version #系统内核
sysctl -a #查看系统参数
```

## 数据库
- MySQL
  MySQL的配置文件，在Windows操作系统中存储于my.ini中，在Linux操作系统中存储于my.cnf。
  - my.ini
    my.ini存放在MySQL安装的根目录，以下为my.ini中的参数简介。
    | 序号 | 参数项 | 描述 |
    | --- | --- | --- |
    | 1 | port | 数据库的端口 |
    | 2 | basedir | MySQL的安装路径 |
    | 3 | datadir | MySQL数据库表的存放位置 |
    | 4 | default-character-set | 默认的字符集，这个字符集是服务器端的 |
    | 5 | default-storage-engine | 默认存储引擎 |
    | 6 | sql-mode | 通过这个参数设置检验SQL语句的严格程度 |
    | 7 | max_connection | 允许同时访问MySQL服务器的最大连接数，其中一个连接是保留的，留给管理员专用。 |
    | 8 | query_cache_size | 查询时的缓存大小，缓存中可以存储以前通过select语句查询过的信息，再次查询时就可以直接从缓存中拿出信息。 |
    | 9 | table_cache | 所有进程打开表的总数 |
    | 10 | tmp_table_size | 内存中临时表的总数 |
    | 11 | thread_cache_size | 保留客户端线程的缓存 |
    | 12 | myisam_max_sort_file_size | MySQL重建索引时所允许的最大临时文件的大小 |
    | 13 | myisam_sort_buffer_size | 重建索引时的缓存大小 |
    | 14 | key_buffer_size | 关键词的缓存大小 |
    | 15 | read_buffer_size | MyISAM表全表扫描的缓存大小 |
    | 16 | read_rnd_buffer_size | 表示将排序好的数据存入该缓存中 |

  - my.cnf
    my.cnf是MySQL在Linux/Unix/AIX等系统下启动的配置文件。一般放在MySQL的安装目录中，用户也可以放在其他目录加载。
    `locate my.cnf`列出所有的my.cnf文件。`ps aux|grep mysql| grep 'my.cnf'`找出所有加载了指定配置文件的MySQL进程。
  
  MySQL全局配置 `show global variables`.
  MySQL中影响性能的全局变量（参考）
  1. bulk_insert_buffer_size
  2. concurrent_insert
     并发插入，当表没有空洞时，某进程获取读锁的情况下，其他进程可以在表尾部进行插入。
     0：不允许并发插入
     1：当表没有空洞时，执行并发插入
     2: 不过是否有空洞都执行并发插入
     默认是1
  3. delay_key_write
  4. delayed_insert_limit,delayed_insert_timeout,delayed_queue_size
  5. expire_log_days
     自动删除超过指定天数的日志，设置为0时，表示不自动删除。
  6. flush, flush_time
  7. join_buffer_size
  8. key_buffer)suze
  9. read_buffer_size， read_rnd_buffer_size
