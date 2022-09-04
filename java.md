# JDK选型、安装与配置

## JAVA抽象层
![image-20220119215300941](https://s2.loli.net/2022/01/19/W9XatgJvz5ujbiB.png)

### JVM（JAVA虚拟机）

JVM有很多组件，最开始用户的代码时通过BAD CODE写成，然后被CLASS LOADER加载，加载完之后就是JVM可以识别的内部数据结构。 
BAD CODE可以被执行，也定义了一些数据的类型。 
JAVA的执行引擎通过ATM接口和最底层的操作系统进行交互。

### JRE（JAVA的执行环境）
JRE和JVM几乎是一体的，但是JRE在组织上包括一些基础的类库，比如java.net可以保护网络，java.io可以保护文件，j.u.c可以帮助构建并发的应用程序，这也是JAVA流行的重要原因。
### JDK（开发工具包）
各种语言都有相应的开发工具包，JDK就是JAVA的开发工具包，里面包含了开发工具。如果需要开发JAVA程序，则需要开发包里面拥有JAVA的编译器。
## “正统”OpenJDK
Hotspot是OpenJDK里面默认的JAVA虚拟机实现。OpenJDK是由JSP这个组织去规划它的路线，进而实现它。在OpenJDK基础上加上Oracle特性就是可以在Oracle官网上下载下俩的Oracle JDK。
第三方厂商也会基于OpenJDK去构建自己的构造，比如自己的发行版，例如亚马逊的Corretto、Azul的Zulu,阿里巴巴也提供了JAVA发行版，在OpenJDK的基础上加上阿里巴巴云原生特性，形成了阿里的Dragonwell。

## JDK选型小结
1. 优先选择OpenJDK
2. Oracle不再免费提供最新的OpenJDK
3. AdoptOpenJDK下的Dragonwell是一个好的替代品。
# Java编程基础
## Java编程入门
### 初识java开发
定义第一个程序代码  
```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```
Java程序是需要经过两次处理后才可以执行的：
（1）对源代码程序进行编译：javac HelloWorld.java，会出现一个HelloWorld.class的字节码文件——利用JVM进行编译，编译出一套与平台无关的字节码文件*.class；
（2）在JVM上进行程序的解释执行：java HelloWorld——解释的即使字节码文件，字节码文件的后缀是不需要编写的。

为了更加方便的理解java程序的主要结构，下面针对第一个程序进行完整的解释：
在java程序开发之中的最基础的单元是类，所有的程序都必须封装在类中执行，类的基本定义语法如下：`[public] class 类名称 {}`  

在本程序中定义的类名称为 “HelloWorld”，而类的定义有两种形式：
`public class 类名称 {}`：类名称必须与文件名称保持一致，一个.java文件只能有一个public class;
`class 类名称 {}`: 类名称可以与文件名称不一致，但是编译后的.class名称class是定义的类名称，要求解析的是生成的.class名称文件；在一个.java文件里面可以有多个class定义，并且编译之后会形成不同的*.class文件。

主方法：主方法是所有的程序执行的起点，并且一定要定义在类之中  
```java
public class HelloWorld {
    public static void main(String[] args){
        //程序的代码由此开始执行
    }
}
```
java的主方法名称定义非常长，主方法所在的类都成为“主类”，所有的主类都将采用public class来定义。

屏幕打印（系统输出）可以直接在命令行方式下进行内容的显示，有两类语法形式:
（1）输出之后追加换行:System.out.println（输出内容）
（2）输出之后不追加换行:System.out.print（输出内容）、ln（line 换行）
### Java基本概念
#### 注释
注释是程序开发中的一项重要组成技术。合理的注释可以使项目维护更加方便，但是很多公司由于技术管理问题，注释不尽合理，使得项目维护非常不便，在人员更换时期非常痛苦，对后续工作造成了很大的困扰。特别是新人在项目维护的时候，发现只有若干行代码，没有一个注释，给新人带来了巨大的工作压力。所以注释是一个非常重要的概念。

Java中注释共有三类：
（1）单行注释：//，即双斜线之后到当前行结尾的内容被注释。
（2）多行注释：/*  ……/，则/ 和/之间的内容被注释掉，可以跨多行注释。
（3）文档注释：/** …/；（文档注释需要很多选项，一般建议通过开发工具控制，因为采取手写的方式较为麻烦。）

#### 标识符与关键字
在任何一个程序之中实际上都是一个结构的整合体，在Java语言里面有不同的结构，例如：类、方法、变量结构等，那么对于不同的结构一定要有不同的说明。那么对于结构的说明实际上就是标识符，是有命名要求的，但是一般都要求有意义的单词所组成的，同时对于标识符的组成在Java之中的定义如下：由字母、数字、下划线、$所组成，其中不能使用数字开头，不能使用Java中的保留字（关键字）。

最简单的定义形式：是用英文字母开头，例如Student_Name、StudentName，而且对于”＄“一般都有特殊的含义，不建议出现在你自己所编写的代码上。

关键字是系统对于一些结构的描述处理，有着特殊的含义。例如public、class等就有特殊含义，这些就是关键字，不可以把它作为标识符使用。

JDK 1.4的时候出现有assert 关键字，用于异常处理上;
JDK 1.5的时候出现有enum关键字，用于枚举定义上;
未使用到的关键字:goto、const; .
有一些属于特殊含义的单词，严格来讲不算是关键字:true、false、null。
### Java数据类型
#### Java数据类型简介
在Java语言之中对于数据类型一共分为两类:
- 基本数据类型：描述的是一些具体的数字单元，例如：1、1.1
    - 数值型：
        - 整形：byte、short、int、long；默认值：0
        - 浮点型：float、double；默认值：0.0
    - 布尔型：boolean；默认值：false
    - 字符型：char；默认值：‘\u0000’
- 引用数据类型：牵扯到内存关系的使用；
    - 数组、类、接口。默认值：null

而本次讨论的主要是基本数据类型，这里不牵扯到复杂的内存关系的匹配操作。每一种数据类型都有每一种类型保存的数据范围

![image-20220117123159277](https://s2.loli.net/2022/01/17/o8a2Wunm3hiG9pB.png)

不同的类型保存有不同范围的数据，但是这里面实际上就牵扯到了数据类型的选择上，这里给出一些使用原则：

- 如果要是描述数字首选的一定是int（整数）、double（小数）；
- 如果要进行数据传输或者是进行文字编辑转换使用byte类型（二进制处理操作）；
- 处理中文的时候最方便的操作使用的是字符char来完成（可选概念）；
- 描述内存或文件大小、描述表的主键列（自动增长）可以使用long；

#### 整型数据

整型数据一共有四种，按照保存的范围由小到大分别是：duet、short、int 、long ，在Java里面任何一个的整形常量（例：10 不会改变）其默认的类型都是int型（只要是整数就是int类型的数据）。

```java
public class JavaDemo {
    public static void main(String[] args) {
        //int 变量名称=常量（10是一个常量，整数类型为int）
        int x = 10;//定义了一个整型变量x
        x = 20;//改变了x的现有内容
        //int型变量*int型变量=int型变量
        System.out.println(x * x);
    }
}
```

10永远不会改变，但是x是一个变量，x的内容是可以发生改变的。

任何的数据类型都是有其可以保存的数据范围的（正常使用下很少会出现此范围的数据）

```java
public class JavaDemo {
    public static void main(String[] args) {
        int max = Integer.MAX_VALUE;//获取int的最大值
        int min = Integer.MIN_VALUE;//获取int的最小值
        System.out.println(max);//2147483647
        System.out.println(min);//-2147483648
        System.out.println("--------------无以言表的分割线------------------");
        //int型变量+int型常量=int型计算结果
        System.out.println(max + 1);//-2147483648
        System.out.println(max + 2);//-2147483647
        //int型变量+int型常量=int型计算结果
        System.out.println(min - 1);//2147483647
    }
}
```

通过此时的执行结果可以发现这些数字在进行处理得时候如果超过了其最大的保存范围，那么将出现有循环的问题，而这些问题在Java中被称之为数据溢出，如果想解决这种溢出，就可以继续扩大使用范围，比int更大的范围是long。

- 在操作的时候预估数据范围，如果返现范围不够就是用更大范围

```Java
public class JavaDemo {
    public static void main(String[] args) {
        //long long变量=int的数值
        long max = Integer.MAX_VALUE;//获取int的最大值
        long min = Integer.MIN_VALUE;//获取int的最小值
        System.out.println(max);//2147483647
        System.out.println(min);//-2147483648
        System.out.println("--------------无以言表的分割线------------------");
        //long型变量+int型常量=long型计算结果
        System.out.println(max + 1);//2147483648
        System.out.println(max + 2);//2147483649
        //long型变量+int型常量=long型计算结果
        System.out.println(min - 1);//-2147483649
    }
}
```

- 除了可以定义long型的变量之外，也可以直接在常量上进行处理，默认的整数常量都是int型，那么可以为他追加字母“L”或者直接使用“long”转换。

```Java
public class JavaDemo {
    public static void main(String[] args) {
        int max = Integer.MAX_VALUE;//获取int的最大值
        int min = Integer.MIN_VALUE;//获取int的最小值
        System.out.println(max);//2147483647
        System.out.println(min);//-2147483648
        System.out.println("--------------无以言表的分割线------------------");
        //int型变量+long型常量=long型计算结果
        System.out.println(max + 1L);//2147483648
        System.out.println(max + 2l);//2147483649
        //int型变量+long型常量=long型计算结果
        System.out.println((long) min - 1);//-2147483649
    }
}
```

现在发现数据类型之间是可以转换的，即：范围小的数据类型可以自动转换为范围大的数据类型，如果反过来范围大的数据类型要转为范围小的数据类型，那么就必须采用强制性的处理模式，同时还需要考虑可能带来的数据溢出。

```java
public class JavaDemo {
    public static void main(String[] args) {
        long num = 2147483649L;//此数据已经超过了int范围,故需要在后面加L
        //int temp = num;//long范围比int范围大，不能够直接转换,不兼容的类型: 从long转换到int可能会有损失
        int temp = (int) num;//-2147483647
        System.out.println(temp);
    }
}
```

程序支持有数据转换处理，但是如果不是必须的情况下不建议这种转换。

在进行整型处理的时候，还有一个byte类型特别需要注意，首先这个类型的范围很小：-128~127

```Java
public class JavaDemo {
    public static void main(String[] args) {
        byte num = 20;
        System.out.println(num);
    }
}
```

正常来讲在Java程序里面20这个数字应该是int型，但是在为byte赋值的时候并没有因为是int型而发生强制类型转换，这是因为Java对byte做了特殊处理，即：如果没超过byte范围的常量可以自动由int变成byte，如果超过了就必须强制转换。

```java
public class JavaDemo {
    public static void main(String[] args) {
        byte num = (byte)200;
        System.out.println(num);//-56
    }
}
```

由于现在200已经超过了byte的范围，所以产生了数据溢出的情况。

#### 浮点型数据

浮点型数据描述的是小数，而在java里面任意的一个小数常量其对应的类型为double，所以在以后描述小数的时候都建议直接使用double进行定义。

```java
public class JavaDemo {
    public static void main(String[] args) {
        //10.2是一个小数，其对应的类型为double
        double x = 10.2;
        int y = 10;
        //double * int = double 
        double result = x * y;
        System.out.println(result);
    }
```

所有的数据类型进行自动转型的时候都是由小类型向大类型进行自动转化处理。

默认类型为double，但是也可以定义位数相对较少的float变量，此时重新赋值的时候就必须要采用强制类型转换。

```java
public class JavaDemo {
    public static void main(String[] args) {
        //10.2是一个小数，其对应的类型为double
        float x = (float) 10.2;
        float y = 10.1F;
        System.out.println(x * y);//float
    }
}
```

通过一系列的代码分析发现，整形是不包含有小数点的，而浮点型是包含有小数点的。

```java
public class JavaDemo {
    public static void main(String[] args) {
        int x = 10;
        int y = 4;
        System.out.println(x / y);
    }
}
```

此时的计算结果为2，得到2的主要原因在于，整形是不保存小数点的，所以现在的计算结果为2.5 ，那么抛开小数点来看，最终的结果只是2。如果想得到所需要的正确的计算，那么就需要进行转型处理。计算的时候一定要注意选择的数据类型，它将会直接决定小数点的问题，这一点至关重要。

#### 字符型数据

字符型使用的是char进行定义的，在Java之中使用单引号定义的内容就是一个字符。例如定义一个字符型变量：

```java
public class JavaDemo {
    public static void main(String[] args) {
        char c = 'B';
        System.out.println(c);
    }
}
```

首先要明确在任何的编程语言之中，字符都可以与int进行互相转换，字符中所描述的内容可以通过int获取其内容对应的系统编码。最早的计算机搭造的只是010101，但是如果用01的数字（例如110 101 数字等）来描述，尽管简化了一些过程，但却很难理解的。

```java
public class JavaDemo {
    public static void main(String[] args) {
        char c = 'A';//一个字符变量
        int num = c;//可以获得字符的编码
        System.out.println(num);//65
    }
}
```

对于以上的程序获得了编码，这里面有几个范围需要注意：

- 大写字母：A（65）~Z（90）；
- 小写字母：a（97）~z（122）；

- 数字范围：0（48）~9（57）；

通过编码范围可以发现大小写字母之间差了32个数字的长度，于是就可以实现大小写的转换处理。

```java
public class JavaDemo {
    public static void main(String[] args) {//小写字母转换为大写字母
        char c = 'a';//一个字符变量
        int num = c;//可以获得字符的编码
        num=num-32;
        System.out.println((char)num);
    }
}
```

到此为止，所有的操作都与传统的c语言的方式是一样的，但是需要注意的是在java里面char主要是进行中文的处理，所以java中的char类型可以保存中文数据。

```java
public class JavaDemo {
    public static void main(String[] args) {
        char c = '我';//一个字符变量
        int num = c;//可以获得字符的编码
        System.out.println(num);
    }
}
```

之所以在java语言里面可以使用char进行中文数据的保存，是因为java使用的是unicode这种十六进制的编码,这种编码的主要特点是可以包括任意的文字内容，所以使得程序开发更加的简单。

#### 布尔数据

布尔是一位数学家的名字，布尔主要描述的是一种逻辑的处理结果，在java中使用boolean来进行布尔类型变量的定义，需要注意的是，布尔类型的取值范围只有两个数据：true、false

```java
public class JavaDemo {
    public static void main(String[] args) {
        boolean flag = true;
        if (flag) {//判断flag的内容，如果是true就执行
            System.out.println("我很帅！");
        }
    }
}
```

但是需要说明，一些编程语言没有提供布尔类型，所以会使用0表示false，使用非0表示true，这样的逻辑在java中不存在。

#### String字符串

在任何语言里面都没有提供所谓的字符串这种数据类型，但是从实际的使用上来讲各个编程语言为了方便程序的开发，也都会提供字符串的相应描述，在Java里面使用是是String作为字符串的定义。

由于String类的存在较为特殊，所以其可以像普通变量那样采用直接赋值的方式进行字符串的定义，并且要求使用双引号进行字符串描述。

```java
public class JavaDemo {
    public static void main(String[] args) {
        String str = "Hello World !";
        System.out.println(str);
    }
}
```

在进行字符串变量使用的时候也可以使用“+”来进行字符串的连接处理。

```java
public class JavaDemo {
    public static void main(String[] args) {
        String str = "Hello ";
        str = str + "World";
        str += "!!!";
        System.out.println(str);
    }
}
```

但是需要考虑另外一点，此时对于“+”就有了两种描述：字符串的连接、数字的加法计算。那么下面来观察一个程序。

```java
public class JavaDemo {
    public static void main(String[] args) {
        double x = 10.1;
        int y = 20;
        String str = "计算结果" + x + y;
        System.out.println(str);
    }
}
```

在Java语言里面，数据范围大的数据类型与数据范围小的数据类型进行计算的时候，所有范围小的数据类型自动转型为范围大的数据类型，但是如果此时有String字符串了，则所有的类型无条件先变为String。

在描述字符串的时候也可以使用转义字符进行一些处理，\后面加相应内容，TAB（t)、换行（n）

```java
public class JavaDemo {
    public static void main(String[] args) {
        System.out.println("\tHello World!!!\nHello \"MLDN\"");
    }
}
```

这些字符是可以在学习的过程之中进行一些简单的格式化显示处。

### java运算符

#### 运算符简介

概念：所有的程序开发都是一种数字的处理游戏，那么对于数字的处理，一定会有所谓的操作，而这些操作模式就称为运算。

例如：如果要进行加法运算肯定使用的“+”这样的运算符来完成，而对于运算符而言，也是存在有先后的关系，像小学学习四则运算，采用先乘除后加减的顺序来完成。

首先对于程序开发而言，里面会提供有大量的基础运算符，那么这些运算符也都会提供有各自的优先顺序。

![image-20220119203000600](https://s2.loli.net/2022/01/19/7RAbQ6wU93jTVpY.png)

关键性的问题是，对于程序的开发而言，不建议编写很复杂的计算。

如果项目代码里面按照这样的逻辑编写了代码，基本上没有人看懂，也容易出错。所以对于程序代码而言，实际上已经告别了复杂程序逻辑时代，更多情况下是希望大家去编写一些简单易懂的代码。

#### 数学运算符

在Java数学运算提供了标准的支持，包括四则运算都是支持的。

在进行变量计算的时候，编程语言一般也都会有简化的运算符（+=、*=、−=、÷=、%=）支持。

```java
public class JavaDemo{
    public static void main(String[] args){
        int num = 10;
        num %= 3;
        System.out.println(num);
	}
}
```

在数学计算里面最头疼是“++”、“−−”，因为这两种运算符有两类使用方式：

- ++变量、−−变量：先进行变量的自增或者自减，而后再进行数字的计算；

- 变量++、变量−−：先使用变量进行计算，而后再进行自增或自减。

```java
public class JavaDemo {
    public static void main(String[] args) {
        int x = 10;
        int y = 20;
        //首先x自增，然后进行计算，最后y进行自减
        int result = ++x - y--;
        System.out.println(result);
        System.out.println(x);
        System.out.println(y);
    }
}
```

#### 关系运算符

**关系运算的主要特征：进行大小的比较处理。**

包括：（>）、小于（<）、大于等于（>=）、小于等于（<=）、不等（！=）、相等（==）。所有的关系运算返回的判断结果都是布尔类型的数据；

```java
public class JavaDemo {
    public static void main(String[] args) {
        int x = 10;
        int y = 20;
        boolean flag = x > y;//false
        System.out.println(flag);
    }
}
```

---

进行关系判断的时候特别需要注意的就是相等的判断问题。在Java里面“=”表示的是赋值运算，而内容相同的比较是“==”。

---

进行关系运算的时候可以针对于所有的基本数据类型，例如：也可以直接使用字符来处理。

```java
public class JavaDemo {
    public static void main(String[] args) {
        char c = '建';
        boolean flag = 24314 == c ;  //true
        System.out.println(flag) ;
    }
}
```

#### 三目（逻辑）运算符

进行程序开发的时候三目运算符使用的非常多，而且合理的利用三目运算可以避免一些大范围的程序编写。三目是一种所谓的赋值运算处理。它是需要设置一个逻辑关系的判断之后才可以进行的赋值操作，基本语法如下：

关系运算？关系满足时的内容：关系不满足时的内容

```java
public class JavaDemo {
    public static void main(String[] args) {//判断两个数字的大小，将最大值保存。
        int x = 10;
        int y = 20;
        //判断x与y的大小关系来决定最终max变量内容
        int max = x > y ? x : y ;
        System.out.println(max) ;
    }
}
```

虽然允许进行嵌套处理，但是程序的可读性变的很差，根据实际的情况确定是否使用。

```java
public class JavaDemo {
    public static void main(String[] args) {
        int x = 10;
        int y = 20;
        int z = 15;
        int max = x > y ? (x > z ? x : z) : (y > z ? y : z);
        System.out.println(max);
    }
}
```

#### 位运算符

位运算指的是可以直接进行二进制的计算处理，主要有：与（&）、或（|）、异或（^）、反码（~）、移位处理。

如果要想理解操作，则一定要清楚十进制与二进制之间的转换处理逻辑：数字除2取余。

```java
public class JavaDemo {
    public static void main(String[] args) {
        int x = 13;
        int y = 7;
        System.out.println(x & y);//5
    }
}
```

```java
public class JavaDemo {
    public static void main(String[] args) {
        int x = 2;
        System.out.println(x << 2);//8
    }
}
```

### Java程序逻辑控制

在程序开发的过程之中一共会存在有3种程序逻辑：顺序结构、分支结构、循环结构

#### if分支结构

 If 分支结构主要是针对关系表达式进行判断处理的分支操作。对于分支语句主要有三类使用形式，使用的关键字有：if、else。

```java
public class JavaDemo {
    public static void main(String[] args) {
        Double score = 90.00;
        if (score >= 90.00 && score <= 100) {
            System.out.println("优等生");
        } else if (score >= 60 && score < 90) {
            System.out.println("良等生");
        } else {
            System.out.println("差等生");

        }
    }
}
```

#### switch开关语句

swith 是一个开关语句，它主要是根据内容来进行的判断，需要注意的是swith中可以判断的只能够是数据（int、char、枚举、string）而不能够使用逻辑判断。

```java
public class JavaDemo {
    public static void main(String[] args) {
        int ch = 2;
        switch (ch) {
            case 2 -> {
                System.out.println("设置的内容是2");
                break;
            }
            case 1 -> {
                System.out.println("设置的内容是1");
                break;
            }
            default -> {
                throw new IllegalStateException("Unexpected value: " + ch);
            }
        }
    }
}
```

#### while循环

所谓的循环结构,指的是某代码被重复执行的处理操作。在程序之中提供有while语句来实现循环的定义,那么该语句有两种定义形式。

```java
public class JavaDemo {
    public static void main(String[] args) {
        int sum = 0;
        int num = 1;
        while (num <= 100) {
            sum += num;
            num++;
        }
        System.out.println(sum);
    }
}
```

在编写程序时，除了使用while循环之外，也可以使用do...while循环来进行处理。while循环与do...while循环的最大差别是,while循环是先判断后执行，而do...while循环是先执行一次后再进行判断。

```java
public class JavaDemo {
    public static void main(String[] args) {
        int sum = 0;
        int num = 1;
        do {
            sum += num;
            num++;
        } while (num <= 100);
        System.out.println(sum);
    }
}
```

#### for循环

除了while ，do while ,for循环也是一种常规的循环结构。

```java
public class JavaDemo {
    public static void main(String[] args) {
        int sum = 0;
        for (int i = 0; i <= 100; i++) {
            sum += i;
        }
        System.out.println(sum);
    }
}
```

对于while和for循环的选择参考标准

- 在明确确定循环次数的情况下优先选择for循环。
- 在不知道循环次数，但是知道循环结束条件的情况下使用while循环。

#### 循环控制

在循环语句定义的时候还有两个控制语句:break、continue。

break主要的功能是退出整个循环结构。

```java
public class JavaDemo {
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            if (i == 3) {
                break;
            }
            System.out.print(i + "、");
        }
    }
}
```

continue严格来讲只是结束当前的一次调用(结束当前循环)。

```java
public class JavaDemo {
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            if (i == 3) {
                continue;
            }
            System.out.print(i + "、");
        }
    }
}
```

在C语言里面有一个goto的指令，在Java中可以利用continue实现部分goto的功能。这种代码是Java支持的，但对于此类代码强烈不建议开发者在代码中出现。

```java
public class JavaDemo {
    public static void main(String[] args) {
        point:
        for (int i = 0; i < 10; i++) {
            for (int j = 0; j < 3; j++) {
                if (i == j) {
                    continue point;
                }
                System.out.print(i + "、");
            }
            System.out.println();
        }
    }
}
```

#### 循环嵌套

一个循环语句之中嵌套其它的循环语句就称为循环嵌套处理，循环嵌套层次越多时间复杂度就越高。

```java
public class JavaDemo {
    public static void main(String[] args) {//打印乘法表
        for (int i = 1; i < 10; i++) {
            for (int j = 1; j <= i; j++) {
                System.out.print(j + "*" + i + "=" + (i * j) + "\t");
            }
            System.out.println();
        }
    }
}
```

```java
public class JavaDemo {
    public static void main(String[] args) {
        int line =5;
        for (int i = 0; i < line; i++) {
            for (int j = 0; j < line-i; j++) {
                System.out.print(" ");
            }
            for (int j = 0; j <= i; j++) {
                System.out.print("* ");
            }
            System.out.println();
        }
    }
}
```

### 方法的定义及使用

在程序之中很多情况下是有可能需要重复执行一些代码的。

#### 方法的定义

本次方法是定义在主类之中并且由主方法直接调用的，所以方法的定义语法形式如下：

```java
public static 返回值类型 方法名称([参数类型 变量，参数类型 变量，......]){
	//
	return 返回值；
}
```

对于返回值而言就可以使用Java中数据定义类型。（包含基本数据类型，引用数据类型）。 return所返回的数据类型与方法的返回值类型相同，如果不返回数据，则该方法上可以使用void进行说明。

在进行方法名称定义的时候要求第一个单词的字母小写，而后每个单词的首字母大写。变量名称和方法名称是一样的规则。

```java
public class JavaDemo {
    public static void main(String[] args) {
        printMessage();
        printMessage();
    }

    public static void printMessage() {
        System.out.println("这是无参数无返回值方法");
    }
}
```

 方法的本质就是方便使用者进行重复的调用，并且所有的程序一定都是通过主方法执行。

#### 方法重载

当方法名称相同，参数的类型或个数不同的时候就称为方法重载。通过程序做简单的分析，要定义一个加法的处理方法，该方法可以接收两个int变量、三个int变量、两个double变量的加法处理。

```java
public class JavaDemo {
    public static void main(String[] args) {
        int resultA = sum(10, 20);
        int resultB = sum(10, 20, 30);
        double resultC = sum(10.2, 20.3);
        System.out.println(resultA);
        System.out.println(resultB);
        System.out.println(resultC);
    }

    public static int sum(int x, int y) {
        return x + y;
    }

    public static int sum(int x, int y, int z) {
        return x + y + z;
    }

    public static double sum(double x, double y) {
        return x + y;
    }
}
```

## Java面向对象编程

### 类与对象

#### 面向对象简介

Java语言最大的特点在于面向对象的编程设计，并且面向对象的编程设计也在由于 Java自身的发展而不断发展。同时很多当初不支持面向对象的编程开始转向的面向对象，但是依然有许多的开发者认为面向过程比面向对象更好。

最早流行的编程语言C、C++、Java。其中C语言已经成为了面向过程的代表，C++，Java是面向对象的编程语言。

所谓的面向过程指的是面对于一个问题的结果方案，更多的情况下是不会做出重用的设计思考的，而面向对象的设计形式为模块化设计，并且可以进行重用配置，在整个的面向对象的设计里面更多情况下考虑的是标准，在使用的时候根据标准进行拼装。

面向对象设计有三个主要特征：

- 封装性：内部的操作对外部不可见，当内部的操作都不可直接使用的时候才是安全的
- 继承性：在已有结构的基础上继续进行功能的扩充
- 多态性：是在继承性的基础上扩充而来的概念，指的是类型的转换处理

在进行面向对象程序的开发之中一般还有三个步骤：

- OOA：面向对象分析
- OOD：面向对象设计
- OOP：面向对象编程

#### 类与对象简介

类是对某一类事物的共性的抽象概念，而对象描述的是一个具体的物体。

类是一个模板，而对象才是类可以使用的实例，先有类再有对象。

在类之中一般都会有两个组成：

- 成员属性（Field）：有些时候为了简化称其为属性；
  - 一个人的年龄、姓名等；
- 操作方法（Method）：定义对象具有的处理行为；
  - 这个人可以唱歌、跳舞、游泳等；

#### 类与对象的定义及使用

在Java之中类是一个独立的结构体，所以需要使用class来进行定义。

```java
class person{
    String name;
    int age;
    public void tell(){
        System.out.println("姓名："+name+"、年龄："+age);
    }
}
```

现在有了类之后，如果想要使用类就必须通过对象来完成，产生对象的方法如下：

- 声明并实例化对象：类名称 对象名称=new 类名称（）；
- 分步骤完成：
  - 声明对象：类名称 对象名称=null；
  - 实例化对象：对象名称=new 类名称（）

获取了实例化对象之后，需要通过对象进行类中的操作调用，此时有两种调用方式：

- 调用类中的属性：实例化对象.成员属性；
- 调用类中的方法：实例化对象.方法名称()；

```java
public class JavaDemo {
    public static void main(String[] args) {
        person per = new person();
        per.name = "张三";
        per.age = 18;
        per.tell();
    }
}
```

如果没有对对象属性进行设置，则对象的该属性为默认值。

#### 对象内存分析

Java之中类属于引用数据类型，引用数据类型使用的最大困难指出在于要进行内存的管理。

```java
public class JavaDemo {
    public static void main(String[] args) {
        person per = new person();
        per.name = "张三";
        per.age = 18;
        per.tell();
    }
}
```

如果要进行内存分析，首先给出两块最为常用的内存空间：

- 堆内存：保存的是对象的具体信息，在程序之中堆内存空间的开辟是通过new完成的。
- 栈内存：保存的是一块堆内存的地址，

![image-20220315203219856](https://s2.loli.net/2022/03/15/LWRmKceShuzVdw2.png)

如果只是声明了对象，但是没有为对象进行实例化，如果此时调用其成员属性或方法，则会抛出错误NullPointerException（空指向异常），就是没有开辟堆内存产生的问题，而且只有引用数据类型存在此类问题。

#### 对象引用分析

类本身属于引用数据类型，那么就牵扯到内存的引用传递，即同一块堆内存空间可以被不同的栈内存所指向，也可以更换指向。

```java
public class JavaDemo {
    public static void main(String[] args) {
        person per1 = new person();
        per1.name = "张三";
        per1.age = 18;
        person per2 = per1;//引用传递
        per2.age = 80;
        per1.tell();
    }
}
```

这个时候的引用传递是在主方法之中定义的，也可以通过方法实现引用传递处理。

```java
public class JavaDemo {
    public static void main(String[] args) {
        person per1 = new person();
        per1.name = "张三";
        per1.age = 18;
        change(per1);
        per1.tell();
    }

    public static void change(person temp) {
        temp.age = 80;
    }
}
```

引用传递可以发生在方法上，这个时候一定要观察方法的参数类型和方法的执行过程。

#### 引用与垃圾产生分析

引用传递处理不当会产生垃圾，现作简单分析。

```java
public class JavaDemo {
    public static void main(String[] args) {
        person per1 = new person();
        person per2 = new person();
        per1.name = "张三";
        per1.age = 18;
        per2.name = "李四";
        per2.age = 19;
        per2 = per1;//引用传递
        per2.age = 80;
        per1.tell();
    }
}
```

上述程序产生了没有任何栈内存指向的堆内存空间，即垃圾，所有的垃圾都将被GC(Garbage Collector)不定期回收，释放空间，但是如果垃圾过多，将会影响到GC的处理性能。

#### 成员属性封装

在类中，一般而言方法都是对外提供服务的，所以不需要封装处理，但是属性需要较高的安全性，故需要采用封装的方法对属性进行保护。

在默认的情况下，类属性是可以另一个类通过对象进行访问的，可以利用private关键字对属性进行封装，封装之后外部不可以直接访问类属性，即：对外部不可见，对内部可见。外部访问封装的属性方式如下：

- 【setter、getter】设置或取得属性可以使用setXxx()、getXxx()方法，以：private String name为例
  - 设置属性方法：public void setName(String n)
  - 取得属性方法：public String getName() 

```java
class person {
    private String name;
    private int age;

    public void tell() {
        System.out.println("姓名：" + name + "、年龄：" + age);
    }

    public void setName(String n) {
        name = n;
    }

    public void setAge(int a) {
        age = a;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        person per1 = new person();
        per1.setName("张三");
        per1.setAge(18);
        per1.tell();
    }
}
```

在开发中，类中所有的属性都必须使用private进行封装，并且提供有setter、getter方法。

#### 构造方法与匿名对象

为了简化编程，可以通过构造方法实现实例化对象中的属性初始化处理，构造方法要求如下：

- 构造方法名称必须与类名保持一致；
- 构造方法不允许设置任何的返回值类型，即没有返回值定义；
- 构造方法实在使用关键字new实例化对象的时候自动调用的。

```java
class person {
    private String name;
    private int age;

    public person(String n, int a) {//定义有参构造
        name = n;
        age = a;
    }

    public void tell() {
        System.out.println("姓名：" + name + "、年龄：" + age);
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        person per1 = new person("张三", 18);
        per1.tell();
    }
}
```

如果没有定义构造方法，则java默认提供一个无参的构造方法，这个构造方法实在编译时自动创建的。

构造方法重载只需要考虑参数的类型和数量。

只通过实例化对象来进行类的操作也是可以的，这种形式的对象没有名字，称为匿名对象。

```java
public class JavaDemo {
    public static void main(String[] args) {
        new person("张三", 18).tell();
    }
}
```

此时依然通过对象进行了类中tell()方法的调用，但是由于此对象没有任何的引用名称，所以该对象使用一次后就成为垃圾，等待被GC回收和释放。

### this关键字

使用this可以实现三类结构的描述：

- 当前类中的属性：this.属性
- 当前类中的方法（构造方法、普通方法）：this()、this.方法名称()
- 描述当前对象

#### this调用本类属性

访问本类中的属性，必须加上this

```java
class person {
    private String name;
    private int age;

    public person(String name, int age) {//定义有参构造
        this.name = name;
        this.age = age;
    }
}
```

#### this调用本类方法

- 构造方法调用：this()
- 普通方法调用：this.方法名称()

```java
class person {
    private String name;
    private int age;

    public person(){
        System.out.println("可以被重用的代码");
    }
    
    public person(String name, int age) {//定义有参构造
        this();
        this.setAge(age);
        this.setName(name);
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return this.age;
    }

    public String getName() {
        return this.name;
    }
}
```

构造方法必须在实例化新对象的时候调用，所以this()的语句只允许被放在构造方法的首行。

### static关键字

static关键字主要用来定义属性和方法。

#### 声明static属性

```java
class person {//定义属于同一个国家的类
    private String name;
    private int age;
    static String country = "中华民国";

    public person(String name, int age) {//定义有参构造、
        this.setAge(age);
        this.setName(name);
    }


    public void tell() {
        System.out.println("姓名：" + this.name + "、年龄：" + this.age+"、国家："+country);
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        person per1 = new person("张三", 18);
        person per2 = new person("lisi", 17);
        person.country ="中华人民共和国";
        per1.tell();
        per2.tell();
    }
}
```

static属性虽然定义在类之中，但是其并不受到类实例化对象的控制，即，static属性可以在没有类实例化对象的情况下使用。

#### 声明static方法

static方法可以直接由类名称在没有实例化对象的情况下进行使用。

```java
public static void setCountry(String c){
        country=c;
    }

person.setCountry("中华人民共和国");
```

- static方法只允许调用static属性和static方法
- 非static方法允许调用static属性和static方法

static定义的方法或者属性不是代码编写之初应该考虑的问题，只有在回避实例化对象调用并且描述公共属性的情况下才会使用static定义方法或者属性。

### 代码块

在程序之中使用{}定义的结构就称为代码块，根据代码块出现的位置以及定义的关键字的不同，代码块可以分为：普通代码块，构造代码块，静态代码块，同步代码块，其中同步代码块在多线程部分进行介绍。

#### 普通代码块

定义在一个方法之中的代码块，可以在一个方法之中进行一些结构的拆分，以防止相同变量名称所带来的相互影响。

```java
public class JavaDemo {
    public static void main(String[] args) {
        {
            int x = 10;
            System.out.println(x);
        }
        int x = 100;
        System.out.println(x);
    }
}
```

#### 构造代码块

定义在一个类之中的。

构造块会优先于构造方法执行，并且每一次实例化对象时都会调用构造块中的代码。

```java
class person{
    public person(){
        System.out.println("构造方法执行");
    }
    {
        System.out.println("构造块");
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        new person();
        new person();
    }
}
```

#### 静态代码块

主要指的是使用static关键字定义的代码块，静态代码块的定义需要考虑到两种情况：

- 非主类中定义静态块

```java
class person{
    public person(){
        System.out.println("构造方法执行");
    }
    static {
        System.out.println("静态块执行");
    }
    {
        System.out.println("构造块");
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        new person();
        new person();
    }
}
```

静态代码块会优先于构造块执行，且只执行一次，其主要目的是初始化类中的静态属性

```java
class message {
    public static String getCountry() {
        return "中华人民共和国";
    }
}

class person {
    private static String country;

    static {
        country = message.getCountry();
        System.out.println(country);
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        new person();
        new person();
    }
}
```

- 主类中定义静态块

```java
public class JavaDemo {
    static {
        System.out.println("程序初始化");
    }
    public static void main(String[] args) {
        System.out.println("主方法执行");
    }
}
```

静态代码块优先于主方法先执行。

### 数组的定义与使用

#### 数组的基本定义

在java里面将数组定义为了引用数据类型，所以数组的使用牵扯到内存分配，可以想到使用new关键字进行处理，数组的定义格式为：

- 数组的动态初始化，初始化之后数组每一个元素保存的内容为其数据类型的默认值
  - 数组类型 数组名称[]=new 数据类型[长度]
  - 数组类型[] 数组名称=new 数据类型[长度]
- 数组的静态初始化，在数组定义的时候就为其设置好了里面的内容
  - 简化格式：数据类型[] 数组名称={数据1，数据2，数据3，......}
  - 完整格式：数据类型[] 数组名称=new 数据类型[]{数据1，数据2，数据3，......}

数组定义完成后可按照如下方法进行使用：

- 通过角标进行元素访问，角标从0开始，角标超出限制会报错ArrayIndexOutOfBoundsException
- 数组长度：数组名称.length

```java
public class ArrayDemo {
    public static void main(String[] args) {
        int[] data = new int[3];
        System.out.println(data[0]);
        System.out.println(data[1]);
        System.out.println(data[2]);
        int[] num = new int[]{11, 23, 45};
        System.out.println(num[0]);
        System.out.println(num[1]);
        System.out.println(num[2]);
    }
}
```

#### 数组引用传递分析

```java
public class ArrayDemo {
    public static void main(String[] args) {
        int[] num = new int[]{11, 23, 45};
        System.out.println(num[0]);
        System.out.println(num[1]);
        System.out.println(num[2]);
        int[] temp = num;
        temp[0] = 99;
        for (int i = 0; i < temp.length; i++) {
            System.out.println(temp[i]);
        }
    }
}
```

#### foreach输出

可以将数组中的每一个元素的内容取出保存在变量里面，这样就可以通过变量访问数组元素，无需设置下表，防止数组越界。

```java
public class ArrayDemo {
    public static void main(String[] args) {
        int[] num = new int[]{11, 23, 45};
        for (int temp : num) {
            System.out.println(temp);
        }
    }
}
```

#### 二维数组

二维数组使用的定义语法：

- 数组的动态初始化：
  - 数据类型 [] [] 数组名称=new 数据类型[行个数] [列个数]；
- 数组的静态初始化：
  - 数据类型 [] [] 数组名称=new 数据类型[] []{{数据1，数据2，数据3，......}，{数据1，数据2，数据3，......}，{数据1，数据2，数据3，......}，......}；

```java
public class ArrayDemo {
    public static void main(String[] args) {
        int[][] data = new int[][]{{1, 2, 3}, {4, 3}, {6}};
        for (int i = 0; i < data.length; i++) {
            for (int j = 0; j < data[i].length; j++) {
                System.out.print(data[i][j]+" ");
            }
            System.out.println();
        }
    }
}
```

#### 数组与方法

数组可以通过方法实现引用传递。

```java
public class ArrayDemo {
    public static void main(String[] args) {
        int[][] data = initArray();
        printArray(data);
    }

    public static int[][] initArray() {
        return new int[][]{{1, 2, 3}, {4, 3}, {6}};
    }

    public static void printArray(int[][] data) {
        for (int i = 0; i < data.length; i++) {
            for (int j = 0; j < data[i].length; j++) {
                System.out.print(data[i][j] + " ");
            }
            System.out.println();
        }
    }
}
```

- 数组排序：java.util.Array.sort(数组名称)
- 数组拷贝：System.arraycopy(源数组，源数组开始点，目标数组，目标数组开始点，拷贝长度)

#### 方法可变参数

```java
class ArrayUtil {
    public static int sum(int... data) {
        int sum = 0;
        for (int i = 0; i < data.length; i++) {
            sum += data[i];
        }
        return sum;
    }
}

public class ArrayDemo {
    public static void main(String[] args) {
        System.out.println(ArrayUtil.sum(1, 2, 3));
        System.out.println(ArrayUtil.sum(new int[]{1, 2, 3}));
    }
}
```

#### 对象数组

```java
class Person {
    private String name;
    private int age;

    public Person(String name, int age) {//定义有参构造
        this.name = name;
        this.age = age;
    }

    public void tell() {
        System.out.println("姓名：" + name + "、年龄：" + age);
    }
}

public class ArrayDemo {
    public static void main(String[] args) {
        Person[] per = new Person[]{
                new Person("张三", 20),
                new Person("李四", 20),
                new Person("王五", 20)
        };
        for (int i = 0; i < per.length; i++) {
            per[i].tell();
        }
    }
}
```

### String类

#### String类简介

String严格意义上说不能算是一种基本数据类型，但是由于JVM的支持，我们可以像使用基本数据类型一样使用String，可以进行直接赋值。

String类中定义了一个数组来进行操作。

```java
@Stable
private final byte[] value;
```

#### 字符串比较

```java
public class StringDemo {
    public static void main(String[] args) {
        String strA = "test";
        String strB = new String("test");
        System.out.println(strA == strB);//false
        System.out.println(strA.equals(strB));//true
    }
}
```

“==”比较的是strA和strB对应的栈内存内容，equals()比较的是strA和strB堆内存保存的内容。

#### 字符串常量

严格来说程序中不存在字符串常量，字符串常量其实是String类的匿名对象。

```java
public class StringDemo {
    public static void main(String[] args) {
        String strA = "test";
        String strB = new String("test");
        System.out.println("test".equals(strB));//"test"为匿名对象
    }
}
```

#### String类常用方法

https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html

### 继承的定义与使用

在已有类的功能上继续进行扩充，由此可以实现代码的重用。

#### 继承的实现

依靠extends关键字来实现继承：

```java
class 子类 extends 父类 {}
```

很多情况下会将子类称为派生类，把父类称为超类（SuperClass）

```java
class student extends person {

    private String school;

    public String getSchool() {
        return school;
    }

    public void setSchool(String school) {
        this.school = school;
    }
}
```

#### 子类对象实例化流程

子类对象实例化的时候默认先进行父类对象实例化

```java
class person {
   public person() {
       System.out.println("父类对象实例化");
    }
}

class student extends person {

    public student(){
        super();//
        System.out.println("子类对象实例化");
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        new student();
    }
}
```

super()表示子类构造父类构造的语句，且只调用父类中的无参构造方法。如果父类中没有定义无参构造方法，则必须利用super()明确调用有参构造

```java
class student extends person {
    private String school;

    public student(String name, int age, String school) {
        super(name, age);
        this.school = school;
    }
}
```

#### 继承的相关限制

- java之中不允许多重继承，只允许多层继承。
- 在进行继承关系定义的时候，实际上子类可以继承父类中的所有操作结构。但是对于私有操作属于隐式继承，而所有的非私有操作属于显示继承。

```java
class person {
    private String name;

    public void setName(String n) {
        name = n;
    }

    public String getName() {
        return name;
    }
}

class student extends person {
    public student(String name) {
        setName(name);
    }

    public void fun() {
        //System.out.println(name);//无法直接访问name属性
        System.out.println(getName());
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        student stu = new student("lindaqina");
        stu.fun();
    }
}
```

### 覆写

子类与父类产生继承关系之后，如果发现父类中设计不足并且需要保留父类中的方法或者属性名称的情况下就会发生覆写。

#### 方法覆写

当子类定义了与父类名称相同，参数类型以及个数相同的方法的时候，就称为方法的覆写。

```java
class channel{
    public void connect(){
        System.out.println("父类进行资源链接");
    }
}
class databaseChannel extends channel{
    @Override
    public void connect(){
        super.connect();
        System.out.println("子类进行数据库资源链接");
    }
}
public class JavaDemo {
    public static void main(String[] args) {
        databaseChannel channel=new databaseChannel();
        channel.connect();
    }
}
```

子类调用父类中方法的时候要使用super

#### 方法覆写限制

被覆写的方法不能够拥有比父类方法更为严格的访问控制权限

访问控制权限：public>default>private

如果父类中的方法使用了default权限定义，那么子类定义该方法的时候只能够使用public或者default定义；如果父类中的方法使用了public定义，那么子类中的方法只能够使用public定义。

```java
class channel{
    public void connect(){
        System.out.println("父类进行资源链接");
    }
    public void fun(){
        this.connect();
    }
}
class databaseChannel extends channel{
    @Override
    public void connect(){
        //super.connect();
        System.out.println("子类进行数据库资源链接");
    }
}
public class JavaDemo {
    public static void main(String[] args) {
        databaseChannel channel=new databaseChannel();
        channel.fun();
    }
}
```

父类中connect()为private时，他对子类不可见，子类中的connect()并不是对其重写。

#### 属性覆盖

当子类定义了与父类名称相同的属性的时候称为属性覆盖。

#### final关键字

final在java之中描述的是一种终接器的概念，其功能为：定义不能被继承的类，不能被覆写的方法和属性。

```java
final class test{}	
public final void fun(){}
public static final int on=1;//全局常量
```

#### Annotation注解

Annonation是以一种注解的形式实现的程序开发。下面是几个基本注解：

- @Override：覆写
- @Deprecated：过期操作
- @SuppressWarnings：压制警告

### 多态性

多态性是面向对象的第三大主要特征，多态性是在继承性的基础上扩展出来的概念，也就是说可以实现父子类之间的相互转换处理。

#### 多态性简介

实现模式：

- 方法的多态性
  - 方法的重载：实现不同功能的执行
  - 方法的覆写：同一个方法根据子类的不同会有不同的实现
- 对象的多态性：父子实例之间的转换处理
  - 对象向上转型：父类 父类实例=子类实例，自动完成转换
  - 对象向下转型：子类 子类实例=（子类）父类实例，强制完成转换

从实际情况来讲，大部分考虑的都是对象的向上转型

#### 对象向上转型

```java
class channel{
    public void connect(){
        System.out.println("父类进行资源链接");
    }
}
class databaseChannel extends channel{
    @Override
    public void connect(){
        //super.connect();
        System.out.println("子类进行数据库资源链接");
    }
}
class webServer extends channel{
    @Override
    public void connect(){
        System.out.println("webServer资源链接");
    }
}
public class JavaDemo {
    public static void main(String[] args) {
        fun(new databaseChannel());
        fun(new webServer());
    }
    public static void fun(channel channel){
        channel.connect();
    }
}
```

对象向上转型方便与进行参数统一设计。虽然野可以通过方法重载的方法实现相同效果，但是如果子类很多，需要写很多重载函数，不利于代码维护。

#### 对象向下转型

向下转型主要特点在于需要使用到一些子类自己特殊的定义处理。

```java
class person {
    public void print() {
        System.out.println("人类行为");
    }
}

class superMan extends person {
    public String fly() {
        return "i can fly";
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        person per = new superMan();
        per.print();
        superMan man = (superMan) per;
        System.out.println(man.fly());
    }
}
```

向下转型之前一定要首先发生向上转型。

#### instanceof关键字

向下转型是一件存在有安全隐患的操作，所以为了保证向下转型的安全，所以在转换之前需要通过instanceof进行判断。

```java
public class JavaDemo {
    public static void main(String[] args) {
        person per = new superMan();
        if(per instanceof superMan){
            superMan man = (superMan) per;
            System.out.println(man.fly());
        }
    }
}
```

### Object类

Object类可以解决参数的统一问题，也就是说使用Object类可以接受所有的数据类型。

#### Object类的基本概念

在Java之中只有一个类是不存在有继承关系的，那么这个类就是Object，也就是说所有的类默认情况下都是Object的子类，以下两种类的定义完全相同：

```java
class person{}
class person extends Object{}
```

该类提供有无参构造方法。

Object类是所有类的父类，可以使用Object类接受所有的子类。因此，如果一个方法要求可以接收所有类对象的时候就可以利用Object实现处理。

需要注意，对于所有的引用数据类型都可以使用Object类进行接收，例如数组。

```java
public class JavaDemo {
    public static void main(String[] args) {
        Object obj = new int[]{1, 2, 3};//向上转型
        if (obj instanceof int[]) {
            int[] data = (int[]) obj;//向下转型
            for (int temp : data) {
                System.out.println(temp + "、");
            }
        }
    }
}
```

#### Object相关方法

https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html

- 获取对象完整信息：public [String](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html) toString()

对象输出时会自动调用toString()函数，所以是否调用toString()效果相同。因此，在开发过程中， 对象信息的获取可以直接覆写此方法

- 对象比较：public boolean equals([Object](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html) obj)

### 抽象类

#### 抽象类基本概念

父类的设计优先考虑抽象类。

抽象类的主要作用在于对子类中覆写方法进行约定，在抽象类里面可以定义一些抽象方法以实现这样的约定。

抽象方法指的是使用了abstract关键字定义的并且没有提供方法体的方法，而抽象方法所在的类必须为抽象类，抽象类必须使用abstract关键字进行定义（在普通类的基础上追加抽象方法就是抽象类）。

```java
abstract class Message {//定义抽象类
    private String type;//消息类型

    public abstract String getConnectInfo();//抽象方法

    public void setType(String type) {//普通方法
        this.type = type;
    }

    public String getType() {
        return this.type;
    }
}
```

抽象类不是完整的类不能直接new

- 抽象类必须提供有子类，子类使用extends继承一个抽象类
- 抽象类的子类一定要覆写抽象类中的全部抽象方法
- 抽象类的对象实例化可以利用对象多态性通过子类向上转型的方式完成

```java
class DatabaseMessage extends Message{
    @Override
    public String getConnectInfo(){
        return "数据库链接信息";
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        Message msg=new DatabaseMessage();
        msg.setType("客户消息");
        System.out.println(msg.getConnectInfo());
        System.out.println(msg.getType());
    }
}
```

- 抽象类无法直接实例化
- 使用抽象类的主要目的是进行过渡，解决类继承问题所带来的代码重复处理

#### 抽象类的相关说明

- 抽象类可以提供有构造方法，而且子类也一定会按照子类对象的实例化原则进行父类构造调用。
- 抽象类中允许没有抽象方法，但是即使没有抽象方法，也无法直接使用关键字new进行实例化。
- 抽象类之中可以提供有static方法，并且该方法不受到抽象类对象的局限。static方法不受到实例化对象或结构的限制

#### 模板设计模式

```java
abstract class Action {
    public static final int EAT = 1;
    public static final int SLEEP = 5;
    public static final int WORK = 10;

    public void command(int code) {
        switch (code) {
            case EAT: {
                this.eat();
                break;
            }
            case SLEEP: {
                this.sleep();
                break;
            }
            case WORK: {
                this.work();
                break;
            }
            case EAT + SLEEP + WORK: {
                this.eat();
                this.sleep();
                this.work();
                break;
            }
        }
    }

    public abstract void eat();

    public abstract void sleep();

    public abstract void work();
}

class Robot extends Action {
    @Override
    public void eat() {
        System.out.println("机器人需要接通电源充电");
    }

    @Override
    public void sleep() {
    }

    @Override
    public void work() {
        System.out.println("机器人按照指令工作");
    }
}

class Person extends Action {
    @Override
    public void eat() {
        System.out.println("人吃饭");
    }

    @Override
    public void sleep() {
        System.out.println("人睡觉");
    }

    @Override
    public void work() {
        System.out.println("人工作");
    }
}

class Pig extends Action {
    @Override
    public void eat() {
        System.out.println("猪吃饭");
    }

    @Override
    public void sleep() {
        System.out.println("猪睡觉");
    }

    @Override
    public void work() {
    }
}


public class JavaDemo {
    public static void main(String[] args) {
        Action robotAction = new Robot();
        Action personAction = new Person();
        Action pigAction = new Pig();

        robotAction.command(Action.EAT);
        robotAction.command(Action.WORK);
        personAction.command(Action.WORK);
        pigAction.command(Action.WORK);
        pigAction.command(Action.SLEEP);
    }
}
```

抽象类可以实现对子类方法的统一管理。

### 包装类

包装类的主要功能是基本数据类型的对象转换。

基本数据类型并不是一个类，所以现在如果想要将基本数据类型以类的形式进行处理，那么就需要对其进行包装。

#### 包装类实现原理分析

```java
class Int{
    private int data;
    public Int(int data){
        this.data=data;
    }
    public int intValue(){
        return this.data;
    }
}
public class JavaDemo {
    public static void main(String[] args) {
        Object obj=new Int(10);//装箱：将基本数据类型保存在包装类之中
        int x=((Int)obj).intValue();//拆箱：从包装类对象中获取基本数据类型
        System.out.println(x);
    }
}
```

基本数据类型进行包装处理之后可以像对象一样进行引用传递，同时也可以使用Object类进行接收。

基本数据类型一共有八种，故jdk提供有对应的八种包装类，如下：

- public abstract class **Number** extends [Object](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html) implements [Serializable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/Serializable.html)
  - public final class **Integer** extends [Number](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Number.html) implements [Comparable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Comparable.html)<Integer>, [Constable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/constant/Constable.html), [ConstantDesc](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/constant/ConstantDesc.html)
  - public final class **Byte** extends [Number](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Number.html) implements [Comparable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Comparable.html)<Byte>, [Constable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/constant/Constable.html)
  - public final class **Long** extends [Number](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Number.html) implements [Comparable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Comparable.html)<Long>, [Constable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/constant/Constable.html), [ConstantDesc](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/constant/ConstantDesc.html)
  - public final class **Short** extends [Number](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Number.html) implements [Comparable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Comparable.html)<Short>, [Constable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/constant/Constable.html)
  - public final class **Float** extends [Number](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Number.html) implements [Comparable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Comparable.html)<Float>, [Constable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/constant/Constable.html), [ConstantDesc](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/constant/ConstantDesc.html)
  - public final class **Double** extends [Number](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Number.html) implements [Comparable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Comparable.html)<Double>, [Constable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/constant/Constable.html), [ConstantDesc](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/constant/ConstantDesc.html)
- public final class **Boolean** extends [Object](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html) implements [Serializable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/Serializable.html), [Comparable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Comparable.html)<Boolean>, [Constable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/constant/Constable.html)
- public final class **Character** extends [Object](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html) implements [Serializable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/Serializable.html), [Comparable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Comparable.html)<Character>, [Constable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/constant/Constable.html)

number类中提供有六个获取包装类中基本数据类型的方法，如下

![image-20220320141729112](https://s2.loli.net/2022/03/20/AZhan3GQVwDF6kf.png)

#### 装箱与拆箱

数据装箱：将基本数据类型保存到包装类之中，一般可以通过构造方法实现

数据拆箱：从包装类之中获取基本数据类型

jdk1.9之后，包装类之中提供给构造方法就变为了过期处理（@Deprecated），不建议用户再继续使用，这是因为jdk1.5之后为了方便处理提供了自动的装箱拆箱操作。

```java
public class JavaDemo {
    public static void main(String[] args) {
        Integer obj = new Integer(10);
        int num = obj.intValue();
        System.out.println(num);
        Boolean obj1=new Boolean(true);
        boolean flag=obj1.booleanValue();
        System.out.println(flag);
        Integer obj2=10;//自动装箱
        obj2++;//直接参与数学运算
        int num2=obj2;//自动拆箱
        System.out.println(num2);
    }
}
```

除了提供有这种自动的数学运算支持之外，使用自动装箱最大的好处是可以实现Object接收基本数据类型

```java
public class JavaDemo {
    public static void main(String[] args) {
        Object obj = 19.2;//double自动装箱为Double，向上转型为Object
        //obj++;//这是错误的
        double num = (Double) obj;//向下转型为包装类，在自动拆箱
        System.out.println(num * 2);
    }
}
```

对于包装类的相等判断问题：

```java
public class JavaDemo {
    public static void main(String[] args) {
        Integer x=128;
        Integer y=128;
        System.out.println(x == y);
        System.out.println(x.equals(y));
    }
}
```

### 接口的定义与使用

#### 接口基本定义

抽象类与普通类相比最大的优势在于可以实现对子类覆写方法的控制，但是在抽象类之中仍然可以保留一些普通方法，而普通方法可能会涉及到一些安全或者隐私的操作问题，那么这样在进行开发的过程之中，如果要想对外部隐藏全部的实现细节，则可以通过接口进行描述。

接口可以理解为一个纯粹的抽象类（最原始的定义接口之中是只包含有抽象方法与全局常量的），但是从jdk1.8开始，由于引入了Lambda表达式的概念，接口的定义也得到了加强，除了抽象方法与全局常量之外，还可以定义普通方法和静态方法。

如果从设计本身的角度来讲，接口之中因该以抽象方法和全局常量为主。

在java之中，接口主要使用interface关键字来进行定义。

```java
//由于类名称和接口名称相同，为了方便区分，在接口名称前面加字母I
interface IMessage{
    public static final String info="lalal";
    public abstract String getInfo();
}
```

- 接口需要被子类实现（implements），一个子类可以实现多个父接口；
- 子类（如果不是抽象类），一定要腹泻接口之中的全部抽象方法
- 接口对象可以利用子类对象的向上转型进行实例化

```java
//由于类名称和接口名称相同，为了方便区分，在接口名称前面加字母I
interface IMessage {
    public static final String info = "lalal";

    public abstract String getInfo();
}

interface IChannel {
    public abstract boolean connect();
}

class Message implements IMessage, IChannel {
    @Override
    public String getInfo() {
        if (this.connect()) {
            return "得到消息";
        }
        return "通道创建失败，消息发送失败";
    }

    @Override
    public boolean connect() {
        System.out.println("消息发送通道已经成功建立");
        return true;
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        IMessage msg = new Message();
        IChannel chl = (IChannel) msg;
        System.out.println(msg.getInfo());
    }
}
```

- 

使用接口的主要原因是一个子类可以实现多个接口，但是这个时候需要注意对象的转型问题。

由于Message子类实现了IMessage与IChannel两个接口，所以这个子类可以是这两个接口任意一个的实例，此时这两个接口实例可以转换。

接口不允许继承父类，所以接口不会是Object的子类，但是接口可以通过Object进行接收。

```java
public class JavaDemo {
    public static void main(String[] args) {
        IMessage msg = new Message();
        Object obj = msg;
        IChannel chl = (IChannel) obj;
        System.out.println(chl.connect());
    }
}
```

接口不能继承父类，但是可以通过extends继承多个父接口

```java
interface IService extends IMessage,IChannel{}	
```

实际开发中接口使用的三种形式：

- 进行标准设置
- 表示一种操作的能力
- 暴露远程方法视图，一般在RPC分布式开发中使用

#### 接口定义加强

jdk1.8之后，在接口中允许开发者定义普通方法，但是必须使用default进行声明。

```java
public default boolean connect(){}
```

接口也可以定义static方法



#### 使用接口定义标准

接口最重要的应用就是进行标准的制定。

```java
interface IUSB{
    public boolean check();
    public void work();
}
class Computer{
    public void plugin(IUSB usb){
        if(usb.check()){
            usb.work();
        }
    }
}
class Keyboard implements IUSB{
    @Override
    public boolean check(){
        return true;
    }
    @Override
    public void work(){
        System.out.println("可以开始进行打字了");
    }
}
class Print implements IUSB{
    @Override
    public boolean check(){
        return true;
    }
    @Override
    public void work(){
        System.out.println("可以开始进行打印了");
    }
}
public class JavaDemo {
    public static void main(String[] args) {
        Computer computer=new Computer();
        computer.plugin(new Keyboard());
        computer.plugin(new Print());
    }
}
```

#### 工厂设计模式

```java
interface IFood {
    public void eat();
}

class Bread implements IFood {
    @Override
    public void eat() {
        System.out.println("吃面包");
    }
}

class Milk implements IFood {
    @Override
    public void eat() {
        System.out.println("喝牛奶");
    }
}

class Factory {
    public static IFood getInstance(String className) {
        if ("bread".equals(className)) {
            return new Bread();
        } else if ("milk".equals(className)) {
            return new Milk();
        } else {
            return null;
        }
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        String classname = "bread";
        IFood food = Factory.getInstance(classname);
        food.eat();
    }
}
```

#### 代理设计模式（Proxy）

代理设计的主要功能是帮助用户关注核心业务的开发。

代理设计模式的主要特点是：一个接口提供有两个子类，其中一个是真实业务操作类，另外一个子类是代理业务操作类，没有代理业务操作，真实业务无法进行。

```java
interface IEat{
    public void get();
}
class EatReal implements IEat{
    @Override
    public void get() {
        System.out.println("开始吃饭");
    }
}
class EatProxy implements IEat{
    private IEat eat;
    public EatProxy(IEat eat){
        this.eat=eat;
    }
    public void prepare(){
        System.out.println("准备食材");
    }

    @Override
    public void get() {
        this.prepare();
        this.eat.get();
        this.clear();
    }

    public void clear(){
        System.out.println("收拾碗筷");
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        IEat eat=new EatProxy(new EatReal());
        eat.get();
    }
}
```

#### 抽象类与接口区别

在实际的开发之中，抽象类和接口的定义形式是非常相似的，这是因为jdk1.8之后接口内部也可以定义普通方法和静态方法了。

|    区别    |                     抽象类                     |                     接口                     |
| :--------: | :--------------------------------------------: | :------------------------------------------: |
| 定义关键字 |          abstract class 抽象类名称{}           |             interface 接口名称{}             |
|    组成    | 构造、抽象、普通、静态方法，全局常量，成员属性 |        抽象、普通、静态方法，全局常量        |
|    权限    |              可以使用各种权限定义              |               只能够使用public               |
|  子类使用  |       子类通过extends可以继承一个抽象类        |   子类使用implements关键字可以实现多个接口   |
|    关系    |             抽象类可以实现多个接口             | 接口不允许继承抽象类，但是允许继承多个父接口 |

使用：

- 抽象类或接口必须定义子类
- 子类一定要覆写抽象类或接口中全部的抽象方法
- 通过子类的向上转型实现抽象类或接口对象实例化

当抽象类和接口都可以使用的情况下优先考虑接口，因为接口可以避免子类的单继承局限。

### 泛型

泛型是从jdk1.5之后追加到Java语言里面的，其主要目的是为了解决ClassCastException的问题，在进行对象的向下转型时一直存在安全隐患，Java希望通过泛型慢慢解决此问题。

#### 泛型问题引出

```java
class Point {
    private Object x;
    private Object y;

    public void setX(Object x) {
        this.x = x;
    }

    public void setY(Object y) {
        this.y = y;
    }

    public Object getX() {
        return x;
    }

    public Object getY() {
        return y;
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        Point point = new Point();
        point.setX(10);
        point.setY(20);
        int x = (Integer) point.getX();
        int y = (Integer) point.getY();
        System.out.println("x坐标：" + x + "、y坐标：" + y);
    }
}
```

这个类允许开发者保存三种数据类型。

本程序之所以可以保存三种数据类型，这是因为Object可以接收所有的数据类型，但是这会产生极大的安全隐患。

```java
public class JavaDemo {
    public static void main(String[] args) {
        Point point = new Point();
        point.setX(10);
        point.setY("北纬20°");
        int x = (Integer) point.getX();
        int y = (Integer) point.getY();
        System.out.println("x坐标：" + x + "、y坐标：" + y);
    }
}
```

该程序在编译时不会产生任何错误，但是程序在执行时会产生ClassCastException的异常类型，这是因为Object接受的数据类型范围太广了，如果这样的错误可以直接出现在编译的过程之中，就可以避免运行时的尴尬。

#### 泛型基本定义

 避免出现”ClassCastException“最好的做法是可以直接回避掉对象的强制转换，于是出现了泛型技术。

泛型的本质在于，类中的属性或方法的参数与返回值类型可以由对象实例化的时候动态决定。这就需要在类定义的时候使用明确的定义占位符（泛型标记），如果实例化时不设置泛型类型，则自动设置为Object

```java
class Point<T> {//T是Type的简介，可以定义多个泛型
    private T x;
    private T y;

    public void setX(T x) {
        this.x = x;
    }

    public void setY(T y) {
        this.y = y;
    }

    public Object getX() {
        return x;
    }

    public Object getY() {
        return y;
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        Point<Integer> point = new Point<>();
        Point<String> point1=new Point<>();
        point.setX(10);
        point.setY(20);
        point1.setX("东经10度");
        point1.setY("北纬20度");
        int x = (Integer) point.getX();
        int y = (Integer) point.getY();
        System.out.println("x坐标：" + x + "、y坐标：" + y);
        String x1 = (String) point1.getX();
        String y1 = (String) point1.getY();
        System.out.println("x1坐标：" + x1 + "、y1坐标：" + y1);
    }
}
```

泛型的使用注意点：

- 泛型之中只允许设置引用数据类型，如果现在要操作基本数据类型必须使用包装类
- 从jdk1.7开始，泛型对象实例化可以简化为Point<Integer> point = new Point<>();

使用泛型可以解决大部分的类对象的强制转换处理，这样的程序才是一个合格的设计。

#### 泛型通配符

虽然泛型帮助开发者解决了对象强制转换带来的安全隐患，但是也带来了新的问题

引用传递处理：示例中fun()方法仅可以传递String类型的Message对象，于是问题出现了，我们想要的是fun()方法可以接收任意一种泛型类型的Message对象。需要注意的是无法通过方法重载解决该问题

```java
class Message<T> {
    private T content;

    public T getContent() {
        return content;
    }

    public void setContent(T content) {
        this.content = content;
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        Message<String> msg = new Message<>();
        msg.setContent("lalalal");
        fun(msg);
    }

    public static void fun(Message<String> temp) {
        System.out.println(temp.getContent());
    }
}
```

需要使用通配符（？）解决该问题，此时在fun()方法里面由于采用了Message结合通配符的处理所以可以接收所有的类型，并且不允许修改只允许获取数据。

```java
public class JavaDemo {
    public static void main(String[] args) {
        Message<String> msg = new Message<>();
        msg.setContent("lalalal");
        fun(msg);
        Message<Integer> data = new Message<>();
        data.setContent(1);
        fun(data);
    }

    public static void fun(Message<?> temp) {
        System.out.println(temp.getContent());
    }
}
```

在”？“的基础上提供有两类通配符：

- ？extends类：设置泛型的上限
  - 例如：”？extends Number“表示该泛型类型只允许设置Number或Number的子类
- ？super类：设置泛型的下限
  - 例如："? super String"表示只能够使用String或String的父类

```java
class Message<T extends Number> {
    private T content;

    public T getContent() {
        return content;
    }

    public void setContent(T content) {
        this.content = content;
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        Message<Integer> data = new Message<>();
        data.setContent(1);
        fun(data);
    }

    public static void fun(Message<? extends Number> temp) {
        System.out.println(temp.getContent());
    }
}
```

```java
class Message<T> {
    private T content;

    public T getContent() {
        return content;
    }

    public void setContent(T content) {
        this.content = content;
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        Message<String> msg = new Message<>();
        msg.setContent("lalalal");
        fun(msg);
    }

    public static void fun(Message<? super String> temp) {
        System.out.println(temp.getContent());
    }
}
```

#### 泛型接口

```java
interface IMessage<T>{
    public String echo(T t);
}
```

对于泛型接口的子类而言有两种方法实现方式

- 在子类中继续设置泛型定义

```java
interface IMessage<T> {
    public String echo(T t);
}

class Message<S> implements IMessage<S> {
    @Override
    public String echo(S t) {
        return "ech0:" + t;
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        IMessage<String> msg=new Message<String>();
        System.out.println(msg.echo("lalalala"));
    }
}
```

- 在子类实现父接口的时候直接定义出具体泛型类型

```java
interface IMessage<T> {
    public String echo(T t);
}

class Message implements IMessage<String> {
    @Override
    public String echo(String t) {
        return "ech0:" + t;
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        IMessage<String> msg=new Message();
        System.out.println(msg.echo("lalalala"));
    }
}
```

#### 泛型方法

在泛型类之中如果将泛型标记写在了方法上，那么这样的方法就成为泛型方法，但是泛型方法不一定要出现在泛型类中。

```java
public class JavaDemo {
    public static void main(String[] args) {
        Integer[] num=fun(1,2,3);
        for (int i = 0; i < num.length; i++) {
            System.out.print(num[i]+"、");
        }
    }
    public static <T>T[] fun(T...args){
        return args;
    }
}
```

### 包的定义及使用

在项目开发过程之中，利用包可以实现类的包装，所有的类都要放在包里面，

#### 包的定义

同一个目录之中不允许存放有相同的程序类文件，但是开发过程中很难保证类的不重复。 所以为了可以进行类的方便管理，那么往往可以将程序文件放在不同目录下，这个目录就称为包。包=目录。





-----

### 单例和多例设计模式

#### 单例设计

例：windows回收站

在系统加载类时就提供有实例化对象

```java
class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {
    }

    public void print() {
        System.out.println("test");
    }

    public static Singleton getInstance() {
        return INSTANCE;
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        Singleton instance = null;
        instance = Singleton.getInstance();
        instance.print();
    }
}
```

在类第一次使用的时候就行实例化对象处理

```java
class Singleton {
    private static Singleton instance;

    private Singleton() {
    }

    public void print() {
        System.out.println("test");
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        Singleton instance = null;
        instance = Singleton.getInstance();
        instance.print();
    }
}
```

#### 多例设计

例：描述性别类只能有男、女两个实例化对象，描述颜色基色只能有红色、绿色、蓝色三个实例化对象。

```java
class Color {
    private static final Color RED = new Color("红色");
    private static final Color GREEN = new Color("绿色");
    private static final Color BLUE = new Color("蓝色");
    private String title;

    private Color(String title) {
        this.title = title;
    }

    @Override
    public String toString() {
        return this.title;
    }

    public static Color getInstance(String color) {
        switch (color) {
            case "red":
                return RED;
            case "green":
                return GREEN;
            case "blue":
                return BLUE;
            default:
                return null;
        }
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        Color c = Color.getInstance("green");
        System.out.println(c);
    }
}
```

### 枚举

枚举的主要作用是定义有限个数对象的一种结构，枚举就属于多例设计。

#### 定义枚举类

```java
enum Color {//枚举类
    RED, GREEN, BLUE;//实例化对象
}

public class JavaDemo {
    public static void main(String[] args) {
        Color c = Color.RED;//获取实例化对象
        System.out.println(c);
        for (int i = 0; i < Color.values().length; i++) {
            System.out.println(Color.values()[i]);
        }
        switch (c){
            case RED -> {
                System.out.println("红色");
                break;
            }
            case BLUE -> {
                System.out.println("蓝色");
                break;
            }
            case GREEN -> {
                System.out.println("绿色");
                break;
            }
            default -> {
                System.out.println("meiyou");
            }
        }
    }
}
```

#### Enum类

https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Enum.html

枚举本质是一个类，会默认继承Enum类。

public abstract class **Enum<E extends Enum<E>>** 

extends [Object](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html) 

implements [Constable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/constant/Constable.html), [Comparable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Comparable.html)<E>, [Serializable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/Serializable.html)

```java
enum Color {//枚举类
    RED, GREEN, BLUE;//实例化对象
}

public class JavaDemo {
    public static void main(String[] args) {
        Color c = Color.RED;//获取实例化对象
        System.out.println(c);
        for (int i = 0; i < Color.values().length; i++) {
            System.out.println(Color.values()[i].ordinal()+":"+Color.values()[i].name());
        }
    }
}
```

枚举之中每一个对象的序号都是根据枚举对象的定义顺序来决定的。

#### 定义枚举结构

枚举类是一种多例设计模式，其中的构造方法不能够使用非私有方法。

枚举类可以继承接口。

```java
interface IMessage {
    public String getMessage();
}

enum Color implements IMessage {//枚举类
    RED("红色"), GREEN("绿色"), BLUE("蓝色");//实例化对象,枚举对象要写在首行
    private String title;

    private Color(String title) {
        this.title = title;
    }

    @Override
    public String toString() {
        return this.title;
    }

    @Override
    public String getMessage() {
        return this.title;
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        IMessage msg = Color.BLUE;
        System.out.println(msg.getMessage());
        for (int i = 0; i < Color.values().length; i++) {
            System.out.println(Color.values()[i].ordinal() + ":" + Color.values()[i].name());
        }
    }
}
```

枚举类可以直接定义抽象方法，而且其中的每一个对象都要各自对该方法进行覆写。（用处不大）

#### 枚举应用举例

```java
enum Gender {
    MALE("男"), FEMALE("女");
    private String title;

    private Gender(String title) {
        this.title = title;
    }

    @Override
    public String toString() {
        return "Gender{" +
                "title='" + title + '\'' +
                '}';
    }
}

class Person {
    private String name;
    private int age;
    private Gender gender;

    public Person(String name, int age, Gender gender) {
        this.name = name;
        this.age = age;
        this.gender = gender;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", gender=" + gender +
                '}';
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        System.out.println(new Person("张三", 20, Gender.MALE));
    }
}
```

### 异常的捕获及处理

#### 认识异常对程序的影响

异常指的是导致程序中断执行的一种指令流。

为了保证程序出现非致命错误之后程序依然可以正常完成，需要有一个完善的异常处理机制。

#### 处理异常

在Java之中如果要进行异常的处理，需要使用如下几个关键字：try、catch、finally，其基本处理结构如下

```java
try{
	//可能出现异常的语句
} catch (异常类型 异常对象) {
	//异常处理
} catch (异常类型 异常对象) {
	//异常处理
} catch (异常类型 异常对象) {
	//异常处理
} finally{
	//不管异常是否处理都要执行
}
```





```java
public class JavaDemo {
    public static void main(String[] args) {
        System.out.println("--------程序开始执行---------");
        try {
            System.out.println(10 / 0);
        } catch (ArithmeticException e) {
            System.out.println(e);
            e.printStackTrace();
        }finally {
            System.out.println("-------始终执行本语句--------");
        }
        System.out.println("--------程序执行完毕---------");
    }
}
```



#### 异常处理流程

![image-20220321234144195](https://s2.loli.net/2022/03/21/xuIgLpG6Yloi3Mq.png)

#### throws关键字

```java
class MyMath {
    public static int div(int x, int y) throws Exception {
        return x / y;
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        try {
            System.out.println(MyMath.div(10, 0));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### throw关键字

作用：手工产生一个异常类实例化对象，并且进行异常的抛出处理。

```java
public class JavaDemo {
    public static void main(String[] args) {
        try {
            throw new Exception("自己抛着玩的对象");//异常对象不是由系统生成的，而是自己定义的
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### 异常处理模型

```java
class MyMath {
    //异常交给被调用处处理则一定要在方法上使用throws
    public static int div(int x, int y) throws Exception {
        int temp = 0;
        System.out.println("除法计算开始");
        try {
            temp = x / y;
        } catch (Exception e) {
            throw e;
        } finally {
            System.out.println("除法计算结束");
        }
        return temp;
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        try {
            System.out.println(MyMath.div(10, 0));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

实际开发中，类似上述代码中catch与throw操作可以省略

```java
class MyMath {
    //异常交给被调用处处理则一定要在方法上使用throws
    public static int div(int x, int y) throws Exception {
        int temp = 0;
        System.out.println("除法计算开始");
        try {
            temp = x / y;
        } finally {
            System.out.println("除法计算结束");
        }
        return temp;
    }
}
```

#### RuntimeException

考虑到代码编写的方便，提供有一个灵活可选的异常处理父类“RuntimeException”，这个类的子类可以不需要进行强制性异常处理。

#### 自定义异常类

自定义异常有两种实现方式：

- 继承Exception
- 继承RuntimeException

```java
class BombException extends RuntimeException {
    public BombException(String msg) {
        super(msg);
    }
}

class Food {
    public static void eat(int num) throws BombException {
        if (num > 10) {
            throw new BombException("吃太多了,肚子撑爆了");
        } else {
            System.out.println("没啥事，接着吃");
        }
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        Food.eat(11);
    }
}
```

#### assert断言

jdk1.4之后java提供有断言功能，确定代码执行到某行之后一定是所期待的结果。

但是在实际开发之中，对于断言而言并不一定准确，可能存在偏差，但是这种偏差不应该影响程序的执行。

```java
public class JavaDemo {
    public static void main(String[] args) {
        int x = 10;
        //x变量中间经过多次操作
        assert x == 100 : "x的内容不是100";
        System.out.println(x);
    }
}
```

如果想要执行断言，则必须在程序执行的时候加入参数：java -ea JavaDemo

所以说，在java里面没有将断言设置为一个程序必须执行的步骤，需要特定环境下才可以执行。

### 内部类

#### 内部类基本概念

在一个类的内部可以定义其他的类，这样的类就称为内部类。

```java
class Outer {
    private String msg = "江南大学";

    public void fun() {
        Inner in = new Inner();
        in.print();
    }

    class Inner {
        public void print() {
            System.out.println(Outer.this.msg);
        }
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        Outer out = new Outer();
        out.fun();
    }
}
```

#### 内部类相关说明

内部类可以方便的访问外部类的私有成员和方法，外部类也可以轻松访问内部类私有成员和方法。

因此，使用内部类之后，内部类与外部类之间私有成员的相互访问就不需要setter、getter等方法。

```java
class Outer {
    private String msg = "江南大学";

    public void fun() {
        Inner in = new Inner();
        in.print();
        System.out.println(in.info);
    }

    class Inner {
        private String info = "今天天气不好，收衣服了";

        public void print() {
            System.out.println(Outer.this.msg);
        }
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        Outer out = new Outer();
        out.fun();
    }
}
```

-----

在内部类编译完成之后会产生一个“Outer$Inner.class"类文件，所以内部类的全称即”外部类.内部类“

```java
外部类.内部类 内部类对象=new 外部类().new 内部类();
```

```java
class Outer {
    private String msg = "江南大学";

    class Inner {
        public void print() {
            System.out.println(Outer.this.msg);
        }
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        Outer.Inner in = new Outer().new Inner();
        in.print();
    }
}
```

如果想要内部类只允许外部类使用，需要在内部类定义时使用private进行定义，此时内部类无法在外部使用。

内部接口

```java
interface IChannel {
    public void send(IMessage msg);

    interface IMessage {
        public String getContent();
    }
}

class ChannelImpl implements IChannel {
    @Override
    public void send(IMessage msg) {
        System.out.println("发送信息：" + msg.getContent());

    }

    class MessageImpl implements IMessage {
        @Override
        public String getContent() {
            return "jiangnandaxue";
        }
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        IChannel channel = new ChannelImpl();
        channel.send(((ChannelImpl) channel).new MessageImpl());
    }
}
```

内部抽象类可以定义在普通类、抽象类、接口内部。

```java
interface IChannel {
    public void send();

    abstract class AbstractMessage {
        public abstract String getContent();
    }
}

class ChannelImpl implements IChannel {
    @Override
    public void send() {
        AbstractMessage msg = new MessageImpl();
        System.out.println(msg.getContent());
    }

    class MessageImpl extends AbstractMessage {
        @Override
        public String getContent() {
            return "jiangnandaxue";
        }
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        IChannel channel = new ChannelImpl();
        channel.send();
    }
}
```

如果定义了一个接口，可以在接口内部利用内部类实现该接口。

```java
interface IChannel {
    public void send();

    class ChannelImpl implements IChannel {
        @Override
        public void send() {
            System.out.println("jiangnandaxue");
        }
    }

    public static IChannel getInstance() {
        return new ChannelImpl();
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        IChannel channel = IChannel.getInstance();
        channel.send();
    }
}
```

内部类是一种非常灵活的结构，需要灵活使用。

#### static定义内部类

如果在内部类上使用管理static定义，那么这个内部类就变成了”内部类“。

外部实例化static内部类方法如下

```java
外部类.内部类 内部类对象=new 外部类().内部类();
```

```java
class Outer {
    private static final String MSG = "jiangnandaxue";

    static class Inner {
        public void print() {
            System.out.println(Outer.MSG);
        }
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        Outer.Inner in = new Outer.Inner();
        in.print();
    }
}
```

在实际开发中，static定义内部类的方法并不常用，static定义内部接口十分常见。

之所以使用static定义的内部接口，主要是因为这些操作是属于一组相关的定义，有了外部接口之后可以更加明确的描述出这些接口的主要功能。

```java
interface IMessageWrap {
    static interface IMessage {
        public String getContent();
    }

    static interface IChannel {
        public boolean connect();
    }

    public static void send(IMessage msg, IChannel channel) {
        if (channel.connect()) {
            System.out.println(msg.getContent());
        } else {
            System.out.println("消息通道无法建立，消息发送失败。");
        }
    }
}

class DefaultMessage implements IMessageWrap.IMessage {
    @Override
    public String getContent() {
        return "江南大学";
    }
}

class NetChannel implements IMessageWrap.IChannel {
    @Override
    public boolean connect() {
        return true;
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        IMessageWrap.send(new DefaultMessage(), new NetChannel());
    }
}
```

#### 在方法中定义内部类

内部类可以在任意结构中进行定义，这就包括了：类中、方法中、代码块中，在实际开发中，在方法中定义内部类的操作较多。

```java
class Outer {
    private String msg = "江南大学";

    public void fun(long time) {
        class Inner {
            private void print() {
                System.out.println(Outer.this.msg);
                System.out.println(time);
            }
        }
        new Inner().print();
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        new Outer().fun(23454351342L);
    }
}
```

#### 匿名内部类

匿名内部类是一种简化的内部类，主要在抽象类和接口的子类上使用。

```java
interface IMessage {
    public void send(String str);
}

class MessageImpl implements IMessage {
    @Override
    public void send(String str) {
        System.out.println(str);
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        IMessage msg = new MessageImpl();
        msg.send("江南大学");
    }
}
```

如果说现在IMessage接口中的MessageImpl子类只使用唯一的一次，则没有必要将其定义为单独的类，此时可以使用匿名内部类。

```java
interface IMessage {
    public void send(String str);
}

public class JavaDemo {
    public static void main(String[] args) {
        IMessage msg = new IMessage() {//匿名内部类
            @Override
            public void send(String str) {
                System.out.println(str);
            }
        };
        msg.send("江南大学");
    }
}
```

有些时候为了更加方便的体现出匿名内部类的使用，往往可以利用静态方法做一个内部的匿名内部类实现。

```java
interface IMessage {
    public void send(String str);

    public static IMessage getInstance() {
        return new IMessage() {
            @Override
            public void send(String str) {
                System.out.println(str);
            }
        };
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        IMessage.getInstance().send("江南大学");
    }
}
```

### 函数式编程

#### Lambda表达式

```java
@FunctionalInterface
interface IMessage {
    public void send(String str);
}

public class JavaDemo {
    public static void main(String[] args) {
        IMessage msg = (str) -> {
            System.out.println("发送消息：" + str);
        };
        msg.send("江南大学");
    }
}
```

Lambda表达式的使用要求：SAM（Single Abstract Method），

只提供一个方法的接口称为函数式接口，只有函数式接口才可以使用Lambda表达式。

Lambda表达式使用格式：

- 方法没有参数：

  ```java
  ()->{}
  ```

- 方法有参数

  ```java
  (参数,参数)->{}
  ```

- 只有一行返回语句

  ```java
  (参数,参数)->语句;
  ```

#### 方法引用

引用数据类型最大的特点是可以进行内存的指向处理。

jdk1.8之后也提供有方法的引用，即，不同的方法名称可以描述同一个方法。

在java中，方法的引用有如下四种形式：

- 引用静态方法：类名称  ::static 方法名称
- 引用某个实例对象的方法：实例化对象 :: 普通方法
- 引用特定类型的方法：特定类 :: 普通方法
- 引用构造方法：类名称 :: new

```java
//P描述的是参数，R描述的是返回值
@FunctionalInterface
interface IFunction<P, R> {
    public R trans(P p);
}

public class JavaDemo {
    public static void main(String[] args) {
        IFunction<Integer, String> fun = String::valueOf;//引用静态方法
        String str = fun.trans(100);
        System.out.println(str.length());
    }
}
```

```java
@FunctionalInterface
interface IFunction<R> {
    public R upper();
}

public class JavaDemo {
    public static void main(String[] args) {
        IFunction<String> fun = "String"::toUpperCase;//引用实例化对象的方法
        String str = fun.upper();
        System.out.println(str);
    }
}
```

```java
class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

@FunctionalInterface
interface IFunction<R> {
    public R create(String s, int a);
}

public class JavaDemo {
    public static void main(String[] args) {
        IFunction<Person> fun = Person::new;
        System.out.println(fun.create("张三", 20));
    }
}
```

#### 内建函数式接口

 java.util.function支持我们直接使用函数式接口。

常用接口如下：

- 功能型函数式接口

```java
@FunctionalInterface
public interface Function<T,R>{
	public R apply(T t)
}
public class JavaDemo {
    public static void main(String[] args) {
        Function<String, Boolean> fun = "Hello"::startsWith;
        System.out.println(fun.apply("He"));
    }
}
```

- 消费型函数式接口：只能够进行数据的处理操作，没有任何返回

```java
@FunctionalInterface
public interface Consumer<T>{
	public void accept(T t)
}
public class JavaDemo {
    public static void main(String[] args) {
        Consumer<String> fun = System.out::println;
        fun.accept("江南大学");
    }
}
```

- 供给型函数式接口

```java
@FunctionalInterface
public interface Supplier<T>{
	public T get();
}
public class JavaDemo {
    public static void main(String[] args) {
        Supplier<String> fun = "ZHUWEIHAO"::toLowerCase;
        System.out.println(fun.get());
    }
}
```

- 断言型函数式接口：进行判断处理

```java
@FunctionalInterface
public interface Predicate<T>{
	public boolean test(T t);
}
public class JavaDemo {
    public static void main(String[] args) {
        Predicate<String> fun = "zhu"::equalsIgnoreCase;
        System.out.println(fun.test("ZHU"));
    }
}
```

在实际开发中，如果JDK本身提供的函数式接口可以满足我们的要求，就没有必要另行定义。

## Java高级编程

### Java多线程编程

#### 进程与线程

在Java语言中最大的特点是支持多线程开发(也是为数不多支持多线程的编程语言)，所以在整个的JAVA技术的学习里面，如果不能够对多线程的概念有一个全面并且细致的了解，则在日后进行一些项目设计的过程之中是并发访问设计的过程之中就会出现严重的技术缺陷。

传统的DOS采用的是单进程处理，而单进程处理的最大特点：在同一个时间段上只允许一个程序在执行。 

后来到了Windows 的时代就开启了多进程的设计，于是就表示在一个时间段上可以同时运行多个程序，并且这些程序将进行资源的轮流抢占。所以在同一个时间段上会有多个程序依次执行，但是在同一个时间点上只会有一个进程执行，而后来到了多核的CPU，由于可以处理的CPU多了，那么即便有再多的进程出现，也可以比单核CPU处理的速度有所提升。

线程是在进程基础之上划分的更小的程序单元，线程是在进程基础上创建并且使用的，所以线程依赖于进程的支持，但是线程的启动速度要比进程快很多，所以当使用多线程进行并发处理的时候，其执行的性能要高于进程。   

Java是多线程的编程语言，所以Java在进行并发访问处理的时候可以得到更高的处理性能。

#### Thread类实现多线程

如果要想在 Java 之中实现多线程的定义，那么就需要有一个专门的线程主体类进行线程的执行任务的定义，而这个主体类的定义是有要求的，必须实现特定的接口或者继承特定的父类才可以完成。

Java 里面提供有一个 java.lang.Thread 的程序类，那么一个类只要继承了此类就表示这个类为线程的主体类；

```java
class MyThread extends Thread{//线程的主体类

}
public class ThreadDemo {
}
```

但是并不是说这个类就可以直接实现多线程处理了，因为还需要覆写 Thread 类中提供的一个 run(public void run()）方法，而这个方法就属于线程的主方法。

```java
class MyThread extends Thread {//线程的主体类
    private String title;

    public MyThread(String title) {
        this.title = title;
    }

    @Override
    public void run() {//线程的主体方法
        for (int i = 0; i < 10; i++) {
            System.out.println(this.title + "运行.x=" + i);
        }
    }
}

public class ThreadDemo {
}
```

多线程要执行的功能都应该在 run()方法中进行定义。

需要说明的是：在正常情况下如果要想使用一个类中的方法，那么肯定要产生实例化对象，而后去调用类中提供的方法，但是 run()方法是不能够被直接调用的，因为这里面牵扯到一个操作系统的资源调度问题，所以要想启动多线程必须使用start() 方法完成（public void start()）。

```java
public class ThreadDemo {
    public static void main(String[] args) {
        new MyThread("线程A").start();
        new MyThread("线程B").start();
        new MyThread("线程C").start();
    }
}
```

通过此时的调用可以发现，虽然调用了是 start()方法，但是最终执行的是 run()方法，并且所有的线程对象都是交替执行的。

-----

小问题：为什么多线程的启动不直接使用 run() 方法而必须使用 Thread 类中的 start()方法呢?用操作方法

```java
public synchronized void start() {
    /**
     * This method is not invoked for the main method thread or "system"
     * group threads created/set up by the VM. Any new functionality added
     * to this method in the future may have to also be added to the VM.
     *
     * A zero status value corresponds to state "NEW".
     */
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    /* Notify the group that this thread is about to be started
     * so that it can be added to the group's list of threads
     * and the group's unstarted count can be decremented. */
    group.add(this);

    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
            /* do nothing. If start0 threw a Throwable then
              it will be passed up the call stack */
        }
    }
}
private native void start0();
```

发现在 start() 方法里面会抛出一个“IllegalThreadStateException”异常类对象，但是整个的程序并没有使用 throws 或者是明确的 try..catch 处理，因为该异常一定是 RuntimeException 的子类，每一个线程类的对象只允许启动一次,如果重复启动则就抛出此异常。

```java
public class ThreadDemo {
    public static void main(String[] args) {
        MyThread mt =new MyThread("线程A");
        mt.start();
        mt.start();
    }
}
```

在 Java 程序执行的过程之中考虑到对于不同层次开发者的需求，所以其支持有本地的操作系统函数调用，而这项技术就被称为 JNI（Java Native Inteface）技术，但是 Java 开发过程之中并不推荐这样使用，利用这项技术可以使用一些操作系统提供底层函数进行一些特殊的处理，而在 Thread 类里面提供的 start0()就表示需要将此方法依赖于不同的操作系统实现。

![image-20220524102841043](https://s2.loli.net/2022/05/24/VGdS3r4q27beQW1.png)

任何情况下，只要定义了多线程，多线程的启动永远只有一种方案:Thread 类中的start()方法。

#### Runnable接口实现多线程

虽然可以通过Thread类的继承来实现多线程的定义，但是在Java程序里面对于继承永远都是存在有单继承局限的，所以在Java里面又提供有第二种多线程的主体定义结构形式:实现java.lang.Runnable接口，此接口定义如下：

```java
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface {@code Runnable} is used
     * to create a thread, starting the thread causes the object's
     * {@code run} method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method {@code run} is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}
