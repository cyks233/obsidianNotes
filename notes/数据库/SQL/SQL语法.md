#数据库 
### 基础
1. 选择语句：`select`
	```sql
	SELECT column1, column2, ... 
	FROM table_name;
	
	SELECT * FROM table_name;
	```

2. 去重：`distinct` ^distinct
	```sql
	SELECT column1, column2, ...
	FROM table_name
	WHERE condition;
	```


3. 过滤记录：`where`
	```sql
	SELECT column1, column2, ...
	FROM table_name
	WHERE condition;
	```

4. `and | or`  运算符
	and运算符需要条件全都满足，会显示一条记录；or运算符只需要两边的条件之一满足，就会显示一条记录。两者可以结合成为更复杂的表达式 ^and-or
	```sql
	SELECT * 
	FROM table_name
	WHERE condition1 and condition2 and ...
	
	SELECT * 
	FROM table_name
	WHERE condition1 or condition2 or ...
	
	SELECT * 
	FROM table_name
	WHERE condition1 and (condition2 or condition3)
	```

5. 排序关键字：`order by`
	ASC表示升序，DESC表示降序，不写明是默认ASC
	升序和降序需要在排序的字段后面写明：colunm1后面没写，默认升序；colunm2写desc，则只有column2按降序排序 ^order-by
	```sql
	SELECT column1, column2, ...
	FROM table_name
	ORDER BY column1, column2, ... ASC|DESC;
	```


6. `insert into`语句用于给表插入新的数据 ^insert
	```sql
	INSERT INTO table_name
	VALUES (value1,value2,value3,...);
	
	INSERT INTO table_name (column1,column2,column3,...)
	VALUES (value1,value2,value3,...);
	```


7. 更新语句：`update`
	用于修改表的数据。一般情况下需要配合where使用，不使用where会修改所有表的数据 ^update
	```sql
	UPDATE table_name
	SET column1 = value1, column2 = value2, ...
	WHERE condition;
	```

8. 删除语句：`delete`用于删除表中的行 ^delete
	```sql
	DELETE FROM table_name
	WHERE condition;
	```

9. `group by`子句：
	用于结合聚合函数，根据一个或多个列对结果集进行分组 ^group-by
	```sql
		                      聚合函数
	SELECT column_name, aggregate_function(column_name)  
	FROM table_name  
	WHERE column_name operator value  
	GROUP BY column_name;
	```
10. `having` 子句 
	由于`where`无法与聚合函数一起使用，需要用到`having`子句
	可以让我们筛选分组后的各组数据 ^having
	```sql
	SELECT column_name, aggregate_function(column_name) 
	FROM table_name 
	WHERE column_name operator value 
	GROUP BY column_name 
	HAVING aggregate_function(column_name) operator value;
	```
11. 修改表：`alter table`
	```sql
	create table tableName
	(
	columnName1 dataType(size),
	columnName2 dataType(size),
	columnName3 dataType(size),
	...
	);
	```
### 进阶
1. `select top|limit|rownum` 
	用于规定要返回的记录数目
	>三种数据库的语法不同，SQL server使用`select top`，而MySQL使用`limit`，Oracle则使用`rownum` ^top

	```sql
	SQLServer
	SELECT TOP number|percent column_name(s)
	FROM table_name;
	
	MySQL
	SELECT column_name(s)
	FROM table_name
	LIMIT number;
	
	oracle
	SELECT column_name(s)
	FROM table_name
	WHERE ROWNUM <= number;
	```

2. `like`操作符
	用于在 WHERE 子句中搜索列中的指定模式。 ^like
	```sql
	SELECT column1, column2, ...
	FROM table_name
	WHERE column LIKE pattern;
	```
	通配符 `%   _  [charlist]` ，与like操作符配合使用
	
   | 通配符       | 用法                                                    |
   | ------------ | ------------------------------------------------------- |
   | ` %a`        | 结尾为a的数据                                           |
   | `a%`         | 开头为a的数据                                           |
   | `%a%`        | 含有a的数据                                             |
   | `_a_`        | 三位且中间为a的数据                                     |
   | `_a`         | 两位且末尾为a的数据                                     |
   | `a_`         | 两位且开头为a的数据                                     |
   | `[ab]c[de]f` | 包含中括号内字符的数据 acdf,acef,bcdf,bcef              |
   | `[a-c]de`    | 包含中括号内字符范围的数据 ade,bde,cde                  |
   | `a[^b]c`     | 所有开头为a，结尾为c，且中间不为b的数据 acc,adc,aec ... |

3. `in`操作符
	用于在 `where`子句中规定多个值。可用于[[SQL语句|子查询]] ^in
	```sql
	SELECT column1, column2, ...
	FROM table_name
	WHERE column IN (value1, value2, ...);

	SELECT column1, column2, ...
	FROM table_name
	WHERE column NOT IN (value1, value2, ...);
	```

