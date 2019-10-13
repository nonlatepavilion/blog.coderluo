---
title: 数据库压测工具Sysbench安装
author: coderluo
img: featureImages 中的某个值
top: true
cover: true
coverImg: /images/1.jpg
summary: msyql数据库压测工具学习之Sysbench
categories: 数据库
tags:
  - Sysbench
date: 2019-10-13 21:13:39
---



> 最近公司准备去oracle，迁移到mysql集群。分布式数据库中间件我们技术选型选择了mycat。这无疑前期的准备工作需要做好，在运维同学的帮助下，集群已经搭建好，接下来我们任务就是压测。这篇文章简单记录我所选择的压测工具Sysbench的安装和使用。

## sysbench 简介

>   Sysbench是一个模块化的、跨平台、多线程基准测试工具，主要用于评估测试各种不同系统参数下的数据库负载情况。它主要包括以下几种方式的测试：cpu性能，磁盘io性能，线程调度性能，内存分配及传输速度和数据库性能。由于本人是dba，因此重点关注sysbench测试数据库的场景。目前sysbench支持mysql，postgreSQL，oracle三种数据源。



## 测试环境

系统：Centos-8

Sysbench：1.0.15,下载地址：[https://github.com/akopytov/sysbench](https://github.com/akopytov/sysbench)

## 安装步骤

### sysbench的一些依赖安装

```shell
yum -y install  make automake libtool pkgconfig libaio-devel vim-common
如果没有安装mysql,则需要安装mysql-devel
yum install mysql-devel
```



### 安装sysbench

1. 进入到sysbench源码目录

```shell
cd /data/sysbench/sysbench-1.0.15
```



2. 执行autogen.sh用它来生成configure这个文件

```shell
./autogen.sh
```



3. 执行configure && make && make install 来完成sysbench的安装

```shell
./configure --prefix=/wls/sysbench/ --build=x86_64 
make && make install
```

注：如果不添加--build=x86_64可能会提示如下错误：

```shell
checking build system type... Invalid configuration `x86_64-unknown-linux-': machine `x86_64-unknown-linux' not recognized

configure: error: /bin/sh config/config.sub x86_64-unknown-linux- failed
```



## 验证是否安装成功

```shell
[root@05823ea529b0 sysbench-1.0.15]# /wls/sysbench/bin/sysbench --version
sysbench 1.0.15
[root@05823ea529b0 sysbench-1.0.15]# /wls/sysbench/bin/sysbench cpu run
sysbench 1.0.15 (using bundled LuaJIT 2.1.0-beta2)

Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Prime numbers limit: 10000

Initializing worker threads...

Threads started!

CPU speed:
    events per second:  1158.09

General statistics:
    total time:                          10.0005s
    total number of events:              11583

Latency (ms):
         min:                                    0.79
         avg:                                    0.86
         max:                                    1.92
         95th percentile:                        1.01
         sum:                                 9985.69

Threads fairness:
    events (avg/stddev):           11583.0000/0.00
    execution time (avg/stddev):   9.9857/0.00
```



##  sysbench 查看帮助

```shell
[root@05823ea529b0 sysbench-1.0.15]# /wls/sysbench/bin/sysbench --help
Usage:
  sysbench [options]... [testname] [command]

Commands implemented by most tests: prepare run cleanup help

General options:		# 通用选项
  --threads=N                     # 要使用的线程数，默认 1 个 [1]
  --events=N                      # 最大允许的事件个数，默认为[0]
  --time=N                        # 最大的总执行时间，以秒为单位默认为[10]
  --forced-shutdown=STRING        number of seconds to wait after the --time limit before forcing shutdown, or 'off' to disable [off]
  --thread-stack-size=SIZE        size of stack per thread [64K]
  --rate=N                        # 平均事务率。0为无限率[0]
  --report-interval=N             # 以秒为单位定期报告具有指定间隔的中间统计信息， 0 禁用中间报告，默认为0
  --report-checkpoints=[LIST,...] dump full statistics and reset all counters at specified points in time. The argument is a list of comma-separated values representing the amount of time in seconds elapsed from start of test when report checkpoint(s) must be performed. Report checkpoints are off by default. []
  --debug[=on|off]                # 打印更多 debug 信息 [off]
  --validate[=on|off]             perform validation checks where possible [off]
  --help[=on|off]                 print help and exit [off]
  --version[=on|off]              print version and exit [off]
  --config-file=FILENAME          # 包含命令行选项的文件
  --tx-rate=N                     deprecated alias for --rate [0]
  --max-requests=N                deprecated alias for --events [0]
  --max-time=N                    deprecated alias for --time [0]
  --num-threads=N                 deprecated alias for --threads [1]

Pseudo-Random Numbers Generator options:
  --rand-type=STRING random numbers distribution {uniform,gaussian,special,pareto} [special]
  --rand-spec-iter=N number of iterations used for numbers generation [12]
  --rand-spec-pct=N  percentage of values to be treated as 'special' (for special distribution) [1]
  --rand-spec-res=N  percentage of 'special' values to use (for special distribution) [75]
  --rand-seed=N      seed for random number generator. When 0, the current time is used as a RNG seed. [0]
  --rand-pareto-h=N  parameter h for pareto distribution [0.2]

Log options:
  --verbosity=N verbosity level {5 - debug, 0 - only critical messages} [3]

  --percentile=N       percentile to calculate in latency statistics (1-100). Use the special value of 0 to disable percentile calculations [95]
  --histogram[=on|off] print latency histogram in report [off]

General database options:

  --db-driver=STRING  specifies database driver to use ('help' to get list of available drivers) [mysql]
  --db-ps-mode=STRING prepared statements usage mode {auto, disable} [auto]
  --db-debug[=on|off] print database-specific debug information [off]


Compiled-in database drivers:
  mysql - MySQL driver

mysql options:     # MySQL 数据库专用选项
  --mysql-host=[LIST,...]          MySQL server host [localhost]
  --mysql-port=[LIST,...]          MySQL server port [3306]
  --mysql-socket=[LIST,...]        MySQL socket
  --mysql-user=STRING              MySQL user [sbtest]
  --mysql-password=STRING          MySQL password []
  --mysql-db=STRING                MySQL database name [sbtest]
  --mysql-ssl[=on|off]             use SSL connections, if available in the client library [off]
  --mysql-ssl-cipher=STRING        use specific cipher for SSL connections []
  --mysql-compression[=on|off]     use compression, if available in the client library [off]
  --mysql-debug[=on|off]           trace all client library calls [off]
  --mysql-ignore-errors=[LIST,...] list of errors to ignore, or "all" [1213,1020,1205]
  --mysql-dry-run[=on|off]         Dry run, pretend that all MySQL client API calls are successful without executing them [off]

Compiled-in tests:
  fileio - File I/O test
  cpu - CPU performance test
  memory - Memory functions speed test
  threads - Threads subsystem performance test
  mutex - Mutex performance test

See 'sysbench <testname> help' for a list of options for each test.
```



## sysbench 对数据库进行压力测试的过程

- prepare 阶段 这个阶段是用来做准备的、比较说建立好测试用的表、并向表中填充数据;
- run阶段 这个阶段是才是去跑压力测试的SQL;
- cleanup 阶段 这个阶段是去清除数据的、也就是prepare阶段初始化好的表要都drop掉;



## sysbench 中的测试类型大致可以分成内置的，lua脚本自定义的测试

1. 内置: fileio 、cpu 、memory 、threads 、 mutex 
2. lua脚本自定义型：sysbench 自身内涵了一些测试脚本放在了安装目录下:

```shell
[root@05823ea529b0 sysbench-1.0.15]# ls -al /wls/sysbench/share/sysbench/
total 72
drwxr-xr-x 3 root root  4096 Oct 13 10:29 .
drwxr-xr-x 4 root root  4096 Oct 13 10:29 ..
-rwxr-xr-x 1 root root  1452 Oct 13 10:29 bulk_insert.lua
-rw-r--r-- 1 root root 14369 Oct 13 10:29 oltp_common.lua
-rwxr-xr-x 1 root root  1290 Oct 13 10:29 oltp_delete.lua
-rwxr-xr-x 1 root root  2415 Oct 13 10:29 oltp_insert.lua
-rwxr-xr-x 1 root root  1265 Oct 13 10:29 oltp_point_select.lua
-rwxr-xr-x 1 root root  1649 Oct 13 10:29 oltp_read_only.lua
-rwxr-xr-x 1 root root  1824 Oct 13 10:29 oltp_read_write.lua
-rwxr-xr-x 1 root root  1118 Oct 13 10:29 oltp_update_index.lua
-rwxr-xr-x 1 root root  1127 Oct 13 10:29 oltp_update_non_index.lua
-rwxr-xr-x 1 root root  1440 Oct 13 10:29 oltp_write_only.lua
-rwxr-xr-x 1 root root  1919 Oct 13 10:29 select_random_points.lua
-rwxr-xr-x 1 root root  2118 Oct 13 10:29 select_random_ranges.lua
drwxr-xr-x 4 root root  4096 Oct 13 10:29 tests
```



## 通过sysbench自带的lua脚本对mysql进行测试

















