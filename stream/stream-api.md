# Java Stream API 入门篇

# Java Stream API 入门篇

**代码简洁**，函数式编程写出的代码简洁且意图明确，使用stream接口让你从此告别for循环。

**多核友好**，Java函数式编程使得编写并行程序从未如此简单，你需要的全部就是调用一下parallel()方法。

stream并不是某种数据结构，它只是数据源的一种视图。这里的数据源可以是一个数组，Java容器或I/O channel等。正因如此要得到一个stream通常不会手动创建，而是调用对应的工具方法，比如：

- 调用Collection.stream()或者Collection.parallelStream()方法；
- 调用Arrays.stream(T[] array)方法。

常见的stream接口继承关系如图：

> 为不同数据类型设置不同stream接口，可以1.提高性能，2.增加特定接口函数。

```
BaseStream

	|--	IntStream   int类型

	|--	LongStream   long类型

	|--	DoubleStream	double类型

	|--	Stream    其他类型

```

虽然大部分情况下stream是容器调用Collection.stream()方法得到的，但stream和collections有以下不同：

- **无存储**。stream不是一种数据结构，它只是某种数据源的一个视图，数据源可以是一个数组，Java容器或I/O channel等。
- **为函数式编程而生**。对stream的任何修改都不会修改背后的数据源，比如对stream执行过滤操作并不会删除被过滤的元素，而是会产生一个不包含被过滤元素的新stream。
- **惰式执行**。stream上的操作并不会立即执行，只有等到用户真正需要结果的时候才会执行。
- **可消费性**。stream只能被“消费”一次，一旦遍历过就会失效，就像容器的迭代器那样，想要再次遍历必须重新生成。

对stream的操作分为为两类，**中间操作(intermediate operations)和结束操作(terminal operations)**，二者特点是：

1. **中间操作总是会惰式执行**，调用中间操作只会生成一个标记了该操作的新stream，仅此而已。
2. **结束操作会触发实际计算**，计算发生时会把所有中间操作积攒的操作以pipeline的方式执行，这样可以减少迭代次数。计算完成之后stream就会失效。

| 操作类型 |      | 接口方法                                     |
| ---- | ---- | ---------------------------------------- |
| 中间操作 | 无状态  | concat()  filter() flatMap() map() peek()  parallel() sequentila() unordered() mapToInt() mapToLong() mapToDouble() flatMapToInt() |
|      | 有状态  | distinct() sorted() limit() skip()       |
| 结束操作 | 非短路  | collect() count() forEach() forEachOrdered() max() min() reduce() toArray() |
|      | 短路   | allMatch() anyMatch() noneMatch() findFirst() fndAny() |

> 区分中间操作和结束操作最简单的方法，就是看方法的返回值，返回值为stream的大都是中间操作，否则是结束操作。

## **stream方法使用**

stream跟函数接口关系非常紧密，没有函数接口stream就无法工作。回顾一下：函数接口是指内部只有一个抽象方法的接口。通常函数接口出现的地方都可以使用Lambda表达式，所以不必记忆函数接口的名字。

### max()--最大元素

```java
Optional<String> max = stream.max((s1,s2)->s1.length()-s2.length());
			String string = max.get();
```



### findAny()--任一一元素

```java
	Optional<String> optional = stream.filter(s->s.length()==1).findAny();
			String string = optional.get();
```



### anyMatch()--任一匹配

```java
boolean allMatch = stream.anyMatch(s->s.length()==3);
```



### noneMatch()--无一匹配

```java
stream.noneMatch(s->s.length()==3);
```



### allMatch()--全部匹配

```java
stream.allMatch(s->s.length()==1);
```



### toArray()--转成数组

```java
Object[] array = stream.toArray();
```



### forEachOrdered()--遍历

>并行流中仍然按照原顺序遍历

```java

    List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);  
    list.stream().parallel().forEach(x -> System.out.print(x));  
    list.stream().parallel().forEachOrdered(x -> System.out.print(x));  

```



### skip()--去除前n个元素

```java
Stream<String> stream = Stream.of("1", "2", "3", "4");
		stream.skip(2)//截去前两个元素，只取后面的元素
		.forEach(System.out::println);
```



### limit()--指定元素大小

```java
	Stream<String> stream = Stream.of("1", "2", "3", "4");
		stream.limit(2l)//限制两个元素，只取前两个元素
		.forEach(System.out::println);
```



### mapToInt()--元素转int

```java
Stream<String> stream = Stream.of("1", "2", "3", "4");
		stream.mapToInt(Integer::parseInt).forEach(System.out::println);
```



### parallel()--并行流

> 流可以是顺序的也可以是并行的。顺序流的操作是在单线程上执行的，而并行流的操作是在多线程上并发执行的。 

