---
title: JAVA反射介绍及Class详解
date: 2017-04-12 20:14:01
tags:
- reflect
- java
categories: java
---

### 反射的用途
  反射对于想要在运行时去检查或修改运行在Java虚拟机中的程序的运行时行为的场合，经常用到。

  - 可扩展的特征
  应用程序可以通过使用可扩展对象的全名称创建实例来使用外部用户定义的类
  - 类浏览器和可视化开发环境
  一个类浏览器需要能够枚举出类的成员。可视化开发环境可以借助于使用反射提供的类型信息来帮助开发者写正确的代码。
  - 调试器和测试工具
  调试器能够检查类的基本成员.测试工具可以利用反射来得到类中定义的API，来确保测试用例的高度覆盖（检查单元测试覆盖率）。

### 反射的缺点
反射很强大，但不能滥用。能不用反射就不用，不得不用反射时需要考虑以下几点：
  - 性能开销
  因为反射设计动态解析的类型，所以不能执行某些Java虚拟机优化。因此反射操作的性能比非反射对象的性能更慢，应该在对性能敏感的程序中频繁调用的地方尽量避免使用反射。
  - 安全限制
  反射需要一个运行时权限，但运行在安全管理器下该权限可能不存在。对于必须在安全受限的上下文来说，很重要，如Applet
  - 内部暴露
  由于反射允许代码执行非反射代码中认为非法的操作，如访问私有字段和方法，此时使用反射能导致意外地副作用，导致代码功能失调、破坏可移植性。同时，反射代码破坏了抽象性，因此可能通过更新平台来改变行为（不懂）

### Class
每一个对象要么是引用类型要么是基本类型。引用类型全部继承自java.lang.Object。Classes、enums、arrays及interfaces都是引用类型。基本类型只包括8种：boolean、byte、short、long、char、int、float、double。引用类型例如java.lang.String，所有基本类型的包装类型如java.lang.Double，接口java.io.Serializable，及枚举javax.swing.SortOrder。

针对每一种对象类型，Java虚拟机实例化java.lang.Class的一份不可变实例。Class提供了方法来检查对象的运行时属性，包括成员和类型信息，同时Class也提供了创建新的classes及对象的功能。更重要的是，Class是所有反射API的入口。

#### 重新获取Class对象
  所有反射操作的入口是java.lang.Class。除了java.lang.reflect.ReflectPermission类，java.lang.reflect包下的其他类都没有公共构造函数。要获得这些类，有必要调取Class的相应地方法。有几种方法来来获取Class，根据是代码否有权限访问一个对象，类的名称，类型或一个已存在的Class。
##### Object.getClass()
  如果能取到类的某个实例对象，最简单的获取其Class对象的方式是调用Object.getClass()方法。当然，这种方式只适合于全部继承自Object类的引用类型。如：

    Class c = "foo".getClass();
  返回String类的Class对象

    Class c = System.console().getClass();
  和虚拟机关联的有一个唯一的console对象，通过System.console()对象返回，getClass()返回的是java.io.Console类的Class对象。

    enum E { A, B }
    Class c = A.getClass();
  A是枚举类型E的一个实例，getClass()方法返回枚举类型E对应的Class对象

    byte[] bytes = new byte[1024];
    Class c = bytes.getClass();
  由于数组是Object对象，所以也能对数组的实例调用getClass()方法。返回的Class对应组件类型为byte的数组

    import java.util.HashSet;
    import java.util.Set;

    Set<String> s = new HashSet<String>();
    Class c = s.getClass();

  该例中，java.util.Set是java.util.HashSet类型的实例对象实现的一个接口。getClass()返回的Class对应于java.util.HashSet。

##### .class语法
如果有类型但是没有实例对象，通过追加".class"到类型的名称后面也可以获取到Class。这也是基本类型最简单的方式获取Class。

      boolean b;
      Class c = b.getClass();   // 编译报错

      Class c = boolean.class;  // 正确
    注意boolean.getClass()会产生编译错误，因为boolean类型时基本类型，不能被引用。.class语法返回类型boolean对应的Class。

      Class c = java.io.PrintStream.class;
    返回java.io.PrintStream类型对应的Class对象

      Class c = int[][][].class;
    The .class也用来获取多维数组类型对应的Class

