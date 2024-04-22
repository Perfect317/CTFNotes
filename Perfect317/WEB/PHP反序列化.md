# 1.序列化

## 数组序列化

```php
 <?php
highlight_file(__FILE__);
$a = array('benben','dazhuang','laoliu');
echo $a[0];
echo serialize($a);
?>
benbena:3:{i:0;s:6:"benben";i:1;s:8:"dazhuang";i:2;s:6:"laoliu";}
```

## 对象序列化

```php
 <?php
highlight_file(__FILE__);
class test{
    private $pub='benben';
    function jineng(){
        echo $this->pub;
    }
}
$a = new test();
echo serialize($a);
?>
O:4:"test":1:{s:9:"testpub";s:6:"benben";}
```

## 保护修饰符

```php
 <?php
highlight_file(__FILE__);
class test{
    protected $pub='benben';
    function jineng(){
        echo $this->pub;
    }
}
$a = new test();
echo serialize($a);
?>
O:4:"test":1:{s:6:"%00*%00pub";s:6:"benben";}
```

## 私有修饰符

```php
 <?php
highlight_file(__FILE__);
class test{
    private $pub='benben';
    function jineng(){
        echo $this->pub;
    }
}
$a = new test();
echo serialize($a);
?>
O:4:"test":1:{s:9:"%00test%00pub";s:6:"benben";}
```

## 成员调用对象属性

```php
 <?php
highlight_file(__FILE__);
class test{
    var $pub='benben';
    function jineng(){
        echo $this->pub;
    }
}
class test2{
    var $ben;
    function __construct(){
        $this->ben=new test();
    }
}
$a = new test2();
echo serialize($a);
?>
O:5:"test2":1:{s:3:"ben";O:4:"test":1:{s:3:"pub";s:6:"benben";}}
```

# 2.魔术方法

__construct 构造函数

__destruct 析构函数

__sleep 序列化时调用该函数

__wakeup 反序列化时调用该函数

__to string 对象当做字符串时调用该函数

__invoke 对象当做函数时调用该函数

__call 调用不存在的方法

__callStatic 静态调用的方法不存在

__set 给不存在的成员变量赋值

__get 调用的成员变量不存在

__isset 对不可访问的属性使用isset或empty时

```php
 <?php
highlight_file(__FILE__);
error_reporting(0);
class User {
    private $var;
    public function __isset($arg1 )
    {
        echo  $arg1;
    }
}
$test = new User() ;
isset($test->var);
?>

var 
```

__unset 对不可访问的属性使用unset

```php
 <?php
highlight_file(__FILE__);
error_reporting(0);
class User {
    private $var;
    public function __unset($arg1 )
    {
        echo  $arg1;
    }
}
$test = new User() ;
unset($test->var);
?>

var 
```

__clone 拷贝完成一个对象后

```php
 <?php
highlight_file(__FILE__);
error_reporting(0);
class User {
    private $var;
    public function __clone( )
    {
        echo  "__clone test";
          }
}
$test = new User() ;
$newclass = clone($test)
?>
__clone test
```

__debuginfo 调用var_dump时调用

# 3.例题

```php
 <?php
//flag is in flag.php
highlight_file(__FILE__);
error_reporting(0);
class Modifier {
    private $var;
    public function append($value)
    {
        include($value);
        echo $flag;
    }
    public function __invoke(){
        $this->append($this->var);
    }
}

class Show{
    public $source;
    public $str;
    public function __toString(){
        return $this->str->source;
    }
    public function __wakeup(){
        echo $this->source;
    }
}

class Test{
    public $p;
    public function __construct(){
        $this->p = array();
    }

    public function __get($key){
        $function = $this->p;
        return $function();
    }
}

if(isset($_GET['pop'])){
    unserialize($_GET['pop']);
}
?> 
```