```

但是此时由于不再继承Thread父类了，那么对于此时的MyThread类中也就不再支持有start()这个继承的方法，可是如果不使用Thread.start()方法是无法进行多线程启动的，那么就需要去观察一下Thread类所提供的构造。

- 构造方法：public Thread([Runnable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Runnable.html) target)

```java
class MyThread implements Runnable {//线程的主体类
    private String title;

    public MyThread(String title) {
        this.title = title;
    }

    @Override
    public void run() {//线程的主体方法
        for (int i = 0; i < 10; i++) {
            System.out.println(this.title + "运行.x=" + i);
        }
    }
}

public class ThreadDemo {
    public static void main(String[] args) {
        Thread thread1 = new Thread(new MyThread("线程对象1"));
        thread1.start();
        Thread thread2 = new Thread(new MyThread("线程对象2"));
        thread2.start();
        Thread thread3 = new Thread(new MyThread("线程对象3"));
        thread3.start();
    }
}
```

多线程实现里面可以发现，由于只是实现了Runnable 接口对象，所以此时线程主体类上就不再有单继承局限了，这样的设计才是一个标准型的设计。

从JDK1.8开始，Runnable接口使用了函数式接口定义，所以也可以直接利用Lambda表达式进行线程类实现。

```java
public class ThreadDemo {
    public static void main(String[] args) {
        for (int x = 0; x < 3; x++) {
            String title = "线程对象" + x;
            Runnable runnable = () -> {
                for (int y = 0; y < 10; y++) {
                    System.out.println(title + "运行.y=" + y);
                }
            };
            new Thread(runnable).start();
        }
    }
}

