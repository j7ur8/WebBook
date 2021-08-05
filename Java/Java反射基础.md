# 0X01 前言

反射过程是指获取特定类，创建类对象、并调用执行对应方法和属性这样一个过程，涉及到获取类的class对象、获取类对象、获取类的方法、获取类的属性、获取类的构造函数、执行类的方法等几个操作。

# 0X02 JAVA反射基础

Java反射机制涉及到以下几个类：

- java.lang.Class：类对象
- java.lang.reflect.Constructor：类的构造器对象
- java.lang.reflect.Field：类的属性对象
- java.lang.reflect.Method：类的方法对象

关于上述四个类的方法参考：https://www.jianshu.com/p/9be58ee20dee

### 获取类的class对象

获取类，其实就是获取每个类对应的`java.lang.Class`对象也就是class对象。有以下四种办法：

- 利用`类名.class` （需要import，或者在同文件中定义了该类）
- 利用`对象.getClass()`（需要实例化对象）
- 利用`Class.forName("完整类名")` （常用）
- 利用`loadClass("完整类名")`（常用）

测试代码如下

```java
public class GetClass{

	public static void main(String[] args)throws Exception{
		GetClass newclass = new GetClass();
		Class obj1 = GetClass.class;
		Class obj2 = newclass.getClass();
		Class obj3 = Class.forName("GetClass");
		Class obj4 = ClassLoader.getSystemClassLoader().loadClass("GetClass");
		if(obj1==obj2 && obj2==obj3 && obj3==obj4){
			System.out.println("123");
		}
	}
}
```



关于类加载器，延伸说下JVM的几种默认类加载器

**引导类加载器（BootstrapClassLoader）**

引导类加载器(BootstrapClassLoader)，底层原生代码是C++语言编写，属于jvm一部分，不继承java.lang.ClassLoader类，也没有父加载器，主要负责加载核心java库(即JVM本身)，存储在/jre/lib/rt.jar目录当中，只加载包名为java、javax、sun等开头的类。

如Object类是是所有对象的父类，归属于引导类加载器，不存在父加载器。

```java

public class Test {
    public static void main(String[] args)throws Exception{
		System.out.println("Object类加载器："+ Object.class.getClassLoader());
	}
}

```

**扩展类加载器（ExtensionsClassLoader）**

扩展类加载器(ExtensionsClassLoader)，由sun.misc.Launcher$ExtClassLoader类实现，用来在/jre/lib/ext或者java.ext.dirs中指明的目录加载java的扩展库。Java虚拟机会提供一个扩展库目录，此加载器在目录里面查找并加载java类。

如ZipPath类加载器

```java
import com.sun.nio.zipfs.ZipPath;

public class Test {

    public static void main(String[] args)throws Exception{
		System.out.println("ZipPath类加载器："+ ZipPath.class.getClassLoader());
	}
}
//ZipPath类加载器：sun.misc.Launcher$ExtClassLoader@70dea4e
```

**App类加载器/系统类加载器（AppClassLoader）**

App类加载器/系统类加载器（AppClassLoader），由sun.misc.Launcher$AppClassLoader实现，一般通过通过(java.class.path或者Classpath环境变量)来加载Java类，也就是我们常说的classpath路径。通常我们是使用这个加载类来加载Java应用类，可以使用ClassLoader.getSystemClassLoader()来获取它。

```java

public class Test {

    public static void main(String[] args)throws Exception{
		System.out.println("Test类加载器："+ Test.class.getClassLoader());
	}
}
//Test类加载器：sun.misc.Launcher$AppClassLoader@73d16e93
```

**自定义类加载器（UserDefineClassLoader）**

自定义类加载器(UserDefineClassLoader)，除了上述java自带提供的类加载器，我们还可以通过继承java.lang.ClassLoader类的方式实现自己的类加载器。具体实现方法我们等下单独讲解。

### 获取类的对象

一般使用Class对象的newInstance()方法创建类的对象，且使用newInstance()创建类的对象会触发类的无参构造函数。

```java

class ccc{

	public ccc(){
		System.out.println("无参构造函数执行");
	}

}

public class GetClass{

	public static void main(String[] args)throws Exception{
		Class c = Class.forName("ccc");
		Object obj_c = c.newInstance();
	}
}
```

