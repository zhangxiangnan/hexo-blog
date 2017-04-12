---
title: JDK动态代理
tags:
---

### 介绍
一个动态代理类是一个运行时实现一系列接口的类，这样对代理类的某个实例通过其实现的某个接口进行方法调用时，会被encoded并通过一个统一的接口派发到另一个对象。这样，一个动态代理类可用来为一系列接口创建一个类型安全地代理对象而不用预先生成代理类。对代理类实例的方法调用被派发到该实例的invocation handler的一个单独方法上，同时方法调用被加密encoded成为java.lang.reflect.Method对象，用来区分调用的方法，以及包含参数的Object类型数组。

动态代理类对一个需要对接口API提供类型安全地调用的应用程序或者类库来说有用。如，一个程序可以用动态代理类来创建实现了任意多事件监听接口的对象（这些接口都继承自java.Util.EventListener),这样就可以以一种统一的风格方式来处理不同种类的事件，例如记录所有的事件到文件。

**动态代理之所以动态是因为JDK在运行时生成代理类，而非提前写好**

### JDK动态代理
Proxy类为创建动态代理类（proxy classes）和实例提供了静态方法，并且Proxy类也是所有通过这些方法生成的动态代理类的父类。
为某个接口Foo，创建代理如下：
    // 定义一个调用处理器
     InvocationHandler handler = new MyInvocationHandler(...);
     // 生成代理类
     Class proxyClass = Proxy.getProxyClass(
         Foo.class.getClassLoader(), new Class[] { Foo.class });
     Foo f = (Foo) proxyClass.
         getConstructor(new Class[] { InvocationHandler.class }).
         newInstance(new Object[] { handler });

简化:
     // 生成代理实例
     Foo proxyFooInstance = (Foo) Proxy.newProxyInstance(Foo.class.getClassLoader(),
                                          new Class[] { Foo.class },
                                          handler);
### 代理类生成过程
  如何查询生成的代理类？
  **-Dsun.misc.ProxyGenerator.saveGeneratedFiles=true 加到JVM启动参数中**，将JDK动态生成的字节码保存到硬盘中
  代理类生成过程有两步：
    - 代理类字节码生成（内部有缓存，先从缓存取，无则生成）
    - 使用传递的类加载器加载字节码到虚拟机中  

### 动态代理类API
一个动态代理类（以下简称代理类）是一个在运行时当其被创建时便被指定要实现一系列接口的一个类。
一个代理接口是指被代理类实现的一个接口。
一个代理实例是代理类的一个实例对象。每一个代理实例有相关联的invocation handler对象, 该对象实现了InvocationHandler接口.

在一个代理实例上通过它的代理接口触发的方法调用将派发到该代理实例的invocation handler的invoke方法；该invoke方法需要传递代理实例、一个java.lang.Method对象用来识别被调用的方法，以及一个包括参数的Object数组。invocation handler适当地处理方法调用并将返回结果作为代理实例方法调用的返回结果。

一个代理类有以下属性：
 - 代理类是public、final、非抽象
 - The unqualified name of a proxy class is unspecified，但为代理类保留以字符串“$Proxy”开头的类命名空间。
 - 代理类继承自java.lang.Proxy类
 - 代理类准确地实现创建时指定的一个或多个接口，和指定的接口顺序一致
 - 如果一个代理类实现了一个非public接口，那么代理类将定义在与该接口相同的包中。否则代理类的包也是未指定的。注意包的封闭性不会阻止代理类在运行时被成功创建在指定包中，Note that package sealing will not prevent a proxy class from being successfully defined in a particular package at runtime, and neither will classes already defined by the same class loader and the same package with particular signers.
 - 由于代理类实现了运行时指定的所有接口，对代理类的Class object指定getInterfaces方法将返回包含相同接口的数组（和运行时指定的顺序一样），对代理类的Class object执行getMethods方法将返回所有接口里的所有方法的Method对象的数组，getMethod方法将在代理接口里查找期望的方法。
 - Proxy.isProxyClass方法如果传入代理类返回true，代理类指Proxy.getProxyClass返回的类或者Proxy.newProxyInstance返回的实例的类字节码，否则返回false。
 - 代理类的java.security.ProtectionDomaino和boostrap类加载器加载的system classes系统类一样，如java.lang.Object类，因为代理类的代码是通过受信赖的系统代码生成的。protection domain保护域通常通过java.security.AllPermission授予。
 - 每一个代理类有一个公共的构造函数，有一个参数，类型为接口InvocationHandler的实现类，来为代理实例设置invocation handler。2种方式来实例化代理实例，一种是利用反射API来访问公共构造函数来构造，另一种是调用Proxy.newProxyInstance方法，该方法将Proxy.getProxyClass方法调用和调用构造函数传递一个invocation handler结合起来。

