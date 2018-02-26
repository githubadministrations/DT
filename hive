常见的hive查询操作有：count、group by、order by、join、distribute by、sort by、clusrer by、union all
一、常见的聚合函数
1、count计数
count(*) 所有值不全为NULL时，加1操作；
count(1) 不管有没有值，只要有这条记录，值就加1
count(col) col列里面的值为null，值不会加1，这个列里面的值不为NULL，才加1 
2、sum求和
    sum(可转成数字的值)  返回bigint
3、avg求平均值
    avg(可转成数字的值)   返回double
4、distinct不同值个数
    count(distinct col)
二、order by按照某些字段排序
样例：select col1,other... from table where conditio order by col1,col2 [asc|desc]
注意：order by 后面可以有多列进行排序，默认按字典排序
order by为全局排序
order by需要reduce操作，且只有一个reduce，与配置无关。数据量很大时，慎用。
执行流程：

 从表中读取数据，执行where条件，以col1，col2列的值做成组合key，其他列值为value，然后在把数据传到同一个reduce中，根据需要的排序方式进行。
三、group by按照某些字段的值进行分组，有相同值放到一起。
    样例：
Sql代码  收藏代码
select col1[,col2],count(1),sel_expr(聚合操作) from table  
where condition        --->Map端执行  
group by col1[,col2]   --->Reduce端执行  
[having]               --->Reduce端执行  
    注意：
select后面非聚合列，必须出现在group by中
select后面除了普通列就是一些聚合操作
group by后面也可以跟表达式，比如substr(col)
   特性
使用了Reduce操作，受限于reduce数量，设置reduce参数mapred，reduce，tasks
输出文件个数与reduce数相同，文件大小与reduce处理的数据量有关
   问题
网络负载过重
数据倾斜，优化参数hive.groupby.skewindata为true,会启动一个优化程序，避免数据倾斜。
四、join表连接
    两个表m,n之间按照on条件连接，m中的一条记录和n中的一条记录组成一条新记录。
    join等值连接（内连接），只有某个值在m和n中同时存在时。
    left outer join左外连接，左边表中的值无论是否在b中存在时，都输出：右边表中的值，只有在左边表中存在时才输出。
    right outer join和left outer join相反。
    left semi join类似exists。即查找a表中的数据，是否在b表中存在，找出存在的数据。
    mapjoin：在map端完成join操作，不需要用reduce，基于内存做join，属于优化操作。
五、Mapjoin
    在map端把小表加载到内存中，然后读取大表和内存中的小表完成连接操作。其中使用了分布式缓存技术。
    有点：不消耗集群的reduce资源；减少了reduce操作，加快程序执行；降低网络负载。
    缺点：占用部分内存，所以加载到内存中的表不能过大，因为每个计算节点都会加载一次；生成较多的小文件。
    执行方式：
自动方式，配置以下参数：hive自动根据sql，选择使用common join或者map join【set hive.auto.convert.join=true;hive.mapjoin.smalltable.filesize=25mb】
手动指定：select /*+mapjoin(n)*/ m.col,m.col2,n.col3 from m join n on m.col=n.col。注意/*+mapjoin(n)*/不能省略，只需替换表名n值即可。
    mapjoin的使用场景：
关联操作中有一张表非常小
不等值的连接操作
六、hive分桶JOIN
    对于每一个表或者分区，hive可以进一步组织成桶，也就是说桶是更为细粒度的数据范围划分。
    hive是针对某一列进行分桶。hive采用对列值哈希，然后除以桶的个数求余的方式决定该条记录存放在哪个桶中。
    好处：获得更高的查询处理效率；使取样更高效。
Sql代码  收藏代码
create table bucketed_user(  
   id  int,  
   name  string  
)clustered by (id) sorted by (name) into 4 buckets row format delimited fields terminated by '\t' stored as textfile;  
  
set hive.enforce.bucketing=true;  
分桶的使用  
select * from bucketed_user tablesample(bucket 1 out of 2 on id)  
  
BUCKET JOIN  
  
set hive.optimize.bucketmapjoin=true;  
set hive.optimize.bucketmapjoin.sortedmerge=true;  
set hive.input.format=org.apache.hadoop.hive.ql.io.BucketizedHiveInputFormat;  
   连接两个在（包含连接列）相同列上划分了桶的表，可以使用Map端连接高效的实现。比如join操作。
   对于join操作两个表有一个相同的列，如果对这两个表都进行了桶操作。那个将保存相同列值的痛进行join操作就可以，可以大大减少join的数据量。
    对于map端连接的情况，两个表以相同方式划分桶。处理左边表内某个桶的mpper知道右边表内相匹配的行在对应的桶内。因此，mapper只需要获取那个桶即可进行连接。这一优化方法并不一定要求表必须桶的个数相同，两个表的桶个数是倍数关系也可以。
七、distribute by、sort by
   distribute分散数据。distribute by col----按照col列把数据散列到不同的reduce。
   sort by排序，sort by col----按照col列把数据排序。
 distribute by与group by对比。都是按照key值划分数据，都是用reduce操作，唯一不同的是distribute by只是单纯的分散数据，而group by把相同key的数据聚集到一起，后续必须是聚合操作。
order by与sort by对比：order by是全局排序；sort by只是确保每个reduce上面输出的数据有序。如果只有一个reduce时，和order by作用一样。
八、cluster by
    把有相同值的数据聚集到一起，并排序。效果等价于distribute by col sort by col。cluster by col<==>distribute by col sort by col。
