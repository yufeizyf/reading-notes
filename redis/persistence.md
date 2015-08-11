Redis持久化方式
=================
Redis支持两种方式持久化：  

* RDB方式。指定“规则”，然后按照规则定时将内存中的数据存储至硬盘。  
* AOF方式。每次执行命令后将命令本身记录下来。

 
 
这两种方式有着各自的优缺点。

###1. RDB方式 
RDB方式是通过快照完成的，当某一时刻符合一定规则时，Redis会将内存中的数据生成一份副本并存储在磁盘上。  

生成快照的几种情形： 
 
* 根据配置规则进行自动快照  
* 执行SAVE或BGSAVE命令时  
* 执行FLUSHALL命令时  
* 执行复制操作时  

1.1 根据配置规则进行快照  
     Redis的配置文件redis.conf中关于snapshot的配置。  
     
	################################ SNAPSHOTTING ################################  
 	#   
 	# Save the DB on disk:   
 	#   
 	# save <seconds> <changes>   
 	#   
 	# Will save the DB if both the given number of seconds and the given   
 	# number of write operations against the DB occurred.   
 	#   
 	# In the example below the behaviour will be to save:   
 	# after 900 sec (15 min) if at least 1 key changed   
 	# after 300 sec (5 min) if at least 10 keys changed  
 	# after 60 sec if at least 10000 keys changed   
 	#   
 	# Note: you can disable saving completely by commenting out all "save" lines.   
 	#   
 	# It is also possible to remove all the previously configured save   
 	# points by adding a save directive with a single empty string   argument   
 	# like in the following example:   
 	#   
 	# save “”   
 	save 900 1   
 	save 300 10   
 	save 60 10000
 	
 每条规则占一行，以save M N每条规则占一行。形如save M  N  
M 时间窗口  
N 键的改动个数  
每当M时间内间的改动个数超过N时，redis便会进行快照。这些条件的关系是“或”的关系  

1.2 执行SAVE或BGSAVE命令  

* SAVE命令（同步）  
执行SAVE命令，Redis会以同步的方式进行快照，在进行快照的期间会阻塞所有来自客户端的请求。当执行完会返回OK。  
	
		127.0.0.1:6379>save
		OK  
		
* BGSAVE命令（异步）  
  执行BGSAVE命令，Redis会以异步的方式进行快照，快照的同时redis-server还可以继续处理客户端发来的请求。
  `与SAVE不同，这时会直接返回saving started，然后进行快照。可以通过LASTSAVE命令获取最后一次成功执行快照的时间。`
  
  		127.0.0.1:6379>bgsave
  		Background saving started
  		127.0.0.1:6379>lastsave
  		(integer)1439216583
  		
1.3 执行FLUSHALL命令（清空）  
执行FLUSHALL命令时，如果redis.conf中配置了save  M  N的快照条件，无论清空过程是否触发SAVE条件，均会执行快照。（Ps：如果配置save M  N，则不会进行快照）    

1.4 执行复制命令  
     主从模式下。

`RDB快照过程`
		
		（1） Redis使用fork复制一份当前进程（父进程）的副本（子进程）。子进程获得的数据是fork时刻的，子进程将这份数据写入硬盘，而父进程之后进行写操作不会影响子进程的工作过程。  
		（2） 父进程继续接收并处理客户端发来的请求，子进程将数据写入硬盘的临时文件  
		（3） 当子进程写完所有数据后，会用该临时文件替换先前的RDB文件
		
`RDB方式中存在的问题`
		
		（1）进行快照的时候，会复制一份父进程的数据，这样会额外占用一定内存
		（2）如果Redis异常退出，则最后一次快照以后的数据会丢失。