##### Class.forName()
如果有一个类的全限定名称，使用静态方法Class.forName()能获取到对应的Class对象。该方式不能用来加载基本类型。

    Class c = Class.forName("com.duke.MyLocaleServiceProvider");
该语句根据传入的全限定名来创建一个class对象

    Class cDoubleArray = Class.forName("[D");
    Class cStringArray = Class.forName("[[Ljava.lang.String;");

如果是数组类型传入的名称，则按下面Class.getName的描述；变量cDoubleArray表示基本类型double数组对应的Class（和double[].class一样），变量cStringArray表示String的二维数组对应的Class（和String[][].class一样）。

##### Class.getName()：
  返回Class对象代表的实体的字符串名称（类、接口、数组、基本类型或void）
  - 如果该class对象表示一个引用类型（非数组类型）就依据java语言规范，返回类的二进制名称。
  - 如果该class对象表示一个基本类型或者void类型，返回对应基本类型或者void的java语言关键字对应的字符串
  - 如果该class对象表示一个数组类，则名称由元素类型的名称前面追加字符"["表示数组嵌套的深度，元素类型名称的编码如下：    

元素类型|编码   

boolean	   	|Z    
    --|--
byte	   |	B
char	   	|C
类或接口	  | 	类全限定名称
double	   |	D
float	   |	F
int	   |	I
long	 |  	J
short	 |  	S

类或接口名称就如上述说明的类的二进制名称

  示例:   

方法调用	   	| 结果    
    --|--
String.class.getName() | "java.lang.String"
byte.class.getName() | "byte"
(new Object[3]).getClass().getName() | "[Ljava.lang.Object;"
(new int[3][4][5][6][7][8][9]).getClass().getName() | "[[[[[[[I"

##### 基本类型包装类的TYPE字段
对于基本类型来说.Class语法获取其Class对象非常方便，也是首选方式；还有另一种方式来获取Class。每一个基本类型和void类型都有一个包装类在java.lang中，将基本类型装箱为包装类型。每一个包装类型有一个TYPE字段，即是基本类型对应的Class对象

Class c = Double.TYPE;// 相等于double.class
Class c = Void.TYPE;//相等于void.class.

##### 返回Classes的方法
有几个反射API可以返回classes，但是只有当某个Class已经直接或间接得到才能访问那几个API。    

**Class.getSuperclass()**
  返回指定class的超类class

    Class c = javax.swing.JButton.class.getSuperclass();// 返回javax.swing.AbstractButton.    

**Class.getClasses()**    
返回指定class的所有的公共classes、接口及枚举成员，包含继承的成员。

    Class<?>[] c = Character.class.getClasses();//返回内部多个公共class成员   

**Class.getDeclaredClasses()**    
返回class里所有显示声明的类、接口、枚举（公共、私有等都包含）

    Class<?>[] c = Character.class.getDeclaredClasses();//返回显示声明的几个类

**Class.getDeclaringClass()**   
如果Class对象表示的接口或类是另一个类的成员，则返回声明该成员的类对应的Class对象；否则如果不是任何其他类的成员则返回null。如果该Class对象表示一个数组类型、基本类型、或void，则返回null。
匿名类的声明没有声明类但有一个封闭类    

    public class MyClass {
        static Object o = new Object() {
            public void m() {}
        };
        static Class<c> = o.getClass().getDeclaringClass();
    }
o定义的匿名类的声明类为null

**Class.getEnclosingClass()**
返回class的直接封闭类   

    Class c = Thread.State.class().getEnclosingClass();//枚举类Thread.State的封闭类是Thread

    public class MyClass {
        static Object o = new Object() {
            public void m() {}
        };
        static Class<c> = o.getClass().getEnclosingClass();
    }
    o定义的匿名内部类的封闭类是Mycalss

**java.lang.reflect.Field.getDeclaringClass()**   
返回这些成员被声明的Class。    

    import java.lang.reflect.Field;
    Field f = System.class.getField("out");
    Class c = f.getDeclaringClass();//返回System,out字段在System类里定义