4. `between`操作符
	用于选取介于两个值之间的数值范围内的值，`a between 1 and 10` 与 `a>=1 and a<=10` 相等 ^between
	```sql
	SELECT column1, column2, ...
	FROM table_name
	WHERE column BETWEEN value1 AND value2;
	```

5. 别名：`as`
	重命名，提升可读性 ^as
	```sql
	列的别名
	SELECT column_name AS alias_name
	FROM table_name;
	
	表的别名
	SELECT column_name(s)
	FROM table_name AS alias_name;
	```

6. 连接操作：`join`
	用于将两张或多张表的行结合起来，基于这些表中共同的字段
	```sql
	SELECT column1, column2, ...
	FROM table1
	JOIN table2 ON condition;
	```
   连接的分类
   1. 内连接：`inner join`
	   与join是一样的。通过两表之间共有的部分进行连接
      ![[Pasted image 20230722160912.png]] ^inner-join
      ```sql
      SELECT column_name(s) 
      FROM table1  
      INNER JOIN table2  
      ON table1.column_name=table2.column_name;
      ```
   2. 左外连接：`left outer join`
      又称左连接。连接时返回左表(table1)的所有数据，右表中没连接上的行的字段返回NULL。 
      ![[Pasted image 20230722162235.png]]^left-join
      ```sql
      SELECT column_name(s) 
      FROM table1 
      LEFT OUTER JOIN table2  
      ON table1.column_name=table2.column_name;
      ```
	3. 右外连接：`right outer join`
	   又称右连接。连接时返回右表(table1)的所有数据，左表中没连接上的行的字段返回NULL。 
	   ![[Pasted image 20230722163036.png]]^right-join
	   ```sql
	   SELECT column_name(s)
       FROM table1
       RIGHT OUTER JOIN table2
       ON table1.column_name=table2.column_name;
       ```
	4. 全连接：`full outer join`
	   连接时只要两表之中存在一个表匹配，就返回所有行 ^full-outer-join
	   ![[Pasted image 20230722163657.png]]
	   ```sql
	   SELECT column_name(s)
	   FROM table1
	   FULL OUTER JOIN table2
	   ON table1.column_name=table2.column_name;
	   ```
7. `union`操作符
	用于合并两个或多个select语句的结果集
	>注意：union内部的每个select语句都必须拥有**相同数量的列**，每列必须拥有**相似的数据类型**，每个select中的**列的顺序必须相同**。`union`默认选取不同的值。 ^union
	
	 ```sql
	SELECT column_name(s) FROM table1
	UNION
	SELECT column_name(s) FROM table2;
	```
	如果允许重复值，使用`union all`
	```sql
	SELECT column_name(s) FROM table1
	UNION ALL
	SELECT column_name(s) FROM table2;
	```
8. `select into`语句
	用于将一个表的数据复制到新的表中,* 号可以换成希望插入的列名 ^select-into
	```sql
	SELECT *
	INTO newtable [IN externaldb]
	FROM table1;
	```
	**mysql**的方式是`insert into ... select`
	```sql
	INSERT INTO table2
	SELECT * FROM table1;
	```
9. `exists`运算符
	用于判断查询子句是否有记录，如果有一条或多条记录存在返回 True，否则返回 False。
	`not exists`相反，查询子句没有记录是返回True，否则返回False ^exists
	```sql
	SELECT column_name(s)
	FROM table_name
	WHERE EXISTS
	(SELECT column_name FROM table_name WHERE condition);
	
	SELECT column_name(s)
	FROM table_name
	WHERE NOT EXISTS
	(SELECT column_name FROM table_name WHERE condition);
	```

### 函数
>SQL中的函数分为聚合函数和标量函数
>聚合函数计算从列中取得的值，并返回一个单一的值；
>标量函数基于输入值，并返回一个单一的值。
- 聚合函数 
	1. `AVG()`:返回数值列的平均值 ^avg
		```sql
		SELECT AVG(column_name) FROM table_name
		```
	2. `COUNT()`:返回匹配指定条件的行数 ^count
		```sql
		返回指定列的值的数目(不计NULL)
		SELECT COUNT(column_name) FROM table_name;
		
		返回表中的记录数
		SELECT COUNT(*) FROM table_name;
		
		返回指定列的不同值的数目
		SELECT COUNT(DISTINCT column_name) FROM table_name;
		``` 
	3. `MAX()`:返回指定列的最大值 ^max
		```sql
		SELECT MAX(column_name) FROM table_name;
		```
	4. `MIN()`:返回指定列的最小值 ^min
		```sql
		SELECT MIN(column_name) FROM table_name;
		```
	5. `SUM()`:返回数值列的总数 ^sum
		```sql
		SELECT SUM(column_name) FROM table_name;
		```
