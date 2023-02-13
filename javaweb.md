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

