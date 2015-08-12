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


###2. AOF方式  
与RDB方式不同，AOF是将Redis执行的每一条`写命令`追加到硬盘文件中。

2.1 AOF配置  
     默认情况下AOF是关闭的，需要开启。
     
	     	############################## APPEND ONLY MODE ###############################
	
		# By default Redis asynchronously dumps the dataset on disk. This mode is
		# good enough in many applications, but an issue with the Redis process or
		# a power outage may result into a few minutes of writes lost (depending on
		# the configured save points).
		#
		# The Append Only File is an alternative persistence mode that provides
		# much better durability. For instance using the default data fsync policy
		# (see later in the config file) Redis can lose just one second of writes in a
		# dramatic event like a server power outage, or a single write if something
		# wrong with the Redis process itself happens, but the operating system is
		# still running correctly.
		#
		# AOF and RDB persistence can be enabled at the same time without problems.
		# If the AOF is enabled on startup Redis will load the AOF, that is the file
		# with the better durability guarantees.
		#
		# Please check http://redis.io/topics/persistence for more information.
		
		appendonly no    #默认是no   yes表示开启    
		
		# The name of the append only file (default: "appendonly.aof")
		
		appendfilename “appendonly.aof”     #默认的文件名
		
		# The fsync() call tells the Operating System to actually write data on disk
		# instead of waiting for more data in the output buffer. Some OS will really flush
		# data on disk, some other OS will just try to do it ASAP.
		#
		# Redis supports three different modes:
		#
		# no: don't fsync, just let the OS flush the data when it wants. Faster.
		# always: fsync after every write to the append only log. Slow, Safest.
		# everysec: fsync only one time every second. Compromise.
		#
		# The default is "everysec", as that's usually the right compromise between
		# speed and data safety. It's up to you to understand if you can relax this to
		# "no" that will let the operating system flush the output buffer when
		# it wants, for better performances (but if you can live with the idea of
		# some data loss consider the default persistence mode that's snapshotting),
		# or on the contrary, use "always" that's very slow but a bit safer than
		# everysec.
		#
		# More details please check the following article:
		# http://antirez.com/post/redis-persistence-demystified.html
		#
		# If unsure, use "everysec".
		
		# appendfsync always            #控制Redis写入硬盘的频率n
		appendfsync everysec
		# appendfsync no
		
		# When the AOF fsync policy is set to always or everysec, and a background
		# saving process (a background save or AOF log background rewriting) is
		# performing a lot of I/O against the disk, in some Linux configurations
		# Redis may block too long on the fsync() call. Note that there is no fix for
		# this currently, as even performing fsync in a different thread will block
		# our synchronous write(2) call.
		#
		# In order to mitigate this problem it's possible to use the following option
		# that will prevent fsync() from being called in the main process while a
		# BGSAVE or BGREWRITEAOF is in progress.
		#
		# This means that while another child is saving, the durability of Redis is
		# the same as "appendfsync none". In practical terms, this means that it is
		# possible to lose up to 30 seconds of log in the worst scenario (with the
		# default Linux settings).
		#
		# If you have latency problems turn this to "yes". Otherwise leave it as
		# "no" that is the safest pick from the point of view of durability.
		
		no-appendfsync-on-rewrite no
		
		# Automatic rewrite of the append only file.
		# Redis is able to automatically rewrite the log file implicitly calling
		# BGREWRITEAOF when the AOF log size grows by the specified percentage.
		#
		# This is how it works: Redis remembers the size of the AOF file after the
		# latest rewrite (if no rewrite has happened since the restart, the size of
		# the AOF at startup is used).
		#
		# This base size is compared to the current size. If the current size is
		# bigger than the specified percentage, the rewrite is triggered. Also
		# you need to specify a minimal size for the AOF file to be rewritten, this
		# is useful to avoid rewriting the AOF file even if the percentage increase
		# is reached but it is still pretty small.
		#
		# Specify a percentage of zero in order to disable the automatic AOF
		# rewrite feature.
		
		#重写控制 （重写的意义见下面）
		auto-aof-rewrite-percentage 100   #当目前的AOF文件大小超过上一次重写时的AOF文件大小的百分之多少时会再次进行重写
		auto-aof-rewrite-min-size 64mb    #限制了允许重写的最小AOF文件的大小
		
		# An AOF file may be found to be truncated at the end during the Redis
		# startup process, when the AOF data gets loaded back into memory.
		# This may happen when the system where Redis is running
		# crashes, especially when an ext4 filesystem is mounted without the
		# data=ordered option (however this can't happen when Redis itself
		# crashes or aborts but the operating system still works correctly).
		#
		# Redis can either exit with an error when this happens, or load as much
		# data as possible (the default now) and start if the AOF file is found
		# to be truncated at the end. The following option controls this behavior.
		#
		# If aof-load-truncated is set to yes, a truncated AOF file is loaded and
		# the Redis server starts emitting a log to inform the user of the event.
		# Otherwise if the option is set to no, the server aborts with an error
		# and refuses to start. When the option is set to no, the user requires
		# to fix the AOF file using the "redis-check-aof" utility before to restart
		# the server.
		#
		# Note that if the AOF file will be found to be corrupted in the middle
		# the server will still exit with an error. This option only applies when
		# Redis will try to read more data from the AOF file but not enough bytes
		# will be found.
		aof-load-truncated yes

 重写的意义：  
 
 		如下：
		SET foo 1
		SET foo 2
		SET foo 3
		redis会写三次AOF文件，而前两次的结果会被第三次覆盖，这样前两次的记录没有意义了	
		重写会在AOF记录中只记录第三次的操作，这样有助于减少AOF文件的大小