```java
stream.parallel().forEach(System.out::println);
```



### peek()--遍历

> 对stream中的每一个元素进行遍历消费，返回一个新流   Stream<T> peek(Consumer<? super T> action);

```java
	Stream<String> stream = Stream.of("I", "love", "you", "too");
		stream.peek(TestM::getStr)//中间操作
          .forEach(System.out::println);//结束操作
```



### count()--元素个数

```java
Stream<String> stream = Stream.of("I", "love", "you", "too");
		long count = stream.count();//4个元素
```



### **forEach()**--遍历

> 我们对forEach()方法并不陌生，在Collection中我们已经见过。方法签名为void forEach(Consumer<? super E> action)，作用是对容器中的每个元素执行action指定的动作，也就是对元素进行遍历。

```java
		// 使用Stream.forEach()迭代
		String[] arry = {"I", "love", "you", "too"}; 
		Stream<String> stream = Stream.of(arry);
		stream.forEach(str -> System.out.println(str));
```

### filter()--过滤

> 函数原型为Stream<T> filter(Predicate<? super T> predicate)，作用是返回一个只包含满足predicate条件元素的Stream。

```java
		String[] arry = {"I", "love", "you", "too"}; 
		Stream<String> stream = Stream.of(arry);
		stream.filter(str->str.length()==3)//中间操作，过滤元素长度等于3的元素
			  .forEach(str -> System.out.println(str));//最终操作
```

### distinct()--去重

>函数原型为Stream<T> distinct()，作用是返回一个去除重复元素之后的Stream

```java
		String[] arry = {"I", "love", "you", "too","too","you"}; 
		Stream<String> stream = Stream.of(arry);
		stream.distinct()//中间操作,去除重复元素
			  .forEach(str -> System.out.println(str));//最终操作
```

### **sorted()--排序**

> 排序函数有两个，一个是用自然顺序排序，一个是使用自定义比较器排序，函数原型分别为Stream<T>　sorted()和Stream<T>　sorted(Comparator<? super T> comparator)。

```java
		Stream<String> stream= Stream.of("I", "love", "you", "too");
		stream.sorted((str1, str2) -> str1.length()-str2.length())//中间操作，按元素长度排序
		      .forEach(str -> System.out.println(str));//最终操作，遍历打印
```

### map()--转换

> 函数原型为<R> Stream<R> map(Function<? super T,? extends R> mapper)，作用是返回一个对当前所有元素执行执行mapper之后的结果组成的Stream。直观的说，就是对每个元素按照某种操作进行转换，转换前后Stream中元素的个数不会改变，但元素的类型取决于转换之后的类型。

```java
		Stream<String> stream = Stream.of("I", "love", "you", "too");
		stream.map(str -> str.toUpperCase())//转换成大写
		      .forEach(str -> System.out.println(str));
```

### flatMap()--合并

> 函数原型为<R> Stream<R> flatMap(Function<? super T,? extends Stream<? extends R>> mapper)，作用是对每个元素执行mapper指定的操作，并用所有mapper返回的Stream中的元素组成一个新的Stream作为最终返回结果。说起来太拗口，通俗的讲flatMap()的作用就相当于把原stream中的所有元素都”摊平”之后组成的Stream，转换前后元素的个数和类型都可能会改变。

```java
		Stream<List<Integer>> stream = Stream.of(Arrays.asList(1,2), Arrays.asList(3, 4, 5));
		stream.flatMap(list -> list.stream())//转换到一个流
    		  .forEach(i -> System.out.println(i));
```

# Java Stream API 进阶篇

**规约操作**（reduction operation）又被称作折叠操作（fold），是通过某个连接动作将所有元素汇总成一个汇总结果的过程。元素求和、求最大值或最小值、求出元素总个数、将所有元素转换成一个列表或集合，都属于规约操作。Stream类库有两个通用的规约操作reduce()和collect()，也有一些为简化书写而设计的专用规约操作，比如sum()、max()、min()、count()等。

最大或最小值这类规约操作很好理解（至少方法语义上是这样），我们着重介绍reduce()和collect()

### reduce()--生成一个值

reduce操作可以实现从一组元素中生成一个值，sum()、max()、min()、count()等都是reduce操作，将他们单独设为函数只是因为常用。

reduce()的方法定义有三种重写形式：

- Optional<T> reduce(BinaryOperator<T> accumulator)
- T reduce(T identity, BinaryOperator<T> accumulator)
- <U> U reduce(U identity, BiFunction<U,? super T,U> accumulator, BinaryOperator<U> combiner)