### 创建代理实例
每一个代理类有一个公共的构造函数，有一个参数，类型为接口InvocationHandler的实现类，来为代理实例设置invocation handler。2种方式来实例化代理实例，一种是利用反射API来访问公共构造函数来构造，另一种是调用Proxy.newProxyInstance方法，该方法将Proxy.getProxyClass方法调用和调用构造函数传递一个invocation handler结合起来。

### 代理实例有以下属性：
  - 给定一个代理实例proxy和一个代理类实现的接口中的一个Foo，以下表达式返回true：
     proxy instanceof Foo
以下转换操作会成功（不会抛出ClassCastException）：
     (Foo) proxy
  - 静态Proxy.getInvocationHandler方法返回传递的代理实例参数相关联的invocation handler。如果传递的参数不是代理实例，报参数异常。
  - 对代理实例的接口方法调用会被encoded并派发到invocation handler的invoke方法：
    - 代理实例本身被传递到invoke方法的第一个参数，类型为Object
    - 传递给invoke的第二个参数为java.lang.reflect.Method实例，对应调用的代理实例的接口方法。Method object的声明类是声明该方法的接口，它可以是代理类继承方法的代理接口的父类接口。
    - 第三个参数是传递给代理实例方法的参数，为object类型的数组，基本类型参数会被装箱为包装类型，如java.lang.Integer or java.lang.Boolean. invoke方法的实现类可以自由修改该数组的内容。
    - invoke方法返回的值会当做对代理实例方法调用的返回值。如果接口方法声明的返回类型是基本类型，那么invoke方法返回的类型必须是对应包装类型的一个实例；否则必须可分配给声明的返回类型。如果invoke方法返回null，同时接口返回返回类型是基本类型，对代理实例的方法调用会抛出一个空指针异常。如果invoke方法返回值和方法声明的返回类型不兼容，代理实例会抛出ClassCastException。
    - 如果invoke方法抛出异常，代理实例的方法调用也会抛出该异常。异常类型必须是可分配给接口方法声明的任一异常类型或非受检异常java.lang.RuntimeException或java.lang.Error.
    - 如果invoke方法抛出一个不能分配给接口方法通过throws声明的异常类型中任一，对代理实例的方法调用会抛出UndeclaredThrowableException（通过invoke方法抛出的异常进行构造）
  - 每一个代理实例都有一个相关联的invocation handler，通过代理类的构造函数传递。静态Proxy.getInvocationHandler方法会返回通过参数传递和代理实例相关联的invocation handler
  - 针对代理实例的某个接口方法的调用会encoded并派发到invocation handler的invoke方法。    
  - 针对代理实例调用java.lang.Object类里声明的hashCode、equals、或toString方法会被encoded并派发到invocation handler的invoke方法，和接口方法调用的方法一样；传递去调用的Method对象的声明类是java.lang.Object类。其他代理实例的继承自java.lang.Object的public方法没有被代理类被覆盖，所有这些方法的调用表现行为和java.lang.Object的实例一样。

