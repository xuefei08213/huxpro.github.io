---
layout:     post
title:      "泛型程序设计"
subtitle:   ""
date:       2019-07-01 12:00:00
author:     "xuefei"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 读书笔记
---

# 泛型程序设计

## 为什么会出现泛型以及使用泛型的好处

在Java中增加泛型类之前，泛型程序设计是用继承实现的。ArrayList类维护一个Object引用的数组,每次调用get方法时都返回一个Object对象，调用者根据需要进行强制转换。

```java
public class ArrayList {
    private Object[] elementData;
    private int mod = 0;

    public ArrayList() {
        elementData = new Object[8];
    }

    public Object get(int index) {
        return elementData[index];
    }

    public void add(Object o) {
        elementData[mod] = o;
        mod++;
    }
}
```

```java
public class GenericClient {

    public static void main(String[] args) {
        ArrayList arrayList = new ArrayList();
        arrayList.add("generic");
        arrayList.add(new Date());

        String generic = (String) arrayList.get(0);
        System.out.println(generic);

		  // Exception in thread "main" _java.lang.ClassCastException_: java.util.Date cannot be cast to java.lang.String
        String date = (String) arrayList.get(1);
        System.out.println(date);
    }
}
```

从上面的例子可以看出用Object会存在两个问题

+ 可以向数组中添加任何类型的对象。
+ 每次获取值时必须手动进行强制类型转换.但是如果插入一个Date类型，强转时却转换为String类型，则会出异常。



泛型提供了一个更好的解决方案：类型参数（type parameters）。例如，JDK中的ArrayList类有一个类型参数用来指示元素的类型。

```java
public class GenericArrayList<E> {

    private Object[] elementData;

    private int mod = 0;

    public GenericArrayList() {
        elementData = new Object[8];
    }

    @SuppressWarnings("unchecked")
    public E get(int index) {
        return (E) elementData[index];
    }

    public void add(E e) {
        elementData[mod] = e;
        mod++;
    }

    public static void main(String[] args) {
    
        // 指定list中只能存储String类型
        GenericArrayList<String> arrayList = new GenericArrayList<>();
        arrayList.add("a");
        arrayList.add("b");

        // 获取元素时不需要手动强制转换（编译器会自动添加强制转换指令）
        String s = arrayList.get(0);
        System.out.println(s);
    }
}
```

使用泛型的好处

+ 使代码具有更好的可读性
+ 编译器可以进行检查，避免插入错误类型的对象

## 泛型分类
### 泛型类
一个泛型类（generic class）就是具有一个活多个类型变量的类。
泛型类的写法一般是用尖括号<>将泛型变量（T）括起来，并放在类名的后面。

```java
    public class Pair<T>{}
```

泛型类可以有多个类型变量

```java
    public class Pair<T,U>{}
```

类定义中的类型变量指定方法的返回类型以及域和局部变量的类型

```java
    public class Pair<T>{
        private T first;
        public T getFirst() {
            return first;
        }
    }
```

### 泛型方法
泛型方法可以定义在普通的类中，也可以定义在泛型类中。

```java
class Arrayalg{
    public static <T> T getMiddle(T...a){
        return a[a.length / 2];
    }
}
```

当调用泛型方法时，在方法名前的尖括号中放入具体的类型：

```java
String middle = Arrayalg.<String> getMiddle("jonh","Q","public");
```

但是，在大多数情况下，方法调用中的类型参数会省略，编译器有足够的信息能推断出所调用的方法。

```java
String middle = Arrayalg.getMiddle("jonh","Q","public");
```

特殊例子

```java
double middle = Arrayalg.getMiddle(3.14,1726,0);
```

对于上面的情况，编译器会自动打包参数为1个Double类型和两个Integer类型，而后找到它们的共同超类型。事实上，找到2个这样的超类型：Number和Comparable，其本身也是一个泛型类型。**在这种情况下，可以采取的补救措施就是将所有的参数都写为double值。**

## 类型变量的限定

有时候，类或者方法需要对类型变量加以约束。典型的例子是在进行比较的时候，需要比较对象的类实现了Comparable接口才行，如果没有实现该接口，就会产生编译错误。不过，可以通过对类型变量T设定限定（bound）实现这一点：

```java
public static <T extends Comparable> T min(T[] a)...
```

类型变量的限定的记法如下：

```java
<T extends BoundingType>
```

表示T应该是绑定类型的字类型（subtype）。T和绑定类型可以是类，也可以是接口。选择extends的原因是因为更接近子类的概念。
一个类型变量活通配符可以有多个限定，例如

```java
T extends Comparable & Serializable
```

限定类型用“&”分隔，而逗号用来分隔类型变量。
跟JAVA中的继承类似，限定可以有多个接口，但最多只能有一个类。如果用一个类作为限定，它必须是限定列表中的第一个。

## 完整示例

```java
public class Pair<T> {
    private T first;
    private T second;

    public Pair() {
        this.first = null;
        this.second = null;
    }

    public Pair(T first, T second) {
        this.first = first;
        this.second = second;
    }

    public T getFirst() {
        return first;
    }

    public void setFirst(T first) {
        this.first = first;
    }

    public T getSecond() {
        return second;
    }

    public void setSecond(T second) {
        this.second = second;
    }
}
```