//这种方法更加能够体现出Lambda表达式的优势
public class ThreadDemo {
    public static void main(String[] args) {
        for (int x = 0; x < 3; x++) {
            String title = "线程对象" + x;
            new Thread(() -> {
                for (int y = 0; y < 10; y++) {
                    System.out.println(title + "运行.y=" + y);
                }
            }).start();
        }
    }
}
```

#### Thread与Runnable的关系

在多线程的实现过程之中已经有了两种做法:Thread 类、Runnable 接口，如果从代码的结构本身来讲肯定使用 Runnable 是最方便的，因为其可以避免单继承的局限，同时也可以更好的进行功能的扩充。

hread 类的定义:public class **Thread** extends [Object](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html) implements [Runnable](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Runnable.html)

发现现在 Thread 类也是 Runnable接口的子类，那么在之前继承 Thread 类的时候实际上覆写的还是 Runnable 接口的 run() 方法

![image-20220524111117708](https://s2.loli.net/2022/05/24/XSI8tU4PJ5myLrR.png)

在进行 Thread 启动多线程的时候调用的是 start() 方法，而后找到的是 run()方法。

通过 Thread 类的构造方法传递了一个 Runnable 接口对象的时候，那么该接口对象将被 Thread 类中的 target 属性所保存，在 start() 方法执行的时候会调用Thread类中的 run() 方法，而这个 run() 方法去调用 Runnable 接口子类被覆写过的 run() 方法。

多线程开发的本质实质上是在于多个线程可以进行同一资源的抢占，那么 Thread 主要描述的是线程,而资源的描述是通过 Runnable 完成的。

![image-20220524111746454](https://s2.loli.net/2022/05/24/YOh4QxPFWv2UMtG.png)

- 卖票程序来实现多个进程的资源并发访问

```java
class MyThread implements Runnable {//线程的主体类
    private int ticket = 5;

    @Override
    public void run() {//线程的主体方法
        for (int i = 0; i < 100; i++) {
            if (this.ticket > 0) {
                System.out.println("卖票.ticket=" + this.ticket--);
            }
        }
    }
}

public class ThreadDemo {
    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        new Thread(myThread).start();//第一个线程启动
        new Thread(myThread).start();//第二个线程启动
        new Thread(myThread).start();//第三个线程启动
    }
}
```

![image-20220524113005637](https://s2.loli.net/2022/05/24/zI6n9ySN3fq2Hpw.png)

#### Callable实现多线程

从最传统的开发来讲如果要进行多线程的实现肯定依靠的就是 Runnable。

但是 Runnable 接口有一个缺点：当线程执行完毕之后后无法获取一个返回值，所以从 JDK 1.5 之后就提出了一个新的线程实现接口: java.util.concurrent.Collable 接口。 

```java
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

可以发现 Callable 定义的时候可以设置一个泛型，此泛型的类型就是返回数据的类型，这样的好处是可以避免向下转型所带来的安全隐患。

