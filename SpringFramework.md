Spring实战（第五版）：https://ebook.qicoder.com/spring-in-action-5th/



![image-20230310202718372](SpringFramework.assets/image-20230310202718372.png)

# IOC

## IOC原理

xml解析 -> 工厂模式 -> 反射

ioc容器底层就是对象工厂

Spring提供IOC容器的两种实现方式（两个接口）

- BeanFactory：IOC容器基本实现，加载配置文件的时候不会创建对象，在获取（使用）对象的时候才去创建对象
- ApplicationContext：BeanFactory的子接口，加载配置文件的时候就会创建对象

![image-20230227113043983](SpringFramework.assets/image-20230227113043983.png)

![image-20230227113546889](SpringFramework.assets/image-20230227113546889.png)

## Bean管理

bean管理指的是两个操作

- spring创建对象
- spring注入属性

bean管理操作有两种方式

- 基于xml配置文件方式
- 基于注解方式

### 基于xml方式

#### 创建对象

![image-20230227114842618](SpringFramework.assets/image-20230227114842618.png)

- id：唯一标识
- class：全路径名

创建对象的时候默认执行无参构造方法完成对象创建

#### 注入属性

DI：dependency injection，依赖注入，是IOC的一种实现方式

属性注入方法：

- 使用set方法进行注入

![image-20230306201259897](SpringFramework.assets/image-20230306201259897.png)

- 有参构造方法注入，要编写有参构造函数

![image-20230306203054402](SpringFramework.assets/image-20230306203054402.png)

![image-20230306203130412](SpringFramework.assets/image-20230306203130412.png)

- p空间注入，其实也是用set方法进行注入，可以在set方法上打断点进行观察，“q”是可以自定义的

![image-20230306204047024](SpringFramework.assets/image-20230306204047024.png)

#### 注入其他类型属性

> 字面量

##### 空值

![image-20230306204941036](SpringFramework.assets/image-20230306204941036.png)

##### 属性值包含特殊符号

> 把特殊字符进行转义

把特殊字符内容写到CDATA

![image-20230306205643401](SpringFramework.assets/image-20230306205643401.png)

##### 外部bean

![image-20230306210114918](SpringFramework.assets/image-20230306210114918.png)

##### 内部bean和级联赋值

![image-20230307100057614](SpringFramework.assets/image-20230307100057614.png)

![image-20230307101005825](SpringFramework.assets/image-20230307101005825.png)

##### 注入集合属性

![image-20230307105518022](SpringFramework.assets/image-20230307105518022.png)

```xml
    <util:list id="courseList">
        <value>数据结构</value>
        <value>算法设计与分析</value>
    </util:list>

    <bean id="stu" class="com.zhuweihao.SpringFramework.pojo.Stu">
        <property name="course" ref="courseList"/>
        <property name="score">
            <array>
                <value>12</value>
                <value>45</value>
            </array>
        </property>
        <property name="performance">
            <map>
                <entry key="JAVA" value="34"/>
                <entry key="C++" value="67"/>
            </map>
        </property>
        <property name="classmate">
            <set>
                <ref bean="user"/>
            </set>
        </property>
    </bean>
```

> 应用中可能有一部分是公用的，我们可以将其抽取出来，要在xml配置文件中添加下面的内容
>
> ![image-20230307105502033](SpringFramework.assets/image-20230307105502033.png)

#### 工厂bean（FactoryBean）

> 普通bean：在配置文件中定义的bean类型就是返回类型
>
> 工厂bean：在配置文件中定义的bean类型可以和返回类型不一样

使用方法：

1. 创建类，实现接口FactoryBean
2. 实现接口里面的方法，在实现的方法中定义返回的bean类型

> 一般情况下，Spring通过反射机制利用<bean>的class属性指定实现类实例化Bean，在某些情况下，实例化Bean过程比较复杂，如果按照传统的方式，则需要在<bean>中提供大量的配置信息。配置方式的灵活性是受限的，这时采用编码的方式可能会得到一个简单的方案。Spring为此提供了一个org.springframework.bean.factory.FactoryBean的工厂类接口，用户可以通过实现该接口定制实例化Bean的逻辑。FactoryBean接口对于Spring框架来说占用重要的地位，Spring自身就提供了70多个FactoryBean的实现。它们隐藏了实例化一些复杂Bean的细节，给上层应用带来了便利。从Spring3.0开始，FactoryBean开始支持泛型，即接口声明改为FactoryBean<T>的形式

```java
public class myFactoryBean implements FactoryBean<Book> {

    //定义返回bean
    @Override
    public Book getObject() throws Exception {
        Book book = new Book("算法设计","张三");
        return book;
    }

    @Override
    public Class<?> getObjectType() {
        return null;
    }
}
```

![image-20230307112810759](SpringFramework.assets/image-20230307112810759.png)

#### Bean的作用域

在Spring中，可以设置创建的bean实例是单实例还是多实例，默认情况下创建的bean是一个单实例对象

使用<scope>标签进行配置，可取值：

- singleton，默认值，表示是单实例对象，在加载spring配置文件时就会创建对象
- prototype，表示是多实例对象，不会再加载配置文件时创建，在获取对象的时候才会创建

![image-20230307113342617](SpringFramework.assets/image-20230307113342617.png)

#### bean生命周期

1. 创建对象，注入属性
2. 调用初始化方法
3. 对象获取成功，可以使用
4. 销毁对象

![image-20230307115508010](SpringFramework.assets/image-20230307115508010.png)

一般的数据对象不需要额外的配置，但是一些资源服务对象有初始化和销毁的必要

###### bean的后置处理器

配置后置处理器的方法：实现接口BeanPostProcessor

```java
public class myBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("初始化之前执行的方法");
        return BeanPostProcessor.super.postProcessBeforeInitialization(bean, beanName);
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("初始化之后执行的方法");
        return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
    }
}
```

**在applicationContext.xml配置bean后，所有的bean在被实例化的时候都会执行上面的方法**