```
Optional<T> reduce(BinaryOperator<T> accumulator)

我们先看第一个变形，其接受一个函数接口BinaryOperator<T>，而这个接口又继承于BiFunction<T, T, T>
BinaryOperator接口，可以看到reduce方法接受一个函数，这个函数有两个参数，第一个参数是上次函数执行的返回值（也称为中间结果），第二个参数是stream中的元素，这个函数把这两个值相加，得到的和会被赋值给下次执行这个函数的第一个参数。要注意的是：第一次执行的时候第一个参数的值是Stream的第一个元素，第二个参数是Stream的第二个元素
第一变形：未定义初始值，从而第一次执行的时候第一个参数的值是Stream的第一个元素，第二个参数是Stream的第二个元素

T reduce(T identity, BinaryOperator<T> accumulator)

第二个变形，与第一种变形相同的是都会接受一个BinaryOperator函数接口，不同的是其会接受一个identity参数，用来指定Stream循环的初始值。定义了初始值，从而第一次执行的时候第一个参数的值是初始值，第二个参数是Stream的第一个元素。如果Stream为空，就直接返回该值。另一方面，该方法不会返回Optional，因为该方法不会出现null。


<U> U reduce(U identity, BiFunction<U,? super T,U> accumulator, BinaryOperator<U> combiner)
Stream是支持并发操作的，为了避免竞争，对于reduce线程都会有独立的result，combiner的作用在于合并每个线程的result得到最终结果。这也说明了了第三个函数参数的数据类型必须为返回数据类型了。
```



虽然函数定义越来越长，但语义不曾改变，多的参数只是为了指明初始值（参数identity），或者是指定并行执行时多个部分结果的合并方式（参数combiner）。reduce()最常用的场景就是从一堆值中生成一个值。用这么复杂的函数去求一个最大或最小值，你是不是觉得设计者有病。其实不然，因为“大”和“小”或者“求和”有时会有不同的语义。

```java
Stream<String> stream = Stream.of("I", "love", "you", "too");
Optional<String> longest = stream.reduce((s1, s2) -> s1.length()>=s2.length() ? s1 : s2);
//Optional<String> longest = stream.max((s1, s2) -> s1.length()-s2.length());
System.out.println(longest.get());
```

上述代码会选出最长的单词love，其中Optional是（一个）值的容器，使用它可以避免null值的麻烦。当然可以使用Stream.max(Comparator<? super T> comparator)方法来达到同等效果，但reduce()自有其存在的理由。

需求：求出一组单词的长度之和。这是个“求和”操作，操作对象输入类型是String，而结果类型是Integer。

```java
// 求单词长度之和
Stream<String> stream = Stream.of("I", "love", "you", "too");
Integer lengthSum = stream.reduce(0,　// 初始值　// (1)
        (sum, str) -> sum+str.length(), // 累加器 // (2)
        (a, b) -> a+b);　// 部分和拼接器，并行执行时才会用到 // (3)
// int lengthSum = stream.mapToInt(str -> str.length()).sum();
System.out.println(lengthSum);
```

上述代码标号(2)处将i. 字符串映射成长度，ii. 并和当前累加和相加。这显然是两步操作，使用reduce()函数将这两步合二为一，更有助于提升性能。如果想要使用map()和sum()组合来达到上述目的，也是可以的。

reduce()擅长的是生成一个值，如果想要从Stream生成一个集合或者Map等复杂的对象该怎么办呢？终极武器collect()横空出世！

### collect()--生成一个集合

#### Stream -> collection

不夸张的讲，如果你发现某个功能在Stream接口中没找到，十有八九可以通过collect()方法实现。collect()是Stream接口方法中最灵活的一个，学会它才算真正入门Java函数式编程。先看几个热身的小例子：

```java
		Stream<String> stream = Stream.of("I", "love", "you", "too");
		List<String> list = stream.collect(Collectors.toList()); // stream -> list
		 Set<String> set = stream.collect(Collectors.toSet()); // Stream -> set
		 Map<String, Integer> map = stream.collect(Collectors.toMap(Function.identity(), String::length)); // Stream -> map
		ArrayList<String> arrayList = stream.collect(Collectors.toCollection(ArrayList::new));// Stream -> arrayList
		HashSet<String> hashSet = stream.collect(Collectors.toCollection(HashSet::new));// Stream -> HashSet
```



上述代码分别列举了如何将Stream转换成List、Set和Map。虽然代码语义很明确，可是我们仍然会有几个疑问：

1. Function.identity()是干什么的？
2. String::length是什么意思？
3. Collectors是个什么东西？收集器（Collector）是为Stream.collect()方法量身打造的工具接口（类）

**接口的静态方法和默认方法**

Function是一个接口，那么Function.identity()是什么意思呢？这要从两方面解释：

