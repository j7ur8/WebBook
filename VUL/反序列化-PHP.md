参考：
- https://www.php.net/manual/zh/language.oop5.magic.php


**利用条件**
- 存在反序列化操作并且反序列化的值可控。  
- 存在可以触发魔术方法的类。   
  
以上只能使存在反序列化漏洞的网站可以触发反序列化漏洞，但是其受到的影响受限于我们构造的漏洞链的危害程度。    

## 魔术方法

### `__sleep()和__wakeup() `
sleep()执行于serialize操作执行之前。    
wakeup()执行于反序列化操作后的第一步，该步骤在php一些版本中可被[绕过](/PHP/unserialize绕过.md)，但其较为依赖`__destruct`  魔术方法的存在，因为在我们绕过wakup的执行后，其如果存在析构函数，会调用析构函数结束类。  
demo.php
```php
<?php
class test{
	public $a;
	public $b;
	public function __construct($bb){
		$this->a='1';
		$this->b=$bb;
		echo "__construct"."\n";
	}
	public function __sleep(){
		echo "sleep"."\n";
		return array('a');
	}
	public function __wakeup(){
		echo "wakeup"."\n";
	}
	public function __destruct(){
		echo $this->a;
		echo "__destruct"."\n";
	}
}
$a=new test('b');
$ar = serialize($a);
echo $ar."\n";
echo "111111"."\n";
print_r(@unserialize('O:4:"test":2:{s:1:"a";s:1:"1";}')); //O:4:"test":1:{s:1:"a";s:1:"1";}
?>
```
### `__toString()` 
toString() 方法用于一个类被当成字符串时应怎样回应（PHP 5.2.0 之前，`__toString()`方法只有在直接使用于 echo 或 print 时才能生效）  
```php
<?php
class TestClass{
    public $foo;
    public function __construct($foo) {
        $this->foo = $foo;
    }
    
    public function __toString() {
        return $this->foo;
    }
}
$class = new TestClass('Hello');
echo $class;
?>
```

### `__invoke()` 
当尝试以调用函数的方式调用一个对象时  
```php
<?php
class CallableClass 
{
    function __invoke($x) {
        var_dump($x);
    }
}
$obj = new CallableClass;
$obj(5);//int(5)
var_dump(is_callable($obj)); //bool(true)
?>
```

### `__set()、__get()、__isset()、__unset()` 
在给不可访问属性赋值时，`__set()`会被调用。  
读取不可访问属性的值时，`__get()`会被调用。  
当对不可访问属性调用`isset()`或 empty() 时，`__isset()`会被调用。  
当对不可访问属性调用`unset()`时，`__unset()`会被调用。  
demo.php  
```php
<?php
class PropertyTest {
    //  被重载的数据保存在此  
    private $data = array();
     //  重载不能被用在已经定义的属性  
    public $declared = 1;
     //  只有从类外部访问这个属性时，重载才会发生 
    private $hidden = 2;

    public function __set($name, $value) {
        echo "Setting '$name' to '$value'\n";
        $this->data[$name] = $value;
    }
    public function __get($name) {
        echo "Getting '$name'\n";
        if (array_key_exists($name, $this->data)) {
            return $this->data[$name];
        }
        $trace = debug_backtrace();
        trigger_error(
            'Undefined property via __get():'.$name.' in ' . $trace[0]['file'] .'on line '.$trace[0]['line'],E_USER_NOTICE);
        return null;
    }
    //  PHP 5.1.0之后版本 
    public function __isset($name) {
        echo "Is '$name' set?\n";
        return isset($this->data[$name]);
    }
    //  PHP 5.1.0之后版本 
    public function __unset($name) {
        echo "Unsetting '$name'\n";
        unset($this->data[$name]);
    }
    // 非魔术方法  
    public function getHidden() {
        return $this->hidden;
    }
}

echo "<pre>\n";
$obj = new PropertyTest;
$obj->a = 1;
echo $obj->a . "\n\n";
var_dump(isset($obj->a));
unset($obj->a);
var_dump(isset($obj->a));
echo "\n";
echo $obj->declared . "\n\n";
echo "Let's experiment with the private property named 'hidden':\n";
echo "Privates are visible inside the class, so __get() not used...\n";
echo $obj->getHidden() . "\n";
echo "Privates not visible outside of class, so __get() is used...\n";
echo $obj->hidden . "\n";
?>
```

### `__call和__callStatic`
在对象中调用一个不可访问方法时，`__call()`会被调用。  
在静态上下文中调用一个不可访问方法时，`__callStatic()`会被调用。
demo.php
```php
<?php
class MethodTest {
    public function __call($name, $arguments) {
        // 注意: $name 的值区分大小写
        echo "Calling object method '$name' "
             . implode(', ', $arguments). "\n";
    }
    /**  PHP 5.3.0之后版本  */
    public static function __callStatic($name, $arguments) {
        // 注意: $name 的值区分大小写
        echo "Calling static method '$name' "
             . implode(', ', $arguments). "\n";
    }
}
$obj = new MethodTest;
$obj->runTest('in object context');

MethodTest::runTest('in static context');  // PHP 5.3.0之后版本
?>
```

### `__clone`
当复制完成时，如果定义了`__clone()`方法，则新创建的对象（复制生成的对象）中的`__clone()`方法会被调用，可用于修改属性的值（如果有必要的话）。  
demo.php
```php
<?php
class SubObject
{
    static $instances = 0;
    public $instance;

    public function __construct() {
        $this->instance = ++self::$instances;
    }
    public function __clone() {
        $this->instance = ++self::$instances;
    }
}

class MyCloneable{
    public $object1;
    public $object2;

    function __clone(){
        // 强制复制一份this->object， 否则仍然指向同一个对象
        $this->object1 = clone $this->object1;
    }
}

$obj = new MyCloneable();

$obj->object1 = new SubObject();
$obj->object2 = new SubObject();

$obj2 = clone $obj;

print("Original Object:\n");
print_r($obj);

print("Cloned Object:\n");
print_r($obj2);

?>
```
