# Spark读取Kafka

## 数据结构：

```java
public class ConsumerRecord<K, V> {
    public static final long NO_TIMESTAMP = -1L;
    public static final int NULL_SIZE = -1;
    /** @deprecated */
    @Deprecated
    public static final int NULL_CHECKSUM = -1;
    private final String topic;
    private final int partition;
    private final long offset;
    private final long timestamp;
    private final TimestampType timestampType;
    private final int serializedKeySize;
    private final int serializedValueSize;
    private final Headers headers;
    private final K key;
    private final V value;
    private final Optional<Integer> leaderEpoch;
    }
```

```java
public abstract class DStream<T> implements Serializable, Logging {
    private transient StreamingContext ssc;
    private final ClassTag<T> evidence$1;
    private transient HashMap<Time, RDD<T>> generatedRDDs;
    private Time zeroTime;
    private Duration rememberDuration;
    private StorageLevel storageLevel;
    private final boolean mustCheckpoint;
    private Duration checkpointDuration;
    private final DStreamCheckpointData<T> checkpointData;
    private transient boolean restoredFromCheckpointData;
    private DStreamGraph graph;
    private final CallSite creationSite;
    private final Option<String> baseScope;
    private transient Logger org$apache$spark$internal$Logging$$log_;
    }
```

```java
public abstract class RDD<T> implements Serializable, Logging {
    private transient boolean isBarrier_;
    private Enumeration.Value outputDeterministicLevel;
    private transient SparkContext _sc;
    private transient Seq<Dependency<?>> deps;
    private final ClassTag<T> evidence$1;
    private final transient Option<Partitioner> partitioner;
    private final int id;
    private transient String name;
    private final Object stateLock;
    private volatile Seq<Dependency<?>> dependencies_;
    private transient volatile Partition[] partitions_;
    private StorageLevel storageLevel;
    private final transient CallSite creationSite;
    private final transient Option<RDDOperationScope> scope;
    private Option<RDDCheckpointData<T>> checkpointData;
    private final boolean checkpointAllMarkedAncestors;
    private transient boolean doCheckpointCalled;
    private transient Logger org$apache$spark$internal$Logging$$log_;
    private transient volatile boolean bitmap$trans$0;
    private volatile boolean bitmap$0;
    }
```

```java
public class JavaDStream<T> extends AbstractJavaDStreamLike<T, JavaDStream<T>, JavaRDD<T>> {
    private final DStream<T> dstream;
    private final ClassTag<T> classTag;
    }
```

## 主程序：

### wordcount

```java
package com.atsugon;

import java.util.*;
import org.apache.spark.SparkConf;
import org.apache.spark.TaskContext;
import org.apache.spark.api.java.*;
import org.apache.spark.api.java.function.*;
import org.apache.spark.streaming.Durations;
import org.apache.spark.streaming.api.java.*;
import org.apache.spark.streaming.kafka010.*;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.common.TopicPartition;
import org.apache.kafka.common.serialization.StringDeserializer;
import scala.Tuple2;

public class readFromKafka {
    public static void main(String[] args) throws InterruptedException {
        // Create a local StreamingContext with two working thread and batch interval of 1 second
        SparkConf conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount");
        JavaStreamingContext jssc = new JavaStreamingContext(conf, Durations.seconds(1));

        Map<String, Object> kafkaParams = new HashMap<>();
       // kafkaParams.put("bootstrap.servers", "localhost:9092,anotherhost:9092");
        kafkaParams.put("bootstrap.servers", "localhost:9092");
        kafkaParams.put("key.deserializer", StringDeserializer.class);
        kafkaParams.put("value.deserializer", StringDeserializer.class);
        kafkaParams.put("group.id", "use_a_separate_group_id_for_each_stream");
        kafkaParams.put("auto.offset.reset", "earliest");
       // kafkaParams.put("enable.auto.commit", false);
        //Collection<String> topics = Arrays.asList("topicA", "topicB");
        Collection<String> topics = Arrays.asList("topicA");
        JavaInputDStream<ConsumerRecord<String, String>> stream =
                KafkaUtils.createDirectStream(
                        jssc,
                        LocationStrategies.PreferConsistent(),
                        ConsumerStrategies.<String, String>Subscribe(topics, kafkaParams)
                );
        stream.foreachRDD(x->{
            if(x.count()>0)
                x.foreach(s-> System.out.println(s.key()+":"+s.value()));
        });
//        //stream.print();

        JavaDStream<String> stringJavaDStream = stream.map(x -> {
            return x.value();
        }).flatMap(x -> Arrays.asList(x.split(" ")).iterator());


        // Count each word in each batch
        JavaPairDStream<String, Integer> pairs =stringJavaDStream.mapToPair(s -> new Tuple2<>(s, 1));
        JavaPairDStream<String, Integer> wordCounts = pairs.reduceByKey((i1, i2) -> i1 + i2);

        wordCounts.print();
        //start the job
        jssc.start();
        jssc.awaitTermination();
    }
}

```

### 保存wordcount