- 标量函数
	1. `UCASE()`:将字段的值转换为大写 ^ucase
		```sql
		SELECT UCASE(column_name) FROM table_name;
		
		SQL Server的语法是 UPPER()
		SELECT UPPER(column_name) FROM table_name;
		```
	2. `LCASE()`:把字段的值转换为小写 ^lcase
		```sql
		SELECT LCASE(column_name) FROM table_name;
		
		SQL Server的语法是 LOWER()
		SELECT LOWER(column_name) FROM table_name;
		```
	3. `MID()`:从文本字段中提取字符 ^mid
	
		|        参数          |                                     描述                                             |
		| ------------- | ------------------------------------------------ |
		| `column_name` | 必需。要提取字符的字段                           |
		| `start`       | 必需。规定开始的位置(起始值为1)                  |
		| `length`      | 可选。要返回的字符数，如果省略，返回剩余字符数。 |
		
		```sql
		SELECT MID(column_name[,start,length]) FROM table_name;
		
		实例
		SELECT MID(name,1,4) AS ShortTitle
		FROM Websites;
		```
	4. `LEN()`:返回文本字段中值的长度。 ^len
		```sql
		SELECT LEN(column_name) FROM table_name;

		MySQL的语法是 LENGTH()
		SELECT LENGTH(column_name) FROM table_name;
		```
	5. `ROUND()`:把数值字段舍入为指定的小数位数。 ^round
		
		| 参数          | 描述               |
		| ------------- | ------------------ |
		| `column_name` | 必需。要舍入的字段 |
		| `decimals`              | 可选。规定要返回的小数位数                   |
		```sql
		SELECT ROUND(column_name,decimals) FROM TABLE_NAME;
		```
	6. `NOW()`:返回当前系统的日期和时间 ^now
		```sql
		SELECT NOW() FROM table_name;
		```
	7. `FORMAT()`:对字段的显示进行格式化 ^format
		
		| 参数           | 描述                 |
		| -------------- | -------------------- |
		| `column_name ` | 必需。要格式化的字段 |
		| `format`          | 必需。规定格式             |
		```sql
		SELECT FORMAT(column_name,format) FROM table_name;
		```


***

#### 模式
1. 模式定义：`create schema <模式名> authorization <用户名>` 
	为wang定义一个学生课程模式S-T：`create schema "S-T" authorization <wang>`
2. 模式定义+视图：
	`create schema <模式名> authorization <用户名> [<表定义子句>|<视图定义子句>|<授权定义子句>]`
3. 模式删除：`drop schema <模式名> < cascade | restrict >`
	`cascade`：级联，删除模式的同时也把该模式的所有数据库对象删除 
	`restrict`：限制，如果该模式下有下属对象，比如表视图，就拒绝这个删除语句的执行

#### 表
1. 表的定义：`create table <表名> (字段名 类型 字段约束，字段名 类型 字段约束，字段名 类型 字段约束);`
	![[图片(1).png]]
	![[图片.png]]
	 修改表：`alter table`
	```sql
	create table tableName
	(
	columnName1 dataType(size),
	columnName2 dataType(size),
	columnName3 dataType(size),
	...
	);
	```
2. 表的删除：`drop table <表名> [ restrict | cascade ]`
##### 约束
1. 主键约束
	`primary key`: 在主键的属性后面加 PRIMARY KEY
	```sql
	CREATE TABLE Persons  
	(  
	P_Id int NOT NULL PRIMARY KEY,  
	LastName varchar(255) NOT NULL,  
	FirstName varchar(255),  
	Address varchar(255),  
	City varchar(255)  
	)
	```
	添加主键约束：
	```sql
	ALTER TABLE Persons  
	ADD PRIMARY KEY (P_Id)
	```
2. 外键约束：
	`foreign key`:在外键属性的后面加 foreign key references 表(该表的主键属性)
	```sql
	CREATE TABLE Orders  
	(  
	O_Id int NOT NULL PRIMARY KEY,  
	OrderNo int NOT NULL,  
	P_Id int FOREIGN KEY REFERENCES Persons(P_Id)  
	)
	```
	添加外键约束：
	```sql
	ALTER TABLE Orders  
	ADD FOREIGN KEY (P_Id)  
	REFERENCES Persons(P_Id)
	```
1. check约束：
	```sql
	CREATE TABLE Persons  
	(  
	P_Id int NOT NULL CHECK (P_Id>0),  
	LastName varchar(255) NOT NULL,  
	FirstName varchar(255),  
	Address varchar(255),  
	City varchar(255)  
	)
	```
	添加check约束：
	```sql
	ALTER TABLE Persons  
	ADD CHECK (P_Id>0)
	```
1. unique约束：
	```sql
	```
4. default约束：
	```sql
	```
#### 索引
1. 索引的建立：
	`create [unique] [cluster] index <索引名> on <表名>(<列名> [<次序>] [,<列名>[次序]] ... )` 
	`cluster`：聚簇索引，物理顺序与索引的逻辑顺序相同 
	`unique`：唯一索引 `create unique index Stusno on Student(Sno);`

2. 索引的修改：
	`alter index <旧索引名> rename to <新索引名>` `alter index SCno rename to SCSno;`

3. 索引的删除：`drop index <索引名>` `drop index Stusname;`