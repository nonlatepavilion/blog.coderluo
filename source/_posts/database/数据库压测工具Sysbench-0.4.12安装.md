---
title: 数据库压测工具Sysbench-0.4.12安装
tags:
  - Sysbench
author: coderluo
img: http://media.coderluo.top//article/sysbenchsysbench%E5%B0%81%E9%9D%A2-1.png
top: true
cover: true
coverImg: http://media.coderluo.top//article/sysbenchsysbench%E5%B0%81%E9%9D%A2-1.png
categories: 数据库
date: 2019-10-13 21:13:39
---




> 最近公司准备去oracle，迁移到mysql集群。分布式数据库中间件我们技术选型选择了mycat。这无疑前期的准备工作需要做好，在运维同学的帮助下，集群已经搭建好，接下来我们任务就是压测。这篇文章简单记录我所选择的压测工具Sysbench的安装和使用。

## sysbench 简介

>   Sysbench是一个模块化的、跨平台、多线程基准测试工具，主要用于评估测试各种不同系统参数下的数据库负载情况。它主要包括以下几种方式的测试：cpu性能，磁盘io性能，线程调度性能，内存分配及传输速度和数据库性能。由于本人是dba，因此重点关注sysbench测试数据库的场景。目前sysbench支持mysql，postgreSQL，oracle三种数据源。



## 测试环境

系统：Centos-8