**java.lang.reflect.Method.getDeclaringClass()**    
返回这些成员被声明的Class。

**java.lang.reflect.Constructor.getDeclaringClass()**   
返回这些成员被声明的Class。    

**Class.getGenericInterfaces()**

返回该对象表示的类或接口直接实现的接口的类型(Type类型)
  - 如果某个实现的接口是参数化类型，则返回的Type对象必须准确反映出源码中使用的实际类型参数，如果以前没有创建表示每个实现接口的参数化类型，则创建该类型。
  - 如果该对象是一个类，则返回该类实现的所有的接口数组，顺序按照该对象表示的类实现接口时的顺序对应；对于数组来说，按照Cloneable、Serializable的顺序返回。
  - 如果该对象是一个接口，返回该接口直接继承的所有接口，顺序按照和接口继承时声明的接口顺序一致。
  - 如果该对象是类或接口，但是没有实现任何接口，返回空数组。
  - 如果该对象是基本类型或void，返回空数组。

**Class.getInterfaces()**

返回类型时Class类型
- 如果该对象是一个类，则返回该类实现的所有的接口数组，顺序按照该对象表示的类实现接口时的顺序对应；对于数组来说，按照Cloneable、Serializable的顺序返回。
- 如果该对象是一个接口，返回该接口直接继承的所有接口，顺序按照和接口继承时声明的接口顺序一致。
- 如果该对象是类或接口，但是没有实现任何接口，返回空数组。
- 如果该对象是基本类型或void，返回空数组。

**Class.getModifiers()**

返回类或接口的java语言修饰符，加密成一个整数。修饰符包含Java虚拟机为public、protected、private、final、static、abstract、interface的常量，需要使用Modifier的方法来解码
  - 如果是数组类型，public、private、protected修饰符是和数组的组件类型一致
  - 如果是基本类型或者void，总是有public修饰符，总无protected及private修饰符。
  - 如果是数组类型、基本类型或void，总有final修饰符，总是无interface修饰符。其他修饰符不能由此规则决定。
  - 如果是数组类型，总有abstract修饰符

#### 检查类的修饰符和类型
class可以定义一个或多个修饰符来决定运行时行为：
  - 访问修饰符：public, protected, and private
  - 需要重写的修饰符：abstract
  - 限制为某个实例的修饰符：: static
  - 禁止值被修改的修饰符： final
  - 强制执行严格的浮点行为的修饰符：strictfp
  - 注解
并不是所有的修饰符可以用在所有类上，如接口不能用final修饰，enum不能用abstract修饰。java.lang.reflect.Modifier包含了所有可能的修饰符的声明，也包含了对Class.getModifiers()方法返回的修饰符标记的解码方法。