```java
public class PairTest {

    public static void main(String[] args) {
        String[] words = { "Mary", "had", "a", "little", "lamb" };
        Pair<String> mm = ArrayAlg.minmax(words);
        System.out.println(mm.getFirst());
        System.out.println(mm.getSecond());
        
        int[] nums = {1,2,4};
        ArrayAlg._minmax_(nums);  // 因为_int_没有实现Comparable接口，因此会编译报错
        
        Integer[] integers = {Integer.parseInt("1")};
        ArrayAlg.minmax(integers);
    }
}

class ArrayAlg {
    public static <T extends Comparable<T>> Pair<T> minmax(T[] a) {
        if (a == null || a.length == 0)
            return null;

        T min = a[0];
        T max = a[0];
        for (int i = 1; i < a.length; i++) {
            if (min.compareTo(a[i]) > 0)
                min = a[i];
            if (max.compareTo(a[i]) < 0)
                max = a[i];
        }

        return new Pair<>(min, max);
    }
}
```

## 类型擦除
在JAVA虚拟机中是不存在泛型的，所以JAVA在编译时会进行泛型擦除。
擦除类型变量，并替换为限定类型（无限定类型的变量使用Object），如果存在多个限定类型，就使用第一个限定类型进行替换。

### 泛型类的泛型擦除
通过JAVA编译之后的源码验证上面的结论

#### 没有限定参数类型

```java
public class Pair<T> {

    private T first;
    private T second;

    public Pair() {
        this.first = null;
        this.second = null;
    }

    public Pair(T first, T second) {
        this.first = first;
        this.second = second;
    }

    public T getFirst() {
        return first;
    }

    public void setFirst(T first) {
        this.first = first;
    }

    public T getSecond() {
        return second;
    }

    public void setSecond(T second) {
        this.second = second;
    }

    public static void main(String[] args) {
        Pair<String> pair = new Pair<>();
        pair.setFirst("1");
        pair.setSecond("2");

        String s = pair.getFirst();
    }

}
```

将上面的代码翻编译之后可以发现变量的类型被转换为Object

