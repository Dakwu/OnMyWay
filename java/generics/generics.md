# Contents

- [泛型](#泛型)
- [原始类型](#原始类型)
- [泛型方法](#泛型方法)
- [受限类型参数](#受限类型参数)
- [泛型中的继承和子类型](#泛型中的继承和子类型)
- [通配符](#通配符)
    - [上界通配符](#上界通配符)
    - [无界通配符](#无界通配符)
    - [下界通配符](#下界通配符)
    - [通配符和子类型](#通配符和子类型)
- [泛型擦除](#泛型擦除)
    - [类型擦除](#类型擦除)
    - [泛型方法擦除](#泛型方法擦除)
- [泛型擦除的影响以及桥梁方法](#泛型擦除的影响以及桥梁方法)
- [使用泛型的限制](#使用泛型的限制)
    - [不能在泛型类型中使用基础数据类型](#不能在泛型类型中使用基础数据类型)
    - [不能实例化类型参数](#不能实例化类型参数)
    - [不能使用类型参数声明静态变量](#不能使用类型参数声明静态变量)
    - [不能强制转换或使用 `instanceof`](#不能强制转换或使用-instanceof)
    - [不能创建泛型数组](#不能创建泛型数组)
    - [不能够创建, 捕获或者抛出参数化类型对象](#不能够创建-捕获或者抛出参数化类型对象)
    - [如果两个重载的方法在泛型擦除后的原始类型是一致的, 也是不允许的](#如果两个重载的方法在泛型擦除后的原始类型是一致的-也是不允许的)


# 泛型

**定义一个泛型类**

```java
public class Box<T> {
    // T 表示 Type
    private T t;

    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
```

在尖括号中定义的 `T` 称之为类型参数(或类型变量), 类型参数可以是类, 接口, 数组等任意非基本数据类型. 定义一个泛型接口也是同样的方式.

**实例化泛型类**

实例化一个泛型类与实例化一个普通类是类似的, 只需额外传递一个类型实参即可:

```java
Box<Integer> interBox = new Box<>();
```

**泛型参数命名约定**

- E - Element
- K - Key
- N - Number
- T - Type
- V - Value

# 原始类型

当实例化一个泛型类时, 如果省略了泛型信息, 实际上创建了一个原始类型对象.

```java
public class Box<T> {
    public void set(T t) { /* ... */}

    // ...
}

Box rawBox = new Box(); // Box 是 Box<T> 的原始类型
```

为了向后兼容, 将一个参数化的类型赋值给原始类型是允许的:

```java
Box<String> stringBox = new Box<>();
Box rawBox = stringBox; // OK
```

但是如果将一个原始类型赋值给参数化的类型, 编译器会给有警告:

```java
Box rawBox = new Box();
Box<Integer> intBox = rawBox;   // warning: unchecked conversion
```

同样, 如果用原始类型调用泛型方法, 也会得到一个警告:

```java
Box<String> stringBox = new Box<>();
Box rawBox = stringBox;
rawBox.set(8);  // warning: unchecked invocation to set(T)
```

这些警告提醒你原始类型绕过了泛型检查, 这会导致只有等到运行时才能捕获到不安全的代码. 因此, 应该避免使用原始类型.

# 泛型方法

声明一个泛型方法与声明泛型类是类似的, 只是泛型方法的类型参数的作用域仅限于方法内:

```java
public class Util {
    public static <K, V> boolean compare(Pair<K, V> p1, Pair<K, V> p2) {
        return p1.getKey().equals(p2.getKey()) &&
                p1.getValue().equals(p2.getValue());
    }
}

class Pair<K, V> {
    private K key;
    private V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public void setKey(K key) { this.key = key; }
    public void setValue(V value) { this.value = value; }
    public K getKey() { return key; }
    public V getValue() { return value; }
}
```

然后可以像这样调用泛型方法:

```java
Pair<Integer, String> p1 = new Pair<>(1, "apple");
Pair<Integer, String> p2 = new Pair<>(2, "pear");
boolean same = Util.compare(p1, p2);
```

# 受限类型参数

为了限定类型实参, 可以使用 `extends` 关键字

```java
public class Box<T> {
    private T t;

    public void setT(T t) { this.t = t; }

    public T getT() { return t; }

    public <U extends Number> void inspect(U u) {
        System.out.println(t.getClass().getName());
        System.out.println(u.getClass().getName());
    }

    public static void main(String[] args) {
        Box<Integer> integerBox = new Box<>();
        integerBox.setT(10);
        integerBox.inspect(10.2);
        integerBox.inspect("some text"); // error
    }
}
```

使用受限参数还可以调用其中的方法:

```java
public class NaturalNumber<T extends Integer> {
    private T t;
    public NaturalNumber(T t) { this.t = t;}

    public boolean isEven() {
        return t.intValue() % 2 == 0; // 调用 Integer 中的方法
    }
}
```

类型参数是可以有多个界限的:

```java
class A { /* ... */}
interface B { /* ... */}
interface C { /* ... */}

// A 如果是类, 必须放在最前面
class D<T extends A & B & C> { /* ... */} 

// compile-time error
class D<T extends B & A & C> { /* ... */} 
```

# 泛型中的继承和子类型

给定两个具体的类型 `A` 和 `B` (比如 `Number` 和 `Integer`), 无论 `A` 和 `B` 是否有关系, `MyClass<A>` 和 `MyClass<B>` 是没有任何关系的, 它们共同的父类只有 `Object`.

![](/assets/generics/generics-subtypeRelationship.gif)

我们可以继承或实现一个泛型类或接口来创建一个它的子类, 比如定义一个自己的 `List` 接口, `PayloadList`, 其中关联了一个可选的的泛型 `P`:

```java
interface PayloadList<E, P> extends List<E> {
    void setPayload(int index, P p);
}
```

下面这些参数化的 `PayloadList` 就是 `List<String>` 的子类:

- `PayloadList<String, Integer>`
- `PayloadList<String, String>`
- `PayloadList<String, Exception>`

![](/assets/generics/generics-payloadListHierarchy.gif)

# 通配符

## 上界通配符

定义一个上界通配符, 使用 `?` 加 `extends` 关键字:

```java
public static void process(List<? extends Foo> list) { /* ... */ }
```

一个实例:

```java
    public static double sumOfList(List<? extends Number> list) {
        double s = 0.0;
        for (Number n : list) {
            s += n.doubleValue();
        }

        return s;
    }

    public static void main(String[] args) {
        List<Integer> integerList = Arrays.asList(1, 2, 3, 4);
        List<Double> doubleList = Arrays.asList(1.3, 4.5, 55.1);

        System.out.println("sum of integerList: " + sumOfList(integerList)); // 10.0
        System.out.println("sum of doubleList: " + sumOfList(doubleList)); // 60.9
    }
```

## 无界通配符

无界通配符使用一个 `?` 来表示, 比如 `List<?>` , 无界通配符通常在以下两种情况使用:

- 定义的方法可以使用 `Object` 类中的方法实现
- 当方法的实现不依赖于类型参数时, 比如 `List` 中的 `size(), clear()` 方法等

考虑如下方法:

```java
    public static void printList(List<Object> list) {
        for (Object elem : list) {
            System.out.println(elem + " ");
        }
        System.out.println();
    }
```

我们想通过该方法打印任意的 `List`, 但实际上它只能打印 `List<Object>`, 而不能打印 `List<String>, List<Integer>` 等, 因为它们并不是 `List<Object>` 的子类, 这时应该使用 `List<?>`:

```java
    public static void printList(List<?> list) {
        for (Object elem : list) {
            System.out.println(elem + " ");
        }
        System.out.println();
    }
```

对于任意类型 `A`, `List<A>` 都是 `List<?>` 的子类型, 所以 `printList` 方法可以打印任意类型的 `List`

```java
    List<Integer>  li = Arrays.asList(1, 2, 3, 4);
    printList(Li);
```

> 使用 `List<Object>` 时, 可以向其中插入任意元素, 但使用 `List<?>` 时, 只能向其j中插入 `null`

## 下界通配符

下界通配符使用 `?` 加 `super` 关键字: `<? super A>`, 下界通配符限定了指定类型以及它的父类型.

> 上界通配符和下界通配符不能同时使用

定义一个方法, 操作一个 `Integer` 或 `Integer` 父类的 `List`:

```java
    public static void addNumbers(List<? super Integer> list) {
        for (int i = 0; i < 10; i++) {
            list.add(i);
        }
    }
```

## 通配符和子类型

使用通配符可以为泛型类之间创造关联关系:

```java
    List<? extends Integer> intList = new ArrayList<>();
    List<? extends Number> numberList = intList;    // OK, List<? extends Integer> 是 List<? extends Number> 的子类型
```

![](/assets/generics/generics-wildcardSubtyping.gif)

# 泛型擦除

## 类型擦除

在类型擦除的过程中, Java编译器会擦除掉所有的类型参数. 如果类型参数是受限的, 就会用它的第一个边界替换, 如果不是, 就会用 `Object` 替换

```java
    public class Node<T> {
        private T data;
        private Node<T> next;

        public Node(T data, Node<T> next) {
            this.data = data;
            this.next = next;
        }

        public T getData() { return data; }
    }
```

因为 `T` 是不受限的, 编译器会用 `Object` 替换它:

```java
    public class Node {
        private Object data;
        private Node next;

        public Node(Object data, Node next) {
            this.data = data;
            this.next = next;
        }

        public Object getData() { return data; }
    }
```

当类型参数受限时, 比如:

```java
    public class Node<T extends Comparable<T>> {
        private T data;
        private Node<T> next;

        public Node(T data, Node<T> next) {
            this.data = data;
            this.next = next;
        }

        public T getData() { return data; }
    }
```

此时, `T` 会被它的第一个边界类 `Object` 替换:

```java
    public class Node {
        private Comparable data;
        private Node next;

        public Node(Comparable data, Node next) {
            this.data = data;
            this.next = next;
        }

        public Comparable getData() { return data; }
    }
```

## 泛型方法擦除

泛型方法与泛型类的擦除是一样的

```java
    public static <T> int count(T[] anArray, T elem) {
        int cnt = 0;
        for (T e : anArray) {
            if (e.equals(elem)) {
                ++cnt;
            }
        }
        return cnt;
    }
```

同样, `T` 会被 `Object` 替换:

```java
    public static  int count(Object[] anArray, Object elem) {
        int cnt = 0;
        for (Object e : anArray) {
            if (e.equals(elem)) {
                ++cnt;
            }
        }
        return cnt;
    }
```

对于受限参数, 假设有如下几个类:

```java
    class Shape { /* ... */}
    class Circle extends Shape { /* ... */}
    class Rectangle extends Shape { /* ... */}
```

定义一个绘制不同形状的泛型方法:

```java
    public static <T extends Shape> void draw(T shape) { /* ... */} 
```

编译器会用 `Shape` 替换掉 `T`:

```java
    public static  void draw(Shape shape) { /* ... */} 
```

# 泛型擦除的影响以及桥梁方法

考虑如下两个类:

```java
    public class Node<T> {
        public T data;
    
        public Node(T data) { this.data = data; }
    
        public void setData(T data) {
            System.out.println("Node.setData");
            this.data = data;
        }
    }
    
    public class MyNode extends Node<Integer>{
        public MyNode(Integer data) {
            super(data);
        }
    
        @Override
        public void setData(Integer data) {
            System.out.println("MyNode.setData");
            super.setData(data);
        }
    }
```

经过泛型擦除后, 这两个类实际上会变成:

```java
    public class Node {
        public Object data;
    
        public Node(Object data) { this.data = data; }
    
        public void setData(Object data) {
            System.out.println("Node.setData");
            this.data = data;
        }
    }
    
    public class MyNode extends Node{
        public MyNode(Integer data) {
            super(data);
        }
    
        @Override
        public void setData(Integer data) {
            System.out.println("MyNode.setData");
            super.setData(data);
        }
    }
```

可以看到, `MyNode setData` 方法与 `Node setData` 方法的参数类型已经不一致了. 为了解决这个问题, Java编译器会自动生成一个桥梁方法来保证子类型的正确工作. 对于 `MyNode` 来说, 编译器会生成如下 `setData` 桥梁方法:

```java
    // Bridge method generated by the compiler
    public void setData(Object data) {
        setData((Integer) data);
    }

    public void setData(Integer data) {
        System.out.println("MyNode.setData");
        super.setData(data);
    }
```

# 使用泛型的限制

## 不能在泛型类型中使用基础数据类型

```java
    class Pair<K, V> {
        private K key;
        private V value;

        public Pair(K key, V value) {
            this.key = key;
            this.value = value;
        }

    }
    
    Pair<int, char> p = new Pair<>(8, 'a'); // 编译错误
    Pair<Integer, Character> p = new Pair<>(8, 'a');    // OK
```

## 不能实例化类型参数

```java
    public static <E> append(List<E> list) {
        E elem = new E();   // compile-time error
        list.add(elem);
    }
```

可以通过反射解决这个问题:

```java
    public static <E> void append(List<E> list, Class<E> cls) throws Exception {
        E elem = cls.newInstance(); // OK
        list.add(elem);
    }

    List<String> list = new ArrayList<>();
    append(list, String.class);
```

## 不能使用类型参数声明静态变量

```java
    public class MobileDevice<T> {
        private static T os;    // compile-time error;
    }
```

因为类的静态变量是被类的所有非静态对象共享的, 如果以上代码允许的话, 

```java
    MobileDevice<Smartphone> phone = new MobileDevice<>();
    MobileDevice<Pager> pager = new MobileDevice<>();
    MobileDevice<TabletPC> pc = new MobileDevice<>();
```

那么这个 `os` 到底是 `Smartphone`, 还是 `Pager`, 还是 `Tablet PC` 呢?

## 不能强制转换或使用 `instanceof`

```java
   public <E> void rtti(List<E> list) {
       if (list instanceof ArrayList<Integer>) {   // 泛型擦除, compile-time error
           // ...
       }
   }
```

不过可以退而求其次使用通配符来判断 `list` 是否是 `ArrayList`:

```java
    public static void rtti(List<?> list) {
        if (list instanceof ArrayList<?>) {
            // ...
        }
    }
```

在强制转换时, 除非编译器知道类型参数总是有效的, 否则也是不允许的:

```java
    List<Integer> li = new ArrayList<>();
    List<Number> ln = (List<Number>) li;    // compile-time error

    List<String> l1 = new ArrayList<>();
    ArrayList<String> l2 = (ArrayList<String>) l1;  // OK
```

## 不能创建泛型数组

```java
    List<Integer>[] arrayOfList = new ArrayList<Integer>[2];    // compile-time error
```

## 不能够创建, 捕获或者抛出参数化类型对象

泛型类不能直接或间接继承 `Throwable` 类

```java
    class MathException<T> extends Exception { /* ... */} // compile-time error

    class QueueFullException<T> extends Throwable { /* ... */}   // compile-time error
```

不能在 `catch` 块中捕获类型参数的实例

```java
    public static <T extends Exception, J> void execute(List<J> jobs) {
        try {
            for (J job : jobs)
                // ...
        } catch (T e) {   // compile-time error
            // ...
        }
    }
```

但是在 `throws` 后面可以跟类型参数

```java
    class Parser<T extends Exception> {
        public void parse(File file) throws T {     // OK
            // ...
        }
    }
```

## 如果两个重载的方法在泛型擦除后的原始类型是一致的, 也是不允许的

```java
    // compile-time error
    public void print(Set<String> stringSet) { }
    public void print(Set<Integer> integerSet) { }
```


参考: https://docs.oracle.com/javase/tutorial/java/generics/index.html
