# 接口默认方法
Java8允许我使用关键词default,对接口增加一个非抽象的方法的实现.这个特性也被称作为扩展方法.
这是第一个例子:
```java
interface Formula {
      double calculate(int a);

       default double sqrt(int a) {
        return Math.sqrt(a);
      }
 }
 ```
除去抽象方法calculate,接口Formula也定义了默认方法sqrt.具体类仅仅只需要实现抽象方法calculate.默认方法sqrt可以开箱即用.
```java
Formula formula = new Formula() {
    @Override
    public double calculate(int a) {
        return sqrt(a * 100);
    }
};

formula.calculate(100);     // 100.0
formula.sqrt(16);           // 4.0
```
Formula作为一个匿名类实现.代码是冗长的：实现这样一个简单计算sqrt(a*100)使用了6行代码.我们将会在下一节看到,在java8中，将会有更优雅的方式来实现单个方法的类。
# Lambda表达式
让我们从在先版本java怎么对一个字符串队列排序的简单的例子开始。
```java
List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");
Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return b.compareTo(a);
    }
});
```
为了对给定的队列中的元素排序，这个静态方法Collections.sort接受了一个list和比较器.通常情况下，你可以实现一个匿名的比较器，传递给排序方法. 代替整日里创匿名类的方法，java8带来了更短的语法：lambda expressions.
```java
Collections.sort(names,(String a,String b) -> {
    return b.compareTo(a);
  });
```
正如你看到的，代码更短，更容易读.但是它可以变得更短.
```java
Collections.sort(names,(String a,String b) -> b.compareTo(a));
```
对已一行代码的方法，可以省略括号{}和关键词return.但它甚至可以变得更短.
```java
Collections.sort(names, (a, b) -> b.compareTo(a));
```
java编译器可以推断出参数的类型，所以就可以省略.让我们进一步深入看看在实际中怎么使用lambda表达式
#函数式接口
Lambda表达式如何适配java的类型系统？每个Lambda表达式都适配被接口指定的类型.一个所谓的函数式接口必须包含`仅有一个抽象方法`声明.每个Lambda表达式必须符合这个抽象方法.因为默认方法不是抽象的，所以你可以随意增加至函数式接口.<br>
只要接口仅含有一个抽象方法，我们就可以把它作为Lambda表达式.为了确保你的接口符合要求，你应该增加`@FunctionalInterface`注解.只要你增加第二个抽象方法声明到接口，编译器感知到了注解就会抛出编译错误.<br>
例子：<br>
```java
@FunctionalInterface
interface Converter<F, T> {
    T convert(F from);
}
```
```java
Converter<String, Integer> converter = (from) -> Integer.valueOf(from);
Integer converted = converter.convert("123");
System.out.println(converted);    // 123
```
记住：如果`@FunctionalInterface`注解忽略，代码也是有效的<br>
#方法和构造函数的引用
利用静态方法的引用，上面的代码可以进一步简化
```java
Converter<String, Integer> converter = Integer::valueOf;
Integer converted = converter.convert("123");
System.out.println(converted);    // 123
```
java8允许你通过关键词`::`传递对方法，构造函数的引用.上面的代码演示了怎么引用静态方法.同样，我们也可以引用对象方法
```java
class Something {
    String startsWith(String s) {
        return String.valueOf(s.charAt(0));
    }
}
```
```java
Something something = new Something();
Converter<String, String> converter = something::startsWith;
String converted = converter.convert("Java");
System.out.println(converted);    // "J"
```
让我们来看看怎么通过关键词`::`应用构造函数.首先我们定义一个有不同构造函数的对象
```java
class Person {
    String firstName;
    String lastName;

    Person() {}

    Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
```
接下来我们定义一个Person的工厂函数来创建新的Person<br>
```java
interface PersonFactory<P extends Person> {
    P create(String firstName, String lastName);
}
```
代替手工实现工厂方法，我们通过引用把这一切都粘合起来.
```java
PersonFactory<Person> personFactory = Person::new;
Person person = personFactory.create("Peter", "Parker");
```
我们通过`Persion::new`创建了一个Person构造函数的引用.java编译器靠匹配`PersonFactory.create`签名来选择正确的构造函数.
# Lambda范围
从Lambda表达式访问外部范围的变量非常相似匿名类.<br>
你可以访问本地外部范围的final变量，同样实例变量和类变量.<br>
* 访问局部变量<br>
我们可以从Lambda表达式外部范围读取本地final局部变量.
```java
final int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);

stringConverter.convert(2);     // 3
```
与匿名类不同的是，`num`不需要声明为final.下面的代码也是有效的:
```java
int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);

stringConverter.convert(2);     // 3
```
但是对编译来说，`num`必须隐式的为final.下面的代码不能通过编译.<br>
```java
int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);
num = 3;
```
* 访问字段和静态变量
跟局部变量不同，我们有在Lambda表达式中对实例字段和静态变量读写权限.这个行为对匿名类是非常典型的.
```java
class Lambda4 {
    static int outerStaticNum;
    int outerNum;

    void testScopes() {
        Converter<Integer, String> stringConverter1 = (from) -> {
            outerNum = 23;
            return String.valueOf(from);
        };

        Converter<Integer, String> stringConverter2 = (from) -> {
            outerStaticNum = 72;
            return String.valueOf(from);
        };
    }
}
```
* 访问默认接口方法
还记得在第一节Formula的例子吗？接口`Formula`定义了一个默认方法`sqrt`,它能够从每个Formula实例或者匿名类被访问.这不能在Lambda表达式中工作.<br>匿名方法不能在Lambda中被访问。下面代码不能编译.
```java
Formula formula = (a) -> sqrt( a * 100);
```
# 内置函数式接口
java8中内置了很多函数式接口.他们中的一些在java老的版本中就被熟知了,比如`Comparator`和`Runnable`.这些内置的接口通过`@FunntionalInterface`注解被扩展为能够支持Lambda表达式.<br>
但是java8也充满了新的函数式接口来使生活更容易.哪些新的函数式接口中的一些在Guava库中已经被熟知.即使你对Guava库熟悉了，你也应该近距离的看一下这些接口是怎样被一些有用的方法扩展的。
* 谓词 <br>
Predicates是单参数布尔值函数.这个接口包含了许多默认方法来使谓词转变为逻辑表达式（与或者非）.
```java
Predicate<String> predicate = (s) -> s.length() > 0;

predicate.test("foo");
predicate.negate().test("foo");

Predicate<Boolean> nonNull = Objects::nonNull;
Predicate<Boolean> isNull = Objects::isNull;

Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();
```
* 函数 <br>
函数接口是接受单一参数并且产生一个结果.默认方法可以用来链接多个函数接口（compose,andThen）.
```java
Function<String,Integer> toInteger = Integer::valueOf;
Function<String,String> backToString = toInteger.andThen(String:valueOf);

backToString.apply("123");
```
* 生产者 <br>
Suppliers产生一个通用类型的结果。与Functions不同，Suppliers不接受参数。
```java
Supplier<Person> personSupplier = Person::new;
personSupplier.get();
```
* 消费者 <br>
消费者描述了对单个输入参数进行操作的行为.
```java
Consumer<Person> greeter = (p) -> System.out.println("hello," + p.firstName);
greeter.accept(new Person("zhao","yangang"));
```
* 比较器 <br>
比较器从老版本的java就熟知了.java 8 增加了一些默认方法至接口.
```java
Comparator<Person> comparator = (p1, p2) -> p1.firstName.compareTo(p2.firstName);

Person p1 = new Person("John", "Doe");
Person p2 = new Person("Alice", "Wonderland");

comparator.compare(p1, p2);             // > 0
comparator.reversed().compare(p1, p2);  // < 0
```
# 流
`java.util.Stream`描述了一个或者多个操作的流.流操作或者是中间的或者是终端的。终端操作返回某种类型的结果,中间操作返回刘本身所以你可以在一行中连接多个方法调用.流创建在一个源上，例如`java.util.Collection`像队列或者集合（map不支持的）.流操作或者被顺序执行或者并行.<br>
让我们看一下流是怎样工作的.首先我们创建一个字符串列表.<br>
```java
List<String> stringCollection = new ArrayList<>();
stringCollection.add("ddd2");
stringCollection.add("aaa2");
stringCollection.add("bbb1");
stringCollection.add("aaa1");
stringCollection.add("bbb3");
stringCollection.add("ccc");
stringCollection.add("bbb2");
stringCollection.add("ddd1");
```
java 8中的集合被扩展，所以你可以简单的创建流，依靠调用`Collection.stream()`or`Collection.parallelStream()`.下节解释大部分流操作.<br>
* 过滤器<br>
过滤器接受一个谓词来过滤流中的元素.这个操作能使我们对结果调用另一个流操作（`forEach`）.ForEach接受一个消费者来对过滤后的流中的每个元素进行操作.ForEach是一个终结操作。它返回空，所以不能调用另外流操作。
```java
stringCollection
   .stream()
   .filter((s) -> s.startsWith("a"))
   .forEach(System.out::println);
```
* 排序<br>
排序是一个中间操作，返回一个排序号的流视图.假如你不传递一个定制的`Comparator`,元素将按照自然顺序排序.
```java
stringCollection
          .stream()
          .sorted()
          .filter((s) -> s.startsWith("a"))
          .forEach(System.out::println);
```
记住`sorted`仅仅是创建了一个排序的流的视图而没有处理背后的集合。`stringCollection`是没有动的.
```java
System.out.println(stringCollection);
```
* map
map是一个通过给定操作将每个元素转变为另一个对象.下面的操作是将每个字符转化为大写.但是你也可以用`map`将每个对象转化为另一个类型.最终流的类型依赖你传递给`map`的类型。
```java
stringCollection
     .stream()
     .map(String::toUpperCase)
     .sorted((a,b) -> b.compareTo(a))
     .forEach(System.out::println)
```
* Match
各种匹配操作用来检查是否一个谓词匹配流。所有这些操作都是终端，返回一个boolean结果
```java
boolean anyStartsWithA =
    stringCollection
        .stream()
        .anyMatch((s) -> s.startsWith("a"));

System.out.println(anyStartsWithA);      // true

boolean allStartsWithA =
    stringCollection
        .stream()
        .allMatch((s) -> s.startsWith("a"));

System.out.println(allStartsWithA);      // false

boolean noneStartsWithZ =
    stringCollection
        .stream()
        .noneMatch((s) -> s.startsWith("z"));

System.out.println(noneStartsWithZ);      // true
```
* count
数量是一个终端操作，返回流中元素的数量.
```java
long startsWithB =
    stringCollection
        .stream()
        .filter((s) -> s.startsWith("b"))
        .count();

System.out.println(startsWithB);    // 3
```
* Reduce
Reduce对流中的每个元素执行reduction终端操作。结果是拥有reduced后的一个`Optional`
```java
Optional<String> reduced =
    stringCollection
        .stream()
        .sorted()
        .reduce((s1, s2) -> s1 + "#" + s2);

reduced.ifPresent(System.out::println);
// "aaa1#aaa2#bbb1#bbb2#bbb3#ccc#ddd1#ddd2"
```
# 并行流
正如上面所提到的，流可以是串行化，也可以是并行的。串行化的只在一个线程里执行而并行可以在多个线程中执行.<br>
下面的例子展示了怎么依靠并行流来提高性能.<br>
首先，我们创建一个不重复元素的大队列.<br>
```java
int max = 1000000;
List<String> values = new ArrayList<>(max);
for (int i = 0; i < max; i++) {
    UUID uuid = UUID.randomUUID();
    values.add(uuid.toString());
}
```
现在我们计算一下排序这个集合流的时间.<br>
* 串行排序
```java
long t0 = System.nanoTime();

long count = values.stream().sorted().count();
System.out.println(count);

long t1 = System.nanoTime();

long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0);
System.out.println(String.format("sequential sort took: %d ms", millis));

// sequential sort took: 899 ms
```
* 并行排序
```java
long t0 = System.nanoTime();

long count = values.parallelStream().sorted().count();
System.out.println(count);

long t1 = System.nanoTime();

long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0);
System.out.println(String.format("parallel sort took: %d ms", millis));

// parallel sort took: 472 ms
```
正如你看到的，两个代码片段几乎是一样的，但是并行快乐50%。我们所要做的就是从`stream`修改为`parallelStream`
# Map
正如我们提到的，maps不支持流。代替其的是maps现在支持很多新的方法来完成一般的任务.
```java
Map<Integer, String> map = new HashMap<>();

for (int i = 0; i < 10; i++) {
    map.putIfAbsent(i, "val" + i);
}

map.forEach((id, val) -> System.out.println(val));
```
上面的嗲吗是可以自解：`pubIfAbsent`阻止我们进行null检查附件操作;`forEach`接受一个消费者对每个值进行操作。<br>
下面的代码展示了关于map功能怎么计算de代码
```java
map.computeIfPresent(3, (num, val) -> val + num);
map.get(3);             // val33

map.computeIfPresent(9, (num, val) -> null);
map.containsKey(9);     // false

map.computeIfAbsent(23, num -> "val" + num);
map.containsKey(23);    // true

map.computeIfAbsent(3, num -> "bam");
map.get(3);             // val33
```
接下来，我们学习怎么移除一个key对应的尸体，仅仅是在对应一个值时.
```java
map.remove(3, "val3");
map.get(3);             // val33

map.remove(3, "val33");
map.get(3);             // null
```
另外一个方法是：
```java
map.getOrDefault(42, "not found");  // not found
```
合并map的实体也是非常容易的<br>
```java
map.merge(9, "val9", (value, newValue) -> value.concat(newValue));
map.get(9);             // val9

map.merge(9, "concat", (value, newValue) -> value.concat(newValue));
map.get(9);          
```
Merge或者假如key不存在时就插入key/value，如果存在调用Merge函数改变现有值.<br>
_Note：上述操作都不是同步或者原子操作._
# 日期API
java8在`java.time`包含了一堆新的日期和时间函数。新的日期API可以[Joda-Time](http://www.joda.org/joda-time/)相比,然而又不一样.下面这些例子覆盖了绝大多数重要部分.<br>
该部分介绍比较简单，先不翻译。
* 注解
先不翻译.