#### 自动装配

上面通过property，value，ref等标签配置的方式指定属性值（不是String等普通属性）的方式为手动装配

自动装配：根据指定装配规则（属性名称或者属性类型），Spring自动将匹配的属性值注入。

- 根据属性名称
- 根据属性类型

![image-20230307154723360](SpringFramework.assets/image-20230307154723360.png)

#### 外部属性文件

示例：配置德鲁伊连接池

![image-20230307161250503](SpringFramework.assets/image-20230307161250503.png)

### 基于注解方式

#### 创建对象

要使用下面的注解，要开启组件扫描

![image-20230307163156776](SpringFramework.assets/image-20230307163156776.png)

```java
@Component(value = "fruitController")   //<bean id="fruitController" class="com.zhuweihao.SpringFramework.controller.FruitController"/>
@Controller //默认值是将类名称的首字母小写形式，这个注解和上面的效果相同
public class FruitController {
    public void test(){
        System.out.println("注解方式创建对象");
    }
}
```

测试方法：

![image-20230307163839862](SpringFramework.assets/image-20230307163839862.png)

> 下面的四个注解效果相同，但是为了代码可读性，不可混用

###### @Component

###### @Service

###### @Controller

###### @Repository

#### 注入属性



###### @Autowired

根据属性类型进行注入

当有多个相同类型的属性的时候，可以配合@Qualifier一起使用

###### @Qualifier

根据属性名称进行注入

###### @Resource

可以根据类型注入，也可以根据名称注入

![image-20230307170534672](SpringFramework.assets/image-20230307170534672.png)

注意这不是spring官方的注解

###### @Value

注入普通类型属性

可用于读取配置文件

#### 完全注解开发

创建配置类

![image-20230307172458849](SpringFramework.assets/image-20230307172458849.png)

测试类

![image-20230307172440313](SpringFramework.assets/image-20230307172440313.png)



# AOP

## 代理模式（Proxy）

### 基本介绍

代理模式：为一个对象提供一个替身，以控制对这个对象的访问。即通过代理对象访问目标对象，这样做的好处是，可以在目标对象实现的基础上，增强额外的功能操作，即扩展对象的功能。

被代理的对象可以是远程对象，创建开销大的对象或需要安全控制的对象

代理模式有不同的形式，主要有三种：

- 静态代理
- 动态代理（JDK代理、接口代理）
- CGLIB代理：可以在内存动态的创建对象，不需要实现接口，有些地方他把划作动态代理的范畴

### 静态代理

静态代理在使用时，需要定义接口或者父类，被代理对象与代理对象一起实现相同的接口或者继承相同的父类。

![image-20230307211256438](SpringFramework.assets/image-20230307211256438.png)

![image-20230307211440248](SpringFramework.assets/image-20230307211440248.png)

- 优点：在不修改目标对象的功能前提下，能通过代理对象对目标功能扩展
- 缺点：因为代理对象需要与目标对象实现一样的接口，所以会有很多代理类，一旦接口增加方法，目标对象与代理对象都需要维护

### 动态代理

代理对象不需要实现接口，但是目标对象要实现接口，否则不能用动态代理

代理对象的生成是利用JDK的API，动态的在内存中构建代理对象

动态代理也叫做：JDK代理，接口代理

JDK中生成代理对象的API：

- java.lang.reflect.Proxy：https://docs.oracle.com/javase/8/docs/api/index.html

![image-20230307212431978](SpringFramework.assets/image-20230307212431978.png)

```java
public class ProxyFactory {
    //维护一个目标对象
    private Object target;

    public ProxyFactory(Object target) {
        this.target = target;
    }

    //给目标对象生成一个代理对象
    public Object getProxyInstance(){

        /**
         *     public static Object newProxyInstance(ClassLoader loader,
         *                                           Class<?>[] interfaces,
         *                                           InvocationHandler h)
         *     1.ClassLoader loader:指定当前目标对象使用的类加载器，获取加载器的方法固定
         *     2.Class<?>[] interfaces：目标对象实现的接口类型，使用泛型方法确认类型
         *     3.InvocationHandler h：InvocationHandler是一个接口，h是他的匿名实现类。每个代理实例都有一个关联的调用处理程序。当我们通过动态代理对象调用一个方法时候，这个方法的调用就会被转发到实现InvocationHandler接口类的invoke方法来调用。
         */
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new InvocationHandler() {
                    /**
                     * proxy:代理类代理的真实代理对象com.sun.proxy.$Proxy0
                     * method:我们所要调用某个对象真实的方法的Method对象
                     * args:指代代理对象方法传递的参数
                     */
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        System.out.println("JDK代理开始");
                        //通过反射机制调用目标对象的方法
                        Object invoke = method.invoke(target, args);
                        System.out.println("JDK代理提交");
                        return invoke;
                    }
                });
    }
}
```

![image-20230307221628374](SpringFramework.assets/image-20230307221628374.png)

测试方法：

![image-20230307220650791](SpringFramework.assets/image-20230307220650791.png)

> 需要追踪一下源码，好好理解

### CGLIB代理

静态代理和JDK代理模式都要求目标对象是实现一个接口，但是有时候目标对象只是一个单独的对象，并没有实现任何接口，这个时候可使用目标对象子类来实现代理，这就是CGLIB代理。

CGLIB代理也叫做子类代理，它是内存中构建一个子类对象从而实现对目标对象功能扩展，有些书也将CGLIB代理归属到动态代理。

CGLIB是一个强大的高性能的代码生成包，它可以在运行期扩展java类和实现Java接口，它广泛的被许多AOP框架使用，例如Spring AOP，实现方法拦截

在AOP编程中如何选择代理模式：

- 目标对象需要实现接口，用JDK代理
- 目标对象不需要实现接口，用CGLIB代理

CGLIB包的底层是通过使用字节码处理框架ASM来转换字节码并生成新的类

在内存中动态构建子类，注意代理的类不能为final