![](https://s2.loli.net/2022/05/24/l5GqiCRr879FBdY.png)

```java
class MyThread implements Callable<String> {
    @Override
    public String call() throws Exception {
        for (int x = 0; x < 10; x++) {
            System.out.println("********线程执行.x=" + x);
        }
        return "线程执行完毕";
    }
}

public class ThreadDemo {
    public static void main(String[] args) throws Exception{
        FutureTask<String> futureTask=new FutureTask<>(new MyThread());
        new Thread(futureTask).start();
        System.out.println("线程返回数据:"+futureTask.get());
    }
}
```

Runnable与Callable的区别

- Runnable 是在 JDK1.0的时候提出的多线程的实现接口，而 Callable 是在 JDK 1.5之后提出的；
- java.lang.Runnable 接口之中只提供有一个 run()方法，并且没有返回值;
- java.util.concurrent.Callable 接口提供有 call()方法，可以有返回值;

#### 多线程运行状态

对于多线程的开发而言，编写程序的过程之中总是按照：

定义线程主体类，而后通过Thread类进行线程，但是并不意味着你调用了start()方法，线程就已经开始运行了，因为整体的线程处理有自己的一套运行的状态。

![](https://s2.loli.net/2022/05/24/vQMhepF3yZJHoU7.png)

- 任何一个线程的对象都应该使用Thread类进行封装，所以线程的启动使用的是start（），但是启动的时候实际上若干个线程都将进入到一种就绪状态，现在并没有执行；
- 进入到就绪状态之后就需要等待进行资源的调度，当某一个线程调度成功之后进入到运行状态（run（）方法），但是所有的线程不可能一直持续执行下去，中间需要产生一些暂停的状态，例如：某个线程执行一段时间之后就需要让出资源；而后这个线程就进入到阻塞状态随后重新回归到就绪状态；
- 当run()方法执行完毕之后，实际上该线程的主要任务也就结束了，那么此时就可以直接进入到停止状态。

### 线程常用操作方法

多线程的主要操作方法都在Thread类中定义了

#### 线程的命名和取得

多线程的运行状态是不确定的，那么在程序开发之中为了可以获取到一些需要使用的线程就只能依靠线程的名字来进行操作。

所以线程的名字是一个至关重要的概念，在Thread类之中就提供有线程名称的处理；

- 构造方法：public Thread(Runnable target,String name);
- 设置名字：public final void setName(String name)；
- 取得名字：public final String getName()；

对于线程对象的获得是不可能只靠一个this来完成的，因为线程的状态不可控，但是有一点是明确的，所有的线程对象都一定要执行run()方法，那么这个时候可以考虑获取当前线程，在Thread类里面提供有获取当前线程的一个方法。

- 获取当前线程：public static Thread current Thread()；

```java
class MyThread implements Runnable{
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }
}
public class ThreadDemo {
    public static void main(String[] args) throws Exception{
        MyThread myThread=new MyThread();
        new Thread(myThread,"线程A").start();//设置了线程的名字
        new Thread(myThread).start();             //未设置线程名字
        new Thread(myThread).start();
        new Thread(myThread,"线程C").start();//设置了线程的名字
    }
}
```

当开发者为线程设置名字的时候，如果没有设置名字，则会自动生成一个不重复的名字，这种自动的属性命名主要是依靠了static属性完成的，在Thread类里面定义了如下操作：

```java
/* For autonumbering anonymous threads. */
private static int threadInitNumber;
private static synchronized int nextThreadNum() {
    return threadInitNumber++;
}
```

```java
class MyThread implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }
}

public class ThreadDemo {
    public static void main(String[] args) throws Exception {
        MyThread myThread = new MyThread();
        new Thread(myThread, "线程A").start();//设置了线程的名字
        myThread.run();
    }
}
```

通过此时的代码可以发现当使用了“myThread.run()”直接在主方法之中调用线程类对象中的run（）方法所获得的线程对象的名字为“main”所以可以得出一个结论：主方法也是一个线程。

那么现在问题来了，所有的线程都是在进程上的划分，那么进程在哪里？

每当使用Java命令执行程序的时候就表示启动了一个JVM的进程，一台电脑上可以同时启动若干个JVM进程所以每一个JVM的进程都会有各自的线程。



在任何的开发之中，主线程可以创建若干个子线程，创建子线程的目的是可以将一些复杂逻辑或者比较耗时的逻辑交给子线程处理；

```java
public class ThreadDemo {
    public static void main(String[] args) throws Exception {
        System.out.println("1、执行操作任务一：");
        new Thread(()->{
            int temp = 0;
            for (int x = 0; x < Integer.MIN_VALUE; x++) {
                temp += x;
            }
        }).start();
        System.out.println("2、执行操作任务二：");
        System.out.println("3、执行操作任务三：");
    }
}
```

主线程负责处理整体流程，而子线程负责处理耗时流程。

#### 线程的休眠

如果现在希望某一个线程可以暂缓执行，那么可以使用休眠的处理。

在 Thread 类之中定义的休眠的方法如下：

- public static void sleep(long millis)  throws [InterruptedException](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/InterruptedException.html)
- public static void sleep(long millis, int nanos)  throws [InterruptedException](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/InterruptedException.html)

在进行休眠的时候有可能会产生中断异常“InterruptedException”,中断异常属于Exception 的子类，所以证明该异常必须进行休眠处理。

```java
public class ThreadDemo {
    public static void main(String[] args) throws Exception {
        new Thread(()->{
            for (int x = 0; x < 10; x++) {
                System.out.println(Thread.currentThread().getName()+"、x="+x);
                try {
                    Thread.sleep(1000);//暂缓执行
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"线程对象").start();
    }
}
```

休眠的主要特点是可以自动实现线程的唤醒，以继续进行后续的处理。但是需要注意的是，如果现在你有多个线程对象，那么休眠也是有先后顺序的。

```java
public class ThreadDemo {
    public static void main(String[] args) throws Exception {
        for (int num = 0; num < 5; num++) {
            new Thread(() -> {
                for (int x = 0; x < 10; x++) {
                    System.out.println(Thread.currentThread().getName() + "、x=" + x);
                    try {
                        Thread.sleep(1000);//暂缓执行
                        System.out.println("---------------");
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }, "线程对象-" + num).start();
        }
    }
}
```

![](https://s2.loli.net/2022/05/24/voXR2hpAt9BwaTH.png)

#### 线程中断

在之前发现线程的休眠里面提供有一个中断异常，实际上就证明线程的休眠是可以被打断的，而这种打断肯定是由其他线程完成的，在Thread类里面提供有这种中断执行的处理方法：

- 判断线程是否被中断：public boolean isInterrupted()
- 中断线程执行：public static boolean interrupted()

```java
public class ThreadDemo {
    public static void main(String[] args) throws Exception {
        Thread thread = new Thread(() -> {
            System.out.println("-----72个小时的疯狂我需要补充精力-----");
            try {
                Thread.sleep(10000);
            } catch (InterruptedException e) {
                //e.printStackTrace();
                System.out.println("打扰我睡觉，老子杀了你");
            }
            System.out.println("-----睡足了接着嗨-----");
        });
        thread.start();
        Thread.sleep(1000);
        if (!thread.isInterrupted()) {
            System.out.println("我偷偷的打扰一下你的睡眠");
            thread.interrupt();
        }
    }
}
```

所有正在执行的线程都是可以被中断的，中断线程必须进行异常的处理。

#### 线程的强制执行

所谓的线程的强制执行指的是当满足于某些条件之后，某一个线程对象可以一直独占资源一直到该线程的程序执行结束。

```java
public class ThreadDemo {
    public static void main(String[] args) throws Exception {
        Thread thread = new Thread(() -> {
            for (int x = 0; x < 100; x++) {
                System.out.println(Thread.currentThread().getName() + "执行.x=" + x);
            }
        }, "我要的线程");
        thread.start();
        for (int x = 0; x < 100; x++) {
            System.out.println("霸道的main线程 number=" + x);
        }
    }
}
```

这个时候的主线程和子线程都在交替执行着，但是如果说现在你希望主线程独占执行。

那么就可以利用Thread类中的方法强制执行：public final void join()  throws [InterruptedException](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/InterruptedException.html)

```java
public class ThreadDemo {
    public static void main(String[] args) throws Exception {
        Thread mainthread = Thread.currentThread();
        Thread thread = new Thread(() -> {
            for (int x = 0; x < 100; x++) {
                if (x == 3) {
                    try {
                        mainthread.join();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "执行.x=" + x);
            }
        }, "我要的线程");
        thread.start();
        for (int x = 0; x < 100; x++) {
            Thread.sleep(10);
            System.out.println("霸道的main线程 number=" + x);
        }
    }
}
```

在进行线程强制执行的时候一定要获取强制执行对象之后才可以执行join()调用。

#### 线程的礼让

线程的礼让指的是将资源让出去让别的线程先执行

线程的礼让可以使用Thread中提供的方法：

- 礼让：public static void yield()

```java
public class ThreadDemo {
    public static void main(String[] args) throws Exception {
        Thread mainthread = Thread.currentThread();
        Thread thread = new Thread(() -> {
            for (int x = 0; x < 100; x++) {
                if(x%3==0){
                    Thread.yield();
                    System.out.println("线程礼让执行");
                }
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "执行.x=" + x);
            }
        }, "我要的线程");
        thread.start();
        for (int x = 0; x < 100; x++) {
            Thread.sleep(10);
            System.out.println("霸道的main线程 number=" + x);
        }
    }
}
```

礼让执行的时候每一次调用yield()方法都只会礼让一次当前的资源。

#### 线程优先级

从理论上来讲线程的优先级越高越有可能先执行（也越有可能先抢占到资源）。

在Thread类里面针对于优先级的操作提供有如下的两个处理方法：

- 设置优先级：public final void setPriority(int newPriority)
- 获取优先级：public final int getPriority()

在进行优先级定义的时候都是通过int 型的数字来完成的，而对于此数字的选择在Thread类里面就定义有三个常量：

- 最高优先级：public static final int MAX_PRIORITY = 10;
- 中等优先级：public static final int NORM_PRIORITY =5 ;
- 最低优先级：public static final int MIN_PRIORITY =1 ;

主线程是属于中等优先级，而默认创建的线程也是中等优先级。

优先级高的可能先执行而不是绝对先执行。

```java
public class ThreadDemo {
    public static void main(String[] args) throws Exception {
        System.out.println(new Thread().getPriority());
        System.out.println(Thread.currentThread().getPriority());
    }
}
```

### 线程的同步与死锁

#### 同步问题引出

在多线程的处理之中，可以利用 Runnable 描述多个线程操作的资源，而  Thread 描述每一个线程对象，于是当多个线程访问同一资源的时候如果处理不当就会产生数据的错误操作。

下面编写一个简单的卖票程序，将创建若干个线程对象实现卖票的处理操作。

```java
class MyThread implements Runnable {
    private int ticket = 10;    //总票数为10张

    @Override
    public void run() {
        while (true) {
            if (this.ticket > 0) {
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "卖票，ticket=" + this.ticket--);
            } else {
                System.out.println("--------票已卖光--------");
                break;
            }
        }
    }
}

public class ThreadDemo {
    public static void main(String[] args) throws Exception {
        MyThread myThread = new MyThread();
        new Thread(myThread, "票贩子A").start();
        new Thread(myThread, "票贩子B").start();
        new Thread(myThread, "票贩子C").start();
    }
}
```

通过程序的结果可以看出上述程序存在数据同步的问题

![](https://s2.loli.net/2022/05/24/UxP6yS19Wsbnojd.png)

#### 线程同步处理

解决同步问题的关键是锁，指的是当某一个线程执行操作的时候，其它线程外面等待。

同步就是指多个操作在同一个时间段内只能有一个线程进行，其他线程要等待此线程完成之后才可以继续执行。

![](https://s2.loli.net/2022/05/24/YAb3pZKF9Wkh4R1.png)

如果要想在程序之中实现这把锁的功能，就可以使用synchronized关键字来实现，利用此关键字可以定义同步方法或同步代码块，在同步代码块的操作里面的代码只允许一个线程执行。

- 利用同步代码块进行处理：

```java
synchronized(同步对象){
	同步代码操作；
}
```

一般要进行同步对象处理的时候可以采用当前对象 this 进行同步。

```java
class MyThread implements Runnable {
    private int ticket = 10;    //总票数为10张

    @Override
    public void run() {
        while (true) {
            synchronized (this) {
                if (this.ticket > 0) {
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + "卖票，ticket=" + this.ticket--);
                } else {
                    System.out.println("--------票已卖光--------");
                    break;
                }
            }
        }
    }
}

public class ThreadDemo {
    public static void main(String[] args) throws Exception {
        MyThread myThread = new MyThread();
        new Thread(myThread, "票贩子A").start();
        new Thread(myThread, "票贩子B").start();
        new Thread(myThread, "票贩子C").start();
    }
}
```

加入同步处理之后，程序的整体的性能下降了

- 利用同步方法解决

只需要在方法定义上使用 svnchronized 关键字即可。

```java
class MyThread implements Runnable {
    private int ticket = 10;    //总票数为10张

    public synchronized boolean sale() {
        if (this.ticket > 0) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "卖票，ticket=" + this.ticket--);
            return true;
        } else {
            System.out.println("--------票已卖光--------");
            return false;
        }
    }

    @Override
    public void run() {
        while (this.sale()) {
        }
    }
}

public class ThreadDemo {
    public static void main(String[] args) throws Exception {
        MyThread myThread = new MyThread();
        new Thread(myThread, "票贩子A").start();
        new Thread(myThread, "票贩子B").start();
        new Thread(myThread, "票贩子C").start();
    }
}
```

在日后学习 Java 类库的时候会发现，系统中许多的类上使用的同步处理采用的都是同步方法。

#### 线程死锁

死锁是在进行多线程同步的处理之中有可能产生的一种问题，所谓的死锁指的是若干个线程彼此互相等待的状态。

死锁实际上是一种开发中出现的不确定的状态，有的时候代码如果处理不当则会不定期出现死锁，这是属于正常开发中的调试问题。

若干个线程访问同一资源时一定要进行同步处理，而过多的同步会造成死锁。

### ”生产者-消费者“模型

#### 生产者与消费者基本模型

在多线程的开发过程之中最为著名的案例就是生产者与消费者操作，该操作的主要流程如下：

- 生产者负责信息内容的生产
- 每当生产者生产完成一项完整的信息之后消费者要从这里面取走信息
- 如果生产者没有生产者则消费者要等待它生产完成，如果消费者还没有对信息进行消费，则生产者应该等待消费处理完成后再继续进行生产

可以将生产者与消费者定义为两个独立的线程类对象， 既然生产者与消费者是两个独立的线程，那么这两个独立的线程之间就需要有一个数据的保存集中点，那么可以单独定义一个 Message 类实现数据的保存。

![](https://s2.loli.net/2022/05/24/6bvR1jM39d5FWz8.png)

```java
class Producer implements Runnable {
    private Message message;

    public Producer(Message message) {
        this.message = message;
    }

    @Override
    public void run() {
        for (int x = 0; x < 100; x++) {
            if (x % 2 == 0) {
                this.message.setTitle("Tom");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                this.message.setContent("good boy");
            } else {
                this.message.setTitle("Bob");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                this.message.setContent("bad boy");
            }
        }
    }
}

class Consumer implements Runnable {
    private Message message;

    public Consumer(Message message) {
        this.message = message;
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(this.message.getTitle() + " - "+ this.message.getContent());
        }
    }
}

class Message {
    private String title;
    private String content;

    public void setContent(String content) {
        this.content = content;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getContent() {
        return content;
    }

    public String getTitle() {
        return title;
    }
}

public class ThreadDemo {
    public static void main(String[] args) throws Exception {
        Message message=new Message();
        new Thread(new Producer(message)).start();
        new Thread(new Consumer(message)).start();
    }
}
```

通过整个代码的执行你会发现此时有两个主要问题:

- 数据不同步了;

- 生产一个取走一个，但是发现有了重复生产和重复取出问题。

#### 解决同步问题

如果要解决问题，首先解决的就是数据同步的处理问题，如果要想解决数据同步最简单的做法是使用 synchronized 关键字定义同步代码块或同步方法，于是这个时候对于同步的处理就可以直接在 Message 类中完成。

```java
class Message {
    private String title;
    private String content;

    public synchronized void set(String title, String content) {
        this.title = title;
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        this.content = content;
    }

    public synchronized String get() {
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return this.title + " - " + this.content;
    }
}
```

在进行同步处理的时候肯定需要有一个同步的处理对象，那么此时肯定要将同步操作交由 Message 类处理是最合适的。

这个时候发现数据已经可以正常的保持一致了，但是对于重复操作的问题依然存在。

#### 利用Object类解决重复操作

如果说现在要想解决生产者与消费者的问题，那么最好的解决方案就是使用等待与唤醒机制

等待与唤醒的操作机制，主要依靠的是 Object 类中提供的方法处理的:

- 等待：public final void wait() throws [InterruptedException](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/InterruptedException.html)
- 设置等待时间：public final void wait(long timeoutMillis, int nanos) throws [InterruptedException](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/InterruptedException.html)

- 唤醒第一个等待线程：public final void notify()
- 唤醒全部等待线程：public final void notifyAll()

如果此时有若干个等待线程的话，那么 notify() 表示的是唤醒第一个等待的，而其它的线程继续等待.而 notifyAll()表示醒所有等待的线程，哪个线程的优先级高就有可能先执行。

```java
class Message {
    private String title;
    private String content;
    private boolean flag = true;
    //flag = true表示允许生产但是不允许消费；flag = false表示允许消费但是不允许生产

    public synchronized void set(String title, String content) {
        if (this.flag == false) {
            try {
                super.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        this.title = title;
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        this.content = content;
        this.flag = false;
        super.notify();
    }

    public synchronized String get() {
        if (this.flag == true){
            try {
                super.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        try {
            return this.title + " - " + this.content;
        }finally {
            this.flag=true;
            super.notify();
        }
    }
}
```

这种处理形式就是在进行多线程开发过程之中最原始的处理方案，整个的等待、同步唤醒机制都有开发者自行通过原生代码实现控制。

### 多线程深入话题

#### 优雅的停止线程

在多线程操作之中如果要启动多线程肯定使用的是 Thread 类中的 start() 方法，而如果对于多线程需要进行停止处理，Thread 类原本提供有 stop() 方法。但是对于这些方法从 JDK1.2 版本开始就已经将其废除了，而且一直到现在也不再建议出现在你的代码中。除了 stop() 之外还有几个方法也被禁用。

- 停止多线程: public void stop()
- 销毁多线程: public void destroy()
- 挂起线程: public final void suspend() 暂停执行
- 恢复挂起的线程执行: public final void resume()

之所以废除掉这些方法，主要的原因是因为这些方法有可能导致线程的死锁。

所以从 JDK1.2 开始就都不建议使用，如果要想实现线程的停止需要通过一种柔和的方式来进行。

```java
public class ThreadDemo {
    public static boolean flag = true;
    public static void main(String[] args) throws Exception {
        new Thread(() -> {
            long num = 0;
            while (flag) {
                try {
                    Thread.sleep(50);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "正在运行、num" + num++);
            }
        }, "执行线程").start();
        Thread.sleep(200);//运行200毫秒
        flag =false;//停止线程
    }
}
```

#### 后台守护线程

在多线程里面可以进行守护线程的定义，也就是说如果现在主线程的程序或者其它的线程还在执行的时候，守护线程将一直存在，并且运行在后台状态。

在 Thread 类里面提供有如下的守护线程的操作方法:

- 设置为守护线程: public final void setDaemon(boolean on); 
-  判断是否为守护线程:public final boolean isDaemon();

```java
public class ThreadDemo {
    public static boolean flag = true;

    public static void main(String[] args) throws Exception {
        Thread userthread = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "正在运行、i" +i);
            }
        }, "用户线程");
        Thread daemonthread = new Thread(() -> {
            for (int i = 0; i < Integer.MAX_VALUE; i++) {
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "正在运行、i" +i);
            }
        }, "守护线程");
        daemonthread.setDaemon(true);
        userthread.start();
        daemonthread.start();
    }
}
```

可以发现所有的守护线程都是围绕在用户线程的周围，如果程序执行完毕了，守护线程也就消失了在整个的 JVM 里面最大的守护线程就是 GC 线程。

程序执行中 GC 线程会一直存在，如果程序执行完毕，GC 线程也将消失。

#### volatile关键字

在多线程的定义之中，volatile 关键字主要是在属性定义上使用的，表示此属性为直接数据操作，而不进行副本的拷贝处理。

在一些书上就将其错误的理解为同步属性。

在正常进行变量处理的时候往往会经历如下的几个步骤：

- 获取变量原有的数据内容副本;
- 利用副本为变量进行数学计算;
- 将计算后的变量，保存到原始空间之中;

如果一个属性上追加了 volatile 关键字，表示的就是不使用副本而是直接操作原始变量，相当于节约了拷贝副本，重新保存的步骤

![](https://s2.loli.net/2022/05/24/on4pm2UryWQfDCz.png)

```java
class MyThread implements Runnable {
    private volatile int ticket = 10;    //总票数为10张

    public synchronized boolean sale() {
        if (this.ticket > 0) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "卖票，ticket=" + this.ticket--);
            return true;
        } else {
            System.out.println("--------票已卖光--------");
            return false;
        }
    }

    @Override
    public void run() {
        while (this.sale()) {
        }
    }
}

public class ThreadDemo {
    public static void main(String[] args) throws Exception {
        MyThread myThread = new MyThread();
        new Thread(myThread, "票贩子A").start();
        new Thread(myThread, "票贩子B").start();
        new Thread(myThread, "票贩子C").start();
    }
}
```

volatile与synchronized的区别

- volatile主要在属性上使用，而synchronized是在代码块与方法上使用

- volatile 无法描述同步的处理，它只是一种直接内存的处理，避免了副本的操作，而 synchronized 是实现同步的

### 多线程综合案例

#### 数字加减

设计 4 个线程对象，两个线程执行减操作，两个线程执行加操作

```java
class Resource {
    private int num = 0;//要进行加减操作的数据
    private boolean flag = true;//加减的切换

    //flag=true表示可以进行加法操作，无法进行减法操作；flag=false表示可以进行减法操作，无法进行加法操作
    public synchronized void add() throws Exception {
        if (flag == false) {
            super.wait();
        }
        Thread.sleep(100);
        this.num++;
        System.out.println("加法操作：num=" + this.num);
        this.flag = false;
        super.notifyAll();
    }

    public synchronized void sub() throws Exception {
        if (flag == true) {
            super.wait();
        }
        Thread.sleep(100);
        this.num--;
        System.out.println("减法操作：num=" + this.num);
        this.flag = true;
        super.notifyAll();
    }
}

class AddThread implements Runnable {
    private Resource resource;

    public AddThread(Resource resource) {
        this.resource = resource;
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            try {
                this.resource.add();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}

class SubThread implements Runnable {
    private Resource resource;

    public SubThread(Resource resource) {
        this.resource = resource;
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            try {
                this.resource.sub();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}

public class ThreadDemo {
    public static void main(String[] args) throws Exception {
        Resource resource = new Resource();
        AddThread addThread = new AddThread(resource);
        new Thread(addThread, "加法线程A").start();
        new Thread(addThread, "加法线程B").start();
        SubThread subThread = new SubThread(resource);
        new Thread(subThread, "减法线程C").start();
        new Thread(subThread, "减法线程D").start();
    }
}
```

#### 生产电脑

设计一个生产电脑和搬运电脑类，要求生产出一台电脑就搬走一台电脑，如果没有新的电脑生产出来，则搬运工要等待新电脑产出，如果生产出的电脑没有搬走，则要等待电脑搬走之后再生产，并统计出生产的电脑数量。

在本程序之中实现的就是一个标准的生产者与消费者的处理模型

```java
class Computer {
    private static int count = 0;//表示生产的个数
    private String name;
    private double price;

    public Computer(String name, double price) {
        this.name = name;
        this.price = price;
        count++;
    }

    @Override
    public String toString() {
        return "Computer{" +
                "count='" + count + '\'' +
                ", name='" + name + '\'' +
                ", price=" + price +
                '}';
    }
}

class Resource {
    private Computer computer;

    public synchronized void make() throws Exception {
        if (this.computer != null) {
            super.wait();
        }
        Thread.sleep(100);
        this.computer = new Computer("组装电脑", 1.1);
        super.notifyAll();
    }

    public synchronized void get() throws Exception {
        if (this.computer == null) {
            super.wait();
        }
        Thread.sleep(10);
        System.out.println(this.computer);
        this.computer = null;
        super.notifyAll();
    }
}

class Producer implements Runnable {
    private Resource resource;

    public Producer(Resource resource) {
        this.resource = resource;
    }

    @Override
    public void run() {
        for (int x = 0; x < 50; x++) {
            try {
                this.resource.make();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}

class Consumer implements Runnable {
    private Resource resource;

    public Consumer(Resource resource) {
        this.resource = resource;
    }

    @Override
    public void run() {
        for (int x = 0; x < 50; x++) {
            try {
                this.resource.get();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}

public class ThreadDemo {
    public static void main(String[] args) throws Exception {
        Resource resource = new Resource();
        new Thread(new Producer(resource)).start();
        new Thread(new Consumer(resource)).start();
    }
}
```

#### 竞争抢答

实现一个竞拍抢答程序:要求设置三个抢答者美三个线程)，而后同时发出抢答指令，抢答成功者给出成功提示，抢答未成功者给出失败提示。

对于这一个多线程的操作由于里面需要牵扯到数据的返回问题，那么现在最好使用的 Callable.是比较方便的处理形式

```java
class MyThread implements Callable<String> {
    private boolean flag = false;

    @Override
    public String call() throws Exception {
        synchronized (this) {
            if (this.flag == false) {
                flag = true;
                return Thread.currentThread().getName() + "抢答成功";
            } else {
                return Thread.currentThread().getName() + "抢答失败";
            }
        }
    }
}

public class ThreadDemo {
    public static void main(String[] args) throws Exception {
        MyThread myThread = new MyThread();
        FutureTask<String> futureTask = new FutureTask<String>(myThread);
        new Thread(futureTask, "竞赛者A").start();
        new Thread(futureTask, "竞赛者B").start();
        new Thread(futureTask, "竞赛者C").start();
        System.out.println(futureTask.get());
    }
}
```

### Java常用类库

#### Java基础类库

##### StringBuffer类

[StringBuffer (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/StringBuffer.html#())

String类 是在所有项目开发之中一定会使用到的一个功能类，并且这个类拥有如下的特点：

- 每一个字符串的常量都属于一个 String 类的匿名对象，并且不可更改
- String 有两个常量池:静态常量池、运行时常量池
- String 类对象实例化建议使用直接赋值的形式完成，这样可以直接将对象保护在对象池中以方便下次重用

String 类的弊端：

- 内容不允许修改

```java
public class JavaAPIDemo {
    public static void main(String[] args) {
        String str = "Hello";
        change(str);
        System.out.println(str);
    }

    public static void change(String temp) {
        temp += "World!";
    }
}
```

为了解决此问题，专门提供有一个 StringBuffer 类可以实现字符串的内容修改

StringBuffer 并不像 String 类那样拥有两种对象实例化方式，StringBuffer 必须像普通类对象那样，首先进行对象的实例化，而后才可以调用方法执行处

- 构造方法：public StringBuffer()；public StringBuffer([String](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html) str)
- 数据追加：public [StringBuffer](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/StringBuffer.html) append([Object](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html) obj)

```java
public class JavaAPIDemo {
    public static void main(String[] args) {
        StringBuffer stringBuffer = new StringBuffer("Hello");
        change(stringBuffer);
        System.out.println(stringBuffer);
    }

    public static void change(StringBuffer temp) {
        temp.append(" World!");
    }
}
```

大部分情况下不是会出现改变字符串的内容，这种改变指的并不是针对于静态常量池的改变。

```java
public class JavaAPIDemo {
    public static void main(String[] args) {
        String str1 = "www.zhuweihao.com";
        String str2 = "www." + "zhuweihao" + ".com";
        System.out.println(str1 == str2);
    }
}
```

这个时候的 strB 对象的内容并不算是改变，所有的“+”在编译之后都变为了 StringBuffer 中的 append() 方法，并且在程序之中 StringBuffer 与 String 类对象之间可以直接相互转换：

- String 类对象变为 StringBuffer 可以依靠 StringBuffer 类的构造方法或者使用append 方法
- 所有类对象都可以通过 toString() 方法将其变为 String 类型

在 StringBuffer 类里面除了可以支持有字符串内容的修改之外，实际上也提供有了一些 String 类所不具备的方法。

- 插入数据

- 删除指定范围数据

- 内容反转

```java
public class JavaAPIDemo {
    public static void main(String[] args) {
        StringBuffer stringBuffer = new StringBuffer("www..com");
        stringBuffer.insert(4, "zhuweihao");
        System.out.println(stringBuffer);
        stringBuffer.delete(4, 13);
        System.out.println(stringBuffer.reverse());
    }
}
```

String、StringBuffer、StringBuilder 的区别：

- String 类是字符串的首选类型，其最大的特点是内容不允许修改；
- StringBuffer '与 StringBuilder 类的内容允许修改；
- StringBuffer 是在 JDK 1.0 的时候提供的，属于线程安全的操作，而 StringBuilder 是在 JDK 1.5 之后提供的，不属于线程安全的操作。

##### CharSequence接口

[CharSequence (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/CharSequence.html)

CharSequence 是一个描述字符串结构的接口，在这个接口里面一般发现有三种常用子类：String 类、StringBuffer 类、StringBuikder 类。

![](https://s2.loli.net/2022/05/25/tjyHhE45KxIsPLg.png)

所以字符串加入公共的描述类型，就是 CharSequence ，只要有字符串，就可以被 CharSequence 接口识别化，所有的字符串都可以这样接收。

```java
CharSequence str="ww.mldn.cn" //子类实例向父接口转型。
```

CharSequence 本身是一个接口，在该接口之中也定义有如下操作方法：

- 获取只能够索引字符：char charAt(int index)
- 获取字符串的长度：int length()
- 截取部分字符串：[CharSequence](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/CharSequence.html) subSequence(int start, int end)

所以以后只要看见了 CharSequence 描述的就是一个字符串。

##### AutoCloseable接口

[AutoCloseable (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/AutoCloseable.html)

AutoCloseable 主要是用于日后进行资源开发的处理上，以实现资源的自动关闭（释放资源）。

例如：在以后进行文件、网络、数据库开发的过程之中由于服务器的资源有限，所以使用之后一定要关闭资源，这样才可以被更多的使用者所使用。

```java
interface IMessage {
    public void send();//消息发送
}

class NetMessage implements IMessage {//实现消息的处理机制
    private String msg;

    public NetMessage(String msg) {
        this.msg = msg;
    }

    @Override
    public void send() {
        System.out.println("【发送消息】" + this.msg);
    }

    public boolean open() {//获取资源连接
        System.out.println("【OPEN】获取消息发送连接资源");
        return true;
    }

    public void close() {
        System.out.println("【CLOSE】关闭消息发送通道");
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) {
        NetMessage netMessage=new NetMessage("zhuweihao");
        if(netMessage.open()){//是否打开了连接
            netMessage.send();//消息发送
            netMessage.close();//关闭连接
        }
    }
}
```

此时实现了一个模拟代码的处理流程，但有个问题，既然所有的资源完成处理之后都必须进行关闭操作，那么能否实现一种自动关闭的功能呢？

在这样的要求下，推出了 AutoCloseable  访问接口，这个接口是在 JDK1.7  的时候提供的，并且该接口只提供有一个方法。

关闭方法: void close()  throws Exception

![](https://s2.loli.net/2022/05/25/4FAeBWswVoY5RTL.png)

在整个的过程中，只有结合了 AutoCloseable ，整个程序才能实现自动的Close 调用，这种操作形式是在 JDK1.7 之后新增的处理，在以后的章节之中会接触到资源的关闭问题，往往都会见到 AutoCloseable 接口的使用。

这个接口要和异常捆绑在一起明确使用才能正确完成。

```java
interface IMessage extends AutoCloseable{
    public void send();//消息发送
}

class NetMessage implements IMessage{//实现消息的处理机制
    private String msg;

    public NetMessage(String msg) {
        this.msg = msg;
    }

    @Override
    public void send() {
        if (this.open()){
            System.out.println("【发送消息】" + this.msg);
        }
    }

    public boolean open() {//获取资源连接
        System.out.println("【OPEN】获取消息发送连接资源");
        return true;
    }

    @Override
    public void close() throws Exception{
        System.out.println("【CLOSE】关闭消息发送通道");
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception{
        try (IMessage netMessage=new NetMessage("zhuweihao")){
            netMessage.send();
        }catch (Exception e){}
    }
}
```

##### Runtime类

[Runtime (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Runtime.html)

Runtime 描述的是运行时的状态，也就是说在整个的 JVM 之中，Runtime 类是唯一一个与 JVM 运行状态有关的类，并且都会默认提供有一一个该类的实例化对象。

由于在最每一个 JVM 进程里面只允许提供有一个 Runtime 类的对象，所以这个类的构造方法被默认私有化了，那么就证明该类使用的是单例设计模式，并且单例设计模式一定会提供有一个 static 方法获取本类。

![](https://s2.loli.net/2022/05/25/m4xVKQNHXzlMWfR.png)

 

由于 Runtime 类属于单例设计模式，如果要想获取实例化对象，那么就可以依靠类中的 getRuntime() 方法完成：

- 获取实例化对象: 

  ```java
  public static Runtime getRuntime()
  ```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Runtime runtime = Runtime.getRuntime();
        System.out.println(runtime.availableProcessors());
    }
}
```

在 Runtime 类里面还提供有以下四个重要的操作方法：

- 获取最大可用内存空间：public long maxMemory()；默认的配置为本机系统的 4分之 1
- 获取可用内存空间：public long totalMemory()；默认的配置为本机系统的 64分之 1
- 获取空闲内存空间：public long freeMemory()
- 手工进行GC处理：public void gc()

GC（Garbage Collector) 垃圾收集器，是可以由系统自动调用的垃圾释放功能，或者使用 Runtime 类的 gc（）手工调用。

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Runtime runtime = Runtime.getRuntime();
        System.out.println(runtime.availableProcessors());
        System.out.println("1.Max_Memory" + runtime.maxMemory());
        System.out.println("1.Total_Memory" + runtime.totalMemory());
        System.out.println("1.Free_Memory" + runtime.freeMemory());
        String str = "";
        for (int i = 0; i < 10000; i++) {//产生大量垃圾空间
            str += i;
        }
        System.out.println("2.Max_Memory" + runtime.maxMemory());
        System.out.println("2.Total_Memory" + runtime.totalMemory());
        System.out.println("2.Free_Memory" + runtime.freeMemory());
        Thread.sleep(2000);
        runtime.gc();
        System.out.println("3.Max_Memory" + runtime.maxMemory());
        System.out.println("3.Total_Memory" + runtime.totalMemory());
        System.out.println("3.Free_Memory" + runtime.freeMemory());
    }
}
```

##### System类

[System (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/System.html)

System 类是一直陪伴着我们学习的程序类，之前使用的系统输出采用的就是System 类中的方法，在 System 类里面也定义有一些其它的处理方法。

- 数组拷贝：public static void arraycopy([Object](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html) src, int srcPos, [Object](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html) dest, int destPos, int length)
- 获取当前的日期时间数值：public static long currentTimeMillis()
- 进行垃圾回收：public static void gc()

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        long start = System.currentTimeMillis();
        Runtime runtime = Runtime.getRuntime();
        String str = "";
        for (int i = 0; i < 10000; i++) {//产生大量垃圾空间
            str += i;
        }
        long end = System.currentTimeMillis();
        System.out.println("操作耗时：" + (end - start));
    }
}
```

在 System 类里面会发现也提供有一个 gc()方法，但是这个 gc() 方法并不是重新定义的新方法，而是继续调用了 Runtime 类中的 gc() 操作

```java
public static void gc() {
        Runtime.getRuntime().gc();
    }
```

##### Cleaner类

[Cleaner (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/ref/Cleaner.html)

Cleaner 是在 JDK 1.9 之后提供的一个对象清理操作，其主要的功能是进行 finialize() 方法的替代。

在 C++ 语言里面有两种特殊的函数：构造函数、析构函数(对象手工回收)，在 Java里面所有的垃圾空间都是通过 GC 自动回收的，所以很多情况下是不需要使用这类析构函数的，也正是因为如此，所以 Java 并没有提供这方面支持。

但是 Java 本身依然提供了给用户收尾的操作，每一个实例化对象在回收之前至少给它一个喘息的机会，最初实现对象收尾处理的方法是 Object 类中所提供的 finalize (方法，这个方法的定义如下：

```java
@Deprecated(since="9")
protected void finalize()
                 throws Throwable
```

从 JDK1.9 开始建议开发者使用 AutoCloseable 或者使用 java.lang.ref.Cleaner 类进行回收处理( Cleaner 也支持有 AutoCloseable 处理)。

```java
class Member implements Runnable {
    public Member() {
        System.out.println("【构造】");
    }

    @Override
    public void run() {
        System.out.println("【回收】");
    }
}

class MemberCleaning implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();
    private Member member;
    private Cleaner.Cleanable cleanable;

    public MemberCleaning() {
        this.member = new Member();
        this.cleanable = cleaner.register(this, this.member);
    }

    @Override
    public void close() throws Exception {
        this.cleanable.clean();//启动多线程
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        try (MemberCleaning memberCleaning = new MemberCleaning()) {
            //相关代码
        } catch (Exception e) {
        }
    }
}
```

##### 对象克隆

所谓的对象克隆指的就是对象的复制，而且属于全新的复制。即:使用已有对象内容创建一个新的对象，如果要想进行对象克隆需要使用到Object类中提供的clone()方法：

```java
protected Object clone()
                throws CloneNotSupportedException
```

所有的类都会继承 Object 父类，所以所有的类都一定会有 clone() 方法，但是并不是所有的类都希望被克隆。

所以如果要想实现对象克隆，那么对象所在的类需要实现-一个 Cloneable 接口，此接口并没有任何的方法提供，是因为它描述的是一种能力。

```java
class Member implements Cloneable {
    private String name;
    private int age;

    public Member(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Member{" + super.toString() +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();//调用父类中提供的clone()方法
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Member memberA = new Member("张三", 20);
        Member memberB = (Member) memberA.clone();
        System.out.println(memberA);
        System.out.println(memberB);
    }
}
```

#### 数字操作类

##### Math类

[Math (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Math.html)

程序就是一个数学的处理过程，所以在 Java 语言本身也提供有相应的数字处理的类库支持。

Math 类的主要功能是进行数学计算的操作类，提供有基础的计算公式，这个类的构造方法被私有化了，而且该类之中提供的所有方法都是 static 型的方法，即：这些方法都可以通过类名称直接调用。

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        System.out.println(Math.abs(-1.2));
        System.out.println(Math.max(3.4, 4.3));
        System.out.println(Math.log10(10));
        System.out.println(Math.round(15.1));
        System.out.println(Math.pow(2, 4));
    }
}
```

虽然在 Math 类里面提供有四舍五入的处理方法,但是这个四舍五入字在进行处理的时候是直接将小数点后的所有位进行处理了，这样肯定不方便，最好的做法是可以实现指定位数的保留。

```java
class MathUtil {
    private MathUtil() {
    }

    /**
     * 实现数据的四舍五入操作
     *
     * @param num   要进行四舍五入操作的数字
     * @param sacle 四舍五入保留的小数位数
     * @return 四舍五入最后的结果
     */
    public static double round(double num, int sacle) {
        return Math.round(num * Math.pow(10, sacle)) / Math.pow(10, sacle);
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        System.out.println(MathUtil.round(19.3452, 2));
    }
}
```

Math 类里面提供的基本上都是基础的数学公式，需要的时候需要自己重新整合。

##### Random类

[Random (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Random.html#nextInt())

java.util.Random 类的主要功能是产生随机数，这个类主要是依靠内部提供的方法来完成

- 产生一个不大于边界的随机非负整数：

  ```java
  public int nextInt()
  ```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Random random = new Random();
        for (int i = 0; i < 10; i++) {
            System.out.print(random.nextInt(10) + "、 ");
        }
    }
}
```

##### 大数字处理类

[java.math (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/math/package-summary.html)

[BigInteger (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/math/BigInteger.html)

[BigDecimal (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/math/BigDecimal.html)

大数字处理类可以实现海量数字的计算

![](https://s2.loli.net/2022/05/25/Cu3BcXMe524GIF6.png)

BigInteger 类构造：

```java
public BigInteger(String val);
```

BigDecimal 类构造：

```java
public BigDecimal(String val);
```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        BigInteger bigIntegerA = new BigInteger("12345678901234567890");
        BigInteger bigIntegerB = new BigInteger("1234567890");
        System.out.println("加法" + bigIntegerA.add(bigIntegerB));
        System.out.println("减法" + bigIntegerA.subtract(bigIntegerB));
        System.out.println("乘法" + bigIntegerA.multiply(bigIntegerB));
        System.out.println("除法" + bigIntegerA.divide(bigIntegerB));
    }
}
```

需要注意的是，虽然提供有大数字类处理方法，但仍然需要考虑性能问题。

如果在计算时没有超过基本数据类型所包含的位数强烈不建议使用大数字处理类方法，因为性能很低。

#### 日期操作类

##### Date类

[Date (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Date.html)

在 Java 里面提供有一个 java.util.Date 的类，这个类如果直接实例化就可以获取当前的日期时间

Date 类中只是对 long 数据的一种包装。

Date 类中提供日期与 long 数据类型之间转换的方法：

- 将long转为Date：public Date(long date)
- 将Date转为long：public long getTime()

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Date date = new Date();
        long current = date.getTime();
        System.out.println(current);
        current += 864000 * 1000;//10天的秒数
        System.out.println(new Date(current));//long转为Date
    }
}
```

long 之中可以保存毫秒的数据级，这样方便程序处理。

##### SimpleDateFormat 日期处理类

[SimpleDateFormat (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/text/SimpleDateFormat.html#(java.lang.String))

虽然 date 可以获取当前的日期时间，但是默认情况下 Date 类输出的日期结构并不能够被国人所习惯，那么现在就需要对显示的格式进行格式化处理，为了可以格式化日期，在 java.text 包中提供有 SimpleDateFormat 程序类。

![](https://s2.loli.net/2022/05/25/y274vrNYDxpC3Sm.png)

在该类中提供有如下的方法：

- 【DateFormat继承】将日期格式化：

  ```java
  public final String format(Date date)
  ```

- 【DateFormat继承】将字符串转为日期：

  ```java
  public Date parse(String source)
             throws ParseException
  ```

- 构造方法：

  ```java
  public SimpleDateFormat(String pattern)
  ```

  - 日期格式：年（yyyy）、月（MM）、日（dd）、时（HH）、分（mm）、秒（ss）、毫秒（SSS）
  - ![image-20220525171256055](https://s2.loli.net/2022/05/25/eD34sRgC6BKfNZp.png)

```java
public class JavaAPIDemo {//日期格式化
    public static void main(String[] args) throws Exception {
        Date date = new Date();
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS");
        String str = simpleDateFormat.format(date);
        System.out.println(str);
    }
}
```

除了可以将日期格式化为字符，也可以实现字符串转化为日期格式显示

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String birthday = "2052-05-25 17:15:59.567";
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS");
        Date date=simpleDateFormat.parse(birthday);
        System.out.println(date);
    }
}
```

如果在进行字符串定义的时候，所使用的日期时间数字超过了指定的合理范围，则会自动进行进位处理。

#### 正则表达式

##### 认识正则表达式

通过之前的学习可以发现，String 一个非常万能的类型，因为 String 不仅仅可以支持有各种字符串的处理操作，也支持有向各个数据类型的转换功能，所以在项目的开发之中，只要是用户输入的信息基本上都可以用 String 表示

在向其他类型转换的时候，为了保证转换的正确性，往往需要对其进行一些复杂的验证处理，这种情况下如果只是单纯的依靠 String 类中的方法是非常麻烦的。



现在假设有一个字符串要求你判断字符串是否由数字组成，如果由数字所组成则将变为数字进行乘法计算。

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String str = "123";
        if (isNumber(str)) {
            int num = Integer.parseInt(str);
            System.out.println(num * 2);
        }
    }

    public static boolean isNumber(String str) {
        char[] data = str.toCharArray();
        for (int i = 0; i < data.length; i++) {
            if (data[i] > '9' || data[i] < '0') {
                return false;
            }
        }
        return true;
    }
}
```

实际上这种验证的功能是非常简单的，但是这如此简单的功能却需要开发者编写大量程序逻辑代码那么如果是更加复杂的验证呢？那么在这样的情况下，对于验证来讲最好的做法就是利用正则表达式来完成。

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String str = "123";
        if (str.matches("\\d+")) {
            int num = Integer.parseInt(str);
            System.out.println(num * 2);
        }
    }
}
```

正则表达式最早是从 Perl 语言里面发展而来的，而后在 JDK1.4 以前如果需要使用到正则表达式的相关定义则需要单独引入其他的*. jar 文件，但是从 JDK1.4 之后，正则已经默认被 JDK 所支持，并且提供有 java.util.regex 开发包，同时针对于 String 类也提出进行了一些修改，使其可以有方法之间支持正则处理。

使用正则最大特点在于方便进行验证处理，以及方便进行复杂字符串的修改处理。

##### 常用正则标记（背）

[Pattern (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/regex/Pattern.html)

从JDK1.4 开始提供有 java.util regex 开发包，里边定义这个包里面提供有一-个Pattern 程序类，在这个程序类里面定义有所有支持的正则标记。

- 【单个】字符匹配

  - 任意字符

    ```java
    public class JavaAPIDemo {
        public static void main(String[] args) throws Exception {
            String str = "a";//要判断的数据
            String regex = "a";//正则表达式
            System.out.println(str.matches(regex));//"a"输出true；”aa“、"b"输出false
        }
    }
    ```

  - "\\\\"：匹配"\\"

  - "\n"：匹配换行

  - "\t"：匹配制表符

- 【单个】字符集（可以从集合中任选一个字符）

  - [abc]：可能是a、b、c之中的任意一个

    ```java
    public class JavaAPIDemo {
        public static void main(String[] args) throws Exception {
            String str = "a";//要判断的数据
            String regex = "[abc]";//正则表达式
            System.out.println(str.matches(regex));
        }
    }
    ```

  - [^abc]：表示不是a、b、c中的任意一个

  - [a-zA-Z]：表示由一个任意字母所组成，不区分大小写

  - [0-9]：表示由一位数字所组成

- 【单个】简化字符集

  - .：表示任意的一个字符

    ```java
    public class JavaAPIDemo {
        public static void main(String[] args) throws Exception {
            String str = "-";//要判断的数据
            String regex = ".";//正则表达式
            System.out.println(str.matches(regex));
        }
    }
    ```

  - \d：等价于”[0-9]“

  - \D：等价于`[^0-9]`范围

  - \s：匹配任意的一位空格，可能是空格、换行、制表符

    ```java
    public class JavaAPIDemo {
        public static void main(String[] args) throws Exception {
            String str = "a\n";//要判断的数据
            String regex = "\\D\\s";//正则表达式
            System.out.println(str.matches(regex));
        }
    }
    ```

  - \S：匹配任意的非空格数据

  - \w：匹配字母、 数字、下划线， 等价于“[a-A-Z_0-9]”

  - \W： 匹配非字母、数字、下划线，等价于 `[^\w]`

- 边界匹配

  - ^：匹配边界开始
  - $：匹配边界结束

- 数量表示，默认情况下只有添加上了数量单位才可以匹配多为字符

  - 表达式？：该正则可以出现0次或1次

  - 表达式*：该正则可以出现0次、1次或多次

  - 表达式+：该正则可以出现一次或多次

    ```java
    public class JavaAPIDemo {
        public static void main(String[] args) throws Exception {
            String str = "aabc";//要判断的数据
            String regex = "\\w+";//正则表达式
            System.out.println(str.matches(regex));
        }
    }
    ```

  - 表达式{n}：表达式的长度正好为 n 

  - 表达式{n,}：表达式的长度为 n 以上

  - 表达式{n,m}：表达式的长度为 n~m 

    ```java
    public class JavaAPIDemo {
        public static void main(String[] args) throws Exception {
            String str = "aabc";//要判断的数据
            String regex = "\\w{5,7}";//正则表达式
            System.out.println(str.matches(regex));
        }
    }
    ```

- 逻辑表达式：可以连接多个正则

  - 表达式X表达式Y：X 表达式之后紧跟上 Y 表达式;
  - 表达式X|表达式Y：有一个表达式满足即可;
  - (表达式)：为表达式设置一个整体描述， 可以为整体描述设置数量单位

##### String类对正则的支持

在进行正则表达式大部分处理的情况下都会基于 String 类来完成，并且在 String 类里面提供有如下与正则有关的操作方法:

- 将指定字符串进行正则判断

  ```java
  public boolean matches(String regex)
  ```

- 替换全部

  ```java
  public String replaceAll(String regex, String replacement)
  ```

- 替换首个

  ```java
  public String replaceFirst(String regex, String replacement)
  ```

- 正则拆分

  ```java
  public String[] split(String regex)
  public String[] split(String regex, int limit)
  ```

![image-20220526105737491](https://s2.loli.net/2022/05/26/UAo9DwY5pigMGLV.png)

下面通过一些具体的范例来对正则的使用进行说明。

```java
//实现字符串替换，删除字母数字以外的所有字符
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String str = "^*(&gu465&*(67g(*&G*&T5765G*TY*%F*";
        String regex = "[^a-zA-Z_0-9]+";
        System.out.println(str.replaceAll(regex, ""));
    }
}
//实现字符串的拆分
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String str = "^*(&gu465&*(67g(*&G*&T5765G*TY*%F*";
        String regex = "\\d+";
        String[] result = str.split(regex);
        for (int i = 0; i < result.length; i++) {
            System.out.println(result[i] + "、");
        }
    }
}
```

在正则处理的时候对于拆分与替换的操作相对容易一些， 但是比较麻烦的是数据验证部分。

```java
//判断一个数据是否为实数，如果是小数则将其变为double类型
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String str = "100.32";
        String regex = "\\d+(\\.\\d+)?";
        System.out.println(str.matches(regex));
    }
}
//判断一个字符串是否由日期所组成，如果是由日期所组成则将其转为Date类型。
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String str = "1931-10-23";
        String regex = "\\d{4}-\\d{2}-\\d{2}";
        if (str.matches(regex)) {
            System.out.println(new SimpleDateFormat("yyyy-MM-dd").parse(str));
        }
    }
}
```

需要注意的是，正则表达式无法对里面的内容进行判断，只能够对格式进行判断处理。

```java
//判断给定的电话号码是否正确
//51283346、01051283346、(010)-51283346
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String str = "(010)-51283346";
        String regex = "((\\d{3,4})|(\\(\\d{3,4}\\)-))?\\d{7,8}";
        System.out.println(str.matches(regex));
    }
}
/**
 * 验证 email格式
 * email的用户名可以由字母、数字、_所组成; （不应该使用_开头）
 * email的域名可以由字母、数字、_、-所组成;
 * 域名的后缀必须是:.cn、 .com、 .net、 .com.cn、 .gov;
 */
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String str = "zhuweihao18@gmail.com";
        String regex = "[a-zA-Z0-9]\\w+@\\w+\\.(cn|com|gov|net)";
        System.out.println(str.matches(regex));
    }
}
```

##### java.util.regex开发包

[java.util.regex (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/regex/package-summary.html)

虽然在大部分的情况下都可以以利用 String 类实现正则的操作，但是也有一 些情况下需要使用到 java.util.regex 开发包中提供的正则处理类。

在这个包里面一共定义有两个类: 

- Pattern (正则表达式编译)

  - pattern 类提供有正则表达式的编译处理支持

    ```java
    public static Pattern compile(String regex)
    ```

  - 同时也提供有字符串的拆分操作

    ```java
    public String[] split(CharSequence input)
    ```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String str = "^*(&gu465&*(67g(*&G*&T5765G*TY*%F*";
        String regex = "[^a-zA-Z]+";
        Pattern pattern = Pattern.compile(regex);
        String[] result = pattern.split(str);
        for (int i = 0; i < result.length; i++) {
            System.out.println(result[i] + "、 ");
        }
    }
}
```

- Matcher (匹配)，实现了正则匹配的处理类，这个类的对象实例化依靠Pttern类完成

  - Paterm 类提供的方法:

    ```java
    public Matcher matcher(CharSequence input)
    ```

  - 当获取了 Matcher 类的对象之后就可以利用该类中的方法进行如下操作：

    - 正则匹配

      ```java
      public boolean matches()
      ```

    - 字符串替换

      ```java
      public String replaceAll(String replacement)
      ```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String str = "1012345sdfg345&*&……HIU、";
        String regex = "\\D+";
        Pattern pattern = Pattern.compile(regex);
        Matcher matcher = pattern.matcher(str);
        System.out.println(matcher.replaceAll(""));
    }
}
```

如果纯粹是以拆分，替换，匹配三种操作为例根本用不到 java.util.regex 开发包，只依靠 String 类就都可以实现了，但是 mather 类里面提供有一种分组的功能，而这种分组的功能是 String 不具备的。

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        //要求取出”#{内容}“标记中的所有内容
        String str = "INSERT INTO dept(deptno,dname,loc) VALUES (#{deptno},#{dname},#{loc})";
        String regex = "#\\{\\w+\\}";
        Pattern pattern = Pattern.compile(regex);
        Matcher matcher = pattern.matcher(str);
        while (matcher.find()) {//是否由匹配成功的内容
            System.out.println(matcher.group(0).replaceAll("#|\\{|\\}", ""));
        }
    }
}
```



#### 国际化程序实现

所谓的国际化的程序指的是同一个程序代码可以根据不同的语言描述，但是程序处理的核心业务是相同的。

##### 国际化程序实现原理

现在假设有一款世界都认可的企业管理平台，那么这个企业的老板决定将这个产品推广到世界各个大大型上市公司，于是这些公司可能来自于：中国、美国、德国，那么在这样的情况下，如果要想实现国际化的程序开发，那么要解决两个问题：

- 如何可以定义保存文字的文件信息
- 如何可以根据不同的区域语言的编码读取指定的资源信息

![](https://s2.loli.net/2022/05/26/ZMAq7uHcICGeXO4.png)

##### Locale类

[Locale (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Locale.html)

通过分析可以发现，如果要想实现国际化，那么首先需要解决的就是不同国家用户的区域和语言的编码问题，而在 java.util包里面提供有一个专门描述区域和语言编码的类:Locale。

 可以使用 Locale类中的两个构造方法进行实例化。

```java
public Locale(String language)
public Locale(String language, String country)
```

此时需要的是国家和语言的代码，而中文的代码：zhCN、美国英语的代码：en _US，

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Locale locale = new Locale("zh", "CN");//中文环境
        System.out.println(locale);
    }
}
```

如果说现在要想自动获得当前的运行环境，那么现在就可以利用 Locale 类本身默认的方式进行实例化

- 读取本地默认环境

  ```java
  public static Locale getDefault()
  public class JavaAPIDemo {
      public static void main(String[] args) throws Exception {
          Locale locale = Locale.getDefault();
          System.out.println(locale);
      }
  }
  ```

  在实际的开发过程之中，很多人可能并不关心国家和语言的编码，所以为了简化开发，Locale 也将世界上一些比较著名的国家的编码设置为了常量。

  ```JAVA
  public class JavaAPIDemo {
      public static void main(String[] args) throws Exception {
          Locale locale = Locale.CHINA;
          System.out.println(locale);
      }
  }
  ```

  使用常量的优势在于可以避免一些区城编码信息的繁琐。

##### ResouBundle读取资源文件

现在已经准备好资源文件，那么随后就需要进行资源文件的读取操作了，而读取资源文件主要依靠的是 java.util.ResourceBundle类完成

```java
public abstract class ResourceBundle extends Object
```

ResoufcBundle 是一个抽象类，如果说现在要想进行此类对象的实例化可以直接利用该类中提供的一个静态方法完成

```java
//baseName:描述的是资源文件的名称，但是没有后缀
public static final ResourceBundle getBundle(String baseName)
//根据key读取资源内容
public final String getString(String key);
```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        ResourceBundle resourceBundle = ResourceBundle.getBundle("Messages");
        String val = resourceBundle.getString("info");
        System.out.println(val);
    }
}
```

如果资源没有放在包里面，则直接编写资源名称即可

在进行资源读取的时候数据的key一定要存在，如果不存在则会出现异常信息。

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        ResourceBundle resourceBundle = ResourceBundle.getBundle("Messages",Locale.US);
        String val = resourceBundle.getString("info");
        System.out.println(val);
    }
}
```



#### 开发支持类库

##### UUID类

UUID 是一种生成无重复字符串的一种程序类， 这种程序类的主要功能是根据时间戳实现--个自动的无重复的字符串定义。

一般在获取UUID的时候往往都是随机生成的一个内容，所以可以通过如下方式获取：

- 获取 UUID 对象

```java
public static UUID randomUUID()
```

- 根据字符串获取 UUID 内容

```java
public static UUID fromString(String name)
```

在对一些文件进行自动命名处理的情况下，UUID 类型非常好用。

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        UUID uuid = UUID.randomUUID();
        System.out.println(uuid.toString());
    }
}
```

##### Optional类

[Optional (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Optional.html)

Optional 类的主要功能是进行null的相关处理，在以前进行程序开发时，为了防止程序之中出现空指向异常，往往追加有null的验证。

在引用接受的一方，我们往往都是被动的一方进行判断。

为了解决这种被动的操作处理，在Java中提供了一个有一个optional类，这个类可以实现空的处理操作，在这个类里面提供有如下的一些操作方法：

- 返回空数据：

  ```java
  public static <T> Optional<T> empty()
  ```

- 获取数据

  ```java
  public T get()
  ```

- 保存数据，但是不允许出现null，如果为null，则抛出NullPointerException异常

  ```java
  public static <T> Optional<T> of(T value)
  ```

- 保存数据，允许为空

  ```java
  public static <T> Optional<T> ofNullable(T value)	
  ```

- 空的时候返回其他数据

  ```java
  public T orElse(T other)
  ```

![](https://s2.loli.net/2022/05/30/LVza8hPGSyMHq5j.png)

```java
interface IMessage {
    public String getContent();
}

class MessageImpl implements IMessage {
    @Override
    public String getContent() {
        return "zhuweihao";
    }
}

class MessageUtil {
    private MessageUtil() {
    }

    public static Optional<IMessage> getMessage() {
        return Optional.of(new MessageImpl());
    }

    public static void useMessage(IMessage msg) {
        System.out.println(msg.getContent());
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        MessageUtil.useMessage(MessageUtil.getMessage().get());
    }
}
```

在所有引用数据类型的操作处理之中，null 是一个重要的技术问题，JDK 1.8 后提供的新的类，对于 Null 的处理很有帮助，同时也是在日后进行项目开发之中使用次数很多的一个程序类。

##### ThreadLocal类

[ThreadLocal (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/ThreadLocal.html)

在了解  ThreadLocal 类作用时，先编写一个简单的程序做一个先期的分析。

```java
class Channel{//消息的发送通道
    private static Message message;

    public static void setMessage(Message message) {
        Channel.message = message;
    }
    public static void send(){
        System.out.println("【消息发送】"+message.getInfo());
    }
}
class  Message{//要发送的消息体
    private String info;
    public void setInfo(String info){
        this.info=info;
    }

    public String getInfo() {
        return info;
    }
}
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Message message = new Message();
        message.setInfo("zhuweihao");
        Channel.setMessage(message);
        Channel.send();
    }
}
```

![](https://s2.loli.net/2022/05/30/nVZeOKpkSyNFUBd.png)

当前的程序实际上采用的是一种单线程的模式来进行处理的。

```JAVA
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        new Thread(() -> {
            Message message = new Message();
            message.setInfo("111");
            Channel.setMessage(message);
            Channel.send();
        }, "消息发送者A").start();
        new Thread(() -> {
            Message message = new Message();
            message.setInfo("222");
            Channel.setMessage(message);
            Channel.send();
        }, "消息发送者B").start();
        new Thread(() -> {
            Message message = new Message();
            message.setInfo("333");
            Channel.setMessage(message);
            Channel.send();
        }, "消息发送者C").start();
    }
}
```

这时消息的处理产生了影响

![](https://s2.loli.net/2022/05/30/NXJyr8WRnD9ixPH.png)

在保持 Channel( 所有发送的通道）核心结构不改变的情况下，需要考虑到每个线程的独立操作问题。

那么在这样的情况下，我们发现对于 Channel 类而言除了要保留有发送的消息之外，还应该多存放有一个每线程的标记（当前线程）。

这时我们就可以通过 ThreadLocal 类来实现

在 ThreadLocal 类中，通过有如下的操作方法：

- 构造方法

  ```java
  public ThreadLocal()
  ```

- 设置数据（设置线程副本）

  ```java
  public void set(T value)
  ```

- 获取数据（获取当前线程副本）

  ```java
  public T get()
  ```

- 删除数据（移除当前线程副本）

  ```java
  public void remove()
  ```

![](https://s2.loli.net/2022/05/30/8olYDCvBSpQaue4.png)

```java
class Channel {//消息的发送通道
    private static final ThreadLocal<Message> THREADLOCAL=new ThreadLocal<Message>();
    private Channel() {}
    public static void setMessage(Message message) {
        THREADLOCAL.set(message);//向ThreadLocal中保存数据
    }

