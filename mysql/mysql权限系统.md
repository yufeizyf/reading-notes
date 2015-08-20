mysql权限系统
===========
mysql权限系统保证用户只能做其被允许的操作。当用户连接mysql服务器时，用户的身份由用户的主机（连接服务器的机器）和用户名来决定。连接后发出请求后，系统根据用户的身份和用户的操作授予权限。

###mysql存取控制
两个阶段：
  
* 阶段一：服务器决定用户是否连接。
* 阶段二：对连接成功的用户，服务器检查用户发出的每个请求，看当前用户是否具备权限执行它。例如，如果用户从数据库表中选择(select)行或从数据库删除（DROP）表，服务器确定用户是否对表有SELECT权限或对数据库有DROP权限。

mysql在对存取进行权限控制时基于5张表：user表，db表，host表，tables_priv表和columns_priv表。

		------------------------------------------------------------------------------------
	   |    user    | user表决定是否允许或拒绝到来的连接，对允许的连接，用户具有超级权限              |
	    ------------------------------------------------------------------------------------
	   |    db      | db表决定用户可以从哪个主机上存取那个数据库。范围适用于数据库和数据库对应的表      |
	    ------------------------------------------------------------------------------------
	   |    host    | host表作为db表的扩展被使用。如果想要一个用户能从若干主机使用一个数据库，将db表中的|
	   |            | Host条目设为空值， 然后将那些主机的每一个添加到host表                        |
	    ------------------------------------------------------------------------------------
	   |tables_priv | 该表与db表相似，不同之处是它用于表而不是数据库                               |
	    ------------------------------------------------------------------------------------
	   |columns_priv| 该表作用几乎与db和tables_priv表一样，不同之处是它提供的是针对某些表的特定列的权限|
	    ------------------------------------------------------------------------------------
	    
这5张表具有优先级，如果高优先级的表提供了适当的权限的话，那么就无需查阅优先级较低的授权表了。如果高优先级的表中对应命令的值为N，那么就需要进一步查看低优先级的授权表。
	  
###USER, DB, HOST
mysql连接控制主要用到user表，db表，host表

* 假设现在user表中有一条记录，且该记录的Host和User字段如下
		
		  ---------------------
		 |     user  table     |
		  ---------------------
		 | Host   |  @test.com |
		  ---------------------
		 | User   |  asd       |
		  --------------------- 
* db表中的记录如下

		  ---------------------
		 |      db   table     |
		  ---------------------
		 | Host   |  @test.com |
		  ---------------------
		 | User   |  asd       |
		  ---------------------
		 | DB     |  asdtest   |
		  ---------------------

下面有几种情形：

情形1：

		用户“dsa”连接服务器时将被拒绝。因为user表中不存在该信息。

情形2：

user表中数据库权限（Select_priv）为N，db表中数据库权限（Select_priv）为Y
		
		1. 用户asd尝试连接时将会成功。因为user表中有用户asd的记录
		2. 用户asd试图在数据库asdtest上执行Select命令。
		3. 服务器查看user表，对应于Select命令的条目的值为N，即表示拒绝。
		4. 服务器然后查看db表，对应于Select命令的表项的值为Y，即表示允许。
		5. 该请求将成功执行，因为该用户的db表中的SELECT_priv字段的值为Y。
		
情形3：

user表中数据库权限（Select_priv）为Y，db表中数据库权限（Select_priv）为N

		1. 用户asd尝试连接时将会成功。同情形2
		2. 用户asd试图在数据库asdtest上执行Select命令。
		3. 服务器查看user表，对应于Select命令的表项的值为Y，即表示允许。 因为在user表之内授与的权限是全局性的，所以该请求会成功执行。
		
情形4：

user表中数据库权限为N，db表中数据库权限为N

		1. 用户asd尝试连接时将会成功。
		2. 用户asd试图在数据库asdtest上执行Select命令。
		3. 服务器查看user表，对应于Select命令的表项的值为N，即表示拒绝。
		4. 服务器现在会查看db表，对应于Select命令的表项的值为N，即表示拒绝。
		5. 服务器现在将查找tables_priv和columns_priv表。如果用户的请求符合表中赋予的权限，则准予访问。 否则，访问就会被拒绝。
		

###TABLE_PRIV, COLUMNS_PRIV

tables_priv和columns_priv表是对用户在表上或者列上进行权限控制。`只应当通过GRANT/REVOKE命令进行修改`。

* tables_priv
		
		    -------------------------
		   |       tables_priv       |
		    -------------------------
		   |         Host            |
		    -------------------------
		   |          Db             |
		    -------------------------
		   |         User	         |
		    -------------------------
		   |       Table_name        |
		    ------------------------- 
		   |        Granter          |
		    -------------------------
		   |       Timestamp         |
		    -------------------------
		   |       Table_priv        |
		    -------------------------
		   |       Column_priv       |
		    -------------------------
Tables_priv包含的字段如图所示。Host，Db，User和之前的Db表类似，这里多了Table_name, Table_priv, Column_priv。  
Table_name: 权限控制适用的表。  
Table_priv: 对这个表拥有的权限（Select、Insert、Update、Delete、Create、Drop、Grant、References、Index和Alter）  
Column_priv: 对这个表的字段拥有的权限（Select、Insert、Update和References）

* Columns_priv

		    -------------------------
		   |       tables_priv       |
		    -------------------------
		   |         Host            |
		    -------------------------
		   |          Db             |
		    -------------------------
		   |         User	         |
		    -------------------------
		   |       Table_name        |
		    ------------------------- 
		   |       Column_name       |
		    -------------------------
		   |       Timestamp         |
		    -------------------------
		   |       Column_priv       |
		    -------------------------
		
Columns_priv包含的字段如图所示。Host，Db，User和之前的Db表类似，这里多了Table_name, Column_name, Column_priv。  
Table_name和Column_name共同确定了是哪一张表的哪一个字段。
Column_priv表示该字段上有哪些权限控制（Select、 Insert、 Update和References）

示例：

		GRANT SELECT ON asdtable TO asd@test.com;
该命令允许来自主机test.com的用户asd在表asdtable上执行一个SELECT语句。  
注意，当user,db或者host表中对应该用户的记录中的SELECT字段的值`都为N时`，才需要访问这个表。  
如果三者之一的SELECT字段中`有一个值为Y`的话，那么就`无需`控制该tables_priv表。
    
    	GRANT SELECT, INSERT ON asdtest.asdtable TO asd@test.com;
该命令允许来自主机test.com的用户asd对数据库asdtest中的数据表asdtable执行SELECT和INSERT语句.	   
		
		REVOKE SELECT on asdtest.asdtable from asd@test.com;
该命令撤消来自主机test.com的用户asd对数据库asdtest中的表asdtable的执行SELECT的权限。

		GRANT SELECT(id,name,address,phone),update(address,phone) ON asdtest.asdtable TO asd@test.com;
该命令将授予来自主机test.com的用户asd对数据库asdtest中asdtable表内的id,name,address,phone执行SELECT的权限，以及address和phone字段执行UPDATE操作的权限。

		REVOKE UPDATE(address,phone) ON asdtest.asdtable FROM asd@test.com;
该命令将撤消来自主机test.com的用户asd对数据库asdtest中asdtable表内的address和phone字段执行UPDATE操作的权限。