如下例展示了如何获取类的声明的组件类型包括修饰符、泛型参数、实现的接口及继承的父类的路径。如果累实现了java.lang.reflect.AnnotatedElement接口，也能拿到运行时的注解信息
    import java.lang.annotation.Annotation;
    import java.lang.reflect.Modifier;
    import java.lang.reflect.Type;
    import java.lang.reflect.TypeVariable;
    import java.util.Arrays;
    import java.util.ArrayList;
    import java.util.List;
    import static java.lang.System.out;

    public class ClassDeclarationSpy {
        public static void main(String... args) {
        	try {
        	    Class<?> c = Class.forName(args[0]);//根据类名称得到class对象
        	    out.format("Class:%n  %s%n%n", c.getCanonicalName());// 类的规范名称
        	    out.format("Modifiers:%n  %s%n%n",
        		       Modifier.toString(c.getModifiers()));// 输出类的修饰符

        	    out.format("Type Parameters:%n");
        	    TypeVariable[] tv = c.getTypeParameters();// 获取类型参数
        	    if (tv.length != 0) {
            		out.format("  ");
            		for (TypeVariable t : tv)
            		    out.format("%s ", t.getName());
            		out.format("%n%n");
        	    } else {
        		      out.format("  -- No Type Parameters --%n%n");
        	    }

        	    out.format("Implemented Interfaces:%n");
        	    Type[] intfs = c.getGenericInterfaces();// 实现接口
        	    if (intfs.length != 0) {
            		for (Type intf : intfs)
            		    out.format("  %s%n", intf.toString());
            		out.format("%n");
        	    } else {
        		      out.format("  -- No Implemented Interfaces --%n%n");
        	    }

        	    out.format("Inheritance Path:%n");// 继承路径
        	    List<Class> l = new ArrayList<Class>();
        	    printAncestor(c, l);
        	    if (l.size() != 0) {
            		for (Class<?> cl : l)
            		    out.format("  %s%n", cl.getCanonicalName());
            		out.format("%n");
        	    } else {
      		        out.format("  -- No Super Classes --%n%n");
        	    }

        	    out.format("Annotations:%n");
        	    Annotation[] ann = c.getAnnotations();// 注解
        	    if (ann.length != 0) {
            		for (Annotation a : ann)
            		    out.format("  %s%n", a.toString());
            		out.format("%n");
        	    } else {
      		        out.format("  -- No Annotations --%n%n");
        	    }

                // production code should handle this exception more gracefully
        	} catch (ClassNotFoundException x) {
        	    x.printStackTrace();
        	}
        }

        private static void printAncestor(Class<?> c, List<Class> l) {
        	Class<?> ancestor = c.getSuperclass();// 实现的父类
         	if (ancestor != null) {
        	    l.add(ancestor);
        	    printAncestor(ancestor, l);
         	}
        }
    }

几个不同输入参数下的结果示例如下:

    $ java ClassDeclarationSpy java.util.concurrent.ConcurrentNavigableMap
    Class:
      java.util.concurrent.ConcurrentNavigableMap

    Modifiers:
      public abstract interface

    Type Parameters:
      K V

    Implemented Interfaces:
      java.util.concurrent.ConcurrentMap<K, V>
      java.util.NavigableMap<K, V>

    Inheritance Path:
      -- No Super Classes --

    Annotations:
      -- No Annotations --
    This is the actual declaration for java.util.concurrent.ConcurrentNavigableMap in the source code:

    public interface ConcurrentNavigableMap<K,V>
        extends ConcurrentMap<K,V>, NavigableMap<K,V>
    注意由于上述例子参数是接口，隐含abstract修饰符。编译器为每个接口添加该修饰符。同样，该接口有2个泛型参数，K、V。K、V具体的额外信息可通过TypeVariable的方法得到。上述说明接口也可以实现接口。

    $ java ClassDeclarationSpy "[Ljava.lang.String;"
    Class:
      java.lang.String[]

    Modifiers:
      public abstract final

    Type Parameters:
      -- No Type Parameters --

    Implemented Interfaces:
      interface java.lang.Cloneable
      interface java.io.Serializable

    Inheritance Path:
      java.lang.Object

    Annotations:
      -- No Annotations --
    由于数组是运行时对象，所有的类型信息都由java虚拟机提供。特别的，数组实现了Cloneable、java.io.Serializable接口，并且数组的直接父类总是Object类。

    $ java ClassDeclarationSpy java.io.InterruptedIOException
    Class:
      java.io.InterruptedIOException

    Modifiers:
      public

    Type Parameters:
      -- No Type Parameters --

    Implemented Interfaces:
      -- No Implemented Interfaces --

    Inheritance Path:
      java.io.IOException
      java.lang.Exception
      java.lang.Throwable
      java.lang.Object

    Annotations:
      -- No Annotations --

    $ java ClassDeclarationSpy java.security.Identity
    Class:
      java.security.Identity

    Modifiers:
      public abstract

    Type Parameters:
      -- No Type Parameters --

    Implemented Interfaces:
      interface java.security.Principal
      interface java.io.Serializable

    Inheritance Path:
      java.lang.Object

    Annotations:
      @java.lang.Deprecated()
    java.security.Identity被注解标记为过时的api，可以用来通过反射代码来监测过时API
    注意：并不是所有的注解都能通过反射拿到，需要注解的保留策略的类型为Runtime类型（ java.lang.annotation.RetentionPolicy of RUNTIME）的注解才能在运行时拿到。