### 多个代理接口中重复的方法
当代理类的两个或更多接口里包含相同的方法名称和参数签名，代理类实现的接口的顺序变得很重要。当调用代理实例的这样一个重复的方法时，传递到invocation handler的Method对象不一定是代理的方法被调用的接口引用类型所分配的声明类。该限制存在是因为代理类里对应的方法实现不能确定调用哪一个接口。因此，当调用代理实例上的某个重复方法时，代理类实现的接口列表中包含该方法（直接或通过继承包含）的第一个接口中的与该重复方法相对应的Method的对象被传递到invocation handler对象的invoke方法里，不论方法调用发生时的引用类型。

如果一个代理接口包含和java.lang.Object类里的hashCode、equals、或toString方法相同的方法名称和参数签名时，调用代理实例的这样一个方法，传递给invocation handler的Method对象将把java.lang.Object当做声明类。即，在决定哪一个Method对象传递给invocation handler时，java.lang.Object类的public、非final方法逻辑优先于所有的代理接口。

记住当一个重复方法被派发到invocation handler时，invoke方法可能仅仅抛出受检异常，该受检异常可分配给它可以调用的所有代理接口的throws子句异常类型的一个。如果invoke方法抛出的受检异常不属于它可调用的代理接口列表中的一个接口声明的异常类型，则对代理实例的调用会抛出非受检异常UndeclaredThrowableException。这个限制意味着对传递给invoke方法的Method对象调用getExceptionTypes方法返回的异常类型中，并不是所有的异常都可以通过invoke方法被成功抛出。

### 创建代理类
代理类和代理类的实例都是通过java.lang.reflect.Proxy的静态方法创建。
Proxy.getProxyClass方法返回给定class loader和接口数组定义的java.lang.Class。代理类将使用指定的类加载器加载，且会实现所有提供的接口。如果类加载器里已有实现了相同顺序的接口列表，则直接返回已有的代理类；否则就会动态生成实现了这些接口列表的代理类，并用该类加载器加载。

针对传递给Proxy.getProxyClass的参数有几个限制：
  - 接口列表的所有Class必须为接口，不能是一般class或者基本类型。
  - 接口列表里的元素不能含有重复一样的接口，即接口class不能一样
  - 所有的接口类型必须通过类名对指定类加载器可见，即，对每个类加载器cl和每个接口i，如下表达式必须成立：
        Class.forName(i.getName(), false, cl) == i
  - 所有非公共public的接口必须在一个相同的包里；否则代理类不可能实现所有接口，不管代理类定义在哪个包.
  - 针对有相同签名的指定接口的任意成员方法集合：
    - 如果任意一个方法的返回类型是基本类型primitive或void类型，那么所有方法必须有相同的返回类型
    - 否则，其中一个方法必须有一个返回类型，该返回类型可当做其他方法的返回类型。
  - 生成的代理类不能超过虚拟机加载类的任何限制，如VM限制一个类的接口数量限制为65535；在这种情况下，接口数组大小不能超过65535.

如果任意一个限制触发，Proxy.getProxyClass抛出IllegalArgumentException，若接口数组参数或任一元素为null，NullPointerException抛出
记住指定代理接口列表的顺序是很重要的：接口顺序不同会生成不同的代理类。代理类是通过代理接口的顺序来区分的，以便于当两个或多个代理接口某个方法有相同的名称和参数签名时提供确定性方法调用encoding。

