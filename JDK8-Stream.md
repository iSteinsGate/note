# JDK8-Stream流

## 构造流

## 无限流

### generate

```java
@Test
public void whenGenerateStream_thenGetInfiniteStream() {
    Stream.generate(Math::random)
      .limit(5)
      .forEach(System.out::println);
}
```

### iterate

构造一个无限的流

```java
@Test
public void whenLimitInfiniteStream_thenGetFiniteElements() {
    Stream<Integer> infiniteStream = Stream.iterate(2, i -> i * 2);

    List<Integer> collect = infiniteStream
      .skip(3)
      .limit(5)
      .collect(Collectors.toList());
    
    assertEquals(collect, Arrays.asList(16, 32, 64, 128, 256));
}
```



## 中间操作

### skip

### limit

```java
@Test
public void whenLimitInfiniteStream_thenGetFiniteElements() {
    Stream<Integer> infiniteStream = Stream.iterate(2, i -> i * 2);

    List<Integer> collect = infiniteStream
      .skip(3)
      .limit(5)
      .collect(Collectors.toList());
    
    assertEquals(collect, Arrays.asList(16, 32, 64, 128, 256));
}
```

### distinct

```java
@Test
public void whenApplyDistinct_thenRemoveDuplicatesFromStream() {
    List<Integer> intList = Arrays.asList(2, 5, 3, 2, 4, 3);
    List<Integer> distinctIntList = intList.stream().distinct().collect(Collectors.toList());
    
    assertEquals(distinctIntList, Arrays.asList(2, 5, 3, 4));
}
```

### filter

```java
@Test
public void whenFilterEmployees_thenGetFilteredStream() {
    Integer[] empIds = { 1, 2, 3, 4 };
    List<Employee> employees = Stream.of(empIds)
      .map(employeeRepository::findById)
      .filter(e -> e != null)
      .filter(e -> e.getSalary() > 200000)
      .collect(Collectors.toList());
}
```

### map

```java
@Test
public void whenMapIdToEmployees_thenGetEmployeeStream() {
    Integer[] empIds = { 1, 2, 3 };
    
    List<Employee> employees = Stream.of(empIds)
      .map(employeeRepository::findById)
      .collect(Collectors.toList());
 
}
```



### flatMap

```java
@Test
public void whenFlatMapEmployeeNames_thenGetNameStream() {
    List<List<String>> namesNested = Arrays.asList( 
      Arrays.asList("Jeff", "Bezos"), 
      Arrays.asList("Bill", "Gates"), 
      Arrays.asList("Mark", "Zuckerberg"));

    List<String> namesFlatStream = namesNested.stream()
      .flatMap(Collection::stream)
      .collect(Collectors.toList());
}

```

### sorted

```java
@Test
public void whenSortStream_thenGetSortedStream() {
    List<Employee> employees = empList.stream()
      .sorted((e1, e2) -> e1.getName().compareTo(e2.getName()))
      .collect(Collectors.toList());

    assertEquals(employees.get(0).getName(), "Bill Gates");
    assertEquals(employees.get(1).getName(), "Jeff Bezos");
    assertEquals(employees.get(2).getName(), "Mark Zuckerberg");
}
```

### peek

```java
@Test
public void whenIncrementSalaryUsingPeek_thenApplyNewSalary() {
    Employee[] arrayOfEmps = {
        new Employee(1, "Jeff Bezos", 100000.0), 
        new Employee(2, "Bill Gates", 200000.0), 
        new Employee(3, "Mark Zuckerberg", 300000.0)
    };

    List<Employee> empList = Arrays.asList(arrayOfEmps);
    
    empList.stream()
      .peek(e -> e.salaryIncrement(10.0))
      .peek(System.out::println)
      .collect(Collectors.toList());

}
```

### reduce（归约）

T reduce(T identity, BinaryOperator<T> accumulator)

1. 累计求和

```java
public static void main(String[] args) {
        int sum = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9).reduce(0, (acc, n) -> acc + n);
        System.out.println(sum); // 45
    }
```

2. 将配置文件的每一行配置通过`map()`和`reduce()`操作聚合成一个`Map<String, String>`

```java
 // 按行读取配置文件:
        List<String> props = Lists.newArrayList("profile=native", "debug=true", "logging=warn", "interval=500");

        HashMap reduce = props.stream()
                .map(kv -> {
                    String[] split = kv.split("\\=", 2);
                    HashMap hashMap = new HashMap();
                    hashMap.put(split[0], split[1]);
                    return hashMap;
                })
                .reduce(new HashMap<String, String>(), (m, kv) -> {
                    m.putAll(kv);
                    return m;
                });
        reduce.forEach((k,v)->{
            System.out.println(k +"===" + v);
        });

运行结果：
    logging = warn
    interval = 500
    debug = true
    profile = native
```

## 终止操作

### collect（常用）

#### toCollection/toSet/toList

```java
@Test
public void whenToVectorCollection_thenGetVector() {
    Vector<String> empNames = empList.stream()
            .map(Employee::getName)
            .collect(Collectors.toCollection(Vector::new));
    
    assertEquals(empNames.size(), 3);
}
```

#### summarizingDouble/summaryStatistics

```java
@Test
public void whenApplySummarizing_thenGetBasicStats() {
    DoubleSummaryStatistics stats = empList.stream()
      .collect(Collectors.summarizingDouble(Employee::getSalary));

    assertEquals(stats.getCount(), 3);
    assertEquals(stats.getSum(), 600000.0, 0);
    assertEquals(stats.getMin(), 100000.0, 0);
    assertEquals(stats.getMax(), 300000.0, 0);
    assertEquals(stats.getAverage(), 200000.0, 0);
}

@Test
public void whenApplySummaryStatistics_thenGetBasicStats() {
    DoubleSummaryStatistics stats = empList.stream()
      .mapToDouble(Employee::getSalary)
      .summaryStatistics();

    assertEquals(stats.getCount(), 3);
    assertEquals(stats.getSum(), 600000.0, 0);
    assertEquals(stats.getMin(), 100000.0, 0);
    assertEquals(stats.getMax(), 300000.0, 0);
    assertEquals(stats.getAverage(), 200000.0, 0);
}
```

