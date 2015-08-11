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