目标对象的方法如果为final/static，那么就不会被拦截，即不会执行目标对象额外的业务方法。

> CGLIB原理：动态生成一个要代理类的子类，子类重写要代理的类的所有不是final的方法。在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑。它比使用java反射的JDK动态代理要快。
>
> CGLIB底层：使用字节码处理框架ASM，来转换字节码并生成新的类。不鼓励直接使用ASM，因为它要求你必须对JVM内部结构包括class文件的格式和指令集都很熟悉。
>
> CGLIB缺点：对于final方法，无法进行代理。

![image-20230308110229235](SpringFramework.assets/image-20230308110229235.png)

```java
public class ProxyFactory implements MethodInterceptor {

    /**
     * 维护一个目标对象
     */
    private Object target;

    /**
     * 构造器，传入一个被代理的对象
     */
    public ProxyFactory(Object target) {
        this.target = target;
    }

    /**
     * @return 返回一个代理对象，是target对象的代理对象
     */
    public Object getProxyInstance() {
        //1.创建一个工具类
        Enhancer enhancer = new Enhancer();
        //2.设置父类
        enhancer.setSuperclass(target.getClass());
        //3.设置回调函数
        enhancer.setCallback(this);
        //4.创建子类对象
        return enhancer.create();
    }

    /**
     * 重写intercept方法，拦截父类方法调用
     *
     * @param o
     * @param method
     * @param objects
     * @param methodProxy
     * @return
     * @throws Throwable
     */
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("cglib代理模式开始");
        Object invoke = method.invoke(target, objects);
        System.out.println("cglib代理模式提交");
        return invoke;
    }
}
```

![image-20230308102842592](SpringFramework.assets/image-20230308102842592.png)

### 代理模式（Proxy）的变体

#### 防火墙代理

内网通过代理穿透防火墙，实现对公网的访问

#### 缓存代理

比如，当请求图片文件等资源时，先到缓存代理取，如果取到资源则结束，如果取不到资源再到公网或者数据库取，然后缓存

#### 远程代理

远程对象的本地代表，通过它可以把远程对象当本地对象来调用，远程代理通过网络和真正的远程对象沟通信息

#### 同步代理

主要使用在多线程编程中，完成多线程间同步工作

## 基本概念

参考博客：https://zhuanlan.zhihu.com/p/37497663，https://blog.csdn.net/jjclove/article/details/124386972

-----

AOP：Aspect Oriented Programming，面向切面编程

- AOP全称（Aspect Oriented Programming）面向切片编程的简称。AOP面向方面编程基于IOC，是对OOP的有益补充；
- AOP利用一种称为“横切”的技术，剖解开封装的对象内部，并将那些影响了 多个类的公共行为封装到一个可重用模块，并将其名为“Aspect”，即方面。所谓“方面”，简单地说，就是将那些与业务无关，却为业务模块所共同调用的 逻辑或责任封装起来，比如日志记录，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可操作性和可维护性。
- 实现AOP的技术，主要分为两大类：一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；二是采用静态织入的方式，引入特定的语法创建“方面”，从而使得编译器可以在编译期间织入有关“方面”的代码。
- Spring实现AOP：JDK动态代理和CGLIB代理。JDK动态代理：其代理对象必须是某个接口的实现，它是通过在运行期间创建一个接口的实现类来完成对目标对象的代理；其核心的两个类是InvocationHandler和Proxy。 CGLIB代理：实现原理类似于JDK动态代理，只是它在运行期间生成的代理对象是针对目标类扩展的子类。CGLIB是高效的代码生成包，底层是依靠ASM（开源的java字节码编辑类库）操作字节码实现的，性能比JDK强；需要引入包asm.jar和cglib.jar。使用AspectJ注入式切面和@AspectJ注解驱动的切面实际上底层也是通过动态代理实现的

> AspectJ不是Spring组成部分，是一个独立的AOP框架，在开发中配合起来使用进行AOP操作较为方便

**AOP的作用：**

1. 面向切面编程（AOP）提供另外一种角度来思考程序结构，通过这种方式弥补了面向对象编程（OOP）的不足。
2. 利用AOP对业务逻辑的各个部分进行隔离，降低业务逻辑的耦合性，提高程序的可重用型和开发效率。
3. 主要用于对同一对象层次的公用行为建模

-----------------



**这里先给出一个比较专业的概念定义**：

- `Aspect`（切面）： Aspect 声明类似于 Java 中的类声明，在 Aspect 中会包含着一些 Pointcut 以及相应的 Advice。**把增强应用到切入点的过程**
- `Joint point`（连接点）：表示在程序中明确定义的点，典型的包括方法调用，对类成员的访问以及异常处理程序块的执行等等，它自身还可以嵌套其它 joint point。**类里面哪些方法可以被增强，这些方法称为连接点**
- `Pointcut`（切点）：表示一组 joint point，这些 joint point 或是通过逻辑关系组合起来，或是通过通配、正则表达式等方式集中起来，它定义了相应的 Advice 将要发生的地方。**真正被增强的方法，称为切入点**
- `Advice`（增强）：Advice 定义了在 `Pointcut` 里面定义的程序点具体要做的操作，它通过 before、after 和 around 来区别是在每个 joint point 之前、之后还是代替执行的代码。
- `Target`（目标对象）：织入 `Advice` 的目标对象.。
- `Weaving`（织入）：将 `Aspect` 和其他对象连接起来, 并创建 `Advice` object 的过程

----------------

![image-20230308114339556](SpringFramework.assets/image-20230308114339556.png)

## 基于AspectJ实现AOP操作

- 基于xml配置文件实现
- 基于注解方式实现（常用）

切入点表达式

作用：知道对哪个类里面的哪个方法进行增强

语法结构：execution(\[权限修饰符][返回类型]\[带类全路径的方法名称](\[参数列表]))

#### 注解方式

