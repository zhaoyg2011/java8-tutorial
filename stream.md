#翻译出处：http://winterbe.com/posts/2014/07/31/java8-stream-tutorial-examples/
这是一个例子驱动的教程，给你一个深度对java Streams的总览.
# 流是怎样工作
# 不同种类的流
# 执行顺序
现在我们已经学写了怎样创建以及使用不同种类的流,现在我们来深入流在底层是怎样工作的.<br>
中间操作一个重要的特征是懒。让我们看一下这个没有终端操作的例子.
```java
Stream.of("d2","a2","b1","b3","c")
      .filter(s -> {
        System.out.println("filter:" + s);
        return true;
      });
```
当我们执行这个代码片段，控制台没有打印任何东西.这是因为中间操作仅仅在有终端操作时才能执行.<br>
让我们用终端操作`forEach`来扩展上面的例子.<br>
```java
Stream.of("d2","a2","b1","b3","c")
      .filter(s -> {
        System.out.println("filter:" + s);
        return true;
      })
      .forEach(s -> System.out.println(forEach:" + s));
```
执行这段代码在控制台上输出了我们想要的结果.<br>
```java
filter:  d2
forEach: d2
filter:  a2
forEach: a2
filter:  b1
forEach: b1
filter:  b3
forEach: b3
filter:  c
forEach: c
```
结果可能是很奇怪的.一个幼稚的方式是对流中的每个元素水平的一个接一个执行。但是相反每个元素垂直移动.第一个字符串`d2`传递给`filter`随后`forEach`,之后为第二字符串“a2”.<br>
这个行为能够降低每个元素真正操作的数量,正如我们下面例子演示.<br>
```java
Stream.of("d2", "a2", "b1", "b3", "c")
    .map(s -> {
        System.out.println("map: " + s);
        return s.toUpperCase();
    })
    .anyMatch(s -> {
        System.out.println("anyMatch: " + s);
        return s.startsWith("A");
    });

// map:      d2
// anyMatch: D2
// map:      a2
// anyMatch: A2
```
当谓词应用在了给定的输入元素，这个操作`anyMatch`返回了`true`。对第二个元素“A2”为真。由于流链的垂直执行，在这种情况下`map`仅仅执行了两次。所以代替对所有流元素执行,`map`将尽可能少被调用.
# 为什么顺序有重大影响
下面的例子有两个中间操作`map`和`filter`和终端操作`forEach`.我们再一次审视一下这些操作是怎样被执行的.
```java
Stream.of("d2", "a2", "b1", "b3", "c")
    .map(s -> {
        System.out.println("map: " + s);
        return s.toUpperCase();
    })
    .filter(s -> {
        System.out.println("filter: " + s);
        return s.startsWith("A");
    })
    .forEach(s -> System.out.println("forEach: " + s));

// map:     d2
// filter:  D2
// map:     a2
// filter:  A2
// forEach: A2
// map:     b1
// filter:  B1
// map:     b3
// filter:  B3
// map:     c
// filter:  C
```
正如你可能猜到的`map`和`filter`被调用了5次而`forEach`仅仅被调用了一次<br>
如果我们能够改变操作的顺序，通过移动`filter`至开始的位置,我们能够大量降低实际操作的数量.
```java
tream.of("d2", "a2", "b1", "b3", "c")
    .filter(s -> {
        System.out.println("filter: " + s);
        return s.startsWith("a");
    })
    .map(s -> {
        System.out.println("map: " + s);
        return s.toUpperCase();
    })
    .forEach(s -> System.out.println("forEach: " + s));

// filter:  d2
// filter:  a2
// map:     a2
// forEach: A2
// filter:  b1
// filter:  b3
// filter:  c
```
现在`map`仅仅被调用一次，所以对大数目的输入元素，操作管道也执行更快.记住联合复杂方法链的时机.
我们扩展一下上面的例子，增加一个操作`sorted`.
```java
Stream.of("d2", "a2", "b1", "b3", "c")
    .sorted((s1, s2) -> {
        System.out.printf("sort: %s; %s\n", s1, s2);
        return s1.compareTo(s2);
    })
    .filter(s -> {
        System.out.println("filter: " + s);
        return s.startsWith("a");
    })
    .map(s -> {
        System.out.println("map: " + s);
        return s.toUpperCase();
    })
    .forEach(s -> System.out.println("forEach: " + s));
```
排序是一个特殊的中间操作.他是所谓的有状态的操作，因为为了排序一个集合中的元素，你必须在排序中保持状态.
执行这个例子导致了下面的输出:
```
sort:    a2; d2
sort:    b1; a2
sort:    b1; d2
sort:    b1; a2
sort:    b3; b1
sort:    b3; d2
sort:    c; b3
sort:    c; d2
filter:  a2
map:     a2
forEach: A2
filter:  b1
filter:  b3
filter:  c
filter:  d2
```
首先，排序操作被执行在整个集合上.换句话说，排序被垂直操作.所以在这种情况下，`sorted`被调用8次。
马上我们可以再次靠调序操作链来优化性能.
```java
Stream.of("d2", "a2", "b1", "b3", "c")
    .filter(s -> {
        System.out.println("filter: " + s);
        return s.startsWith("a");
    })
    .sorted((s1, s2) -> {
        System.out.printf("sort: %s; %s\n", s1, s2);
        return s1.compareTo(s2);
    })
    .map(s -> {
        System.out.println("map: " + s);
        return s.toUpperCase();
    })
    .forEach(s -> System.out.println("forEach: " + s));

// filter:  d2
// filter:  a2
// filter:  b1
// filter:  b3
// filter:  c
// map:     a2
// forEach: A2
```
在这个例子中，`sorted`从来没有被调用，因为`filter`降低输入集合到一个元素.
# 重用Streams
java8 Streams 不能被重用.只要你调用终端操作，流就被关闭.
```
Stream<String> stream =
    Stream.of("d2", "a2", "b1", "b3", "c")
        .filter(s -> s.startsWith("a"));

stream.anyMatch(s -> true);    // ok
stream.noneMatch(s -> true);   // exception
```
在同一个流中调用`ongMatch`在`anyMatch`之后将导致下面异常.
```java
java.lang.IllegalStateException: stream has already been operated upon or closed
    at java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:229)
    at java.util.stream.ReferencePipeline.noneMatch(ReferencePipeline.java:459)
    at com.winterbe.java8.Streams5.test7(Streams5.java:38)
    at com.winterbe.java8.Streams5.main(Streams5.java:28)
```
为了克服这个限制，我们不得不为每个我们希望执行的终端操作创建一个新的流链。例如，我们可以用已经建立的新的中间操作创建一个流生产者来构建一个新的流。
```java
Supplier<Stream<String>> streamSupplier =
    () -> Stream.of("d2", "a2", "b1", "b3", "c")
            .filter(s -> s.startsWith("a"));

streamSupplier.get().anyMatch(s -> true);   // ok
streamSupplier.get().noneMatch(s -> true);  // ok
```
每次调用`get()`构造一个新的流。
# 高级操作