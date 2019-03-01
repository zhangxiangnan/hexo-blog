title: JAVA反射之字段&方法&构造函数
date: 2017-04-12 20:32:02
tags:
- reflect
- java
categories: java
---

### 反射中字段、方法、构造函数的学习
<!-- more -->

反射定义了一个接口java.lang.reflect.Member，java.lang.reflect.Field、java.lang.reflect.Method、java.lang.reflect.Constructor都实现了该接口。

注意:根据java7的语言规范，类的成员是类体可继承的组件，包括fields、methods、nested classes、interfaces及枚举类型。由于构造函数不能被继承，他们不是成员。这和java.lang.reflect.Member的实现类不同。

### 字段
字段有一个类型和值，java.lang.reflect.Field类提供了访问类型信息和设置、获取某个对象的某字段值的方法。

#### 获取字段类型
一个字段要么是基本类型，要么是引用类型。有8中基本类型：boolean, byte, short, int, long, char, float, and double. 引用类型是直接或间接继承java.lang.Object类的子类，包括接口、数组及枚举类型。

如下是一个输出不同类型字段的类型及泛华类型的展示：
```
    import java.lang.reflect.Field;
    import java.util.List;

    public class FieldSpy<T> {
        public boolean[][] b = {{ false, false }, { true, true } };
        public String name  = "Alice";
        public List<Integer> list;
        public T val;

        public static void main(String... args) {
        try {
            Class<?> c = Class.forName(args[0]);
            Field f = c.getField(args[1]);
            System.out.format("Type: %s%n", f.getType());
            System.out.format("GenericType: %s%n", f.getGenericType());
        } catch (ClassNotFoundException x) {
            x.printStackTrace();
        } catch (NoSuchFieldException x) {
            x.printStackTrace();
        }
        }
    }
```

```
    $ java FieldSpy FieldSpy b
    Type: class [[Z
    GenericType: class [[Z

    $ java FieldSpy FieldSpy name
    Type: class java.lang.String
    GenericType: class java.lang.String

    $ java FieldSpy FieldSpy list
    Type: interface java.util.List
    GenericType: java.util.List<java.lang.Integer>

    $ java FieldSpy FieldSpy val
    Type: class java.lang.Object
    GenericType: T
```
说明:字段b的类型时boolean类型的二维数组，类型名称的语法规则根据Class.getName()。
字段val的类型时java.lang.Object，因为泛型的信息会在编译期间擦除泛型的相关信息。T被类型变量的上层限制替代，这里是java.lang.Object.
Field.getGenericType()方法会在类文件中查找签名属性（如果存在）。如果签名属性不存在，会返回Field.getType()的值（没有因为引入泛型发生改变）。反射的其他名称为getGenericFoo的方法实现思路类似。

#### 获取&解析字段的修饰符
字段声明时允许的几个修饰符：
  - 访问修饰符: public, protected, and private
  - 管理运行时行为的修饰符： transient and volatile
  - 限制到一个实例的修饰符: static
  - 禁止修改值的修饰符：final
  - 注解