```java
package com.atsugon;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.TextOutputFormat;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.spark.SparkConf;
import org.apache.spark.streaming.Durations;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaInputDStream;
import org.apache.spark.streaming.api.java.JavaPairDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.kafka010.ConsumerStrategies;
import org.apache.spark.streaming.kafka010.KafkaUtils;
import org.apache.spark.streaming.kafka010.LocationStrategies;
import scala.Tuple2;

import java.util.Arrays;
import java.util.Collection;
import java.util.HashMap;
import java.util.Map;

public class ReadFromKafkaWriteToFTP {
    public static void main(String[] args) throws InterruptedException {
        // Create a local StreamingContext with two working thread and batch interval of 1 second
        SparkConf conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount");
        JavaStreamingContext jssc = new JavaStreamingContext(conf, Durations.seconds(1));

        Map<String, Object> kafkaParams = new HashMap<>();
        // kafkaParams.put("bootstrap.servers", "localhost:9092,anotherhost:9092");
        kafkaParams.put("bootstrap.servers", "localhost:9092");
        kafkaParams.put("key.deserializer", StringDeserializer.class);
        kafkaParams.put("value.deserializer", StringDeserializer.class);
        kafkaParams.put("group.id", "use_a_separate_group_id_for_each_stream");
        kafkaParams.put("auto.offset.reset", "earliest");
        Collection<String> topics = Arrays.asList("topicA");
        JavaInputDStream<ConsumerRecord<String, String>> stream =
                KafkaUtils.createDirectStream(
                        jssc,
                        LocationStrategies.PreferConsistent(),
                        ConsumerStrategies.<String, String>Subscribe(topics, kafkaParams)
                );
        stream.foreachRDD(x->{
            if(x.count()>0)
                x.foreach(s-> System.out.println(s.key()+":"+s.value()));
        });
//        //stream.print();

        JavaDStream<String> stringJavaDStream = stream.map(x -> {
            return x.value();
        }).flatMap(x -> Arrays.asList(x.split(" ")).iterator());


        // Count each word in each batch
        JavaPairDStream<String, Integer> pairs =stringJavaDStream.mapToPair(s -> new Tuple2<>(s, 1));
        JavaPairDStream<String, Integer> wordCounts = pairs.reduceByKey((i1, i2) -> i1 + i2);

        wordCounts.print();
        wordCounts.saveAsHadoopFiles("/usr/local/datas/prefix","suffix",
                Text.class, IntWritable.class, TextOutputFormat.class);

        //start the job
        jssc.start();
        jssc.awaitTermination();
    }
}

```

```
-------------------------------------------
Time: 1657267220000 ms
-------------------------------------------
(hello,2)
```

