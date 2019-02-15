# JAVA 8新特性

## 方法引用

方法引用提供了非常有用的语法，可以直接引用已有Java类或对象（实例）的方法或构造器。与lambda联合使用，方法引用可以使语言的构造更紧凑简洁，减少冗余代码。

方法引用通过方法的名字来指向一个方法。方法引用使用一对冒号 :: 

在Java8中，我们可以直接通过方法引用来**简写**lambda表达式中已经存在的方法。

```
Arrays.sort(stringsArray, String::compareToIgnoreCase);
```

这种特性就叫做方法引用(Method Reference)。方法引用的标准形式是:`类名::方法名`

有以下四种形式的方法引用:

| 类型               | 示例                                   |
| :--------------- | :----------------------------------- |
| 引用静态方法           | ContainingClass::staticMethodName    |
| 引用某个对象的实例方法      | containingObject::instanceMethodName |
| 引用某个类型的任意对象的实例方法 | ContainingType::methodName           |
| 引用构造方法           | ClassName::new                       |

[代码参考](https://www.cnblogs.com/JohnTsai/p/5806194.html) 

## 接口默认方法

Java 8用默认方法与静态方法这两个新概念来扩展接口的声明。**默认方法**使接它可以包含实现代码，但与传统的接口又有些不一样，它允许在已有的接口中添加新方法，而同时又保持了与旧版本代码的兼容性。

默认方法与抽象方法不同之处在于抽象方法必须要求实现，但是默认方法则没有这个要求。相反，每个接口都必须提供一个所谓的默认实现，这样所有的接口实现者将会默认继承它（如果有必要的话，可以覆盖这个默认实现）。

接口可以声明（并且可以提供实现）**静态方法** 。在JVM中，默认方法的实现是非常高效的，并且通过字节码指令为方法调用提供了支持。默认方法允许继续使用现有的Java接口，而同时能够保障正常的编译过程。

【注意】：在声明一个默认方法前，请仔细思考是不是真的有必要使用默认方法，因为默认方法会带给程序歧义，并且在复杂的继承体系中容易产生编译错误。

## 访问接口的默认方法

## Optional类

Optional 类是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。

Optional 是个容器：它可以保存类型T的值，或者仅仅保存null。Optional提供很多有用的方法，这样我们就不用显式进行空值检测。

Optional 类的引入很好的解决空指针异常。

[用法参考](http://www.runoob.com/java/java8-optional-class.html) 

## Stream

Java 8 API添加了一个新的抽象称为流Stream，可以让你以一种声明的方式处理数据。

Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。

Stream API可以极大提高Java程序员的生产力，让程序员写出高效率、干净、简洁的代码。

这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。

元素流在管道中经过中间操作（intermediate operation）的处理，最后由最终操作(terminal operation)得到前面处理的结果。

> Stream（流）是一个来自数据源的元素队列并支持聚合操作
>
> - 元素是特定类型的对象，形成一个队列。 Java中的Stream并不会存储元素，而是按需计算。
> - **数据源** 流的来源。 可以是集合，数组，I/O channel， 产生器generator 等。
> - **聚合操作** 类似SQL语句一样的操作， 比如filter, map, reduce, find, match, sorted等。
> - **Pipelining**: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)。
> - **内部迭代**： 以前对集合遍历都是通过Iterator或者For-Each的方式, 显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了内部迭代的方式， 通过访问者模式(Visitor)实现。

**生成流**

- **stream()** − 为集合创建串行流。
- **parallelStream()** − 为集合创建并行流。

```
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
```

**forEach**

```
Random random = new Random();
random.ints().limit(10).forEach(System.out::println);// 'forEach' 用于迭代流中的每个数据
```

**map**

```
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
// 获取对应的平方数  map 方法用于映射每个元素到对应的结果
List<Integer> squaresList = numbers.stream().map(i -> i*i).distinct().collect(Collectors.toList());
```

flatmap  : flatMap 方法可用 Stream 替换值，然后将多个 Stream 连接成一个 Stream

```
List<Integer> together = Stream.of(asList(1, 2), asList(3, 4))
								.flatMap(numbers -> numbers.stream())
								.collect(toList());
```

**filter** 

```
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
// 获取空字符串的数量   filter 方法用于通过设置的条件过滤出元素
int count = strings.stream().filter(string -> string.isEmpty()).count();
```

**sorted**

```
Random random = new Random();
random.ints().limit(10).sorted().forEach(System.out::println);//sorted 方法用于对流进行排序
```

**并行（parallel）程序**

```
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
// 获取空字符串的数量    parallelStream 是流并行处理程序的代替方法
int count = strings.parallelStream().filter(string -> string.isEmpty()).count();
```

**Collectors**

Collectors 类实现了很多归约操作，例如将流转换成集合和聚合元素。Collectors 可用于返回列表或字符串。由 Stream 里的值生成一个列表，是一个及早求值操作。

```
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList()); 
System.out.println("筛选列表: " + filtered);
String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(", "));
System.out.println("合并字符串: " + mergedString);
```

**统计**

```
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5); 
IntSummaryStatistics stats = numbers.stream().mapToInt((x) -> x).summaryStatistics();
 
System.out.println("列表中最大的数 : " + stats.getMax());
System.out.println("列表中最小的数 : " + stats.getMin());
System.out.println("所有数之和 : " + stats.getSum());
System.out.println("平均数 : " + stats.getAverage());
```

[用法参考](http://www.importnew.com/11908.html#streams) 

---

内部迭代与外部迭代

for,foreach,iterator 属于外部迭代；stream属于内部迭代

判断一个操作是惰性求值还是及早求值很简单：只需看它的返回值。如果返回值是 Stream，那么是惰性求值；如果返回值是另一个值或为空，那么就是及早求值。整个过程和建造者模式有共通之处。建造者模式使用一系列操作设置属性和配置，最后调用一个 build 方法，这时，对象才被真正创建。

## 函数式接口

函数式接口(Functional Interface)就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。默认方法与静态方法并不影响函数式接口的契约，可以任意使用;函数式接口里允许定义 java.lang.Object 里的 public 方法.

函数式接口可以被隐式转换为 lambda 表达式。函数式接口可以使用Lambda表达式，lambda表达式会被匹配到这个抽象方法上 

> 我们可以将lambda表达式当作任意只包含一个抽象方法的接口类型，确保你的接口一定达到这个要求，你只需要给你的接口添加 @FunctionalInterface 注解，编译器如果发现你标注了这个注解的接口有多于一个抽象方法的时候会报错的

**提醒：**加不加 **@FunctionalInterface** 对于接口是不是函数式接口没有影响，该注解只是提醒编译器去检查该接口是否仅包含一个抽象方法

jdk1.8新增的函数接口：

```
java.util.function
```

[使用参考](http://www.runoob.com/java/java8-functional-interfaces.html)    [常见函数式接口](http://www.cnblogs.com/heimianshusheng/p/5672641.html)

## Lambda表达式

Lambda 表达式，也可称为闭包，它是推动 Java 8 发布的最重要新特性。

Lambda 允许把函数作为一个方法的参数（函数作为参数传递进方法中）。使用 Lambda 表达式可以使代码变的更加简洁紧凑。

lambda 表达式的语法格式如下:

```
(parameters) -> expression
或
(parameters) ->{ statements; }
```

lambda表达式的重要特征:

- **可选类型声明：**不需要声明参数类型，编译器可以统一识别参数值。
- **可选的参数圆括号：**一个参数无需定义圆括号，但多个参数需要定义圆括号。
- **可选的大括号：**如果主体包含了一个语句，就不需要使用大括号。
- **可选的返回关键字：**如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值。

#### 变量作用域

lambda 表达式只能引用标记了 final 的外层局部变量，这就是说不能在 lambda 内部修改定义在域外的局部变量，否则会编译错误。

也可以直接在 lambda 表达式中访问外层的局部变量.

lambda 表达式的局部变量可以不用声明为 final，但是必须不可被后面的代码修改（即隐性的具有 final 的语义）.

在 Lambda 表达式当中不允许声明一个与局部变量同名的参数或者局部变量。

**代码示例**

```
Arrays.asList( "a", "b", "d" ).forEach( e -> System.out.println( e ) );
```

> 在匿名内部类中，如果需要引用它所在方法里的变量，需要将变量声明为final；java 8 可引用非final变量，但是改变量在既成事实上必须是final.虽然无需将变量声明为final，但在Lambda表达式中，也无法用作非终态变量。如果坚持用作非终态变量，编译器就会报错。
>
> 既成事实上的 final 是指只能给该变量赋值一次。换句话说，Lambda 表达式引用的是值，而不是变量。
>
> Lambda 表达式也被称为闭包：未赋值的变量与周边环境隔离起来，进而被绑定到一个特定的值
>
> Lambda 表达式都是静态类型的。

## Java8 Base64

Java 8 内置了 Base64 编码的编码器和解码器。

Base64工具类提供了一套静态方法获取下面三种BASE64编解码器：

- **基本：**输出被映射到一组字符A-Za-z0-9+/，编码不添加任何行标，输出的解码仅支持A-Za-z0-9+/。
- **URL：**输出映射到一组字符A-Za-z0-9+_，输出是URL和文件。
- **MIME：**输出隐射到MIME友好格式。输出每行不超过76字符，并且使用'\r'并跟随'\n'作为分割。编码输出最后没有行分割。

主要是两个内嵌类：

- **static class Base64.Decoder**  该类实现一个解码器用于，使用 Base64 编码来解码字节数据。
- **static class Base64.Encoder**   该类实现一个编码器，使用 Base64 编码来编码字节数据。



## 新的时间日期API 

### 本地化时间API

**Local(本地)** − 简化了日期时间的处理，没有时区的问题。

LocalDate/LocalTime 和 LocalDateTime 类可以在处理时区不是必须的情况。

```
	// 获取当前的日期时间
    LocalDateTime currentTime = LocalDateTime.now();
    
    LocalDate date1 = currentTime.toLocalDate();
    Month month = currentTime.getMonth();
    int day = currentTime.getDayOfMonth();
    int seconds = currentTime.getSecond();
    
    LocalDateTime date2 = currentTime.withDayOfMonth(10).withYear(2012);
    
    // 12 december 2014
    LocalDate date3 = LocalDate.of(2014, Month.DECEMBER, 12);
    // 22 小时 15 分钟
    LocalTime date4 = LocalTime.of(22, 15);      
    // 解析字符串
    LocalTime date5 = LocalTime.parse("20:15:30");    
```

### 使用时区的日期时间API

**Zoned(时区)** − 通过制定的时区处理日期时间。

```
	// 获取当前时间日期
    ZonedDateTime date1 = ZonedDateTime.parse("2015-12-03T10:15:30+05:30[Asia/Shanghai]");
    System.out.println("date1: " + date1);
        
    ZoneId id = ZoneId.of("Europe/Paris");
    System.out.println("ZoneId: " + id);
        
    ZoneId currentZone = ZoneId.systemDefault();
    System.out.println("当期时区: " + currentZone);
```



## Lambda表达式常见应用

#### 简单应用

```
public class Java8Tester {
   public static void main(String args[]){
      Java8Tester tester = new Java8Tester();
        
      // 类型声明
      MathOperation addition = (int a, int b) -> a + b;
        
      // 不用类型声明
      MathOperation subtraction = (a, b) -> a - b;
        
      // 大括号中的返回语句
      MathOperation multiplication = (int a, int b) -> { return a * b; };
        
      // 没有大括号及返回语句
      MathOperation division = (int a, int b) -> a / b;
        
      System.out.println("10 + 5 = " + tester.operate(10, 5, addition));
      System.out.println("10 - 5 = " + tester.operate(10, 5, subtraction));
      System.out.println("10 x 5 = " + tester.operate(10, 5, multiplication));
      System.out.println("10 / 5 = " + tester.operate(10, 5, division));
        
      // 不用括号
      GreetingService greetService1 = message ->
      System.out.println("Hello " + message);
        
      // 用括号
      GreetingService greetService2 = (message) ->
      System.out.println("Hello " + message);
        
      greetService1.sayMessage("Runoob");
      greetService2.sayMessage("Google");
   }
    
   interface MathOperation {
      int operation(int a, int b);
   }
    
   interface GreetingService {
      void sayMessage(String message);
   }
    
   private int operate(int a, int b, MathOperation mathOperation){
      return mathOperation.operation(a, b);
   }
}
```

#### 使用 lambdas 来实现 Runnable 接口

```
// 1.1使用匿名内部类  
new Thread(new Runnable() {  
    @Override  
    public void run() {  
        System.out.println("Hello world !");  
    }  
}).start();  
  
// 1.2使用 lambda expression  
new Thread(() -> System.out.println("Hello world !")).start();  
  
// 2.1使用匿名内部类  
Runnable race1 = new Runnable() {  
    @Override  
    public void run() {  
        System.out.println("Hello world !");  
    }  
};  
  
// 2.2使用 lambda expression  
Runnable race2 = () -> System.out.println("Hello world !");  
   
// 直接调用 run 方法(没开新线程哦!)  
race1.run();  
race2.run();  
```

#### 对集合进行迭代

```
	List<String> languages = Arrays.asList("java","scala","python");

    languages.forEach(x -> System.out.println(x));
    languages.forEach(System.out::println);
```

#### map与reduce

```
	//map的作用是将一个对象变换为另外一个
	List<Double> cost = Arrays.asList(10.0, 20.0,30.0);
    cost.stream().map(x -> x + x*0.05).forEach(x -> System.out.println(x));
    //reduce实现的则是将所有值合并为一个
    List<Double> cost = Arrays.asList(10.0, 20.0,30.0);
    double allCost = cost.stream().map(x -> x+x*0.05).reduce((sum,x) -> sum + x).get();
    System.out.println(allCost);
```

**filter操作**

```
	List<Double> cost = Arrays.asList(10.0, 20.0,30.0,40.0);
    List<Double> filteredCost = cost.stream().filter(x -> x > 25.0).collect(Collectors.toList());
    filteredCost.forEach(x -> System.out.println(x));
```

#### 与函数式接口Predicate

使用 java.util.function.Predicate 函数式接口以及lambda表达式，可以向API方法添加逻辑，用更少的代码支持更多的动态行为。Predicate接口非常适用于做过滤。

```
    public static void filterTest(List<String> languages, Predicate<String> condition) {
        languages.stream().filter(x -> condition.test(x)).forEach(x -> System.out.println(x + " "));
    }

    public static void main(String[] args) {
        List<String> languages = Arrays.asList("Java","Python","scala","Shell","R");
        System.out.println("Language starts with J: ");
        filterTest(languages,x -> x.startsWith("J"));
        System.out.println("\nLanguage ends with a: ");
        filterTest(languages,x -> x.endsWith("a"));
        System.out.println("\nAll languages: ");
        filterTest(languages,x -> true);
        System.out.println("\nNo languages: ");
        filterTest(languages,x -> false);
        System.out.println("\nLanguage length bigger three: ");
        filterTest(languages,x -> x.length() > 4);
    }
```









