文档：https://heavy_code_industry.gitee.io/code_heavy_industry/pro001-javaweb/lecture/

-------

CS：客户端服务器架构模式

- 优点：充分利用客户端机器的资源，减轻服务器的负荷（一部分安全要求不高的计算任务存储任务放在客户端执行，减轻服务器压力，减少网络通信）
- 缺点：需要安装；升级维护成本较高；

BS：浏览器服务器架构模式

- 优点：客户端不需要安装；维护成本低
- 缺点：所有任务都在服务器端进行，服务器运算存储压力大；需要大量的网络通信。

---------

#### Tomcat

Tomcat是一种轻量级，免费的javaweb服务器，提供对jsp和Servlet的支持

##### 安装和配置

下载：[Index of /apache/tomcat/tomcat-8/v8.5.85/bin (tsinghua.edu.cn)](https://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.5.85/bin/)

解压：目录不要有中文，不要有空格

目录结构：

![image-20230213141744512](javaweb.assets/image-20230213141744512.png)

配置环境变量：tomcat是用java和C来编写的，需要配置JAVA_HOME

启动tomcat：运行bin目录下的start.bat

访问主页：https://localhost:8080

项目部署：就是把文件夹拷贝到webapps下，示例见webapps/examples。启动tomcat后访问http://localhost:8080/examples/index.html

#### idea下创建javaweb项目-部署-运行

创建java项目后右击项目，选择Add Framework Support

![image-20230213144441238](javaweb.assets/image-20230213144441238.png)

勾选Web Application

![image-20230213144625321](javaweb.assets/image-20230213144625321.png)

完成后完整目录如下

![image-20230213144654072](javaweb.assets/image-20230213144654072.png)

配置如下即可启动项目，需要注意的是，application context是可以自定义的

<img src="javaweb.assets/image-20230213151940499.png" alt="image-20230213151940499" style="zoom: 50%;" />

下面的设置中需要注意的有：

URL默认没有/hello01.html，需要手动添加

On 'Update' ations和On frame deactivation最好按照如图示修改，这样在修改代码后可以实现页面动态调整，无需重新启动

<img src="javaweb.assets/image-20230213152555665.png" alt="image-20230213152555665" style="zoom:50%;" />

# Servlet

## 入门

1. 新建项目-新建模块
2. 在模块中添加web application
3. 创建artifact - 部署包
4. lib - artifact 
   - 先有artifact，后来才添加的mysql驱动jar包，此时，这个jar包没有添加到部署包中，有两种解决方案
     - 在Project Structure中Problems会有提示，点击fix选择add to
     - 可以直接将lib文件夹放在WEB-INF下，但是这样只能当前一个模块独享
5. 在部署的时候，修改application Context。然后再回到server选项卡，检查URL的值。URL的值指的是tomcat启动完成后自动打开指定浏览器的默认访问地址。
   - 如果我们的URL是https://localhost:8080/test/，那么我们访问的是index.html，如果没有这个页面则会报404
6. 405问题。当前请求的方法不支持。比如，表单中method=post，servlet必须对应doPost，否则报405错误。
7. 注意<url-pattern>中以斜杠开头。

## 设置编码

输入中文会出现如下乱码

![image-20230214093253744](javaweb.assets/image-20230214093253744.png)

```java
//get方式目前不需要设置编码（基于tomcat8）,tomcat8之前get请求的中文数据，需要如下转码方式
String fname = req.getParameter("fname");
//1.将字符串打散成字节数据
byte[] bytes = fname.getBytes("ISO-8859-1");
fname = new String(bytes, "UTF-8");

//post方式下，需要设置编码，防止中文乱码
req.setCharacterEncoding("UTF-8");
```

![image-20230214094450420](javaweb.assets/image-20230214094450420.png)

如果仍然出现中文乱码问题，有可能是数据库编码问题。

## Servlet的继承关系

### 继承关系

- javax.servlet.Servlet接口
  - javax.servlet.GenericServlet抽象类
    - javax.servlet.http.HttpServlet抽象子类

### 方法

javax.servlet.Servlet接口

- void init(config) - 初始化方法
- void service(request,response) - 服务方法
- void destory() - 销毁方法

javax.servlet.GenericServlet抽象类

- void service(request,response) - 仍然是抽象方法

javax.servlet.http.HttpServlet抽象子类

- void service(requset,response) - 不是抽象方法

  - ```java
    //获取请求方式
    String method = req.getMethod();
    ```

  - 根据请求方式不同，调用不同的do方法

  - 在HttpServlet这个抽象类中中的do方法如果被调用则会报错，方法都差不多，示例如下

  - ```java
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String msg = lStrings.getString("http.method_post_not_supported");
        this.sendMethodNotAllowed(req, resp, msg);
    }
        private void sendMethodNotAllowed(HttpServletRequest req, HttpServletResponse resp, String msg) throws IOException {
            String protocol = req.getProtocol();
            if (protocol.length() != 0 && !protocol.endsWith("0.9") && !protocol.endsWith("1.0")) {
                resp.sendError(405, msg);
            } else {
                resp.sendError(400, msg);
            }
    
        }
    ```

### 小结

1. 继承关系：HttpServlet -> GenericServlet -> Servlet
2. Servlet中的核心方法：inti()，service()，destroy()
3. 服务方法：当有请求时，service方法会自动相应（其实是tomcat容器调用的）。
4. 在HttpServlet中我们会去分析请求的方式，根据请求方式调用do方法，在HttpServlet中这些do方法都是405或400的报错实现，要求子类实现对应的方法，否则会报错。

## Servlet的生命周期

默认情况下，第一次接受请求时，Servlet会进行实例化（构造方法），初始化（init方法），然后开始服务（service方法），后面的请求只会调用service方法，当容器关闭时，所有的servlet实例会被销毁，调用destroy方法。

实例化是tomcat在收到请求时，通过反射实现的

```java
public class AddServlet extends HttpServlet {

    @Override
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        System.out.println("正在服务");
    }

    @Override
    public void destroy() {
        System.out.println("正在销毁");
    }

    @Override
    public void init() throws ServletException {
        System.out.println("正在初始化");
    }
}
```

<load-on-startup>的作用：

- 默认servlet是在第一次请求时进行实例化和初始化，如果设置了<load-on-startup>，则会随tomcat容器启动一起进行实例化和初始化。
- 如果有多个servlet可以配置servlet的启动顺序。0->1->2

```xml
<servlet>
    <servlet-name>AddServlet</servlet-name>
    <servlet-class>com.zhuweihao.servlets.AddServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
```

Servlet在容器中是单例的，线程不安全的

- 单例：所有的请求都由一个servlet进行相应，不会实例化第二个servlet
- 线程不安全：两个线程同时被响应时，一个线程改变了成员变量可能会导致另一个线程代码执行路径发生改变，引起错误。
- 因此，尽量不要在servlet中定义成员变量，如果必须定义，注意不要修改成员变量的值，不要根据成员变量进行逻辑判断

## 会话控制

HTTP协议本身是无状态的。单靠HTTP协议本身无法判断一个请求来自于哪一个浏览器，所以也就没法识别用户的身份状态。

### Cookie



### Session

- 服务器端没调用request.getSession()方法：什么都不会发生
- 服务器端调用了request.getSession()方法
  - 服务器端检查当前请求中是否携带了JSESSIONID的Cookie
    - 有：根据JSESSIONID在服务器端查找对应的HttpSession对象
      - 能找到：将找到的HttpSession对象作为request.getSession()方法的返回值返回
      - 找不到：服务器端新建一个HttpSession对象作为request.getSession()方法的返回值返回
    - 无：服务器端新建一个HttpSession对象作为request.getSession()方法的返回值返回

<img src="javaweb.assets/image-20230214133236033.png" alt="image-20230214133236033" style="zoom: 67%;" />

```java
@Override
protected void service(HttpServletRequest request, HttpServletResponse resp) throws ServletException, IOException {
    // 1.调用request对象的方法尝试获取HttpSession对象
    HttpSession session = request.getSession();
    // 2.调用HttpSession对象的isNew()方法
    boolean wetherNew = session.isNew();
    // 3.打印HttpSession对象是否为新对象
    System.out.println("wetherNew = " + (wetherNew ? "HttpSession对象是新的" : "HttpSession对象是旧的"));
    // 4.调用HttpSession对象的getId()方法
    String id = session.getId();
    // 5.打印JSESSIONID的值
    System.out.println("JSESSIONID = " + id);
}
```

服务器端给Session对象设置最大闲置时间，默认为1800秒

```java
// 获取默认的最大闲置时间
int maxInactiveIntervalSecond = session.getMaxInactiveInterval();
System.out.println("maxInactiveIntervalSecond = " + maxInactiveIntervalSecond);
// 设置默认的最大闲置时间
session.setMaxInactiveInterval(15);
```

<img src="javaweb.assets/image-20230214134558584.png" alt="image-20230214134558584" style="zoom:50%;" />

强制关闭session

```java
session.invalidate();
```



## 请求转发和重定向

发一个请求给Servlet，接力棒就传递到了Servlet手中。而绝大部分情况下，Servlet不能独自完成一切，需要把接力棒继续传递下去，此时我们就需要请求的**『转发』**或**『重定向』**。

### 转发

在请求的处理过程中，Servlet完成了自己的任务，需要把请求转交给下一个资源继续处理。

<img src="javaweb.assets/image-20230214145745841.png" alt="image-20230214145745841" style="zoom:50%;" />

```java
//服务器内部转发
req.getRequestDispatcher("add").forward(req,resp);
```

<img src="javaweb.assets/image-20230214151113472.png" alt="image-20230214151113472" style="zoom: 67%;" />



### 重定向

在请求的处理过程中，Servlet完成了自己的任务，然后以一个响应的方式告诉浏览器：“要完成这个任务还需要你另外再访问下一个资源”。

<img src="javaweb.assets/image-20230214151441967.png" alt="image-20230214151441967" style="zoom:50%;" />

```java
//客户端重定向
resp.sendRedirect("add");
```

<img src="javaweb.assets/image-20230214151311266.png" alt="image-20230214151311266" style="zoom:67%;" />

注意：

- 地址栏发生变化
- 状态码为302
- 响应头中Location标明了重定向的资源

## 作用域

![image-20230215151112614](javaweb.assets/image-20230215151112614.png)

常用API：

- void session.setAttribute(k,v)
- Object session.getAttribute(k)
- void session.removeAttribute(k)



# Thymeleaf

服务器端模板技术

![image-20230215151720080](javaweb.assets/image-20230215151720080.png)

Thymeleaf优势：

- SpringBoot官方推荐使用的视图模板技术，和SpringBoot完美整合。
- 不经过服务器运算仍然可以直接查看原始值，对前端工程师更友好。

![image-20230215152422089](javaweb.assets/image-20230215152422089.png)

### 物理视图和逻辑视图

在Servlet中，将请求转发到一个HTML页面文件时，使用的完整的转发路径就是**物理视图**。

![image-20230215152555380](javaweb.assets/image-20230215152555380.png)

如果我们把所有的HTML页面都放在某个统一的目录下，那么转发地址就会呈现出明显的规律：

/add.html；/index.html；

路径的开头都是：/

路径的结尾都是：.html

所以，路径开头的部分我们称之为**视图前缀**，路径结尾的部分我们称之为**视图后缀**。



物理视图=视图前缀+逻辑视图+视图后缀
