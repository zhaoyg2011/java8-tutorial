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