```java
Classfile /Users/xuefei/git/jpractice/jpractice-base/target/classes/org/jpractice/generic/Pair.class
  Last modified 2019-6-12; size 1606 bytes
  MD5 checksum 460690148cd7dd0d1dfc151f05f59171
  Compiled from "Pair.java"
public class org.jpractice.generic.Pair<T extends java.lang.Object> extends java.lang.Object
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #12.#48        // java/lang/Object."<init>":()V
   #2 = Fieldref           #4.#49         // org/jpractice/generic/Pair.first:Ljava/lang/Object;
   #3 = Fieldref           #4.#50         // org/jpractice/generic/Pair.second:Ljava/lang/Object;
   #4 = Class              #51            // org/jpractice/generic/Pair
   #5 = Methodref          #4.#48         // org/jpractice/generic/Pair."<init>":()V
   #6 = String             #52            // 1
   #7 = Methodref          #4.#53         // org/jpractice/generic/Pair.setFirst:(Ljava/lang/Object;)V
   #8 = String             #54            // 2
   #9 = Methodref          #4.#55         // org/jpractice/generic/Pair.setSecond:(Ljava/lang/Object;)V
  #10 = Methodref          #4.#56         // org/jpractice/generic/Pair.getFirst:()Ljava/lang/Object;
  #11 = Class              #57            // java/lang/String
  #12 = Class              #58            // java/lang/Object
  #13 = Utf8               first
  #14 = Utf8               Ljava/lang/Object;
  #15 = Utf8               Signature
  #16 = Utf8               TT;
  #17 = Utf8               second
  #18 = Utf8               <init>
  #19 = Utf8               ()V
  #20 = Utf8               Code
  #21 = Utf8               LineNumberTable
  #22 = Utf8               LocalVariableTable
  #23 = Utf8               this
  #24 = Utf8               Lorg/jpractice/generic/Pair;
  #25 = Utf8               LocalVariableTypeTable
  #26 = Utf8               Lorg/jpractice/generic/Pair<TT;>;
  #27 = Utf8               (Ljava/lang/Object;Ljava/lang/Object;)V
  #28 = Utf8               (TT;TT;)V
  #29 = Utf8               getFirst
  #30 = Utf8               ()Ljava/lang/Object;
  #31 = Utf8               ()TT;
  #32 = Utf8               setFirst
  #33 = Utf8               (Ljava/lang/Object;)V
  #34 = Utf8               (TT;)V
  #35 = Utf8               getSecond
  #36 = Utf8               setSecond
  #37 = Utf8               main
  #38 = Utf8               ([Ljava/lang/String;)V
  #39 = Utf8               args
  #40 = Utf8               [Ljava/lang/String;
  #41 = Utf8               pair
  #42 = Utf8               s
  #43 = Utf8               Ljava/lang/String;
  #44 = Utf8               Lorg/jpractice/generic/Pair<Ljava/lang/String;>;
  #45 = Utf8               <T:Ljava/lang/Object;>Ljava/lang/Object;
  #46 = Utf8               SourceFile
  #47 = Utf8               Pair.java
  #48 = NameAndType        #18:#19        // "<init>":()V
  #49 = NameAndType        #13:#14        // first:Ljava/lang/Object;
  #50 = NameAndType        #17:#14        // second:Ljava/lang/Object;
  #51 = Utf8               org/jpractice/generic/Pair
  #52 = Utf8               1
  #53 = NameAndType        #32:#33        // setFirst:(Ljava/lang/Object;)V
  #54 = Utf8               2
  #55 = NameAndType        #36:#33        // setSecond:(Ljava/lang/Object;)V
  #56 = NameAndType        #29:#30        // getFirst:()Ljava/lang/Object;
  #57 = Utf8               java/lang/String
  #58 = Utf8               java/lang/Object
{
  private T first;
    descriptor: Ljava/lang/Object;
    flags: ACC_PRIVATE
    Signature: #16                          // TT;

  private T second;
    descriptor: Ljava/lang/Object;
    flags: ACC_PRIVATE
    Signature: #16                          // TT;

  public org.jpractice.generic.Pair();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: aconst_null
         6: putfield      #2                  // Field first:Ljava/lang/Object;
         9: aload_0
        10: aconst_null
        11: putfield      #3                  // Field second:Ljava/lang/Object;
        14: return
      LineNumberTable:
        line 18: 0
        line 19: 4
        line 20: 9
        line 21: 14
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      15     0  this   Lorg/jpractice/generic/Pair;
      LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            0      15     0  this   Lorg/jpractice/generic/Pair<TT;>;

  public org.jpractice.generic.Pair(T, T);
    descriptor: (Ljava/lang/Object;Ljava/lang/Object;)V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=3
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: aload_1
         6: putfield      #2                  // Field first:Ljava/lang/Object;
         9: aload_0
        10: aload_2
        11: putfield      #3                  // Field second:Ljava/lang/Object;
        14: return
      LineNumberTable:
        line 23: 0
        line 24: 4
        line 25: 9
        line 26: 14
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      15     0  this   Lorg/jpractice/generic/Pair;
            0      15     1 first   Ljava/lang/Object;
            0      15     2 second   Ljava/lang/Object;
      LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            0      15     0  this   Lorg/jpractice/generic/Pair<TT;>;
            0      15     1 first   TT;
            0      15     2 second   TT;
    Signature: #28                          // (TT;TT;)V

  public T getFirst();
    descriptor: ()Ljava/lang/Object;
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: getfield      #2                  // Field first:Ljava/lang/Object;
         4: areturn
      LineNumberTable:
        line 29: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lorg/jpractice/generic/Pair;
      LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lorg/jpractice/generic/Pair<TT;>;
    Signature: #31                          // ()TT;

  public void setFirst(T);
    descriptor: (Ljava/lang/Object;)V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: aload_1
         2: putfield      #2                  // Field first:Ljava/lang/Object;
         5: return
      LineNumberTable:
        line 33: 0
        line 34: 5
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       6     0  this   Lorg/jpractice/generic/Pair;
            0       6     1 first   Ljava/lang/Object;
      LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            0       6     0  this   Lorg/jpractice/generic/Pair<TT;>;
            0       6     1 first   TT;
    Signature: #34                          // (TT;)V

  public T getSecond();
    descriptor: ()Ljava/lang/Object;
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: getfield      #3                  // Field second:Ljava/lang/Object;
         4: areturn
      LineNumberTable:
        line 37: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lorg/jpractice/generic/Pair;
      LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lorg/jpractice/generic/Pair<TT;>;
    Signature: #31                          // ()TT;

  public void setSecond(T);
    descriptor: (Ljava/lang/Object;)V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: aload_1
         2: putfield      #3                  // Field second:Ljava/lang/Object;
         5: return
      LineNumberTable:
        line 41: 0
        line 42: 5
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       6     0  this   Lorg/jpractice/generic/Pair;
            0       6     1 second   Ljava/lang/Object;
      LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            0       6     0  this   Lorg/jpractice/generic/Pair<TT;>;
            0       6     1 second   TT;
    Signature: #34                          // (TT;)V

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: new           #4                  // class org/jpractice/generic/Pair
         3: dup
         4: invokespecial #5                  // Method "<init>":()V
         7: astore_1
         8: aload_1
         9: ldc           #6                  // String 1
        11: invokevirtual #7                  // Method setFirst:(Ljava/lang/Object;)V
        14: aload_1
        15: ldc           #8                  // String 2
        17: invokevirtual #9                  // Method setSecond:(Ljava/lang/Object;)V
        20: aload_1
        21: invokevirtual #10                 // Method getFirst:()Ljava/lang/Object;
        24: checkcast     #11                 // class java/lang/String
        27: astore_2
        28: return
      LineNumberTable:
        line 45: 0
        line 46: 8
        line 47: 14
        line 49: 20
        line 50: 28
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      29     0  args   [Ljava/lang/String;
            8      21     1  pair   Lorg/jpractice/generic/Pair;
           28       1     2     s   Ljava/lang/String;
      LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            8      21     1  pair   Lorg/jpractice/generic/Pair<Ljava/lang/String;>;
}
Signature: #45                          // <T:Ljava/lang/Object;>Ljava/lang/Object;
SourceFile: "Pair.java"
```

#### 限定参数类型为Comparable

