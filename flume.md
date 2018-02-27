flume中的一些重要概念：

1. flume Event：flume 事件，被定义为一个具有有效荷载的字节数据流和可选的字符串属性集。(json格式的字符串，由headers和body两部分组成)

 

2. flume Agent：flume 代理，是一个进程承载从外部源事件流到下一个目的地的过程。包含source channel和sink。

3. Source：数据源，消耗外部传递给他的事件，外部源将数据按照flume Source 能识别的格式将Flume 事件发送给flume Source。

4. Channel：数据通道，是一个被动的存储，用来保持事件，直到由一个flume Sink消耗。

5. Sink : 数据汇聚点，代表外部数据存放位置。发送flume event到指定的外部目标.

 

配置文件模版：在 conf目录里面新建一个文件，文件名自定义

＃example.conf：单节点Flume配置
＃命名Agent a1的组件
a1.sources = r1
a1.sinks = k1
a1.channels = c1

＃描述/配置Source
a1.sources.r1.type = netcat   Source 类型（还有其他很多）
a1.sources.r1.bind = 0.0.0.0  绑定ip

a1.sources.r1.port = 44444    端口号

＃描述Sink
a1.sinks.k1.type = logger       sink类型

＃描述内存Channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000 
a1.channels.c1.transactionCapacity = 100

＃为Channle绑定Source和Sink
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
注意：
（1）一个配置文件中可以配置多个Agent，一个Agent中可以包含多个Source、Sink、Channel。
（2）一个Source 可以绑定到多个通道，但一个Sink只能绑定到一个通道

Source类型：

Avro Source 序列化

Exec Source 命令输出作为源

＃描述/配置Source
a1.sources.r1.type = exec
a1.sources.r1.command = ping 192.168.242.102 

Spooling Directory Source  

这个Source允许你将将要收集的数据放置到"自动搜集"目录中。这个Source将监视该目录，并将解析新文件的出现。事件处理逻辑是可插拔的，当一个文件被完全读入通道，它会被重命名或可选的直接删除。
要注意的是，放置到自动搜集目录下的文件不能修改，如果修改，则flume会报错。另外，也不能产生重名的文件，如果有重名的文件被放置进来，则flume会报错

＃描述/配置Source
a1.sources.r1.type = spooldir
a1.sources.r1.spoolDir=/home/park/work/apache-flume-1.6.0-bin/mydata

NetCat Source  监听一个指定端口，并将接收到的数据的每一行转换为一个事件

Sequence Generator Source 简单的序列发生器，不断的产生事件，值是从0开始每次递增1。

HTTP Source 接受HTTP的GET和POST请求作为Flume的事件,其中GET方式应该只用于试验（默认jsonhandler，文件上传 blobhandler）

＃描述/配置Source
a1.sources.r1.type = http
a1.sources.r1.port = 66666

 

主要用来进行测试

通过flume的工具启动agent

$ bin/flume-ng agent --conf ../conf --conf-file ../conf/example.conf --name a1 -Dflume.root.logger=INFO,console

 

Interceptor

 

＃描述/配置Source

a1.sources.r1.type  =  http

a1.sources.r1.port  =  44444

a1.sources.r1.interceptors = i1 i2 i3 i4 i5 i6

a1.sources.r1.interceptors.i1.type = timestamp  时间戳拦截器

#ip是拦截者所在机器的ip

a1.sources.r1.interceptors.i2.type = host　　　　 host拦截器

a1.sources.r1.interceptors.i3.type = static 　　　  静态拦截器

a1.sources.r1.interceptors.i3.key = country

a1.sources.r1.interceptors.i3.value = China

a1.sources.r1.interceptors.i4.type = org.apache.flume.sink.solr.morphline.UUIDInterceptor$Builder UUid拦截器

a1.sources.r1.interceptors.i5.type = search_replace  查找替换拦截器

#将所有的数字替换成*

a1.sources.r1.interceptors.i5.searchPattern = [0-9]  

a1.sources.r1.interceptors.i5.replaceString = *

a1.sources.r1.interceptors.i6.type = regex_filter       正则拦截器

#只要是a开头的抛弃

a1.sources.r1.interceptors.i6.regex = ^a.*　　　　　　

a1.sources.r1.interceptors.i6.excludeEvents = true

 

 

 

 

Processor

......

#描述Sink

a1.sinkgroups = g1

a1.sinkgroups.g1.sinks = k1 k2

a1.sinkgroups.g1.processor.type = failover  失败重连

a1.sinkgroups.g1.processor.priority.k1=5

a1.sinkgroups.g1.processor.priority.k2=10

......

#为Channel绑定Source和Sink

a1.sources.r1.channels=c1

a1.sinks.k1.channel=c1

a1.sinks.k2.channel=c1

------------------------------------------------------

#描述Sink 负载均衡方式

a1.sinkgroups = g1

a1.sinkgroups.g1.sinks = k1 k2

a1.sinkgroups.g1.processor.type = load_balance

a1.sinkgroups.g1.processor.selector = random