```java
@Component
@Aspect
@Order(1)   //如果一个类有多个代理类，可以通过这个注解配置优先级，数字小的先执行
public class TeacherProxy {
    /**
     * 相同的切入点可以进行抽取
     */
    @Pointcut(value = "execution(* com.zhuweihao.SpringFramework.dao.impl.TeacherDaoImpl.teach(..))")
    public void pointCut(){}

    @Before(value = "pointCut()")
    public void before(){
        System.out.println("before.........");
    }
    @After(value = "pointCut()")
    public void after(){
        System.out.println("after..........");
    }
    @Around(value = "pointCut()")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("环绕之前。。。。。");
        //被增强的方法执行
        proceedingJoinPoint.proceed();
        System.out.println("环绕之后。。。。。。");
        throw new Exception();
    }
    @AfterReturning(value = "pointCut()")
    public void afterReturning(){
        System.out.println("afterReturning.......");
    }
    @AfterThrowing(value = "pointCut()")
    public void afterThrowing(){
        System.out.println("afterThrowing.........");
    }
}
```

配置类需要添加注解

![image-20230308142907601](SpringFramework.assets/image-20230308142907601.png)

#### AspectJ配置文件

![image-20230308143818271](SpringFramework.assets/image-20230308143818271.png)



# JdbcTemplate

Spring框架对JDBC框架进行封装，方便实现对数据库的操作。

```java
@Repository
public class FruitDaoImpl implements FruitDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Override
    public Fruit selectById(Integer id) {
        String sql = "select * from Fruit where fid = ?";
        return jdbcTemplate.queryForObject(sql,new BeanPropertyRowMapper<>(Fruit.class),id);
    }

    @Override
    public void addFruit(Fruit fruit) {
        String sql="insert into Fruit(fname,price,fcount,remark) values(?,?,?,?)";
        jdbcTemplate.update(sql,fruit.getFname(),fruit.getFcount(),fruit.getPrice(),fruit.getRemark());
    }

    @Override
    public void deleteById(Integer id) {
        String sql="delete from Fruit where fid=?";
        jdbcTemplate.update(sql,id);
    }

    @Override
    public void updatePriceById(Integer id, Integer price) {
        String sql="update Fruit set price=? where fid=?";
        jdbcTemplate.update(sql,price,id);
    }

    @Override
    public void updateBatch(List<Object[]> bathgArgs) {
        String sql="update Fruit set price=? where fid=?";
        jdbcTemplate.batchUpdate(sql,bathgArgs);
    }
}
```

# 事务操作



事务是数据库操作最基本单元，逻辑上一组操作，要么都成功，如果有一个失败所有操作都失败。

典型示例：银行转账。

事务特性（ACID）：

- 原子性：**Atomicity**，原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。
- 一致性：**Consistency**，事务前后数据的完整性必须保持一致。
- 隔离性：**Isolation**，事务的隔离性是多个用户并发访问数据库时，数据库为每一个用户开启的事务，不能被其他事务的操作数据所干扰，多个并发事务之间要相互隔离。
- 持久性：**Durability**，持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来即使数据库发生故障也不应该对其有任何影响

在Spring进行声明式事务管理，底层使用到AOP原理

```java
@Service
public class BankServiceImpl implements BankService {
    @Resource
    private BankDao bankDao;

    @Override
    @Transactional  //这个注解可以添加到类上面，也可以添加到方法上面
    public void transferAccounts(String transferOutAccount, String transferInAccount, Integer transferAmount) {
        //手动实现事务管理
//        Integer outAccountBalance = bankDao.getAccountBalance(transferOutAccount);
//        if(outAccountBalance<transferAmount){
//            System.out.println("账号余额不足");
//        }else{
//            Integer inAccountBalance = bankDao.getAccountBalance(transferInAccount);
//            try{
//                //开启事务
//                //进行业务操作
//                bankDao.updateMoneyByAccount(transferOutAccount,outAccountBalance-transferAmount);
//                //模拟异常
//                int i=10/0;
//                bankDao.updateMoneyByAccount(transferInAccount,inAccountBalance+transferAmount);
//                //没有发生异常，提交事务
//            }catch (Exception e){
//                //出现异常，事务回滚
//            }
//        }

        //基于注解@Transcational
        Integer outAccountBalance = bankDao.getAccountBalance(transferOutAccount);
        if (outAccountBalance < transferAmount) {
            System.out.println("账号余额不足");
        } else {
            Integer inAccountBalance = bankDao.getAccountBalance(transferInAccount);
            bankDao.updateMoneyByAccount(transferOutAccount, outAccountBalance - transferAmount);
            //模拟异常
            int i = 10 / 0;
            bankDao.updateMoneyByAccount(transferInAccount, inAccountBalance + transferAmount);
        }
    }
}
```

#### 事务注解参数

##### 事务传播行为

参考博客：**https://blog.csdn.net/soonfly/article/details/70305683**

![img](SpringFramework.assets/SouthEast.png)

##### 隔离级别

事务的隔离性使得多事务操作之间不会产生影响。不考虑隔离性会产生很多问题

有三个问题：

- 脏读：一个未提交事务读取到另一个未提交事务的数据，这是相当危险的，因为很可能所有的操作都被回滚。
- 不可重复读：一个事务对同一行数据重复读取两次，但是却得到了不同的结果。比如事务T1读取某一数据后，事务T2对其做了修改，当事务T1再次读该数据时得到与前一次不同的值。又叫虚读。
- 虚（幻）读：事务在操作过程中进行两次查询，第二次查询的结果包含了第一次查询中未出现的数据或者缺少了第一次查询中出现的数据（这里并不要求两次查询的SQL语句相同）。这是因为在两次查询过程中有另外一个事务插入数据造成的。

“脏读”、“不可重复读”和“幻读”，其实都是数据库读一致性问题，必须由数据库提供一定的事务隔离机制来解决。

为了避免上面出现的几种情况，在标准SQL规范中，定义了4个事务隔离级别，由低到高依次为Read uncommitted、Read committed、Repeatable read、Serializable，这四个级别可以逐个解决脏读、不可重复读、幻读这几类问题。