```java
public class PairWithCompare<T extends Comparable<T>> {

    private T first;

    private T second;

    public PairWithCompare() {
        this.first = null;
        this.second = null;
    }

    public PairWithCompare(T first, T second) {
        this.first = first;
        this.second = second;
    }

    public T getFirst() {
        return first;
    }

    public void setFirst(T first) {
        this.first = first;
    }

    public T getSecond() {
        return second;
    }

    public void setSecond(T second) {
        this.second = second;
    }

    public static void main(String[] args) {
        PairWithCompare<String> pair = new PairWithCompare<>();
        pair.setFirst("1");
        pair.setSecond("2");

        String s = pair.getFirst();
    }

}
```

将上面的代码翻编译之后可以发现变量的类型被转换为Comparable

```java
Classfile /Users/xuefei/git/jpractice/jpractice-base/target/classes/org/jpractice/generic/PairWithCompare.class
  Last modified 2019-6-13; size 1691 bytes
  MD5 checksum a3a7d5a85851f409bf04d09828231008
  Compiled from "PairWithCompare.java"
public class org.jpractice.generic.PairWithCompare<T extends java.lang.Comparable<T>> extends java.lang.Object
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #12.#48        // java/lang/Object."<init>":()V
   #2 = Fieldref           #4.#49         // org/jpractice/generic/PairWithCompare.first:Ljava/lang/Comparable;
   #3 = Fieldref           #4.#50         // org/jpractice/generic/PairWithCompare.second:Ljava/lang/Comparable;
   #4 = Class              #51            // org/jpractice/generic/PairWithCompare
   #5 = Methodref          #4.#48         // org/jpractice/generic/PairWithCompare."<init>":()V
   #6 = String             #52            // 1
   #7 = Methodref          #4.#53         // org/jpractice/generic/PairWithCompare.setFirst:(Ljava/lang/Comparable;)V
   #8 = String             #54            // 2
   #9 = Methodref          #4.#55         // org/jpractice/generic/PairWithCompare.setSecond:(Ljava/lang/Comparable;)V
  #10 = Methodref          #4.#56         // org/jpractice/generic/PairWithCompare.getFirst:()Ljava/lang/Comparable;
  #11 = Class              #57            // java/lang/String
  #12 = Class              #58            // java/lang/Object
  #13 = Utf8               first
  #14 = Utf8               Ljava/lang/Comparable;
  #15 = Utf8               Signature
  #16 = Utf8               TT;
  #17 = Utf8               second
  #18 = Utf8               <init>
  #19 = Utf8               ()V
  #20 = Utf8               Code
  #21 = Utf8               LineNumberTable
  #22 = Utf8               LocalVariableTable
  #23 = Utf8               this
  #24 = Utf8               Lorg/jpractice/generic/PairWithCompare;
  #25 = Utf8               LocalVariableTypeTable
  #26 = Utf8               Lorg/jpractice/generic/PairWithCompare<TT;>;
  #27 = Utf8               (Ljava/lang/Comparable;Ljava/lang/Comparable;)V
  #28 = Utf8               (TT;TT;)V
  #29 = Utf8               getFirst
  #30 = Utf8               ()Ljava/lang/Comparable;
  #31 = Utf8               ()TT;
  #32 = Utf8               setFirst
  #33 = Utf8               (Ljava/lang/Comparable;)V
  #34 = Utf8               (TT;)V
  #35 = Utf8               getSecond
  #36 = Utf8               setSecond
  #37 = Utf8               main
  #38 = Utf8               ([Ljava/lang/String;)V
  #39 = Utf8               args
  #40 = Utf8               [Ljava/lang/String;
  #41 = Utf8               pair
  #42 = Utf8               s
  #43 = Utf8               Ljava/lang/String;
  #44 = Utf8               Lorg/jpractice/generic/PairWithCompare<Ljava/lang/String;>;
  #45 = Utf8               <T::Ljava/lang/Comparable<TT;>;>Ljava/lang/Object;
  #46 = Utf8               SourceFile
  #47 = Utf8               PairWithCompare.java
  #48 = NameAndType        #18:#19        // "<init>":()V
  #49 = NameAndType        #13:#14        // first:Ljava/lang/Comparable;
  #50 = NameAndType        #17:#14        // second:Ljava/lang/Comparable;
  #51 = Utf8               org/jpractice/generic/PairWithCompare
  #52 = Utf8               1
  #53 = NameAndType        #32:#33        // setFirst:(Ljava/lang/Comparable;)V
  #54 = Utf8               2
  #55 = NameAndType        #36:#33        // setSecond:(Ljava/lang/Comparable;)V
  #56 = NameAndType        #29:#30        // getFirst:()Ljava/lang/Comparable;
  #57 = Utf8               java/lang/String
  #58 = Utf8               java/lang/Object
{
  private T first;
    descriptor: Ljava/lang/Comparable;
    flags: ACC_PRIVATE
    Signature: #16                          // TT;

  private T second;
    descriptor: Ljava/lang/Comparable;
    flags: ACC_PRIVATE
    Signature: #16                          // TT;

  public org.jpractice.generic.PairWithCompare();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: aconst_null
         6: putfield      #2                  // Field first:Ljava/lang/Comparable;
         9: aload_0
        10: aconst_null
        11: putfield      #3                  // Field second:Ljava/lang/Comparable;
        14: return
      LineNumberTable:
        line 18: 0
        line 19: 4
        line 20: 9
        line 21: 14
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      15     0  this   Lorg/jpractice/generic/PairWithCompare;
      LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            0      15     0  this   Lorg/jpractice/generic/PairWithCompare<TT;>;

  public org.jpractice.generic.PairWithCompare(T, T);
    descriptor: (Ljava/lang/Comparable;Ljava/lang/Comparable;)V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=3
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: aload_1
         6: putfield      #2                  // Field first:Ljava/lang/Comparable;
         9: aload_0
        10: aload_2
        11: putfield      #3                  // Field second:Ljava/lang/Comparable;
        14: return
      LineNumberTable:
        line 23: 0
        line 24: 4
        line 25: 9
        line 26: 14
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      15     0  this   Lorg/jpractice/generic/PairWithCompare;
            0      15     1 first   Ljava/lang/Comparable;
            0      15     2 second   Ljava/lang/Comparable;
      LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            0      15     0  this   Lorg/jpractice/generic/PairWithCompare<TT;>;
            0      15     1 first   TT;
            0      15     2 second   TT;
    Signature: #28                          // (TT;TT;)V

  public T getFirst();
    descriptor: ()Ljava/lang/Comparable;
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: getfield      #2                  // Field first:Ljava/lang/Comparable;
         4: areturn
      LineNumberTable:
        line 29: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lorg/jpractice/generic/PairWithCompare;
      LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lorg/jpractice/generic/PairWithCompare<TT;>;
    Signature: #31                          // ()TT;

  public void setFirst(T);
    descriptor: (Ljava/lang/Comparable;)V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: aload_1
         2: putfield      #2                  // Field first:Ljava/lang/Comparable;
         5: return
      LineNumberTable:
        line 33: 0
        line 34: 5
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       6     0  this   Lorg/jpractice/generic/PairWithCompare;
            0       6     1 first   Ljava/lang/Comparable;
      LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            0       6     0  this   Lorg/jpractice/generic/PairWithCompare<TT;>;
            0       6     1 first   TT;
    Signature: #34                          // (TT;)V

  public T getSecond();
    descriptor: ()Ljava/lang/Comparable;
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: getfield      #3                  // Field second:Ljava/lang/Comparable;
         4: areturn
      LineNumberTable:
        line 37: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lorg/jpractice/generic/PairWithCompare;
      LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lorg/jpractice/generic/PairWithCompare<TT;>;
    Signature: #31                          // ()TT;

  public void setSecond(T);
    descriptor: (Ljava/lang/Comparable;)V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: aload_1
         2: putfield      #3                  // Field second:Ljava/lang/Comparable;
         5: return
      LineNumberTable:
        line 41: 0
        line 42: 5
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       6     0  this   Lorg/jpractice/generic/PairWithCompare;
            0       6     1 second   Ljava/lang/Comparable;
      LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            0       6     0  this   Lorg/jpractice/generic/PairWithCompare<TT;>;
            0       6     1 second   TT;
    Signature: #34                          // (TT;)V

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: new           #4                  // class org/jpractice/generic/PairWithCompare
         3: dup
         4: invokespecial #5                  // Method "<init>":()V
         7: astore_1
         8: aload_1
         9: ldc           #6                  // String 1
        11: invokevirtual #7                  // Method setFirst:(Ljava/lang/Comparable;)V
        14: aload_1
        15: ldc           #8                  // String 2
        17: invokevirtual #9                  // Method setSecond:(Ljava/lang/Comparable;)V
        20: aload_1
        21: invokevirtual #10                 // Method getFirst:()Ljava/lang/Comparable;
        24: checkcast     #11                 // class java/lang/String
        27: astore_2
        28: return
      LineNumberTable:
        line 45: 0
        line 46: 8
        line 47: 14
        line 49: 20
        line 50: 28
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      29     0  args   [Ljava/lang/String;
            8      21     1  pair   Lorg/jpractice/generic/PairWithCompare;
           28       1     2     s   Ljava/lang/String;
      LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            8      21     1  pair   Lorg/jpractice/generic/PairWithCompare<Ljava/lang/String;>;
}
Signature: #45                          // <T::Ljava/lang/Comparable<TT;>;>Ljava/lang/Object;
SourceFile: "PairWithCompare.java"
```

