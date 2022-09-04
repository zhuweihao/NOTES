# IDEA打包

java文档知识

https://docs.oracle.com/javase/tutorial/deployment/jar/defman.html

### 一、IDEA自带打包插件

https://blog.csdn.net/Hugh_Guan/article/details/110224621

内容：此种方式可以自己选择制作胖包或者瘦包，但推荐此种方式制作瘦包。
输出：输出目录在out目录下
流程步骤：

第一步： 依次选择 file->projecct structure->artifacts->点击+ (选择jar)->选择 from module with dependencies

![img](https://img-blog.csdnimg.cn/20201127122646929.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1Z2hfR3Vhbg==,size_16,color_FFFFFF,t_70#pic_center)

第二步：弹出窗口中指定Main Class，是否选择依赖jar包，是否包含测试。（尽量不选依赖包，防止依赖包选择不全）

![img](https://img-blog.csdnimg.cn/20201127122646929.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1Z2hfR3Vhbg==,size_16,color_FFFFFF,t_70#pic_center)

![img](https://img-blog.csdnimg.cn/20201127122723284.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1Z2hfR3Vhbg==,size_16,color_FFFFFF,t_70#pic_center)

第三步：点击Build–>Build Artifacts–>选择bulid二、maven插件打包

并没有把依赖jar包也打进去。

![image-20220609142459481](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20220609142459481.png)

### 二、maven插件打包

**输出**：输出目录在target目录下

#### 2.1 制作瘦包（直接打包，不打包依赖包）

**内容**：仅打包出项目中的代码到JAR包中。
**方式**：在pom.xml中添加如下plugin; 随后执行maven install

```xml
 <!-- java编译插件 -->
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-compiler-plugin</artifactId>
	<version>指定版本</version>
	<configuration>
		<source>1.8</source>
		<target>1.8</target>
		<encoding>UTF-8</encoding>
	</configuration>
</plugin>
```

#### 2.2 制作瘦包和依赖包（相互分离）

**内容：**将依赖JAR包输出到lib目录方式（打包方式对于JAVA项目是通用的）将项目中的JAR包的依赖包输出到指定的目录下,修改outputDirectory配置，如下面的**${project.build.directory}/lib**。

**方式：**pom.xml的build>plugins中添加如下配置。

点击maven project（右边栏）->选择Lifecycle->点击package打包。
注意：如果想将打包好的JAR包通过命令直接运行，如java -jar xx.jar。需要制定manifest配置的classpathPrefix与上面配置的相对应。如上面把依赖JAR包输出到了lib，则这里的classpathPrefix也应指定为lib/；同时，并指定出程序的入口类，在配置mainClass节点中配好入口类的全类名。

```xml
<plugins>
<!-- java编译插件 -->
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-compiler-plugin</artifactId>
	<configuration>
        <!--source和target默认是1.5，对应的JDK版本是5，1.8对应JDK8-->
		<source>1.8</source>
		<target>1.8</target>
		<encoding>UTF-8</encoding>
	</configuration>
</plugin>
    <!--程序员自己写的包的配置-->
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-jar-plugin</artifactId>
	<configuration>
		<archive>
			<manifest>
				<addClasspath>true</addClasspath>
				<classpathPrefix>lib/</classpathPrefix>
				<mainClass>com.yourpakagename.mainClassName</mainClass>
			</manifest>
		</archive>
	</configuration>
</plugin>
    <!--依赖包的配置-->
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-dependency-plugin</artifactId>
	<executions>
		<execution>
			<id>copy</id>
			<phase>install</phase>
			<goals>
				<goal>copy-dependencies</goal>
			</goals>
			<configuration>
				<outputDirectory>${project.build.directory}/lib</outputDirectory>
			</configuration>
		</execution>
	</executions>
</plugin>
</plugins>

```

打包后的文件目录层级

![image-20220609141925806](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20220609141925806.png)

由此方式创建的MANIFEST文件内容是

```
Manifest-Version: 1.0
Built-By: lenovo
Created-By: Apache Maven 3.3.9
Build-Jdk: 1.8.0_65
Main-Class: com.sugon.App
```

**注意**：默认的classpath会在jar包内。为了方便,可以在Main方法配置后加上manifestEntries配置，指定classpath。

```xml
<plugin>  
	<groupId>org.apache.maven.plugins</groupId>  
	<artifactId>maven-jar-plugin</artifactId>  
	<configuration>  
		<classesDirectory>target/classes/</classesDirectory>  
		<archive>  
			<manifest>  
				<!-- 主函数的入口 -->  
				<mainClass>com.yourpakagename.mainClassName</mainClass>  
				<!-- 打包时 MANIFEST.MF文件不记录的时间戳版本 -->  
				<useUniqueVersions>false</useUniqueVersions>  
				<addClasspath>true</addClasspath>  
				<classpathPrefix>lib/</classpathPrefix>  
			</manifest>  
			<manifestEntries>  
				<Class-Path>.</Class-Path>  
			</manifestEntries>  
		</archive>  
	</configuration>  
</plugin>  

```

**扩展**

```
${basedir} 项目根目录
${project.build.directory} 构建目录，缺省为target
${project.build.outputDirectory} 构建过程输出目录，缺省为target/classes
project.build.finalName产出物名称，缺省为{project.artifactId}-${project.version}
${project.packaging} 打包类型，缺省为jar
${project.xxx} 当前pom文件的任意节点的内容
```

#### 2.3 制作胖包（项目依赖包和项目打为一个包）

**内容：**将项目中的依赖包和项目代码都打为一个JAR包
**方式：**pom.xml的build>plugins中添加如下配置；

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>  
	<artifactId>maven-assembly-plugin</artifactId>  
	<version>2.5.5</version>  
	<configuration>  
		<archive>  
			<manifest>  
				<mainClass>com.xxg.Main</mainClass>  
			</manifest>  
		</archive>  
		<descriptorRefs>  
			<descriptorRef>jar-with-dependencies</descriptorRef>  
		</descriptorRefs>  
	</configuration>  
</plugin> 

```

点击maven project（右边栏）->选择Plugins->选择assembly->点击assembly:assembly
注意：

1. 针对传统的JAVA项目打包；
2. 打包指令为插件的assembly命令，尽量不用package指令。



#### 2.4 制作胖包（transform部分自定义）

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-shade-plugin</artifactId>
	<version>2.4.3</version>
	<executions>
		<execution>
			<phase>package</phase>
			<goals>
				<goal>shade</goal>
			</goals>
			<configuration>
				<filters>
					<filter>
						<artifact>*:*</artifact>
						<excludes>
							<exclude>META-INF/*.SF</exclude>
							<exclude>META-INF/*.DSA</exclude>
							<exclude>META-INF/*.RSA</exclude>
						</excludes>
					</filter>
				</filters>
				<transformers>
					<transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
						<resource>META-INF/spring.handlers</resource>
					</transformer>
					<transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
						<resource>META-INF/spring.schemas</resource>
					</transformer>
					<transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
						<resource>META-INF/spring.tooling</resource>
					</transformer>
					<transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
						<mainClass>com.xxx.xxxInvoke</mainClass>
					</transformer>
				</transformers>
				<minimizeJar>true</minimizeJar>
				<shadedArtifactAttached>true</shadedArtifactAttached>
			</configuration>
		</execution>
	</executions>
</plugin>

```

### 三、加载外部配置

原文链接：https://blog.csdn.net/sayyy/article/details/81120749

背景知识

自JDK 1.2以后,JVM采用了委托(delegate)模式来载入class．采用这种设计的原因可以参考http://java.sun.com/docs/books/tutorial/ext/basics/load.html

Java虚拟机(JVM)寻找Class的顺序：

1. **Bootstrap classes**

属于Java 平台核心的class,比如java.lang.String等.及rt.jar等重要的核心级别的class.这是由JVM Bootstrap class loader来载入的.一般是放置在{java_home}\jre\lib目录下

2. **Extension classes**

基于Java扩展机制,用来扩展Java核心功能模块.比如Java串口通讯模块comm.jar.一般放置{Java_home}\jre\lib\ext目录下

3. **User classes**

开发人员或其他第三方开发的Java程序包，通过命令行的-classpath或-cp,或者通过设置CLASSPATH环境变量来引用。

JVM通过放置在{java_home}\lib\tools.jar来寻找和调用用户级的class。常用的java，c也是通过调用tools.jar来寻找用户指定的路径来编译Java源程序.这样就引出了User class路径搜索的顺序或优先级别的问题.

3.1  缺省值:调用Java或javawa的当前路径(.),是开发的class所存在的当前目录
3.2  CLASSPATH环境变量设置的路径.如果设置了CLASSPATH,则CLASSPATH的值会覆盖缺省值
3.3  执行Java的命令行-classpath或-cp的值,如果制定了这两个命令行参数之一,它的值会覆盖环境变量CLASSPATH的值
3.4  -jar 选项:如果通过java -jar 来运行一个可执行的jar包,这当前jar包会覆盖上面所有的值。换句话说,-jar 后面所跟的jar包的优先级别最高,如果指定了-jar选项,所有环境变量和命令行制定的搜索路径都将被忽略。**JVM APPClassloader**将只会以jar包为搜索范围。

这也是为什么应用程序打包成可执行的jar包后,不管你怎么设置classpath都不能引用到第三方jar包的东西了。

有关可执行jar有许多相关的安全方面的描述,可以参考http://java.sun.com/docs/books/tutorial/jar/ 来全面了解.

### 实验

问题：Java运行时，指定classpath外部配置文件，将替换内部的配置文件，该功能常用于灵活的配置文件更新。

设计个maven小项目，进行验证。

实验设计：

猜测：pom文件经过打包之后，会由maven生成pom.properties配置文件。

代码：

```java
public static void main(String[] args) throws IOException {
        Properties properties = new Properties();
        properties.load(ConnectToMysql.class.getClassLoader()
                               .getResourceAsStream("db.properties"));
        String username = properties.getProperty("username");
        String password = properties.getProperty("password");
        String url = properties.getProperty("url");
        System.out.println(username+"\n"+password+"\n"+url);
    }
```

**配置文件A:db.properties**

![image-20220612140037460](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20220612140037460.png)

```properties
username=root
password=123456
url=mysql://localhost:3306/my
```

控制台输入命令：

```shell
E:\javaworkspace\target>java -jar classpath-1.0-SNAPSHOT.jar

```

结果

```properties
root
123456
mysql://localhost:3306/my
```

**配置文件B db1.properties**

![image-20220612140233242](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20220612140233242.png)



```properties
username=user00
password=123123
url=mysql://localhost:3306/mine
```

控制台输出命令

```shell
E:\javaworkspace\target>java -Xbootclasspath/a:E:/javaworkspace/target/ -jar classpath-1.0-SNAPSHOT.jar
```

结果

```properties
user00
123123
mysql://localhost:3306/mine
```

得证

另外使用这种方式不会修改原来的配置文件的内容，在 Java官网jar的使用教程中，有修改和更新配置文件的功能，他们都对原来的配置文件进行了修改。

扩展资料，Springboot配置加载

https://blog.csdn.net/weixin_39550258/article/details/114556481?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2-114556481-blog-86217286.pc_relevant_blogantidownloadv1&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2-114556481-blog-86217286.pc_relevant_blogantidownloadv1&utm_relevant_index=2