    public static void send() {
        System.out.println(Thread.currentThread().getName() + "【消息发送】" + THREADLOCAL.get().getInfo());
    }
}

class Message {//要发送的消息体
    private String info;

    public void setInfo(String info) {
        this.info = info;
    }

    public String getInfo() {
        return info;
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        new Thread(() -> {
            Message message = new Message();
            message.setInfo("111");
            Channel.setMessage(message);
            Channel.send();
        }, "消息发送者A").start();
        new Thread(() -> {
            Message message = new Message();
            message.setInfo("222");
            Channel.setMessage(message);
            Channel.send();
        }, "消息发送者B").start();
        new Thread(() -> {
            Message message = new Message();
            message.setInfo("333");
            Channel.setMessage(message);
            Channel.send();
        }, "消息发送者C").start();
    }
}
```

每一个线程通过ThreadLocal只允许保存一个数据。

##### 定时器

定时器的主要操作是进行定时任务的处理，如果要想实现定时的处理操作主要需要有一个定时操作的主体类，以及一个定时任务的控制。可以使用两个类实现：

- java.util.TimerTask类：实现定时任务的处理

- java.util.Timer类：进行任务的启动，启动的方法为

  - 任务启动：

    ```java
    public void schedule(TimerTask task, long delay)
    ```

```java
class MyTask extends TimerTask {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "、定时任务执行，当前时间" + new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS").format(new Date()));
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Timer timer = new Timer();
        //定义间隔任务，100ms后开始执行，每秒执行1次
        timer.scheduleAtFixedRate(new MyTask(),100,1000);
        timer.schedule(new MyTask(), 0);
    }
}
```

![](https://s2.loli.net/2022/05/30/IDWulKx14abdqvp.png)

Timer 对调度的支持是基于绝对时间，而不是相对时间的，由此任务对系统时钟的改变是敏感的；这种定时是由 JDK 最原始的方式提供的支持，但实际上开发之中利用此类方式进行定时处理的代码会非常复杂

##### Base64加密与解密

[Base64 (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Base64.html)

 Base64.Encoder：进行加密才能处理；

– 加密处理：

```java
public byte[] encode(byte[] src);
```

 Base64.Decoder：进行解密处理；

– 解密处理：

```java
public byte[] decode(String src);
```



```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String str = "zhuweihao";
        byte[] encode = Base64.getEncoder().encode(str.getBytes(StandardCharsets.UTF_8));
        System.out.println(encode);
        String msg = new String(encode);
        System.out.println(msg);
        byte[] decode = Base64.getDecoder().decode(msg);
        for (int i = 0; i < decode.length; i++) {
            System.out.print(decode[i]);
        }
        System.out.println();
        System.out.println(new String(decode));
    }
}
```

虽然 Base64 可以实现加密与解密的处理，但是其由于其是一个公版的算法，所以如果对其进行加密，最好的做法是使用盐值操作。

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String salt = "daoluande";
        String str = new String("zhuweihao");
        str = str + "{" + salt + "}";
        String endcodestr = new String(Base64.getEncoder().encode(str.getBytes()));
        System.out.println(endcodestr);
        String decodestr = new String(Base64.getDecoder().decode(endcodestr));
        System.out.println(decodestr);
    }
}
```

即便现在有盐值也可以发现加密的效果也不是很好，最好的做法是多次加密。

```java
class StringUtil {
    private static final String SALT = "daoluande";//公共的盐值
    private static final int REPEAT = 3;//加密次数

    /**
     * 加密处理
     *
     * @param str 要加密的字符串，需要与盐值结合
     * @return 加密后的数据
     */
    public static String encode(String str) {
        String temp = str + "{" + SALT + "}";
        byte[] data = temp.getBytes();
        for (int i = 0; i < REPEAT; i++) {
            data = Base64.getEncoder().encode(data);
        }
        return new String(data);
    }

    /**
     * 进行解密处理
     *
     * @param str 要解密的内容
     * @return 解密后的内容
     */
    public static String decode(String str) {
        byte[] data = str.getBytes();
        for (int i = 0; i < REPEAT; i++) {
            data = Base64.getDecoder().decode(data);
        }
        return new String(data).replaceAll("\\{\\w+\\}", "");
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String str = StringUtil.encode("zhuweihao");
        System.out.println(str);
        str = StringUtil.decode(str);
        System.out.println(str);
    }
}
```

#### 比较器

##### 比较器问题引出

所谓的比较器指的就是进行大小关系的确定判断，下面首先来分析一下比较器存在的意义。

如果要进行数组操作，肯定使用 java.util.Arrays. 的操作类完成，这个类里面提供有绝大部分的数组操作支持，同时在这个类里面还提供有一种对象数组的排序支持: 