所以一个新的代理类当每次调用Proxy.getProxyClass（传递相同的类加载器和接口列表）时不必须每次都动态生成一次代理类；动态代理类的API的实现应该维护生成代理类的缓存，通过对应的加载器和接口列表来映射，同时，实现应该小心，不要引用类加载器，接口和代理类，以便于类加载器及其所有类在适当时被垃圾回收。
### 例子一
    import org.junit.Test;

    import java.lang.reflect.InvocationHandler;
    import java.lang.reflect.Method;
    import java.lang.reflect.Proxy;

    `/**
     * Created by zhangxiangnan on 17/4/5.
     */`
    public class JdkDynamicProxy {
        interface HelloService {
            String hello(String name);
        }

        class HelloServiceImpl implements HelloService {
            public String hello(String name) {
                return "hello " + name;
            }
        }

        class HelloInvocationHandler implements InvocationHandler {
            private Object instance;

            public Object getInstance() {
                return instance;
            }

            public void setInstance(Object instance) {
                this.instance = instance;
            }

            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("调用前");
                Object result = null;
                try {
                    result = method.invoke(instance, args);
                    System.out.println(result);
                } catch (Exception e) {
                    e.printStackTrace();
                }
                System.out.println("调用后");
                return result;
            }
        }

        @Test
        public void test() {
            HelloService helloService = new HelloServiceImpl();// 被代理实例
            HelloInvocationHandler helloInvocationHandler = new HelloInvocationHandler();
            //代理实例
            HelloService proxyInstance = (HelloService) Proxy.newProxyInstance(helloService.getClass().getClassLoader(), helloService.getClass().getInterfaces(), helloInvocationHandler);
            // 设置调用处理器的被代理实例
            helloInvocationHandler.setInstance(helloService);
            proxyInstance.hello("jack");// 调用被代理实例
        }
    }
### 例子二
设想简化例子一，生成代理实例的代码和invoke的实现放到一个类里。

    public interface HelloService {
      String hello(String name);
    }

    public class HelloServiceImpl implements HelloService {
        public String hello(String name) {
            return "hello " + name;
        }
    }

    public class HelloInvocationHandler implements InvocationHandler {
        private Object target;

        public HelloInvocationHandler(Object target) {
            this.target = target;
        }

        public static Object newProxyInstance(HelloService target) {
            Object proxyInstance = Proxy.newProxyInstance(helloService.getClass().getClassLoader(), helloService.getClass().getInterfaces(), new HelloInvocationHandler(target));
            return proxyInstance;
        }

        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            System.out.println("调用前");
            Object result = null;
            try {
                result = method.invoke(target, args);
                System.out.println(result);
            } catch (Exception e) {
                e.printStackTrace();
            }
            System.out.println("调用后");
            return result;
        }
    }

    public class Client {
      public static void main(String[] args) {
          HelloService proxyInstance = (HelloService) HelloInvocationHandler.newProxyInstance(new HelloServiceImpl());
          proxyInstance.hello("jack");
      }
    }

### 例子三

    public interface Foo {
        Object bar(Object obj) throws BazException;
    }

    public class FooImpl implements Foo {
        Object bar(Object obj) throws BazException {
            // ...
        }
    }

    public class DebugProxy implements java.lang.reflect.InvocationHandler {

        private Object obj;

        public static Object newInstance(Object obj) {
            return java.lang.reflect.Proxy.newProxyInstance(
                obj.getClass().getClassLoader(),
                obj.getClass().getInterfaces(),
                new DebugProxy(obj));
        }

        private DebugProxy(Object obj) {
            this.obj = obj;
        }

        public Object invoke(Object proxy, Method m, Object[] args)
            throws Throwable
        {
            Object result;
            try {
                System.out.println("before method " + m.getName());
                result = m.invoke(obj, args);
            } catch (InvocationTargetException e) {
                throw e.getTargetException();
            } catch (Exception e) {
                throw new RuntimeException("unexpected invocation exception: " +
                                           e.getMessage());
            } finally {
                System.out.println("after method " + m.getName());
            }
            return result;
        }
    }

为Foo接口的某个实现类构造DebugProxy，调用某个方法：

    Foo foo = (Foo) DebugProxy.newInstance(new FooImpl());
    foo.bar(null);