Sysbench：sysbench-0.4.12.10,下载地址：[https://github.com/akopytov/sysbench](https://github.com/akopytov/sysbench)

也可以直接点击下载: http://downloads.mysql.com/source/sysbench-0.4.12.10.tar.gz

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
cd /data/sysbench/sysbench-0.4.12
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
sysbench 0.4.12.10
[root@05823ea529b0 sysbench-1.0.15]# ./sysbench --test=cpu run
sysbench 0.4.12.10 (using bundled LuaJIT 2.1.0-beta2)

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
Usage:
  sysbench [general-options]... --test=<test-name> [test-options]... command

General options:
  --num-threads=N             number of threads to use [1]
  --max-requests=N            limit for total number of requests [10000]
  --max-time=N                limit for total execution time in seconds [0]
  --forced-shutdown=STRING    amount of time to wait after --max-time before forcing shutdown [off]
  --thread-stack-size=SIZE    size of stack per thread [32K]
  --init-rng=[on|off]         initialize random number generator [off]
  --seed-rng=N                seed for random number generator, ignored when 0 [0]
  --tx-rate=N                 target transaction rate (tps) [0]
  --tx-jitter=N               target transaction variation, in microseconds [0]
  --report-interval=N         periodically report intermediate statistics with a specified interval in seconds. 0 disables intermediate reports [0]
  --report-checkpoints=[LIST,...]dump full statistics and reset all counters at specified points in time. The argument is a list of comma-separated values representing the amount of time in seconds elapsed from start of test when report checkpoint(s) must be performed. Report checkpoints are off by default. []
  --test=STRING               test to run
  --debug=[on|off]            print more debugging info [off]
  --validate=[on|off]         perform validation checks where possible [off]
  --help=[on|off]             print help and exit
  --version=[on|off]          print version and exit

Log options:
  --verbosity=N      verbosity level {5 - debug, 0 - only critical messages} [4]

  --percentile=N      percentile rank of query response times to count [95]

Compiled-in tests:
  fileio - File I/O test
  cpu - CPU performance test
  memory - Memory functions speed test
  threads - Threads subsystem performance test
  mutex - Mutex performance test
  oltp - OLTP test

Commands: prepare run cleanup help version

See 'sysbench --test=<name> help' for a list of options for each test.

```



## sysbench 对数据库进行压力测试的过程

- prepare 阶段 这个阶段是用来做准备的、比较说建立好测试用的表、并向表中填充数据;
- run阶段 这个阶段是才是去跑压力测试的SQL;
- cleanup 阶段 这个阶段是去清除数据的、也就是prepare阶段初始化好的表要都drop掉;



## sysbench 中的测试类型大致可以分成内置的，lua脚本自定义的测试

1. 内置: fileio 、cpu 、memory 、threads 、 mutex 
2. lua脚本自定义型：sysbench 自身内涵了一些测试脚本放在了安装目录下:

```shell
查看帮忙命令:
[root@izwz909ewdz83smewux7a7z bin]# ./sysbench --help
Usage:
  sysbench [general-options]... --test=<test-name> [test-options]... command

General options:
  --num-threads=N             number of threads to use [1]
  --max-requests=N            limit for total number of requests [10000]
  --max-time=N                limit for total execution time in seconds [0]
  --forced-shutdown=STRING    amount of time to wait after --max-time before forcing shutdown [off]
  --thread-stack-size=SIZE    size of stack per thread [32K]
  --init-rng=[on|off]         initialize random number generator [off]
  --seed-rng=N                seed for random number generator, ignored when 0 [0]
  --tx-rate=N                 target transaction rate (tps) [0]
  --tx-jitter=N               target transaction variation, in microseconds [0]
  --report-interval=N         periodically report intermediate statistics with a specified interval in seconds. 0 disables intermediate reports [0]
  --report-checkpoints=[LIST,...]dump full statistics and reset all counters at specified points in time. The argument is a list of comma-separated values representing the amount of time in seconds elapsed from start of test when report checkpoint(s) must be performed. Report checkpoints are off by default. []
  --test=STRING               test to run
  --debug=[on|off]            print more debugging info [off]
  --validate=[on|off]         perform validation checks where possible [off]
  --help=[on|off]             print help and exit
  --version=[on|off]          print version and exit

Log options:
  --verbosity=N      verbosity level {5 - debug, 0 - only critical messages} [4]

  --percentile=N      percentile rank of query response times to count [95]

Compiled-in tests:
  fileio - File I/O test
  cpu - CPU performance test
  memory - Memory functions speed test
  threads - Threads subsystem performance test
  mutex - Mutex performance test
  oltp - OLTP test

Commands: prepare run cleanup help version

See 'sysbench --test=<name> help' for a list of options for each test.

```



## 通过sysbench自带的lua脚本对mysql进行测试

1. 生成表文件

```shell
[root@izwz909ewdz83smewux7a7z bin]# ./sysbench --test=oltp --oltp-num-tables=100000 --oltp-num-tables=10 --mysql-db=sysbench --mysql-user=root --mysql-password='mydb' --mysql-host=localhost --mysql-port=3306 --db-driver=mysql --init-rng=on prepare

```



2. 多线程测试

```shell
[root@izwz909ewdz83smewux7a7z bin]# ./sysbench --test=oltp --oltp-table-size=100000 --mysql-db=sysbench --mysql-user=root --mysql-password='mydb' --mysql-host=localhost --mysql-port=3306 --db-driver=mysql --oltp-read-only=off --max-time=60 --num-threads=8 --report-interval=10 --oltp-dist-type=uniform --max-requests=0 --percentile=99 run
sysbench 0.4.12.10:  multi-threaded system evaluation benchmark

Running the test with following options:
Number of threads: 8
Report intermediate results every 10 second(s)
Random number generator seed is 0 and will be ignored


Doing OLTP test.
Running mixed OLTP test
Using Uniform distribution
Using "BEGIN" for starting transactions
Using auto_inc on the id column
Using 1 test tables
Threads started!
[  10s] Intermediate results: 8 threads, tps: 582.994023, reads/s: 8233.315586, writes/s: 2940.469852 response time: 48.261058ms (99%)
[  20s] Intermediate results: 8 threads, tps: 589.090480, reads/s: 8248.666697, writes/s: 2945.952392 response time: 35.829896ms (99%)
[  30s] Intermediate results: 8 threads, tps: 487.603128, reads/s: 6829.243810, writes/s: 2439.015647 response time: 65.415122ms (99%)
[  40s] Intermediate results: 8 threads, tps: 487.700557, reads/s: 6830.607800, writes/s: 2439.502786 response time: 78.027721ms (99%)
[  50s] Intermediate results: 8 threads, tps: 478.852588, reads/s: 6703.936228, writes/s: 2394.262939 response time: 71.711459ms (99%)
[  60s] Intermediate results: 8 threads, tps: 510.752034, reads/s: 7150.528473, writes/s: 2553.760169 response time: 43.136643ms (99%)
Time limit exceeded, exiting...
(last message repeated 7 times)
Done.

OLTP test statistics:
    queries performed:
        read:                            440076	读总数
        write:                           157170	写总数
        other:                           62812	他操作总数(SELECT、INSERT、UPDATE、DELETE之外的操作，例如COMMIT等)
        total:                           660058	总数
    transactions:                        31378  (522.91 per sec.)	总事务数(每秒事务数)
    deadlocks:                           56     (0.93 per sec.)	
    read/write requests:                 597246 (9953.02 per sec.)	读写总数(每秒读写次数)
    other operations:                    62812  (1046.75 per sec.)

General statistics:
    total time:                          60.0065s	总耗时
    total number of events:              31378	共发生多少事务数
    total time taken by event execution: 479.8756	所有事务耗时相加(不考虑并行因素)
    response time:
         min:                                  3.77ms	最小耗时
         avg:                                 15.29ms	平均耗时
         max:                                180.78ms	最长耗时
         approx.  99 percentile:              58.12ms	超过99%平均耗时

Threads fairness:
    events (avg/stddev):           3922.2500/6.02
    execution time (avg/stddev):   59.9845/0.00


```



3. 清理上面生成的测试表

```shell
[root@izwz909ewdz83smewux7a7z bin]# ./sysbench --test=oltp --oltp-num-tables=100000 --oltp-num-tables=10 --mysql-db=sysbench --mysql-user=root --mysql-password='mydb' --mysql-host=localhost --mysql-port=3306 --db-driver=mysql --init-rng=on cleanup
sysbench 0.4.12.10:  multi-threaded system evaluation benchmark

Dropping table 'sbtest'...
Dropping table 'sbtest1'...
Dropping table 'sbtest2'...
Dropping table 'sbtest3'...
Dropping table 'sbtest4'...
Dropping table 'sbtest5'...
Dropping table 'sbtest6'...
Dropping table 'sbtest7'...
Dropping table 'sbtest8'...
Dropping table 'sbtest9'...
Done.
[root@izwz909ewdz83smewux7a7z bin]# 

```



由于上面的命令都比较简单,相信大家都能看明白,有什么疑问也欢迎大家留言交流.



**可以对数据库进行调优后，再使用sysbench对OLTP进行测试，看看TPS是不是会有所提高。**

**注意：sysbench的测试只是基准测试，并不能代表实际企业环境下的性能指标。**

个人网站: http://coderluo.top

欢迎关注笔者公众号: **爱上敲代码**, 会定期分享Java技术干活,让枯燥的技术游起来!

![扫码关注](http://media.coderluo.top//gongzhoanghao/erweima.jpg)