但是如果类的构造函数是私有的、或无构造函数，执行代码将会报错或不触发构造函数

```java

class ccc{

	private ccc(){
		System.out.println("无参构造函数执行");
	}

}

public class GetClass{

	public static void main(String[] args)throws Exception{
		Class c = Class.forName("ccc");
		Object obj_c = c.newInstance(); //报错
	}
}
```

### 获取类的属性

主要有以下四种方法：

| 函数名                               | 功能                             |
| ------------------------------------ | -------------------------------- |
| Field[] getFields()                  | 获取所有public修饰的属性         |
| Field[] getField(String name)        | 获取指定名称的public修饰的属性   |
| Field[] getDeclaredFields()          | 获取所有属性，不考虑修饰符       |
| Field[] getDeclareField(String name) | 获取指定名称的属性，不考虑修饰符 |

测试代码如下：

```java
import java.lang.reflect.Field;

public class FieldTest{
	public String name;
    public String profession;
    protected int age;
    private String number;
    char sex;

    public static void main(String[] args){
        try{

            Class c1 = Class.forName("FieldTest"); // 创建Class对象

            Field[] fieldArray1 = c1.getDeclaredFields(); //获取全部成员变量
            Field[] fieldArray2 = c1.getFields();// 获取全部public成员变量

            for (Field field : fieldArray1){
               System.out.println(field.getName());
            }

            System.out.println("-------分割线---------");

            for (Field field : fieldArray2){
               System.out.println(field.getName());
           }
            System.out.println("-------分割线---------");

           Field fieldArray3 = c1.getField("name"); // 获取指定名称的public修饰的成员变量
           System.out.println(fieldArray3.getName());

           System.out.println("-------分割线---------");
           Field fieldArray4 = c1.getDeclaredField("number"); // 获取指定的成员变量
           System.out.println(fieldArray4.getName());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 获取类的方法

主要有如下四个函数实现

| 函数名                                                | 功能                                     |
| :---------------------------------------------------- | ---------------------------------------- |
| Method getMethod(String name,parameterTypes)          | 返回指定name的public方法                 |
| Method getDeclaredMethod(String name, parameterTypes) | 返回指定name的方法                       |
| Method[] getMethods()                                 | 获取所有的（父类、自身、接口）public方法 |
| Method[] getDeclaredMethods()                         | 获取该类中的所有方法                     |

测试代码如下

```java
import java.lang.reflect.Method;

public class MethodTest {

    public void study(String s) {
        System.out.println("学习中..." + s);
    }

    protected void run() {
        System.out.println("跑步中...");
    }

    void eat() {
        System.out.println("吃饭中...");
    }

    private String sleep(int age) {
        System.out.println("睡眠中..." + age);
        return "sleep";
    }