![这里写图片描述](SpringFramework.assets/SouthEast-16784319472116.png)


未提交读取（Read Uncommitted）
Spring标识：ISOLATION_READ_UNCOMMITTED。允许脏读取，但不允许更新丢失。如果一个事务已经开始写数据，则另外一个事务则不允许同时进行写操作，但允许其他事务读此行数据。该隔离级别可以通过“排他写锁”实现。

已提交读取（Read Committed）
Spring标识：ISOLATION_READ_COMMITTED。允许不可重复读取，但不允许脏读取。这可以通过“瞬间共享读锁”和“排他写锁”实现。读取数据的事务允许其他事务继续访问该行数据，但是未提交的写事务将会禁止其他事务访问该行。

可重复读取（Repeatable Read）
Spring标识：ISOLATION_REPEATABLE_READ。禁止不可重复读取和脏读取，但是有时可能出现幻读数据。这可以通过“共享读锁”和“排他写锁”实现。读取数据的事务将会禁止写事务（但允许读事务），写事务则禁止任何其他事务。

序列化（Serializable）
Spring标识：ISOLATION_SERIALIZABLE。提供严格的事务隔离。它要求事务序列化执行，事务只能一个接着一个地执行，不能并发执行。仅仅通过“行级锁”是无法实现事务序列化的，必须通过其他机制保证新插入的数据不会被刚执行查询操作的事务访问到。

##### Timeout

事务需要在一定的时间内进行提交，如果规定时间内不提交就会进行回滚

默认值是-1，即不会超时

##### readOnly

readOnly默认值为false，表示可以查询，也可以添加修改删除操作

设置为true，则只可以进行查询操作

##### rollbackFor

设置出现哪些异常进行事务回滚

##### noRollbackFor

设置出现哪些异常不进行事务回滚

#### xml

![image-20230310151543615](SpringFramework.assets/image-20230310151543615.png)

![image-20230310151602659](SpringFramework.assets/image-20230310151602659.png)

# 整合日志框架

```xml
        <!-- 引入日志管理相关依赖-->
        <!--slf4j与log4j2实现所需要的日志依赖   start-->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.13.3</version>
        </dependency>
        <!--log4j-core中有log4j-api的依赖包 ,所以可以不添加og4j-api依赖-->
<!--        <dependency>-->
<!--            <groupId>org.apache.logging.log4j</groupId>-->
<!--            <artifactId>log4j-api</artifactId>-->
<!--            <version>2.13.3</version>-->
<!--        </dependency>-->
        <!--log4j2和slf4j的连接包 需要绑定到log4j2 core核心包上-->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-slf4j-impl</artifactId>
            <version>2.13.3</version>
        </dependency>
        <!--slf4j 日志门面-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.30</version>
        </dependency>
```

# @Nullable注解

可以使用的位置：

- 方法上面：表示返回值可以为空
- 属性上面：表示属性值可以为空
- 参数上面：表示参数值可以为空

# 函数式风格GenericApplicationContext/AnnotationConfigApplicationContext

```java
public class testGenericApplicationContext {
    @Test
    public void test(){
        GenericApplicationContext genericApplicationContext = new GenericApplicationContext();
        genericApplicationContext.refresh();
        genericApplicationContext.registerBean("generic",TestGeneric.class,()-> {
            return new TestGeneric();
        });
        TestGeneric generic = (TestGeneric) genericApplicationContext.getBean("generic");
        System.out.println("generic = " + generic);
    }
}
```

# 整合JUnit5单元测试框架

##### JUnit4

![image-20230306114803507](SpringFramework.assets/image-20230306114803507.png)

注意点：

- locations为编译后路径![image-20230306114909687](SpringFramework.assets/image-20230306114909687.png)
- 注意与后面的方法所导入的包不同![image-20230306114952186](SpringFramework.assets/image-20230306114952186.png)

依赖引入：

```xml

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
```



##### 整合JUnit5

![image-20230306115153026](SpringFramework.assets/image-20230306115153026.png)

依赖包如下

![image-20230306115218153](SpringFramework.assets/image-20230306115218153.png)

```xml
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.7.2</version>
            <scope>test</scope>
        </dependency>
```

# WebFlux

## 观察者模式

天气预报demo，传统方式

```java
//第三方
public class CurrentConditions {
    //温度，气压，湿度
    private float temperature;
    private float pressure;
    private float humidity;

    //weatherData调用
    public void update(float temperature,float pressure,float humidity){
        this.temperature=temperature;
        this.pressure=pressure;
        this.humidity=humidity;
        display();
    }

    private void display() {
        System.out.println("temperature = " + temperature);
        System.out.println("pressure = " + pressure);
        System.out.println("humidity = " + humidity);
    }
}
```



```java
//数据生产方
public class WeatherData {
    private float temperature;
    private float pressure;
    private float humidity;
    private CurrentConditions currentConditions;

    public WeatherData(CurrentConditions currentConditions) {
        this.currentConditions = currentConditions;
    }

    public float getTemperature() {
        return temperature;
    }

    public float getPressure() {
        return pressure;
    }

    public float getHumidity() {
        return humidity;
    }
    //调用接入方更新方法
    public void dataChange(){
        currentConditions.update(temperature,pressure,humidity);
    }
    //数据更新时进行调用
    public void setData(float temperature,float pressure,float humidity){
        this.temperature=temperature;
        this.pressure=pressure;
        this.humidity=humidity;
        //将最新消息推送给接入方currentConditions
        dataChange();
    }
}
```

![image-20230311125241479](SpringFramework.assets/image-20230311125241479.png)

缺点：

- 无法动态添加第三方
- 如果要添加第三方会违反ocp原则（开闭原则，对扩展开放，对修改关闭）

可以使用观察者模式进行改进

### 原理

观察者模式：对象之间是多对一依赖的一种设计方案，被依赖的对象为Subject，依赖的对象为Observer，Subject通知Observer变化，例如，在上面的例子中

