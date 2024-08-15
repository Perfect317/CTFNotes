漏洞名称 ：SQL注入 、SQL盲注

漏洞描述：所谓SQL注入，就是通过把SQL命令插入到Web表单提交或输入域名或页面请求的查询字符串，最终达到欺骗服务器执行恶意的SQL命令。具体来说，它是利用现有应用程序，将（恶意）的SQL命令注入到后台数据库引擎执行的能力，它可以通过在Web表单中输入（恶意）SQL语句得到一个存在安全漏洞的网站上的数据库，而不是按照设计者意图去执行SQL语句。 造成SQL注入漏洞原因有两个：一个是没有对输入的数据进行过滤（过滤输入），还有一个是没有对发送到数据库的数据进行转义（转义输出）。

检测条件 ：

1、 Web业务运行正常
2、 被测网站具有交互功能模块，涉及到参数提交等等。
3、 例如待测目标URL，假设为http://www.exmaple.com/page.xxx
4、 待测目标存在参数输入，假设为name=value

检测方法：

一、 通过web漏洞扫描工具进行对网站爬虫后得到的所有链接进行检测，或者手工判断是否存在注入点，一旦确认存在漏洞，可利用自动化工具sqlmap去尝试注入。几种常见的判断方法：

1、 数字型。测试方法：
http://host/test.php?id=100 and 1=1 返回成功
http://host/test.php?id=100 and 1=2 返回失败


2、 字符型。测试方法：
http://host/test.php?name=rainman ’ and ‘1’=‘1 返回成功
http://host/test.php?name=rainman ’ and ‘1’=‘2 返回失败


3、 搜索型。搜索型注入：简单的判断搜索型注入漏洞存在不存在的办法是：
3.1）先搜索（'），如果出错，说明 90%存在这个漏洞。
3.2)然后搜索（%），如果正常返回，说明 95%有洞了。
3.3)然后再搜索一个关键字，比如（2006）吧，正常返回所有 2006 相关的信息。
3.4)再搜索（2006%'and 1=1 and '%'='）和（2006%'and 1=2 and '%'='）


4、 绕过验证（常见的为管理登陆）也称万能密码
(1) 用户名输入： ‘ or 1=1 or ‘ 密码：任意
(2)Admin’ - -（或‘ or 1=1 or ‘ - -）(admin or 1=1 --) (MS SQL)(直接输入用户名，不进行密码验证)(3)用户名输入：admin 密码输入：’ or ‘1’=’1 也可以
(4) 用户名输入：admin' or 'a'='a 密码输入：任意
(5) 用户名输入：‘ or 1=1 - -
(6) 用户名输入：admin‘ or 1=1 - - 密码输入：任意
(7) 用户名输入：1'or'1'='1'or'1'='1 密码输入：任意


5、 不同的 SQL 服务器连结字符串的语法不同，比如 MS SQL Server 使用
符号+来连结字符串，而 Oracle 使用符号||来连结：
http://host/test.jsp?ProdName=Book’ 返回错误
http://host/test.jsp?ProdName=B’+’ook 返回正常
http://host/test.jsp?ProdName=B’||’ook 返回正常说明有 SQL 注入

二、 如果应用程序已经过滤了’和+等特殊字符，我们仍然可以在输入时过把字符转换成URL编码（即字符ASCII码的16进制）来绕过检查。

修复方案：

建议在代码中对数字类型的参数先进行数字类型变换，然后再代入到SQL查询语句中，这样任何注入行为都不能成功。并且考虑过滤一些参数，比如get参数和post参数中对于SQL语言查询的部分。所以防范的时候需要对用户的输入进行检查。特别式一些特殊字符，比如单引号，双引号，分号，逗号，冒号，连接号等进行转换或者过滤。以下为需过滤的敏感字符或者语句：

需要过滤的特殊字符及字符串有：
net user
xp_cmdshell
add
exec master.dbo.xp_cmdshell
net localgroup administrators
select
count
Asc
char
mid
‘
：
"
insert
delete from
drop table
update
truncate
from
%

常见的几种web语言修复方案如下：

1、 如果网站使用的语言为asp或者aspx，则可参考以下修复方法：

ASP一般编程上可参考以下代码编程思路，过滤GET/POST请求模块代码。

```asp
dim SQL_injdata
SQL_injdata = "'|and|exec|insert|select|delete|update|count|*|%
|chr|mid|master|truncate|char|declare"
SQL_inj = split（SQL_Injdata,"|"）
If Request.QueryString<>"" Then
For Each SQL_Get In Request.QueryString
For SQL_Data=0 To Ubound（SQL_inj）
if instr （ Request.QueryString
（SQL_Get）,SQL_Inj（SQL_DATA））>0
Then
Response.Write "参数中包含非法字符"
Response.end
end if
next
Next
```