#### 翻译泛型表达式

当程序调用泛型方法时，如果擦除返回类型，编译器会强制类型转换。
也就是说编译器会将原先的方法分为两条虚拟机指令（以上面代码中的pair.getFirst()为例子）

- 对原始方法的调用
- 将返回的Object（或第一个限定类型）转换为String

### 翻译泛型方法

类型擦除也会出现在泛型方法中
桥接方法

```java
public class DateInterval extends Pair<LocalDate> {

    @Override
    public void setSecond(LocalDate second) {
        if (second.compareTo(getFirst()) >= 0) {
            super.setSecond(second);
        }
    }

    public static void main(String[] args) {
        DateInterval dateInterval = new DateInterval();
        Pair<LocalDate> pair = dateInterval;
        pair.setSecond(LocalDate.now());
    }
}
```

```java
Classfile /Users/xuefei/git/jpractice/jpractice-base/target/classes/org/jpractice/generic/DateInterval.class
  Last modified 2019-6-14; size 1168 bytes
  MD5 checksum 491865f0f73325511d0aa89c406babae
  Compiled from "DateInterval.java"
public class org.jpractice.generic.DateInterval extends org.jpractice.generic.Pair<java.time.LocalDate>
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #11.#37        // org/jpractice/generic/Pair."<init>":()V
   #2 = Methodref          #6.#38         // org/jpractice/generic/DateInterval.getFirst:()Ljava/lang/Object;
   #3 = Class              #39            // java/time/chrono/ChronoLocalDate
   #4 = Methodref          #9.#40         // java/time/LocalDate.compareTo:(Ljava/time/chrono/ChronoLocalDate;)I
   #5 = Methodref          #11.#41        // org/jpractice/generic/Pair.setSecond:(Ljava/lang/Object;)V
   #6 = Class              #42            // org/jpractice/generic/DateInterval
   #7 = Methodref          #6.#37         // org/jpractice/generic/DateInterval."<init>":()V
   #8 = Methodref          #9.#43         // java/time/LocalDate.now:()Ljava/time/LocalDate;
   #9 = Class              #44            // java/time/LocalDate
  #10 = Methodref          #6.#45         // org/jpractice/generic/DateInterval.setSecond:(Ljava/time/LocalDate;)V
  #11 = Class              #46            // org/jpractice/generic/Pair
  #12 = Utf8               <init>
  #13 = Utf8               ()V
  #14 = Utf8               Code
  #15 = Utf8               LineNumberTable
  #16 = Utf8               LocalVariableTable
  #17 = Utf8               this
  #18 = Utf8               Lorg/jpractice/generic/DateInterval;
  #19 = Utf8               setSecond
  #20 = Utf8               (Ljava/time/LocalDate;)V
  #21 = Utf8               second
  #22 = Utf8               Ljava/time/LocalDate;
  #23 = Utf8               StackMapTable
  #24 = Utf8               main
  #25 = Utf8               ([Ljava/lang/String;)V
  #26 = Utf8               args
  #27 = Utf8               [Ljava/lang/String;
  #28 = Utf8               dateInterval
  #29 = Utf8               pair
  #30 = Utf8               Lorg/jpractice/generic/Pair;
  #31 = Utf8               LocalVariableTypeTable
  #32 = Utf8               Lorg/jpractice/generic/Pair<Ljava/time/LocalDate;>;
  #33 = Utf8               (Ljava/lang/Object;)V
  #34 = Utf8               Signature
  #35 = Utf8               SourceFile
  #36 = Utf8               DateInterval.java
  #37 = NameAndType        #12:#13        // "<init>":()V
  #38 = NameAndType        #47:#48        // getFirst:()Ljava/lang/Object;
  #39 = Utf8               java/time/chrono/ChronoLocalDate
  #40 = NameAndType        #49:#50        // compareTo:(Ljava/time/chrono/ChronoLocalDate;)I
  #41 = NameAndType        #19:#33        // setSecond:(Ljava/lang/Object;)V
  #42 = Utf8               org/jpractice/generic/DateInterval
  #43 = NameAndType        #51:#52        // now:()Ljava/time/LocalDate;
  #44 = Utf8               java/time/LocalDate
  #45 = NameAndType        #19:#20        // setSecond:(Ljava/time/LocalDate;)V
  #46 = Utf8               org/jpractice/generic/Pair
  #47 = Utf8               getFirst
  #48 = Utf8               ()Ljava/lang/Object;
  #49 = Utf8               compareTo
  #50 = Utf8               (Ljava/time/chrono/ChronoLocalDate;)I
  #51 = Utf8               now
  #52 = Utf8               ()Ljava/time/LocalDate;
{
  public org.jpractice.generic.DateInterval();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method org/jpractice/generic/Pair."<init>":()V
         4: return
      LineNumberTable:
        line 14: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lorg/jpractice/generic/DateInterval;

  public void setSecond(java.time.LocalDate);
    descriptor: (Ljava/time/LocalDate;)V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_1
         1: aload_0
         2: invokevirtual #2                  // Method getFirst:()Ljava/lang/Object;
         5: checkcast     #3                  // class java/time/chrono/ChronoLocalDate
         8: invokevirtual #4                  // Method java/time/LocalDate.compareTo:(Ljava/time/chrono/ChronoLocalDate;)I
        11: iflt          19
        14: aload_0
        15: aload_1
        16: invokespecial #5                  // Method org/jpractice/generic/Pair.setSecond:(Ljava/lang/Object;)V
        19: return
      LineNumberTable:
        line 18: 0
        line 19: 14
        line 21: 19
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      20     0  this   Lorg/jpractice/generic/DateInterval;
            0      20     1 second   Ljava/time/LocalDate;
      StackMapTable: number_of_entries = 1
        frame_type = 19 /* same */

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: new           #6                  // class org/jpractice/generic/DateInterval
         3: dup
         4: invokespecial #7                  // Method "<init>":()V
         7: astore_1
         8: aload_1
         9: astore_2
        10: aload_2
        11: invokestatic  #8                  // Method java/time/LocalDate.now:()Ljava/time/LocalDate;
        14: invokevirtual #5                  // Method org/jpractice/generic/Pair.setSecond:(Ljava/lang/Object;)V
        17: return
      LineNumberTable:
        line 24: 0
        line 25: 8
        line 26: 10
        line 27: 17
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      18     0  args   [Ljava/lang/String;
            8      10     1 dateInterval   Lorg/jpractice/generic/DateInterval;
           10       8     2  pair   Lorg/jpractice/generic/Pair;
      LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
           10       8     2  pair   Lorg/jpractice/generic/Pair<Ljava/time/LocalDate;>;

  // 桥接方法
  public void setSecond(java.lang.Object);
    descriptor: (Ljava/lang/Object;)V
    flags: ACC_PUBLIC, ACC_BRIDGE, ACC_SYNTHETIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: aload_1
         2: checkcast     #9                  // class java/time/LocalDate
         5: invokevirtual #10                 // Method setSecond:(Ljava/time/LocalDate;)V
         8: return
      LineNumberTable:
        line 14: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lorg/jpractice/generic/DateInterval;
}
Signature: #32                          // Lorg/jpractice/generic/Pair<Ljava/time/LocalDate;>;
SourceFile: "DateInterval.java"
```
总之需要记住有关Java泛型转换的事实：

