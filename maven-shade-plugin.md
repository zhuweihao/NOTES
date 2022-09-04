### 介绍

[Apache Maven Shade Plugin – Introduction](https://maven.apache.org/plugins/maven-shade-plugin/index.html)

shade用来将项目打包成一个可执行jar包，包括其依赖包和一些重命名的依赖包

官网上有这样一个名词uber-jar，就是我们常说的fat-jar胖包

通常情况之下jar包用来存储编译好的class文件和resources文件。如果java工程当中依赖于第三方库，那么仅仅打包本项目中编译好的class和resources而成的jar包无法直接运行，仍然需要额外的依赖和引入第三方包。fat-jar就是将依赖的第三方库也打包放入已经编译好的jar中，形成一个“All-in-one”的不需要依赖其他任何第三方包可独立运行部署的jar。

网络解释如下：

![image-20220720231831571](https://s2.loli.net/2022/07/20/eXt6TYyvG7owuEI.png)

![image-20220720232205177](https://s2.loli.net/2022/07/20/QC6qKI3TfEyncrO.png)



shade只能绑定在package阶段

### 用法示例

#### 控制项目依赖

artifactSet用于控制整个包，filter可以在jar包内部实现更细致地控制，既可以移除代码文件，也可以移除配置文件。

```xml
<artifactSet>
    <excludes>
        <exclude>org.apache.flink:force-shading</exclude>
        <exclude>com.google.code.findbugs:jsr305</exclude>
        <exclude>org.slf4j:*</exclude>
        <exclude>log4j:*</exclude>
    </excludes>
</artifactSet>
<filters>
    <filter>
        <!-- Do not copy the signatures in the META-INF folder.
        Otherwise, this might cause SecurityExceptions when using the JAR. -->
        <artifact>*:*</artifact>
        <excludes>
            <exclude>META-INF/*.SF</exclude>
            <exclude>META-INF/*.DSA</exclude>
            <exclude>META-INF/*.RSA</exclude>
        </excludes>
    </filter>
</filters>
```

除了用户指定的过滤器之外，该插件还可以配置为自动删除项目未使用的所有依赖项类，从而最大限度地减小生成的uber JAR

```xml
<configuration>
	<minimizeJar>true</minimizeJar>
</configuration>
```

为项目中的类指定包含筛选器会隐式排除该项目中所有未指定的类，下面的配置将覆盖此行为，以便仍然包含所有未指定的类。

```xml
<excludeDefaults>false</excludeDefaults>

<filter>
    <artifact>foo:bar</artifact>
    <excludeDefaults>false</excludeDefaults>
    <includes>
        <include>foo/Bar.class</include>
    </includes>
</filter>
```

#### 重定位类

如果 uber-JAR 作为其他项目的依赖项被重用，则直接在 uber JAR 中包含来自工件依赖项的类可能会由于类路径上的重复类而导致类加载冲突。为了解决这个问题，可以重新定位包含在shade工件中的类，以创建其字节码的私有副本

```xml
<relocations>
    <relocation>
        <pattern>org.codehaus.plexus.util</pattern>
        <shadedPattern>org.shaded.plexus.util</shadedPattern>
        <excludes>
            <exclude>org.codehaus.plexus.util.xml.Xpp3Dom</exclude>
            <exclude>org.codehaus.plexus.util.xml.pull.*</exclude>
        </excludes>
    </relocation>
</relocations>
```



#### 附加shade工件

默认情况下，shade插件会覆盖基于项目的jar包，而生成包含所有依赖的jar包。但有时需要原始的jar包和shade后的jar包同时被部署，可以配置如下。

```xml
<configuration>
  <shadedArtifactAttached>true</shadedArtifactAttached>
  <!-- 名称会作为后缀在shade构件jar包后 -->
  <shadedClassifierName>jackofall</shadedClassifierName> 
</configuration>
```



#### 资源转换

在打包时，存在将多个构件中的class文件或资源文件聚合的需求。shade插件提供了丰富的Transformer工具类。这里介绍一些常用的Transformer。

##### ManifestResourceTransformer

往MANIFEST文件中写入Main-Class是可执行包的必要条件。ManifestResourceTransformer可以轻松实现。

```xml
<transformers>
    <transformer
            implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
        <mainClass>com.zhuweihao.SparkStreaming.kafka</mainClass>
    </transformer>
</transformers>
```

##### AppendingTransformer

用来处理多个jar包中存在重名的配置文件的合并，尤其是spring。

```xml
<configuration>
	<transformers>
		<transformer
			implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
			<resource>META-INF/spring.handlers</resource>
		</transformer>
		<transformer
			implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
			<resource>META-INF/spring.schemas</resource>
		</transformer>
	</transformers>
</configuration>

```

##### ServicesResourceTransformer

提供某些接口实现的 JAR 文件通常附带一个 `META-INF/services/` 目录，该目录将接口映射到其实现类，以供服务定位器查找。要重新定位这些实现类的类名，并将同一接口的多个实现合并到一个服务条目中，可以使用 `ServicesResourceTransformer`

```xml
<configuration>
    <transformers>
        <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
    </transformers>
</configuration>
```

#### BigData-Spark配置

```xml
<build>
    <plugins>
        <!-- We use the maven-shade plugin to create a fat jar that contains all necessary dependencies. -->
        <!-- Change the value of <mainClass>...</mainClass> if your program entry point changes. -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.3.0</version>
            <executions>
                <!-- Run shade goal on package phase -->
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <artifactSet>
                            <excludes>
                                <exclude>org.apache.flink:force-shading</exclude>
                                <exclude>com.google.code.findbugs:jsr305</exclude>
                                <exclude>org.slf4j:*</exclude>
                                <exclude>log4j:*</exclude>
                            </excludes>
                        </artifactSet>
                        <filters>
                            <filter>
                                <!-- Do not copy the signatures in the META-INF folder.
                                Otherwise, this might cause SecurityExceptions when using the JAR. -->
                                <artifact>*:*</artifact>
                                <excludes>
                                    <exclude>META-INF/*.SF</exclude>
                                    <exclude>META-INF/*.DSA</exclude>
                                    <exclude>META-INF/*.RSA</exclude>
                                </excludes>
                            </filter>
                        </filters>
                        <transformers>
                            <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <mainClass>com.zhuweihao.SparkStreaming.kafka</mainClass>
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>

        <!-- Java Compiler -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>

        <!-- Scala Compiler -->
        <plugin>
            <groupId>org.scala-tools</groupId>
            <artifactId>maven-scala-plugin</artifactId>
            <version>2.15.2</version>
            <executions>
                <execution>
                    <id>scala-compile</id>
                    <goals>
                        <goal>compile</goal>
                    </goals>
                    <configuration>
                        <!--includes是一个数组，包含要编译的code-->
                        <includes>
                            <include>**/*.scala</include>
                        </includes>
                    </configuration>
                </execution>
                <execution>
                    <id>scala-test-compile</id>
                    <goals>
                        <goal>testCompile</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
        
    </plugins>
</build>
```

### Flink开源打包实例

```
./mvnw clean package -DskipTests #linux # this will take up to 10 minutes
mvnw clean package -DskipTests #windows
```

下载开源项目后，在flink目录下执行上面的命令，该打包过程耗时巨长，官方文档说this will take up to 10 minutes，远远不止

![image-20220721011223249](https://s2.loli.net/2022/07/21/zFBjmL6u92CnNRQ.png)

ps：我笑了，我以为成功了，结果就这样还没成功，莫名其妙

#### pom.xml

```xml
<plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-shade-plugin</artifactId>
   <configuration>
      <!-- This section contains the core configuration that is applied to every jar that we create.-->
      <filters combine.children="append">
         <filter>
            <artifact>*</artifact>
            <excludes>
               <!-- Globally exclude log4j.properties from our JAR files. -->
               <exclude>log4j.properties</exclude>
               <exclude>log4j2.properties</exclude>
               <exclude>log4j-test.properties</exclude>
               <exclude>log4j2-test.properties</exclude>
               <!-- Do not copy the signatures in the META-INF folder.
               Otherwise, this might cause SecurityExceptions when using the JAR. -->
               <exclude>META-INF/*.SF</exclude>
               <exclude>META-INF/*.DSA</exclude>
               <exclude>META-INF/*.RSA</exclude>
               <!-- META-INF/maven can contain 2 things:
                  - For archetypes, it contains an archetype-metadata.xml.
                  - For other jars, it contains the pom for all dependencies under the respective <groupId>/<artifactId>/ directory.

                  We want to exclude the poms because they may be under an incompatible license,
                  however the archetype metadata is required and we need to keep that around.

                  This pattern excludes directories under META-INF/maven.
                  ('?*/**' does not work because '**' also matches zero directories;
                  everything that matches '?*' also matches '?*/**')

                  The initial '**' allows the pattern to also work for multi-release jars that may contain such entries under
                  'META-INF/versions/11/META-INF/maven/'.
                  -->
               <exclude>**/META-INF/maven/?*/?*/**</exclude>
            </excludes>
         </filter>
      </filters>
      <transformers combine.children="append">
         <!-- The service transformer is needed to merge META-INF/services files -->
         <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
         <!-- The ApacheNoticeResourceTransformer collects and aggregates NOTICE files -->
         <transformer implementation="org.apache.maven.plugins.shade.resource.ApacheNoticeResourceTransformer">
            <projectName>Apache Flink</projectName>
            <encoding>UTF-8</encoding>
         </transformer>
      </transformers>
   </configuration>
   <executions>
      <execution>
         <id>shade-flink</id>
         <phase>package</phase>
         <goals>
            <goal>shade</goal>
         </goals>
         <configuration>
            <shadeTestJar>false</shadeTestJar>
            <shadedArtifactAttached>false</shadedArtifactAttached>
            <createDependencyReducedPom>true</createDependencyReducedPom>
            <dependencyReducedPomLocation>${project.basedir}/target/dependency-reduced-pom.xml</dependencyReducedPomLocation>
            <!-- Filters MUST be appended; merging filters does not work properly, see MSHADE-305 -->
            <filters combine.children="append">
               <!-- drop entries into META-INF and NOTICE files for the dummy artifact -->
               <filter>
                  <artifact>org.apache.flink:flink-shaded-force-shading</artifact>
                  <excludes>
                     <exclude>**</exclude>
                  </excludes>
               </filter>
               <!-- io.netty:netty brings its own LICENSE.txt which we don't need -->
               <filter>
                  <artifact>io.netty:netty</artifact>
                  <excludes>
                     <exclude>META-INF/LICENSE.txt</exclude>
                  </excludes>
               </filter>
            </filters>
            <artifactSet>
               <includes>
                  <!-- Unfortunately, the next line is necessary for now to force the execution
                  of the Shade plugin upon all sub modules. This will generate effective poms,
                  i.e. poms which do not contain properties which are derived from this root pom.
                  In particular, the Scala version properties are defined in the root pom and without
                  shading, the root pom would have to be Scala suffixed and thereby all other modules.
                  Removing this exclusion will also cause compilation errors in at least
                  1 module (flink-connector-elasticsearch5), for unknown reasons.
                  -->
                  <include>org.apache.flink:flink-shaded-force-shading</include>
               </includes>
            </artifactSet>
         </configuration>
      </execution>
   </executions>
</plugin>
```



#### bug

![image-20220721000225293](https://s2.loli.net/2022/07/21/Sau9Ip8rBDUk6ej.png)

太特么扯淡了，downloading半天都没下好，百度后解决方案如下：

- 要么配置一下代理，重新执行就好了
- 要么手动下载链接文件，放到对应的目录里

要挂网下载，放到对应目录后注意修改名称



打包成功后target下没有相应的jar包，奇奇怪怪

未解决



```
Failed to execute goal com.diffplug.spotless:spotless-maven-plugin:2.13.0:check (spotless-check) on project flink-annotations: The following files had format violations:
```

删除后重新下载打包后出现上面的报错，

原因猜测：第一次下载是在idea中导入项目后进行打包的，这次是下载项目后直接进行打包的，idea可能在导入项目的时候进行了一些配置

解决方案：

[构建IoTDB数据库时未能执行目标com.diffplug.spotless:spotless-maven-plugin - 我爱学习网 (5axxw.com)](https://www.5axxw.com/questions/content/q2dnay)

我们使用spotless来格式化代码。在编译或提交之前，请运行`mvn spotless:apply`格式化代码。



### DataX开源打包实例

```
mvn -U clean package assembly:assembly -Dmaven.test.skip=true
```

github下载项目后，在datax目录下执行上面的命令进行打包

![image-20220721011336261](https://s2.loli.net/2022/07/21/WxVrpUmHYjQFwC6.png)

#### 配置文件

##### pom.xml

```xml
<build>
    <plugins>
        <plugin>
            <artifactId>maven-assembly-plugin</artifactId>
            <configuration>
                <finalName>datax</finalName>
                <descriptors>
                    <descriptor>package.xml</descriptor>
                </descriptors>
            </configuration>
            <executions>
                <execution>
                    <id>make-assembly</id>
                    <phase>package</phase>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.3.2</version>
            <configuration>
                <source>${jdk-version}</source>
                <target>${jdk-version}</target>
                <encoding>${project-sourceEncoding}</encoding>
            </configuration>
        </plugin>
    </plugins>
</build>
```

##### package.xml

```xml
<assembly>
    <id>datax</id>
    <formats>
        <format>tar.gz</format>
        <format>dir</format>
    </formats>
    <includeBaseDirectory>false</includeBaseDirectory>
    <fileSets>
        <fileSet>
            <directory>transformer/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>core/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>

        <!-- reader -->
        <fileSet>
            <directory>mysqlreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>oceanbasev10reader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>drdsreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>oraclereader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>sqlserverreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>db2reader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>postgresqlreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>kingbaseesreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>rdbmsreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>

        <fileSet>
            <directory>odpsreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>otsreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
		<fileSet>
             <directory>otsstreamreader/target/datax/</directory>
             <includes>
                 <include>**/*.*</include>
             </includes>
             <outputDirectory>datax</outputDirectory>
         </fileSet>
        <fileSet>
            <directory>txtfilereader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>ossreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>mongodbreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>tdenginereader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>streamreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
         <fileSet>
            <directory>ftpreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>hdfsreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>hbase11xreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>hbase094xreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>opentsdbreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>cassandrareader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>gdbreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>hbase11xsqlreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>hbase20xsqlreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>tsdbreader/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>

        <!-- writer -->
        <fileSet>
            <directory>mysqlwriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>tdenginewriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>drdswriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>odpswriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>txtfilewriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>ftpwriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>osswriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>adswriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>streamwriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>otswriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>mongodbwriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
		<fileSet>
            <directory>oraclewriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>sqlserverwriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
		<fileSet>
            <directory>postgresqlwriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>kingbaseeswriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>rdbmswriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>ocswriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>hdfswriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>hbase11xwriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>hbase094xwriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>hbase11xsqlwriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>elasticsearchwriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>hbase20xsqlwriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>tsdbwriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>adbpgwriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>cassandrawriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>clickhousewriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>oscarwriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>oceanbasev10writer/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>gdbwriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>kuduwriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>hologresjdbcwriter/target/datax/</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
            <outputDirectory>datax</outputDirectory>
        </fileSet>
    </fileSets>
</assembly>

```



#### 打包结果

![image-20220721004755528](https://s2.loli.net/2022/07/21/G9r6AZcFPYwWT7f.png)

### assembly与shade对比

#### assembly

- 优点
  - assembly不会将依赖的jar包合并，如果shade打fat jar，所有相关的类（含依赖）会被打进一个jar包，此时的问题是这个包除了比较大外，还失去了通过替换jar包更新程序的灵活性。
  - 支持定制化打包方式，assembly除了打包依赖外，还能include用户定义的目录或文件。如一般项目都会在bin目录中放启动脚本也可以打包一些配置文件等。
- 缺点
  - 手动依赖修剪。assembly虽然可以通过定义include/exclude修剪依赖，但是需要用户明确自己的代码中用到了哪些，没用到哪些，否则如果应该include没有include或被exclude了，那么很容易出`No Class Found Exception`

#### shade

- 优点
  - 支持修剪不必要的依赖，不像assembly需要用户自己进行修剪，shade能过通过字节码分析自动修剪掉不必要的依赖。
  - 能够通过替换包名避免依赖冲突
- 缺点
  - 不能打瘦包，只能打胖包
  - 不能打包脚本，配置文件等

#### 