```java
public static void sort(Object[] a);
```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Integer[] data = new Integer[]{10, 9, 4, 7, 18};
        Arrays.sort(data);
        System.out.println(Arrays.toString(data));
        String[] strings = new String[]{"ab", "de", "cfe", "b"};
        Arrays.sort(strings);
        System.out.println(Arrays.toString(strings));
    }
}
```

java.lang.Integer 与 java.lang.String 两个类都是由系统提供的程序类。

```java
class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Person[] people = new Person[]{
                new Person("张三", 14),
                new Person("李四", 34),
                new Person("王五", 23)
        };
        Arrays.sort(people);
        System.out.println(Arrays.toString(people));
    }
}
```

运行时异常

> Exception in thread "main" java.lang.ClassCastException: class com.zhuweihao.Person cannot be cast to class java.lang.Comparable (com.zhuweihao.Person is in unnamed module of loader 'app'; java.lang.Comparable is in module java.base of loader 'bootstrap')
> 	at java.base/java.util.ComparableTimSort.countRunAndMakeAscending(ComparableTimSort.java:320)
> 	at java.base/java.util.ComparableTimSort.sort(ComparableTimSort.java:188)
> 	at java.base/java.util.Arrays.sort(Arrays.java:1041)
> 	at com.zhuweihao.JavaAPIDemo.main(JavaAPIDemo.java:150)

任意的一个类默认情况下是无法使用系统内部的类实现数组排序或比较需求的，是因为没有明确的指定出到底该如何比较的定义（没有比较规则），那么这个时候在 Java 里面为了统一比较规则的定义，所以提供有比较器的接口：Comparable

##### Comparable比较器

通过分析可以发现如果要实现对象的比较肯定需要有比较器来制定比较规则，而比较的规则就通过Comparable来实现，对于 Comparable 而言，需要清楚其基本的定义结构：

![](https://s2.loli.net/2022/05/30/6gs4Sf2ckIdZNzQ.png)

```java
class Person implements Comparable<Person> {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    @Override
    public int compareTo(Person o) {
        return this.age - o.age;
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Person[] people = new Person[]{
                new Person("张三", 14),
                new Person("李四", 34),
                new Person("王五", 23)
        };
        Arrays.sort(people);
        System.out.println(Arrays.toString(people));
    }
}
```

##### Comparator比较器

Comparator 属于一种挽救的比较器支持，其主要的目的是解决一些没有使用 Comparable 排序的类的对象数组排序。



现在程序项目己经开发完成了，并且由于先期的设计并没有去考虑到所谓的比较器功能后来经过了若干版本的迭代更新之后发现需要对 Person 类进行排序处理，但是又不能够去修改 Person 类（无法实现 Compparable 接口），所以这个时候就需要采用一种挽救的形式来实现比较，在 Arrays 类里面排序有另外一种实现。

基于 Comparator 的排序处理：

```java
public static <T> void sort(T[] a, Comparator<? super T> c)
```

![](C:\Users\ZWH\Desktop\8ca436145cca48688e81486ec3c74095.png)

```java
class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

class PersonComparator implements Comparator<Person> {
    @Override
    public int compare(Person o1, Person o2) {
        return o1.getAge() - o2.getAge();
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Person[] people = new Person[]{
                new Person("张三", 14),
                new Person("李四", 34),
                new Person("王五", 23)
        };
        Arrays.sort(people, new PersonComparator());
        System.out.println(Arrays.toString(people));
    }
}
```

在 java.util.Comparator里面最初只定义有一个排序的compare()方法

```java
int compare(T o1, T o2);
```

但是后来持续发展，又定义了许多 static 方法。

Comparable 与 Comparator 的区别？

- java.lang.Comparable 是在类定义的时候实现的父接口，主要用于定义排序规则，里面只有 compareTo()方法。

- java.util.Comparator 是挽救的比较器操作，需要设罝单独的比较器规则类实现排序，里面有compare()方法。



#### 类库使用案例分析

##### StringBuffer使用

定义一个 String Buffer 类对象，然后通过append()方法向对象中添加 26 个小写字母，要求每次只添加一次，共添加26次，然后按照逆序的方式输出，并且可以删除前 5 个字符。

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        StringBuffer stringBuffer = new StringBuffer();
        for (int i = 'a'; i < 'z'; i++) {
            stringBuffer.append((char) i);
        }
        stringBuffer.reverse().delete(0, 5);
        System.out.println(stringBuffer);
    }
}
```

##### 随机数组

利用 Random 类产生 5 个 1~30 之间(包括 1 和 30)的随机整数。

Random 产生随机数的操作之中会包含有数字0

```java
class NumberFactory {
    private static Random random = new Random();

    /**
     * 通过随机数来生成一个数组的内容，改内容不包括0
     * @param len 数组大小
     * @return 生成的随机数组
     */
    public static int[] create(int len) {
        int[] data = new int[len];
        int foot = 0;
        while (foot < data.length) {
            int num = random.nextInt(31);
            if (num != 0) {
                data[foot++] = num;
            }
        }
        return data;
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        int[] result = NumberFactory.create(5);
        System.out.println(Arrays.toString(result));
    }
}
```

##### Email验证

输入一个 Email 地址，然后使用正则表达式验证该 Email 地址是否正确。

对于此时的输入可以通过命令参数实现数据的收入，如果要想进行验证，最好的做法是设置一个单独的验证处理类。

```java
class Validator {
    private Validator() {
    }

    public static boolean isEmail(String email) {
        if (email == null || "".equals(email)) {
            return false;
        }
        String regex = "[a-zA-Z0-9]\\w+@\\w+\\.(cn|com|gov|net)";
        return email.matches(regex);
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        if (args.length != 1) {
            System.out.println("程序执行错误，没有输入初始化参数，正确格式为：java JavaAPIDemo email地址");
            System.exit(1);
        }
        String email = args[0];
        if (Validator.isEmail(email)) {
            System.out.println("输入email正确");
        } else {
            System.out.println("输入Email错误");
        }
    }
}
```

##### 扔硬币

编写程序，用 0-1 之间的随机数来模拟扔硬币试验，统计扔 1000 次后出现正、反面的次数并输出。

```java
class Coin {
    private int front;
    private int back;
    private Random random = new Random();

    public int getFront() {
        return front;
    }

    public int getBack() {
        return back;
    }

    public void throwCoin(int num) {
        for (int i = 0; i < num; i++) {
            int temp = random.nextInt(2);
            if (temp == 0) {
                this.front++;
            } else {
                this.back++;
            }
        }
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Coin coin = new Coin();
        coin.throwCoin(1000);
        System.out.println("正面出现次数" + coin.getFront());
        System.out.println("反面出现次数" + coin.getBack());
    }
}
```

##### IP验证

编写正则表达式，判断给定的是否是一个合法的 IP 地址。

```java
class Validator {
    public static boolean validateIP(String ip) {
        if (ip == null || "".equals(ip)) {
            return false;
        }
        String regex = "([12][0-9]?[0-9]?\\.){3}([12][0-9]?[0-9]?)";
        if (ip.matches(regex)) {
            String[] result = ip.split("\\.");
            for (int i = 0; i < result.length; i++) {
                int temp = Integer.parseInt(result[i]);
                if (temp > 255) {
                    return false;
                }
            }
        } else {
            return false;
        }
        return true;
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String ip = "192.168.1.1";
        System.out.println(Validator.validateIP(ip));
    }
}
```

##### HTML拆分

给定下面的 HTML 代码

```html
 <font face="Arial,Serif" size="+2" color="red">
```

要求对内容进行拆分，拆分之后的结果是:

face Arial,Serif

size +2

color red

对于此时的操作最简单的做法就是进行分组处理。

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String str = "<font face=\"Arial,Serif\" size=\"+2\" color=\"red\">";
        String regex = "\\w+=\"[a-zA-Z0-9,\\+]+\"";
        Matcher matcher = Pattern.compile(regex).matcher(str);
        while (matcher.find()) {
            String temp = matcher.group(0);
            String[] result = temp.split("=");
            System.out.println(result[0] + "\t" + result[1].replaceAll("\"", ""));
        }
    }
}
```

##### 国家代码

编写程序，实现国际化应用，从命令输入国家的代号，例如，1 表示中国，2表示美国，然后根据输入代号的不同调用不同的资源文件显示信息。

本程序的实现肯定要通过Locale类的对象来指定区域，随后利用ResourceBundle类加载资源文件，而继续使用初始化参数形式来定义中文的资源文件完成。

```java
class MessageUtil {
    public static final int CHINA = 1;
    public static final int USA = 2;
    private static final String KEY = "info";
    private static final String BASENAME = "Messages";

    private Locale getLocale(int num) {
        switch (num) {
            case CHINA -> {
                return new Locale("zh", "CN");
            }
            case USA -> {
                return new Locale("en", "US");
            }
            default -> {
                return null;
            }
        }
    }

    public String getMessage(int num) {
        Locale locale = this.getLocale(num);
        if (locale == null) {
            return "nothing";
        } else {
            return ResourceBundle.getBundle(BASENAME, locale).getString(KEY);
        }
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        if (args.length != 1) {
            System.out.println("输入错误");
            System.exit(1);
        }
        int choose = Integer.parseInt(args[0]);
        System.out.println(new MessageUtil().getMessage(choose));
    }
}
```



##### 学生信息比较

按照“姓名：年龄:成绩|姓名:年龄:成绩”的格式定义字符串“张三:21:98|李四:22:89|王五:20:70“，要求将每组值分别保存在  Student 对象之中，并对这些对象进行排序，排序的原则为：按照成绩由高到低排序，如果成绩相等，则按照年龄由低到高排序。

```java
class Student implements Comparable<Student> {
    private String name;
    private int age;
    private double score;

    public Student(String name, int age, double score) {
        this.name = name;
        this.age = age;
        this.score = score;
    }

    @Override
    public int compareTo(Student o) {
        if (this.score < o.score) {
            return 1;
        } else if (this.score > o.score) {
            return -1;
        } else {
            return this.age - o.age;
        }
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", score=" + score +
                '}';
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String string = "张三:21:98|李四:22:89|王五:20:70";
        String[] result = string.split("\\|");
        Student[] students = new Student[result.length];
        for (int i = 0; i < result.length; i++) {
            String[] temp = result[i].split(":");
            students[i] = new Student(temp[0], Integer.parseInt(temp[1]), Double.parseDouble(temp[2]));
        }
        Arrays.sort(students);
        System.out.println(Arrays.toString(students));
    }
}
```



### Java IO编程



#### 文件操作

##### File类基本操作

[File (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/File.html)

在 Java 语言里面提供有对于文件操作，系统操作的支持，而这个支持就在java.io.File 类中进行了定义，在整个 java.io. 包里面，File 类是唯一一个与文件本身操作(创建，删除，重命名等)有关的类，如果想进行File 类的操作，必须要提供有完整的路径，而后可以调用相应的方法进行处理。

打开 JDK 文档可以发现，File 类是 Comparable 接口的子类，所以 File 类的对象是可以进行排序处理的。

在进行 File 类处理的时候需要为其设置访问路径，那么对于路径的配置主要通过File 类的构造方法处理。

- 构造方法

```java
//pathname-路径
public File(String pathname)
//parent-父目录，child-子目录
public File(String parent, String child)
```



```java
//创建新的文件
public boolean createNewFile() throws IOException
//判断文件是否存在
public boolean exists()    
//删除文件
public boolean delete()
```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        File file = new File("F:\\text.txt");
        if (file.exists()) {
            System.out.println("文件已经存在");
            file.delete();
        } else {
            System.out.println("文件创建成功");
        }
    }
}
```

##### File类操作深入

现在已经实现了文件的基础操作，但是对于这个操作里面也是存在有一些问题的，下面针对于之前的代码进行优化处理。

在实际的软件项目开发和运行的过程之中，往往都会在 Windows 中进行项目的开发，而在项目部署的时候基于 Linux 或 Unix 系统来进行项目发布以保证生产环节的安全性；在不同的操作系统之中会存在有不同的路径分割符：Windows 分隔符“\”、Linux分隔符“/”

所以在最初进行开发的时候就必须考虑不同系统环境下的分隔符的问题，所以为了解决此问题，File 类提供有一个常量:

```java
public static final String separator;
```

正常的路径编写

```java
File file = new File("F:"+File.separator+"text.txt");
```

但是随着系统的适应性的不断加强，对于当前的路径操作，也可以随意使用了

```java
File file = new File("F:/text.txt");
```

在使用 File 类进行文件处理的时候需要注意的是：程序→JVM →操作系统函数→文件处理。

重复删除或创建的时候有可能会出现有延迟的问题，所以这个时候最好的方案是别重名。



在进行文件创建的时候有一个重要的前提：文件的父路径必须首先存在

- 获取父路径

  ```java
  public File getParentFile()
  public String getParent()
  ```

- 创建目录

  ```java
  public boolean mkdir()
  public boolean mkdirs()
  ```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        File file = new File("F:" + File.separator + "hello" + File.separator + "message" + File.separator + "text.txt");
        if (!file.getParentFile().exists()) {
            file.getParentFile().mkdirs();
        }
        if (file.exists()) {
            System.out.println("文件已经存在");
            file.delete();
        } else {
            file.createNewFile();
            System.out.println("文件创建成功");
        }
    }
}
```

这种判断并且建立父目录的操作在很多的情况下可能只需要一次，但是如果将这个判断一直都停留在代码里面，那么就会造成时间复杂度的提升，所以这个时候如果要想提升性能，请先保证目录已经创建。

##### 获取文件信息

除了可以进行文件的操作之外也可以通过File类来获取一些文件本身提供的信息

- 文件是否可读

  ```java
  public boolean canRead()
  ```

- 文件是否可写

  ```java
  public boolean canWrite()
  ```

- 获取文件长度（字节长度）

  ```java
  public long length()
  ```

- 最后一次修改日期时间

  ```java
  public long lastModified()
  ```

- 判断是否为目录

  ```java
  public boolean isDirectory()
  ```

- 判断是否是文件

  ```java
  public boolean isFile()
  ```

- 既然可以判断给定的路径是文件还是目录，那么就可以进一步的判断，如果发现是目录，则应该列出目录中的全部内容

  ```java
  public File[] listFiles()
  ```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        File file = new File("F:" + File.separator + "hello" + File.separator + "message" + File.separator + "text.txt");
        System.out.println("文件是否可读：" + file.canRead());
        System.out.println("文件是否可写：" + file.canWrite());
        System.out.println("文件大小：" + file.length());
        System.out.println("最后的修改时间" + new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date(file.lastModified())));
        System.out.println("是否为目录：" + file.isDirectory());
        System.out.println("是否为文件：" + file.isFile());
        File fff = new File("F:" + File.separator);
        if (fff.isDirectory()) {
            File[] result = fff.listFiles();
            for (int i = 0; i < result.length; i++) {
                System.out.println(result[i]);
            }
        }
    }
}
```

##### 案例：列出目录结构

现在可以由开发者任意设置一个目录的路径，而后将这个目录中所有的文件的信息全部列出，包括子目录中的所有文件。

这样的处理情况下最好的做法就是利用递归的形式来完成。

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        File file = new File("F:" + File.separator);
        listDir(file);
    }

    public static void listDir(File file) {
        if (file.isDirectory()) {
            File[] result = file.listFiles();
            if (result != null) {
                for (int i = 0; i < result.length; i++) {
                    listDir(result[i]);
                    System.out.println(result[i]);
                }
            }
        }
    }
}
```



##### 案例：文件批量改名

编写程序，程序运行时输入目录名称，并把该目录下的所有文件名后缀修改为 .txt

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        File file = new File("F:" + File.separator + "hello" + File.separator);
        rename(file);
    }

    public static void rename(File file) {
        if (file.isDirectory()) {
            File[] result = file.listFiles();
            if (result != null) {
                for (int i = 0; i < result.length; i++) {
                    rename(result[i]);
                }
            }
        } else {
            String filename = null;
            if (file.isFile()) {
                filename = file.getName().substring(0, file.getName().lastIndexOf(".")) + ".txt";
                System.out.println(filename);
            }
            File newFile = new File(file.getParentFile(), filename);
            file.renameTo(newFile);
        }
    }
}
```



#### 字节流与字符流

##### 流的基本概念

在 java.io 包里面 File 类是唯一一个与文件本身有关的程序处理类，但是 File 类只能够操作文件本身而不能够操作文件内容。

在实际的开发之中IO操作的核心意义在于：输入与输出操作，对于程序而言，输入与输出可能来自于不同的环境。

例如，通过电脑连接服务器上进行浏览的时候，实际上此时客户端发出了一个信息，而后服务器接收到此信息之后进行回应处理。

![](https://s2.loli.net/2022/05/30/KLNRutb6FUAP5kQ.png)

对于服务器或者是客户端而言实质上传递的就是一种数据流的处理形式，而所谓的数据流指的就是字节数据。

对于这种流的处理形式在 java.io 包里面提供有两种支持：

- 字节处理流：OutputStream（输出字节流）、InputStream（输入字节流）

- 字符处理流：Writer（输出字符流）、Reader（输入字符流）

所有的流操作都应该采用如下统的步骤进行，下面以文件处理的流程为例：

1. 如果现在要进行的是文件的读写操作，则一定要通过 File 类找到一个文件路径；
2. 通过字节流或字符流的子类为父类对象实例化；
3. 利用字节流或字符流中的方法实现数据的输入与输出操作；
4. 流的操作属于资源操作，资源操作必须进行关闭处理。

##### OutputStream字节输出流

[OutputStream (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/OutputStream.html)

字节的数据是以 byte 类型为主实现的操作，在进行字节内容输出的时候可以使用 OutputStream 类完成，这个类的基本定义如下：

```java
public abstract class OutputStream extends Object implements Closeable, Flushable
```

首先可以发现这个类实现了两个接口，于是基本的对应关系如下：

![](https://s2.loli.net/2022/05/30/DoWVdZ5ImjYygbh.png)

```java
public interface Closeable extends AutoCloseable {
    public void close() throws IOException;
}
```

```java
public interface Flushable {
    void flush() throws IOException;
}
```

OutputStream 类定义的是一个公共的输出操作标准，而在这个操作标准里面，一共定义有三个内容输出的方法：

- 输出单个字节数据

  ```java
  public abstract void write(int b) throws IOException;
  ```

- 输出一组字节数据

  ```java
  public void write(byte b[]) throws IOException
  ```

- 输出部分字节数据

  ```java
  public void write(byte b[], int off, int len) throws IOException 
  ```

但是需要注意的一个核心问题是: OutputStream 类毕竟是一个抽象类，而这个抽象类如果想要获得实例化对象，按照认识应该通过子类实例的向上转型完成。

如果说现在要进行的文件处理操作，则可以使用 FileOutputStream 子类

因为最终都需要发生向上转型的处理关系，所以对于此时的FileOutputStream子类，核心的关注点就可以放在构造方法上：

- 【覆盖】构造方法：

  ```java
  public FileOutputStream(File file) throws FileNotFoundException
  ```

- 【追加】构造方法：

  ```java
  public FileOutputStream(File file, boolean append)
          throws FileNotFoundException
  ```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        File file = new File("F:" + File.separator + "hello" + File.separator + "message" + File.separator + "text.txt");
        if (!file.getParentFile().exists()) {
            file.getParentFile().mkdirs();
        }
        OutputStream outputStream = new FileOutputStream(file);
        String string = "zhuweihao";
        outputStream.write(string.getBytes());
        outputStream.close();
    }
}
```

本程序是采用了最为标准的形式实现了输出的操作处理，并且在整体的处理之中，只是创建了文件的父目录，但是并没有创建文件，而在执行后会发现文件可以自动帮助用户创建。

由于 OutputStream 子类也属于 AutoCloseable 接口子类，所以对于 close()方法也可以简化使用。

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        File file = new File("F:" + File.separator + "hello" + File.separator + "message" + File.separator + "text1.txt");
        if (!file.getParentFile().exists()) {
            file.getParentFile().mkdirs();
        }
        try (OutputStream outputStream = new FileOutputStream(file, true)) {
            String string = "zhuweihao\r\n";
            outputStream.write(string.getBytes());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

是否使用自动的关闭取决于你项目的整体结构

##### InputStream字节输入流

[InputStream (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/InputStream.html)

与 OutputStream 类对应的一个流就是字节输入流，InputStream 类主要实现的就字节数据读取，该类定义如下

```java
public abstract class InputStream implements Closeable
```

在 InputStream 类里面定义了如下的几个核心方法：

- 读取单个字节数据，返回字节数据，如果现在已经读取完全部数据，返回-1

  ```java
  public abstract int read() throws IOException;
  ```

- 读取一组字节数据，返回所读取字节数组长度，如果没有数据可以读取，或者说已经读取完全部数据，返回-1，

  ```java
  public int read(byte b[]) throws IOException
  ```

- 读取部分字节数据，返回读取的字节个数

  ```java
  public int read(byte b[], int off, int len) throws IOException
  ```

InputStream 类属于一个抽象类，这时应该依靠它的子类来实例化对象，如果要从文件读取，则一定要使用 FileInputStream 子类，对于子类而言，只关心父类对象实例化，构造方法：

```java
public FileInputStream(File file) throws FileNotFoundException
```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        File file = new File("F:" + File.separator + "hello" + File.separator + "message" + File.separator + "text1.txt");
        InputStream inputStream = new FileInputStream(file);
        byte[] data = new byte[1024];
        int len = inputStream.read(data);
        System.out.println("【" + new String(data, 0, len) + "】");
        inputStream.close();
    }
}
```

对于字节输入流里面最为麻烦的问题就在于：使用 read() 方法读取的时候只能够以字节数组为主进行接收。

特别需要注意的是从 JDK1.9 开始在InputStream类里面增加了一个新的方法：

```java
public byte[] readAllBytes() throws IOException
```

如果你现在要读取的内容很大很大的时候，那么使用上面的读取方式会直接搞死你的程序。

##### Writer字符输出流

[Writer (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/Writer.html)

使用 OutputStream 字节输出流进行数据输出的时候使用的都是字节类型的数据，而很多情况下字符串的输出是比较方便的，所以对于 java.io 而言，在 JDK1.1的时候又推出了字符输出流：Writer，这个类的定义如下：

```java
public abstract class Writer implements Appendable, Closeable, Flushable
```

![](https://s2.loli.net/2022/05/31/FcoYJTiaPWqZprX.png)

- 输出字符数组

  ```java
  public void write(char cbuf[]) throws IOException
  ```

- 输出字符串

  ```java
  public void write(String str) throws IOException
  ```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        File file = new File("F:" + File.separator + "hello" + File.separator + "message" + File.separator + "text3.txt");
        if (!file.getParentFile().exists()) {
            file.getParentFile().mkdirs();
        }
        Writer writer = new FileWriter(file, true);
        String str = "zhuweihao";
        writer.write(str);
        writer.append("我是中国人");
        writer.close();
    }
}
```

使用 Writer 输出的最大优势在于可以直接利用字符串完成。

Writer 是字符流，字符处理的优势在于中文数据。

##### Reader字符输入流

Reader 是实现字符输入流的一种类型，其本身属于一个抽象类，这个类的定义如下：

```java
public abstract class Reader implements Readable, Closeable
```

- 接收数据

  ```java
  public int read(char[] cbuf) throws IOException
  ```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        File file = new File("F:" + File.separator + "hello" + File.separator + "message" + File.separator + "text3.txt");
        if (file.exists()) {
            Reader reader = new FileReader(file);
            char[] chars = new char[1024];
            int len = reader.read(chars);
            System.out.println("读取内容：" + new String(chars, 0, len));
            reader.close();
        }
    }
}
```

字符流读取的时候只能够按照数组的形式来实现处理操作。

##### 字节流与字符流的区别

现在通过一系列的分析已经可以清楚字节流与字符流的基本操作了，但是对于这两类流依然是存在有区别的，重点分析一下输出的处理操作。

使用OutputStream和Writer输出的最后发现都使用了 close (方法进行了关闭处理

在使用 OutputStream 类输出的时候如果发现没有使用 close() 方法关闭输出流，内容依然可以实现正常的输出。

在使用 Writer 类输出的时候，如果没有使用 close() 方法关闭输出流，那么这时内容无法进行输出，因为 Writer 使用到了缓冲区。

当使用了 close() 方法的时候，实际上会出现有强制刷新缓区的情况，所以这个时候会将内容进行输出。如果没有关闭，那么将无法进行输出操作。所以此时如果在不关闭的情况下，要想将全部的内容输出可以使用 flush() 方法强制性清空。

字节流在进行处理的时候并不会使用到缓冲区，而字符流会使用到缓冲区。

另外，使用缓冲区的字符流更加于进行中文数据的处理。所以在日后的程序开发之中，如果要涉及到包含有中文信息的输出一般都会使用字符流处理。

##### 转换流

所谓的转换流指的是可以实现字节流与字符流操作的功能转换。

例如：进行输出的时候，OutputStream需要将内容变为字节数组后才可以输出，而 Writer 可以直接输出字符串，这一点是方便的，所以很多人就认为需要提供有一种转换的机制来实现不同流类型的转换操作。

为此在 java.io 包里面提供有两个类：

InputStreamReader、OutputStreanmwriter。

```java
public class InputStreamReader extends Reader{
    public InputStreamReader(InputStream in)
}
public class OutputStreamWriter extends Writer{
    public OutputStreamWriter(OutputStream out)
}
```

![](https://s2.loli.net/2022/05/31/Ebwar5ROZmKVLJk.png)接收字节流对象利用向上转换变成字符流对象。

转换的本质在于对象的转型和构造方法的接收。通过类的继承结构与构造方法，可以发现所谓的转换处理就是将接收到的字节流对象通过向上转流型变为字符流对象。

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        File file = new File("F:" + File.separator + "hello" + File.separator + "message" + File.separator + "text3.txt");
        if (!file.getParentFile().exists()) {
            file.getParentFile().mkdirs();
        }
        OutputStream outputStream = new FileOutputStream(file);
        Writer writer = new OutputStreamWriter(outputStream);
        writer.write("\r\n朱伟豪");
        writer.close();
    }
}
```

此转换在是中文的情况下，处理较为方便，在不是中文的情况下，处理不够方便，一般不使用。

观察 FileWriter，FileReader 类的继承关系。

![image-20220531102406133](https://s2.loli.net/2022/05/31/tvFdWqIB2PJ5a8H.png)



![image-20220531102718455](https://s2.loli.net/2022/05/31/7uWopQzlYbgqImv.png)



![image-20220531103501350](https://s2.loli.net/2022/05/31/s1P4ZntdgIrBeoU.png)



cpu请求到的数据是字节数据，而整个流程中磁盘传的是字节数据，而有了缓存之后相当于所有的数据已经经过处理。

意味着这些数据要读取到缓存区之中，读取到缓存区中直接过程就在于会对数据进行先期处理。在处理过程中就极为方便处理中文数据。

在电脑中，文件都是二进制数据的集合，缓冲区会进行处理。转换流的处理在定义结构上更加清楚描述出所有读取到的字节数据，并要求进行转换处理。

这就是转换流存在的意义所在。

##### 案例：文件拷贝

在操作系统里面有一个copy命令，这个命令的主要功能是可以实现文件的拷贝处理。

现在要求模拟这个命令，通过初始化参数输入拷贝的源文件路径与拷贝的目标路径，实现文件的拷贝处理。

需求分析：

- 需要实现文件的拷贝操作，那么这种拷贝就有可能拷贝各种类型的文件，所以此时选择使用字节流。
- 在进行拷贝的时候，有可能需要考虑到大文件的拷贝问题。

实现方案如下：

- 方案一：使用 InputStream 将全部要拷贝的内容直接读取到程序里面，而后一次性的输出到目标文件；此方案的缺点就在于，如果现在拷贝的文件很大，基本上程序就死了；
- 方案二：采用部分拷贝，读取一部分输出一部分数据，如果现在要采用第二种做法，核心的操作方法为 InputStream 和 OutputStream。

```java
class FileUtil {
    private File srcFile;
    private File desFile;

    public FileUtil(String srcFile, String desFile) {
        this(new File(srcFile), new File(desFile));
    }

    public FileUtil(File srcFile, File desFile) {
        this.srcFile = srcFile;
        this.desFile = desFile;
    }

    public boolean copy() throws Exception {
        if (!this.srcFile.exists()) {
            System.out.println("源文件不存在");
            return false;
        }
        if (!this.desFile.getParentFile().exists()) {
            this.desFile.getParentFile().mkdirs();
        }
        byte[] data = new byte[1024];
        InputStream inputStream = null;
        OutputStream outputStream = null;
        try {
            inputStream = new FileInputStream(this.srcFile);
            outputStream = new FileOutputStream(this.desFile);
            //inputStream.transferTo(outputStream);可替换下面的循环语句
            int len = 0;
            while (len != -1) {
                len = inputStream.read(data);
                System.out.println(len);
                if (len != -1) {
                    outputStream.write(data, 0, len);
                }
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            throw e;
        } finally {
            if (inputStream != null) {
                inputStream.close();
            }
            if (outputStream != null) {
                outputStream.close();
            }
        }
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        if (args.length != 2) {
            System.out.println("输入错误");
            System.exit(1);
        }
        long start = System.currentTimeMillis();
        FileUtil fileUtil = new FileUtil(args[0], args[1]);
        System.out.println(fileUtil.copy() ? "文件拷贝成功" : "文件拷贝失败");
        long end = System.currentTimeMillis();
        System.out.println("拷贝用时：" + (end - start));
    }
}
```

从 jdk1.9 开始，inputstream 和 reader 类中都追加有数据转存的处理操作方法:

```java
public long transferTo(OutputStream out) throws IOException
```

```java
public long transferTo(Writer out) throws IOException
```

使用转存的方式处理

现在对此程序要求进一步扩展，可以实现一个文件目录的拷贝。一旦进行了文件目录的拷贝，还需要拷贝所有的子目录中的文件。

```java
class FileUtil {
    private File srcFile;
    private File desFile;

    public FileUtil(String srcFile, String desFile) {
        this(new File(srcFile), new File(desFile));
    }

    public FileUtil(File srcFile, File desFile) {
        this.srcFile = srcFile;
        this.desFile = desFile;
    }

    public boolean copy() throws Exception {
        return this.copyFileImpl(this.srcFile, this.desFile);
    }

    public boolean copyDir() throws Exception {
        if (!this.srcFile.exists()) {
            System.out.println("源文件不存在");
            return false;
        }
        try {
            this.copyImpl(this.srcFile);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    private void copyImpl(File file) throws Exception {
        if (file.isDirectory()) {
            File[] results = file.listFiles();
            if (results != null) {
                for (int i = 0; i < results.length; i++) {
                    copyImpl(results[i]);
                }
            }
        } else {
            String newFilePath = file.getPath().replace(this.srcFile.getPath() + File.separator, "");
            File newFile = new File(this.desFile, newFilePath);
            this.copyFileImpl(file, newFile);
        }
    }

    private boolean copyFileImpl(File srcFile, File desFile) throws Exception {
        if (!srcFile.exists()) {
            System.out.println("源文件不存在");
            return false;
        }
        if (!desFile.getParentFile().exists()) {
            desFile.getParentFile().mkdirs();
        }
        InputStream inputStream = null;
        OutputStream outputStream = null;
        try {
            inputStream = new FileInputStream(srcFile);
            outputStream = new FileOutputStream(desFile);
            inputStream.transferTo(outputStream);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            throw e;
        } finally {
            if (inputStream != null) {
                inputStream.close();
            }
            if (outputStream != null) {
                outputStream.close();
            }
        }
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        if (args.length != 2) {
            System.out.println("输入错误");
            System.exit(1);
        }
        long start = System.currentTimeMillis();
        FileUtil fileUtil = new FileUtil(args[0], args[1]);
        if (new File(args[0]).isFile()) {
            System.out.println(fileUtil.copy() ? "文件拷贝成功" : "文件拷贝失败");
        } else {//目录拷贝
            System.out.println(fileUtil.copyDir() ? "文件拷贝成功" : "文件拷贝失败");
        }
        long end = System.currentTimeMillis();
        System.out.println("拷贝用时：" + (end - start));
    }
}
```

#### IO深入操作

##### 字符编码

在计算机的世界里面，只认 0、1 的数据，如果要想描述一些文字的编码，就需要对这些二进制的数据进行组合，所以才有了现在可以看见的中文。

但是在进行编码的时候，如果要想正确解释出内容，则一定需要有解码，所以编码和解码肯定要采用同一标准，如果不统一的时候，就会出现乱码。

所以编码的统一是解决出现乱码的唯一途径。那么在实际的开发之中，对于常用的编码，有如下几种：

- GBK/GB2312 国标编码，可以描述中文信息，其中 GB2312 只描述简体中文，而GBK包含有简体中文与繁体中文。
- ISO8859_1 国际通用编码，可以描述所有文字信息，但如果处理不当，也会出现乱码。
- Unicode 编码采用 16 进制的方式存储，可以描述所有的文字信息。其有一个最大的缺点就是大。
- UTF 编码象形文字部分使用16进制编码，而普通的字母采用的是ISO8859-1编码，它的优势在于适合于快速的传输，节约带宽，这也就成为了在开发之中首选的编码。存在 UTF-16 和 UTF_8，其中主要使用utf-8编码。

##### 内存操作流

文件操作流的特点：

程序利用 InputStream 读取文件内容，而后程序利用 OutputStream 向文件输出内容，所有的操作都是以文件为终端的。

内存流的优势：

需要实现 IO 操作，可是又不希望产生文件（相当于临时文件）则可以以内存为终端进行处理。

字节内存操作流：

![](https://s2.loli.net/2022/06/01/7GUdCfSjKi4XLhc.png)

![](https://s2.loli.net/2022/06/01/1vsViIw7tRGHMl5.png)

字符内存操作流：

![](https://s2.loli.net/2022/06/01/amu9xCDoQcfzZBJ.png)



![](https://s2.loli.net/2022/06/01/1SFlhuHXUE2T4LB.png)

在 ByteArrayOutputStream 类里面有一个重要的方法，这个方法可以获取全部保存在内存流里面的数据信息。

- 获取数据：

  ```java
  public synchronized byte[] toByteArray()
  ```

- 使用字符串的形式来获取数据：

  ```java
  public synchronized String toString()
  ```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String str = "zhuweihao";
        InputStream inputStream = new ByteArrayInputStream(str.getBytes());
        OutputStream outputStream = new ByteArrayOutputStream();
        int data = 0;
        while ((data = inputStream.read()) != -1) {
            outputStream.write(Character.toUpperCase((char) data));
        }
        System.out.println(outputStream.toString());
        inputStream.close();
        outputStream.close();
    }
}
```

##### 管道流

管道流的主要功能是实现两个线程之间的 IO 处理操作

![](https://s2.loli.net/2022/06/01/fITvqJBwF4GAOQx.png)

对于管道流也是分为两类：

- 字节管道流：PipedOutputStream、PipedInputStream;

连接处理：

```java
public void connect (PipedInputStream snk) throws IOException;
```

- 字符管道流：PipedWriter、PipedReader。

连接处理：

```java
public void connect (PipedReader snk) throws IOException
```

```java
class SendThread implements Runnable {
    private PipedOutputStream pipedOutputStream;

    public SendThread() {
        this.pipedOutputStream = new PipedOutputStream();
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            try {
                this.pipedOutputStream.write(((i + 1) + "次消息发送-" + Thread.currentThread().getName() + "zhuweihao\n").getBytes());
            } catch (IOException e) {
                e.printStackTrace();
            }
        }


        try {
            this.pipedOutputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public PipedOutputStream getPipedOutputStream() {
        return pipedOutputStream;
    }
}

class ReceiveThread implements Runnable {
    private PipedInputStream pipedInputStream;

    public ReceiveThread() {
        this.pipedInputStream = new PipedInputStream();
    }

    public PipedInputStream getPipedInputStream() {
        return pipedInputStream;
    }

    @Override
    public void run() {
        byte[] data = new byte[1024];
        int len = 0;
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        try {
            while ((len = this.pipedInputStream.read(data)) != -1) {
                byteArrayOutputStream.write(data, 0, len);
            }
            System.out.println(Thread.currentThread().getName() + "接收消息：\n" + byteArrayOutputStream.toString());
            byteArrayOutputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            this.pipedInputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        SendThread sendThread = new SendThread();
        ReceiveThread receiveThread = new ReceiveThread();
        sendThread.getPipedOutputStream().connect(receiveThread.getPipedInputStream());
        new Thread(sendThread, "消息发送线程").start();
        new Thread(receiveThread, "消息接收线程").start();
    }
}
```



##### RandomAccessFile

对于文件内容的处理操作主要是通过 InputStream ( Reader)、OutputStream ( Writer) 来实现，但是利用这些类实现的内容读取只能够将数据部分读取进来。

若有一个文件的大小有 20G，此时按照传统的 IO 操作进行读取和分析无法完成，java.io 包里的 RandomAccessFile 类，可以实现文件跳跃式读取内容(前提:需要有一个完善的保存形式)，数据的保存位数要确定好。

RandomAccessFile 类里面定义有如下的操作方法：

```java
public RandomAccessFile(File file, String mode)
    throws FileNotFoundException
```

RandomAccessFile 最大的特点在于数据的读取处理上，因为所有的数据是按照固定的长度进行的保存，所以读取的时候就可以进行跳字节读取:。

- 向下跳: 

  ```java
  public int skipBytes(int n) throws IOException
  ```

- 向回跳: 

  ```java
  public void seek(long pos) throws IOException
  ```

  

#### 输入与输出支持

##### 打印流

OutputStream 是唯一可以实现输出的操作标准类，应该以其为核心根本，但这个类输出的操作功能有限，所以不方便进行输出各个数据类型，因此要为它做出一层包装。

为了解决输出问题，在 java.io 包里面提供有打印流: PrintStream、 PrintWriter。

![](https://s2.loli.net/2022/06/01/dZqivUJFtz5hfBr.png)

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        File file = new File("F:" + File.separator + "test.txt");
        PrintWriter printWriter = new PrintWriter(new FileOutputStream(file));
        String name = "张三";
        int age = 19;
        double salary = 890.234;
        printWriter.printf("姓名:%s、年龄:%d、收入:%9.2f", name, age, salary);
        printWriter.close();
    }
}
```

比起直接使用 OutputStream 类，使用 PrintWriter、 PrintStream 类的处理操作会更加的简单。

只要是程序进行内容输出的时候全部使用打印流。

##### System类对IO的支持

System 类是一个系统类，且一直都在使用的系统类，而在这个系统类之中提供有三个常量.



标准输出(显示器): `public static final PrintStream out`

错误输出:` public static final PrintStream err;`

标准输入(键盘): `public static final InputStream in。`

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        InputStream inputStream = System.in;
        System.out.println("请输入信息：");
        byte[] data = new byte[1034];
        int len = inputStream.read(data);
        System.err.println("输入内容为：" + new String(data, 0, len));
    }
}
```

##### BuffererReader类

BufferedReader 类提供的是一个缓冲字符输入流的概念，利用 BufferedReader 类可以很好的解决输入流数据的读取问题，这个类最初提供最完善的数据输入的处理(JDK1.5之前，JDK1.5 之后出了一个功能更强大的类代替此类)。

![](https://s2.loli.net/2022/06/01/ojefPHlQOb8naVv.png)

使用这个类处理，是因为这个类提供了一个重要的方法:

读取一行数据: 

```java
public String readLine() throws IOException
```

将利用这个类实现键盘输入数据的标准化定义。

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));
        System.out.println("请输入您的年龄");
        String msg = bufferedReader.readLine();
        if (msg.matches("\\d{1,3}")) {
            int age = Integer.parseInt(msg);
            System.out.println("年龄为：" + age);
        } else {
            System.out.println("输入错误");
        }
    }
}
```

##### Scanner扫描流

java.util.Scanner 是从 JDK 1.5  之后追加的一个程序类，其主要目的是为了解决输入流的访问问题，可以理解为 BufferedReader 的替代功能类。

```java
public final class Scanner
extends Object
implements Iterator<String>, Closeables
```

在 Scanner 类里面有如下几种操作方法:

枸造: 

```java
public Scanner(InputStream source)
```

判断是否有数据: `public boolean hasNext(): `

取出数据: `public String next();`

设置分隔符:` public Scanner useDelimiter(String pattern)`

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入年龄");
        if (scanner.hasNextInt()) {
            int age = scanner.nextInt();
            System.out.println("年龄为：" + age);
        } else {
            System.out.println("输入错误");
        }
        scanner.close();
    }
}
```

使用 Scanner 输入数据还有一个最大的特点：可以直接利用正则进行验证判断。

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入生日");
        if (scanner.hasNext("[1,2]\\d{3}-[0,1]\\d-[0-3]\\d")) {
            String str = scanner.next("[1,2]\\d{3}-[0,1]\\d-[0-3]\\d");
            System.out.println("生日" + new SimpleDateFormat("yyyy-MM-dd").parse(str));
        } else {
            System.out.println("输入错误");
        }
        scanner.close();
    }
}
```

现在可以发现 Scanner 的整体设计要好于 BufferedReader，而且要比直接使用InputStream 类读取要方便。

例如，要读取一个文本文件中的所有内容信息，如果采用的是 InputStream 类，那么就必须依靠内存输出流进行临时数据的保存，并且要判断读取的内容是否是换行。

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Scanner scanner = new Scanner(new File("F:\\9DM THE KING OF FIGHTERS XV\\9DM THE KING OF FIGHTERS XV\\游戏说明.txt"));
        scanner.useDelimiter("\n");
        while (scanner.hasNext()) {
            System.out.println(scanner.next());
        }
        scanner.close();
    }
}
```