以下是一个使用的调用处理器类（invocation handler），为继承自java.lang.Object的方法提供了默认的代理行为，并且实现了根据调用方法的接口来委托某些代理方法调用给不同的对象:

  public class Delegator implements InvocationHandler {

      // preloaded Method objects for the methods in java.lang.Object
      private static Method hashCodeMethod;
      private static Method equalsMethod;
      private static Method toStringMethod;
      static {
          try {
              hashCodeMethod = Object.class.getMethod("hashCode", null);
              equalsMethod =
                  Object.class.getMethod("equals", new Class[] { Object.class });
              toStringMethod = Object.class.getMethod("toString", null);
          } catch (NoSuchMethodException e) {
              throw new NoSuchMethodError(e.getMessage());
          }
      }

      private Class[] interfaces;
      private Object[] delegates;

      public Delegator(Class[] interfaces, Object[] delegates) {
          this.interfaces = (Class[]) interfaces.clone();
          this.delegates = (Object[]) delegates.clone();
      }

      public Object invoke(Object proxy, Method m, Object[] args)
          throws Throwable
      {
          Class declaringClass = m.getDeclaringClass();

          if (declaringClass == Object.class) {
              if (m.equals(hashCodeMethod)) {
                  return proxyHashCode(proxy);
              } else if (m.equals(equalsMethod)) {
                  return proxyEquals(proxy, args[0]);
              } else if (m.equals(toStringMethod)) {
                  return proxyToString(proxy);
              } else {
                  throw new InternalError(
                      "unexpected Object method dispatched: " + m);
              }
          } else {
              for (int i = 0; i < interfaces.length; i++) {
                  if (declaringClass.isAssignableFrom(interfaces[i])) {
                      try {
                          return m.invoke(delegates[i], args);
                      } catch (InvocationTargetException e) {
                          throw e.getTargetException();
                      }
                  }
              }

              return invokeNotDelegated(proxy, m, args);
          }
      }

      protected Object invokeNotDelegated(Object proxy, Method m,
                                          Object[] args)
          throws Throwable
      {
          throw new InternalError("unexpected method dispatched: " + m);
      }

      protected Integer proxyHashCode(Object proxy) {
          return new Integer(System.identityHashCode(proxy));
      }

      protected Boolean proxyEquals(Object proxy, Object other) {
          return (proxy == other ? Boolean.TRUE : Boolean.FALSE);
      }

      protected String proxyToString(Object proxy) {
          return proxy.getClass().getName() + '@' +
              Integer.toHexString(proxy.hashCode());
      }
  }

委托器的子类可以覆盖invokeNotDelegated来实现代理方法调用的行为，而不是直接委托给其他对象，也可以覆盖proxyHashCode,proxyEquals，proxyToString来覆盖代理类继承自java.lang.Object的默认方法行为。

未Foo接口的某个实现累构造一个委托器Delegator：
    Class[] proxyInterfaces = new Class[] { Foo.class };
    Foo foo = (Foo) Proxy.newProxyInstance(Foo.class.getClassLoader(),
        proxyInterfaces,
        new Delegator(proxyInterfaces, new Object[] { new FooImpl() }));

注意上述Delegator的实现旨在说明问题，但性能可能不高；可以不使用缓存和比较hashCode、equals、toString方法的Methods对象，可以简单通过名字查找，因为java.lang.Object的这几个方法都没被重载。

### 几个问题
  - 实现接口数量有限制，65535
  - 生成的代理类重写了hashCode、toString、equals方法，调用这几个方法触发的是重写后逻辑（代理接口有这几个方法定义也是执行重写的方法），其他Object的方法是继承的OBject类的方法逻辑
  - 多个代理接口有重复的方法名，按实现的代理接口顺序，谁在前调用谁的方法；生成的代理类不会有重复的方法
  - 代理类的包名默认是com.sun.Proxy，若实现的接口是非public的，则和该接口同包；类名默认是$Proxy+一个自增值