气象局（数据生产方）：Subject

第三方（数据消费方）：Observer

![image-20230311130536777](SpringFramework.assets/image-20230311130536777.png)

![image-20230311132612399](SpringFramework.assets/image-20230311132612399.png)

```java
public class WeatherData implements Subject {
    private float temperature;
    private float pressure;
    private float humidity;
    private ArrayList<Observer> observers;

    public void setData(float temperature, float pressure, float humidity) {
        this.temperature = temperature;
        this.pressure = pressure;
        this.humidity = humidity;
        //将最新消息推送给接入方currentConditions
        notifyObservers();
    }

    public WeatherData() {
        observers = new ArrayList<Observer>();
    }

    @Override
    public void registerObserver(Observer o) {
        observers.add(o);
    }

    @Override
    public void removeObserver(Observer o) {
        observers.remove(o);
    }

    //遍历所有观察者，并遍历
    @Override
    public void notifyObservers() {
        for (int i = 0; i < observers.size(); i++) {
            observers.get(i).update(this.temperature, this.pressure, this.humidity);
        }
    }
}
```

![image-20230311132626644](SpringFramework.assets/image-20230311132626644.png)

```java

public class BaiduSite implements Observer{
    private float temperature;
    private float pressure;
    private float humidity;

    //weatherData调用
    @Override
    public void update(float temperature,float pressure,float humidity){
        this.temperature=temperature;
        this.pressure=pressure;
        this.humidity=humidity;
        display();
    }

    private void display() {
        System.out.println("baidu temperature = " + temperature);
        System.out.println("baidu pressure = " + pressure);
        System.out.println("baidu humidity = " + humidity);
    }
}
```

测试

![image-20230311132659706](SpringFramework.assets/image-20230311132659706.png)

### Observable

观察者模式在java中的应用：Observer，Observable

官方文档：

https://docs.oracle.com/javase/8/docs/api/java/util/Observer.html

https://docs.oracle.com/javase/8/docs/api/java/util/Observable.html

-------------

![image-20230311133703098](SpringFramework.assets/image-20230311133703098.png)

![image-20230311133724452](SpringFramework.assets/image-20230311133724452.png)

注意：

Observable是类，不是接口，类中已经实现了核心的方法，等价于前面说过的Subject

Oberver就是上面例子中的Observer



## 基本概念

![image-20230310211614575](SpringFramework.assets/image-20230310211614575.png)

Spring 框架中包含的原始 Web 框架 Spring Web MVC 是专门为 Servlet API 和 Servlet 容器构建的。响应式 Web 框架 Spring WebFlux 是后来在5.0版本中添加的。它是完全非阻塞的，支持反应流背压，并运行在 Netty，Undertwow 和 Servlet 3.1 + 容器等服务器上。

这两个 Web 框架都反映了它们源模块的名称(Spring-webmvc 和 Spring-webflow) ，并且在 Spring 框架中并存。每个模块都是可选的。应用程序可以使用其中一个模块，或者在某些情况下使用两个模块ーー例如，带有反应式 WebClient 的 Spring MVC 控制器。

#### 异步非阻塞

WebFlux是一种异步非阻塞的框架

异步和同步：针对调用者，调用者发送请求，如果等着对方回应之后才去做其他事情就是同步，如果发送请求之后不等着对方回应就去做其他事情就是异步。

阻塞和非阻塞：针对被调用者，被调用者收到请求之后，做完请求任务之后才给出反馈就是阻塞，收到请求之后马上给出反馈然后再去做其他事情就是非阻塞。

#### webflux特点：

- 反应式编程和非阻塞：反应和非阻塞通常不会使应用程序运行得更快。但在某些情况下，它们可以，例如，如果使用 WebClient 并行运行远程调用。总的来说，以非阻塞方式做事情需要更多的工作，这可能会略微增加所需的处理时间。

  反应式和非阻塞性的主要预期好处是能够使用少量、固定数量的线程和更少的内存进行伸缩。这使得应用程序在负载下更具弹性，因为它们以更可预测的方式扩展。然而，为了观察这些好处，您需要一些延迟(包括慢速和不可预测的网络 I/O 的混合)。这就是反应堆开始显示其优势的地方，而且差异可能是巨大的。

- 函数式编程：webflux中可以使用java8函数式编程方式实现路由请求

#### Spring MVC or WebFlux?

![image-20230310212535422](SpringFramework.assets/image-20230310212535422.png)

- 两个框架都可以使用注解方式，都运行在Tomcat等容器中
- SpringMVC采用命令式编程，WebFlux采用异步响应式编程

举例：

SpringCloud Gateway是基于Spring WebFlux的，因为网关要处理大量请求，需要异步非阻塞的框架。

## 响应式编程

https://ebook.qicoder.com/spring-in-action-5th/di-san-bu-fen-xiang-ying-shi-spring.html



https://blog.csdn.net/qq_37958845/article/details/119275357

--------------

### 基本介绍

背景：响应式编程是一种新的编程技术，其目的是构建响应式系统。对于响应式系统而言，任何时候都需要确保具备即时响应性，这是大多数日常业务场景所需要的，但却是一项非常复杂而有挑战性的任务。
说明：所谓的“响应式”并不是一件颠覆式的事情，而只是一种新型的编程模式。它不局限于某种开发框架，也并非解决分布式环境下所有问题的银弹，而是随着技术的发展自然而然诞生的一种技术体系。

### 原因

#### 传统 Web 请求

在服务 A（服务消费者）调用服务 B（服务提供者）场景中，当服务 A 向服务 B 发送 HTTP 请求时，线程 A 只有在发起请求和响应结果的一小部分时间内在有效使用 CPU，而更多的时间则只是在 阻塞式 地等待来自服务 B 中线程的处理结果。显然，整个过程的 CPU 利用效率是很低的，很多时间线程被浪费在了 I/O 阻塞上，无法执行其他的处理过程，如下图所示：