![image-20220708163607612](https://s2.loli.net/2022/07/08/uEsXLzcg8jRWhAl.png)

### 上传功能

```java
package com.atsugon;

import cn.hutool.core.io.FileUtil;
import cn.hutool.extra.ftp.Ftp;
import cn.hutool.extra.ftp.FtpMode;
import sun.net.ftp.FtpClient;

import java.io.File;
import java.io.IOException;


public class FtpUpload {
    public static void main(String[] args) throws IOException {
        final Ftp ftp = new Ftp("hadoop102",21,"ftp",null);
        //ftp.upload("/home/vsftpd/ftp-user1/", FileUtil.file("/usr/local/datas/student.txt"));

        
       // ftp = ftp.reconnectIfTimeout();
        ftp.cd("pub");
        System.out.println(ftp.ls("pub"));
        System.out.println("pwd:"+ftp.pwd());
        ftp.setMode(FtpMode.Passive);
        //ftp.getClient().enterLocalPassiveMode();
       //上传本地文件
       // File file = FileUtil.file("/usr/local/datas/student.txt");

       //文件都要是绝对路径
        boolean flag = ftp.upload(null, new File("/usr/local/datas/student.txt"));
        System.out.println(flag);
        System.out.println(ftp.ls("pub"));
        ftp.close();
    }
}
/var/ftp/pub
```

#### 第一次

```
pwd:/pub
[test]
true
[student.txt, test]
```

#### 第二次

```
[student.txt, test]
pwd:/pub
false
[student.txt, test]
```

从上述结果可以看出，普通用户的上传重名的文件不会覆写，不会上传。

### 代码合并

```java
package com.atsugon;

import cn.hutool.core.io.FileUtil;
import cn.hutool.core.util.CharsetUtil;
import cn.hutool.extra.ftp.Ftp;
import cn.hutool.extra.ftp.FtpMode;
import org.apache.commons.io.FileUtils;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.TextOutputFormat;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.function.VoidFunction;
import org.apache.spark.streaming.Durations;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaInputDStream;
import org.apache.spark.streaming.api.java.JavaPairDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.kafka010.ConsumerStrategies;
import org.apache.spark.streaming.kafka010.KafkaUtils;
import org.apache.spark.streaming.kafka010.LocationStrategies;
import scala.Tuple2;

import java.io.File;
import java.io.IOException;
import java.util.*;

public class ReadFromKafkaWriteToFTP {
    public static void main(String[] args) throws InterruptedException, IOException {
        // Create a local StreamingContext with two working thread and batch interval of 1 second
        SparkConf conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount");
        JavaStreamingContext jssc = new JavaStreamingContext(conf, Durations.seconds(15));

        Map<String, Object> kafkaParams = new HashMap<>();
        // kafkaParams.put("bootstrap.servers", "localhost:9092,anotherhost:9092");
        kafkaParams.put("bootstrap.servers", "localhost:9092");
        kafkaParams.put("key.deserializer", StringDeserializer.class);
        kafkaParams.put("value.deserializer", StringDeserializer.class);
        kafkaParams.put("group.id", "use_a_separate_group_id_for_each_stream");
        kafkaParams.put("auto.offset.reset", "earliest");
        Collection<String> topics = Arrays.asList("topicA");
        JavaInputDStream<ConsumerRecord<String, String>> stream =
                KafkaUtils.createDirectStream(
                        jssc,
                        LocationStrategies.PreferConsistent(),
                        ConsumerStrategies.<String, String>Subscribe(topics, kafkaParams)
                );
        stream.foreachRDD(new VoidFunction<JavaRDD<ConsumerRecord<String, String>>>() {
            @Override
            public void call(JavaRDD<ConsumerRecord<String, String>> consumerRecordJavaRDD) throws Exception {
                consumerRecordJavaRDD.foreachPartition(new VoidFunction<Iterator<ConsumerRecord<String, String>>>() {
                    @Override
                    public void call(Iterator<ConsumerRecord<String, String>> consumerRecordIterator) throws Exception {
                        Ftp ftpClient = new Ftp("hadoop102", 21, "ftp", null, CharsetUtil.CHARSET_UTF_8, FtpMode.Passive);

                        List<String> list = new ArrayList<>();

                        while (consumerRecordIterator.hasNext()) {

                            final ConsumerRecord<String,String> value = consumerRecordIterator.next();

                            list.add(value.value().toString());

                            if (list.size() > 10) {
                                FileUtils.writeLines(new File("/usr/local/datas/wordCounts.txt"), "utf-8", list, true);
                            }
                        }
                        if (list.size() > 0) {
                            FileUtils.writeLines(new File("/usr/local/datas/wordCounts.txt"), "utf-8", list, true);
                        }
                        ftpClient.cd("pub");
                        boolean upload = ftpClient.upload("pub", new File("/usr/local/datas/wordCounts.txt"));
                        System.out.println("upload:"+upload);
                        ftpClient.close();
                    }
                });
            }
        });

        //start the job
        jssc.start();
        jssc.awaitTermination();
    }
}

```

结果

```
upload:true
upload:false
```

![image-20220712103823528](E:\image\image-20220712103823528.png)

```
vim wordCounts.txt
```

![image-20220712103928999](E:\image\image-20220712103928999.png)

查看ftp

![image-20220712104138191](E:\image\image-20220712104138191.png)



```
vim wordCounts.txt
```

![image-20220712104234023](https://s2.loli.net/2022/07/12/lIC5dGQFfoPtZOh.png)

总结：

同名文件的提交，ftp第一次提交第二次拒绝。

使用时间等作为文件的后缀

是否可以覆写呢？

是否可以提交文件夹？

使用文件名+时间的格式来保存多个文件

```java
package com.atsugon;

import cn.hutool.core.io.FileUtil;
import cn.hutool.core.util.CharsetUtil;
import cn.hutool.extra.ftp.Ftp;
import cn.hutool.extra.ftp.FtpMode;
import org.apache.commons.io.FileUtils;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.TextOutputFormat;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.function.VoidFunction;
import org.apache.spark.streaming.Durations;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaInputDStream;
import org.apache.spark.streaming.api.java.JavaPairDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.kafka010.ConsumerStrategies;
import org.apache.spark.streaming.kafka010.KafkaUtils;
import org.apache.spark.streaming.kafka010.LocationStrategies;
import scala.Tuple2;

import java.io.File;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.*;

public class ReadFromKafkaWriteToFTP {
    public static void main(String[] args) throws InterruptedException, IOException {
        // Create a local StreamingContext with two working thread and batch interval of 1 second
        SparkConf conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount");
        JavaStreamingContext jssc = new JavaStreamingContext(conf, Durations.seconds(15));

        Map<String, Object> kafkaParams = new HashMap<>();
        // kafkaParams.put("bootstrap.servers", "localhost:9092,anotherhost:9092");
        kafkaParams.put("bootstrap.servers", "localhost:9092");
        kafkaParams.put("key.deserializer", StringDeserializer.class);
        kafkaParams.put("value.deserializer", StringDeserializer.class);
        kafkaParams.put("group.id", "use_a_separate_group_id_for_each_stream");
        kafkaParams.put("auto.offset.reset", "earliest");
        Collection<String> topics = Arrays.asList("topicA");
        JavaInputDStream<ConsumerRecord<String, String>> stream =
                KafkaUtils.createDirectStream(
                        jssc,
                        LocationStrategies.PreferConsistent(),
                        ConsumerStrategies.<String, String>Subscribe(topics, kafkaParams)
                );
        stream.foreachRDD(new VoidFunction<JavaRDD<ConsumerRecord<String, String>>>() {
            @Override
            public void call(JavaRDD<ConsumerRecord<String, String>> consumerRecordJavaRDD) throws Exception {
                consumerRecordJavaRDD.foreachPartition(new VoidFunction<Iterator<ConsumerRecord<String, String>>>() {
                    @Override
                    public void call(Iterator<ConsumerRecord<String, String>> consumerRecordIterator) throws Exception {
                        Ftp ftpClient = new Ftp("hadoop102", 21, "ftp", null, CharsetUtil.CHARSET_UTF_8, FtpMode.Passive);
                        ftpClient.cd("pub");
                        boolean flag=false;
                        List<String> list = new ArrayList<>();
                        Date date = new Date();
                        SimpleDateFormat dateFormat= new SimpleDateFormat("MM-dd:hh:mm:ss");

                        String time = dateFormat.format(date);
                        System.out.println(time);
                        if (consumerRecordIterator.hasNext()){
                            flag=true;
                        }
                        while (consumerRecordIterator.hasNext()) {

                            final ConsumerRecord<String,String> value = consumerRecordIterator.next();

                            list.add(value.value().toString());

                            if (list.size() > 10) {
                                FileUtils.writeLines(new File("/usr/local/datas/wordCounts"+time+".txt"), "utf-8", list, true);
                            }
                        }
                        if (list.size() > 0) {
                            FileUtils.writeLines(new File("/usr/local/datas/wordCounts"+time+".txt"), "utf-8", list, true);
                        }
                        if (flag){
                            boolean upload = ftpClient.upload(null, new File("/usr/local/datas/wordCounts"+time+".txt"));
                            System.out.println("upload:"+upload);
                        }
                        ftpClient.close();
                    }
                });
            }
        });
        //start the job
        jssc.start();
        jssc.awaitTermination();
    }
}

```

输出

```
07-11:08:11:32
upload:true

07-11:08:11:45

07-11:08:12:00
upload:true
```

![image-20220712111532048](E:\image\image-20220712111532048.png)

![image-20220712111703148](E:\image\image-20220712111703148.png)





### 打包部署成可执行jar包

pom文件配置

```pom
<plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.3.0</version>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                        <configuration>
                            <finalName>demo</finalName>
                            <descriptorRefs>
                                <descriptorRef>jar-with-dependencies</descriptorRef>
                            </descriptorRefs>
                            <outputDirectory>output</outputDirectory>
                            <archive>
                                <manifest>
                                    <addClasspath>true</addClasspath>
                                    <mainClass>com.atsugon.ReadFromKafkaWriteToFTP</mainClass>                             <!-- 你的主类名 -->
                                </manifest>
                            </archive>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
```

可运行，运行正确

部署spark-submit提交

```
spark-submit --master local[2] /root/IdeaProjects/FlinkTutorial/Spark/output/demo-jar-with-dependencies.jar

```

BUG

```shell
Exception in thread "main" java.lang.NoSuchMethodError: scala.Product.$init$(Lscala/Product;)V
	at org.apache.spark.streaming.kafka010.PreferConsistent$.<init>(LocationStrategy.scala:38)
	at org.apache.spark.streaming.kafka010.PreferConsistent$.<clinit>(LocationStrategy.scala)
	at org.apache.spark.streaming.kafka010.LocationStrategies$.PreferConsistent(LocationStrategy.scala:56)
	at org.apache.spark.streaming.kafka010.LocationStrategies.PreferConsistent(LocationStrategy.scala)
	at com.atsugon.ReadFromKafkaWriteToFTP.main(ReadFromKafkaWriteToFTP.java:49)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.spark.deploy.JavaMainApplication.start(SparkApplication.scala:52)
	at org.apache.spark.deploy.SparkSubmit.org$apache$spark$deploy$SparkSubmit$$runMain(SparkSubmit.scala:855)
	at org.apache.spark.deploy.SparkSubmit.doRunMain$1(SparkSubmit.scala:161)
	at org.apache.spark.deploy.SparkSubmit.submit(SparkSubmit.scala:184)
	at org.apache.spark.deploy.SparkSubmit.doSubmit(SparkSubmit.scala:86)
	at org.apache.spark.deploy.SparkSubmit$$anon$2.doSubmit(SparkSubmit.scala:930)
	at org.apache.spark.deploy.SparkSubmit$.main(SparkSubmit.scala:939)
	at org.apache.spark.deploy.SparkSubmit.main(SparkSubmit.scala)
```

目前程序的依赖使用的是2.12版本的scala，而spark的版本是2.10.6，版本不匹配，因此需要更换Spark版本。

更换之后运行成功。

```

```

### 使用分区号来区别文件+从配置文件读取配置。

#### 1.参考

Properties 类是 Hashtable 的子类，该对象用于处理属性文件
由于属性文件里的 key、value 都是字符串类型，所以 Properties 里的 key 和 value 都是字符串类型
存取数据时，建议使用setProperty(String key,String value)方法和getProperty(String key)方法

```java
 //Properties用来读取配置文件
    @Test
    public void test1() throws IOException {

        Properties pros =  new Properties();
// 此时的文件默认在当前的module下。
// 读取配置文件的方式一：
//使用字节流或字符流进行加载Properties文件 ， 然后创建Properties对象进行加载 ， 然后进行读取
 //存取数据时，建议使用setProperty(String key,String value)方法和getProperty(String key)方法
        FileInputStream fis = new FileInputStream("jdbc.properties");
         fis = new FileInputStream("src\\jdbc1.properties");
        pros.load(fis);

//        读取配置文件的方式二：使用ClassLoader
//        配置文件默认识别为：当前module的src下,无论当前用来获取类加载器的类位置在哪里
        ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();//获取类加载器
        InputStream is = classLoader.getResourceAsStream("jdbc.properties");//类加载器加载Properties配置文件
        pros.load(is);//Properties文件加载相应的流
        String user = pros.getProperty("user");
        String password = pros.getProperty("password");
        System.out.println("user = " + user + ",password = " + password);
    }

```

在[maven](https://so.csdn.net/so/search?q=maven&spm=1001.2101.3001.7020)工程中，我们会将配置文件放到src/main/resources下面，例如

我们需要确认resource 下的文件编译之后存放的位置。

它编译的路径直接位于classes下面，这个路径其实就是classPath的路径，所以，在resources [根目录](https://so.csdn.net/so/search?q=根目录&spm=1001.2101.3001.7020)下的配置文件其实就是 classPath的路径。

```java
public static void main(String[] args) throws ParserConfigurationException, Exception{  

        ClassLoader classLoader = TestDom.class.getClassLoader();  

        URL resource = classLoader.getResource("test.xml");  

        String path = resource.getPath();  

        System.out.println(path);  

        InputStream resourceAsStream = classLoader.getResourceAsStream("test.xml"); 
```


这样我们就可以直接拿到路径，调用 getResourceAsStream 方法 可以直接拿到目标文件的输入流。

几种读取配置文件的方式比较(代码在src/main/java目录下，资源文件在src/main/resources/目录下)：

```java
InputStream is = this.getClass().getResourceAsStream(test.xml);  //拿不到资源

InputStream is = this.getClass().getResourceAsStream("/" +test.xml); // 拿到资源

InputStream is = this.getClass().getClassLoader().getResourceAsStream(test.xml); //拿到资源

//第一种方式会从当前类的目录下去找，这个文件如果不和该类在一个目录下，就找不到。

//第二种方式会从编译后的整个classes目录下去找，maven也会把资源文件打包进classes文件夹，所以可以找到。

//第三种方式中ClassLoader就是从整个classes目录找的，所以前面无需再加/。
```

 

#### 2.实战

```java
package com.atsugon;

import cn.hutool.core.io.FileUtil;
import cn.hutool.core.util.CharsetUtil;
import cn.hutool.extra.ftp.Ftp;
import cn.hutool.extra.ftp.FtpMode;
import org.apache.commons.io.FileUtils;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.TextOutputFormat;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.spark.SparkConf;
import org.apache.spark.TaskContext;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.function.VoidFunction;
import org.apache.spark.streaming.Durations;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaInputDStream;
import org.apache.spark.streaming.api.java.JavaPairDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.kafka010.ConsumerStrategies;
import org.apache.spark.streaming.kafka010.KafkaUtils;
import org.apache.spark.streaming.kafka010.LocationStrategies;
import scala.Tuple2;

import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.text.SimpleDateFormat;
import java.util.*;

public class ReadFromKafkaWriteToFTP {
    public static void main(String[] args) throws InterruptedException, IOException {
        // Create a local StreamingContext with two working thread and batch interval of 1 second
        SparkConf conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount");
        JavaStreamingContext jssc = new JavaStreamingContext(conf, Durations.seconds(15));
        //从配置文件读取配置
        Properties properties = new Properties();
        // 读取配置文件的方式二：使用ClassLoader
        // 配置文件默认识别为：当前module的src下,无论当前用来获取类加载器的类位置在哪里
        //获取类加载器
        ClassLoader classLoader = ReadFromKafkaWriteToFTP.class.getClassLoader();
        //类加载器加载Properties配置文件
        InputStream is = classLoader.getResourceAsStream("kafka.properties");
        properties.load(is);//Properties文件加载相应的流
        String  bootstrap_servers= properties.getProperty("bootstrap_servers");
        String group_id = properties.getProperty("group_id");
        String auto_offset_reset = properties.getProperty("auto_offset_reset");
        String topics = properties.getProperty("topics");

        //get properties of ftp
        Properties propertiesFtp = new Properties();
        InputStream isFtp = classLoader.getResourceAsStream("ftp.properties");
        propertiesFtp.load(isFtp);//Properties文件加载相应的流
        String  host= propertiesFtp.getProperty("host");
        int port = Integer.parseInt(propertiesFtp.getProperty("port"));
        String user = propertiesFtp.getProperty("user");
        String password = propertiesFtp.getProperty("password");
        String catalogue = propertiesFtp.getProperty("catalogue");

        System.out.println("  "+host + " " + port+" "+user+" "+password+" ** "+catalogue);
        Map<String, Object> kafkaParams = new HashMap<>();
        // kafkaParams.put("bootstrap.servers", "localhost:9092,anotherhost:9092");
        kafkaParams.put("bootstrap.servers", bootstrap_servers);
        kafkaParams.put("key.deserializer", StringDeserializer.class);
        kafkaParams.put("value.deserializer", StringDeserializer.class);
        kafkaParams.put("group.id", group_id);
        kafkaParams.put("auto.offset.reset", auto_offset_reset);
        //Collection<String> topicList = Arrays.asList("topicA");
        Collection<String> topicList=Arrays.asList(topics.split(","));
        JavaInputDStream<ConsumerRecord<String, String>> stream =
                KafkaUtils.createDirectStream(
                        jssc,
                        LocationStrategies.PreferConsistent(),
                        ConsumerStrategies.<String, String>Subscribe(topicList, kafkaParams)
                );
        //JavaDStream<ConsumerRecord<String, String>> repartition = stream.repartition(2);
        stream.foreachRDD(new VoidFunction<JavaRDD<ConsumerRecord<String, String>>>() {
            @Override
            public void call(JavaRDD<ConsumerRecord<String, String>> consumerRecordJavaRDD) throws Exception {
                consumerRecordJavaRDD.foreachPartition(new VoidFunction<Iterator<ConsumerRecord<String, String>>>() {
                    @Override
                    public void call(Iterator<ConsumerRecord<String, String>> consumerRecordIterator) throws Exception {
                        Ftp ftpClient = new Ftp(host, port, user, password, CharsetUtil.CHARSET_UTF_8, FtpMode.Passive);
                        ftpClient.cd(catalogue);
                        boolean flag=false;
                        List<String> list = new ArrayList<>();
                        int partitionId = TaskContext.getPartitionId();
                        System.out.println("partitionId:"+partitionId);
                        Date date = new Date();
                        SimpleDateFormat dateFormat= new SimpleDateFormat("MM-dd:hh:mm:ss");
                        String time = dateFormat.format(date);
                        System.out.println(time);
                        if (consumerRecordIterator.hasNext()){
                            flag=true;
                        }
                        while (consumerRecordIterator.hasNext()) {

                            final ConsumerRecord<String,String> value = consumerRecordIterator.next();

                            list.add(value.value().toString());

                            if (list.size() > 10) {
                                FileUtils.writeLines(new File("/usr/local/datas/wordCounts"+partitionId+time+".txt"), "utf-8", list, true);
                            }
                        }
                        if (list.size() > 0) {
                            FileUtils.writeLines(new File("/usr/local/datas/wordCounts"+partitionId+time+".txt"), "utf-8", list, true);
                        }
                        if (flag){
                            boolean upload = ftpClient.upload(null, new File("/usr/local/datas/wordCounts"+partitionId+time+".txt"));
                            System.out.println("upload:"+upload);
                        }
                        ftpClient.close();
                    }
                });
            }
        });
        //start the job
        jssc.start();
        jssc.awaitTermination();
    }
}

```

#### 打包

```
 spark-submit --master local[2] --properties-file /usr/local/datas/conf/kafka.properties /root/IdeaProjects/FlinkTutorial/Spark/output/demo-jar-with-dependencies.jar
 
  spark-submit --master local[2] --properties-file /usr/local/datas/conf/ftp.properties /root/IdeaProjects/FlinkTutorial/Spark/output/demo-jar-with-dependencies.jar

 spark-submit --master local[2] --properties-file /usr/local/datas/conf/kafka.properties --properties-file  /usr/local/datas/conf/ftp.properties/root/IdeaProjects/FlinkTutorial/Spark/output/demo-jar-with-dependencies.jar

上述三个都不行，--properties-file仅用来指定spark的配置参数的。
```

```
[root@hadoop102 output]# java -Xbootclasspath/a:/usr/local/datas/conf/ -jar demo-jar-with-dependencies.jar 
运行成功！！！！
```

```
java -jar -Dloader.path=/usr/local/datas/conf/ demo-jar-with-dependencies.jar
失败
```

```
java -jar -Dspring.config.location=/usr/local/datas/conf/ftp.properties demo-jar-with-dependencies.jar
失败
```

因为-jar会覆盖路径，让我们先在jar包内找。



然后如果说我们即要配置spark的参数，又要修改kafka和ftp的参数该怎末办呢？

不会了。。。

去assembly插件看看能否把配置文件单独摘出来，里面配置好配置文件的路径，然后只需要修改配置路径里的配置文件，这样就可以解决上述问题了。

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>FlinkTutorial</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>Spark</artifactId>

    <name>Spark</name>
    <!-- FIXME change it to the project's website -->
    <url>http://www.example.com</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-streaming-kafka-0-10_2.12</artifactId>
            <version>3.0.0</version>
        </dependency>
        <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-core</artifactId>
        <version>2.10.3</version>
        </dependency>
        <!--        ftp上传-->
        <dependency>
            <groupId>commons-net</groupId>
            <artifactId>commons-net</artifactId>
            <version>3.8.0</version>
        </dependency>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.2.0</version>
        </dependency>
    </dependencies>

    <build>
        <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
            <plugins>
                <!-- clean lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#clean_Lifecycle -->
                <plugin>
                    <artifactId>maven-clean-plugin</artifactId>
                    <version>3.1.0</version>
                </plugin>
                <!-- default lifecycle, jar packaging: see https://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_jar_packaging -->
                <plugin>
                    <artifactId>maven-resources-plugin</artifactId>
                    <version>3.0.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.0</version>
                </plugin>
                <plugin>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <version>2.22.1</version>
                </plugin>
                <plugin>
                    <artifactId>maven-jar-plugin</artifactId>
                    <version>3.0.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-install-plugin</artifactId>
                    <version>2.5.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-deploy-plugin</artifactId>
                    <version>2.8.2</version>
                </plugin>
                <!-- site lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#site_Lifecycle -->
                <plugin>
                    <artifactId>maven-site-plugin</artifactId>
                    <version>3.7.1</version>
                </plugin>
                <plugin>
                    <artifactId>maven-project-info-reports-plugin</artifactId>
                    <version>3.0.0</version>
                </plugin>
            </plugins>
        </pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.3.0</version>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                        <configuration>

                            <finalName>demo</finalName>
<!--                            <descriptorRefs>-->
<!--                                <descriptorRef>jar-with-dependencies</descriptorRef>-->
<!--                            </descriptorRefs>-->
                            <descriptors>${project.basedir}/src/main/resources/package.xml</descriptors>
                            <outputDirectory>output</outputDirectory>
                            <archive>
                                <manifest>
                                    <addClasspath>true</addClasspath>
                                    <classpathPrefix>lib</classpathPrefix>
                                    <mainClass>com.atsugon.ReadFromKafkaWriteToFTP</mainClass>
                                </manifest>
                                <manifestEntries>
                                    <Class-Path>../conf/</Class-Path>
<!--                                    <Class-Path>../conf/kafka.properties</Class-Path>-->
                                </manifestEntries>
                            </archive>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>

```



```xml
<assembly>
    <!--打包名称，唯一标识-->
    <id>demo</id>
    <!--打包格式，可以手动修改-->
    <formats>
        <format>jar</format>
    </formats>
<!--    very important-->
    <includeBaseDirectory>false</includeBaseDirectory>
    <!--文件设置-->

    <fileSets>
<!--        <fileSet>-->
<!--            &lt;!&ndash;目标目录,会处理目录里面的所有文件&ndash;&gt;-->
<!--            <directory>${project.basedir}/src/main/resources</directory>-->
<!--            &lt;!&ndash;相对于打包后的目录&ndash;&gt;-->
<!--            <outputDirectory>conf</outputDirectory>-->
<!--            &lt;!&ndash;文件过滤&ndash;&gt;-->
<!--            <includes>-->
<!--                <include>*.*</include>-->
<!--            </includes>-->
<!--        </fileSet>-->

            <fileSet>
                <directory>${project.build.directory}/classes</directory>
                <outputDirectory>/</outputDirectory>
            </fileSet>

    </fileSets>
<!--    <files>-->
<!--        &lt;!&ndash;包含打包后的jar文件，可以不加入<outputDirectory/>,默认打包的目录&ndash;&gt;-->
<!--        <file>-->
<!--            <source>${project.build.directory}/${project.build.finalName}.jar</source>-->
<!--        </file>-->
<!--    </files>-->
<!--    <dependencySets>-->
    <dependencySets>
        <dependencySet>
            <outputDirectory>/</outputDirectory>
            <unpack>true</unpack>
            <scope>runtime</scope>
        </dependencySet>
    </dependencySets>

</assembly>
```

实验结果失败

另外一种方式

当我们把jar包打好后，且配置了外部的配置路径，我们将配置文件取出放在我们配置好的外部配置路径。然后再打包不包含配置文件的jar包，再运行来查看结果。

实验

```
<assembly>
    <!--打包名称，唯一标识-->
    <id>demo1</id>
    <!--打包格式，可以手动修改-->
    <formats>
        <format>jar</format>
    </formats>
<!--    very important-->
    <includeBaseDirectory>false</includeBaseDirectory>
    <!--文件设置-->

    <fileSets>
<!--        <fileSet>-->
<!--            &lt;!&ndash;目标目录,会处理目录里面的所有文件&ndash;&gt;-->
<!--            <directory>${project.basedir}/src/main/resources</directory>-->
<!--            &lt;!&ndash;相对于打包后的目录&ndash;&gt;-->
<!--            <outputDirectory>../conf</outputDirectory>-->
<!--            &lt;!&ndash;文件过滤&ndash;&gt;-->
<!--            <includes>-->
<!--                <include>*.*</include>-->
<!--            </includes>-->
<!--        </fileSet>-->

<!--            <fileSet>-->
<!--                <directory>${project.build.directory}/classes</directory>-->
<!--                <outputDirectory>/</outputDirectory>-->
<!--                <excludes>-->
<!--                    <exclude>ftp.properties</exclude>-->
<!--                    <exclude>kafka.properties</exclude>-->
<!--                    <exclude>package.xml</exclude>-->
<!--                </excludes>-->
<!--            </fileSet>-->

    </fileSets>
<!--    <files>-->
<!--        &lt;!&ndash;包含打包后的jar文件，可以不加入<outputDirectory/>,默认打包的目录&ndash;&gt;-->
<!--        <file>-->
<!--            <source>${project.build.directory}/${project.build.finalName}.jar</source>-->
<!--        </file>-->
<!--    </files>-->

    <dependencySets>
        <dependencySet>
            <outputDirectory>/</outputDirectory>
            <unpack>true</unpack>
            <excludes>
                <exclude>ftp.properties</exclude>
                <exclude>kafka.properties</exclude>
                <exclude>package.xml</exclude>
            </excludes>
            <scope>runtime</scope>
        </dependencySet>
    </dependencySets>

</assembly>
```

实验无法排除配置文件。

上述畅想没有实现。再讨论吧。

## kafka位置偏移量是否需要手动管理

https://blog.csdn.net/qq_36329973/article/details/104825902

### Kafka-0.10.1.X版本之后的auto.kafka.reset：

|          |                                                              |
| -------- | ------------------------------------------------------------ |
| earliest | 当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费 |
| latest   | 当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据 |
| none     | topic各分区都存在已提交的offset时，从offset后开始消费；只要有一个分区不存在已提交的offset，则抛出异常 |


​	
​	
​	



# 卫哥参考代码

```java
package com.sugon.kafka;

import java.io.File;
import java.util.*;

import cn.hutool.core.util.CharsetUtil;
import cn.hutool.extra.ftp.Ftp;
import cn.hutool.extra.ftp.FtpMode;
import org.apache.commons.io.FileUtils;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.VoidFunction;
import org.apache.spark.streaming.Durations;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaInputDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.kafka010.*;

/**
 * /usr/sdp/5.0.0.RELEASE/kafka/bin/kafka-consumer-groups.sh  --bootstrap-server 172.22.5.15:9092 --describe --group group2
 * {"plate_color_name": "蓝", "plate_no": "京A57562", "zone": "气象局大院", "location": "116.476915,39.809484", "createTime": "2021-09-21 15:38:30", "endTime": "2021-09-21 15:56:30"}
 **/
public class SparkKafkaTest {

    public static void main(String[] args) throws Exception {
        String brokers = "172.22.5.15:9092";
        String topics = "flink_ww_02";

        SparkConf sparkConf = new SparkConf().setMaster("local").setAppName("sparkKafkaTest");
        JavaStreamingContext jssc = new JavaStreamingContext(sparkConf, Durations.seconds(15));

        jssc.sparkContext().setLogLevel("WARN");
        Set<String> topicsSet = new HashSet<>(Arrays.asList(topics.split(",")));
        Map<String, Object> kafkaParams = new HashMap<>();
        kafkaParams.put("bootstrap.servers", brokers);
        kafkaParams.put("group.id", "group2");
        kafkaParams.put("auto.offset.reset", "earliest");
//        kafkaParams.put("enable.auto.commit", "false");
        kafkaParams.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        kafkaParams.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

        JavaInputDStream<ConsumerRecord<Object, Object>> lines = KafkaUtils.createDirectStream(
                jssc,
                LocationStrategies.PreferConsistent(),
                ConsumerStrategies.Subscribe(topicsSet, kafkaParams)
        );


        JavaDStream<String> flatResult = lines.map(new Function<ConsumerRecord<Object, Object>, String>() {
            @Override
            public String call(ConsumerRecord<Object, Object> v1) throws Exception {
                return v1.value().toString();
            }
        });

        lines.foreachRDD(new VoidFunction<JavaRDD<ConsumerRecord<Object, Object>>>() {
            @Override
            public void call(JavaRDD<ConsumerRecord<Object, Object>> consumerRecordJavaRDD) throws Exception {

                consumerRecordJavaRDD.foreachPartition(new VoidFunction<Iterator<ConsumerRecord<Object, Object>>>() {

                    /**
                     * 1. 不同分区 写不同的文件，文件命名问题 ？
                     * 2. 防止某个文件过大 控制写文件的大小 ？
                     * 3. 文件清理工作，上传成功，进行清理
                     * 4. kafka 偏移量维护
                       5. 任务运行
                     * @param consumerRecordIterator
                     * @throws Exception
                     */
                    @Override
                    public void call(Iterator<ConsumerRecord<Object, Object>> consumerRecordIterator) throws Exception {

                        Ftp ftpClient = new Ftp("10.0.13.236", 21, "sugon", "sugon_456", CharsetUtil.CHARSET_UTF_8, FtpMode.Passive);

                        List<String> list = new ArrayList<>();

                        while (consumerRecordIterator.hasNext()) {

                            final ConsumerRecord<Object, Object> value = consumerRecordIterator.next();

                            list.add(value.value().toString());

                            if (list.size() > 10) {
                                FileUtils.writeLines(new File("/Users/weiwei/dev/log/cs.txt"), "utf-8", list, true);
                            }
                        }
                        if (list.size() > 0) {
                            FileUtils.writeLines(new File("/Users/weiwei/dev/log/cs.txt"), "utf-8", list, true);
                        }
                        ftpClient.cd(".");

                        ftpClient.upload(".", new File("/Users/weiwei/dev/log/cs.txt"));

                        ftpClient.close();
                    }

                });
            }
        });


//        lines.foreachRDD(rdd -> {
//            OffsetRange[] offsetRanges = ((HasOffsetRanges) rdd.rdd()).offsetRanges();
//
//            System.out.println("offsetRanges: " + Arrays.toString(offsetRanges));
//            // some time later, after outputs have completed
//            ((CanCommitOffsets) lines.inputDStream()).commitAsync(offsetRanges);
//        });


        flatResult.print();

        jssc.start();

        jssc.awaitTermination();


//        jssc.awaitTerminationOrTimeout(40000);
    }

}
```

# 问题代码（已解决，未更新）

```java
package com.atsugon;

import cn.hutool.core.io.FileUtil;
import cn.hutool.extra.ftp.Ftp;
import cn.hutool.extra.ftp.FtpMode;
import org.apache.commons.io.FileUtils;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.TextOutputFormat;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.function.VoidFunction;
import org.apache.spark.streaming.Durations;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaInputDStream;
import org.apache.spark.streaming.api.java.JavaPairDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.kafka010.ConsumerStrategies;
import org.apache.spark.streaming.kafka010.KafkaUtils;
import org.apache.spark.streaming.kafka010.LocationStrategies;
import scala.Tuple2;

import java.io.File;
import java.io.IOException;
import java.util.*;

public class ReadFromKafkaWriteToFTP {
    public static void main(String[] args) throws InterruptedException, IOException {
        // Create a local StreamingContext with two working thread and batch interval of 1 second
        SparkConf conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount");
        JavaStreamingContext jssc = new JavaStreamingContext(conf, Durations.seconds(15));

        Map<String, Object> kafkaParams = new HashMap<>();
        // kafkaParams.put("bootstrap.servers", "localhost:9092,anotherhost:9092");
        kafkaParams.put("bootstrap.servers", "localhost:9092");
        kafkaParams.put("key.deserializer", StringDeserializer.class);
        kafkaParams.put("value.deserializer", StringDeserializer.class);
        kafkaParams.put("group.id", "use_a_separate_group_id_for_each_stream");
        kafkaParams.put("auto.offset.reset", "earliest");
        Collection<String> topics = Arrays.asList("topicA");
        JavaInputDStream<ConsumerRecord<String, String>> stream =
                KafkaUtils.createDirectStream(
                        jssc,
                        LocationStrategies.PreferConsistent(),
                        ConsumerStrategies.<String, String>Subscribe(topics, kafkaParams)
                );
        stream.foreachRDD(x->{
            if(x.count()>0)
                x.foreach(s-> System.out.println(s.key()+":"+s.value()));
        });
//        //stream.print();

        JavaDStream<String> stringJavaDStream = stream.map(x -> {
            return x.value();
        }).flatMap(x -> Arrays.asList(x.split(" ")).iterator());


        // Count each word in each batch
        JavaPairDStream<String, Integer> pairs =stringJavaDStream.mapToPair(s -> new Tuple2<>(s, 1));
        JavaPairDStream<String, Integer> wordCounts = pairs.reduceByKey((i1, i2) -> i1 + i2);

        wordCounts.print();
        wordCounts.foreachRDD(new VoidFunction<JavaPairRDD<String, Integer>>() {
            @Override
            public void call(JavaPairRDD<String, Integer> stringIntegerJavaPairRDD) throws Exception {
                
                Ftp ftp = new Ftp("hadoop102",21,"ftp",null);
                ftp.cd("pub");
                System.out.println("pwd:"+ftp.pwd());
                ftp.setMode(FtpMode.Passive);
                List<String> list = new ArrayList<>();
                while(!stringIntegerJavaPairRDD.isEmpty()){
                    list.add()
                    FileUtils.writeLines(new File("/Users/weiwei/dev/log/cs.txt"), "utf-8", list, true);

//                    boolean flag = ftp.upload(null, new File("/usr/local/datas/wordcount.txt"));
//                    System.out.println(flag);
//                    System.out.println(ftp.ls("pub"));
//                    ftp.close();
                }
            }
        });
        //start the job
        jssc.start();
        jssc.awaitTermination();
    }
}

```

