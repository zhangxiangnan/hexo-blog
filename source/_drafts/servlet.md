---
title: servlet
tags:
---

### 背景
当网络开始用于提供服务时，服务提供商就认识到需要动态的web内容。 Applet是实现此目标的最早尝试之一，其重点是使用客户端平台提供动态用户体验。 同时，开发人员还调查了使用服务器平台来提供动态用户体验。 最初，通用网关接口（CGI）脚本是用于生成动态内容的主要技术。 CGI脚本技术虽然广泛应用，但具有平台依赖性和可扩展性不足等诸多缺点。 为了解决这些限制，Java Servlet技术被创建为一种便携式的方式来提供动态的面向用户的内容。

### servlet是什么
servlet是一种Java编程语言类，用于扩展通过请求 - 响应编程模型访问的托管应用程序的服务器的功能。尽管Servlet能够响应任何类型的请求，但是经常是用来扩展web服务器管理的应用程序。针对这样的程序，Java Servlet技术定义了HTTP特定的servlet类。

java.servlet和javax.servlet包提供用于编写servlet的接口和类。所有的servlet必须实现Servlet接口，该接口定义了生命周期的方法。当实现一个通用服务时，可以使用或扩展Java Servlet API提供的GenericServlet类。HttpServlet类提供了诸如doGet、doPost方法来处理特定于HTTP的服务。

### Servlet生命周期
servlet的生命周期是servlet部署的容器来控制的额。当一个请求映射到一个servlet时，容器执行如下步骤：

  - 如果该servlet的实例不存在，web容器则执行如下操作：

    - 加载servlet类

    - 创建该servlet类的一个实例

    - 通过调用init方法来初始化该servlet实例

  - 容器调用service方法，传递请求&响应对象。
如果需要移除servlet，容器通过调用servlet的destroy方法来销毁servlet。

#### 处理Servlet生命周期事件
在servlet的生命周期中，可以通过定义listener对象来监控和对事件作出响应反应，这些linstener的方法会在生命周期事件发生时被调用。想要使用listener对象，必须先定义&指定listener类。

##### 定义监听类Listener
可以定义一个监听器类来实现监听接口。下表列出了可以监控的事件及对应必须实现的接口。当监听器方法被调用时，会传入一个包含对应事件信息的事件对象。如，HttpSessionListener接口传入一个HttpSessionEvent，包含了HttpSession。

Servlet生命周期事件

对象|	事件	| 监听接口和事件类
-- | -- | --
Web上下文（web context） | 初始化&销毁（Initialization and destruction）| javax.servlet.ServletContextListener & ServletContextEvent
Web上下文（web context） | 属性添加、移除、替换 | javax.servlet.ServletContextAttributeListener and ServletContextAttributeEvent
Session |创建&无效&激活&钝化&超时（Creation, invalidation, activation, passivation, and timeout）|javax.servlet.http.HttpSessionListener, javax.servlet.http.HttpSessionActivationListener, and HttpSessionEvent
Session |  属性添加&移除&替换 |javax.servlet.http.HttpSessionAttributeListener and HttpSessionBindingEvent
Request | 一个servlet请求已经被web组件处理|javax.servlet.ServletRequestListener and ServletRequestEvent
Request | 属性添加&移除&替换 | javax.servlet.ServletRequestAttributeListener and ServletRequestAttributeEvent

使用@WebListener注解来定义一个监听器获取指定web程序上下文上的不同的操作事件。使用该注解的类必须实现其中一个下列接口：

      javax.servlet.ServletContextListener
      javax.servlet.ServletContextAttributeListener
      javax.servlet.ServletRequestListener
      javax.servlet.ServletRequestAttributeListener
      javax.servlet..http.HttpSessionListener
      javax.servlet..http.HttpSessionAttributeListener