Field.getModifiers()方法用来一个整数值，代表该字段声明的修饰符集（一个或多个修饰符），该整数值中位表示的修饰符在java.lang.reflect.Modifier定义。
下例展示了如何根据给定的修饰符查找字段，以及判断字段是否是合成（编译器生成的）及是否是枚举常量
```
    import java.lang.reflect.Field;
    import java.lang.reflect.Modifier;
    import static java.lang.System.out;

    enum Spy { BLACK , WHITE }

    public class FieldModifierSpy {
        volatile int share;
        int instance;
        class Inner {}

        public static void main(String... args) {
        	try {
        	    Class<?> c = Class.forName(args[0]);
        	    int searchMods = 0x0;
        	    for (int i = 1; i < args.length; i++) {
        		      searchMods |= modifierFromString(args[i]);
        	    }

        	    Field[] flds = c.getDeclaredFields();
        	    out.format("Fields in Class '%s' containing modifiers:  %s%n",
        		       c.getName(),
        		       Modifier.toString(searchMods));
        	    boolean found = false;
        	    for (Field f : flds) {
            		int foundMods = f.getModifiers();
            		// Require all of the requested modifiers to be present
            		if ((foundMods & searchMods) == searchMods) {
            		    out.format("%-8s [ synthetic=%-5b enum_constant=%-5b ]%n",
            			       f.getName(), f.isSynthetic(),
            			       f.isEnumConstant());
            		    found = true;
            		}
        	    }

        	    if (!found) {
        		      out.format("No matching fields%n");
        	    }
                // production code should handle this exception more gracefully
        	} catch (ClassNotFoundException x) {
        	    x.printStackTrace();
        	}
        }

        private static int modifierFromString(String s) {
        	int m = 0x0;
        	if ("public".equals(s))           m |= Modifier.PUBLIC;
        	else if ("protected".equals(s))   m |= Modifier.PROTECTED;
        	else if ("private".equals(s))     m |= Modifier.PRIVATE;
        	else if ("static".equals(s))      m |= Modifier.STATIC;
        	else if ("final".equals(s))       m |= Modifier.FINAL;
        	else if ("transient".equals(s))   m |= Modifier.TRANSIENT;
        	else if ("volatile".equals(s))    m |= Modifier.VOLATILE;
        	return m;
        }
    }


    $ java FieldModifierSpy FieldModifierSpy volatile
    Fields in Class 'FieldModifierSpy' containing modifiers:  volatile
    share    [ synthetic=false enum_constant=false ]

    $ java FieldModifierSpy Spy public
    Fields in Class 'Spy' containing modifiers:  public
    BLACK    [ synthetic=false enum_constant=true  ]
    WHITE    [ synthetic=false enum_constant=true  ]

    $ java FieldModifierSpy FieldModifierSpy\$Inner final
    Fields in Class 'FieldModifierSpy$Inner' containing modifiers:  final
    this$0   [ synthetic=true  enum_constant=false ]

    $ java FieldModifierSpy Spy private static final
    Fields in Class 'Spy' containing modifiers:  private static final
    $VALUES  [ synthetic=true  enum_constant=false ]// 枚举类有private static final类型的合成字段$VALUES
```
注意编译器会生成一些合成的运行时需要的字段，可使用Field.isSynthetic()来判断是否合成字段，合成的字段根据编译器不同不同。然而内部类引入this$0字段 (即嵌套类为非静态成员类）来持有最外层类的引用；枚举类引入$VALUES字段实现隐式地定义静态方法values().合成的类成员的名字未被指定，不同的编译器实现或不同版本中可能不同。Class.getDeclaredFields()方法会返回包含合成字段的数组，但是Class.getField()方法不会返回，因为合成字段通常不是public的。
因为Field字段实现了接口java.lang.reflect.AnnotatedElement，因此运行时能够获取到保留策略为java.lang.annotation.RetentionPolicy.RUNTIME的注解信息。

#### 设置&获取字段值
给定某个类的一个实例，是能够用反射来设置类的字段的值。这通常仅在特殊情况下不能够以常规方式设置值。因为这么做破坏了类的设计意图，应该慎用。
示例：
```
      import java.lang.reflect.Field;
      import java.util.Arrays;
      import static java.lang.System.out;

      enum Tweedle { DEE, DUM }

      public class Book {
          public long chapters = 0;
          public String[] characters = { "Alice", "White Rabbit" };
          public Tweedle twin = Tweedle.DEE;

          public static void main(String... args) {
          	Book book = new Book();
          	String fmt = "%6S:  %-12s = %s%n";

          	try {
          	    Class<?> c = book.getClass();

          	    Field chap = c.getDeclaredField("chapters");
          	    out.format(fmt, "before", "chapters", book.chapters);
            	    chap.setLong(book, 12);
          	    out.format(fmt, "after", "chapters", chap.getLong(book));

          	    Field chars = c.getDeclaredField("characters");
          	    out.format(fmt, "before", "characters",
          		       Arrays.asList(book.characters));
          	    String[] newChars = { "Queen", "King" };
          	    chars.set(book, newChars);
          	    out.format(fmt, "after", "characters",
          		       Arrays.asList(book.characters));

          	    Field t = c.getDeclaredField("twin");
          	    out.format(fmt, "before", "twin", book.twin);
          	    t.set(book, Tweedle.DUM);
          	    out.format(fmt, "after", "twin", t.get(book));

                  // production code should handle these exceptions more gracefully
          	} catch (NoSuchFieldException x) {
          	    x.printStackTrace();
          	} catch (IllegalAccessException x) {
          	    x.printStackTrace();
          	}
          }
      }


    $ java Book
    BEFORE:  chapters     = 0
     AFTER:  chapters     = 12
    BEFORE:  characters   = [Alice, White Rabbit]
     AFTER:  characters   = [Queen, King]
    BEFORE:  twin         = DEE
     AFTER:  twin         = DUM
```
注意：通过反射设置字段的值有一定的性能开销，因为必须进行各种操作，比如访问权限验证。从运行时角度看，效果一样，并且操作就像在代码中直接改变值一样是原子性的。
反射的使用会导致丢失一些运行时优化，如下代码很可能被虚拟机优化，但是使用Field.set*()就不会进行优化。
```
    int x = 1;
    x = 2;
    x = 3;
```
#### 常见代码错误

##### IllegalArgumentException由于不可转换类型（due to Inconvertible Types）
当使用反射给引用类型的整数赋值基本类型的数值时，就会报该错误。不是用反射的话，编译器会执行自动装箱操作，将基本类型装箱为引用类型，这样类型检查就没问题，但是用反射的话，类型检查只发生在运行时，没有机会去执行装箱操作。
```
      import java.lang.reflect.Field;

      public class FieldTrouble {
          public Integer val;

          public static void main(String... args) {
      	FieldTrouble ft = new FieldTrouble();
      	try {
      	    Class<?> c = ft.getClass();
      	    Field f = c.getDeclaredField("val");
        	    f.setInt(ft, 42);               // IllegalArgumentException

              // production code should handle these exceptions more gracefully
      	} catch (NoSuchFieldException x) {
      	    x.printStackTrace();
       	} catch (IllegalAccessException x) {
       	    x.printStackTrace();
      	}
          }
      }
      $ java FieldTrouble
      Exception in thread "main" java.lang.IllegalArgumentException: Can not set
        java.lang.Object field FieldTrouble.val to (long)42
              at sun.reflect.UnsafeFieldAccessorImpl.throwSetIllegalArgumentException
                (UnsafeFieldAccessorImpl.java:146)
              at sun.reflect.UnsafeFieldAccessorImpl.throwSetIllegalArgumentException
                (UnsafeFieldAccessorImpl.java:174)
              at sun.reflect.UnsafeObjectFieldAccessorImpl.setLong
                (UnsafeObjectFieldAccessorImpl.java:102)
              at java.lang.reflect.Field.setLong(Field.java:831)
              at FieldTrouble.main(FieldTrouble.java:11)

```
解决办法：
```
    f.set(ft, new Integer(43));
```
提示：当时使用反射设&获取一个字段的值的时候，编译器没机会来执行装箱操作。编译器只能转换Class.isAssignableFrom()方法的规范描述的相关转换。如下：
```
      Integer.class.isAssignableFrom(int.class) == false// 反射时引用类型到基本类型不成功
      int.class.isAssignableFrom(Integer.class) == false// 反射时基本类型到引用类型不成功
```
##### NoSuchFieldException for Non-Public Fields
```
      $ java FieldSpy java.lang.String count
      java.lang.NoSuchFieldException: count
              at java.lang.Class.getField(Class.java:1519)
              at FieldSpy.main(FieldSpy.java:12)
```
提示：Class.getField()及Class.getFields()方法返回class对象代表的类、枚举、接口的公共成员方法。想获取类声明的所有方法（不是继承），使用Class.getDeclaredFields()方法。
##### IllegalAccessException when Modifying Final Fields
如果尝试获取&设置私有或其他无法访问的字段的值，或设置final字段的值（不论修饰符是什么）可能会抛出IllegalAccessException异常。

```
      import java.lang.reflect.Field;

      public class FieldTroubleToo {
          public final boolean b = true;

          public static void main(String... args) {
      	FieldTroubleToo ft = new FieldTroubleToo();
      	try {
      	    Class<?> c = ft.getClass();
      	    Field f = c.getDeclaredField("b");
      // 	    f.setAccessible(true);  // solution
      	    f.setBoolean(ft, Boolean.FALSE);   // IllegalAccessException

              // production code should handle these exceptions more gracefully
      	} catch (NoSuchFieldException x) {
      	    x.printStackTrace();
      	} catch (IllegalArgumentException x) {
      	    x.printStackTrace();
      	} catch (IllegalAccessException x) {
      	    x.printStackTrace();
      	}
          }
      }

    $ java FieldTroubleToo
    java.lang.IllegalAccessException: Can not set final boolean field
      FieldTroubleToo.b to (boolean)false
            at sun.reflect.UnsafeFieldAccessorImpl.
              throwFinalFieldIllegalAccessException(UnsafeFieldAccessorImpl.java:55)
            at sun.reflect.UnsafeFieldAccessorImpl.
              throwFinalFieldIllegalAccessException(UnsafeFieldAccessorImpl.java:63)
            at sun.reflect.UnsafeQualifiedBooleanFieldAccessorImpl.setBoolean
              (UnsafeQualifiedBooleanFieldAccessorImpl.java:78)
            at java.lang.reflect.Field.setBoolean(Field.java:686)
            at FieldTroubleToo.main(FieldTroubleToo.java:12)
```
提示：class初始化后，存在一个访问限制组织修改final字段值。Field声明为继承自AccessibleObject，提供了方法来抑制此检查。但这会产生副作用；如有时即使值已经被修改，但是程序的其他部分仍可能使用旧值。AccessibleObject.setAccessible()仅在安全上下文允许的情况下才能成功。
### 方法
方法拥有返回值、参数、及可能抛出异常。java.lang.reflect.Method类提供了获取参数和返回值的类型信息的方法，也经常用来执行指定对象的方法。

#### 获取方法类型信息
一个字段要么是基本类型要么是引用类型，有8中基本类型：boolean、byte、short、int、long、char、float、double。一个引用类型指直接或间接继承java.lang.Object类包括接口、数组、枚举类型的任意对象。
```
      import java.lang.reflect.Field;
      import java.util.List;

      public class FieldSpy<T> {
          public boolean[][] b = {{ false, false }, { true, true } };
          public String name  = "Alice";
          public List<Integer> list;
          public T val;// 参数化类型

          public static void main(String... args) {
          	try {
          	    Class<?> c = Class.forName(args[0]);
          	    Field f = c.getField(args[1]);
          	    System.out.format("Type: %s%n", f.getType());
          	    System.out.format("GenericType: %s%n", f.getGenericType());

                  // production code should handle these exceptions more gracefully
          	} catch (ClassNotFoundException x) {
          	    x.printStackTrace();
          	} catch (NoSuchFieldException x) {
          	    x.printStackTrace();
          	}
          }
      }

    $ java FieldSpy FieldSpy b
    Type: class [[Z
    GenericType: class [[Z
    $ java FieldSpy FieldSpy name
    Type: class java.lang.String
    GenericType: class java.lang.String
    $ java FieldSpy FieldSpy list
    Type: interface java.util.List
    GenericType: java.util.List<java.lang.Integer>
    $ java FieldSpy FieldSpy val
    Type: class java.lang.Object
    GenericType: T
```
字段b是二维boolean数组，其类型名称的语法在Class.getName()描述。
字段val的类型结果是继承自java.lang.Object，因为通过类型擦除实现泛型，在编译期间删除删除有关泛型的信息。所以T被类型变量的上界替换，该例是java.lang.Object.

Field.getGenericType()在类文件中查找签名属性（如果存在），如果不存在，会降级为Field.getType()（没有因为泛型导致被改变）。反射中其他有类似getGenericFoo方法来获取Foo的某个值的实现都类似。

#### 获取方法或构造函数参数的名称和其他信息
可以使用java.lang.reflect.Executable.getParameters方法来获取任何方法或构造函数的形式参数(Method和Constructor类继承了Executable，因此继承了Executable.getParameters方法)然而，.class文件不保存默认不保存形参名称。这是因为许多生成和使用类的工具不希望更大的静态或动态的包含参数名称的类文件占位。尤其，这些工具不得不处理更大的.class类文件，JVM也会使用更多的内存。另外，某些参数名称，如secret、password可能暴露安全敏感的方法信息。

为了在制定类文件中保存形参名称，这样就能够在反射时拿到这些形参的名称，可以使用-parameters配置javac编译器来编译源文件。
```
      import java.lang.reflect.`*`;
      import java.util.function.`*`;
      import static java.lang.System.out;

      public class MethodParameterSpy {

        private static final String  fmt = "%24s: %s%n";

        <E extends RuntimeException> void genericThrow() throws E {}

        public static void printClassConstructors(Class c) {
          Constructor[] allConstructors = c.getConstructors();
          out.format(fmt, "Number of constructors", allConstructors.length);
          for (Constructor currentConstructor : allConstructors) {
              printConstructor(currentConstructor);
          }  
          Constructor[] allDeclConst = c.getDeclaredConstructors();
          out.format(fmt, "Number of declared constructors",
              allDeclConst.length);
          for (Constructor currentDeclConst : allDeclConst) {
              printConstructor(currentDeclConst);
          }          
        }

        public static void printClassMethods(Class c) {
         Method[] allMethods = c.getDeclaredMethods();
          out.format(fmt, "Number of methods", allMethods.length);
          for (Method m : allMethods) {
              printMethod(m);
          }        
        }

        public static void printConstructor(Constructor c) {
          out.format("%s%n", c.toGenericString());
          Parameter[] params = c.getParameters();
          out.format(fmt, "Number of parameters", params.length);
          for (int i = 0; i < params.length; i++) {
              printParameter(params[i]);
          }
        }

        public static void printMethod(Method m) {
          out.format("%s%n", m.toGenericString());
          out.format(fmt, "Return type", m.getReturnType());
          out.format(fmt, "Generic return type", m.getGenericReturnType());

          Parameter[] params = m.getParameters();
          for (int i = 0; i < params.length; i++) {
              printParameter(params[i]);
          }
        }

        public static void printParameter(Parameter p) {
          out.format(fmt, "Parameter class", p.getType());
          out.format(fmt, "Parameter name", p.getName());
          out.format(fmt, "Modifiers", p.getModifiers());
          out.format(fmt, "Is implicit?", p.isImplicit());//
          out.format(fmt, "Is name present?", p.isNamePresent());//参数名字是否存在
          out.format(fmt, "Is synthetic?", p.isSynthetic());//是否自动生成
        }

        public static void main(String... args) {        
            try {
                printClassConstructors(Class.forName(args[0]));
                printClassMethods(Class.forName(args[0]));
            } catch (ClassNotFoundException x) {
                x.printStackTrace();
            }
        }
      }

      import java.util.`*`;

      public class ExampleMethods<T> {

          public boolean simpleMethod(String stringParam, int intParam) {
              System.out.println("String: " + stringParam + ", integer: " + intParam);
              return true;
          }

          public int varArgsMethod(String... manyStrings) {
              return manyStrings.length;
          }

          public boolean methodWithList(List<String> listParam) {
              return listParam.isEmpty();
          }

          public <T> void genericMethod(T[] a, Collection<T> c) {
              System.out.println("Length of array: " + a.length);
              System.out.println("Size of collection: " + c.size());
          }

      }

      执行命令：
      java MethodParameterSpy ExampleMethods

      输出：

      Number of constructors: 1

      Constructor #1
      public ExampleMethods()

      Number of declared constructors: 1

      Declared constructor #1
      public ExampleMethods()

      Number of methods: 4

      Method #1
      public boolean ExampleMethods.simpleMethod(java.lang.String,int)
                 Return type: boolean
         Generic return type: boolean
             Parameter class: class java.lang.String
              Parameter name: stringParam
                   Modifiers: 0
                Is implicit?: false
            Is name present?: true
               Is synthetic?: false
             Parameter class: int
              Parameter name: intParam
                   Modifiers: 0
                Is implicit?: false
            Is name present?: true
               Is synthetic?: false

      Method #2
      public int ExampleMethods.varArgsMethod(java.lang.String...)
                 Return type: int
         Generic return type: int
             Parameter class: class [Ljava.lang.String;
              Parameter name: manyStrings
                   Modifiers: 0
                Is implicit?: false
            Is name present?: true
               Is synthetic?: false

      Method #3
      public boolean ExampleMethods.methodWithList(java.util.List<java.lang.String>)
                 Return type: boolean
         Generic return type: boolean
             Parameter class: interface java.util.List
              Parameter name: listParam
                   Modifiers: 0
                Is implicit?: false
            Is name present?: true
               Is synthetic?: false

      Method #4
      public <T> void ExampleMethods.genericMethod(T[],java.util.Collection<T>)
                 Return type: void
         Generic return type: void
             Parameter class: class [Ljava.lang.Object;
              Parameter name: a
                   Modifiers: 0
                Is implicit?: false
            Is name present?: true
               Is synthetic?: false
             Parameter class: interface java.util.Collection
              Parameter name: c
                   Modifiers: 0
                Is implicit?: false
            Is name present?: true
               Is synthetic?: false

  - getType: 返回参数的声明类型的Class类
  - getName: 返回参数的名字。如果名字存在，则返回.class类文件返回的名称，否则该方法返回自动生成的名称，形式如argN，N是定义该参数的方法的参数的索引。
      例如，假如没有指定-parameters编译配置来编译类文件，则如下输出：

        public boolean ExampleMethods.simpleMethod(java.lang.String,int)
                 Return type: boolean
         Generic return type: boolean
             Parameter class: class java.lang.String
              Parameter name: arg0
                   Modifiers: 0
                Is implicit?: false
            Is name present?: false
               Is synthetic?: false
             Parameter class: int
              Parameter name: arg1
                   Modifiers: 0
                Is implicit?: false
            Is name present?: false
               Is synthetic?: false
```
  - getModifiers :返回形参具有的各种特征表示的整数，该数值是下列值的和，如果应用于形式参数：

Value (in decimal)	| Value (in hexadecimal) |	Description
-- | --|--
16	| 0x0010	| 形参定义为final
4096|	0x1000	| 形参是synthetic合成的. 或者可以调用方法isSynthetic.
32768|	0x8000	| 形参在源码中声明为隐式的，或者可以调用isImplicit方法。
  - isImplicit: 返回源码中声明的参数是否是隐性的

  - isNamePresent: 根据.class类文件来决定形参是否有一个名字，有返回true。
  - isSynthetic: 如果源码中声明的参数既不是隐性的也不是明确在源码定义的则返回true。

##### 隐性和合成参数
  某些构造函数是如果没有被显示声明则会在源码中隐性声明。如ExampleMethods例子无构造函数，一个默认构造函数就会隐性声明。MethodParameterSpy例子打印的隐性声明构造函数如下：
```
      Number of declared constructors: 1
      public ExampleMethods()
```
考虑如下片段：
```
      public class MethodParameterExamples {
        public class InnerClass { }
      }
```
InnerClass是一个非京台嵌套类或内部类。内部类的有个构造函数也是隐性声明的。然而，该隐性构造包含一个参数，当java编译器编译内部类时，会生成一个类似下面的：
```
      public class MethodParameterExamples {
        public class InnerClass {
            final MethodParameterExamples parent;
            InnerClass(final MethodParameterExamples this$0) {
                parent = this$0;
            }
        }
      }
```
InnerClass构造包含一个参数，参数类型是包含内部类InnerClass的类，即MethodParameterExamples。参照如下输出:
```
      public MethodParameterExamples$InnerClass(MethodParameterExamples)
             Parameter class: class MethodParameterExamples
              Parameter name: this$0
                   Modifiers: 32784
                Is implicit?: true
            Is name present?: true
               Is synthetic?: false
```
因为InnerClass类的构造函数是隐性指定的，所以参数也是隐性的。
注意：
Java编译器为内部类的构造函数创建一个形式参数，以使编译器能够将创建表达式中的引用（表示直接包含的实例）传递给成员类的构造函数。值32784表示InnerClass构造函数的参数同时是finla（16）和隐性的（32768）。Java语言允许变量名称含有$符号，但是约定来说，在变量名称中不用$符号。Java编译器提供的构造函数如果他们不能对应到源码里明确地或隐含的构造函数，则就是是synthetic合成的，除非他们是类初始化方法。合成的构造函数根据不同的编译器实现而不同。考虑如下：
```
      public class MethodParameterExamples {
        enum Colors {
            RED, WHITE;
        }
      }
```
Java编译器针对该类会生成几个方法，兼容.class类文件结构，并提供enum构造预期的功能。如，Java编译器会创建一个类文件如下：
```
      final class Colors extends java.lang.Enum<Colors> {
        public final static Colors RED = new Colors("RED", 0);
        public final static Colors BLUE = new Colors("WHITE", 1);

        private final static values = new Colors[]{ RED, BLUE };

        private Colors(String name, int ordinal) {
            super(name, ordinal);
        }

        public static Colors[] values(){
            return values;
        }

        public static Colors valueOf(String name){
            return (Colors)java.lang.Enum.valueOf(Colors.class, name);
        }
      }
```
java编译器创建了三个构造和方法，为该枚举构造： Colors(String name, int ordinal), Colors[] values(), and Colors valueOf(String name). 方法values和valueOf是隐性声明的. 因此他们的形参名称也是隐性的。

枚举的Colors(String name, int ordinal)构造函数是一个默认构造函数，隐性声明。然而，该构造的形参（name和ordinal）则是非隐性声明的，因为这些形参既不是明确的，也不是隐性的，是合成的。 (一个枚举构造的默认构造的形参不是隐性声明的，因为不同的编译器构造的形式要求不一样；另一个java编译器可能指定不同的形参。当编译器编译使用了枚举常量的表达式时，他们仅仅依赖枚举构造的公共静态字段，不依赖构造函数及常量初始化的过程。）
因此：
```
        enum Colors:

        Number of constructors: 0

        Number of declared constructors: 1

        Declared constructor #1
        private MethodParameterExamples$Colors()
               Parameter class: class java.lang.String
                Parameter name: $enum$name
                     Modifiers: 4096
                  Is implicit?: false
              Is name present?: true
                 Is synthetic?: true
               Parameter class: int
                Parameter name: $enum$ordinal
                     Modifiers: 4096
                  Is implicit?: false
              Is name present?: true
                 Is synthetic?: true

        Number of methods: 2

        Method #1
        public static MethodParameterExamples$Colors[]
          MethodParameterExamples$Colors.values()
                   Return type: class [LMethodParameterExamples$Colors;
           Generic return type: class [LMethodParameterExamples$Colors;

        Method #2
        public static MethodParameterExamples$Colors
          MethodParameterExamples$Colors.valueOf(java.lang.String)
                   Return type: class MethodParameterExamples$Colors
           Generic return type: class MethodParameterExamples$Colors
               Parameter class: class java.lang.String
                Parameter name: name
                     Modifiers: 32768
                  Is implicit?: true
              Is name present?: true
                 Is synthetic?: false
```
#### 获取&解析方法修饰符
有几个可能成为方法声明的几个修饰符：

  - 访问修饰符: public, protected, and private
  - 限制到一个实例的修饰符： static
  - 禁止改变值的修饰符：final
  - 需要覆盖的修饰符： abstract
  - 组织可重入的修饰符：synchronized
  - 表示用另一种语言实现的修饰符: native
  - 强制执行严格的浮点行为: strictfp
  - 注解：Annotations

MethodModifierSpy例子展示的指定方法的修饰符，及方法是否是编译器生成（synthetic），是否包含可变参数，是否是桥接方法（编译器生成来支持通用接口).
```
        import java.lang.reflect.Method;
        import java.lang.reflect.Modifier;
        import static java.lang.System.out;

        public class MethodModifierSpy {

            private static int count;
            private static synchronized void inc() { count++; }
            private static synchronized int cnt() { return count; }

            public static void main(String... args) {
            	try {
            	    Class<?> c = Class.forName(args[0]);
            	    Method[] allMethods = c.getDeclaredMethods();
            	    for (Method m : allMethods) {
            		if (!m.getName().equals(args[1])) {
            		    continue;
            		}
            		out.format("%s%n", m.toGenericString());
            		out.format("  Modifiers:  %s%n",
            			   Modifier.toString(m.getModifiers()));
            		out.format("  [ synthetic=%-5b var_args=%-5b bridge=%-5b ]%n",
            			   m.isSynthetic(), m.isVarArgs(), m.isBridge());
            		inc();
            	    }
            	    out.format("%d matching overload%s found%n", cnt(),
            		       (cnt() == 1 ? "" : "s"));

                    // production code should handle this exception more gracefully
            	} catch (ClassNotFoundException x) {
            	    x.printStackTrace();
            	}
            }
        }

      $ java MethodModifierSpy java.lang.Object wait
      public final void java.lang.Object.wait() throws java.lang.InterruptedException
        Modifiers:  public final
        [ synthetic=false var_args=false bridge=false ]
      public final void java.lang.Object.wait(long,int)
        throws java.lang.InterruptedException
        Modifiers:  public final
        [ synthetic=false var_args=false bridge=false ]
      public final native void java.lang.Object.wait(long)
        throws java.lang.InterruptedException
        Modifiers:  public final native
        [ synthetic=false var_args=false bridge=false ]
      3 matching overloads found

      $ java MethodModifierSpy java.lang.StrictMath toRadians
      public static double java.lang.StrictMath.toRadians(double)
        Modifiers:  public static strictfp
        [ synthetic=false var_args=false bridge=false ]
      1 matching overload found
      $ java MethodModifierSpy MethodModifierSpy inc
      private synchronized void MethodModifierSpy.inc()
        Modifiers: private synchronized
        [ synthetic=false var_args=false bridge=false ]
      1 matching overload found

      $ java MethodModifierSpy java.lang.Class getConstructor
      public java.lang.reflect.Constructor<T> java.lang.Class.getConstructor
        (java.lang.Class<T>[]) throws java.lang.NoSuchMethodException,
        java.lang.SecurityException
        Modifiers: public transient
        [ synthetic=false var_args=true bridge=false ]
      1 matching overload found

      $ java MethodModifierSpy java.lang.String compareTo
      public int java.lang.String.compareTo(java.lang.String)
        Modifiers: public
        [ synthetic=false var_args=false bridge=false ]
      public int java.lang.String.compareTo(java.lang.Object)
        Modifiers: public volatile
        [ synthetic=true  var_args=false bridge=true  ]

        2 matching overloads found
```
注意对Class.getConstructor()执行Method.isVarArgs()返回true，这因为该方法声明如下：
```
      public Constructor<T> getConstructor(Class<?>... parameterTypes)
```
而非：
```
      public Constructor<T> getConstructor(Class<?> [] parameterTypes)
```
注意String.compareTo()方法的输出有2个方法，一个是String.java声明的：
```
      public int compareTo(String anotherString);
```      
另一个是编译器生成的桥接方法或合成方法。这种情况是因为String事先了参数化泛型接口Comparable。在类型擦除时，继承的方法Comparable.compareTo()的参数类型从java.lang.Object变为java.lang.String。由于Comparable接口中的compareTo方法的参数化类型和String的方法在类型擦除后，不在匹配，不存在覆盖overriding。在所有的情形中，这会产生一个编译错误，因为接口没有被实现。桥接方法的作用就是避免这种问题。

Method实现了java.lang.reflect.AnnotatedElement，所以保留策略为运行时的注解java.lang.annotation.RetentionPolicy.RUNTIME都能够获取到.
#### 调用方法
反射提供了执行类的方法的手段。通常，反射调用方法仅仅当在非反射代码里不可能转化类的实例到指定类型时才是必须的。通过java.lang.reflect.Method.invoke()可以执行方法的调用，第一个参数表示执行哪一个实例的指定方法（如果方法是静态的，第一个参数应该是null），第二个参数表示方法执行所需的参数。如果底层方法抛出异常，会被java.lang.reflect.InvocationTargetException包装。方法的原始异常可以通过异常链机制的InvocationTargetException.getCause()方法获取。

##### 找到&执行指定声明的方法
```
        import java.lang.reflect.InvocationTargetException;
        import java.lang.reflect.Method;
        import java.lang.reflect.Type;
        import java.util.Locale;
        import static java.lang.System.out;
        import static java.lang.System.err;

        public class Deet<T> {
          private boolean testDeet(Locale l) {
          	// getISO3Language() may throw a MissingResourceException
          	out.format("Locale = %s, ISO Language Code = %s%n", l.getDisplayName(), l.getISO3Language());
          	return true;
          }

          private int testFoo(Locale l) { return 0; }
          private boolean testBar() { return true; }

          public static void main(String... args) {
        	if (args.length != 4) {
        	    err.format("Usage: java Deet <classname> <langauge> <country> <variant>%n");
        	    return;
        	}

        	try {
        	    Class<?> c = Class.forName(args[0]);
        	    Object t = c.newInstance();

        	    Method[] allMethods = c.getDeclaredMethods();
        	    for (Method m : allMethods) {
        		String mname = m.getName();
        		if (!mname.startsWith("test")
        		    || (m.getGenericReturnType() != boolean.class)) {
        		    continue;
        		}
         		Type[] pType = m.getGenericParameterTypes();
         		if ((pType.length != 1)
        		    || Locale.class.isAssignableFrom(pType[0].getClass())) {
         		    continue;
         		}

        		out.format("invoking %s()%n", mname);
        		try {
        		    m.setAccessible(true);
        		    Object o = m.invoke(t, new Locale(args[1], args[2], args[3]));
        		    out.format("%s() returned %b%n", mname, (Boolean) o);

        		// Handle any exceptions thrown by method to be invoked.
        		} catch (InvocationTargetException x) {
        		    Throwable cause = x.getCause();
        		    err.format("invocation of %s failed: %s%n",
        			       mname, cause.getMessage());
        		}
        	    }

                // production code should handle these exceptions more gracefully
        	} catch (ClassNotFoundException x) {
        	    x.printStackTrace();
        	} catch (InstantiationException x) {
        	    x.printStackTrace();
        	} catch (IllegalAccessException x) {
        	    x.printStackTrace();
        	}
          }
        }
```

Deet invokes getDeclaredMethods() which will return all methods explicitly declared in the class. Also, Class.isAssignableFrom() is used to determine whether the parameters of the located method are compatible with the desired invocation. Technically the code could have tested whether the following statement is true since Locale is final:

Locale.class == pType[0].getClass()
However, Class.isAssignableFrom() is more general.

$ java Deet Deet ja JP JP
invoking testDeet()
Locale = Japanese (Japan,JP),
ISO Language Code = jpn
testDeet() returned true
$ java Deet Deet xx XX XX
invoking testDeet()
invocation of testDeet failed:
Couldn't find 3-letter language code for xx
First, note that only testDeet() meets the declaration restrictions enforced by the code. Next, when testDeet() is passed an invalid argument it throws an unchecked java.util.MissingResourceException. In reflection, there is no distinction in the handling of checked versus unchecked exceptions. They are all wrapped in an InvocationTargetException

Invoking Methods with a Variable Number of Arguments

Method.invoke() may be used to pass a variable number of arguments to a method. The key concept to understand is that methods of variable arity are implemented as if the variable arguments are packed in an array.

The InvokeMain example illustrates how to invoke the main() entry point in any class and pass a set of arguments determined at runtime.


import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.Arrays;

public class InvokeMain {
    public static void main(String... args) {
	try {
	    Class<?> c = Class.forName(args[0]);
	    Class[] argTypes = new Class[] { String[].class };
	    Method main = c.getDeclaredMethod("main", argTypes);
  	    String[] mainArgs = Arrays.copyOfRange(args, 1, args.length);
	    System.out.format("invoking %s.main()%n", c.getName());
	    main.invoke(null, (Object)mainArgs);

        // production code should handle these exceptions more gracefully
	} catch (ClassNotFoundException x) {
	    x.printStackTrace();
	} catch (NoSuchMethodException x) {
	    x.printStackTrace();
	} catch (IllegalAccessException x) {
	    x.printStackTrace();
	} catch (InvocationTargetException x) {
	    x.printStackTrace();
	}
    }
}
First, to find the main() method the code searches for a class with the name "main" with a single parameter that is an array of String Since main() is static, null is the first argument to Method.invoke(). The second argument is the array of arguments to be passed.

$ java InvokeMain Deet Deet ja JP JP
invoking Deet.main()
invoking testDeet()
Locale = Japanese (Japan,JP),
ISO Language Code = jpn
testDeet() returned true

#### Troubleshooting covers common errors encountered when finding or invoking methods

### 构造函数

The Reflection APIs for constructors are defined in java.lang.reflect.Constructor and are similar to those for methods, with two major exceptions: first, constructors have no return values; second, the invocation of a constructor creates a new instance of an object for a given class.

#### Finding Constructors illustrates how to retrieve constructors with specific parameters
#### Retrieving and Parsing Constructor Modifiers shows how to obtain the modifiers of a constructor declaration and other information about the constructor
#### Creating New Class Instances shows how to instantiate an instance of an object by invoking its constructor
#### Troubleshooting describes common errors which may be encountered while finding or invoking constructors
