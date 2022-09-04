## 实验

## PageRank

### PageRank介绍

![image-20220707102701467](E:\image\image-20220707102701467.png)



算法步骤实现介绍：

- 将每一个URL的权重都设置为1.
- 对于每一次迭代，将每一个URL的权重贡献（权重/指向的URL个个数）发送给邻居
- 对于每一个URL，将收到的权重贡献相加成contribs，重新计算ranks=0.15+0.85*contribs。

```
[(A,D), (B,A), (C,A), (C,B), (D,A), (D,C)]
[(A,1.0), (B,1.0), (C,1.0), (D,1.0)]

(A,D)
(B,A)
(C,A)
(C,B)
(D,A)
(D,C)

(A,1.0)
(B,1.0)
(C,1.0)
(D,1.0)

(B,(A,1.0))
(A,(D,1.0))
(C,(A,1.0))
(C,(B,1.0))
(D,(A,1.0))
(D,(C,1.0))

(B,(A,1.0))
(A,(D,1.0))
(C,(A,0.5))
(C,(B,0.5))
(D,(A,0.5))
(D,(C,0.5))
得到一个contribs
(A,1.0)
(D,1.0)
(A,0.5)
(B,0.5)
(A,0.5)
(C,0.5)
：


(B,[(A,1.0)])
(A,[(D,1.0)])
(C,[(A,1.0), (B,1.0)])
(D,[(A,1.0), (C,1.0)])
程序结果：
(A,1.0)
(D,1.0)
(A,0.5)
(B,0.5)
(A,0.5)
(C,0.5)

(B,0.575)
(A,1.8499999999999999)
(C,0.575)
(D,1.0)
```

```java
package spark_example;

import avro.shaded.com.google.common.collect.Iterables;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.*;
import org.apache.spark.sql.types.Decimal;
import scala.Tuple2;

import java.io.Serializable;
import java.text.DecimalFormat;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Iterator;
import java.util.List;

public class PageRank_1 implements Serializable {
    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local[1]");
        conf.setAppName("SPARK ES");
        JavaSparkContext sc = new JavaSparkContext(conf);
//边的关系
        Tuple2[] links = {Tuple2.apply('A', 'D'), Tuple2.apply('B', 'A'),
                Tuple2.apply('C', 'A'), Tuple2.apply('C', 'B')
                , Tuple2.apply('D', 'A'), Tuple2.apply('D', 'C')};
        List<Tuple2> tuple2s = Arrays.asList(links);
       
        System.out.println(tuple2s);
        
    //权重    
        JavaRDD<Tuple2> linksRdd = sc.parallelize(tuple2s);

        Tuple2[] ranks = {Tuple2.apply('A', 1.0), Tuple2.apply('B', 1.0),
                Tuple2.apply('C', 1.0), Tuple2.apply('D', 1.0)};
    
        List<Tuple2> tuple2s1 = Arrays.asList(ranks);
        System.out.println(tuple2s1);
        
        
        JavaRDD<Tuple2> RanksRdd = sc.parallelize(tuple2s1);
        JavaPairRDD<Character, Character> linkPairRDD = linksRdd.mapToPair(s -> {
            return new Tuple2<Character, Character>((Character) s._1, (Character) s._2);
        });
        JavaPairRDD<Character, Double> rankPairRDD = RanksRdd.mapToPair(s -> {
            return new Tuple2<Character, Double>((Character) s._1, (Double) s._2);
        });
        linkPairRDD.foreach(x -> System.out.println(x));
        System.out.println();
        
        
        rankPairRDD.foreach(x -> System.out.println(x));
        
        //进行join构造边权值的关系
        JavaPairRDD<Character, Tuple2<Character, Double>> join = linkPairRDD.join(rankPairRDD);
        System.out.println();
        join.foreach(x -> System.out.println(x));
        
        //根据key进行分组，确定一个节点自己的邻居
        JavaPairRDD<Character, Iterable<Tuple2<Character, Double>>> characterIterableJavaPairRDD = join.groupByKey();
        characterIterableJavaPairRDD.foreach(x -> System.out.println(x));
        
        //根据values进行计算contribs；
        JavaRDD<Object> objectJavaRDD = characterIterableJavaPairRDD.values().flatMap(new FlatMapFunction<Iterable<Tuple2<Character, Double>>, Object>() {

            @Override
            public Iterator<Object> call(Iterable<Tuple2<Character, Double>> tuple2s) throws Exception {
                int count = Iterables.size(tuple2s);
                List results = new ArrayList<>();
                for (Tuple2<Character, Double> tuple2 : tuple2s) {
                   // System.out.println(tuple2._1+": "+tuple2._2 / count);
                    results.add(new Tuple2<>(tuple2._1, tuple2._2 / count));
                }
                return results.iterator();
            }
        });
        objectJavaRDD.foreach(x-> System.out.println(x));
        
        
        //最后一部根据ruduceBykey进行计算，但是flatmap之后只能进行reduce操作，因此需要进行mapToPair操作，转换成键值对的形式。也是在上一步直接使用flatmaptoPair函数一步到位。
        JavaPairRDD<String, Double> objectObjectJavaPairRDD = objectJavaRDD.mapToPair(
                s -> {
                    String[] split = s.toString().replaceAll("\\(","").replaceAll("\\)","")
                                    .split(",");
                    return new Tuple2<String,Double>(split[0], Double.parseDouble(split[1]));
                }
        );
        JavaPairRDD<String, Double> stringDoubleJavaPairRDD = objectObjectJavaPairRDD.reduceByKey(new Function2<Double, Double, Double>() {
            @Override
            public Double call(Double v1, Double v2) throws Exception {
                return v1 + v2;
            }
        }).mapValues(new Function<Double, Double>() {

            @Override
            public Double call(Double v1) throws Exception {
                return 0.15 + 0.85 * v1;
            }
        });
        stringDoubleJavaPairRDD.foreach(x-> System.out.println(x));
    }
}

```