    public static void main(String[] args) {
        try {
            Class c = Class.forName("com.reflect.MethodTest"); // 创建Class对象
            Method[] methods1 = c.getDeclaredMethods(); // 获取所有该类中的所有方法
            Method[] methods2 = c.getMethods(); // 获取所有的public方法，包括类自身声明的public方法，父类中的public方法、实现的接口方法

            for (Method m:methods1) {
                System.out.println(m.);
            }

            System.out.println("-------分割线---------");

            for (Method m:methods2) {
                System.out.println(m);
            }

            System.out.println("-------分割线---------");

            Method methods3 = c.getMethod("study", String.class); // 获取study方法
            System.out.println(methods3);
            System.out.println("-------分割线---------");

            Method method4 = c.getDeclaredMethod("sleep", int.class); // 获取sleep方法
            System.out.println(method4);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 获取类的构造函数

主要有如下四个函数实现

| 函数名                                               | 功能                               |
| ---------------------------------------------------- | ---------------------------------- |
| Constructor<?>[] getConstructors()                   | 只返回public构造函数               |
| Constructor<?>[] getDeclaredConstructors()           | 返回所有构造函数                   |
| Constructor<> getConstructor(parameterTypes)         | 匹配和参数配型相符的public构造函数 |
| Constructor<> getDeclaredConstructor(parameterTypes) | 匹配和参数配型相符的构造函数       |

测试代码如下

```java
import java.lang.reflect.Constructor;
public class ConstructorTest {
    public ConstructorTest() {
        System.out.println("无参构造函数");
    }
    public ConstructorTest(String name) {
        System.out.println("有参构造函数" + name);
    }
    private ConstructorTest(boolean n) {
        System.out.println("私有构造函数");
    }
    public static void main(String[] args) {
        try {
            Class c1 = Class.forName("com.reflect.ConstructorTest");
          Constructor[] constructors1  = c1.getDeclaredConstructors();
          Constructor[] constructors2 = c1.getConstructors();
          for (Constructor c : constructors1) {
                System.out.println(c);
            }
            System.out.println("-------分割线---------");
            for (Constructor c : constructors2) {
                System.out.println(c);
            }
            System.out.println("-------分割线---------");
            Constructor constructors3 = c1.getConstructor(String.class);
            System.out.println(constructors3);
            System.out.println("-------分割线---------");
            Constructor constructors4 = c1.getDeclaredConstructor(boolean.class);
            System.out.println(constructors4);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 执行类的方法

执行类的方法主要用Method类中的invoke方法，与getMethod方法配合调用。

invoke方法如下

```java
public Object invoke(Object obj, Object... args)
```

如果**调用的方法是普通方法，第一个参数就是类对象；如果是静态方法，第一个参数就是类**。

测试代码如下：

```java
import java.lang.reflect.Method;
public class ReflectTest {
    public void reflectMethod() {
        System.out.println("反射测试成功!!!");
    }
    public static void main(String[] args) {
        try {
            Class c = Class.forName("ReflectTest"); // 创建Class对象
            Object m = c.newInstance(); // 创建类实例对象
            Method method = c.getMethod("reflectMethod"); // 获取reflectMethod方法
            method.invoke(m); // 调用类实例对象方法
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## 0x02 利用反射执行命令

测试代码如下：

```java
import java.lang.reflect.Method;
public class RuntimeTest {
    public static void main(String[] args) throws Exception {
        Class c1 = Class.forName("java.lang.Runtime");
        Object m = c1.newInstance();
        Method method = c1.getMethod("exec", String.class);
        method.invoke(m,"/System/Applications/Calculator.app/Contents/MacOS/Calculator");
    }
}
```

发现报错，是因为Runtime的构造函数是private修饰的，我们需要通过getRuntime方法得到Runtime类的实例

```java
import java.lang.reflect.Method;
public class RuntimeTest {
    public static void main(String[] args) throws Exception {
        Class c1 = Class.forName("java.lang.Runtime");
        c1.getMethod("exec", String.class).invoke(c1.getMethod("getRuntime").invoke(c1),"/System/Applications/Calculator.app/Contents/MacOS/Calculator");
    }
}
```

上述代码payload拆分一下就是

```java
import java.lang.reflect.Method;
public class RuntimeTest {
    public static void main(String[] args) throws Exception {
        Class c1 = Class.forName("java.lang.Runtime");
        
        Method getRuntimeMethod = c1.getMethod("getRuntime");
        Object runtime = getRuntimeMethod.invoke(c1);

        Method execMethod = c1.getMethod("exec", String.class);
        execMethod.invoke(runtime,"/System/Applications/Calculator.app/Contents/MacOS/Calculator");
        //c1.getMethod("exec", String.class).invoke(c1.getMethod("getRuntime").invoke(c1),"/System/Applications/Calculator.app/Contents/MacOS/Calculator");
    }
}

```

第4行，获取Runtime类的class对象

第6行，获取getRuntime方法

第7行，获取Runtime类的实例化对象（因为Runtime类的构造函数修饰符为private，需要使用getRuntime方法获取Runtime类的实例化对象）

第9行，获取exec方法，String.class指exec方法的参数类型

第10行，执行exec方法。exec方法为普通方法，所以invoke函数的第一个参数类型为Runtime类的实例化对象。

## 参考

- https://www.zhihu.com/question/24304289
- https://www.jianshu.com/p/9be58ee20dee
- https://www.jianshu.com/p/356e1d7a9d11
- https://xz.aliyun.com/t/9002
- https://xz.aliyun.com/t/9117
- https://xz.aliyun.com/t/7029
- https://xz.aliyun.com/t/2342
- [JAVA安全漫谈](https://govuln.com)