- 虚拟机中没有泛型，只有普通的类和方法
- 所有的类型参数都用它们的限定类型替换
- 桥方法被合成来保持动态
- 为保持类型安全性，必要时插入强制类型转换


泛型的约束和局限性

- 不能用基本类型实例化类型参数：因为类型擦除后会含有Object类型的域，而Object不能存储double值。


- 运行时类型查询只适用于原始类型

	虚拟机中的对象总有一个特定的非泛型类型.因此，所有的查询都产生原始类型。
	
	```java
	if(a instanceof Pair<String>) //Error
	if(a instanceof Pair<T>) // Error
	Pair<String> p = (Pair<String>)a; // Warning--can only test that is a pair
	```
	试图查询一个对象是否属于某个泛型类型时，如果使用instanceof会得到编译错误，如果使用强制类型转换会得到一个警告。同样的道理，getClass方法总是返回原始类型
	
	同样的道理，**getClass方法总是返回原始类型**，例如：
	
	```java
	Pair<String> stringPair = ...
	Pair<Employee> employeePair = ...
	if(stringPair.getClass() == employeePair.getClass()) // they are equal
	```
	
- 不能创建参数化类型的数组

  不能实例化参数化类型数组，例如：
  
  ```java
  Pair<String>[] table = new Pair<String>[10]; // Error
  ```
  在**类型擦除**之后，table的类型变为Pair[],可以把它转换为Object[]:
  
  ```java
  Object[] objectArray = table;
  ```
  数组会记住它的元素类型，如果试图存储其他类型的元素，就会抛出一个ArrayStoreException
  
  ```java
  objectArray[0] = "Hello";  // Error
  ```
  
  不过对于泛型类型，擦除会使这种机制无效，以下赋值：
  
  ```java
  objectArray[0] = new Pair<Employee>(); 
  ```
  虽然能通过数组检查，不过还是会导致类型错误。出于这个原因，不允许创建参数化类型数组。
  
  **需要说明的是，只是不允许创建这些数组，但是声明类型为Pair<String>[]的变量是合法的。不过不能用new Pair<String>[10]来初始化这个变量**
  
  **可以通过声明通配类型数组，然后进行类型转换**
  
  ```java
  Pair<String>[] table = (Pair<String>[])new Pair<?>[10];
  ```
  
