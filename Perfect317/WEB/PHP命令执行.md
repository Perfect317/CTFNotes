---
typora-copy-images-to: ./..\images
---

# 1.命令执行函数

**命令执行后面得加分号 ；**

### 1.system()

直接执行命令

### 2.exec($cmd,$Arrays)

只返回最后一行，返回所有需要输出的arrays

### 3.passthru（' '）

直接执行命令

### 4.shell_exec()

直接执行命令

```
 <?php highlight_file(__FILE__);
 $cmd = $_GET["cmd"];
 $output = shell_exec($cmd); 
 echo $output;
 ?> 
```

### 5.popen($cmd,$mode)

mode为'r'或'w' 无回显，读出后输出

```
 <?php
 highlight_file(__FILE__); 
 $cmd = $_GET["cmd"]; 
 $ben = popen($cmd,'r'); 
 while($s=fgets($ben))
 {   print_r($s); } 
 ?> 
```

### 6.proc_open 

```
 <?php
 header\("content-type:text/html;charset=utf-8"); 
 highlight_file(__FILE__);
 $cmd = $_GET["cmd"]; 
 $array =  array(   array("pipe","r"),  //标准输入 
 array("pipe","w"),  //标准输出内容   
 array("file","/tmp/error-output.txt","a")  //标准输出错误 ); 
 $fp = proc_open($cmd,$array,$pipes); //打开一个进程通道 
 echo stream_get_contents($pipes[1]);  //为什么是$pipes[1]，因为1是输出内容 
 proc_close($fp);
 ?> 
```



### 7.反引号

表示命令执行

### 8.eval（）

执行eval的参数

# 2.操作系统链接符

； 多个命令顺序执行，命令与命令之间相互独立

&多个命令顺序执行，命令与命令之间相互独立，&需要写为%26才有用

&& 前面命令执行成功则执行后面命令，前面错误后面也错误

| 将前面的输出作为后面命令的参数，只显示后面

|| 类似于if-else

# 3.空格过滤绕过

大括号{ls,-l}

$IFS代替空格 $IFS、$IFS&2、$IFS9、${IFS}

重定向字符< , <>

%09(Tab)   %20(Space)

# 4.文件名过滤

### 1.通配符

? 代替一个字符

'*' 自动匹配星号后面的

### 2.单引号双引号

'' ""

('cat fl""ag.p""hp ')

### 3.反斜线

cat f\lag.p\hp

### 4.特殊变量

$1,$9,$*,$@

fl$1ag.p$9hp

### 5.内敛执行

?cmd=passthru('a=fla;b=g;c=.ph;d=p;cat $a$b$c$d');

### 6.利用lunix中的环境变量

?cmd=passthru('echo $PATH'); 输出环境变量

/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

第一位是0

echo f${PATH:5:1}   输出fl

# 5.常见文件读取命令绕过

1.tac

反向显示

2.

# 1.编码绕过
