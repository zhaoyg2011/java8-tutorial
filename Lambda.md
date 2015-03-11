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
为了对给定的队列中的元素排序，这个静态方法`Collections.sort`接受了一个list和比较器.通常情况下，你可以实现一个匿名的比较器，传递给排序方法.
代替整日里创匿名类的方法，java8带来了更短的语法：`lambda expressions`.
```java
Collections.sort(names,(String a,String b) -> {
    return b.compareTo(a);
  });
```
正如你看到的，代码更短，更容易读.但是它可以变得更短.
```java
Collections.sort(names,(String a,String b) -> b.compareTo(a));
```
对已一行代码的方法，可以省略括号`{}`和关键词`return`.但它甚至可以变得更短.
```java
Collections.sort(names, (a, b) -> b.compareTo(a));
```
java编译器可以推断出参数的类型，所以就可以省略.让我们进一步深入看看在实际中怎么使用lambda表达式