- Varargs警告

  在上面一条中不能创建参数化类型的数组，但是向参数个数可变的方法传递一个泛型类型却是特殊情况。
  
  ```java
  public static <T> void addAll(Collection<T> coll,T... ts){
      for(T : ts ) coll.add(t);
  }
  ```
  实际上参数ts是一个数组，包含提供的所有实参数。
  考虑下面的调用：
  
  ```java
  Collection<Pair<String>> table = ...;
  Pair<String> pair1 = ...;
  Pair<String> pair2 = ...;
  addAll(table, pair1, pair2);
  ```
  为了调用这个方法，Java虚拟机建立一个Pair<String>数组，这就违反了前面的规则。但是，对这种情况，只会得到一个警告，而不是错误。
  可以采用两种方式来抑制这个警告：一种是为调用addAll的方法加上注解@SuppressWarning("unchecked");另外一种是直接用@SafeVarargs直接标注addAll方法。
  
  ```java
  SafeVarargs
  public static <T> void addAll(Collection<T> coll,T... ts)
  ```
    
- 不能实例化类型变量

  不能使用像new T(...),new T[...],或T.class这样的表达式中的类型变量。
  
  ```java
  public Pair(){first = new T();second = new T();} // Error
  
  // 在Java8之后最好的解决办法是让调用者提供一个构造器表达式。
  
  Pair<String> p = Pair.makePair(String::new);
  
  // makePair接收一个Supplier<T>,这是一个函数式接口
  public static <T> Pair<T> makePair(Supplier<T> constr){
      return new Pair(constr.get(), constr.get());
  }
  
  // 传统的调用方式
  public static <T> Pair<T> makePair(Class<T> cl){
      try{
          return new Pair<>(cl.newInstance(),cl.newInstance());
      }catch(Exception e){
          return null;
      }
  }
  
  ```