同理，在一个服务内部（以经典的 Web 分层服务为例），也存在着这种阻塞的情况， Controller 层访问 Service 层，Service 层访问 Repository 数据层，数据层访问数据库，然后再依次返回。在这个过程中，每一步的操作过程都存在着前面描述的线程等待问题。也就是说，整个技术栈中的每一个环节都可能是同步阻塞的。

#### 处理方式

针对上面提到的阻塞问题，在 Java 中，为了实现异步非阻塞，常用的方式有两种：回调和 Future，但是这两种方式，都存在一些局限性。

> 回调
> 回调体现的是一种双向的调用方式，实现了服务间的解耦，服务调用方会提供一个 callback 回调方法。在这个 callback 回调方法中，回调的执行是由任务的结果（服务提供方）来触发的，所以我们就可以异步来执行某项任务，从而使得调用链路不发生任何的阻塞。
> 回调的最大问题是复杂性，一旦在执行流程中包含了多层的异步执行和回调，那么就会形成一种嵌套结构，给代码的开发和调试带来很大的挑战。所以回调很难大规模地组合起来使用，因为很快就会导致代码难以理解和维护，从而造成所谓的“回调地狱”问题。
>
> Future
> Future 模式简单理解为这样一种场景：我们有一个需要处理的任务，然后把这个任务提交到 Future，Future 就会在一定时间内完成这个任务，而在这段时间内我们可以去做其他事情。
> 但从本质上讲，Future 以及由 Future 所衍生出来的 CompletableFuture 等各种优化方案就是一种多线程技术。多线程假设一些线程可以共享一个 CPU，而 CPU 时间能在多个线程之间共享，这一点就引入了“上下文切换”的概念。
> 如果想要恢复线程，就需要涉及加载和保存寄存器等一系列计算密集型的操作。因此，大量线程之间的相互协作同样会导致资源利用效率低下。



### 响应式编程实现方法

#### 观察者模式和发布-订阅模式

了解响应式编程技术之前，我们先回顾一下两种设计模式：观察者模式和发布-订阅模式。

观察者模式
观察者模式拥有一个主题（Subject），其中包含其依赖者列表，这些依赖者被称为观察者（Observer）。主题可以通过一定的机制将任何状态变化通知到观察者。

发布-订阅模式

java9中新添加的类：https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/Flow.html

发布-订阅模式，可以认为是对观察者模式的一种改进。因为观察者模式，容易和场景绑定（如：一个场景一个观察者模式），而发布-订阅模式具有更强的通用性。
在这一模式中，发布者和订阅者之间可以没有直接的交互，而是通过发送事件到事件处理平台的方式来完成整合，如下图所示：


了解了这两种模式，我们再来看有什么方式可以处理前面提到的阻塞问题？
如果将获取数据这件事情，通过发布订阅来实现，是不是就可以处理阻塞问题？在一个服务的内部，从 Web 服务层到数据访问层，再到数据库的整个调用链路，同样可以采用发布-订阅模式进行重构。这时候，我们希望当数据库中的数据一有变化（事件）就通知上游组件（通知机制），而不是上游组件通过主动拉取数据的方式来获取数据（阻塞）。

> 调用方不再阻塞等待，而是订阅事件，当事件发生变化的时候，调用方再来处理。



#### 数据流与响应式

基于上面的实现，那么在一个系统中，就会存在很多很多的 事件，每一种事件会基于用户的操作或者系统自身的行为而被触发，并形成了一个事件的集合。针对事件的集合，我们可以把它们看成是一串串联起来的数据流，而系统的响应能力就体现在对这些数据流的即时响应过程上。
数据流对于技术栈而言是一个全流程的概念。也就是说，无论是从底层数据库，向上到达服务层，最后到 Web 服务层，抑或是在这个流程中所包含的任意中间层组件，整个数据传递链路都应该是采用事件驱动的方式来进行运作的。
这样，我们就可以不采用传统的同步调用方式来处理数据，而是由处于数据库上游的各层组件自动来执行事件。这就是响应式编程的核心特点。
相较传统开发所普遍采用的“拉”模式，在响应式编程下，基于事件的触发和订阅机制，这就形成了一种类似“推”的工作方式。这种工作方式的优势就在于，生成事件和消费事件的过程是异步执行的，所以线程的生命周期都很短，也就意味着资源之间的竞争关系较少，服务器的响应能力也就越高。

### 背压机制

#### 基本概念

（1）流
所谓的流就是由生产者生产并由一个或多个消费者消费的元素序列。这种生产者/消费者模型也可以被称为发布者/订阅者模型。

（2）流的处理模型
流的处理，存在两种基本的实现机制：一种就是传统开发模式下的“拉”模式，即消费者主动从生产者拉取元素；而另一种就是上面提到的“推”模式。
在“推”模式下，生产者将元素推送给消费者。相较于“拉”模式，该模式下的数据处理的资源利用率更好。但是，这也引入了流量控制的问题，即如果数据的生产者和消费者处理数据的速度是不一致的，我们应该如何确保系统的稳定性呢？

#### 流量控制

（1）生产者生产数据的速率小于消费者的场景
在这种情况下，因为消费者消费数据没有任何压力，也就不需要进行流量的控制。

（2）生产者生产数据的速率大于消费者消费数据的场景
这种情况比较复杂，因为消费者可能因为无法处理过多的数据而发生崩溃。针对这种情况的一种常见解决方案是在生产者和消费者之间添加一种类似于消息队列的机制。我们知道队列具有存储并转发的功能，所以可以由它来进行一定的流量控制，效果如下图所示。

那么流量控制问题的关键就转变为了如何设计一种合适的队列？通常，我们可以选择三种不同类型的队列来分别支持不同的功能特性。

- 无界队列（Unbounded Queue）

  这种队列原则上拥有无限大小的容量，可以存放所有生产者所生产的消息；同样，因为无界，但系统资源确是有限的，容易出现内存耗尽情况，导致系统崩溃。

