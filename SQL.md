<!-- GFM-TOC -->
* [一、基础](#一基础)
* [二、数据库的操作](#二数据库的操作)
* [三、表的操作](#三表的操作)
* [四、数据的操作](#四数据的操作)
* [五、数据类型](#五数据类型)
* [六、函数](#六函数)
* [七、计算字段](#七计算字段)
* [八、视图](#八视图)
* [九、存储过程](#九存储过程)
* [十、事务](#十事务)
* [十一、游标](#十一游标)
* [十二、约束](#十二约束)
* [十三、索引](#十三索引)
* [十四、触发器](#十四触发器)
* [十五、权限管理](#十五权限管理)
<!-- GFM-TOC -->

# 一、基础

--基础

		--模式定义了数据如何存储、存储什么样的数据以及数据如何分解等信息，数据库和表都有模式。。
		--SQL（Structured Query Language)，标准 SQL 由 ANSI 标准委员会管理，从而称为 ANSI SQL。各个 DBMS 都有自己的实现，如 PL/SQL、Transact-SQL 等。
		--SQL 语句不区分大小写，但是数据库表名、列名和值是否区分依赖于具体的 DBMS 以及配置。
		--SQL 语句中所有空格都被忽略。

--SQL支持以下三种注释：

		# 注释
		SELECT *
		FROM mytable; -- 注释
		/* 注释1
		   注释2 */

--结束、帮助、退出

		；或 \g  --结束（mysql命令行必须以；结尾，MySQL中不需要在单条SQL语句后面加；）
		help 或 \h  --帮助
		quit 或 exit --退出

--显示

		SHOW DATABASES;--显示数据库
		SHOW TABLES;--显示当前选择数据库内容中的所有可用表
		SHOW STATUS;--显示广泛的服务器状态信息
		SHOW CREATE DATABASE;--显示创建特定数据库
		SHOW CREATE TABLE;--显示创建特定表
		SHOW GRANTS;--显示授予用户（所有用户或特定用户）的安全权限
		SHOW ERRORS;--显示服务器错误
		SHOW WARNINGS;--显示警告信息
		help show;--其他

# 二、数据库的操作

--查看当前数据库
    select database();

--显示当前时间、用户名、数据库版本
    select now(), user(), version();

--创建库

    create database[ if not exists] 数据库名 数据库选项
    数据库选项：
        CHARACTER SET charset_name
        COLLATE collation_name

--查看已有库
    show databases[ like 'pattern']

--使用某一数据库
	use 数据库名

--查看当前库信息
    show create database 数据库名

--修改库的选项信息
    alter database 库名 选项信息

--删除库
    drop database[ if exists] 数据库名
        同时删除该数据库相关的目录及其目录内容

		/* 字符集编码 */ ------------------
		-- MySQL、数据库、表、字段均可设置编码
		-- 数据编码与客户端编码不需一致
		SHOW VARIABLES LIKE 'character_set_%'    -- 查看所有字符集编码项
		    character_set_client        客户端向服务器发送数据时使用的编码
		    character_set_results        服务器端将结果返回给客户端所使用的编码
		    character_set_connection    连接层编码
		SET 变量名 = 变量值
		    set character_set_client = gbk;
		    set character_set_results = gbk;
		    set character_set_connection = gbk;
		SET NAMES GBK;    -- 相当于完成以上三个设置
		-- 校对集
		    校对集用以排序
		    SHOW CHARACTER SET [LIKE 'pattern']/SHOW CHARSET [LIKE 'pattern']    查看所有字符集
		    SHOW COLLATION [LIKE 'pattern']        查看所有校对集
		    charset 字符集编码        设置字符集编码
		    collate 校对集编码        设置校对集编码


# 三、表的操作

	--创建表

	    create [temporary] table[ if not exists] [库名.]表名 ( 表的结构定义 )[ 表选项]
			--创建新表时，指定的表名必须不存在，否则将出错；
	  		--防止意外覆盖已有表，需手动删除该表，不能使用创建表语句覆盖；
		    --仅想在一个表不存在时创建，表名后面添加：IF NOT EXISTS
	        --每个字段必须有数据类型
	        --最后一个字段后不能有逗号
	        --temporary 临时表，会话结束时表自动消失
	        --对于字段的定义：
	            -- 字段名 数据类型 [NOT NULL | NULL] [DEFAULT default_value] [AUTO_INCREMENT] [UNIQUE [KEY] | [PRIMARY] KEY] [COMMENT 'string']，
					
					NULL （1）NULL值表示无值或缺值，但不是空串（''）；
					     （2）每个表列或是NULL列，或是NOT NULL列，这种状态在创建时由表的定义规定；
					     （3）对于NOT NULL的列，如果试图插入没有值的列，将返回错误，且插入失败
					主键 （1）主键值必须唯一，且只能使用NOT NULL的列（单列主键值唯一，多列主键值组合唯一）；
					     （2）多个列组成的主键，应该以逗号分隔的列表给出各列名  PRIMARY KEY (`id1`,`id2`)
					      (3) 主键可以在创建表时定义，或者在创建表之后定义
					AUTO_INCREMENT  （1）自动增量：本列每当增加一行时自动增量；
									（2）每个表只允许一个AUTO_INCREMENT列，主键或unique，且必须被索引；
									（3）覆盖AUTO_INCREMENT：可以使用INSERT语句指定一个值，只要它是唯一的，后续的增量将开始使用该手工插入的值
									（4）last_insert_id()：返回最后一个AUTO_INCREMENT值
					DEFAULT 值  （1）插入行时没有给出值，则可以使用默认值；
						    	（2）MySql不允许使用函数值作为默认值，它只支持常量
					COMMENT 注释 例：create table tab ( id int ) comment '注释内容';
				

	--表选项

	    --字符集
	        -- CHARSET = charset_name
	        -- 如果表没有设定，则使用数据库字符集
	    --存储引擎
	        -- ENGINE = engine_name    
	        /*表在管理数据时采用的不同的数据结构，结构不同会导致处理方式、提供的特性操作等不同
	        常见的引擎：InnoDB MyISAM Memory/Heap BDB Merge Example CSV MaxDB Archive
	        不同的引擎在保存表的结构和数据时采用不同的方式
	        MyISAM表文件含义：.frm表定义，.MYD表数据，.MYI表索引
	        InnoDB表文件含义：.frm表定义，表空间数据和日志文件
			*/
	        SHOW ENGINES -- 显示存储引擎的状态信息
	        SHOW ENGINE 引擎名 {LOGS|STATUS} -- 显示存储引擎的日志或状态信息
	    --数据文件目录
	        DATA DIRECTORY = '目录'
	    --索引文件目录
	        INDEX DIRECTORY = '目录'
	    --表注释
	        COMMENT = 'string'
	    --分区选项
	        PARTITION BY ... (详细见手册)

	--查看所有表

	    SHOW TABLES[ LIKE 'pattern']
	    SHOW TABLES FROM 表名
	--查看表机构

	    SHOW CREATE TABLE 表名    （信息更详细）
	    DESC 表名 / DESCRIBE 表名 / EXPLAIN 表名 / SHOW COLUMNS FROM 表名 [LIKE 'PATTERN']
	    SHOW TABLE STATUS [FROM db_name] [LIKE 'pattern']

	--修改表

		--单个表进行多次更改
				ALTER TABLE mytable
				语句1，
				语句2，
				语句3；
	    --修改表本身的选项
	        ALTER TABLE 表名 表的选项
	        EG:    ALTER TABLE 表名 ENGINE=MYISAM;
	    --对表进行重命名
	        RENAME TABLE 原表名 TO 新表名			--可对多个表进行重命名
	        RENAME TABLE 原表名 TO 库名.表名    （可将表移动到另一个数据库）
	        -- RENAME可以交换两个表名
	    --修改表的字段机构
	        ALTER TABLE 表名 操作名
	        --操作名
	            ADD[ COLUMN] 字段名        -- 增加字段
	                AFTER 字段名            -- 表示增加在该字段名后面
	                FIRST                -- 表示增加在第一个
	            ADD PRIMARY KEY(字段名)    -- 创建主键
	            ADD UNIQUE [索引名] (字段名)-- 创建唯一索引
	            ADD INDEX [索引名] (字段名)    -- 创建普通索引
	            ADD 
	            DROP[ COLUMN] 字段名        -- 删除字段
	            MODIFY[ COLUMN] 字段名 字段属性        -- 支持对字段属性进行修改，不能修改字段名(所有原有属性也需写上)
	            CHANGE[ COLUMN] 原字段名 新字段名 字段属性        -- 支持对字段名修改
	            DROP PRIMARY KEY    -- 删除主键(删除主键前需删除其AUTO_INCREMENT属性)
	            DROP INDEX 索引名    -- 删除索引
	            DROP FOREIGN KEY 外键    -- 删除外键
	
	--删除表
	    DROP TABLE[ IF EXISTS] 表名 ...

	--清空表数据
	    TRUNCATE [TABLE] 表名

	--复制表结构
	    CREATE TABLE 表名 LIKE 要复制的表名

	--复制表结构和数据
	    CREATE TABLE 表名 [AS] SELECT * FROM 要复制的表名

	--检查表是否有错误
	    CHECK TABLE tbl_name [, tbl_name] ... [option] ...

	--优化表
	    OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...

	--修复表
	    REPAIR [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ... [QUICK] [EXTENDED] [USE_FRM]

	--分析表
	    ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...
	
	CREATE TABLE mytable [IF NOT EXISTS](
	  id INT NOT NULL AUTO_INCREMENT,	
	  col1 INT NOT NULL DEFAULT 1,
	  col2 VARCHAR(45) NULL,
	  col3 DATE NULL,
	  PRIMARY KEY (`id1`)	--指定该表的主键
	)ENGINE=InnoDB;
	
	引擎类型（常见）：（1）InnoDB一个可靠的事务处理引擎，它不支持全文本搜索；
			  （2）MEMORY功能等同于MYISAM，但数据存储在内存（不是磁盘），速度很快，适合临时表；
			  （3）MYISAM性能极高的引擎，支持全文本搜索，不支持事务处理
	外键不能跨引擎：即使用一个引擎的表不能引用具有使用不同引擎的表的外键。
	/* truncate */ ------------------
	TRUNCATE [TABLE] tbl_name
	清空数据
	删除重建表
	区别：
	1，truncate 是删除表再创建，delete 是逐条删除
	2，truncate 重置auto_increment的值。而delete不会
	3，truncate 不知道删除了几条，而delete知道。
	4，当被用于带分区的表时，truncate 会保留分区

# 四、数据的操作

-- 增

    INSERT [LOW_PRIORITY] [INTO] 表名 [(字段列表)] VALUES (值列表)[, (值列表), ...];
		--LOW_PRIORITY降低INSERT语句的优先级
		--主键插入时必须保证插入主键值与已存在的主键值不一样,主键值可以插入NULL，自动增量，一旦主键值重复，后续操作失败。
        --如果要插入的值列表包含所有字段并且顺序一致，则可以省略字段列表。
        --可同时插入多条数据记录！
		--INSERT 语句一般没有输出。
		--值列表中的值可以是表达式，或者默认值values (field_value, 10+10, now()，default)
		--某些字段可以不插入数据，前提是它有默认default或者是自动增量的主键。
        --REPLACE 与 INSERT 完全一样，可互换。
        --INSERT [INTO] 表名 SET 字段名=值[, 字段名=值, ...]

		/* insert */ ------------------
		--插入完整的1行
			省略对列的指定，但值的顺序必须与列顺序一致（不安全，不推荐）
				insert into tbl_name values ();
			省略对列的指定，使用SET
				 insert into tbl_name set field=value,...；
			对列指定插入行,必须一一对应
				insert into tbl_name (列名1,列名2) values (值1,值2);
		--插入多行（单条INSERT语句效率更高）
			多条INSERT语句
				insert into ... values (...);
				insert into ... values (...);
			单条INSERT语句，前提是每个值列表与列名顺序一致
				insert into tbl_name values (值列表1), (值列表2), (值列表3);--无列名
				insert into tbl_name (列名1，列名2) values (值列表1), (值列表2), (值列表3);--有列名
		--插入检索（SELECT）出来的数据
		        insert into tbl_name select ...;
				insert into tbl_name (字段列表，中间用逗号隔开) select 字段1，字段2 FROM tbl;--对应字段名不需要一致，不关心SELECT的字段名，只关心返回值
				CREATE TABLE newtable AS SELECT * FROM mytable;--将一个表的内容插入到一个新表
		- 可以指定在插入的值出现主键（或唯一索引）冲突时，更新其他非主键列的信息。（暂时不懂）
		        insert into tbl_name values/set/select on duplicate key update 字段=值, …;

-- 查
    SELECT [DISTINCT] [表名].字段1[,字段2] FROM [数据库名].表名 [WHERE GROUP BY HAVING ORDER BY LIMIT]

	/* select */ ------------------
	
	select [all|distinct] select_expr from -> where -> group by [合计函数] -> having -> order by -> limit
	
	a. select_expr
	    -- 可以用 * 表示所有字段。
	        select * from tb;
		-- 可来自多个表的多个字段，在多个字段之间必须加逗号：SELECT id1,name From products;
	    -- 可以使用表达式（计算公式、函数调用、字段也是个表达式）
	        select stu, 29+25, now() from tb;
	    -- 可以为每个列使用别名。适用于简化列标识，避免多个列标识符重复。
	        - 使用 as 关键字，也可省略 as.
	        select stu+10 as add10 from tb;
	
	b. from 子句
	    用于标识查询来源。
	    -- 可以为表起别名。使用as关键字。
	        select * from tb1 as tt, tb2 as bb;
	    -- from子句后，可以同时出现多个表。
	        -- 多个表会横向叠加到一起，而数据会形成一个笛卡尔积。
	        select * from tb1, tb2;
	
	c. where 子句
	    -- 从from获得的数据源中进行筛选，执行匹配时默认不区分大小写，将值与串类型进行比较，要用单引号来限定字符串；值与数值比较不需要用单引号；NULL值行不显示。
	    -- 整型1表示真，0表示假。
	    -- 表达式由运算符和运算数组成。
	        -- 运算数：变量（字段）、值、函数返回值
	        -- 运算符：
	            =, <=>, <>（不等于）, !=（不等于）, <=, <, >=, >, !, &&, ||, 
	            in (not) null, (not) between a and b（包括指定的开始值和结束值）, is (not)
				and, or, not, xor（匹配多个WHERE语句，优先级：括号>AND>OR）
				(not) in (逗号分隔的清单)：功能等同于or，但速度比or快，可以包含其他SELECT语句 
	            is/is not 加上ture/false/unknown，检验某个值的真假
				(not) like + 通配符匹配（%-任何字符出现任意次数，但不能匹配用值NULL的行；_-下划线，匹配单个字符而不是多个字符）不要滥用通配符，通配符位于开头处匹配会非常慢。 
	            <=>与<>功能相同，<=>可用于null比较
	
	d. group by 子句, 分组子句
	    group by 字段/别名 [排序方式]
	    分组后会进行排序。升序：ASC，降序：DESC
	    
	    以下[合计函数]需配合 group by 使用：
	    count 返回不同的非NULL值数目    count(*)、count(字段)
	    sum 求和
	    max 求最大值
	    min 求最小值
	    avg 求平均值
	    group_concat 返回带有来自一个组的连接的非NULL值的字符串结果。组内字符串连接。
	
	e. having 子句，条件子句
	    与 where 功能、用法相同，执行时机不同。
	    where 在开始时执行检测数据，对原数据进行过滤，执行行过滤；
	    having 对筛选出的结果再次进行过滤，即在数据分组后进行过滤，执行分组过滤。
	    having 字段必须是查询出来的，where 字段必须是数据表存在的。
	    where 不可以使用字段的别名，having 可以。因为执行WHERE代码时，可能尚未确定列值。
	    where 不可以使用合计函数。一般需用合计函数才会用 having
	    SQL标准要求HAVING必须引用GROUP BY子句中的列或用于合计（聚合）函数中的列。
	
	f. order by 子句，排序子句
	    order by 排序字段/别名 排序方式 [,排序字段/别名 排序方式]...
		用来排序的字段不一定要显示；
	    升序：ASC（默认），降序：DESC
	    支持多个字段的排序字段之间用逗号隔开，优先级由顺序确定。
	
	g. limit 子句，限制结果数量子句
	    仅对处理好的结果进行数量限制。将处理好的结果的看作是一个集合，按照记录出现的顺序，索引从0开始。
	    limit 起始位置, 获取条数
	    省略第一个参数，表示从索引0开始。limit 获取条数
		LIMIT 5：返回不多于5行 LIMIT 3,4：从行3开始的四行  （行数不够时，只返回它能返回的行数）
	
	h. distinct, all 选项
	    distinct 去除重复记录，，只返回不同的值，且一旦作用，将作用检索的所有字段名
	    默认为 all, 全部记录

	i.子查询
		    -- 子查询需用括号包裹。
		-- from型
		    from后要求是一个表，必须给子查询结果取个别名。
		    - 简化每个查询内的条件。
		    - from型需将结果生成一个临时表格，可用以原表的锁定的释放。
		    - 子查询返回一个表，表型子查询。
		    select * from (select * from tb where id>0) as subfrom where id>1;
		-- where型
		    - 子查询返回一个值，标量子查询。
		    - 不需要给子查询取别名。
		    - where子查询内的表，不能直接用以更新。
		    select * from tb where money = (select max(money) from tb);
		    -- 列子查询
		        如果子查询结果返回的是一列。
		        使用 in 或 not in 完成查询
		        exists 和 not exists 条件
		            如果子查询返回数据，则返回1或0。常用于判断条件。
		            select column1 from t1 where exists (select * from t2);
		    -- 行子查询
		        查询条件是一个行。
		        select * from t1 where (id, gender) in (select id, gender from t2);
		        行构造符：(col1, col2, ...) 或 ROW(col1, col2, ...)
		        行构造符通常用于与对能返回两个或两个以上列的子查询进行比较。
		
		    -- 特殊运算符
		    != all()    相当于 not in
		    = some()    相当于 in。any 是 some 的别名
		    != some()    不等同于 not in，不等于其中某一个。
		    all, some 可以配合其他运算符一起使用。

	j. 联结
		--数据存储在多个表中，联结多个表，返回一组输出；它不是物理实体，在实际数据库表中不存在
		--连接可以替换子查询，并且比子查询的效率一般会更快。
		--可以用 AS 给列名、计算字段和表名取别名，给表名取别名是为了简化 SQL 语句以及连接相同表，表别名只在查询执行中使用。与列别名不一样，表别名不返回到客户端；Oracle中没有AS。
	--	--内联结（内连接又称等值连接，使用 INNER JOIN 关键字，默认就是内连接，可省略inner）
			--使用INNER JOIN
				SELECT A.value, B.value
				FROM [tablea AS] A INNER JOIN [tablea AS] B
				ON A.key = B.key;--on代替where给出条件语句
			--可以不明确使用 INNER JOIN，而使用普通查询并在 WHERE 中将两个表中要连接的列用等值方法连接起来。
				SELECT A.value, B.value
				FROM [tablea AS] A, [tablea AS] B
				WHERE A.key = B.key;
			--在没有条件语句的情况下返回笛卡尔积（也称叉联结）。
			--不限制联结表的数目（联结的表越多，性能越差）
				SELECT prod_name, vend_name, prod_price, quantity
				FROM OrderItems, Products, Vendors
				WHERE Products.vend_id = Vendors.vend_id
				AND OrderItems.prod_id = Products.prod_id
				AND order_num = 20007;
	--  --自联结
			--自连接可以看成内连接的一种，只是连接的表是自身而已。
			--on 可以换成 using
			--一张员工表，包含员工姓名和员工所属部门，要找出与 Jim 处在同一部门的所有员工姓名。
					子查询版本
						SELECT name
						FROM employee
						WHERE department = (
						      SELECT department
						      FROM employee
						      WHERE name = "Jim");
					自连接版本
						SELECT e1.name
						FROM employee AS e1 INNER JOIN employee AS e2  --对同一表用两个别名，消除歧义
						ON e1.department = e2.department
						      AND e2.name = "Jim";
	--  --自然连接
			--内连接和自然连接的区别：内连接提供连接的列，而自然连接自动连接所有同名列。
			--关键字NATURAL JOIN
				SELECT A.value, B.value
				FROM tablea AS A NATURAL JOIN tableb AS B;
	--  --外连接（outer可以省略）
			--外连接保留了没有关联的那些行,数据不存在也会出现在联结结果中。分为左外连接，右外连接以及全外连接，左外连接就是保留左表没有关联的行。
			-- 左外连接 left outer join
 				如果数据不存在，左表记录会出现，而右表为null填充
    		-- 右外连接 right outer join
        		如果数据不存在，右表记录会出现，而左表为null填充
			-- 全外联结（full outer join）
				检索两个表中的所有行并关联那些可以关联的行

	k. 组合查询（复合查询）UNION
			--任何具有多个WHERE子句的SELECT语句都可以作为一个组合查询（where完成不了UNION ALL的工作）
			--每个查询必须包含相同的列、表达式和聚集函数（查询目标一致）。
			--SELECT ... UNION [ALL|DISTINCT] SELECT ...默认 DISTINCT 方式，即所有返回的行都是唯一的，消除了重复行
			--只能包含一个 ORDER BY 子句，并且必须位于语句的最后。
			--建议，对每个SELECT查询加上小括号包裹。

-- 删

    DELETE [LOW_PRIORITY] FROM 表名[ 删除条件子句]
    -- 没有条件子句，则会删除表中所有行，但是不删除表本身
	-- TRUNCATE TABLE mytable;可以清空表，也就是删除所有行。

-- 改

    UPDATE [LOW_PRIORITY] 表名 SET 字段名=新值[, 字段名=新值] [更新条件]
	-- 更新条件：如果没有更新条件，那么所有行将被更新
	-- 用检索（SELECT）出来的数据更新（不懂）
	-- 更新出现错误时，整个更新操作取消；如果想即使出现错误，也继续更新，则使用IGNORE关键字：UPDATE IGNORE 表名 ...
	-- 删除某个列的值，可以设置它为NULL.

# 五、数据类型
-- 数值类型

	--整型

	    类型            字节        范围（有符号位）
	    tinyint        1字节    -128 ~ 127        无符号位：0 ~ 255
	    smallint    2字节    -32768 ~ 32767
	    mediumint    3字节    -8388608 ~ 8388607
	    int            4字节
	    bigint        8字节

    	int(M)    M表示总位数
    		--默认存在符号位，unsigned 属性修改
		    --显示宽度，如果某个数不够定义字段时设置的位数，则前面以0补填，zerofill 属性修改 例：int(5)    插入一个数'123'，补填后为'00123'
    		--在满足要求的情况下，越小越好。
    		--1表示bool值真，0表示bool值假。MySQL没有布尔类型，通过整型0和1表示。常用tinyint(1)表示布尔型。

	--浮点型

	    类型                字节        范围
	    float(单精度)        4字节
	    double(双精度)    8字节
	    浮点型既支持符号位 unsigned 属性，也支持显示宽度 zerofill 属性。
	        不同于整型，前后均会补填0.
	    定义浮点型时，需指定总位数和小数位数。
	        float(M, D)        double(M, D)
	        M表示总位数，D表示小数位数。
	        M和D的大小会决定浮点数的范围。不同于整型的固定范围。
	        M既表示总位数（不包括小数点和正负号），也表示显示宽度（所有显示符号均包括）。
	        支持科学计数法表示。
	        浮点数表示近似值。

	--定点数

	    decimal    -- 可变长度
	    decimal(M, D)    M也表示总位数，D表示小数位数。
	    保存一个精确的数值，不会发生数据的改变，不同于浮点数的四舍五入。
	    将浮点数转换为字符串来保存，每9位数字保存为4个字节。

--字符串类型

	--char, varchar
	    char    定长字符串，速度快，但浪费空间
	    varchar    变长字符串，速度慢，但节省空间
	    M表示能存储的最大长度，此长度是字符数，非字节数。
	    不同的编码，所占用的空间不同。
	    char,最多255个字符，与编码无关。
	    varchar,最多65535字符，与编码有关。
	    一条有效记录最大不能超过65535个字节。
	        utf8 最大为21844个字符，gbk 最大为32766个字符，latin1 最大为65532个字符
	    varchar 是变长的，需要利用存储空间保存 varchar 的长度，如果数据小于255个字节，则采用一个字节来保存长度，反之需要两个字节来保存。
	    varchar 的最大有效长度由最大行大小和使用的字符集确定。
	    最大有效长度是65532字节，因为在varchar存字符串时，第一个字节是空的，不存在任何数据，然后还需两个字节来存放字符串的长度，所以有效长度是64432-1-2=65532字节。
	    例：若一个表定义为 CREATE TABLE tb(c1 int, c2 char(30), c3 varchar(N)) charset=utf8; 问N的最大值是多少？ 答：(65535-1-2-4-30*3)/3

	--blob, text
	    blob 二进制字符串（字节字符串）
	        tinyblob, blob, mediumblob, longblob
	    text 非二进制字符串（字符字符串）
	        tinytext, text, mediumtext, longtext
	    text 在定义时，不需要定义长度，也不会计算总长度。
	    text 类型在定义时，不可给default值

	--binary, varbinary
	    类似于char和varchar，用于保存二进制字符串，也就是保存字节字符串而非字符字符串。
	    char, varchar, text 对应 binary, varbinary, blob.

--日期时间类型

    --一般用整型保存时间戳，因为PHP可以很方便的将时间戳进行格式化。
	    datetime    8字节    日期及时间        1000-01-01 00:00:00 到 9999-12-31 23:59:59
	    date        3字节    日期            1000-01-01 到 9999-12-31
	    timestamp    4字节    时间戳        19700101000000 到 2038-01-19 03:14:07
	    time        3字节    时间            -838:59:59 到 838:59:59
	    year        1字节    年份            1901 - 2155
    
		datetime    “YYYY-MM-DD hh:mm:ss”
		timestamp    “YY-MM-DD hh:mm:ss”
		            “YYYYMMDDhhmmss”
		            “YYMMDDhhmmss”
		            YYYYMMDDhhmmss
		            YYMMDDhhmmss
		date        “YYYY-MM-DD”
		            “YY-MM-DD”
		            “YYYYMMDD”
		            “YYMMDD”
		            YYYYMMDD
		            YYMMDD
		time        “hh:mm:ss”
		            “hhmmss”
		            hhmmss
		year        “YYYY”
		            “YY”
		            YYYY
		            YY

--枚举和集合

	--枚举(enum)
		enum(val1, val2, val3...)
		    在已知的值中进行单选。最大数量为65535.
		    枚举值在保存时，以2个字节的整型(smallint)保存。每个枚举值，按保存的位置顺序，从1开始逐一递增。
		    表现为字符串类型，存储却是整型。
		    NULL值的索引是NULL。
		    空字符串错误值的索引值是0。

	--集合（set）
		set(val1, val2, val3...)
		    create table tab ( gender set('男', '女', '无') );
		    insert into tab values ('男, 女');
		    最多可以有64个不同的成员。以bigint存储，共8个字节。采取位运算的形式。
		    当创建表时，SET成员值的尾部空格将自动被删除。

--选择类型 

	-- PHP角度
		功能满足
		存储空间尽量小，处理效率更高
		考虑兼容问题

	--IP存储 
		只需存储，可用字符串
		如果需计算，查找等，可存储为4个字节的无符号int，即unsigned
		    1) PHP函数转换
		        ip2long可转换为整型，但会出现携带符号问题。需格式化为无符号的整型。
		        利用sprintf函数格式化字符串
		        sprintf("%u", ip2long('192.168.3.134'));
		        然后用long2ip将整型转回IP字符串
		    2) MySQL函数转换(无符号整型，UNSIGNED)
		        INET_ATON('127.0.0.1') 将IP转为整型
		        INET_NTOA(2130706433) 将整型转为IP


#六、函数
-- 内置函数

	-- 数值函数
		abs(x); 			-- 绝对值 abs(-10.9) = 10
		format(x, d);    	-- 格式化千分位数值 format(1234567.456, 2) = 1,234,567.46
		ceil(x);           	-- 向上取整 ceil(10.1) = 11
		floor(x);        	-- 向下取整 floor (10.1) = 10
		round(x);        	-- 四舍五入去整
		mod(m, n);        	-- m%n m mod n 求余 10%3=1
		pi();           	-- 获得圆周率
		pow(m, n);       	-- m^n
		sqrt(x);            -- 算术平方根
		rand();            	-- 随机数
		truncate(x, d);    	-- 截取d位小数
		exp();cos();sin();tan();--指数、余弦、正弦、正切

	-- 时间日期函数
		--日期格式：yyyy-mm-dd 00:00:00
		now(), current_timestamp();     -- 当前日期和时间
		curdate(), current_date();                    -- 当前日期
		curtime(), current_time();                    -- 当前时间
		day();		-- 获取天部分		dayofweek();	--对于一个日期，返回对应的星期几
		year();	--返回一个日期的年份部分	date();    -- 获取日期部分	time();    -- 获取时间部分
		hour(); minute(); second(), month();	--返回一个时间的小时、分钟、秒、月
		date_format('yyyy-mm-dd hh:ii:ss', '%d %y %a %d %m %b %j');    -- 格式化时间
		unix_timestamp();                -- 获得unix时间戳
		from_unixtime();                -- 从时间戳获得时间
		adddata();	--增加一个日期（天、周）
		addtime();	--增加一个时间（时、分）

	-- 字符串函数（可以直接操作字段，对整个字段列的串整体操作）
		length(string)            			-- string长度，字节
		char_length(string)        			-- string的字符个数
		substring(str, position [,length])  -- 从str的position开始,取length个字符
		replace(str ,search_str ,replace_str)    -- 在str中用replace_str替换search_str
		instr(string ,substring)    		-- 返回substring首次在string中出现的位置
		concat(string [,...])    			-- 连接字串
		charset(str)            			-- 返回字串字符集
		lcase(string)            			-- 转换成小写
		left(string, length)    			-- 从string2中的左边起取length个字符
		load_file(file_name)    			-- 从文件读取内容
		locate(substring, string [,start_position])    -- 同instr,但可指定开始位置
		lpad(string, length, pad)    		-- 重复用pad加在string开头,直到字串长度为length
		repeat(string, count)    			-- 重复count次
		rpad(string, length, pad)    		--在str后用pad补充,直到长度为length
		ltrim(string)            			-- 去除前端空格	
		rtrim(string)            			-- 去除后端空格
		trim(string)						--去掉串左右两端的空格
		strcmp(string1 ,string2)    		-- 逐字符比较两字串大小
		lower(string)	--将串转换为小写
		upper(string)	--将文本转换为大写
		soundex(string)	-- 返回该串的发音
		right(string)	--返回串右边的字符
		left(string)	--返回串左边的字符

	-- 流程函数
		case when [condition] then result [when [condition] then result ...] [else result] end   多分支 if(expr1,expr2,expr3)  双分支。

	-- 聚合函数（用于汇总数据的函数）

		--所有聚合函数都可以用来执行多个列上的计算（通过标准的算术操作符）
		--可以在列名前面添加DISTINCT:必须使用列名，不能用于计算或表达式；不用于count(*)count(DISTINCT)
		--组合聚合函数：包含多个聚合函数的语句，语句之间用逗号隔开
			count()；--count(*)对表中所有行进行计数，包括NULL行；count(column)对column列进行行数计数，忽略NULL值
			sum();--忽略值为NULL的行
			max();--要求指定列名（最好是数值排序，非数值也可以）；忽略值为NULL的行
			min();--要求指定列名（最好是数值排序，非数值也可以）；忽略值为NULL的行
			avg();--只能用于确定单个数值列的平均值，获取多个列的平均值，使用多个avg函数；忽略值为NULL的行
			group_concat()

	-- 其他常用函数
			md5();
			default();

--存储函数，自定义函数

	-- 新建
	    CREATE FUNCTION function_name (参数列表) RETURNS 返回值类型
	        函数体
		    --函数名，应该合法的标识符，并且不应该与已有的关键字冲突。
		    --一个函数应该属于某个数据库，可以使用db_name.funciton_name的形式执行当前函数所属数据库，否则为当前数据库。
		    --参数部分，由"参数名"和"参数类型"组成。多个参数用逗号隔开。
		    --函数体由多条可用的mysql语句，流程控制，变量声明等语句构成。
		    --多条语句应该使用 begin...end 语句块包含。
		    --一定要有 return 返回值语句。

	-- 删除
	    DROP FUNCTION [IF EXISTS] function_name;

	-- 查看
	    SHOW FUNCTION STATUS LIKE 'partten'
	    SHOW CREATE FUNCTION function_name;

	-- 修改
	    ALTER FUNCTION function_name 函数选项

        
# 七、计算字段
	--在数据库服务器上完成数据的转换和格式化的工作往往比客户端上快得多，并且转换和格式化后的数据量更少的话可以减少网络通信量。
	--计算字段通常需要使用AS来取别名，否则输出的时候字段名为计算表达式。
		```sql
		SELECT col1 * col2 AS alias
		FROM mytable;
		```
	--CONCAT()用于连接两个字段。许多数据库会使用空格把一个值填充为列宽，因此连接的结果会出现一些不必要的空格，使用TRIM()可以去除首尾空格。
		```sql
		SELECT CONCAT(TRIM(col1), '(', TRIM(col2), ')') AS concat_col
		FROM mytable;
		```


# 八、视图
	-- 什么是视图：
		    视图是一个虚拟表，其内容由查询定义。同真实的表一样，视图包含一系列带有名称的列和行数据。但是，视图并不在数据库中以存储的数据值集形式存在。行和列数据来自由定义视图的查询所引用的表，并且在引用视图时动态生成。
		    视图具有表结构文件，但不存在数据文件，也就不能对其进行索引操作。
		    对其中所引用的基础表来说，视图的作用类似于筛选。定义视图的筛选可以来自当前或其它数据库的一个或多个表，或者其它视图。通过视图进行查询没有任何限制，通过它们进行数据修改时的限制也很少。对视图的操作和对普通表的操作一样。
			可创建的视图数目没有限制。
			视图可以嵌套 。
	-- 视图具有如下好处：
			简化复杂的 SQL 操作，比如复杂的连接，简化业务逻辑；
			只使用实际表的一部分数据；
			通过只给用户访问视图的权限，对客户端隐藏真实的表结构，保证数据的安全性；
			更改数据格式和表示。
	-- 创建视图
			CREATE [OR REPLACE] [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}] VIEW view_name [(column_list)] AS select_statement
			--select语句中，select的约束全部适用。
    		--视图名必须唯一，同时不能与表重名。
    		--视图可以使用select语句查询到的列名，也可以自己指定相应的列名。
    		--可以指定视图执行的算法，通过ALGORITHM指定。
    		--column_list如果存在，则数目必须等于SELECT语句检索的列数
				CREATE VIEW myview AS
				SELECT Concat(col1, col2) AS concat_col, col3*col4 AS compute_col
				FROM mytable
				WHERE col5 = val;
	-- 查看结构
    		SHOW CREATE VIEW view_name 
	-- 删除视图
    		--删除视图后，数据依然存在。
    		--可同时删除多个视图。
    			DROP VIEW [IF EXISTS] view_name ...
	-- 修改视图结构
		--一般覆盖更新视图，先删除，然后重新创建。
    	--一般不修改视图，因为不是所有的更新视图都会映射到表上。
    		ALTER VIEW view_name [(column_list)] AS select_statement
	-- 视图算法(ALGORITHM)
	    MERGE        合并
	        将视图的查询语句，与外部查询需要先合并再执行！
	    TEMPTABLE    临时表
	        将视图执行完毕后，形成临时表，再做外层查询！
	    UNDEFINED    未定义(默认)，指的是MySQL自主去选择相应的算法。

# 九、存储过程
	--存储过程可以看成是对一系列 SQL 操作的批处理；
	--使用存储过程的好处（简单、安全、高性能）：
		代码封装，简化操作、保证了一定的安全性；
		代码复用；
		由于是预先编译，因此具有很高的性能。
	--存储过程的创建
		--命令行中创建存储过程需要自定义分隔符，因为命令行是以 ; 为结束符，而存储过程中也包含了分号，因此会错误把这部分分号当成是结束符，造成语法错误。
			CREATE PROCEDURE 过程名 (参数列表)
			BEGIN
			    过程体
			END
		--参数列表：不同于函数的参数列表，需要指明参数类型
					IN，表示输入型，在调用过程中，将数据输入到过程体内部的参数
					OUT，表示输出型，在调用过程中，将过程体处理完的结果返回到客户端
					INOUT，表示混合型，既可以输入，也可以输出
		--注意，没有返回值。
		--四大类
			--无参数存储过程
					delimiter // --自定义分隔符
					create procedure myFist_proc() ## 创建存储过程
					begin 
					select stu_id from score where grade>80 and c_name='计算机';
					select name from student where id in 
					(select stu_id from(select * from score where grade>80 and c_name='计算机')a 
					where a.stu_id in(select stu_id from score where grade>80 and c_name='英语'));
					end;//
					delimiter;
					show create procedure myFist_proc();
					call myFist_proc();
		--带有输出参数的存储过程
					delimiter //
					create procedure mySecond_proc(out sumStudent int ) ## 创建存储过程
					begin 
					select count(*) into sumStudent from score where grade>80;
					end;//
					delimiter;
					call mySecond_proc(@sumStudent);
					select @sumStudent;
		--带有输入参数的存储过程
					delimiter //
					create procedure myThird_proc(in minScore int ) ## 创建存储过程
					begin 
					select count(*) from score where grade>minScore;
					end;//
					delimiter;
					call myThird_proc(90); 大于90的
					select @sumStudent;
		--带有输入输出参数的存储过程
					delimiter //
					create procedure myFourth_proc(in minScore int,out sumStudent int ) ## 创建存储过程
					begin 
					select count(*) into sumStudent from score where grade>minScore;
					end;//
					delimiter;
					call myFourth_proc(90,@sumStudent);
					select @sumStudent;
	--存储过程的执行
		EXECUTE 存储过程名(对应参数传递);
			--根据DBMS具体实现，可能具有以下功能
			--参数可选，具有不提供参数时的默认值；
			--不按次序给出参数，以“参数=值”的方式给出参数值。
			--输出参数，允许存储过程在正执行的应用程序中更新所用的参数。
			--用SELECT语句检索数据。
			--返回代码，允许存储过程返回一个值到正在执行的应用程序。
	--存储过程的调用
			CALL 过程名
			-- 注意
			- 没有返回值。
			- 只能单独调用，不可夹杂在其他语句中
	--存储过程结果赋值
			--给变量赋值都需要用 select into 语句。
			--每次只能给一个变量赋值，不支持集合的操作。
	--存储过程与函数的区别
			--而一个函数通常专注与某个功能，视为其他程序服务的，需要在其他语句中调用函数才可以，而存储过程不能被其他调用，是自己执行 通过call执行。

# 十、事务
	--基本术语：
		--事务（transaction）指一组 SQL 语句；
		--回退（rollback）指撤销指定 SQL 语句的过程；
		--提交（commit）指将未存储的 SQL 语句结果写入数据库表；
		--保留点（savepoint）指事务处理中设置的临时占位符（placeholder），你可以对它发布回退（与回退整个事务处理不同）。
	--作用
		--事务是指逻辑上的一组操作，组成这组操作的各个单元，要不全成功要不全失败。 
	    --支持连续SQL的集体成功或集体撤销。
	    --事务是数据库在数据晚自习方面的一个功能。
	    --需要利用 InnoDB 或 BDB 存储引擎，对自动提交的特性支持完成。
	   	--InnoDB被称为事务安全型引擎。
	--注意点
		--事务处理用来管理INSERT、UPDA TE和DELETE语句。
		--不能回退 SELECT 语句，回退 SELECT 语句也没意义；也不能回退 CREATE 和 DROP 语句，事务处理中可以使用这些语句，但回退时不可以撤销。
	    --数据定义语言（DDL）语句不能被回滚，比如创建或取消数据库的语句，和创建、取消或更改表或存储的子程序的语句。
	    --事务不能被嵌套
	-- 事务的特性
	    --原子性（Atomicity）：事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。
	    --一致性（Consistency）：事务前后数据的完整性必须保持一致。
	        --事务开始和结束时，外部数据一致
	        --在整个事务过程中，操作是连续的
    	--隔离性（Isolation）：多个用户并发访问数据库时，一个用户的事务不能被其它用户的事物所干扰，多个并发事务之间的数据要相互隔离。
    	--持久性（Durability)：一个事务一旦被提交，它对数据库中的数据改变就是永久性的。
	-- 事务的原理
	    --利用InnoDB的自动提交(autocommit)特性完成。
		--MySQL 的事务提交默认是隐式提交，每执行一条语句就把这条语句当成一个事务然后进行提交。
		--当出现 START TRANSACTION 语句时，会关闭隐式提交；当 COMMIT 或 ROLLBACK 语句执行后，事务会自动关闭，重新恢复隐式提交。
	    --普通的MySQL执行语句后，当前的数据提交操作均可被其他客户端可见。
	    --而事务是暂时关闭“自动提交”机制，需要commit提交持久化数据操作。
		--InnoDB自动提交特性设置
    		SET autocommit = 0|1;    0表示关闭自动提交，1表示开启自动提交。
			    --如果关闭了，那普通操作的结果对其他客户端也不可见，需要commit提交后才能持久化数据操作。
			    --也可以关闭自动提交来开启事务。但与START TRANSACTION不同的是，
        	SET autocommit；--是永久改变服务器的设置，直到下次再次修改该设置。(针对当前连接)
        	START TRANSACTION;--记录开启前的状态，而一旦事务提交或回滚后就需要再次开启事务。(针对当前事务)
	--事务开启
	    START TRANSACTION; 或者 BEGIN;
	    开启事务后，所有被执行的SQL语句均被认作当前事务内的SQL语句，要么完全执行，要么完全不执行。
	--事务提交
	    COMMIT;--回退整个事务
	--事务回滚
	    ROLLBACK;--回退整个事务；如果部分操作发生问题，映射到事务开启前。
	-- 保存点，回退部分事务
    	SAVEPOINT 保存点名称 -- 设置一个事务保存点
    	ROLLBACK TO SAVEPOINT 保存点名称 -- 回滚到保存点
    	RELEASE SAVEPOINT 保存点名称 -- 删除保存点

# 十一、游标
	--在存储过程中使用游标可以对一个结果集进行移动遍历。
	--游标主要用于交互式应用，其中用户需要对数据集中的任意行进行浏览和修改。
	--使用游标的四个步骤：
		 声明游标，这个过程没有实际检索出数据；
		 打开游标；
		 取出数据；
		 关闭游标；

	```sql
	delimiter //
	create procedure myprocedure(out ret int)
	    begin
	        declare done boolean default 0;
	
	        DECLARE mycursor CURSOR FOR
	        SELECT col1 from mytabl;--定义游标
	        declare continue handler for sqlstate '02000' set done = 1; -- 定义了一个 continue handler，当 sqlstate '02000' 这个条件出现时，会执行 set done = 1
	
	        OPEN CURSOR mycursor;--打开游标
	
	        repeat
	            fetch mycursor into ret;--使用游标
	            select ret;--显示游标查询的值
	        until done end repeat;
	
	        CLOSE mycursor;--关闭游标
	    end //
	 delimiter ;
	```

# 十二、约束
	--约束的定义：管理如何插入或处理数据库数据的规则。
	--主键约束
		--任意两行的主键值都不相同。
		--每行都具有一个主键值（即列中不允许NULL值）。
		--包含主键值的列从不修改或更新。
		--主键值不能重用。如果从表中删除某一行，其主键值不分配给新行。
	--外键约束
	    --用于限制主表与从表数据完整性。
			cust_id CHAR(10) NOT NULL REFERENCES Customers(cust_id);--表中定义时,用references,表示cust_id中的任何值都必须是Customers表的cust_id中的值
    		alter table t1 add constraint [`t1_t2_fk`] foreign key (t1_id) references t2(id);--将表t1的t1_id外键关联到表t2的id字段。
        -- 每个外键都有一个名字，可以通过 constraint 指定
		-- 存在外键的表，称之为从表（子表），外键指向的表，称之为主表（父表）。
		-- 作用：保持数据一致性，完整性，主要目的是控制存储在外键表（从表）中的数据。

		    MySQL中，可以对InnoDB引擎使用外键约束：
		    语法：
		    foreign key (外键字段） references 主表名 (关联字段) [主表记录删除时的动作] [主表记录更新时的动作]
		    此时需要检测一个从表的外键需要约束为主表的已存在的值。外键在没有关联的情况下，可以设置为null.前提是该外键列，没有not null。
		
		    可以不指定主表记录更改或更新时的动作，那么此时主表的操作被拒绝。
		    如果指定了 on update 或 on delete：在删除或更新时，有如下几个操作可以选择：
		    1. cascade，级联操作。主表数据被更新（主键值更新），从表也被更新（外键值更新）。主表记录被删除，从表相关记录也被删除。
		    2. set null，设置为null。主表数据被更新（主键值更新），从表的外键被设置为null。主表记录被删除，从表相关记录外键被设置成null。但注意，要求该外键列，没有not null属性约束。
		    3. restrict，拒绝父表删除和更新。

    		注意，外键只被InnoDB存储引擎所支持。其他引擎是不支持的。
	-- 唯一约束
		-- 唯一约束用来保证一列（或一组列）中的数据是唯一的。它们类似于主键，但存在以下重要区别
			--表可包含多个唯一约束，但每个表只允许一个主键。
			--唯一约束列可包含NULL值。
			--唯一约束列可修改或更新。
			--唯一约束列的值可重复使用。
			--与主键不一样，唯一约束不能用来定义外键。
		-- 唯一约束的语法类似于其他约束的语法。唯一约束既可以用UNIQUE关键字在表定义中定义，也可以用单独的CONSTRA INT定义。
	-- 检查约束
		-- 检查约束用来保证一列（或一组列）中的数据满足一组指定的条件。检查约束的常见用途有以下几点
			--检查最小或最大值。例如，防止0个物品的订单（即使0是合法的数）。
			--指定范围。例如，保证发货日期大于等于今天的日期，但不超过今天起一年后的日期。
			--只允许特定的值。例如，在性别字段中只允许M或F。
		-- 使用
			quantity INTEGER NOT NULL CHECK (quantity > 0);--使用CHECK关键字
			ADD CONSTRAINT CHECK (quantity > 0);

# 十三、索引
	--注意
		--索引改善检索操作的性能，但降低了数据插入、修改和删除的性能。在执行这些操作时，DBMS必须动态地更新索引。
		--索引数据可能要占用大量的存储空间。
		--并非所有数据都适合做索引。取值不多的数据（如州）不如具有更多可能值的数据（如姓或名），能通过索引得到那么多的好处。
		--索引用于数据过滤和数据排序。如果你经常以某种特定的顺序排序数据，则该数据可能适合做索引。
		--可以在索引中定义多个列（例如，州加上城市）。这样的索引仅在以州加城市的顺序排序时有用。如果想按城市排序，则这种索引没有用处。
	--索引的创建
		CREATE INDEX prod_name_ind ON PRODUCTS (prod_name);--索引用CREA TE INDEX语句创建，ON用来指定被索引的表，而索引中包含的列（此例中仅有一列）在表名后的圆括号中给出。

# 十四、触发器
    --触发程序是与表有关的命名数据库对象，当该表出现特定事件时，将激活该对象
    监听：记录的增加、修改、删除。
	--触发器内的代码具有以下数据的访问权：
		INSERT操作中的所有新数据；
		UPDA TE操作中的所有新数据和旧数据；
		DELETE操作中删除的数据。
	-- 创建触发器
		CREATE TRIGGER trigger_name trigger_time trigger_event ON tbl_name FOR EACH ROW trigger_stmt
		    参数：
		    trigger_time是触发程序的动作时间。它可以是 before 或 after，以指明触发程序是在激活它的语句之前或之后触发。
		    trigger_event指明了激活触发程序的语句的类型
		        INSERT：将新行插入表时激活触发程序
		        UPDATE：更改某一行时激活触发程序
		        DELETE：从表中删除某一行时激活触发程序
		    tbl_name：监听的表，必须是永久性的表，不能将触发程序与TEMPORARY表或视图关联起来。
		    trigger_stmt：当触发程序激活时执行的语句。执行多个语句，可使用BEGIN...END复合语句结构	
	-- 删除
		DROP TRIGGER [schema_name.]trigger_name
	-- 注意
    	--对于具有相同触发程序动作时间和事件的给定表，不能有两个触发程序。
		--触发器会在某个表执行以下语句时而自动执行：DELETE、INSERT、UPDATE。
		--触发器必须指定在语句执行之前还是之后自动执行，之前执行使用 BEFORE 关键字，之后执行使用 AFTER 关键字。BEFORE 用于数据验证和净化，AFTER 用于审计跟踪，将修改记录到另外一张表中。
		--INSERT 触发器包含一个名为 NEW 的虚拟表。
		--DELETE 触发器包含一个名为 OLD 的虚拟表，并且是只读的。
		--UPDATE 触发器包含一个名为 NEW 和一个名为 OLD 的虚拟表，其中 NEW 是可以被修改地，而 OLD 是只读的。
		--MySQL 不允许在触发器中使用 CALL 语句，也就是不能调用存储过程。

# 十五、权限管理

--新创建的账户没有任何权限。
--账户用 username@host 的形式定义，username@% 使用的是默认主机名。
--MySQL 的账户信息保存在 mysql 这个数据库中。

		USE mysql;
		SELECT user FROM user;

-- 刷新权限

	FLUSH PRIVILEGES

-- 增加用户

	CREATE USER 用户名 IDENTIFIED BY [PASSWORD] 密码(字符串)
	    --必须拥有mysql数据库的全局CREATE USER权限，或拥有INSERT权限。
	    --只能创建用户，不能赋予权限。
	    --用户名，注意引号：如 'user_name'@'192.168.1.1'
	    --密码也需引号，纯数字密码也要加引号
	    --要在纯文本中指定密码，需忽略PASSWORD关键词。要把密码指定为由PASSWORD()函数返回的混编值，需包含关键字PASSWORD

-- 重命名用户

		RENAME USER old_user TO new_user

-- 设置密码

		SET PASSWORD = PASSWORD('密码')    -- 为当前用户设置密码
		SET PASSWORD FOR 用户名 = PASSWORD('密码')    -- 为指定用户设置密码

-- 删除用户

		DROP USER 用户名

-- 分配权限/添加用户

		GRANT 权限列表 ON 表名 TO 用户名 [IDENTIFIED BY [PASSWORD] 'password']
		    --all privileges 表示所有权限
		    --*.* 表示所有库的所有表
		    --库名.表名 表示某库下面的某表

-- 查看权限

		SHOW GRANTS FOR 用户名

-- 查看当前用户权限

    	SHOW GRANTS; 或 SHOW GRANTS FOR CURRENT_USER; 或 SHOW GRANTS FOR CURRENT_USER();

-- 撤消权限

		REVOKE 权限列表 ON 表名 FROM 用户名
		REVOKE ALL PRIVILEGES, GRANT OPTION FROM 用户名    -- 撤销所有权限

-- 权限层级

		-- 要使用GRANT或REVOKE，您必须拥有GRANT OPTION权限，并且您必须用于您正在授予或撤销的权限。
		全局层级：全局权限适用于一个给定服务器中的所有数据库，mysql.user
		    GRANT ALL ON *.*和 REVOKE ALL ON *.*只授予和撤销全局权限。
		数据库层级：数据库权限适用于一个给定数据库中的所有目标，mysql.db, mysql.host
		    GRANT ALL ON db_name.*和REVOKE ALL ON db_name.*只授予和撤销数据库权限。
		表层级：表权限适用于一个给定表中的所有列，mysql.talbes_priv
		    GRANT ALL ON db_name.tbl_name和REVOKE ALL ON db_name.tbl_name只授予和撤销表权限。
		列层级：列权限适用于一个给定表中的单一列，mysql.columns_priv
		    当使用REVOKE时，您必须指定与被授权列相同的列。

-- 权限列表

		ALL [PRIVILEGES]    -- 设置除GRANT OPTION之外的所有简单权限
		ALTER    -- 允许使用ALTER TABLE
		ALTER ROUTINE    -- 更改或取消已存储的子程序
		CREATE    -- 允许使用CREATE TABLE
		CREATE ROUTINE    -- 创建已存储的子程序
		CREATE TEMPORARY TABLES        -- 允许使用CREATE TEMPORARY TABLE
		CREATE USER        -- 允许使用CREATE USER, DROP USER, RENAME USER和REVOKE ALL PRIVILEGES。
		CREATE VIEW        -- 允许使用CREATE VIEW
		DELETE    -- 允许使用DELETE
		DROP    -- 允许使用DROP TABLE
		EXECUTE        -- 允许用户运行已存储的子程序
		FILE    -- 允许使用SELECT...INTO OUTFILE和LOAD DATA INFILE
		INDEX     -- 允许使用CREATE INDEX和DROP INDEX
		INSERT    -- 允许使用INSERT
		LOCK TABLES        -- 允许对您拥有SELECT权限的表使用LOCK TABLES
		PROCESS     -- 允许使用SHOW FULL PROCESSLIST
		REFERENCES    -- 未被实施
		RELOAD    -- 允许使用FLUSH
		REPLICATION CLIENT    -- 允许用户询问从属服务器或主服务器的地址
		REPLICATION SLAVE    -- 用于复制型从属服务器（从主服务器中读取二进制日志事件）
		SELECT    -- 允许使用SELECT
		SHOW DATABASES    -- 显示所有数据库
		SHOW VIEW    -- 允许使用SHOW CREATE VIEW
		SHUTDOWN    -- 允许使用mysqladmin shutdown
		SUPER    -- 允许使用CHANGE MASTER, KILL, PURGE MASTER LOGS和SET GLOBAL语句，mysqladmin debug命令；允许您连接（一次），即使已达到max_connections。
		UPDATE    -- 允许使用UPDATE
		USAGE    -- “无权限”的同义词
		GRANT OPTION    -- 允许授予权限