#### partitioningBy（分区成两部分）

```java
//按奇偶分组
@Test
public void whenStreamPartition_thenGetMap() {
    List<Integer> intList = Arrays.asList(2, 4, 5, 6, 8);
    Map<Boolean, List<Integer>> isEven = intList.stream().collect(
      Collectors.partitioningBy(i -> i % 2 == 0));
    
    assertEquals(isEven.get(true).size(), 4);
    assertEquals(isEven.get(false).size(), 1);
}
```

#### groupingBy（分组）

```java
//按首字母分组
@Test
public void whenStreamGroupingBy_thenGetMap() {
    Map<Character, List<Employee>> groupByAlphabet = empList.stream().collect(
      Collectors.groupingBy(e -> new Character(e.getName().charAt(0))));

    assertEquals(groupByAlphabet.get('B').get(0).getName(), "Bill Gates");
    assertEquals(groupByAlphabet.get('J').get(0).getName(), "Jeff Bezos");
    assertEquals(groupByAlphabet.get('M').get(0).getName(), "Mark Zuckerberg");
}
```

#### mapping

```java
//按照字母分组后并将员工映射成员工id返回
@Test
public void whenStreamMapping_thenGetMap() {
    Map<Character, List<Integer>> idGroupedByAlphabet = empList.stream().collect(
      Collectors.groupingBy(e -> new Character(e.getName().charAt(0)),
        Collectors.mapping(Employee::getId, Collectors.toList())));

    assertEquals(idGroupedByAlphabet.get('B').get(0), new Integer(2));
    assertEquals(idGroupedByAlphabet.get('J').get(0), new Integer(1));
    assertEquals(idGroupedByAlphabet.get('M').get(0), new Integer(3));
}
```

#### reducing

```java
//在这里reduce（）获取每个雇员的薪水增量并返回总和
@Test
public void whenStreamReducing_thenGetValue() {
    Double percentage = 10.0;
    Double salIncrOverhead = empList.stream().collect(Collectors.reducing(
        0.0, e -> e.getSalary() * percentage / 100, (s1, s2) -> s1 + s2));

    assertEquals(salIncrOverhead, 60000.0, 0);
}
//在这里，我们根据员工名字的首字母对员工进行分组。在每个组中，我们找到姓名最长的员工
@Test
public void whenStreamGroupingAndReducing_thenGetMap() {
    Comparator<Employee> byNameLength = Comparator.comparing(Employee::getName);
    
    Map<Character, Optional<Employee>> longestNameByAlphabet = empList.stream().collect(
      Collectors.groupingBy(e -> new Character(e.getName().charAt(0)),
        Collectors.reducing(BinaryOperator.maxBy(byNameLength))));

    assertEquals(longestNameByAlphabet.get('B').get().getName(), "Bill Gates");
    assertEquals(longestNameByAlphabet.get('J').get().getName(), "Jeff Bezos");
    assertEquals(longestNameByAlphabet.get('M').get().getName(), "Mark Zuckerberg");
}
```



#### joining

```
@Test
public void whenCollectByJoining_thenGetJoinedString() {
    String empNames = empList.stream()
      .map(Employee::getName)
      .collect(Collectors.joining(", "))
      .toString();
    
    assertEquals(empNames, "Jeff Bezos, Bill Gates, Mark Zuckerberg");
}
```

### max/min

```java
@Test
public void whenFindMin_thenGetMinElementFromStream() {
    Employee firstEmp = empList.stream()
      .min((e1, e2) -> e1.getId() - e2.getId())
      .orElseThrow(NoSuchElementException::new);

    assertEquals(firstEmp.getId(), new Integer(1));
}

@Test
public void whenFindMax_thenGetMaxElementFromStream() {
    Employee maxSalEmp = empList.stream()
      .max(Comparator.comparing(Employee::getSalary))
      .orElseThrow(NoSuchElementException::new);

    assertEquals(maxSalEmp.getSalary(), new Double(300000.0));
}
```



### allMatch/anyMatch/noneMatch

```
@Test
public void whenApplyMatch_thenReturnBoolean() {
    List<Integer> intList = Arrays.asList(2, 4, 5, 6, 8);
    
    boolean allEven = intList.stream().allMatch(i -> i % 2 == 0);
    boolean oneEven = intList.stream().anyMatch(i -> i % 2 == 0);
    boolean noneMultipleOfThree = intList.stream().noneMatch(i -> i % 3 == 0);
    
    assertEquals(allEven, false);
    assertEquals(oneEven, true);
    assertEquals(noneMultipleOfThree, false);
}
```

### findFirst

### findAny

## 特殊流

### IntStream/DoubleStream

max/average/range

```java
@Test
public void whenFindMaxOnIntStream_thenGetMaxInteger() {
    Integer latestEmpId = empList.stream()
      .mapToInt(Employee::getId)
      .max()
      .orElseThrow(NoSuchElementException::new);
    
    assertEquals(latestEmpId, new Integer(3));
}

@Test
public void whenApplySumOnIntStream_thenGetSum() {
    Double avgSal = empList.stream()
      .mapToDouble(Employee::getSalary)
      .average()
      .orElseThrow(NoSuchElementException::new);
    
    assertEquals(avgSal, new Double(200000));
}
```



#### 

## 参考

[jdk8-stream流教程](https://stackify.com/streams-guide-java-8/)