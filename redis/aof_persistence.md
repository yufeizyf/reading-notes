AOF持久化
========
AOF持久化是通过保存Redis服务器所执行的`写命令`来记录数据库状态的。

##AOF持久化过程
1. 将写命令添加到缓存
2. 将缓存中数据写入AOF文件，并同步至磁盘

###将写命令添加到缓存
Redis的server.h中定义了redisServer的数据结构。其中有几条是关于aof的。
		
		/* AOF persistence */
    int aof_state;                  /* AOF_(ON|OFF|WAIT_REWRITE) */
    int aof_fsync;                  /* Kind of fsync() policy */
    char *aof_filename;             /* Name of the AOF file */
    int aof_no_fsync_on_rewrite;    /* Don't fsync if a rewrite is in prog. */
    int aof_rewrite_perc;           /* Rewrite AOF if % growth is > M and... */
    off_t aof_rewrite_min_size;     /* the AOF file is at least N bytes. */
    off_t aof_rewrite_base_size;    /* AOF size on latest startup or rewrite. */
    off_t aof_current_size;         /* AOF current size. */
    int aof_rewrite_scheduled;      /* Rewrite once BGSAVE terminates. */
    pid_t aof_child_pid;            /* PID if rewriting process */
    list *aof_rewrite_buf_blocks;   /* Hold changes during an AOF rewrite. */
    sds aof_buf;      /* AOF buffer, written before entering the event loop */` 
    int aof_fd;       /* File descriptor of currently selected AOF file */
    int aof_selected_db; /* Currently selected DB in AOF */
    time_t aof_flush_postponed_start; /* UNIX time of postponed AOF flush */
    time_t aof_last_fsync;            /* UNIX time of last fsync() */
    time_t aof_rewrite_time_last;   /* Time used by last AOF rewrite run. */
    time_t aof_rewrite_time_start;  /* Current AOF rewrite start time. */
    int aof_lastbgrewrite_status;   /* C_OK or C_ERR */
    unsigned long aof_delayed_fsync;  /* delayed AOF fsync() counter */
    int aof_rewrite_incremental_fsync;/* fsync incrementally while rewriting? */
    int aof_last_write_status;      /* C_OK or C_ERR */
    int aof_last_write_errno;       /* Valid if aof_last_write_status is ERR */
    int aof_load_truncated;         /* Don't stop on unexpected AOF EOF. */
    /* AOF pipes used to communicate between parent and child during rewrite. */
    int aof_pipe_write_data_to_child;
    int aof_pipe_read_data_from_parent;
    int aof_pipe_write_ack_to_parent;
    int aof_pipe_read_ack_from_child;
    int aof_pipe_write_ack_to_child;
    int aof_pipe_read_ack_from_parent;
    int aof_stop_sending_diff;     /* If true stop sending accumulated diffs
                                      to child process. */
    sds aof_child_diff;             /* AOF diff accumulator child side. */
    
这其中的 `sds aof_buf`是用来缓存Redis写命令的。
当Redis Server接收到client发来的写请求，它会处理并将请求以Redis网络通讯协议的方式追加到aof_buf中。

###将缓存中数据写入AOF文件，并同步至磁盘
Redis中的aof.c/flushAppendOnlyFile负责处理将数据从aof_buf写到AOF文件，并根据具体的配置策略来将AOF文件同步到磁盘。  
配置策略在redis.conf中如下：

		# appendfsync always        
		appendfsync everysec
		# appendfsync no
该配置决定了Redis服务器在将数据从aof写到AOF文件后，调用fsync()方法的频率：

#####always
将aof_buf缓冲区中所有内容写入并同步导AOF文件
#####everysec
将aof_buf缓冲区所有内容写入到AOF文件，如果上次同步时间距现在已超过1s，则再次进行同步。`这个同步操作是有一个线程专门负责的`
####no
将aof_buf缓冲区的所有内容写入到AOF文件，但不对AOF文件进行同步，同步的事情交给操作系统

###AOF文件加载
1. 创建一个伪客户端（fake client），执行AOF文件中的命令 。
		
		因为AOF文件中存的是重建数据库需要的 写 命令。而这些命令需要在一个客户端的上下文中执行，所以这里创建一个fake client，之所以是fake client，是因为该客户端直接原语AOF文件执行命令，不是通过网络连接。
2. 从AOF文件中分析并读取一条写命令
3. 使用fake client执行写命令
4. 重复2，3直到命令执行完

##AOF重写
AOF持久化通过保存写命令来记录数据库的状态，所以随着写命令的增加，AOF文件也会增大。但是，如下所示：

		set a 1
		set a 2
		set a 3
当执行set a 3后前两条执行语句在Redis读取AOF文件重建数据库时，其实并没有什么作用了。AOF重写是为了解决这种情况，通过重写AOF文件，去掉冗余命令，减少AOF文件体积，而又不影响数据库的状态。

`AOF重写并不是基于先前写好的AOF文件，而是基于数据库当前状态（！！！！！）`
#####重写流程

		func aof_rewrite() {
			//创建AOF文件
			create_aof_file
			//遍历数据库
			for db in Reids {
				#跳过空数据库
				#向AOF文件写入SELECT命令
				 write_to_aof(SELECT,db)
				for key in db {
					#跳过超过expire time的key
					switch(key) {
						//按对应类型写
						case string
						case List
						case Set
						case SortedSet
					}
				} 
			}    
		}
#####重写方式
由于AOF重写过程中会重写大量数据，为了防止用户进程等待过久，Redis服务进程会fork一个子进程来进行aof文件的重写。这样服务进程可以继续处理客户端发来的请求。

`Warn: 当子进程重写AOF文件时，如果服务进程执行了新的写命令，则有可能导致重写的AOF文件与数据库状态不一致`

为了解决这个问题，Redis设置了一个AOF重写缓冲区
                                     
		                                        Server
                                |
		 --------    command    |   ------                      
	    | client | ---------->  |  |  命  |  
	     --------               |  |  令  |               -----------              
	                            |  |  处  | ------------>| AOF重写缓冲|                     
	                            |  |  理  |               ----------- 
	                            |  |  器  |                    
	                            |   ------
	                            |
	                            |
处理流程：  
服务进程端：
>1. 执行client发来的命令。
>2. 将执行后的命令添加到AOF缓冲区，和AOF重写缓冲区。
>3. 等待子进程写完的信号。
>4. 阻塞，停止接受client发来的请求，将AOF重写缓冲写入AOF文件
>5. 覆盖现有AOF文件
>6. 重新开始工作
