###RDB文件结构

		----------------------------------------------------
   		| REDIS | db_version | databases | EOF | check_sum |
   		----------------------------------------------------
   * REDIS（5字节）。是RDB文件的开头，当程序载入文件时，可以通过这5个字符快速检查文件是否为RDB文件   
   * db_version（4字节）。记录了Redis文件的版本号
   * databases（长度不定）。存放各个数据库中的键值对数据（下面会展开说明）
   * EOF（1字节）。表示RDB文件正文内容的结束，当读到这个值的时候，表示数据库所有的键值对都已经载入完成了
   * check_sum（8字节）。由前四部分计算出。服务器载入RDB文件时，会根据前四部分计算出一个校验和并与check_sum进行对比，判断RDB文件是否有损坏。
   
   
####databese
	
		--------------------------------------
		| databasei | databasej | database k |
		--------------------------------------
如果Redis的database i, j, k非空，则databases中的结构即如图中所示。  
其中databasei代表第i个database以及database中的键值对。其具体形式如下图所示
		
		-----------------------------------------
		| SELECTDB | db_number | key_value_pair |
		-----------------------------------------

* SELECTDB（1字节）:当Redis读到这个字节时，它可以知道，接下来的要读入的是一个数据库号码
* db_number（长度不定）：根据数据库号码的大小
* key_value_pair（长度不定）：保存了databasei中的全部数据

####key_value_pair
key_value_pair分为两种情况：  
	
1. 带失效时间的key，value

		----------------------------------------
		| EXPIRETIME | ms | TYPE | key | value |
		----------------------------------------

2. 无失效时间的key，value

		----------------------
		| TYPE | key | value |
		----------------------
		
二者的TYPE，key，value字段一样，差别在于带失效时间的key_value_pair多了EXPIRETIME，ms两个字段。  

* TYPE（1字节）：表示value的类型
* EXPIRETIME（1字节）：Redis读到这个字段，便知道接下来将读到一个过期时间（这点和SELECTDB作用很像）
* ms（8字节）：UNIX时间戳
* key（长度不定）：`类型是字符串`
* value（长度不定）：类型由TYPE决定
 



		