在以后的开发过程中，如果程序需要输出数据一定使用打印流，输入数据使用Scanner ( BufferedReader)。

#### 对象序列化

##### 对象序列化基本概念

对象序列化指的是将内存中保存的对象以二进制数据流的形式进行处理，实现对象的保存或者是网络传输。

序列化是将对象转换为可传输格式的过程。 是一种数据的持久化手段。一般广泛应用于网络传输，RMI和RPC等场景中。

![](https://s2.loli.net/2022/06/01/QXm9vWIHLp3KOPs.png)

并不是所有的对象都可以被序列化，在 Java 里面有一个强制性的要求：如果要序列化的对象，那么对象所在的类实现 java.io.Serializable父接口，作为序列化的标记。

这个接口描述的是一种类的能力。

##### 序列化与反序列化

序列化：[ObjectOutputStream (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/ObjectOutputStream.html)

反序列化：[ObjectInputStream (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/ObjectInputStream.html)

```java
class People implements Serializable {
    private String name;
    private int age;

    public People() {
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    public People(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

public class JavaAPIDemo {
    private static final File SAVE_FILE = new File("F:\\zhu.people");

    public static void main(String[] args) throws Exception {
        saveObject(new People("zhangsan",89));
        System.out.println(loadObject());
    }

    public static void saveObject(Object obj) throws Exception {
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream(SAVE_FILE));
        objectOutputStream.writeObject(obj);
        objectOutputStream.close();
    }
    public static Object loadObject() throws Exception{
        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream(SAVE_FILE));
        Object o = objectInputStream.readObject();
        objectInputStream.close();
        return o;
    }
}
```

在 Java 中的对象序列化与反序列化必须使用内部提供的对象操作流,因为这里面牵扯到二进制数据的格式，所以不能够自义处理。另外，如果要想实现一组对象的序列化，则可以使用对象数组完成。

在很多的实际项目开发过程之中，开发者很少能够见到 ObjectOutputStream、 ObjectInput Stream 直接操作，因为会有一些容器帮助开发者自动实现。

##### transient关键字

默认情况下，当执行了对象序列化的时候，会将类中的全部属性的内容进行全部的序列化操作。但是很多情况下，有一些属性并不需要进行序列化的处理，这个时候就可以在属性定义上使用 transient 关键字来完成了。

但是在实际的开发之中，大部分需要被序列化的类往往都是简单 java 类，所以这一个关键字的出现频率并不高。



#### Java IO编程案例



### Java反射编程

#### 认识反射机制

##### 反射机制简介

在 Java 中语言之所以会有如此众多的开源技术支持，很大一部分来自Java最大的特征反射机制。

开发中很重要的一点是代码的重用性

传统的正向操作

1. 导入程序的包
2. 通过类实现实例化对象
3. 根据对象调用类的反复

反射则可以实现根据实例化对象反推其类型，想要实现这样的操作，需要使用Object类中的方法：

```java
public final native Class<?> getClass();
```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Date date = new Date();
        System.out.println(date.getClass());
    }
}
```

##### Class类对象的三种实例化模式

反射之中所有的核心操作都是通过 Class 类对象展开的，Class 类是反射操作的根源所在，但是这个类如果要想获取它的实例化对象，可以采用三种方式完成

```java
public final class Class<T>
extends Object
implements Serializable, GenericDeclaration, Type, AnnotatedElement, TypeDescriptor.OfField<Class<?>>, Constable
```

- Object 类可以根据实例化对象获取 Class 对象

```java
class Person { }
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Person person = new Person();
        Class<? extends Person> personClass = person.getClass();
        System.out.println(personClass);
        System.out.println(personClass.getName());
    }
}
```

缺点：如果现在只想获得 Class 类对象，则必须产生指定类对象后才可以获得

- 【JVM直接支持】采用“类.class”的形式实例化

  ```java
  class Person { }
  public class JavaAPIDemo {
      public static void main(String[] args) throws Exception {
          Class<? extends Person> personClass = Person.class;
          System.out.println(personClass);
          System.out.println(personClass.getName());
      }
  }
  ```

- 【Class类支持】

  ```java
  public class JavaAPIDemo {
      public static void main(String[] args) throws Exception {
          Class<?> personClass = Class.forName("vo.Person");
          System.out.println(personClass);
          System.out.println(personClass.getName());
      }
  }
  ```

  这种模式最大的特点是可以直接采用字符串的形式定义要使用的类型，并且程序中不需要编写任何的import语句。要使用的程序类不存在则会抛出 “ java.lang.ClassNotFoundException ” 异常。

#### 反射应用案例

##### 反射实例化对象

获取 Class 对象之后最大的意义实际上并不是在于只是一个对象的实例化操作形式,更重要的是 Class 类里面提供的反射实例化方法（代替了关键字new）

```java
@SuppressWarnings("removal")
@CallerSensitive
@Deprecated(since="9")
public T newInstance()
	throws InstantiationException, IllegalAccessException
```

JDK1.9之后

```java
clazz.getDeclaredConstructor().newInstance()
```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Class<?> personClass = Class.forName("vo.Person");
        Object o = personClass.newInstance();
        System.out.println(o);
    }
}
```

现在通过反射实现的对象实例化处理,依然要调用类中的无参构造方法。

其本质等价于“类对象= new 类（）”,也就是说当于隐含了关键字 new ,而直接使用字符串进行了替代。

从 JDK1.9 之后 newInstance ()被替代了

因为默认的Class类中的 newInstance()方法只能够调用无参构造

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Class<?> personClass = Class.forName("vo.Person");
        Object o = personClass.getDeclaredConstructor().newInstance();
        System.out.println(o);
    }
}
```

##### 反射与工厂设计模式

如果要想进行对象的实例化处理除了可以使用关键字 new 之外,还可以使用反射机制来完成,一定会思考:为什么要提供有一个反射的实例化? 使用关键字new还是使用反射 ?

如果要想更好的理解此类问题，最好的解释方案就是通过工厂设计模式来解决。工厂设计模式的最大特点：客户端的程序类不直接牵扯到对象的实例化管理，只与接口发生关联，通过工厂类获取指定接口的实例化对象。

在实际的开发之中,接口的主要作用是为不同的层提供一个操作标准。但如果此时直接将一个子类设置实例化操作，一定会有耦合问题，所以使用工厂设计模式来解决此问题

工厂设计模式最有效解决的是子类与客户端的耦合问题，解决的核心思想是在于提供有一个工厂类作为过渡端,随着项目的进行，子类产生的可能越来越多，那么此时就意味着工厂类永远都要进行修改 。

最好的解决方案就是不使用关键字 new 来完成,因为关键字 new 在使用的时候需要有一个明确的类存在。

利用反射机制实现的工厂设计模式,最大的优势在于对于接口子类的扩充将不再影响到工厂类的定义。

```java
interface IFood {
    public void eat();
}

class Bread implements IFood {
    @Override
    public void eat() {
        System.out.println("吃面包");
    }
}

class Milk implements IFood {
    @Override
    public void eat() {
        System.out.println("喝牛奶");
    }
}

class Factory {
    public static IFood getInstance(String className) {
        IFood instance=null;
        try {
            instance=(IFood) Class.forName(className).getDeclaredConstructor().newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return instance;
    }
}
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String classname = "com.zhuweihao.Bread";
        IFood food = Factory.getInstance(classname);
        food.eat();
    }
}
```

在实际开发中存在有大量的接口，并且这些接口都可能需要通过工厂类实例化，此时的工厂设计模式不应该只为一个接口服务，应该变为为所有的接口服务。

```java
interface IFood {
    public void eat();
}

class Bread implements IFood {
    @Override
    public void eat() {
        System.out.println("吃面包");
    }
}

class Milk implements IFood {
    @Override
    public void eat() {
        System.out.println("喝牛奶");
    }
}

class Factory {
    /**
     * 获取接口实例化对象
     *
     * @param className 接口的子类
     * @param clazz     描述接口类型
     * @param <T>
     * @return 如果子类存在返回指定接口实例化对象
     */
    public static <T> T getInstance(String className, Class<T> clazz) {
        T instance = null;
        try {
            instance = (T) Class.forName(className).getDeclaredConstructor().newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return instance;
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        String classname = "com.zhuweihao.Bread";
        IFood food = Factory.getInstance(classname, IFood.class);
        food.eat();
    }
}
```

此时的工厂设计模式将不再受限于指定的接口，可以为所有的接口提供实例化。



##### 反射与单例设计模式

单例设计模式的核心本质在于:

类内部的构造方法私有化,在类的外部产生类产生实例化对象之后通过static方法获取实例化对性行类中的结构调用

单例设计模式的最大特点是在整体的运行过程之中只允许产生一个实例化对象,当有了若干线程之后当前的程序就可以产生多个实例化对象，此时就不是单例设计模式。

问题造成的关键在于代码本身出现了不同步的情况，而要想解决的关键核心在于需要进行同步处理，使用 synchronized 类

```java
class Singleton {
    private volatile static Singleton instance;

    private Singleton() {
        System.out.println(Thread.currentThread().getName() + " 构造函数");
    }

    public void print() {
        System.out.println("test");
    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        for (int i = 0; i < 3; i++) {
            new Thread(() -> {
                Singleton.getInstance().print();
            }, "单例消费端：" + i).start();
        }
    }
}
```

java中使用到单例设计模式的地方：Runtime 类 、Pattern 类、Spring 框架

#### 反射与类操作

在反射机制的处理过程之中不仅仅只是一个实例化对象的处理操作，更多的情况下还有类的组成结构操作，任何一个类的基本组成结构：父类(父接口)、包、属性、方法(构造方法、普通方法)。

##### 反射获取类结构信息

一个类的基本信息主要包括类所在的包名称、父类的定义、父接口的定义

获取包名称：

```java
public Package getPackage()
```

获取继承父类：

```java
public Class <? superT > getSuperclass ()
```

获取实现父接口：

```java
public Class <?>[] getInterfaces ()
```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Class<Person> personClass = Person.class;
        System.out.println(personClass.getPackage());
        System.out.println(personClass.getSuperclass());
        Class<?>[] interfaces = personClass.getInterfaces();
        for (int i = 0; i < interfaces.length; i++) {
            System.out.println(interfaces[i]);
        }
    }
}
```

##### 反射调用构造方法

在一个类之中除了有继承的关系之外最为重要的操作就是类中的结构处理 ，而类中的构造里面首先需要观察的是构造法使用问题。

所有类的构造方法的获取都可以直接通过 Class  类来完成,该类中定义有如下的几种：

- 获取所有构造方法：

  ```java
  public Constructor<?>[] getDeclaredConstructors() throws SecurityException
  ```

- 获取指定构造方法

  ```java
  public Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes)
          throws NoSuchMethodException, SecurityException
  ```

- 获取所有构造方法

  ```java
  public Constructor<?>[] getConstructors() throws SecurityException
  ```

- 获取指定构造方法

  ```java
  public Constructor<T> getConstructor(Class<?>... parameterTypes)
          throws NoSuchMethodException, SecurityException
  ```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Class<Person> personClass = Person.class;
        Constructor<Person> constructor = personClass.getConstructor(String.class, int.class);
        Person zhangsan = constructor.newInstance("zhangsan", 19);
        System.out.println(zhangsan);
    }
}
```

虽然程序代码本身允许开发者调用有参构造处理，但从实际的开发来看所有的使用反射的类中最好提供有无参构实例化来达到统一性

##### 反射调用普通方法

- 获取全部方法

  ```java
  public Method[] getMethods()
                      throws SecurityException
  ```

- 获取指定方法

  ```java
  public Method getMethod(String name, Class<?>... parameterTypes)
                   throws NoSuchMethodException, SecurityException
  ```

- 获取本类全部方法

  ```java
  public Method[] getDeclaredMethods() throws SecurityException
  ```

- 获取本类指定方法

  ```java
  public Method getDeclaredMethod(String name, Class<?>... parameterTypes)
          throws NoSuchMethodException, SecurityException
  ```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Class<?> personClass = Class.forName("vo.Person");
        {//获取全部方法（包括父类方法）
            Method[] methods = personClass.getMethods();
            for (Method met : methods) {
                System.out.println(met);
            }
        }
        System.out.println("---------------");
        {//获取本类全部方法
            Method[] methods = personClass.getDeclaredMethods();
            for (Method met : methods) {
                System.out.println(met);
            }
        }
        System.out.println("---------------");
        {
            Object o = personClass.getDeclaredConstructor().newInstance();
            Method setAge = personClass.getDeclaredMethod("setAge", int.class);
            setAge.invoke(o, 18);
            System.out.println(personClass.getDeclaredMethod("getAge").invoke(o));
        }
    }
}
```

利用此类操作，整体的操作形式不会有任何的明确的类对象产生，一切都是依靠反射机制处理的，这样的处理避免了与某一个类的耦合问题。

##### 反射调用成员

```java
//获取全部本类成员
public Field[] getDeclaredFields() throws SecurityException
//获取本类指定成员
public Field getDeclaredField(String name)
//获取父类全部成员
public Field[] getFields() throws SecurityException
//获取父类指定成员
public Field getField(String name)
```

```java
//设置属性内容
public void set(Object obj, Object value) throws IllegalArgumentException, IllegalAccessException
//获取属性内容
public Object get(Object obj) throws IllegalArgumentException, IllegalAccessException
//解除封装
public void setAccessible(boolean flag)
```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Class<?> personClass = Class.forName("vo.Person");
        Object o = personClass.getConstructor().newInstance();
        Field name = personClass.getDeclaredField("name");
        name.setAccessible(true);
        name.set(o, "mingzi");
        System.out.println(name.get(o));
    }
}
```

通过一系列的分析可以发现，类之中的构造方法，成员属性都可以通过反射实现调用，但是对于成员的反射调用很少这样直接处理，大部分操作都应该通过 setter 和getter 处理，所以以上的代码只能够说是反射的特色，但是不具备有实际的使用能力，而对于 filed 类在实际开发之中只有一个方法最为常用

获取成员类型

```java
public Class<?> getType()
   System.out.println(name.getType().getSimpleName());
```

在以后开发中进行反射处理的时候，往往会利用filed与method类实现类中的setter 方法的调用。



#### 案例：反射与简单java类



#### ClassLoader类加载器

##### ClassLoader类加载器简介

在 Java 语言里面提供有一个系统的环境变量：CLASSPATH，这个环境属性的作用主要是在 JVM进程启动的时候进行类加载路径的定义，在JVM里面可以根据类加载器而后进行指定路径中类的加载，换一种说法找到了类的加载器就意味着找到了类的来源。

![](https://s2.loli.net/2022/06/01/Dtj4QHB6ERFh7Ms.png)

如果说现在要想获得类的加载器，那么一定要通过ClassLoader来获取，而要想获取ClassLoader类的对象，则必须利用Class类（反射的根源）实现。

方法：

```java
public ClassLoader getClassLoader();
```

当获取了 ClassLoader 之后还可以继续获取其父类的 ClassLoader 类对象: `

```java
public final ClassLoader getParent().
```