- 不能构造泛型数组
  

- 泛型类的静态上下文中类型变量无效
  
  不能在静态域或方法中引用类型变量
  
  ```java
  public class Singleton<T>{
      private static T singleInstance; // Error
      private static getSingleInstance(){ //Error
         if(singleInstance == null){
            return singleInstance;
         }
      }
      
  }
  ```
  
  禁止使用带有类型变量的静态域和方法。

- 不能抛出或捕获泛型类的实例
  
  既不能抛出也不能捕获泛型类对象。实际上，甚至泛型类扩展Throwable都是不合法的。例如，下面的代码就不能正常编译：
  
  ```java
  public class Problem<T> extends Exception{...} // Error
  
  // catch子句中不能使用类型变量
  
  public class <T extends Throwable> void work(Class<T> t){
  
      try{
          do work
      }catch(T e){
          Logger.global.info(...);
      } // Error--can't catch type variable
  
  }
  
  ```
  
  不过，在异常规范中使用变量是允许的，以下的方法就合法：
  
  ```java
  public class <T extends Throwable> void work(Class<T> t) throw T{
  
      try{
          do work
      }catch(Throwable realCause){
          t.initCause(realCause);
          throw t;
      }
  
  }
  ```

- 可以消除对受查异常的检查

```java
public abstract class Block {

    public abstract void body() throws Exception;

    public Thread toThread() {
        return new Thread() {
            @Override
            public void run() {
                try {
                    body();
                } catch (Exception e) {
                    Block.throwAs(e);
                }
            }
        };
    }

    @SuppressWarnings("unchecked")
    public static <T extends Throwable> void throwAs(Throwable e) throws T {
        throw (T) e;
    }

}
```

- 注意擦除后的冲突
  因为存在类型擦除，会将类型变量转换为Object或第一个限定类型，所以要考虑到类型擦除之后的类型。下面是在Pair中加入equals方法，在类型擦除后会跟Object.equals(Object)冲突
  
  ```java
  
  // Name clash: The method equals(T) of type Pair<T> has the same erasure as equals(Object) of type Object but does not override it
  
  public boolean equals(T value) {
        return first.equals(value) && second.equals(value);
    }
  
  ```
  
  要想支持擦除的转换， 就需要强行限制一个类或类型变量不能同时成为两个接口类型的子类， 而这两个接口是同一接口的不同参数化。
  
  ```java
  class Employee implements Coinparab1e<Emp1oyee> { . . . } 
  class Manager extends Employee implements Comparable<Hanager>{ . . . } //Error
  ```
  
通配符类型

- 通配符的子类型限定：Pair<? extends Employee>（可以从泛型对象读取）

  不能调用setFirst方法，编译器只知道需要某个Employee的子类型，但不知道是什么类型。它拒绝传递任何特定的类型。毕竟?不能用来匹配
  使用getFirst就不存在这个问题：将getFirst的返回值赋给一个Employee的引用完全合法。
  
  ```java
  public static void printBuddies(Pair<? extends Employee> pair) {

        pair.setFirst(new Employee("")); // Error
        Employee first = pair.getFirst();
        Employee second = pair.getSecond();
        System.out.println(first.getName() + " and " + second.getName() + " is buddies");
    }
  ```

- 通配符的超类型限定：Pair<? super Manager>（可以向泛型对象写入）
  可以为方法提供参数，但不能使用返回值。
  
  ```java
  public static void minmaxBouns(Manager[] managers, Pair<? super Manager> result) {

        if (managers.length == 0)
            return;
        Manager min = managers[0];
        Manager max = managers[0];

        for (int i = 1; i < managers.length; i++) {
            if (managers[i].getBounds() < min.getBounds()) {
                min = managers[i];
            }
            if (managers[i].getBounds() > max.getBounds()) {
                max = managers[i];
            }
        }

        result.setFirst(min);
        result.setSecond(max);

        Manager manager = (Manager) result.getFirst();
        System.out.println(manager.getName());

    }
  ```
  
  ```java
  public static <T extends Comparable<? super T>> Pair<T> minmax(T[] a)
  ```
  
  ```java
   default boolean removeIf(Predicate<? super E> filter) 
   
   ArrayList<Employee> staff = ...
   Predicate<Object> oddHashCode = obj -> obj.hashCode%2  != 0;
   staff.removelf(oddHashCode);
  
  ```

- 无限定通配符

  getFirst的返回值只能赋给一个Object。setFirst方法不能被调用，甚至不能用Object调用。Pair<?>和Pair本质的不同在于：可以用任意Object对象调用原始Pair类的setObject方法。
  
  ```java
    public static boolean hasNulls(Pair<?> p) {
        return p.getFirst() == null || p.getSecond() == null;
    }
    
  ```

- 通配符捕获

  ```java
  public static void swap(Pair<?> p)
  
  public static <T> void swapHelper(Pair<T> p) {
        T t = p.getFirst();
        p.setFirst(p.getSecond());
        p.setSecond(t);
    }
  ```