- 有界丢弃队列

  与无界队列相对的，更合适的方案是选择一种有界队列。它避免内存溢出的情况，但可能会出现消息丢失的情况，因此，它比较适合用于允许丢消息的业务场景，但在消息重要性很高的场景显然不可能采取这种队列。

- 有界阻塞队列

  如果需要确保消息不丢失，则需要引入有界阻塞队列。在这种队列中，我们会在队列消息数量达到上限后阻塞生产者，而不是直接丢弃消息。显然，这种阻塞行为是不可能实现异步操作的，即：有界阻塞队列都不是我们想要的解决方案。

#### 背压机制

通过对流量控制的分析，可以明确，纯“推”模式下的数据流量会有很多不可控制的因素，并不能直接应用，而是需要在“推”模式和“拉”模式之间考虑一定的平衡性，从而优雅地实现流量控制。这就需要引出响应式系统中非常重要的一个概念——背压机制（Backpressure）。
什么是背压？简单来说就是下游能够向上游反馈流量请求的机制。我们知道，如果消费者消费数据的速度赶不上生产者生产数据的速度时，它就会持续消耗系统的资源，直到这些资源被消耗殆尽。
这个时候，就需要有一种机制使得消费者可以根据自身当前的处理能力通知生产者来调整生产数据的速度，这种机制就是背压。采用背压机制，消费者会根据自身的处理能力来请求数据，而生产者也会根据消费者的能力来生产数据，从而在两者之间达成一种动态的平衡，确保系统的即时响应性。

#### 实现

为了实现这种动态的平衡，出现了一套响应式流规范，而针对流量控制的解决方案以及背压机制都包含在响应式流规范中，其中包含了响应式编程的各个核心组件。

### 响应式流规范

#### 核心接口

响应式流的规范可以通过四个接口定义来概括：Publisher，Subscriber，Subscription 和 Processor。Publisher 为每一个 Subscription 的 Subscriber 生产数据。Publisher 接口声明了一个 subscribe() 方法，通过这个方法 Subscriber 可以订阅 Publisher：

```java
public interface Publisher<T> {
    void subscribe(Subscriber<? super T> subscriber);
}
Copy
```

Subscriber 一旦进行了订阅，就可以从 Publisher 中接收消息，这些消息都是通过 Subscriber 接口中的方法进行发送：

```java
public interface Subscriber<T> {
    void onSubscribe(Subscription sub);
    void onNext(T item);
    void onError(Throwable ex);
    void onComplete();
}
Copy
```

> Subscriber 接口包含了一组方法，这些方法构成了数据流请求和处理的基本流程。
> onSubscribe() 方法：
>
> 从命名上看就是一个回调方法，当发布者的 subscribe() 方法被调用时就会触发这个回调。而在该方法中有一个参数 Subscription，可以把这个 Subscription 看作是一种用于订阅的上下文对象。Subscription 对象中包含了这次回调中订阅者想要向发布者请求的数据个数。
>
> onNext() 方法：
>
> 当订阅关系已经建立，那么发布者就可以调用订阅者的 onNext() 方法向订阅者发送一个数据。这个过程是持续不断的，直到所发送的数据已经达到 Subscription 对象中所请求的数据个数。
>
> onComplete() 方法：
>
> 当所发送的数据已经达到 Subscription 对象中所请求的数据个数时，这时候 onComplete() 方法就会被触发，代表这个数据流已经全部发送结束。
>
> onError() 方法：
>
> 一旦在在数据流发送过程中出现了异常，那么就会触发 onError() 方法，我们可以通过这个方法捕获到具体的异常信息进行处理，而数据流也就自动终止了。

Subscriber 通过调用 onSubscribe() 函数将会收到第一个消息。当 Publisher 调用 onSubscribe()，它通过一个 Subscription 对象将消息传输给 Subscriber。消息是通过 Subscription 进行传递的，Subscriber 可以管理他自己的订阅内容：

```java
public interface Subscription {
    void request(long n);
    void cancel();
}
Copy
```

Subscriber 可以调用 request() 去请求被被发送了的数据，或者调用 cancel() 来表明他对接收的数据不感兴趣，并取消订阅。当调用 request() 时，Subscriber 通过传递一个 long 值的参数来表示它将会接收多少数据。这时就会引进 backpressure，用以阻止 Publisher 发送的数据超过 Subscriber 能够处理的数据。在 Publisher 发送了足够的被请求的数据后，Subscriber 可以再次调用 request() 来请求更多的数据。

一旦 Subcriber 已经接收到数据，数据就通过流开始流动了。每一个 Publisher 发布的项目都会通过调用 onNext() 方法将数据传输到 Subscriber。如果出现错误，onError() 方法将被调用。如果 Publisher 没有更多的数据需要发送了，同时也不会再生产任何数据了，将会调用 onComplete() 方法来告诉 Subscriber，它已经结束了。

对于 Processor 接口而言，它连接了 Subscriber 和 Publisher：

```java
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {}
Copy
```

作为 Subscriber，Processor 将会接收数据然后以一定的方式处理这些数据。然后它会摇身一变，变为一个 Publisher，将处理的结果发布到 Subscriber。

正如你所看到的，响应式流规范相当地简单。关于如何从 Publisher 开始建立起一个数据处理的通道，这也是一件很容易的事情了，通过将数据不输入或是输入到多个 Processor 中，然后将最终结果传递到 Subscriber 中就行了。

### Reactor

Reactor有两个核心类，Mono和Flux，这两个类实现接口Publisher，提供丰富操作符。Flux对象实现发布者，返回N个元素；Mono实现翻发布者，返回0或者1个元素。

Flux和Mono都是数据流的发布者，使用Flux和Mono都可以发出三种数据信号

- 元素值
- 错误信号
- 完成信号

错误信号和完成信号都代表终止信号，终止信号用于告诉订阅者数据流结束了，错误信号终止数据流的同时会把错误信息传递给订阅者



## 执行流程和核心API



## 注解编程模型



## 函数时编程模型