输出结果

```
[(A,D), (B,A), (C,A), (C,B), (D,A), (D,C)]
[(A,1.0), (B,1.0), (C,1.0), (D,1.0)]

(A,D)
(B,A)
(C,A)
(C,B)
(D,A)
(D,C)

(A,1.0)
(B,1.0)
(C,1.0)
(D,1.0)

(B,(A,1.0))
(A,(D,1.0))
(C,(A,1.0))
(C,(B,1.0))
(D,(A,1.0))
(D,(C,1.0))

(B,[(A,1.0)])
(A,[(D,1.0)])
(C,[(A,1.0), (B,1.0)])
(D,[(A,1.0), (C,1.0)])

(A,1.0)
(D,1.0)
(A,0.5)
(B,0.5)
(A,0.5)
(C,0.5)

(B,0.575)
(A,1.8499999999999999)
(C,0.575)
(D,1.0)
```

缺点分析：

上述算法经历了join,flatMap,reduceBykey,mapValue等过程。

在spark内部会为每个操作都生成对应的RDD对象。整个PageRank算法的RDD依赖取决于迭代循环的次数的大小。每一次迭代中有两次Shuffle过程。

![image-20220706193125674](https://s2.loli.net/2022/07/07/TiByrnlVF2acEqG.png)

改良第二版

![image-20220706193102728](https://s2.loli.net/2022/07/07/ecEKGZjs8U7QoLl.png)

```java
package spark_example;

import avro.shaded.com.google.common.collect.Iterables;
import org.apache.spark.HashPartitioner;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.Function2;
import scala.Tuple2;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Iterator;
import java.util.List;

public class PageRank_2 {
    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        conf.setMaster("local[1]");
        conf.setAppName("SPARK ES");
        JavaSparkContext sc = new JavaSparkContext(conf);

        Tuple2[] links = {Tuple2.apply('A', 'D'), Tuple2.apply('B', 'A'),
                Tuple2.apply('C', 'A'), Tuple2.apply('C', 'B')
                , Tuple2.apply('D', 'A'), Tuple2.apply('D', 'C')};
        List<Tuple2> tuple2s = Arrays.asList(links);

        JavaRDD<Tuple2> linksRdd = sc.parallelize(tuple2s,2);

        Tuple2[] ranks = {Tuple2.apply('A', 1.0), Tuple2.apply('B', 1.0),
                Tuple2.apply('C', 1.0), Tuple2.apply('D', 1.0)};

        List<Tuple2> tuple2s1 = Arrays.asList(ranks);

        JavaRDD<Tuple2> RanksRdd = sc.parallelize(tuple2s1,2);


        JavaPairRDD<Character, Character> linkPairRDD = linksRdd.mapToPair(s -> {
            return new Tuple2<Character, Character>((Character) s._1, (Character) s._2);
        });
        JavaPairRDD<Character, Character> linksPartion = linkPairRDD.partitionBy(new HashPartitioner(2));
        JavaPairRDD<Character, Double> rankPairRDD = RanksRdd.mapToPair(s -> {
            return new Tuple2<Character, Double>((Character) s._1, (Double) s._2);
        });
//        linkPairRDD.foreach(x -> System.out.println(x));
//        System.out.println();
//        rankPairRDD.foreach(x -> System.out.println(x));
        linksPartion.foreach(x-> System.out.println(x));
        JavaPairRDD<Character, Tuple2<Character, Double>> join = linksPartion.join(rankPairRDD);
//        System.out.println();
        join.foreach(x -> System.out.println(x));
        JavaPairRDD<Character, Iterable<Tuple2<Character, Double>>> characterIterableJavaPairRDD = join.groupByKey();
        //characterIterableJavaPairRDD.foreach(x -> System.out.println(x));
        JavaRDD<Object> objectJavaRDD = characterIterableJavaPairRDD.values().flatMap(
                new FlatMapFunction<Iterable<Tuple2<Character, Double>>, Object>() {
                    @Override
                    public Iterator<Object> call(Iterable<Tuple2<Character, Double>> tuple2s) throws Exception {
                        int count = Iterables.size(tuple2s);
                        List results = new ArrayList<>();
                        for (Tuple2<Character, Double> tuple2 : tuple2s) {
                            // System.out.println(tuple2._1+": "+tuple2._2 / count);
                            results.add(new Tuple2<>(tuple2._1, tuple2._2 / count));
                        }
                        return results.iterator();
                    }
                }
        );
        objectJavaRDD.foreach(x-> System.out.println(x));
        JavaPairRDD<String, Double> objectObjectJavaPairRDD = objectJavaRDD.mapToPair(
                s -> {
                    String[] split = s.toString().replaceAll("\\(","").replaceAll("\\)","")
                            .split(",");
                    //System.out.println(split[0]+"::"+split[1]);
                    return new Tuple2<String,Double>(split[0], Double.parseDouble(split[1]));
                }
        );
        JavaPairRDD<String, Double> stringDoubleJavaPairRDD = objectObjectJavaPairRDD.reduceByKey(new Function2<Double, Double, Double>() {
            @Override
            public Double call(Double v1, Double v2) throws Exception {
                return v1 + v2;
            }
        }).mapValues(new Function<Double, Double>() {

            @Override
            public Double call(Double v1) throws Exception {
                return 0.15 + 0.85 * v1;
            }
        });
        stringDoubleJavaPairRDD.foreach(x-> System.out.println(x));
    }
}

```

```
第一个分区：
(B,A)
(D,A)
(D,C)
第二个分区：
(A,D)
(C,A)
(C,B)
###################
(B,(A,1.0))
(D,(A,1.0))
(D,(C,1.0))

2.
(A,(D,1.0))
(C,(A,1.0))
(C,(B,1.0))
###################
(A,1.0)
(A,0.5)
(C,0.5)
2.
(D,1.0)
(A,0.5)
(B,0.5)
###################
(B,0.575)
(D,1.0)
2.
(A,1.8499999999999999)
(C,0.575)

```



![微信图片_20220706211855](https://s2.loli.net/2022/07/07/jTsouw5SpQfLZHJ.jpg)

