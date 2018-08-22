This mainly focus on the basic knowledge of big data
====================================

1. Google file system HDFS的NameNode/DataNode对应master和chunkserver
-------------------------------------

文件的存储：一个block是1024bytes，当需要保存大文件时：把block变成chunk: 64*1024* blocks  
使用chunk优缺点：减少元数据，减少流量  但是小文件浪费空间  
超大文件： Master内只存偏移量的index，指向其他chunkserver  
但是会极大浪费master的容量。  
解决方法：只在master上存chunk的index，具体偏移量保存在chunkserver上  

如何发现数据损坏：对每个block维持一个checksum，每次读取时对checksum进行验证  
减少chunkserver挂导致的后果：挂在多个chunkservers里面，一式三份。如何选择：1.尽量选择硬盘利用率低 但是 2.限制最新数据块的写入数量，让数据分布均匀 3. 跨机架跨中心，一个存美国两个存中国。  
恢复损坏的chunk：chunkserver告诉master然后由master告诉他，chunkserver找其他chunkserver拿  
发现挂掉的chunkserver 长时间master没有听到没有心跳 有可能会用其他chunkserver来ping他  
挂掉后怎么恢复数据：master检查存活的副本数量，哪个少于3份就恢复  
应对热点：热点平衡进程，记录数据块访问热度/带宽/硬盘利用率，然后将副本复制到更多的chunkserver上  
如何读文件：client向master发出文件请求，master告诉client一个具体的chunkserver，client去找chunkserver要  
如何写文件以及与HDFS的区别：  
https://blog.csdn.net/guyuealian/article/details/51867896
注意！ HDFS 在考虑写入模型时做了一个简化，就是同一时刻只允许一个写入者或追加者。   

2. Big table
------------------------------------------
用table来存文件的sorted的key-value，找到就可以了  
table很大：拆成小表：大表用来存小表的存储位置 小表内是排好序的，但表间不是  table超大：小表再拆分为小小表，小表结构跟大表一样，小小表跟之前的小表结构一样。  
写入：使用内存表加速，先在内存排好序，满了直接写入disk变成小小表 a list of SStable + memorytable  
避免内存表数据丢失：往disk里面写log，说我加了什么，虽然速度变慢了但是能保证数据不丢失 a list of SStable + memorytable + disk里的log   
读数据：在所有表里面都要找，通过便利disk，因为在表见是无序的，这样效率很低  
如何加速：给小小表建立索引，通过把索引加载到内存。索引用来找到数据在disk中的位置。小小表变成了=a list of 64k blocks + index  
继续加速：bloomfilter。小小表变成了=a list of 64k blocks + index + bloomfilter  
如何构建bloomfilter：将数据通过n个hash，映射到binary chunk上。查找的数据hash之后，全部binary都为1才去遍历那个block，过滤掉很多  
表的存储视图：逻辑视图(Sql)转换为物理存储，例如把一个人的照片，身高体重和时间全部放到一条数据里面，不考虑他们的关系，NoSql概念  
整体架构：用户通过Tablet server读取需要数据，Master负责管理和负载均衡，server里面存的是table和logs，一式三份放在不同地方。还有一个集群调度的系统负责监控  
* 总结：Bigtable通过key-value的物理结构存储超大表  


3. NoSql
------------------------------------------
传统的SQL为逻辑视图(database analyse)，NoSql是物理视图,目的是应对 数据越来越多 因此需要 更大更强的服务器和更多的小服务器。  
聚合性数据库  1. 属于key-value数据库: redis   2. Column-base：HBase/Cassandra 通过行列构成key，每一行相当于一个document    3. 属于document的数据库：每个文件的数据放一起,通过文件的id找：mongoDB  
结构：属于无显示结构但是有隐式结构  
聚合方式是固定的，因此如果想用另外一种方式聚合：使用Map-Reduce。  
如何选择：如果只用一种聚集方式：NoSql  
一致性：逻辑一致性：多个用户同时操作  副本一致性：多个副本  
 CAP：    
* 当Partition产生时，如何平衡Consistency和Availability 一致性和延迟性的平衡：网络不通时，需要更新数据，是等到所有的机器都能访问再更新还是随便找一个更新，那么很快就可以用  的平衡。    


4. MapReduce的流程  
------------------------------------
   
![image](https://github.com/GaryWong94/Big-data-source/blob/master/pic/FireShot%20Capture%201%20-%20%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BA%E8%AE%B2%E8%A7%A3%20MapReduce%20-%20YouTube%20-%20https___www.youtube.com_watch_v%3DRz8JCS9TfOQ.png)
