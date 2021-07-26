

# AB命令压测

首先环境非常重要，需要跟生产环境一致。

例如：

- 软件的版本
- 网络带宽

压测可以在服务器本机上跑压测程序，更有针对性看服务器的性能。当然，压测程序本身不应该占用过多的CPU和内存资源而影响到服务。



## 参数说明

- -n：平台中所执行的请求个数，默认执行一个请求；
- -c：一次产生的请求个数，默认是一次一个；
- -t：测试进行的最大秒数，默认没有时间限制；
- -p：包含POST的数据的文件；
- -P
- -T：POST方式所使用的Content-type头信息；
- -V：显示版本号并退出；
- -w：以HTML表格的格数输出结果；
- -i：执行head请求，而不是GET；
- -X：对请求使用代理服务器；
- -C：对请求附加一个Cookie行；
- -H：对请求附加额外的头信息，例如token；
- -h：帮助手册；
- -k：启用Http KeepAlive功能，即在一个HTTP会话中执行多个请求。默认时，不启用KeepAlive功能；
- -g：对测试结果写入到一个文件中；



## 性能指标

### 吞吐率（Request Per Second）

服务器并发处理能力的量化描述，单位是reqs/s，指的是在某个并发用户数下单位时间内

处理的请求数。某个并发用户数下单位时间内能处理的最大请求数，称之为最大吞吐

率。



记住：吞吐率是基于并发用户数的。这句话代表了两个含义：

1. 吞吐率和并发用户数相关；
2. 不同的并发用户数下，吞吐率一般是不同的；



计算公式：总请求数/处理完成这些请求数所花费的时间，即

> Request per second=Complete requests/Time taken for tests

必须要说明的是，这个数值表示当前机器的整体性能，值越大越好。



### 并发连接数（The Number of Concurrent connections）

并发连接数指的是某个时刻服务器所接受的请求数目，简单的讲，就是一个会话。



### 并发用户数（Concurrency Level）

要注意区分这个概念和并发连接数之间的区别，一个用户可能同时会产生多个会话，也

即连接数。在HTTP/1.1下，IE7支持两个并发连接，IE8支持6个并发连接，FireFox3支持

4个并发连接，所以相应的，我们的并发用户数就得除以这个基数。



### 用户平均请求等待时间（Time per request）

计算公式：处理完成所有请求数所花费的时间/（总请求数/并发用户数），即：

> Time per request=Time taken for tests/（Complete requests/Concurrency Level）



### 服务器平均请求等待时间（Time per request:across all concurrent requests）

计算公式：处理完成所有请求数所花费的时间/总请求数，即：

> Time taken for/testsComplete requests

可以看到，它是吞吐率的倒数。同时，它也等于用户平均请求等待时间/并发用户数，即

> Time per request/Concurrency Level



## ab的使用

ab -c 100 -n 100000 -p url.txt -T application/x-www-form-urlencoded http://127.0.0.1:8082/product/decr

表示同时处理100000个请求并运行100次，使用POST方式，数据在url.txt文件中。



## 打印信息

- Server Software：表示被测试的Web服务器软件名称；
- Server Hostname：表示请求的URL主机名；
- Server Port：表示被测试的Web服务器请求的端口；
- Document Path：表示请求的URL中的根绝对路径；
- Document Length：HTTP响应数据的正文长度；
- Concurrency Level：并发用户数；
- Time taken for tests：所有这些请求被处理完成所花费的总时间；
- Complete requests：总请求数量；
- Failed requests：失败的请求数量，例如请求在连接服务器、发送数据等环节发生异常，以及无响应后超时的情况。如果接收到的HTTP响应数据的头信息中含有2XX以外的状态码，则会在测试结果中显示另一个名为“Non-2xx responses”的统计项，用于统计这部分请求数，这些请求并不算在失败的请求中；
- Total transferred：所有请求的响应数据长度总和，包括每个HTTP响应数据的头信息和正文数据的长度。注意这里不包括HTTP请求数据的长度，仅仅为web服务器流向用户PC的应用层数据总长度；
- HTML transferred：所有请求的响应数据中正文数据的总和，也就是减去了Total transferred中HTTP响应数据中的头信息的长度；
- Requests per second：吞吐率；
- Time per request：用户平均请求等待时间；
- Time per requet(across all concurrent request)：服务器平均请求等待时间，
- Transfer rate：这些请求在单位时间内从服务器获取的数据长度，计算公式：Total trnasferred/ Time taken for tests，这个统计很好的说明服务器的处理能力达到极限时，其出口宽带的需求量；
- Percentage of the requests served within a certain time (ms)
  - 这部分数据用于描述每个请求处理时间的分布情况，比如以上测试，80%的请求处理时间都不超过6ms，这个处理时间是指前面的Time per request，即对于单个用户而言，平均每个请求的处理时间。





