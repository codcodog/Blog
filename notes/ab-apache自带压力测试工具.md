---
title: ab-apache自带压力测试工具
date: 2016-11-25 10:34:59
tags:
- 压力测试
- apache
categories: Linux
---

帮同事测试接口，了解到apache的一个压力测试工具，`ab`.  

概述与用法
==========

`ab`, 全称为`apache bench`,是apache自带的一个压力测试工具.  

对接口模拟1000个请求，10个并发量.  
```
ab -n 1000 -c 10 http:/xxxx/test/oracletest 
```
`-n`：请求数.  
`-c`：并发数.  

`ab`命令详细用法，参考官网：[ab](http://httpd.apache.org/docs/2.4/programs/ab.html)


返回：  
```
This is ApacheBench, Version 2.3 <$Revision: 1748469 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking xxxx (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
Completed 1000 requests
Finished 1000 requests


Server Software:        Apache/2.2.15
Server Hostname:        xxxx
Server Port:            80

Document Path:          /test/oracletest
Document Length:        21 bytes 

Concurrency Level:      10 # 并发数
Time taken for tests:   11.508 seconds 
Complete requests:      1000
Failed requests:        0
Total transferred:      412000 bytes
HTML transferred:       21000 bytes
Requests per second:    86.89 [#/sec] (mean) # 每秒多少请求，这个是非常重要的参数数值，服务器的吞吐量 
Time per request:       115.083 [ms] (mean) # 用户平均请求等待时间 
Time per request:       11.508 [ms] (mean, across all concurrent requests) # 服务器平均处理时间，也就是服务器吞吐量的倒数 
Transfer rate:          34.96 [Kbytes/sec] received

Connection Times (ms)
min  mean[+/-sd] median   max
Connect:       20   24   2.3     23      47
Processing:    71   83   6.4     82     137
Waiting:       71   83   6.4     82     137
Total:         94  107   7.1    105     160

Percentage of the requests served within a certain time (ms)
50%    105
66%    108
75%    110
80%    111
90%    116 # 90%的请求在116ms内返回
95%    119
98%    124
99%    131
100%    160 (longest request)
```

几个概念  
=========

**吞吐率（Requests per second）**
> 服务器并发处理能力的量化描述，单位是reqs/s，指的是某个并发用户数下单位时间内处理的请求数。某个并发用户数下单位时间内能处理的最大请求数，称之为`最大吞吐率`.  

**并发连接数**  
> 某个时刻服务器所接受的请求数目，也就是一个会话.  

**并发用户数**  
> 一个用户可能同时会产生多个会话，也即是连接数.  

**用户平均请求等待时间**  
> 处理完成所有请求数所花费的时间/ （总请求数 / 并发用户数），即
> Time per request = Time taken for tests /（ Complete requests / Concurrency Level）  

**服务器平均请求等待时间**  
> 处理完成所有请求数所花费的时间 / 总请求数，即Time taken for / testsComplete requests
> 可以看到，它是吞吐率的倒数。
> 同时，它也=用户平均请求等待时间/并发用户数，即
> Time per request / Concurrency Level