2.对于PHP语言编写的网站，大都是和mysql数据库结合的，可通过如下几种方法结合起来防范SQL注入的漏洞：

（1）修改php中默认配置文件php.ini中的配置，来降低sql注入的风险.

参考路径：/usr/local/apache2/conf/php.ini，不同中间件可能位置不一样。
修改如下几项：

safe_mode = on //开启安全模式
magic_quotes_gpc = On //开启过滤函数
display_errors = Off //禁止错误信息提示
注：把magic_quotes_gpc选项打开，在这种情况下所有的客户端GET和POST的数据都会自动进行addslashes处理，所以此时对字符串值的SQL注入是不可行的，但要防止对数字值的SQL注入，如用intval()等函数进行处理。但如果你编写的是通用软件，则需要读取服务器的magic_quotes_gpc后进行相应处理。

（2）在GET提交的数据中进行过滤select 、update、delete、insert等其他语句。使用正则就构建如下函数：

```php+HTML
<?php
function inject_check($sql_str)
{
return eregi('select|insert|update|delete|')
             }
function verify_id($id=null)
{
if (!$id) { exit('没有提交参数！'); } // 是否为空判断
elseif (inject_check($id)) { exit('提交的参数非法！'); } // 注
射判断
elseif (!is_numeric($id)) { exit('提交的参数非法！'); } // 数
字判断
$id = intval($id); // 整型化
return $id;
}
?>

然后进行对某个参数的过滤：
<?php
if (inject_check($_GET['id']))
{
exit('你提交的数据非法，请检查后重新提交！');
}
else
{
$id = verify_id($_GET['id']); // 这里引用了我们的过滤函数，
对$id进行过滤
echo '提交的数据合法，请继续！';
}
?>
```

（3）在POST提交的数据中，使用函数addslashes()是最终的比较好的方法，构建如下函数：

```php
<?php
function str_check( $str )
{
if (!get_magic_quotes_gpc()) // 判断magic_quotes_gpc是否打开
{
$str = addslashes($str); // 进行过滤
}
$str = str_replace("_", "\_", $str); // 把 '_'过滤掉
$str = str_replace("%", "\%", $str); // 把' % '过滤掉
return $str;
}
?>
对于大批量的数据，修改为如下：
<?php
function post_check($post)
{
if (!get_magic_quotes_gpc()) // 判断magic_quotes_gpc是否为打
开
{
$post = addslashes($post); // 进行magic_quotes_gpc没有打开的
情况对提交数据的过滤
}
$post = str_replace("_", "\_", $post); // 把 '_'过滤掉
$post = str_replace("%", "\%", $post); // 把' % '过滤掉
$post = nl2br($post); // 回车转换
$post= htmlspecialchars($post); // html标记转换
return $post;
}
?>
```


（4）对于MySQL用户，可以使用函数mysql_real_escape_string( )：

```php
<?php
$clean = array();
$mysql = array();
$clean['last_name'] = "O'Reilly";
$mysql['last_name'] =
mysql_real_escape_string($clean['last_name']);
$sql = "INSERT
INTO user (last_name)
VALUES ('{$mysql['last_name']}')";
?>
```


（5）使用支持参数化查询语句和占位符的数据库操作类（如PEAR::DB, PDO等），如使用PEAR::DB的例子：

```php
<?php
$sql = 'INSERT
INTO user (last_name)
VALUES (?)';
$dbh->query($sql, array($clean['last_name']));
?>
```


以上方法结合使用，则可有效的防止在PHP语言网站的SQL注入。

3、 对于jsp语言类型的网站，建议采用如下方法进行修复：

（1）通过参数化查询方式进行SQL注入攻击防护，参考代码：

```jsp
using (SqlConnection conn = new SqlConnection(connectionString))
{
conn.Open();
SqlCommand comm = new SqlCommand();
comm.Connection = conn;
//为每一条数据添加一个参数
comm.CommandText = "select COUNT(*) from Users where
Password = @Password and UserName = @UserName";
comm.Parameters.AddRange(
new SqlParameter[]{
new SqlParameter("@Password", SqlDbType.VarChar)
{ Value = password},
new SqlParameter("@UserName", SqlDbType.VarChar)
{ Value = userName},});
comm.ExecuteNonQuery();
}
```

