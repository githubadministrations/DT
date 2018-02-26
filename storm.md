目标Scope（解决什么问题）
分布式实时大规模数据流的处理
 
官方定义
Stormis a distributed realtime computation system,Storm exposes a set of primitives for doing realtime computation
 
个人理解
Storm所说的Realtime，个人理解不是强调批量数据处理的快速和随机性，而更多的是强调对实时数据流的连续处理（Stream），相对的概念是HadoopMapReduce框架的Batch处理模式，是一个分布式的Stream模式的数据处理框架。
 
== 实现 ==
 
Feature
 
continuous processon data streams ：连续的数据流处理，Storm之所以为Storm的根本了。
 
Scalable ：一方面，Storm集群的大小规模的伸缩性很好，另一方面，对于一个正在运行的Topology，可以动态的改变它的并发度，动态的增减Worker，thread, task等。
 
Guarantees no data loss ：主要体现为数据处理流程中的Ack机制，用来重新加载处理失败的数据流。
 
Extremely robust/ Fault-tolerant ：总之就是鲁棒性，容错性好。个人理解主要是缘于Nimbus daemon 和 Supervisor daemons 是 fail-fast 和无状态的（所有的状态都保存在Zookeeper或本地磁盘中），因此Daemon可以快速的重启和恢复。
 
Programming language agnostic ：支持多语言编程，个人理解就是使用thrift生成server/Client/Topology的各个组件的接口，可以使用多种语言去具体实现。
 
核心思路，架构
 
Storm处理数据的基本单位是Tuple，也就是一个通用的数据容器，支持一些基本数据类型和自定义类型。在Storm的Topology中流动的是由不限定数目的Tuple组成的数据流（Stream）
 

 
Tuple 数据流在Topology中传递处理，所谓的Topology实际上就是由各种数据处理节点链接成的数据处理链，和绝大多数流数据处理框架很类似，比如处理多媒体数据的Gstreamer等。 Spout节点生成Tuple数据流，各种Bolt节点处理转换并输出Tuple流。
 

 
从Cluster集群的角度来看，和多数分布式数据处理系统一样，由任务监督分配和数据处理两部分组成。NimBus Daemon负责分发任务监控状态等，Supervisor Daemon负责实际执行Topology


 
 
从结构上说，Storm提供的就是一个搭建数据处理链，协调数据流动，方便动态水平扩展集群的一个面向Stream的分布式数据处理框架。至于这个框架具体如何获取和处理Stream数据，各种Input, Output, Filter，Join，Aggregation逻辑完全取决于具体Spout和Bolt的实现。
  
适用领域
 
总体而言，Storm针对的是实时连续数据流的持续处理。其Realtime概念并不侧重于本身处理数据的效率有多高。相反，个人觉得为了其拓扑结构的灵活性，数据在多层链接的Spout和Bolt中流动的时候必然带来一定的额外数据传输开销，此外其保证数据处理鲁棒性的Ack机制也必然带来额外的性能损失。所以纯粹论数据的高效处理，不是Storm的强项。数据的连续及时处理才是Storm的强项和适用领域。常见的应用包括各种实时数据/LOG等的预处理，过滤，统计，持久化，实时状态的监控分析等
 
细节
可以指定各个Bolt处理节点的并发度，便于灵活调度任务
 
== 相关项目 ==
 
上下游项目
 
Trident
 
Trident实际是Storm代码的一部分，是构建在Storm框架上的一个更高Level的抽象。本质上说，就是在Storm的Stream处理模式上，用Spout和Bolt实现了一些常见的业务逻辑的支持如Join/Filter/Aggregation/Grouping等，让开发者更方便的使用Storm。
 
各种 spout 实现 : https://github.com/nathanmarz/storm/wiki/Spout-implementations 如何从各种外部系统获取Tuple数据流供Storm处理，当然取决于各种Spout的实现。
 
类似项目
 
S4  ：http://incubator.apache.org/s4/  S4 is a general-purpose, distributed,scalable, fault-tolerant, pluggable platform that allows programmers to easilydevelop applications for processing continuous unbounded streams of data. 看起来和Storm的Scope完全一致。 从实现上看大概在集群的动态性上的支持较差，此外不支持可靠的处理数据，可能丢失事件（这个可能不能说是BUG或缺失，大概是设计思路上的不同）
 
Borealis: http://www.cs.brown.edu/research/borealis/public/not active any more since 2008.
 
== 相关文献 ==
 
http://storm-project.net/ 项目主页
https://github.com/nathanmarz/storm/wiki 项目文档
 
Design文档
https://github.com/nathanmarz/storm/wiki/Concepts 主要概念
https://github.com/nathanmarz/storm/wiki/Implementation-docs 内部实现文档
 
== 其它 ==
 
Storm的实现除了Java，还使用了Clojure这个类Lisp语言。