如下：

      import javax.servlet.ServletContextAttributeListener;
      import javax.servlet.ServletContextListener;
      import javax.servlet.annotation.WebListener;

      @WebListener()
      public class SimpleServletListener implements ServletContextListener,
              ServletContextAttributeListener {
          ...

####处理Servlet错误
当servlet执行时，可能会发生任何数量的异常。 发生异常时，Web容器生成一个包含以下消息的默认页面：

      A Servlet Exception Has Occurred
但也可以指定容器应该返回一个给定异常对应一个指定的错误页面。

### 共享信息
web组件，类似大多数对象，也通常是和其他对象配合来完成任务。web组件可以和以下对象交互配合使用：
  - 使用私有工具对象 (如JavaBeans组件).
  - 共享作为公共范围属性的对象
  - 使用数据库
  - 调用其他web资源

#### 使用范围对象
协作的web组件通过作为四个作用域对象的属性维护的对象来共享信息。可以通过代表这些作用域的类的getAttribute和setAttribute方法来访问这些属性。如下表：

范围对象

Scope Object	|Class	|Accessible From
-- | -- | --
web上下文Web context | javax.servlet.ServletContext | web上下文里的web组件
Session| javax.servlet.http.HttpSession| 处理属于session的请求的web组件
Request|javax.servlet.ServletRequest的子类|处理请求的web组件
Page|javax.servlet.jsp.JspContext| 创建对象的JSP页面The JSP page that creates the object.

#### 控制共享资源的并发访问
在多线程服务器上，共享资源被并发访问。除了范围对象的属性，共享资源还包括内存数据如某个实例或类变量，及外部对象如文件&数据库连接&网络连接.

并发访问可能会在以下几种情况出现：

- 多个web组件同时访问存储到web上下文中的对象。
- 多个web组件访问session中的对象。
- 一个web组件里的多个线程访问某个实例变量.一个web容器针对每一个请求会通常创建一个线程来处理。想要确保一个servlet实例同一时间只能处理一个请求的话，可以使servlet实现SingleThreadModel接口。如果servlet实现了该接口，这样就不会有2个线程同时并发执行servlet的service方法。Web容器可以通过同步对servlet的单个实例的访问或通过维护Web组件实例池并将每个新请求分派给空闲实例来实现此保证。 此接口不会阻止Web组件访问共享资源（如静态类变量或外部对象）导致的同步问题。

当资源可被并发访问的时候，就可能以不一致的方式使用，可以通过同步技术来控制访问的并发导致的不一致。

### 创建&初始化Servlet
使用@WebServlet注解来定义web应用程序的一个servlet组件。该注解加到类上，包含关于正在声明的servlet的元数据信息。通过注解声明的servlet必须制定至少一个URL模式，这通过使用注解的urlPatterns或value属性来完成。其他所有的属性都是可选的，有默认值。当注解上的唯一一个属性是URL pattern时，使用value属性；否则当其他属性也使用时，使用urlPatterns属性。
使用@WebServlet注解的类必须继承自javax.servlet.http.HttpServlet类，如下示例：

      import javax.servlet.annotation.WebServlet;
      import javax.servlet.http.HttpServlet;

      @WebServlet("/report")
      public class MoodServlet extends HttpServlet {
          ...
Web容器在加载和实例化servlet类之后，在传递来自客户端的请求之前初始化一个servlet。如果想要自定义流程让servlet读取持久化配置数据、初始化资源、执行其他一次性操作，可以通过覆盖servlet的init方法，或指定@WebServlet注解的initParams属性。initParams属性包含一个@WebInitParam注解，如果Servlet初始化有误，则会抛出UnavailableException。

针对某个指定的servlet提供其需要的数据可以使用初始化initParams参数，对比之下，上下文参数context parameter提供的数据web应用程序的所有组件都能访问。

### 编写Service方法
servlet提供的service方法被GenericServlet的service方法实现，被HttpServlet对象的doMethod方法（Method可以是Get、Delete、Options、POST、PUT、Trace），或者是实现了Servlet接口的类定义的其他任何指定协议的方法。 service方法可以用于提供给客户某个服务的servlet类的任何方法。

一个service方法的一般模式是从请求里提取信息，访问外部资源，然后基于该信息填充响应结果。对于HTTP servlets，填充响应的正确步骤是如下过程：
  - 从响应respoinse里获取输出流
  - 填充response headers
  - 写任何主体内容到输出流
响应头部（response headers）必须在响应提交之前设置。因为web容器在响应提交之后，会忽略任何设置或添加headers头部的操作。

#### 从请求里获取信息
一个请求里包含了在客户端client和servlet之间传递的消息。所有的请求都实现了ServletRequest接口，该接口定义了访问如下信息的方法：
  - 参数，通常用来在客户端和servlet之间传递信息
  - 对象值属性，通常用来在web容器和servlet之间或协作的servlet之间传递信息
  - 关于用于传达请求的协议以及有关请求中涉及的客户端和服务器的信息
  - 与本地化有关的信息

也可以从请求里获取输入流，然后手动解析数据；要是读取的是字符数据，就使用request对象的getReader方法返回的BufferedReader对象，要是读取二进制数据，就使用getInputStream方法返回的ServletInputStream对象。

HTTP servlets传递一个HTTP请求对象，HttpServletRequest，包含了请求URL、HTTP headers、查询字符串query string等。一个HTTP请求URL包含以下部分：

      http://[host]:[port][request-path]?[query-string]
请求路径进一步由以下元素组成：
  - 上下文路径Context path: "/" + servlet所属web应用程序的上下文根。
  - Servlet路径Servlet path: "/" + 触发该请求的组件别名
  - 路径信息Path info: 请求路径中不属于上下文路径或servlet路径的部分
可以用HttpServletRequest接口定义的的getContextPath、getServletPath、getPathInfo方法来访问这些信息。除了请求URI和路径部分的URL编码区别外，请求URI始终是由上下文路径 + servlet路径 + 路径信息组成。

查询字符串Query strings由一系列参数和值组成，可以用getParameter方法来检索各个参数，有2种方式来生成查询字符串：
  - 查询字符串可以显式地显示在网页中。
  - 当提交GET HTTP方法的表单时，会将查询字符串添加到URL后面。

#### 构建响应
响应包含了服务器和客户端之间传递的数据，所有的响应对象都实现了ServletResponse接口，该接口定义了做如下操作的方法：
  - 获取输出流用来发送数据到客户端。要是发送字符流，使用response的getWriter方法返回的PrintWriter对象；要是在多用途Internet扩展Multipurpose Internet Mail Extensions (MIME)的body响应里发送二进制数据，就使用getOutputStream返回的ServletOutPutStream；针对二进制和文本数据混合的情况，如在多部分响应中，就需要使用ServletOutputStream并手动管理字符部分。
  - 使用setContentType(String)方法来表明response返回的内容类型（如text/html)。该方法必须在response提交之前调用。内容类型名称的注册表由互联网号码分配机构（IANA）保存，网址为http://www.iana.org/assignments/media-types/。
  - 使用setBufferSize(int)方法来表明是否缓冲输出。默认时，写到输出流的任何内容都会立即发送到client端。采用缓冲允许在发送任何内容到客户端之前就写好内容，这样就让servlet有跟多时间来设置合适的状态码和header头部或转向另一个web资源。该方法必须在任何内容写之前或响应对象response提交之前调用。
  - 设置本地化信息，如locale和字符编码。

HTTP响应对象，javax.servlet.http.HttpServletResponse，有表示HTTP headers的字段，如下：
  - 状态码,用来表示请求未正确处理的原因或请求已经被重定向。
  - Cookies,用来在客户端保存应用程序指定的信息。有时，cookies使用维护的标识符来跟踪用户会话。

### 过滤请求和响应
  filter是一个可以转换请求或响应的headr和内容（或者两者）的对象。不同的web组件过滤器不同，因为过滤器通常本身不会产生一个响应。相反过滤器提供可以附加到任何类型的web资源的功能。因此，过滤器不应该依赖具有过滤器作用的web资源；这样，它可以由多种类型的web资源组成。
  过滤器可以执行的主要任务如下：
    - 查询请求并相应地执行
    - 阻止请求-响应对进一步传递
    - 修改请求头和请求数据，来提供一个自定义化版本的request请求。
    - 修改响应头和响应数据，来提供一个自定义化版本的response响应。
    - 和外部资源交互
过滤器的应用包括身份验证、日志、图片转换、数据压缩、加密、xml转换、标记流等。

可以配置零个、一个、以特定顺序的多个过滤器链来过滤某个web资源。过滤器链当包含该组件的web应用程序部署时被指定，当web容器加载该组件时被初始化。
#### 过滤器的使用
过滤器API通过javax.servlet包里的Filter、FilterChain、FilterConfig接口定义，通过实现Filter接口来定义一个过滤器。

使用@WebFilter注解来在web应用程序中定义一个过滤器，该注解加载类上，包含了声明的过滤器的元数据信息。注解的filter必须指定至少一个URL pattern，可以通过注解提供的urlPatterns或value属性来指定。所有的其他属性都是可选的，有默认值。当注解的仅有一个url pattern属性被指定的话，就用value属性；否则当其他属性也使用的话，就用urlPatterns属性。
使用@WebFilter注解的类必须实现javax.servlet.Filter接口.

@WebFilter注解initParams属性可以用来配置过滤器的配置信息。initParams属性包含一个@WebInitParam注解，示例如下：

      import javax.servlet.Filter;
      import javax.servlet.annotation.WebFilter;
      import javax.servlet.annotation.WebInitParam;

      @WebFilter(filterName = "TimeOfDayFilter",
      urlPatterns = {"/`*`"},
      initParams = {
          @WebInitParam(name = "mood", value = "awake")})
      public class TimeOfDayFilter implements Filter {
          ...

Filter接口最重要的方法是doFilter，该方法用来传递请求&响应&过滤链对象，该方法可以执行如下操作：
  - 检查请求头（request headers）
  - 如果过滤器想修改请求头或数据的话，自定义request对象
  - 如果过滤器想修改响应头或数据的话，自定义response对象
  - 执行过滤器链上的下一个过滤器。如果当前过滤器是以目标web组件或静态资源结束的链路上的最后一个过滤器，那么下一个实体就是链路末尾的资源；否则，就是WAR中配置的下一个过滤器。过滤器调用过滤器链chanin对象的doFilter方法来执行下一个实体，传入当前过滤器被调用时传入的request&response或它自定义创建的包装版本的reqeust&response。或者，过滤器也可以选择通过不进行调用下一个实体来阻止请求。在后一种情况下，过滤器负责填写响应。
  - 在调用链路上的下一个过滤器时，检查响应头
  - 抛出异常来表明处理过程发生异常

除了doFilter方法，还必须实现init、destroy方法。init方法是在过滤器初始化时容器调用的。如果传递初始化参数给过滤器，可以从传给init方法的FilterCofnig参数获取。

#### 编写自定义request&response
过滤器有很多方式来修改request&response。如，过滤器可以添加属性到request里或在响应里插入数据。
过滤器修改响应时，通常必须先在返回给客户端之前捕获响应。为了达到这一点，需要传递一个备用流给生成响应的servlet。备用流阻止servlet在完成时关闭原始的响应流并允许过滤器修改servlet的响应。
要将此备用流传递给servlet，过滤器将创建一个响应包装器，该包装器覆盖getWriter或getOutputStream方法以返回此备用流。 包装器被传递给过滤器链的doFilter方法。 包装方法默认调用到包装的请求或响应对象。
要覆盖request方法，包装请求request的对象需要继承ServletRequestWrapper或HttpServletRequestWrapper；要覆盖response的方法，包装响应的对象需要继承ServletResponseWrapper或HttpServletResponseWrapper类。

#### 指定过滤器映射
web容器通过过滤器映射来决定如何应用过滤器到web资源。过滤器映射将过滤器按照名称或web资源则按照URL模式匹配到web组件。
过滤器按照WAR中过滤器映射列表出现的过滤器映射的顺序被调用，可以通过修改WAR包的部署描述文件指定过滤器映射列表。
如果想要记录每一个访问web应用程序的请求，可以配置点击统计过滤器的URL模式为/`*`.

可以将一个过滤器映射到不止一个web资源，也可以将过个过滤器映射到一个web资源。

![Filter-to-Servlet Mapping](http://docs.oracle.com/javaee/7/tutorial/img/jeett_dt_018.png)

过滤器链是传递给过滤器doFilter方法的一个参数，该链通过过滤器链间接形成。过滤器链中过滤器的顺序和web应用程序部署描述文件中配置的映射顺序一致。
当一个过滤器映射到servlet S1，web容器就会调用F1的doFilter方法，S1的过滤器链中的每个过滤器的doFilter方法是通过过滤器链中的前一个过滤器调用chain.doFilter方法。因为S1的过滤器链包含过滤器F1、F3，F1调用chain.doFilter触发F3过滤器doFilter的方法调用。当F3的doFilter方法结束时，控制权交还给F1的doFilter方法。

#### 配置servlet过程
  - 打开项目的web.xml文件，配置filter节点相关信息（filter-name、filter-class、init-param等）
  - 配置filter-mapping节点信息，包括（filter-name、url-pattern、dispatcher）
  - dispatcher的值类型详解（该值可以使用如下多个任意组合使用，不指定默认是REQUEST类型）：
    - REQUEST: 仅仅当请求直接来自客户端，即过滤器应用到用户的发送到服务器servlet的原始请求。
    - ASYNC: 过滤器应用到用户发送到服务器的原始异步请求
    - FORWARD: 表示将过滤器应用到RequestDispatcher.forward()调用，即仅当请求被转发forwarded到一个组件时
    - INCLUDE: 仅仅当请求被引入的某个组件处理时，即过滤器应用到RequestDispatcher.include()方法的调用
    - ERROR: 过滤器应用到仅仅触发错误页面机制的请求

### 调用其他Web资源
  web组件可以直接或间接调用其他web资源。web组件可以通过嵌入返回给客户端的上下文中指向另一个web组件的URL地址来间接调用其他web资源，当这么做的时候，web组件就通过包含引入另一个资源的内容或将请求转发给另一个资源来直接调用其他资源。

  想调用运行某个web组件的服务器的可用资源，需要首先通过getRequestDispatcher("URL")方法来获取RequestDispatcher对象，可以通过某个请求或web上下文获取该对象；然而这两种方式有略微不同的行为。该方法需要一个到请求资源的路径作为参数，一个请求对象里有相对路径（即，路径不以/开始），但是web上下文需要方式需要绝对路径。如果资源不可访问或服务器的那个资源没有实现ReqeustDispatcher对象，那么getRequestDispatcher方法就会返回null。这种情形应该预先处理好。

  #### 在响应里包含其他资源
  web组件返回的响应里包含其他web资源也是非常有用的，如横幅内容或版权信息。要想包含其他资源，需要调用RequestDispatcher对象的include方法：

        include(request, response);
  如果要包含的资源是静态资源，include方法支持 programmatic server-side includes；如果该资源是某个web组件，该方法的作用就是发送请求给包含的web组件，执行该web组件，然后包含响应response里执行的结果。一个被包含的web组件有权限访问request对象但针对响应对象的处理是有限制的。
    - 可以编写内容到response的body体，然后提交响应response
    - 可以设置headers或调用任何影响response的headers的方法如setCoookie

#### 转移控制权到另一个web组件
某些程序中，可能需要让一个web组件对请求进行初步处理，然后让另一个组件生成响应。如可能想部分地处理一个请求，然后根据请求的性质转移给另一个组件。
要转移控制权到另一个web组件，需要调用RequestDispatcher的forward方法。当某个请求被转发forward时，请求URL设置为被转发页面的路径，原始的URI及其组成部分保存在以下属性中：

      javax.servlet.forward.request_uri
      javax.servlet.forward.context_path
      javax.servlet.forward.servlet_path
      javax.servlet.forward.path_info
      javax.servlet.forward.query_string
转发forward方法被调用时应该把生成响应信息的职责转交给转发的资源，假如你在servlet里已经在访问ServletOutputStream或PrintWriter对象，就不能使用forward转发方法，否则会抛出IllegalStateException异常。

### 访问web上下文
web组件执行的上下文是一个实现了ServletContext接口的对象，可以用getServletContext方法来获取web上下文，web上下文提供访问以下内容的方法：
  - 初始化参数Initialization parameters
  - 和上下文相关的资源
  - 对象值属性
  - 记录功能Logging capabilities

### 维护客户端状态
许多应用程序需要客户端的一系列请求能够相互关联，例如，应用程序可以在多个请求间保存购物车状态。基于web的应用程序有必要维护这样的状态，叫做session会话，因为HTTP是无状态的。为了支持需要维护状态的程序，JAVA Servlet技术提供了管理session会话的API并允许实现会话的几种机制。

#### 访问会话
会话是通过HttpSession对象表示。可以通过request对象的getSession方法来访问session，该方法返回和请求相关联的当前session对象；如果请求没有关联session，会闯将一个session。
#### 将对象和会话关联
可以通过名称将对象值属性和session关联起来，这样的属性就能被属于同一个web上下文并且处理相同会话的某个请求的任意web组件访问。
应用程序可以通知web上下文和会话监听器对象的servlet生命周期事件（处理servlet生命周期事件），也可以通知与某个会话关联的某些事件对象，如：
  - 当对象添加到会话或从会话移除时，要接收该通知，需要你的对象实现javax.servlet.http.HttpSessionBindingListener接口
  - 当连接对象的会话被钝化或激活时。当会话在虚拟机间移动或保存到永久存储器或从永久存储器恢复时，会话会被钝化或激活。要接收这些通知，需要实现javax.servlet.http.HttpSessionActivationListener接口。

#### 会话管理
  因为HTTP客户端没办法告知说它不再需要某个会话，所以每个会话都关联一个超时时间，为了回收其资源。超时时间周期可以通过会话的getMaxInactiveInterval和setMaxInactiveInterval访问。
  为了确保活动的会话不会超时，需要定期使用service方法访问会话，因为service方法重置了会话的生存时间计数器。
  当某个客户端交互操作完成时，使用session的invalidate方法来达到销毁服务端的会话，同时移除任何会话数据。
##### 设置session的会话超时时间
  编辑web工程的web.xml文件中的<session-config>下的<session-timeout>属性来设置超时时间,该超时时间指会话连续处于多少分钟不活动状态后，会话即超时销毁。

#### 会话跟踪
  为了关联会话到用户，web容器可以使用几个不同的方法，所有这些都涉及在客户端和服务器之间传递一个标识符。标识符可以是客户端维护放到cookie里，web组件也可以在返回给客户端的每个url都带有标识符。
  如果你的应用程序使用session对象，必须通过重写URLs来确保session跟踪可用，因为客户端可以随时禁用cookie，可以通过对servlet返回的所有URLS方法调用response的encodeURL(URL)方法来重写。该方法当cookie被禁用时会在URL里包含session ID，否则不对URL进行更改。

### 销毁servlet
web容器可以决定servlet何时应该从服务中移除（如，当web容器想回收内存资源或者当容器正在被关闭时）。在这些情形里，容器调用servlet接口的destroy方法。在该方法里，会释放servlet正在使用的所有资源，同时保存记录任何持久状态。destroy方法会释放init方法里创建的数据库对象。

当servlet被移除时，servlet的service方法应该所有都执行结束。服务器通过仅仅在所有的service请求已经返回或者经过一个服务器特定的宽限期后（以先到者为准）才会调用destroy方法来确保这一点。
如果servlet有运行时间比服务器的宽限期更长的操作，这些操作即时destroy方法被调用也会继续执行。必须确保仍然处理客户端请求的任意线程执行结束。
  - 持续跟踪当前运行service方法的线程数
  - 通过使用destroy方法通知长时间运行的线程关闭并等待他们结束来提供一个干净的关机过程。
  - 长时间运行的线程定期检查关机，如果要关机，就停止工作，清理，返回。

#### 跟踪service请求
  为了跟踪service请求，可以在servlet类里引入一个字段统计运行的线程数，该字段应该支持同步访问来增加、减少及返回值。

        public class ShutdownExample extends HttpServlet {
            private int serviceCounter = 0;
            ...
            // Access methods for serviceCounter
            protected synchronized void enteringServiceMethod() {
                serviceCounter++;
            }
            protected synchronized void leavingServiceMethod() {
                serviceCounter--;
            }
            protected synchronized int numServices() {
                return serviceCounter;
            }
        }

service应该每当其被调用时增加计数，每次方法返回时减少计数，新方法应该调用super.service来保留原始service方法的功能：

      protected void service(HttpServletRequest req,
                             HttpServletResponse resp)
                             throws ServletException,IOException {
          enteringServiceMethod();
          try {
              super.service(req, resp);
          } finally {
              leavingServiceMethod();
          }
      }

#### 通知方法
  为了确保干净的关闭过程，destroy方法禁止释放任何共享资源直到服务请求已经完成。一方面可以通过检查service计数器，另一方面可以通知长时间运行的方法该关闭了。为了通知其关闭，需要另一个字段：

    public class ShutdownExample extends HttpServlet {
        private boolean shuttingDown;
        ...
        //Access methods for shuttingDown
        protected synchronized void setShuttingDown(boolean flag) {
            shuttingDown = flag;
        }
        protected synchronized boolean isShuttingDown() {
            return shuttingDown;
        }
    }

  使用这几个字段提供干净的关闭过程的例子：

  public void destroy() {
      // 检查是否仍有线程执行service方法
      // 有的话，通知其他关闭停止
      if (numServices()> 0) {
          setShuttingDown(true);
      }

      // 等待service方法结束
      while (numServices()> 0) {
          try {
              Thread.sleep(interval);
          } catch (InterruptedException e) {
          }
      }
  }

#### 创建优雅的长时间运行的方法
  提供干净的关闭过程的最后一步就是让长时间运行的方法表现的优雅。运行长时间的方法应该检查标识通知他们关闭的字段的值，如果有必要就中断工作：
      public void doPost(...) {
          ...
          for(i = 0; ((i < lotsOfStuffToDo) &&
               !isShuttingDown()); i++) {
              try {
                  partOfLongRunningOperation(i);
              } catch (InterruptedException e) {
                  ...
              }
          }
      }
### 使用JAVA Servlet技术上传文件
对于许多web应用程序来说支持文件上传是一个非常基本和常见的需求。在以前版本的servlet规范中,实现文件上传需要使用依赖外部库或完成复杂的输入处理。Java Servlet规范现在有助于提供可行的、通用、便携的方案。Java servlet技术现在支持文件上传的开箱即用，所以任意实现了该规范的web容器都能够解析multipart请求，并通过HttpServletRequest对象使mime附件可用。
一个新的注解，javax.servlet.annotation.MultipartConfig用来标明该servlet接收multipart/form-data MIME类型的请求。@MultipartConfig注解的servlet通过调用request.getPart(String name)或request.getParts()方法能够获取一个指定multipart/form-data请求的部分组件。

#### @MultipartConfig注解
  @MultipartConfig注解支持以下可选属性：
    - location: 执行文件系统目录的绝对路径,不支持相对于应用程序上下文的路径。此位置用于当文件正在被处理货文件大小超过指定的fileSizeThreshold设置的大小时临时存储文件。默认位置是“”。
    - fileSizeThreshold: 文件大小，字节单位，超出该值会临时存储到磁盘，默认0字节。
    - MaxFileSize: 允许上传文件的最大值，字节。如果大于该值，web容器会抛出IllegalStateException。默认无限。
    - maxRequestSize: multipart/form-data类型请求的允许的最大值，字节。如果所有的上传文件的总体大小超过该阈值，web容器抛出异常，默认大小无限制。

如@MultipartConfig注解可以使用如下：
        @MultipartConfig(location="/tmp", fileSizeThreshold=1024*1024,
            maxFileSize=1024*1024*5, maxRequestSize=1024*1024*5*5)
为了不使用@MultipartConfig硬编码这些属性到文件上传servlet中，可以在web.xml中的servlet配置节点下配置如下：

        <multipart-config>
            <location>/tmp</location>
            <max-file-size>20848820</max-file-size>
            <max-request-size>418018841</max-request-size>
            <file-size-threshold>1048576</file-size-threshold>
        </multipart-config>

#### getParts&getPart方法
Servlet规范支持两个额外的HttpServletRequest方法：
        Collection<Part> getParts()
        Part getPart(String name)
request.getParts()方法返回所有Part对象的集合。如果有多个类型文件的输入，则返回多个Part对象。因为Part对象有名称，getPart(String name)方法可以用来访问某个指定的Part。另外，getParts()方法返回Iterable<Part>，可以用来所有的Part对象的迭代器。

javax.servlet.http.Part接口很简单，提供方法如下：
    - 获取Part的名称、大小、内容类型
    - 查询和Part一块提交的headers信息
    - 删除一个Part对象
    - 写part到磁盘
如，Part接口提供了write(string filename)方法来指定名称写文件。文件可以保存到通过@MultipartConfig注解指定的目录或form表单的Destination字段指定的目录。

### 异步处理
应用程序服务器的web容器一般每个客户端请求使用一个线程，在高负载条件下，容器需要大量的线程服务于所有的客户端请求。可扩展限制包括内存不足或耗尽容器线程池，为了构建可扩展的web应用程序必须保证与请求相关联的线程没有空闲，这样容器可以用他们处理新请求。

与请求关联的线程闲置有2中情况：
  - 线程需要等待响应返回或构建响应之前处理数据。如应用程序在构建响应之前可能需要查询数据库或者访问远程web服务。
  - 产生响应前线程需要等待事件，如应用程序需要等待一个JMS消息，从另一个客户端的新的消息，或某队列里新产生的数据。
这些场景说明阻塞操作限制了web程序的可扩展性，异步处理指将这些阻塞操作分配给新线程，并将与请求关联的线程立即返回给容器。

#### Servlet中的异步处理
JAVA EE为servlet何filter提供了异步支持。如果servlet或过滤器在处理请求时遇到潜在的阻塞操作，则可以将操作分配给异步执行上下文，并将与请求相关联的线程立即返回到容器而不产生响应。 阻塞操作在异步执行上下文中在不同的线程中完成，可以生成响应或将请求发送到另一个servlet。

要开启servlet的异步处理，添加如下配置：

      @WebServlet(urlPatterns={"/asyncservlet"}, asyncSupported=true)
      public class AsyncServlet extends HttpServlet { ... }

javax.servlet.AsyncContext类提供了在service方法里需要执行异步处理所需的功能。要获取AsyncContext的实例，在service方法里调用request对象的startAsync()方法，如下：

      public void doGet(HttpServletRequest req, HttpServletResponse resp) {
         ...
         AsyncContext acontext = req.startAsync();
         ...
      }
该调用将request设置为异步模式，并确保在service方法退出后未提交响应。在异步上下文中，当阻塞操作完成或将请求转发到另一个servlet后，必须生成响应。


AsyncContext提供的功能
Method Signature	| Description
-- | --
void start(Runnable run) | 容器提供了一个处理阻塞操作的另一个线程。可以定义一个实现了Runnable接口的class处理阻塞操作，该类可以当调用start方法时作为内部类或者传递AsyncContext到自定义处理类里。

ServletRequest getRequest() | 返回用来初始化异步上下文的reqeust对象

ServletResponse getResponse() | 返回用来初始化异步上下文的响应对象；在异步上下文可以用来写入阻塞操作的结果
void complete() | 完成异步操作，关闭和异步上下文关联的响应对象。在异步上下文中，当写入信息到响应对象后，调用该方法。
void dispatch(String path) | 转发请求和响应对象到指定的路径；当阻塞操作完成后，使用该方法来让另一个servlet来写入响应

#### 等待资源
展示AsyncContext提供的功能如何处理以下情形：
  1、servlet从GET请求中获取参数
  2、servlet使用资源，如数据库或web service来获取基于参数值的信息，资源有可能慢造成阻塞操作。
  3、servlet使用来自资源的结果产生响应

不适用异步的话：
      @WebServlet(urlPatterns={"/syncservlet"})
      public class SyncServlet extends HttpServlet {
         private MyRemoteResource resource;
         @Override
         public void init(ServletConfig config) {
            resource = MyRemoteResource.create("config1=x,config2=y");
         }

         @Override
         public void doGet(HttpServletRequest request,
                           HttpServletResponse response) {
            response.setContentType("text/html;charset=UTF-8");
            String param = request.getParameter("param");
            String result = resource.process(param);
            //....
         }
      }
异步的话：

      @WebServlet(urlPatterns={"/asyncservlet"}, asyncSupported=true)
      public class AsyncServlet extends HttpServlet {
         //Same variables and init method as in SyncServlet

         @Override
         public void doGet(HttpServletRequest request,
                           HttpServletResponse response) {
            response.setContentType("text/html;charset=UTF-8");
            final AsyncContext acontext = request.startAsync();
            acontext.start(new Runnable() {
               public void run() {
                  String param = acontext.getRequest().getParameter("param");
                  String result = resource.process(param);
                  HttpServletResponse response = acontext.getResponse();
                  // print to the response
                  acontext.complete();
         }
      }

执行区别：
  - request.startAsync()将请求进行异步处理，service方法执行结束时，响应还没有发送到客户端;
  - acontext.start(new Runnable() {...})从容器获取一个新线程;
  - 新线程的run()方法中的代码逻辑会在新线程执行，内部类有访问异步上下文的权限来读取请求参数，写入到响应。调用complete()方法来提交响应，并发送到客户端。
AsyncServlet的service方法立即返回，请求在异步上下文被处理。

### 非阻塞IO
web容器通常一个请求对应一个服务端线程，要开发可扩展web程序，需要确保和客户端请求关联的线程不会空闲来等待阻塞操作完成。异步处理提供了在新线程中执行阻塞操作的机制，立刻将和请求关联的线程返回给容器。即使在service方法里为所有阻塞操作使用了异步处理，和客户端请求关联的线程也会暂时空闲因为IO因素。

如，客户端提交一个大的HTTP POST请求，但是网络连接很慢，server读取请求的速度比客户端发送数据的速度快。使用传统的IO，和请求相关的容器线程等待剩下的请求时有时会空闲。
当处于异步模式下时，JAVA EE为servlets何filters提供了非阻塞IO。如下例讲解了如何在service方法里使用非阻塞IO：
  - 将请求放到异步模式下
  - 在service方法里从请求或响应对象里获取输入流或输出流。
  - 分配一个读监听器监听输入流，写监听器监听输出流。
  - 在监听器的callback回调方法里处理请求和响应

javax.servlet.ServletInputStream对非阻塞IO的支持：

方法	| 描述
--|--
void setReadListener(ReadListener rl) | 关联输入流到包含callback方法的监听器对象来异步读取数据。可以将监听器对象作为一个匿名类或将输入流传入到监听器对象。
boolean isReady() |  如果数据读不阻塞返回true
boolean isFinished() |  如果所有数据已经读取返回true

javax.servlet.ServletOutputStream对非阻塞IO的支持：

Method	| Description
-- | --
void setWriteListener(WriteListener wl)|关联输出流到包含callback方法的监听器对象，来异步写数据。可以提供监听器对象作为匿名类或将输出流传入到写监听器对象。
boolean isReady()|写不阻塞返回true

监听接口对非阻塞IO支持：

Interface|  Methods | Description
-- | -- |--
ReadListener | void onDataAvailable() ,void onAllDataRead(),void onError(Throwable t)|当有数据可读时，所有数据已经读完，发生错误时，ServletInputStream实例调用监听器的这些方法。

WriteListener |void onWritePossible(),void onError(Throwable t)| 当非阻塞写数据可能的话或发生错误，ServletOutputStream实例调用监听器的这些方法。


#### 使用非阻塞IO读取大的HPPT POST请求
下例展示大区大的HTTP POST请求，使用非阻塞模式及非阻塞IO功能：

      @WebServlet(urlPatterns={"/asyncioservlet"}, asyncSupported=true)
      public class AsyncIOServlet extends HttpServlet {
         @Override
         public void doPost(HttpServletRequest request,
                            HttpServletResponse response)
                            throws IOException {
            final AsyncContext acontext = request.startAsync();
            final ServletInputStream input = request.getInputStream();

            input.setReadListener(new ReadListener() {
               byte buffer[] = new byte[4*1024];
               StringBuilder sbuilder = new StringBuilder();
               @Override
               public void onDataAvailable() {
                  try {
                     do {
                        int length = input.read(buffer);
                        sbuilder.append(new String(buffer, 0, length));
                     } while(input.isReady());
                  } catch (IOException ex) { ... }
               }
               @Override
               public void onAllDataRead() {
                  try {
                     acontext.getResponse().getWriter()
                                           .write("...the response...");
                  } catch (IOException ex) { ... }
                  acontext.complete();
               }
               @Override
               public void onError(Throwable t) { ... }
            });
         }
      }

该例使用@WebServlet注解参数asyncSupported=true声明servlet为异步模式，service方法首先将请求放到异步模式下通过调用request对象的startAsync()，service方法获取输入流然后分配给监听器。监听器当请求的部分可读取时读取，并当结束读取请求时写入响应到客户端。

### 协议升级
在HTTP/1.1中，客户端通过使用Upgrade header字段在当前会话中请求切换到一个不同的协议。如果服务端接受请求切换到客户端声明的协议，就会产生一个status为101（协议切换）的HTTP响应。经过切换后，客户端和服务器就使用新协议通信。
如，客户端可以发出HTTP请求来切换到XYZP协议如下：
      GET /xyzpresource HTTP/1.1
      Host: localhost:8080
      Accept: text/html
      Upgrade: XYZP
      Connection: Upgrade
      OtherHeaderA: Value
客户端使用HTTP头部信息指定参数要求使用新协议，服务端可以接受该请求并生成如下响应：

      HTTP/1.1 101 Switching Protocols
      Upgrade: XYZP
      Connection: Upgrade
      OtherHeaderB: Value

      (XYZP data)
JAVA EE 支持的servlet中的HTTP协议升级功能如下表：

类或接口| 方法
-- | --
HttpServletRequest | HttpUpgradeHandler upgrade(Class handler),upgrade方法开始协议升级过程，该方法实例化一个实现HttpUpgradeHandler接口的类并委托连接到该类。当接受来自客户端切换协议的请求时，在service方法类调用upgrade方法。
HttpUpgradeHandler | void init(WebConnection wc),servlet接受请求时调用init方法来切换协议，需要实现该方法来获取web连接对象的输入&输出流，用于实现新协议。
HttpUpgradeHandler | void destroy(),当客户端断开连接时调用该方法，需要实现该方法来释放和处理新协议相关的所有资源。
WebConnection | ServletInputStream getInputStream(),getInputStream方法是访问连接的输入流的入口，可以使用非阻塞IO结合返回的流来实现新协议。
WebConnection | ServletOutputStream getOutputStream(),getOutputStream方法是访问连接的输出流的入口，可以使用非阻塞IO结合返回的流来实现新协议。

下例展示如何接受来自客户端的协议升级请求：

      @WebServlet(urlPatterns={"/xyzpresource"})
      public class XYZPUpgradeServlet extends HttpServlet {
         @Override
         public void doGet(HttpServletRequest request,
                           HttpServletResponse response) {
            if ("XYZP".equals(request.getHeader("Upgrade"))) {
               // 接受协议升级请求
               response.setStatus(101);
               response.setHeader("Upgrade", "XYZP");
               response.setHeader("Connection", "Upgrade");
               response.setHeader("OtherHeaderB", "Value");
               // 委托连接到升级处理器,XYZPUpgradeHandler处理连接
               XYZPUpgradeHandler = request.upgrade(XYZPUpgradeHandler.class);
               // service方法直接返回
            } else {
               // ... write error response ...
            }
         }
      }

      public class XYZPUpgradeHandler implements HttpUpgradeHandler {
         @Override
         public void init(WebConnection wc) {
            ServletInputStream input = wc.getInputStream();
            ServletOutputStream output = wc.getOutputStream();
            // ... 使用输入流输出流（协议指定）来实现XYZP
         }
         @Override
         public void destroy() { ... }
      }
更多细节见： http://jcp.org/en/jsr/detail?id=340

### 文件上传示例

#### 文件上传示例组成部分
  - 一个servlet处理文件上传
  - 一个html表单往servlet发出文件上传请求，html表单包含input类型为file的输入框，该form表单有两个限制：
    - enctype属性必须设置成multipart/form-date
    - method方法必须是POST类型
当表单被点击提交时，表单数据编码发送到后台servlet，后台servlet从输入流中读取数据，解析提取文件信息，存到服务器。
HTML表单：

      <!DOCTYPE html>
      <html lang="en">
          <head>
              <title>File Upload</title>
              <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
          </head>
          <body>
              <form method="POST" action="upload" enctype="multipart/form-data" >
                  File:
                  <input type="file" name="file" id="file" /> <br/>
                  Destination:
                  <input type="text" value="/tmp" name="destination"/>//destination字段表示文件上传的位置
                  </br>
                  <input type="submit" value="Upload" name="upload" id="upload" />
              </form>
          </body>
      </html>

当客户端想要将发送到服务器的数据当做请求的一部分时需要使用POST请求方法，如上传文件或提交表单数据，GET请求方法仅仅发送URL和headers信息到服务器，POST请求也发送信息体（body体）。POST请求不限制请求的数据长度和类型，通常使用header的字段表明消息体的类型。

提交表单时，浏览器读入文件流，组合所有部分，每个部分代表表单的一个字段，各个部件在输入元素后命名，使用字符串分隔符boundary分隔：

        POST /fileupload/upload HTTP/1.1
        Host: localhost:8080
        Content-Type: multipart/form-data;
        boundary=---------------------------263081694432439
        Content-Length: 441
        -----------------------------263081694432439
        Content-Disposition: form-data; name="file"; filename="sample.txt"
        Content-Type: text/plain

        Data from sample file
        -----------------------------263081694432439
        Content-Disposition: form-data; name="destination"

        /tmp
        -----------------------------263081694432439
        Content-Disposition: form-data; name="upload"

        Upload
        -----------------------------263081694432439--


        @WebServlet(name = "FileUploadServlet", urlPatterns = {"/upload"})
        @MultipartConfig// 表明servlet处理multipart/form-data类型的请求
        public class FileUploadServlet extends HttpServlet {
            private final static Logger LOGGER =
                    Logger.getLogger(FileUploadServlet.class.getCanonicalName());

processRequest方法获取destination参数值，从file参数获取文件，接着进行存放处理：

        protected void processRequest(HttpServletRequest request,
                HttpServletResponse response)
                throws ServletException, IOException {
            response.setContentType("text/html;charset=UTF-8");

            // Create path components to save the file
            final String path = request.getParameter("destination");
            final Part filePart = request.getPart("file");
            final String fileName = getFileName(filePart);

            OutputStream out = null;
            InputStream filecontent = null;
            final PrintWriter writer = response.getWriter();

            try {
                out = new FileOutputStream(new File(path + File.separator
                        + fileName));
                filecontent = filePart.getInputStream();

                int read = 0;
                final byte[] bytes = new byte[1024];

                while ((read = filecontent.read(bytes)) != -1) {
                    out.write(bytes, 0, read);
                }
                writer.println("New file " + fileName + " created at " + path);
                LOGGER.log(Level.INFO, "File{0}being uploaded to {1}",
                        new Object[]{fileName, path});
            } catch (FileNotFoundException fne) {
                writer.println("You either did not specify a file to upload or are "
                        + "trying to upload a file to a protected or nonexistent "
                        + "location.");
                writer.println("<br/> ERROR: " + fne.getMessage());

                LOGGER.log(Level.SEVERE, "Problems during file upload. Error: {0}",
                        new Object[]{fne.getMessage()});
            } finally {
                if (out != null) {
                    out.close();
                }
                if (filecontent != null) {
                    filecontent.close();
                }
                if (writer != null) {
                    writer.close();
                }
            }
        }

        private String getFileName(final Part part) {
            final String partHeader = part.getHeader("content-disposition");
            LOGGER.log(Level.INFO, "Part Header = {0}", partHeader);
            for (String content : part.getHeader("content-disposition").split(";")) {
                if (content.trim().startsWith("filename")) {
                    return content.substring(
                            content.indexOf('=') + 1).trim().replace("\"", "");
                }
            }
            return null;
        }

### 异步更新数据到客户端示例
#### 组成
  - servlet，将请求放入异步模式，在队列里存储，并当价格交易量的新数据更新可用时返回响应。
  - 一个bean每秒更新价格和成交量
  - html页面使用js代码发送请求到servlet请求新数据，解析返回数据，并在不刷新页面时更新数据
该示例使用长连接，为web应用程序提供了一种机制来使用HTTP推送更新数据到客户端，不需要用户端发送显式请求。传统HTTP请求-响应模型里，用户发出显示请求（如点击按钮连接或提交表单）来获取服务端的新数据，页面也必须刷新。长连接情况下，服务器异步处理连接，客户端使用js来创建新连接，客户端在接受新数据后立即发出新请求，服务端会保持连接直到新数据可用。

        @WebServlet(urlPatterns={"/dukeetf"}, asyncSupported=true)
        public class DukeETFServlet extends HttpServlet {
        ...
        }

        @Override
        public void init(ServletConfig config) {
           // Queue for requests
           requestQueue = new ConcurrentLinkedQueue<>();
           // Register with the enterprise bean that provides price/volume updates
           pvbean.registerServlet(this);
        }

        // PriceVolumeBean calls this method every second to send updates
        public void send(double price, int volume) {
           // Send update to all connected clients
           for (AsyncContext acontext : requestQueue) {
              try {
                 String msg = String.format("%.2f / %d", price, volume);
                 PrintWriter writer = acontext.getResponse().getWriter();
                 writer.write(msg);
                 logger.log(Level.INFO, "Sent: {0}", msg);
                 // Close the connection,The client (JavaScript) makes a new one instantly
                 acontext.complete();
              } catch (IOException ex) {
                 logger.log(Level.INFO, ex.toString());
              }
           }
        }


        @Override
        public void doGet(HttpServletRequest request,
                          HttpServletResponse response) {
           response.setContentType("text/html");
           // Put request in async mode
           final AsyncContext acontext = request.startAsync();
           // Remove from the queue when done
           acontext.addListener(new AsyncListener() {
              public void onComplete(AsyncEvent ae) throws IOException {
                 requestQueue.remove(acontext);
              }
              public void onTimeout(AsyncEvent ae) throws IOException {
                 requestQueue.remove(acontext);
              }
              public void onError(AsyncEvent ae) throws IOException {
                 requestQueue.remove(acontext);
              }
              public void onStartAsync(AsyncEvent ae) throws IOException {}
           });
           // Add to the queue
           requestQueue.add(acontext);
        }


        @Startup
        @Singleton
        public class PriceVolumeBean {
            // Use the container's timer service
            @Resource TimerService tservice;
            private DukeETFServlet servlet;
            ...

            @PostConstruct
            public void init() {
                // Initialize the EJB and create a timer
                random = new Random();
                servlet = null;
                tservice.createIntervalTimer(1000, 1000, new TimerConfig());
            }

            public void registerServlet(DukeETFServlet servlet) {
                // Associate a servlet to send updates to
                this.servlet = servlet;
            }

            @Timeout
            public void timeout() {
                // Adjust price and volume and send updates
                price += 1.0*(random.nextInt(100)-50)/100.0;
                volume += random.nextInt(5000) - 2500;
                if (servlet != null)
                    servlet.send(price, volume);
            }
        }

HTML代码：

        <html xmlns="http://www.w3.org/1999/xhtml">
        <head>...</head>
        <body onload="makeAjaxRequest();">
          ...
          <table>
            ...
            <td id="price">--.--</td>
            ...
            <td id="volume">--</td>
            ...
          </table>
        </body>
        </html>
JS代码使用XMLHttpRequest的API来在客户端和服务器之间传递数据

        var ajaxRequest;
        function updatePage() {
           if (ajaxRequest.readyState === 4) {
              var arraypv = ajaxRequest.responseText.split("/");
              document.getElementById("price").innerHTML = arraypv[0];
              document.getElementById("volume").innerHTML = arraypv[1];
              makeAjaxRequest();
           }
        }
        function makeAjaxRequest() {
           ajaxRequest = new XMLHttpRequest();
           ajaxRequest.onreadystatechange = updatePage;
           ajaxRequest.open("GET", "http://localhost:8080/dukeetf/dukeetf",
                            true);
           ajaxRequest.send(null);
        }
The XMLHttpRequest API is supported by most modern browsers, and it is widely used in Ajax web client development (Asynchronous JavaScript and XML).

See The dukeetf2 Example Application in Chapter 18, "Java API for WebSocket" for an equivalent version of this example implemented using a WebSocket endpoint.

17.17.2 Running the dukeetf Example Application

This section describes how to run the dukeetf example application using NetBeans IDE and from the command line.

17.17.2.1 To Run the dukeetf Example Application Using NetBeans IDE

Make sure that GlassFish Server has been started (see Starting and Stopping GlassFish Server).

From the File menu, choose Open Project.

In the Open Project dialog box, navigate to:

tut-install/examples/web/servlet
Select the dukeetf folder.

Click Open Project.

In the Projects tab, right-click the dukeetf project and select Run.

This command builds and packages the application into a WAR file (dukeetf.war) located in the target directory, deploys it to the server, and launches a web browser window with the following URL:

http://localhost:8080/dukeetf/
Open the same URL in a different web browser to see how both pages get price and volume updates simultaneously.

17.17.2.2 To Run the dukeetf Example Application Using Maven

Make sure that GlassFish Server has been started (see Starting and Stopping GlassFish Server).

In a terminal window, go to:

tut-install/examples/web/servlet/dukeetf/
Enter the following command to deploy the application:

mvn install
Open a web browser window and type the following address:

http://localhost:8080/dukeetf/
Open the same URL in a different web browser to see how both pages get price and volume updates simultaneously.
### 附录
servlet-3_1-final.pdf https://jcp.org/aboutJava/communityprocess/final/jsr340/index.html