//以‚?‛等位符进行参数化查询
PreparedStatement pstmt = con.prepareStatement("select * from
table where name = ?");
pstmt.setString(1, para);
ResultSet rs = pstmt.executeQuery();
（2）使用MyBatis技术，通过Mapper.xml文件定义SQL语句进行SQL注入攻击防护，参考代码：

<mapper namespace="TestUser">//命名空间
<select id="getById" parameterType="java.lang.String"
resultMap="TestFlowResult">
select
<include refid="TestFlowColumns" />
<![CDATA[
from TEST_TABLE
where
INSPECT_ID = #{id}
]]>
</select>
</mapper>
注：在编写mybatis的映射语句时，尽量采用‚#{xxx}‛这样的格式。若不得不使用‚${xxx}‛这样的参数，要手工地做好过滤工作，来防止sql注入攻击。

（3）使用特殊字符过滤程序防护SQL注入攻击，参考代码：

```java
public static bool SqlCheck(string OldString)
{
bool Checkvalue = false;
string NewString = OldString.ToLower();
string Replace =
"'|and|exec|insert|select|delete|update|count|*|;
|%|union|chr|mid|master|truncate|char|declare|asc|cast|
set|fetch|varchar|sysobjects|drop|alert|script|<|>";
string[] arrReplace = Replace.Split('|');
for (int i = 0; i < arrReplace.Length; i++)
{
if (NewString.IndexOf(arrReplace[i].ToString()) >= 0)
{
bolvalue = true;
break;
}
}
return Checkvalue;
}
```


4、 除此之外，还可以进行对数据库方面进行加固，来防止sql注入的产生:

（1）、 不要以sysadmin的身份连接数据库。而是使用特定的数据库用户，只具有读取，写入和更新数据库中适当数据的适当特权。此帐户定期检查，确定它所具有的特权。
（2）、 以安全的方式创建SQL。让数据库来完成创建SQL的工作，而不是在代码中完成。使用参数化SQL语句，同时也能提高查询的效率。
（3）、 保证数据库用户连接信息非明文保存。

5、 除此之外，还可以进行对管理方面进行加固，来防止sql注入的产生

（1）、 加强编程人员良好的安全编码意识，系统地学习安全编码的知识，减少源代码泄露的风险。
（2）、 建立起良好的代码审核、审查体系。由专人负责代码的审计，增加安全监督环节。







## **（1）预编译语句与参数化查询**

此处的`?`标记会被替换成实际的参数值，从而有效防止了SQL注入。

```java
String sql = "SELECT * FROM users WHERE username = ? AND password = ?";
PreparedStatement pstmt = connection.prepareStatement(sql);
pstmt.setString(1, userInputUsername);
pstmt.setString(2, userInputPassword);
ResultSet rs = pstmt.executeQuery();
```

## **（2）严格过滤特殊字符**

```python
import mysql.connector
from mysql.connector import Error

def escape_input(input_str):
    return mysql.connector.escape_string(input_str)

username = escape_input(request.form['username'])
password = escape_input(request.form['password'])

query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
```

## PHP

0x02 Sql Injection

1、常见的SQL注入漏洞主要是由于程序开发过程中不注意规范书写Sql语句以及对特殊字符的不严格过滤，从而导致客户端可以通过全局变量POST和GET提交恶意代码。

2、Fix：基于黑名单、转义、报错

黑名单

> SQL Filter

```php
$filter = "regexp|from|count|procedure|and|ascii|substr|substring|left|right|union|if|case|pow|exp|order|sleep|benchmark|into|load|outfile|dumpfile|load_file|join|show|select|update|set|concat|delete|alter|insert|create|union|or|drop|not|for|join|is|between|group_concat|like|where|user|ascii|greatest|mid|substr|left|right|char|hex|ord|case|limit|conv|table|mysql_history|flag|count|rpad|\&|\*|\.|-";

if((preg_match("/".$filter."/is",$username)== 1) || (preg_match("/".$filter."/is",$password)== 1)){
    die();
}
```

转义

> php.ini

```
magic_quotes_gpc=on     #php5.4的更高版本中，这个选项被去掉了。
```

magic_quotes_gpc 函数在php中的作用是判断解析用户提示的数据，如包括有:post、get、cookie过来的数据增加转义字符“\”{单引号（’）、双引号（”）与 NULL（NULL 字符）等字符都会被加上反斜线。}，以确保这些数据不会引起程序，特别是数据库语句因为特殊字符引起的污染而出现致命的错误。

> addslashes() 函数

```
addslashes(string) 

addslashes() 函数返回在预定义字符之前添加反斜杠的字符串。
预定义字符是：单引号（'）双引号（"）反斜杠（\）NULL
```

PS：PHP 5.4 之前 PHP 指令 magic_quotes_gpc 默认是 on， 实际上所有的 GET、POST 和 COOKIE 数据都用被 addslashes() 了。 不要对已经被 magic_quotes_gpc 转义过的字符串使用 addslashes()，因为这样会导致双层转义。 遇到这种情况时可以使用函数 get_magic_quotes_gpc() 进行检测。

报错

> 控制错误信息

```
php代码开头添加语句：error_reporting(0);
```





$username = addslashes($username); 

$password = addslashes($password);

addslashes() 函数返回在预定义字符之前添加反斜杠的字符串。

预定义字符是：

- 单引号（'）
- 双引号（"）
- 反斜杠（\）
- NULL