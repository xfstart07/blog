---
title: wrk 压测工具
date: 2018-12-17 16:58:07
categories: Devops
tags:
    - wrk
    - 压测
---

wrk -c10 -t10 -d10s -s post.lua http://localhost:9005/statistic/ ktv_reports --latency

* `-c` 保持10个连接，并发数
* `-t` 开多少个线程
* `-d` 压测时间
* `-s` Lua脚本
* `--latency` 显示延迟分布

<!-- more -->

```
使用方法: wrk <选项> <被测HTTP服务的URL>                            
  Options:                                            
    -c, --connections <N>  跟服务器建立并保持的TCP连接数量  
    -d, --duration    <T>  压测时间           
    -t, --threads     <N>  使用多少个线程进行压测   
                                                      
    -s, --script      <S>  指定Lua脚本路径       
    -H, --header      <H>  为每一个HTTP请求添加HTTP头      
        --latency          在压测结束后，打印延迟统计信息   
        --timeout     <T>  超时时间     
    -v, --version          打印正在使用的wrk的详细版本信息
                                                      
  <N>代表数字参数，支持国际单位 (1k, 1M, 1G)
  <T>代表时间参数，支持时间单位 (2s, 2m, 2h)
```

### 压测结果：

```
Running 30s test @ http://www.bing.com （压测时间30s）
  8 threads and 200 connections （共8个测试线程，200个连接）
  Thread Stats   Avg      Stdev     Max   +/- Stdev
              （平均值） （标准差）（最大值）（正负一个标准差所占比例）
    Latency    46.67ms  215.38ms   1.67s    95.59%
    （延迟）
    Req/Sec     7.91k     1.15k   10.26k    70.77%
    （处理中的请求数）
  Latency Distribution （延迟分布）
     50%    2.93ms
     75%    3.78ms
     90%    4.73ms
     99%    1.35s （99分位的延迟）
  1790465 requests in 30.01s, 684.08MB read （30.01秒内共处理完成了1790465个请求，读取了684.08MB数据）
Requests/sec:  59658.29 （平均每秒处理完成59658.29个请求）
Transfer/sec:     22.79MB （平均每秒读取数据22.79MB）
```

关注指标：`Requests/sec`，`-c`

### Lua 脚本

```
request = function()
  uid = math.random(1, 100)
  wrk.headers["Content-Type"] = "application/json"
  wrk.body = string.format('%s', tostring(uid))
  return wrk.format("POST", "http://localhost:9005/statistic/ktv_reports")
end

```

### 资源

* [Http压测工具wrk使用指南](http://zhaox.github.io/benchmark/2016/12/28/wrk-guidelines)
* https://huoding.com/2017/05/31/620
* https://type.so/linux/lua-script-in-wrk.html
* [wrk 压力测试 http benchmark POST接口 - FelixZh - 博客园](https://www.cnblogs.com/felixzh/p/8400729.html)
