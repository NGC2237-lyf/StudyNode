# 注解（annotation）

## 自定义注解

- 定义新的 Annotation 类型使用 **@interface** 关键字
- 自定义注解自动继承了 java.lang.annotation.Annotation 接口
- Annotation 的**成员变量**在 Annotation 定义中以无参数方法的形式来声明。其方法名和返回值定义了该成员的名字和类型。我们称为配置参数。类型只能是八种基本数据类型、String 类型、Class 类型、enum 类型、Annotation 类型、以上所有类型的数组。
- 可以在定义 Annotation 的成员变量时为其指定初始值，指定成员变量的初始值可使用 **default 关键字**
- 如果只有一个参数成员，建议使用 **参数名为value**
- 如果定义的注解含有配置参数，那么使用时必须指定参数值，除非它有默认值。格式是“参数名 = 参数值”，如果只有一个参数成员，且名称为 value，可以省略 “value = ”
- 没有成员定义的 Annotation 称为 **标记**；包含成员变量的 Annotation 称为元数据 Annotation

> 注意：自定义注解必须配上注解的信息（使用反射）处理流程才有意义。
>
> 自定义注解通常都会指明两个元注解：Retention、Target

```JAVA
public @interface MyAnnotation {
    String value() default "hello";
}
```

如果注解有成员，在使用时要给成员赋值

## JDK 提供的元注解

- 对现有注解进行修饰的注解
- JDK5.0 提供了4个标准的 meta-annotation 类型，分别是：
  - Retention
  - Target
  - Documented
  - Inherited



> 元数据：对现有数据的修饰
>
> 元数据的理解：
>
> String name = “lvyanfnag”
>
> 此时 String 和 name 都是元数据（对 lvyanfnag 进行修饰）

```JAVA
@Retention(RetentionPolicy.CLASS)
public @interface MyAnnotation {
    String value() default "hello";
}
```

------

### @Retention

Retention：指定所修饰的 Annotation 生命周期

SOURCE\CLASS（默认设置）\ RUNTIME 只有声明为 RUNTIME 生命周期的注解，才能通过反射获取。

------

### @Target

用于只当被修饰的 Annotation 能用于修饰哪些程序元素

------

### @Documented

注解在编译器编译的时候，默认为不保留

但是加上 @Documented 之后，保留注解

注意：定义为 Documented  的注解必须设置 Retention 值为 RUNTIME

------

### @Inherited

@Inherited：被它修饰的 Annotation 将具有 **继承性**。如果某个类使用了被 @Inherited 修饰的 Annotation，则其子类将自动具有该注解。

------

## 通过反射获取注解信息

```JAVA
public void testGetAnnotation(){
    Class clazz = Student.class;
    Annotation[] annotations = clazz.getAnnotations();
    for(int i = 0; i < annotations.length ; i++){
        System.out.println(annotations[i])
    }
}
```

## JDK 8 中注解的新特性

JDK 8 中注解的新特性：可重复注解，类型注解

------

### 可重复注解

jdk 8 之前的写法

```JAVA
// 定义一个新的注解，里边的value类型为注解数组
public @interface MyAnnotations {
    MyAnnotation[] value();
}
```

```JAVA
// 在类上注释的时候
@MyAnnotations({@MyAnnotation(value = "hi"),@MyAnnotation(value = "haha")})
```

jdk 8 之后的写法

> // 增加了一个新的注解
> @Repeatable

```JAVA
// 定义一个新的注解，里边的value类型为注解数组
public @interface MyAnnotations {
    MyAnnotation[] value();
}
```

```java
@Retention(RetentionPolicy.CLASS)
@Repeatable(MyAnnotations.class)
public @interface MyAnnotation {
    String value() default "hello";
}
```

> 可能会报错，要将 @MyAnnotation 和 @MyAnnotations 注解上边的 Retention 和 Target 注解范围保持一致

### 类型注解

JDK 1.8 之后，关于元注解 @Target 的参数类型 ElementType 枚举值多了两个：

- TYPE_PARAMETER
- TYPE_USE

在 Java 8 之前，注解只能是在声明的地方所使用， Java 8 开始，注解可以应用在任何地方。

- ElementType.TYPE_PARAMETER 表示该注解能写在类型变量的声明语句中（如：泛型声明）。
- ElementType.TYPE_USE 表示该注解能写在使用类型的任何语句中

```JAVA
//ElementType.TYPE_PARAMETER 用法
// 先在注解上添加 
@Target(ElementType.TYPE_PARAMETER)
public @interface MyAnnotation {
    String value() default "hello";
}

// 作用：可以通过反射得到注解，进行操作
class Generic<@MyAnnotation T>{

}
```