#### 获取Class的成员
Class里提供了2类方法来访问字段、方法、构造函数：枚举所有成员的方法及查找某个指定成员的方法。还有不同的方法来访问类本身直接声明的成员及查找实现的接口或实现类继承下来的成员。

定义字段的Class方法  

API	|是否枚举成员 |是否包括继承的成员 |	是否返回私有成员
-|-|-|-
getDeclaredField() | no | no	|yes
getField()	|no	|yes	|no
getDeclaredFields()|	yes	|no	|yes
getFields()	|yes|	yes	|no

定位方法的Class方法

API	|是否枚举成员| 是否包括继承的成员|是否返回私有成员
-|-|-|-
getDeclaredMethod()	|no	|no	|yes
getMethod()	|no	|yes	|no
getDeclaredMethods()	|yes	|no	|yes
getMethods()	|yes	|yes	|no

定位构造函数的方法

API	|是否枚举成员| 是否包括继承的成员|是否返回私有成员
-|-|-|-
getDeclaredConstructor()	|no	|N/A1	|yes
getConstructor()	|no	|N/A1	|no
getDeclaredConstructors()	|yes	|N/A1	|yes
getConstructors()	|yes	|N/A1	|no
构造函数不能被继承

    import java.lang.reflect.Constructor;
    import java.lang.reflect.Field;
    import java.lang.reflect.Method;
    import java.lang.reflect.Member;
    import static java.lang.System.out;

    enum ClassMember { CONSTRUCTOR, FIELD, METHOD, CLASS, ALL }

    public class ClassSpy {
        public static void main(String... args) {
        	try {
        	    Class<?> c = Class.forName(args[0]);
        	    out.format("Class:%n  %s%n%n", c.getCanonicalName());

        	    Package p = c.getPackage();
        	    out.format("Package:%n  %s%n%n",
        		       (p != null ? p.getName() : "-- No Package --"));

        	    for (int i = 1; i < args.length; i++) {
            		switch (ClassMember.valueOf(args[i])) {
              		case CONSTRUCTOR:
              		    printMembers(c.getConstructors(), "Constructor");
              		    break;
              		case FIELD:
              		    printMembers(c.getFields(), "Fields");
              		    break;
              		case METHOD:
              		    printMembers(c.getMethods(), "Methods");
              		    break;
              		case CLASS:
              		    printClasses(c);
              		    break;
              		case ALL:
              		    printMembers(c.getConstructors(), "Constuctors");
              		    printMembers(c.getFields(), "Fields");
              		    printMembers(c.getMethods(), "Methods");
              		    printClasses(c);
              		    break;
              		default:
              		    assert false;
            		}
        	    }
        	} catch (ClassNotFoundException x) {
        	    x.printStackTrace();
        	}
        }

        private static void printMembers(Member[] mbrs, String s) {
        	out.format("%s:%n", s);
        	for (Member mbr : mbrs) {
        	    if (mbr instanceof Field)
            		out.format("  %s%n", ((Field)mbr).toGenericString());
        	    else if (mbr instanceof Constructor)
        		    out.format("  %s%n", ((Constructor)mbr).toGenericString());// 返回泛型化String
        	    else if (mbr instanceof Method)
            		out.format("  %s%n", ((Method)mbr).toGenericString());
        	 }
        	if (mbrs.length == 0)
        	    out.format("  -- No %s --%n", s);
        	out.format("%n");
        }

        private static void printClasses(Class<?> c) {
        	out.format("Classes:%n");
        	Class<?>[] clss = c.getClasses();
        	for (Class<?> cls : clss)
        	    out.format("  %s%n", cls.getCanonicalName());
        	if (clss.length == 0)
        	    out.format("  -- No member interfaces, classes, or enums --%n");
        	out.format("%n");
        }
    }