![](https://s2.loli.net/2022/06/01/rmueYtbvn6E4958.png)

当你获得了类加载器之后就可以利用类加载器来实现类的反射加载处理。

##### 自定义ClassLoader处理类

清楚了类加载器的功能之后就可以根据自身的需要来实现自定义的类加载器，自定义的类加载器其加载的顺序是在所有系统类加载器的最后。

系统类中的类加载器都是根据 CLASSPATH 路径进行类加载的，而如果有了自定义类的加载器，就可以由开发者任意指派类的加载位置。

![](https://s2.loli.net/2022/06/01/WVSFhi6EeOkNjwu.png)





#### 反射与代理设计模式

##### 静态代理设计模式

代理设计模式是在程序开发之中使用最多的设计模式，代理设计模式的核心是有真实业务实现类与代理业务实现类，并且代理类要完成比真实业务更多的处理操作。

所有的代理设计模式如果按照设计要求来讲，必须是基于接口的设计，也就是说需要首先定义出核心接口的组成，下面模拟一个消息发送的代理操作结构。

```java
interface IMessage {
    public void send();
}

class MessageReal implements IMessage {

    @Override
    public void send() {
        System.out.println("【发送消息】zhuweihao");
    }
}

class MessageProxy implements IMessage {
    private IMessage message;

    public MessageProxy(IMessage message) {
        this.message = message;
    }

    public boolean connect() {
        System.out.println("【消息代理】进行消息发送通道的连接");
        return true;
    }

    public void close() {
        System.out.println("【消息代理】关闭消息通道");
    }

    @Override
    public void send() {
        if (this.connect()) {
            this.message.send();
            this.close();
        }
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        MessageProxy messageProxy = new MessageProxy(new MessageReal());
        messageProxy.send();
    }
}
```

以上的操作代码是一个最为标准的代理设计，但是如果要进一步的去思考会发现客户端的接口与具体的子类产生的耦合问题，所以这样的操作如果从实际的开发来讲最好再引入工厂设计模式进行代理对象的获取。

以上的代理设计模式为静态代理设计，这种静态代理设计的特点在于：一个代理类只为一个接口服务，那么如果说现在准备出了 3000 个业务接口，则按照此种做法就意味着需要编写3000个代理类，并且这3000个代理类的操作形式类似。

所以现在需要解决的问题在于：如何可以让一个代理类满足于所有的业务接口。

##### 动态代理设计模式

通过静态代理设计模式的缺陷可以发现，最好的做法是为所有功能一致的业务操作接口提供有统一的代理处理操作，而这可以通过动态代理机制来实现，但是在动态代理机制里面需要考虑到如下几点问题：

- 不管是动态代理类还是静态代理类都一定要接收真实业务实现子类对象。
- 由于动态代理类不再与某一个具体的接口进行捆绑，所以应该可以动态获取类的接口信息。

![](https://s2.loli.net/2022/06/02/zaZR2hrTkvux4Ig.png)

在进行动态代理实现的操作之中，首先需要关注的就是一个 Invocation Handler 接口，这个接口规定了代理方法的执行。

这个代码由于要跨越到多个接口存在，所以方法名称由程序自己定义。

```JAVA
public interface InvocationHandler {
    /**
     * 代理方法调用，代理主题类里面执行的方法最终都是此方法
     * @param proxy要代理的对象
     * @param method要执行的接口方法名称
     * @param args 传递的参数
     * @return 某一个方法的返回值
     * @throws Throwable 方法调用时出现的错误继续向上抛出
     */
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
}
```

在进行动态代理设计的时候对于动态对象的创建是由JVM底层完成的，此时主要依靠的是 java.lang.reflect.Proxy 程序类，而这个程序类之中只提供有一个核心方法:

```JAVA
/**
* 获取动态对象
* @param loader 获取当前真实主题类的 ClassLoader
* @param interfaces 代理是围绕接口进行的，所以一定要获取真实主题类的接口信息
* @param h 代理处理的方法,(本类对象)
* @return
*/
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
```

![](https://s2.loli.net/2022/06/02/3BRkbA8VTxhPvgF.png)

在整个代理操作中，第一个操作Invocation handler，第二个核心类proxy。

在主类进行操作处理的时候，一定要接收的是业务的真实实现类，proxy 根据真实业务类对象创建代理对象。

主类进行操作用的是proxy返回的代理对象进行操作的，而这个代理对象最重要的特征是它的结构复合业务接口结构。

代理的对象不是直接去调用，它在调用的代理操作会找到 Invocation handler，方法 invoke()。

比如当调用对象的时候，业务接口中有个方法 fun():void，那么代理对象发出的指令也是对象.fun()。但是当调用时，会找到 invoke，而后invoke去调用业务接口方法，业务接口方法就是业务的实现子类，以上就是整体的操作流程。

```java
interface IMessage {
    public void send();
}

class MessageReal implements IMessage {

    @Override
    public void send() {
        System.out.println("【发送消息】zhuweihao");
    }
}

class ZHUProxy implements InvocationHandler {
    private Object target;//保存真实业务对象


    /**
     * 进行真实业务对象与代理业务对象之间的绑定处理
     * @param target 真实业务对象
     * @return Proxy 生成的业务对象
     */
    public Object bind(Object target) {
        this.target = target;
        return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("-----执行方法：" + method);
        Object returnData = null;
        if (this.connect()) {
            returnData = method.invoke(this.target, args);
            this.close();
        }
        return returnData;
    }

    public boolean connect() {
        System.out.println("【消息代理】进行消息发送通道的连接");
        return true;
    }

    public void close() {
        System.out.println("【消息代理】关闭消息通道");
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        IMessage message = (IMessage) new ZHUProxy().bind(new MessageReal());
        message.send();
    }
}
```



### Java类集框架

#### 类集框架简介

##### 类集框架简介

从 JDK 1.2 开始 Java 引入了类集开发框架，所谓的类集指的就是一套动态对象数实现方案，在实际开发之中没有任何一项的开发可以离开数组，但是传统的数组实现起来非常的繁琐，而且长度是其致命伤，正是因为长度问题，所以不可能大范围使用的，但是我们的开发又不可能离开数组，所以最初就只能依靠一些数据结构来实现动态数组，而其中最为重要的两个结构:链表，树，但是面对这些数据结构的实现又不得不面对如下问题:

数据结构的代码实现困难，对于一般的开发者是无法进行使用的;

对于链表或二叉树当进行更新处理的时候的维护是非常麻烦的;

对于链表或二叉树还需要尽可能保证其操作的性能。

所以从 JDK 1.2 开始 Java 引入了类集，主要就是对常见的数据结构进行完整的实现包装并且提供，一系列的接口与实现子类来帮助用户减少数据结构所带来的开发困难，但是最初的类集实现由于 Java 本身的技术对于数据的控制并不严格，全部采用了 Object 类型进行数据接收，而在 JDK 1.5  之后由于泛型技术的推广，所以类集本身也得到良好的改进，可以直接利用泛型来保存相同类型的数据，并且随着数据量的不断增加，从 JDK 1.8 开始类集中的实现算法也得到良好的性能提升。

在整个类集框架里面提供有如下的几个核心接口：Collection、 List、Set、Map、Iterator、 Enumeration、 Queue、ListIterator。

##### Collection接口简介

[Collection (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Collection.html)

Collection 是一个集合接口。 它提供了对集合对象进行基本操作的通用接口方法。Collection接口在Java 类库中有很多具体的实现。是list，set等的父接口。

java.util.Collection 是单值集合操作的最大的父接口，在该接口之中定义有所有的单值数据的处理操作

#### List集合

##### List接口简介

[List (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/List.html)

List 是 Collection 子接口，其最大的特点是允许保存有重复元素数据，该接口的定义如下:

```java
public interface List<E> extends Collection<E>
```

但是需要清楚的是 List 子接口对于 Collection 接口进行了方法扩充。

但是 List 本身依然属于一个接口，那么对于接口要想使用则一定要使用子类来完成定义，在 List 子接口中有三个常用子类：ArrayList、Vector、 LinkedList。

在使用 List 保存自定义类对象的时候如果需要使用到contains 、remove方法进行查询与删除处理时候，一定要保证之中已经成功的覆写了equals方法。

##### ArrayList子类

[ArrayList (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/ArrayList.html)

ArrayList 是一个可改变大小的数组.当更多的元素加入到ArrayList中时，其大小将会动态地增长。内部的元素可以直接通过get与set方法进行访问，因为ArrayList本质上就是一个数组。

ArrayList 是 List 使用最多的一个子类

```java
public class ArrayList<E>
extends AbstractList<E>
implements List<E>, RandomAccess, Cloneable, Serializable
```

![](https://s2.loli.net/2022/06/02/jZUlKXug3WEbhQ4.png)

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        ArrayList<String> strings = new ArrayList<>();
        strings.add("Hello");
        strings.add("Hello");
        strings.add("World");
        System.out.println(strings);
    }
}
```

通过本程序可以发现 List 存储的特征：

- 保存的顺序就是其存储顺序
- List 集合里面允许存在有重复数据

在以上的程序里而虽然实现了集合的输出，但是这种输出的操作是直接利用了每一个类提供的 toString()方法实现的，为了方便的进行输出处理，在 JDK 1.8 之后 Iterable 父接口之中定义有一个 forEach 方法，需要注意的是，此种输出并不是在正常开发情况下要考虑的操作形式。方法定义如下：

```java
default void forEach(Consumer<? super T> action)
```

ArrayIist 默认的构造只会使用默认的空数组，使用的时候才会开辟数组，默认的开辟长度为10

ArrayList自动扩大容量为原来的1.5倍（实现的时候，方法会传入一个期望的最小容量，若扩容后容量仍然小于最小容量，那么容量就为传入的最小容量。扩容的时候使用的Arrays.copyOf方法最终调用native方法进行新数组创建和数据拷贝）

##### LinkedList子类

[LinkedList (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/LinkedList.html)

LinkedList 是一个双链表，在添加和删除元素时具有比ArrayList更好的性能。但在get与set方面弱于ArrayList。

```java
public class LinkedList<E>
extends AbstractSequentialList<E>
implements List<E>, Deque<E>, Cloneable, Serializable
```

LinkedList 还实现了 Deque 接口，该接口比List提供了更多的方法，包括 offer()，peek()，poll()等.

##### Vector子类

[Vector (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Vector.html)

Vector 是一个原始古老的程序类，这个类是在 JDK1.0 的时候就提供的，而后到了JDK1.2 的时候由于许多的开发者已经习惯于使用 Vector，并且许多的系统也是基于Vector 实现的，考虑到其使用广泛性，所以类集框架将其保存了下来，并且让其多实现了一个 List 接口

```java
public class Vector<E>
extends AbstractList<E>
implements List<E>, RandomAccess, Cloneable, Serializable
```

Vector 类如果使用的是无参构造方法，则一定会默认开辟一个 10 个长度的数组，

 Vector和ArrayList在更多元素添加进来时会请求更大的空间。Vector每次请求其大小的双倍空间，而ArrayList每次对size增长50%。

Vector 和ArrayList类似，但属于强同步类。如果你的程序本身是线程安全的（thread-safe，没有在多个线程之间共享同一个集合/对象），那么使用ArrayList是更好的选择。

#### Set集合

[Set (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Set.html#iterator())

##### Set接口简介

Set 集合最大的特点就是不允许保存重复元素，其也是 Collection 子接口。

在 JDK1.9 以前 Set 集合与 Collection 集合的定义并无差别，Set 继续使用了Collection 接口中提供的方法进行操作，但是从 JDK1.9 之后，Set 集合也像 List 集合一样扩充了一些 static 方法。

```java
public interface Set<E> extends Collection<E>
```

需要注意的是 Set 并不像 List 集合那样扩充了许多的新方法，所以无法使用 List 集合中提供的 get 方法，也就是说它无法实现指定索引数据的获取

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Set<String> stringSet = Set.of("Hello", "World", "zhuweihao", "!");
        Iterator<String> stringIterator = stringSet.iterator();
        while (stringIterator.hasNext()){
            System.out.println(stringIterator.next());
        }
    }
}
```

Set 集合的常规使用形式一定是依靠子类进行实例化的，所以 Set 接口之中有两个常用子类：HashSet、TreeSet。

##### HashSet子类

[HashSet (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/HashSet.html)

HashSet 是 Set 接口里面使用最多的一个子类，其最大的特点就是保存的数据是无序的

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
```

![image-20220606095350770](https://s2.loli.net/2022/06/06/8WqKUDhEMLZv6xP.png)

HashSet子类的操作特点：不允许保存重复元素（Set 接口定义的），另外一点HashSet之中保存的数据是无序的。

##### TreeSet子类

[TreeSet (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/TreeSet.html)

Set 接口的另外一个子类就是 TreeSet，与 HashSet 最大的区别在于 TreeSet 集合里面所保存的数据是有序的。

```java
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
```

![image-20220606100329899](https://s2.loli.net/2022/06/06/epI34HgzsGOMokj.png)

当利用 TreeSet 保存数据的时候，所有的数据都将按照数据的升序进行自动排序处理。

```java
class People implements Comparable<People> {
    private String name;
    private int age;

    @Override
    public String toString() {
        return "People{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public People(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public int compareTo(People people) {
        if (this.age < people.age) {
            return -1;
        } else if (this.age > people.age) {
            return 1;
        } else {
            return this.name.compareTo(people.name);
        }
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        TreeSet<People> peopleTreeSet = new TreeSet<>();
        peopleTreeSet.add(new People("zhangsan", 19));
        peopleTreeSet.add(new People("lisi", 19));
        peopleTreeSet.add(new People("wangwu", 16));
        peopleTreeSet.add(new People("xiaobai", 23));
        Iterator<People> peopleIterator = peopleTreeSet.iterator();
        while (peopleIterator.hasNext()) {
            System.out.println(peopleIterator.next());
        }
    }
}
```

TreeSet 是利用了 Comparable 接口来确认重复数据的。

在使用自定义类对象进行比较处理的时候一定要将该类之中的所有属性都依次进行大小关系的匹配，否则如果某一个或某几个属性相同的时候它也会认为是重复数据

HashSet 判断重复元素的方式并不是利用 Comparable 接口完成的，它利用的是Object 类中提供的方法实现的

```java
//对象比较
public boolean equals(Object obj)
//对象编码
public native int hashCode();
```

在进行重复元素判断的时候首先利用 hashCode() 进行编码的匹配，如果该编码不存在则表示数据不存在，证明没有重复，如果该编码存在了，则进一步进行对象比较处理，如果发现重复了，则此数据是不允许保存的。

```java
@Override
public boolean equals(Object o) {
    if (this == o) {
        return true;
    }
    if (o == null || getClass() != o.getClass()) {
        return false;
    }
    People people = (People) o;
    return age == people.age && name.equals(people.name);
}

@Override
public int hashCode() {
    return Objects.hash(name, age);
}
```

在 Java 程序之中真正的重复元素的判断处理利用的就是 hashCode() 与 equals() 两个方法共同作用完成的，而只有在排序要求情况下 (TreeSet)  才会利用Comparable 接口来实现。

#### 集合输出

##### Iterator迭代输出

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        HashSet<People> peopleHashSet = new HashSet<>();
        peopleHashSet.add(new People("zhangsan", 19));
        peopleHashSet.add(new People("lisi", 19));
        peopleHashSet.add(new People("lisi", 19));
        peopleHashSet.add(new People("wangwu", 16));
        peopleHashSet.add(new People("xiaobai", 23));
        Iterator<People> peopleIterator1 = peopleHashSet.iterator();
        while (peopleIterator1.hasNext()) {
            People next = peopleIterator1.next();
            if ((new People("lisi", 19).equals(next))) {
                peopleIterator1.remove();
            } else {
                System.out.println(next);
            }
        }
        System.out.println(peopleHashSet.isEmpty());
        System.out.println(peopleHashSet.size());
    }
}
```

在进行迭代输出的时候如果使用了 Collection.remove()则会造成并发更新异常，导致程序删除出错，而此时只能使用 Iterator.remove()方法实现正常的删除处理。

##### LIstIterator双向迭代输出

使用 Iterator 进行的迭代输出操作有一个特点:只允许由前向后实现输出，而如果说你现在需要进行双向迭代处理，那么必须依靠 Iterator 的子接口: ListIterator

需要注意的是如果要想获取 ListIterator 接口对象 Collection 并没有定义相关的处理方法，但是 List 子接口有，也就是说这个输出接口是专门为 List 集合准备的。

![](https://s2.loli.net/2022/06/06/ZN5f2E3Sxu8OP94.png)

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        ArrayList<People> peopleArrayList = new ArrayList<>();
        peopleArrayList.add(new People("zhangsan", 19));
        peopleArrayList.add(new People("lisi", 19));
        peopleArrayList.add(new People("lisi", 19));
        peopleArrayList.add(new People("wangwu", 16));
        peopleArrayList.add(new People("xiaobai", 23));
        ListIterator<People> peopleIterator1 = peopleArrayList.listIterator();
        while (peopleIterator1.hasNext()) {
            People next = peopleIterator1.next();
            if ((new People("lisi", 19).equals(next))) {
                peopleIterator1.remove();
            } else {
                System.out.println(next);
            }
        }
        System.out.println(peopleArrayList.isEmpty());
        System.out.println(peopleArrayList.size());
        while (peopleIterator1.hasPrevious()) {
            System.out.println(peopleIterator1.previous());
        }
    }
}
```

##### Enumeration枚举输出

Enumeration是在 JDK 1.0 的时候就使用的输出接口，这个输出接口主要是为了Vector类提供输出服务的，一直到后续 JDK 的发展，Enumeration依然只为Vector一个类服务，如果要想获取 Enumeration接口对象，就必须依靠Vector类提供的方法

```java
public Enumeration<E> elements()
```

在 Enumeration 接口之中定义有两个操作方法:

判断是否有下一个元素: 

```java
public boolean hasMoreElements)
```

获取当前元素: 

```java
public E nextElement()
```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Vector<People> peopleVector = new Vector<>();
        peopleVector.add(new People("zhangsan", 19));
        peopleVector.add(new People("lisi", 19));
        peopleVector.add(new People("lisi", 19));
        peopleVector.add(new People("wangwu", 16));
        peopleVector.add(new People("xiaobai", 23));
        Enumeration<People> peopleEnumeration = peopleVector.elements();
        while (peopleEnumeration.hasMoreElements()) {
            People people = peopleEnumeration.nextElement();
            System.out.println(people);
        }
    }
}
```

由于该接口出现的时间比较长了，所以在一些比较早的开发过程之中，也有部分的方法只支持 Enumeration 输出操作。

但是随着类方法的不断完善，大部分的操作都直接利用 Iterator 实现了。

#### Map集合

##### Map接口简介

[Map (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Map.html)

在之前已经学习了Collection接口以及其对应的子接口，可以发现在Collection接口之中所保存的数据全部都只是单个对象，在数据结构里面除了可以进行单个对象的保存之外，实际上也可以进行二元偶对象的保存（key=value）的形式来存储，而存储偶对象的核心意义在于，需要通过key获取对应的value。

在开发里面：Collection 集合保存数据的目的是为了输出，Map 集合保存数据的目的是为了进行 key 的查找。

Map 接口是进行二元偶对象保存的最大父接口，该接口定义如下:

```java
public interface Map<K,V>
```

```java
V get(Object key);
V put(K key, V value);
Set<K> keySet();
Set<Map.Entry<K, V>> entrySet();
V remove(Object key);
boolean containsKey(Object key);
```



##### HashMap子类

[HashMap (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/HashMap.html)

HashMap 是 Map 接口之中最为常见的一个子类，该类的主要特点是无序存储

```java
public class HashMap<K,V>
extends AbstractMap<K,V>
implements Map<K,V>, Cloneable, Serializable
```

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        HashMap<String, Integer> map = new HashMap<>();
        map.put("one", 1);
        map.put("two", 2);
        map.put("three", 3);
        map.put(null, 0);
        map.put("zero", null);
        map.put("one", 11);
        System.out.println(map.get("one"));
        System.out.println(map.get(null));
        System.out.println(map.get("zero"));
        System.out.println(map.get("ten"));
    }
}
```

通过代码可以发现，通过HashMap实例化的Map接口可以针对key或value保存null的数据，

同时也可以发现即便保存数据的key重复，那么也不会出现错误，而是出现内容的替换。

但是对于Map接口中提供的put()方法本身是提供有返回值的，返回值是在重复key的情况下返回旧的value，key不重复返回null。

HashMap的扩容问题：

https://hollischuang.github.io/toBeTopJavaer/#/basics/java-basic/hashmap-capacity



##### LinkedHashMap子类

[LinkedHashMap (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/LinkedHashMap.html)

HashMap 虽然是 Map 集合最为常用的一个子类，但是其本身所保存的数据都是无序的（有序与否对 Map 没有影响)，如果现在希望 Map 集合之中保存的数据的顺序为其增加顺序，则就可以更换子类为 LinkedHashMap（基于链表实现的)。 

既然是链表保存，所以一般在使用 LinkedHashMap 类的时候往往数据量都不要特别大，因为会造成时间复杂度攀升。

使用 LinkedHashMap 进行存储之后所有数据的保存顺序为添加顺序

##### Hashtable子类

[Hashtable (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Hashtable.html)

Hashtable 类是从 JDK1.0 的时候提供的，与 Vector、Enumeration  属于最早的一批动态数组的实现类，后来为了将其继续保存下来所以让其多实现了一个 Map 接口

```java
public class Hashtable<K,V>
extends Dictionary<K,V>
implements Map<K,V>, Cloneable, Serializable
```

 Hashtable 里面进行数据存储的时候设置的 key或 value 都不允许为 null，否则会出现 NullPointerException 异常。

##### Map.Entry接口

[Map.Entry (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Map.Entry.html)

虽然已经清楚了整个的 Map 集合的基本操作形式，但是依然需要有一个核心的问题要解决，Map 集合里面是如何进行数据存储的?对于 List 而言（LinkedList 子类）依靠的是链表的形式实现的数据存储，

那么在进行数据存储的时候一定要将数据保存在一个Node节点之中，虽然在HashMap里面也可以见到 Node 类型定义，通过源代码定义可以发现，HashMap类中的 Node内部类本身实现了 Map.Entry 接口。

```java
static class Node<K,V> implements Map.Entry<K,V>{}
```

所以可以得出结论:所有的key和 value的数据都被封装在Map.Entry接口之中，而此接口定义如下:

```java
public static interface Map.Entry<K.V>
```

并且在这个内部接口里面提供有两个重要的操作方法：

```java
public final K getKey()
public final V getValue()
```

从 JDK1.9 之后，Map 接口里面追加有一个新的方法：

```java
public static <K,V> Map.Entry<K,V> entry(K k, V v)
```

Map.Entry的主要作用就是作为一个Key 和 Value 的包装类型使用

##### Iterator输出Map集合

对于集合的输出而言，最标准的做法就是利用 Iterator 接口来完成，但是需要明确一点的是在 Map 集合里面并没有一个方法可以直接返回 Iterator 接口对象。

所以这种情况下就必须分析不直接提供 Iterator 接口实例化的方法的原因，下面对Collection 与 Map 集合的存储结构进行一个比较。

发现在 Map 集合里面保存的实际上是一组 Map.Entry 接口对象（里面包装的是 Key 与 Value)，所以整个来讲 Map 依然实现的是单值的保存，这样在 Map 集合里面提供有一个方法

```java
public Set<Map.Entry<K,V>> entrySet()
```

将全部的Map 集合转为 Set 集合。

![image-20220606174555275](https://s2.loli.net/2022/06/06/9HiAeNP5ujOdfca.png)

经过分析可以发现如果要想使用 Iterator 实现 Map 集合的输出则必须按照如下步骤处理：

利用 Map 接口中提供的 entrySet() 方法将 Map 集合转为 Set集合

利用 Set 接口中的 iterator() 方法将 Set 集合转为 Iterator 接口实例

利用 Iterator 进行迭代输出获取每一组的 Map.Entry 对象，随后通过 getKey() 与getValue() 获取数据。

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        HashMap<String, Integer> map = new HashMap<>();
        map.put("one", 1);
        map.put("two", 2);
        map.put("three", 3);
        map.put(null, 0);
        map.put("zero", null);
        map.put("one", 11);
        Set<Map.Entry<String, Integer>> entrySet = map.entrySet();
        Iterator<Map.Entry<String, Integer>> iterator = entrySet.iterator();
        while (iterator.hasNext()) {
            Map.Entry<String, Integer> next = iterator.next();
            System.out.println(next.getKey() + ":" + next.getValue());
        }
    }
}
```

##### 自定义Map的Key类型

在使用 Map 集合的时候可以发现对于 Key 和  Value 的类型实际上都可以由使用者任意决定，那么也就意味着可以使用自定义的类来进行Key类型的设置

对于自定义 Key 类型所在的类中一定要覆写 hashCode() 与 equals() 方法，否则无法查找到。

```java
class People implements Comparable<People> {
    private String name;
    private int age;

    @Override
    public String toString() {
        return "People{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public People(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        People people = (People) o;
        return age == people.age && name.equals(people.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }

    @Override
    public int compareTo(People people) {
        if (this.age < people.age) {
            return -1;
        } else if (this.age > people.age) {
            return 1;
        } else {
            return this.name.compareTo(people.name);
        }
    }
}

public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        HashMap<People, String> map = new HashMap<>();
        map.put(new People("zhangsan", 19), "张三");
        System.out.println(map.get(new People("zhangsan", 19)));
    }
}
```

在实际的开发之中对于 Map 集合的 Key常用的类型就是：String、Long、Integer，尽量使用系统类。

如果在进行 HashMap 进行数据操作的时候出现了 Hash  冲突（Hash码相同），为了保证程序的正常执行，会在冲突的位置上将所有 Hash 冲突的内容转为链表保存。

![](https://s2.loli.net/2022/06/06/ABmsMl3b7yD4u9W.png)



#### 集合工具类

##### Stack栈

[Stack (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Stack.html)

```java
public class Stack<E> extends Vector<E>
```



```java
//入栈
public E push(E item)
//出栈，弹出栈顶元素并返回
public synchronized E pop() 
//返回栈顶元素
public synchronized E peek()
//返回元素在栈中的位置
public synchronized int search(Object o)
```

##### Queue队列

[Queue (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Queue.html)

```java
public interface Queue<E> extends Collection<E>
```

队列实现可以使用我们的 LinkedList 的子类来完成

```java
Queue<String> strings = new LinkedList<>();
```

排序队列

```java
public class PriorityQueue<E> extends AbstractQueue<E> implements java.io.Serializable
```

##### properties属性操作

国际化程序讲解的资源文件（*.properties），这类文件的存储结构是按照”key=value‘’ 的形式，这种结构的保存形式与Map集合很相似，但是唯一的区别在于其保存的内容只能够是字符串，所以为了可以方便描述属性的定义。

java.util  包里面提供有 Properties  类型，此类是 Hashtable 的子类。

```java
public class Properties extends Hashtable<Object,Object>
```

```java
public synchronized Object setProperty(String key, String value)
public String getProperty(String key)
public String getProperty(String key, String defaultValue)
public void list(PrintStream out)
```

```JAVA
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        Properties properties = new Properties();
        properties.setProperty("baidu", "www.baidu.com");
        properties.setProperty("zhuweihao", "www.zhuweihao.com");
        System.out.println(properties.getProperty("baidu"));
        System.out.println(properties.getProperty("sina"));
        System.out.println(properties.getProperty("sina", "DEFAULT"));
        properties.store(new FileOutputStream(new File("F:" + File.separator + "info.properties")), "zhushi");
    }
}
```

之所以会提供有 Properties 类,有一个最重要的功能是它可以通过输出流输出属性，也可以使用输入流读取属性内容，而 Map 没有。

Properties 往往用于读取配置资源的信息

##### Collections工具类

[Collections (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Collections.html)

Collections是 Java 提供的一组集合数据的操作工具类，利用它可以实现各个集合的操作。

```java
public class Collections
extends Object
```

![image-20220608103003440](https://s2.loli.net/2022/06/08/ESmFoqKTWIfDGBA.png)

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        ArrayList<String> strings = new ArrayList<>();
        Collections.addAll(strings, "Hello", " World", "!");
        System.out.println(strings);
        Collections.reverse(strings);
        System.out.println(strings);
        System.out.println(Collections.binarySearch(strings, "Hello"));
    }
}
```

大部分情况下的使用没有太多复杂要求，更多情况就是利用集合保存数据去进行输出或查询

#### Stream数据流

##### Stream基本操作

[java.util.stream (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/package-summary.html)

从 JDK1.8 开始，由于已经到了大数据时代，所以在类集里面也支持有数据的流式分析处理操作，为此就专门供一个 Stream 的接口，同时在 Collection 接口里面也提供有为此接口实例化的方法。

```JAVA
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        List<String> strings = new ArrayList<>();
        Collections.addAll(strings, "java", "javascript", "python", "ruby", "go");
        Stream<String> stream = strings.stream();
        long j = stream.filter((s) -> s.toUpperCase(Locale.ROOT).contains("J")).count();
        System.out.println(j);
    }
}
```

以上的程序只是实现了一些最基础的数据的个数统计，而更多条件下，我们可能需要的是获取里面的满足条件数据的内容，所以此时可以进行实现数据的采集操作。

```java
public class JavaAPIDemo {
    public static void main(String[] args) throws Exception {
        List<String> strings = new ArrayList<>();
        Collections.addAll(strings, "java", "javascript", "python", "ruby", "go");
        Stream<String> stream = strings.stream();
        List<String> j = stream.filter((s) -> s.toUpperCase(Locale.ROOT).contains("J")).collect(Collectors.toList());
        System.out.println(j);
    }
}
```

将满足条件的数据收集起来转为 List 集合

Stream 的操作主要是利用自身的特点实现数据的分析处理操作。

##### MapReduce基础模型

在进行数据分析的处理之中有一个最重要的基础模型：MapReduce 模型，对于这个模型一共是分为两个部分：Map 处理部分，Reduce 分析部分，在进行数据分析之前必须要对数据进行合理的处理，而后才可以做统计分析操作。



### Java网络编程

##### 网络编程简介

C/S (Client / Server、客户端与服务器端):要开发出两套程序，一套程序为客户端，另外一套程序为服务器端，如果现在服务器端发生了改变之后客户端也应该进行更新处理。C/S 程序模型，其分为两种开发:TCP(可靠的数据连接)、UDP(不可靠的数据连接）；

这种开发可以由开发者自定义传输协议，并且使用一些比较私密的端口。所以安全性是比较高的，但是开发与维护成本比较高;

B/S (Browse/Server、浏览器与服务器端):只开发一套服务器端的程序,而后利用浏览器作为客户端进行访问，这种开发与维护的成本较低（只有一套程序) 。

但是由于其使用的是公共的HTTP协议并且使用的公共的 80 端口，所以其安全性相对较差，现在的开发基本上以“B/S”结构为主。

##### Echo程序模型

TCP 的程序开发是网络程序的最基本的开发模型，其核心的特点是使用两个类实现数据的交互处理: ServerSocket  (服务器端)、Socket（客户端）

![](https://s2.loli.net/2022/06/08/NcPF5sd9nha7ekD.png)

```java
public class EchoServer {
    public static void main(String[] args) throws Exception {
        //设置服务器端的监听端口
        ServerSocket serverSocket = new ServerSocket(9999);
        System.out.println("等待客户连接");
        //有客户端发出请求,接收请求
        Socket client = serverSocket.accept();
        Scanner scanner = new Scanner(client.getInputStream());
        scanner.useDelimiter("\n");
        PrintStream printStream = new PrintStream(client.getOutputStream());
        boolean flag = true;
        while (flag) {
            if (scanner.hasNext()) {
                String next = scanner.next().trim();
                if ("byebye".equalsIgnoreCase(next)) {
                    System.out.println("byebyebye......");
                    flag = false;
                } else {
                    printStream.println(next);
                }
            }
        }
        client.close();
        serverSocket.close();
    }
}
```

```java
public class EchoClient {
    private static final BufferedReader KEYBOARD_INPUT = new BufferedReader(new InputStreamReader(System.in));

    public static void main(String[] args) throws Exception {
        Socket client = new Socket("localhost", 9999);
        Scanner scanner = new Scanner(client.getInputStream());
        scanner.useDelimiter("\n");
        PrintStream printStream = new PrintStream(client.getOutputStream());
        boolean flag = true;
        while (flag) {
            String string = getString("请输入要发送的内容").trim();
            printStream.println(string);
            if (scanner.hasNext()) {
                System.out.println(scanner.next());
            }
            if ("byebye".equalsIgnoreCase(string)) {
                flag = false;
            }
        }
        scanner.close();
        printStream.close();
        client.close();
    }

    public static String getString(String string) throws Exception {
        System.out.println(string);
        String s = KEYBOARD_INPUT.readLine();
        return s;
    }
}
```

##### UDP程序

实现UDP程序需要两个类: DatagramPacket(数据内容)、DatagramSocket (网络发送与接收) 数据报就好比发送的短消息一样，客户端是否接收到与发送者无关。



### Java数据库编程

##### JDBC简介

所以任何一门编程语言如果要想发展，那么必须对数据库的开发有所支持,同样，Java从最初的时代开始就一直支持着数据库的开发标准—— JDBC(Java Database Connectivity Java 数据库连接)，JDBC 本质上来讲并不属于一个技术，它属于一种服务。

而所有服务的特征，必须按照指定的套路来进行操作。在 Java 里面专门为 JDBC 提供有一个模块 （java.sql)，里面核心的一个开发包（ java.sql)，在 JDBC 里面核心的组成就是 DriverManager 类以及若干接口（Connection、Statement、PreparedStatement、ResultSet)。

```java
public class JDBCDemo {
    private static final String DATABASE_DRIVER = "com.mysql.cj.jdbc.Driver";
    private static final String DATABASE_URL = "jdbc:mysql://localhost:3306/test";
    private static final String DATABASE_USER = "root";
    private static final String DATABASE_PASSWORD = "03283X";
    public static void main(String[] args) throws Exception {
        //向容器之中加载数据库驱动程序
        Class.forName(DATABASE_DRIVER);
        //每一个connection接口对象描述的就是一个用户连接
        Connection connection = DriverManager.getConnection(DATABASE_URL, DATABASE_USER, DATABASE_PASSWORD);
        System.out.println(connection);
        //数据库资源释放
        connection.close();
    }
}
```

maven项目需要在pom.xml文件中添加依赖配置，无需手动添加驱动jar包

```xml
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>8.0.29</version>
</dependency>
```

![image-20220609094930064](https://s2.loli.net/2022/06/09/A8v3f45kTXyFOC6.png)

通过数据库连接的流程可以发现，JDBC是通过工厂的设计模式实现的，DriverManager是一个工厂，不同的数据库生产商利用JDBC提供的标准（接口）实现各自的数据库处理操作。

##### Statement接口

[Statement (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.sql/java/sql/Statement.html)

java.sql.Statement 是 JDBC 之中提供的数据库的操作接口，利用其可以实现数据的更新与查询的处理操作。

```java
public interface Statement extends Wrapper, AutoCloseable
```

该接口是 AutoCloseable 子接口，所以可以得出结论：每一次进行数据库操作完之后都应该关闭 Statement 操作，即：一条 SQL 的执行一定是一个 Statement 接口对象。

想要获取 Statement 接口对象，那么必须依靠 Connection 接口提供的方法：

```java
Statement createStatement() throws SQLException;
```

数据更新处理（INSERT、UPDATE、DELETE）：

```java
int executeUpdate(String sql) throws SQLException;
```

数据查询处理（SELECT、统计查询、复杂查询）：

```java
ResultSet executeQuery(String sql) throws SQLException;
```

这两个数据库的操作方法里面都需要接收 SQL 的字符串，也就是说 Statement 接口可以直接使用 SQL 语句实现开发。

```sql
create table news(
	nid		INT,
    title 	VARCHAR(30),
    reader 	INT,
    price	FLOAT,
    content	int,
    pubdate	DATE,
    CONSTRAINT	pk_nid	PRIMARY KEY(nid)
);
```

```java
public class JDBCDemo {
    private static final String DATABASE_DRIVER = "com.mysql.cj.jdbc.Driver";
    private static final String DATABASE_URL = "jdbc:mysql://localhost:3306/test";
    private static final String DATABASE_USER = "root";
    private static final String DATABASE_PASSWORD = "03283X";

    public static void main(String[] args) throws Exception {
        //向容器之中加载数据库驱动程序
        Class.forName(DATABASE_DRIVER);
        //每一个connection接口对象描述的就是一个用户连接
        Connection connection = DriverManager.getConnection(DATABASE_URL, DATABASE_USER, DATABASE_PASSWORD);
        Statement statement = connection.createStatement();
        String sql = "insert into news(title,reader,price,content,pubdate) values('news',19,3.4,59,'1999-09-03');";
        int i = statement.executeUpdate(sql);
        System.out.println("更新的行数:" + i);
        String update = "update news set title='瞎改的',reader=78 where nid=2";
        int i1 = statement.executeUpdate(update);
        System.out.println("第二次更新受影响的行数" + i1);
        String select = "SELECT * FROM news";
        ResultSet resultSet = statement.executeQuery(select);
        System.out.println(resultSet.toString());
        while (resultSet.next()) {
            int nid = resultSet.getInt("nid");
            String title = resultSet.getString("title");
            int reader = resultSet.getInt("reader");
            double price = resultSet.getDouble("price");
            int content = resultSet.getInt("content");
            Date pubdate = resultSet.getDate("pubdate");
            System.out.println(nid + " " + title + " " + reader + " " + price + " " + content + " " + pubdate);
        }
        System.out.println(connection);
        //数据库资源释放
        connection.close();
    }
}
```

需要注意的是，ResultSet 对象是保存在内存之中的，如果说你查询数据的返回结果过大，那么程序也将出现问题。

##### PreparedStatement接口

[PreparedStatement (Java SE 17 & JDK 17) (oracle.com)](https://docs.oracle.com/en/java/javase/17/docs/api/java.sql/java/sql/PreparedStatement.html)

虽然 Statement 可以操作数据库，但是其在操作的过程之中并不是那么的方便。

而它最大的弊端：需要进行 SQL 语句的拼凑。

为了解决 Statement 接口存在的 SQL 执行问题，所以在 java.sql 包里面又提供有一个Statement 子接口：PrearedStatement,，这个接口最大的好处是可以编写正常的 SQL（数据不再和 SQL 语法混合在一起），同时利用占位符的形式，在 SQL 正常执行完毕后可以进行数据的设置。

```java
//查询
ResultSet executeQuery() throws SQLException;
//更新
int executeUpdate() throws SQLException;
```

![image-20220609134652409](https://s2.loli.net/2022/06/09/6HQXNWi5xwLlfJ2.png)

```java
public class JDBCDemo {
    private static final String DATABASE_DRIVER = "com.mysql.cj.jdbc.Driver";
    private static final String DATABASE_URL = "jdbc:mysql://localhost:3306/test";
    private static final String DATABASE_USER = "root";
    private static final String DATABASE_PASSWORD = "03283X";

    public static void main(String[] args) throws Exception {
        //向容器之中加载数据库驱动程序
        Class.forName(DATABASE_DRIVER);
        //每一个connection接口对象描述的就是一个用户连接
        Connection connection = DriverManager.getConnection(DATABASE_URL, DATABASE_USER, DATABASE_PASSWORD);
        String title = "hahahah";
        int reader = 89;
        double price = 78.32;
        int content = 23;
        Date pubdate = new Date();
        String sql = "insert into news(title,reader,price,content,pubdate) values(?,?,?,?,?);";
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        preparedStatement.setString(1, title);
        preparedStatement.setInt(2, reader);
        preparedStatement.setDouble(3, price);
        preparedStatement.setInt(4, content);
        preparedStatement.setDate(5, new java.sql.Date(pubdate.getTime()));
        int i = preparedStatement.executeUpdate();
        System.out.println(i);
        //数据库资源释放
        connection.close();
    }
}
```

```java
public class JDBCDemo {
    private static final String DATABASE_DRIVER = "com.mysql.cj.jdbc.Driver";
    private static final String DATABASE_URL = "jdbc:mysql://localhost:3306/test";
    private static final String DATABASE_USER = "root";
    private static final String DATABASE_PASSWORD = "03283X";

    public static void main(String[] args) throws Exception {
        //向容器之中加载数据库驱动程序
        Class.forName(DATABASE_DRIVER);
        //每一个connection接口对象描述的就是一个用户连接
        Connection connection = DriverManager.getConnection(DATABASE_URL, DATABASE_USER, DATABASE_PASSWORD);
        String select = "SELECT * FROM news where reader=?";
        PreparedStatement preparedStatement = connection.prepareStatement(select);
        preparedStatement.setInt(1, 19);
        ResultSet resultSet = preparedStatement.executeQuery();
        while (resultSet.next()) {
            int nid = resultSet.getInt("nid");
            String title = resultSet.getString("title");
            int reader = resultSet.getInt("reader");
            double price = resultSet.getDouble("price");
            int content = resultSet.getInt("content");
            Date pubdate = resultSet.getDate("pubdate");
            System.out.println(nid + " " + title + " " + reader + " " + price + " " + content + " " + pubdate);
        }
        //数据库资源释放
        connection.close();
    }
}
```

















