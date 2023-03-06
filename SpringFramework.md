# IOC

#### IOC原理

xml解析 -> 工厂模式 -> 反射

ioc容器底层就是对象工厂

Spring提供IOC容器的两种实现方式（两个接口）

- BeanFactory：IOC容器基本实现，加载配置文件的时候不会创建对象，在获取（使用）对象的时候才去创建对象
- ApplicationContext：BeanFactory的子接口，加载配置文件的时候就会创建对象

![image-20230227113043983](SpringFramework.assets/image-20230227113043983.png)

![image-20230227113546889](SpringFramework.assets/image-20230227113546889.png)

### Bean管理

bean管理指的是两个操作

- spring创建对象
- spring注入属性

bean管理操作有两种方式

- 基于xml配置文件方式
- 基于注解方式

#### 基于xml方式

##### 创建对象

![image-20230227114842618](SpringFramework.assets/image-20230227114842618.png)

- id：唯一标识
- class：全路径名

创建对象的时候默认执行无参构造方法完成对象创建

##### 注入属性

DI：dependency injection，依赖注入，是IOC的一种实现方式

属性注入方法：

- 使用set方法进行注入

![image-20230306201259897](SpringFramework.assets/image-20230306201259897.png)

- 有参构造方法注入，要编写有参构造函数

![image-20230306203054402](SpringFramework.assets/image-20230306203054402.png)

![image-20230306203130412](SpringFramework.assets/image-20230306203130412.png)

- p空间注入，其实也是用set方法进行注入，可以在set方法上打断点进行观察，“q”是可以自定义的

![image-20230306204047024](SpringFramework.assets/image-20230306204047024.png)

##### 注入其他类型属性

> 字面量

###### 空值

![image-20230306204941036](SpringFramework.assets/image-20230306204941036.png)

###### 属性值包含特殊符号

> 把特殊字符进行转义

把特殊字符内容写到CDATA

![image-20230306205643401](SpringFramework.assets/image-20230306205643401.png)

###### 外部bean

![image-20230306210114918](SpringFramework.assets/image-20230306210114918.png)

###### 内部bean和级联赋值



# 日志测试等功能



#### 整合JUnit5单元测试框架

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