1. Java 8允许在接口中加入具体方法。接口中的具体方法有两种，default方法和static方法，identity()就是Function接口的一个静态方法。
2. Function.identity()返回一个输出跟输入一样的Lambda表达式对象，等价于形如t -> t形式的Lambda表达式。

**方法引用**

诸如String::length的语法形式叫做方法引用（method references），这种语法用来替代某些特定形式Lambda表达式。如果Lambda表达式的全部内容就是调用一个已有的方法，那么可以用方法引用来替代Lambda表达式。方法引用可以细分为四类：

| 方法引用的类别  | 举例             |
| -------- | -------------- |
| 引用静态方法   | Integer::sum   |
| 应用对象方法   | list::add      |
| 应用某个类的方法 | String::length |
| 应用构造方法   | HashMap::new   |

#### Stream -> map

前面已经说过Stream背后依赖于某种数据源，数据源可以是数组、容器等，但不能是Map。反过来从Stream生成Map是可以的，但我们要想清楚Map的key和value分别代表什么，根本原因是我们要想清楚要干什么。通常在三种情况下collect()的结果会是Map：

1. 使用Collectors.toMap()生成的收集器，用户需要指定如何生成Map的key和value。
2. 使用Collectors.partitioningBy()生成的收集器，对元素进行二分区操作时用到。
3. 使用Collectors.groupingBy()生成的收集器，对元素做group操作时用到。

情况1：使用toMap()生成的收集器，这种情况是最直接的，前面例子中已提到，这是和Collectors.toCollection()并列的方法。如下代码展示将学生列表转换成由<学生，GPA>组成的Map。非常直观，无需多言。

```java
// 使用toMap()统计学生GPA
Map<Student, Double> studentToGPA =students.stream().collect(
  					Collectors.toMap(
                      	Functions.identity(),// 如何生成key
                      	student -> computeGPA(student)// 如何生成value
                    )
);
```

情况2：使用partitioningBy()生成的收集器，这种情况适用于将Stream中的元素依据某个二值逻辑（满足条件，或不满足）分成互补相交的两部分，比如男女性别、成绩及格与否等。下列代码展示将学生分成成绩及格或不及格的两部分。

```java
// Partition students into passing and failing
Map<Boolean, List<Student>> passingFailing = students.stream()
         .collect(Collectors.partitioningBy(s -> s.getGrade() >= PASS_THRESHOLD));
```

情况3：使用groupingBy()生成的收集器，这是比较灵活的一种情况。跟SQL中的group by语句类似，这里的groupingBy()也是按照某个属性对数据进行分组，属性相同的元素会被对应到Map的同一个key上。下列代码展示将员工按照部门进行分组：

```java
// Group employees by department
Map<Department, List<Employee>> byDept = employees.stream()
            .collect(Collectors.groupingBy(Employee::getDepartment));
```

以上只是分组的最基本用法，有些时候仅仅分组是不够的。在SQL中使用group by是为了协助其他查询，比如1. 先将员工按照部门分组，2. 然后统计每个部门员工的人数。Java类库设计者也考虑到了这种情况，增强版的groupingBy()能够满足这种需求。增强版的groupingBy()允许我们对元素分组之后再执行某种运算，比如求和、计数、平均值、类型转换等。这种先将元素分组的收集器叫做上游收集器，之后执行其他运算的收集器叫做下游收集器(downstream Collector)。

```java
// 使用下游收集器统计每个部门的人数
Map<Department, Integer> totalByDept = employees.stream()
                    .collect(Collectors.groupingBy(Employee::getDepartment,
                                                   Collectors.counting()));// 下游收集器
```

上面代码的逻辑是不是越看越像SQL？高度非结构化。还有更狠的，下游收集器还可以包含更下游的收集器，这绝不是为了炫技而增加的把戏，而是实际场景需要。考虑将员工按照部门分组的场景，如果我们想得到每个员工的名字（字符串），而不是一个个Employee对象，可通过如下方式做到：

```java
// 按照部门对员工分布组，并只保留员工的名字
Map<Department, List<String>> byDept = employees.stream()
                .collect(Collectors.groupingBy(Employee::getDepartment,
                        Collectors.mapping(Employee::getName,// 下游收集器
                                Collectors.toList())));// 更下游的收集器
```

#### 拼接字符串

```java
// 使用Collectors.joining()拼接字符串
		 Stream<String> stream = Stream.of("I", "love", "you");
		 //String joined = stream.collect(Collectors.joining());// "Iloveyou"
		 //String joined = stream.collect(Collectors.joining(","));// "I,love,you"
		 String joined = stream.collect(Collectors.joining(",", "{", "}"));// "{I,love,you}"
```