```JAVA
// ElementType.TYPE_USE 用法
// 先在注解上添加 
@Target(ElementType.TYPE_USE)
public @interface MyAnnotation {
    String value() default "hello";
}
// 作用：通过反射，可以对传入参数进行规范
class Generic<@MyAnnotation T>{
    public void show() throws @MyAnnotation RuntimeException{
        ArrayList<@MyAnnotation String> list = new ArrayList<>();
        int num = (@MyAnnotation int) 10L;
    }
}
```

```JAVA
strategy=GenerationType.AUTO
该注解可以采取自增策略
    
有一些注解的作用很重要，不加的话就实现不了一些功能，比如，数据持久化操作中，通过@Entity注解来标识持久化实体类，如果不使用该注解程序就识别不了持久化实体类。
```



------

# 反射

- **Reflection（反射）**是被视为**动态语言**的关键，反射机制允许程序在执行期借助于 Reflection API 取得任何类的内部信息，并能直接操作任意对象的内部属性及方法。
- 加载完类之后，在堆内存的方法区中就产生了一个 Class 类型的对象（一个类只有一个 Class 对象），这个对象就包含了完整的类的结构信息。我们可以通过这个对象看到类的结构。这个对象就像一面镜子，透过这个镜子看到类的结构。

> 正常方式：引入需要的“包类”名称 --> 通过 new 实例化 --> 取得实例化对象
>
> 反射方式：实例化对象 --> getClass() 方法 --> 得到完整的“包类”名称

## 补充：动态语言&&静态语言

1、动态语言

是一类运行时可以改变其结构的语言：例如新的函数、对象、甚至代码可以被引进，已有的函数可以被删除或是其他结构上的变化。通俗点说就是在**运行时代码可以根据某些条件改变自身结构**。

主要动态语言： Object-C、C#、JavaScript、PHP、Python

2、静态语言

与动态语言相对应的，运**行时结构不可变的语言就是静态语言**。

主要的静态语言：Java、C、C++

> Java 不是动态语言，但 Java 可以称之为“准动态语言”。即 Java 有一定的动态性，我们可以利用反射机制、字节码操作获得类似动态性语言的特性。

## 反射相关的主要API

- java.lang.Class：代表一个类
- java.lang.reflect.Method：代表类的方法
- java.lang.reflet.Field：代表类的成员变量
- java.lang.reflect.Constructor：代表类的构造器

------

### 通过反射获取类

```JAVA
public class UsePersion {
    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        // 通过反射获取 Person 类
        Class clazz = Person.class;
        Constructor constructor = clazz.getConstructor(String.class, Integer.class);
        Object o = constructor.newInstance("Tom", 12);
        System.out.println(o.toString());
    }
}
```

![image-20221008160604169](D:\笔记\照片\image-20221008160604169.png)

### 疑问

- 通过直接 new 的方式或反射的方式都可以调用公共的结构，开发中到底用那个？

建议：直接 new 的方式

反射的特征：动态性

- 反射机制与面向对象中的封装性是不是矛盾的？如何看待两个技术？

不矛盾

## 反射class类的理解

1、类的加载过程

程序经过 javac.exe 命令以后，会生成一个或多个字节码文件（.class 结尾）。

接着我们使用 java.exe 命令对某个字节码文件进行解释运行。相当于将某个字节码文件加载到内存中，此过程就称为类的加载。

加载到内存中的类，我们就称为运行时类，此运行时类，就作为 Class 的一个实例。

2、换句话说，Class 的实例就对应着一个运行时类。

## 获取Class实例的四种方法

- 调用运行时类的属性： `.class`
- 通过运行时类的对象，调用：`getClass()`
- 调用 Class 的静态方法：`forName(String classPath)`   --> classPath是 含包名的类的全路径
- 使用类的加载器：`ClassLoader`

```JAVA
	@Test
    public void test1() throws ClassNotFoundException {
        //方式一：调用运行时类的属性，调用： .class
        Class clazz1 = Person.class;
        //方式二：通过运行时类的对象，调用：  getClass()
        Person person = new Person();
        Class clazz2 = person.getClass();

        //方式三：调用 Class 的静态方法： forName(String classPath)
        Class clazz3 = Class.forName("包含包名的类的全路径名称");
        //方式四：使用类的加载器：ClassLoader
        ClassLoader classLoader = UsePersion.class.getClassLoader();
        Class clazz4 = classLoader.loadClass("包含包名的类的全路径名称");
    }
```

加载到内存中的运行时类，会缓存一定的时间。在此时间之内，我们可以通过不同的方式来获取此运行时类。（这四种方式获取的运行时类都指向同一个内存空间）

------

## Class实例对应的结构的说明

- class
  - 外部类，成员（成员内部类，静态内部类），局部内部类，匿名内部类
- interface：接口
- []：数组
- enum：枚举
- annotation：注解 @interface
- primitive type：基本数据类型
- void

```JAVA
int[] a = new int[10];
int[] b = new int[100];
Class c10 = a.getClass();
Class c11 = b.getClass();
// 只要数组的元素类型与维度一样，就是同一个 Class
```

