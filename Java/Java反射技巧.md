# Java反射技巧

## 反射执行命令Payload

### 利用Runtime

一个常规的反射Runtime执行命令的payload如下

```python
public class RuntimeTest {
    public static void main(String[] args) throws Exception {
        Class.forName("java.lang.Runtime").getMethod("exec", String.class).invoke(Class.forName("java.lang.Runtime").getMethod("getRuntime").invoke(Class.forName("java.lang.Runtime")),"/System/Applications/Calculator.app/Contents/MacOS/Calculator");
    }
}
```

因为Runtime类的构造函数修饰符为private，所以我们需要调用getRuntime方法获取Runtime的实例化对象。那么有没有什么办法可以访问私有方法呢？我们可以通过设置setAccessible(true)，直接访问私有方法或属性。

测试代码如下：

```java
import java.lang.reflect.Constructor;

public class RuntimeTest {
    public static void main(String[] args) throws Exception {
        Class c1= Class.forName("java.lang.Runtime");
        Constructor m = c1.getDeclaredConstructor();
        m.setAccessible(true);
        c1.getMethod("exec", String.class).invoke(m.newInstance(), "/System/Applications/Calculator.app/Contents/MacOS/Calculator");
    }
}
```

### 利用ProcessBuilder

第一个

```java
import java.util.Arrays;
///System/Applications/Calculator.app/Contents/MacOS/Calculator");
public class RuntimeTest {
    public static void main(String[] args) throws Exception {
        Class clazz = Class.forName("java.lang.ProcessBuilder");
        clazz.getMethod("start").invoke(clazz.getConstructor(Class.forName("java.util.List")).newInstance(
        Arrays.asList("/System/Applications/Calculator.app/Contents/MacOS/Calculator")));
    }
}
```

第二个

```java
import java.util.Arrays;
///System/Applications/Calculator.app/Contents/MacOS/Calculator");
//会报错，但是可以生成class文件并执行
public class RuntimeTest {
    public static void main(String[] args) throws Exception {
        Class clazz = Class.forName("java.lang.ProcessBuilder"); 
        clazz.getMethod("start").invoke(clazz.getConstructor(String[].class).newInstance(new String[][]{{"/System/Applications/Calculator.app/Contents/MacOS/Calculator"}}));
    }
}
```

## 反射执行命令之变形Payload

上文介绍了两个常规的反射命令执行payload，但是假设环境不允许我们我们得到

```java
Class.forName("java.lang.Runtime").getMethod(...)
```

只有

```java
Class.forName("java.lang.Class").getMethod(...)
```

那么我们可以通过反射java.lang.Class获取反射需要的getMethod等其他方法。测试代码如下：

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Method;
///System/Applications/Calculator.app/Contents/MacOS/Calculator");
public class RuntimeTest {
    public static void main(String[] args) throws Exception {
        Method method_getmethod=Class.forName("java.lang.Class")
            .getMethod("getMethod", new Class[] {String.class, Class[].class });

        Method method_geRuntime=(Method)method_getmethod
            .invoke(Class.forName("java.lang.Runtime"),"getRuntime",new Class[0]);

        Method method_exec=(Method)method_getmethod
            .invoke(Class.forName("java.lang.Runtime"),"exec",new Class[]{String.class});

        Object obj_runtime = method_geRuntime.invoke(Class.forName("java.lang.Runtime"),new Object[0]);
        method_exec.invoke(obj_runtime,"/System/Applications/Calculator.app/Contents/MacOS/Calculator");

    }
}
```

第6行，反射java.lang.Class获取getMethod方法

第9行，利用上文获取的getMethod方法获取getRuntime方法

第12行，利用上文获取的getMethod方法获取getMethod方法。getMethod方法并不是静态方法，所以invoke传入了Runtime类的Class实例（也是实例，注：invoke静态方法传入类名即可）

第15行，利用getRuntime方法获取Runtime类的实例

第16行，执行命令

以上过程中有一个有意思的点

**invoke 静态方法的obj不那么严格**

测试代码如下：

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Method;
///System/Applications/Calculator.app/Contents/MacOS/Calculator");
public class RuntimeTest {
    public static void main(String[] args) throws Exception {
        Method method_getmethod=Class.forName("java.lang.Class")
            .getMethod("getMethod", new Class[] {String.class, Class[].class });

        Method method_geRuntime=(Method)method_getmethod
            .invoke(Class.forName("java.lang.Runtime"),"getRuntime",new Class[0]);

        Method method_exec=(Method)method_getmethod
            .invoke(Class.forName("java.lang.Runtime"),"exec",new Class[]{String.class});

        //Object obj_runtime = method_geRuntime.invoke(Class.forName("java.lang.Runtime"),new Object[0]);
        //Object obj_runtime = method_geRuntime.invoke(null,new Object[0]);
        //Object obj_runtime = method_geRuntime.invoke("any string",new Object[0]);
        Object obj_runtime = method_geRuntime.invoke(Class.forName("java.lang.Class"),new Object[0]);
        method_exec.invoke(obj_runtime,"/System/Applications/Calculator.app/Contents/MacOS/Calculator");

    }
}
```

第16、17和18行更改了invoke第一个参数的值，同样可以无报错执行命令。这是因为getRuntime是static修饰的静态方法，传入值不影响结果。

## 参考

- https://xz.aliyun.com/t/7029#toc-8
- https://xz.aliyun.com/t/2342