以下是几个示例：

    $ java ClassSpy java.lang.ClassCastException CONSTRUCTOR
    Class:
      java.lang.ClassCastException

    Package:
      java.lang

    Constructor:
      public java.lang.ClassCastException()
      public java.lang.ClassCastException(java.lang.String)// 构造函数不能被继承

    $ java ClassSpy java.nio.channels.ReadableByteChannel METHOD
    Class:
      java.nio.channels.ReadableByteChannel

    Package:
      java.nio.channels

    Methods:
      public abstract int java.nio.channels.ReadableByteChannel.read
        (java.nio.ByteBuffer) throws java.io.IOException
      // 从实现接口继承的方法
      public abstract void java.nio.channels.Channel.close() throws
        java.io.IOException
      public abstract boolean java.nio.channels.Channel.isOpen()

    $ java ClassSpy ClassMember FIELD METHOD
    Class:
      ClassMember

    Package:
      -- No Package --

    Fields:
      // 可以根据 java.lang.reflect.Field.isEnumConstant()来区分枚举字段
      public static final ClassMember ClassMember.CONSTRUCTOR
      public static final ClassMember ClassMember.FIELD
      public static final ClassMember ClassMember.METHOD
      public static final ClassMember ClassMember.CLASS
      public static final ClassMember ClassMember.ALL

    Methods:
      // 方法名都带有类名，这样可以区分如toString()方法是Enum类的，而不是继承自Object的
      public static ClassMember ClassMember.valueOf(java.lang.String)
      public static ClassMember[] ClassMember.values()
      public final int java.lang.Enum.hashCode()
      public final int java.lang.Enum.compareTo(E)
      public int java.lang.Enum.compareTo(java.lang.Object)
      public final java.lang.String java.lang.Enum.name()
      public final boolean java.lang.Enum.equals(java.lang.Object)
      public java.lang.String java.lang.Enum.toString()
      public static <T> T java.lang.Enum.valueOf
        (java.lang.Class<T>,java.lang.String)
      public final java.lang.Class<E> java.lang.Enum.getDeclaringClass()
      public final int java.lang.Enum.ordinal()
      public final native java.lang.Class<?> java.lang.Object.getClass()
      public final native void java.lang.Object.wait(long) throws
        java.lang.InterruptedException
      public final void java.lang.Object.wait(long,int) throws
        java.lang.InterruptedException
      public final void java.lang.Object.wait() hrows java.lang.InterruptedException
      public final native void java.lang.Object.notify()
      public final native void java.lang.Object.notifyAll()

    // 获取字段的声明类
    if (mbr instanceof Field) {
        Field f = (Field)mbr;
        out.format("  %s%n", f.toGenericString());
        out.format("  -- declared in: %s%n", f.getDeclaringClass());
    }

#### 常见问题
##### 编译警告
可能发生的警告为："Note: ... uses unchecked or unsafe operations"
当调用一个方法时，参数值的类型经过校验，很可能经过转化。如下例getMethod()方法引起常见的未收校验的转换警告：

    import java.lang.reflect.Method;

    public class ClassWarning {
        void m() {
    	try {
    	    Class c = ClassWarning.class;
    	    Method m = c.getMethod("m");  // warning

            // production code should handle this exception more gracefully
    	} catch (NoSuchMethodException x) {
        	    x.printStackTrace();
        	}
        }
    }

    $ javac ClassWarning.java
    Note: ClassWarning.java uses unchecked or unsafe operations.
    Note: Recompile with -Xlint:unchecked for details.

    $ javac -Xlint:unchecked ClassWarning.java
    ClassWarning.java:6: warning: [unchecked] unchecked call to getMethod
      (String,Class<?>...) as a member of the raw type Class
    Method m = c.getMethod("m");  // warning
                          ^
因为c被声明为原始类型（不带有类型参数），但是getMethod()方法是一个参数化类型，所以发生未经转换的警告
有2中解决方法，更倾向于修改c的声明，使其带有合适的泛型类型，该例的声明应该为：

    Class<?> c = warn.getClass();
或者可以使用@SuppressWarnings来抑制警告

    Class c = ClassWarning.class;
    @SuppressWarnings("unchecked")
    Method m = c.getMethod("m");  

建议：通常的原则是,警告不应该被忽略， 因为警告可能表明一个bug。参数化声明应该合理使用。如果参数化声明不可能（如和厂商的类库代码交互），则可以使用@SuppressWarnings.