### 反编译动态代理例子

  public final class $Proxy0 extends Proxy implements HelloService {
    private static Method m1;
    private static Method m2;
    private static Method m3;
    private static Method m0;

    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return ((Boolean)super.h.invoke(this, m1, new Object[]{var1})).booleanValue();
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final String hello(String var1) throws  {
        try {
            return (String)super.h.invoke(this, m3, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final int hashCode() throws  {
        try {
            return ((Integer)super.h.invoke(this, m0, (Object[])null)).intValue();
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[]{Class.forName("java.lang.Object")});
            m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
            m3 = Class.forName("cn.zxn.guava.proxy.HelloService").getMethod("hello", new Class[]{Class.forName("java.lang.String")});
            m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
  }

### 序列化
由于java.lang.reflect.Proxy实现了java.io.Serializable接口，代理实现可以被序列化。如果代理实例包含一个实现java.lang.Serializable接口的invocation handler，当写入到java.lang.ObjectOutputStream时就会抛出NotSerializableException异常。注意对于代理类来说，实现java.io.Externalizable跟实现java.io.Serializable有相同的效果：对于代理实例（或一个invocation handler）序列化过程来说，Externalizable的Externalizable和readExternal方法不会被调用。至于所有的Class objects，代理类的Class object总是可序列化的。

代理类没有可序列化的字段，serialVersionUID等于0L.即，当代理类的Class对象传给java.io.ObjectStreamClass的静态查找lookup方法时，返回的ObjectStreamClass实例有以下属性：

  - 调用getSerialVersionUID方法返回0L
  - 调用getFields方法返回空数组
  - 调用getField有参方法返回null

Object序列化的流协议支持一个叫做TC_PROXYCLASSDESC的类型code，这是流格式语法的结束符号；其类型和值由java.io.ObjectStreamConstants接口中的以下常量字段定义：
    final static byte TC_PROXYCLASSDESC = (byte)0x7D;
语法还包含了一下两个规则，第一个是原始newClassDesc规则的可替代扩展：

    newClassDesc:
            TC_PROXYCLASSDESC newHandle proxyClassDescInfo

    proxyClassDescInfo:
            (int)<count> proxyInterfaceName[count] classAnnotation superClassDesc

    proxyInterfaceName:
            (utf)
当一个ObjectOutputStream序列化某个代理类的class对象的类描述符时，通过将其Class对象传递给Proxy.isProxyClass方法确定，它使用TC_PROXYCLASSDESC类型代码而不是TC_CLASSDESC，遵循上面的规则。在proxyClassDescInfo的扩展中，proxyInterfaceName项的序列是所有代理类实现的接口的名称，与调用代理类的getInterfaces接口返回的接口描述符一致。classAnnotation和superClassDesc项与classDescInfo规则含义一样。对于一个代理类，superClassDesc是其超类的类描述符，java.lang.reflect.Proxy; including this descriptor allows for the evolution of the serialized representation of the class Proxy for proxy instances.

For non-proxy classes, ObjectOutputStream calls its protected annotateClass method to allow subclasses to write custom data to the stream for a particular class. For proxy classes, instead of annotateClass, the following method in java.io.ObjectOutputStream is called with the Class object for the proxy class:

    protected void annotateProxyClass(Class cl) throws IOException;
The default implementation of annotateProxyClass in ObjectOutputStream does nothing.

When an ObjectInputStream encounters the type code TC_PROXYCLASSDESC, it deserializes the class descriptor for a proxy class from the stream, formatted as described above. Instead of calling its resolveClass method to resolve the Class object for the class descriptor, the following method in java.io.ObjectInputStream is called:

    protected Class resolveProxyClass(String[] interfaces)
        throws IOException, ClassNotFoundException;
The list of interface names that were deserialized in the proxy class descriptor are passed as the interfaces argument to resolveProxyClass.

The default implementation of resolveProxyClass in ObjectInputStream returns the results of calling Proxy.getProxyClass with the list of Class objects for the interfaces named in the interfaces parameter. The Class object used for each interface name i is the value retuned by calling

        Class.forName(i, false, loader)
where loader is the first non-null class loader up the execution stack, or null if no non-null class loaders are on the stack. This is the same class loader choice made by the default behavior of the resolveClass method. This same value of loader is also the class loader passed to Proxy.getProxyClass. If Proxy.getProxyClass throws an IllegalArgumentException, resolveClass will throw a ClassNotFoundException containing the IllegalArgumentException.
Since a proxy class never has its own serializable fields, the classdata[] in the stream representation of a proxy instance consists wholly of the instance data for its superclass, java.lang.reflect.Proxy. Proxy has one serializable field, h, which contains the invocation handler for the proxy instance.
