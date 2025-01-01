# 一、java重点

## 1. Equals 和 == 的区别？

​	重写equals之前，equals效果等于==，即比较基本数据类型比的是值，比较引用数据类型比的是地址。重写equals后，比较引用数据类型比的是内容是否一致。

## 2. 为什么重写equals一定要重写hashCode？

​	hashcode方法默认：两个地址相等的对象返回相等的hashcode值，地址不等的对象不一定返回相等的hashcode。这会导致如果我们new两个属性值相等的对象，返回的hashcode值很可能不一样（因为地址不等）。并且往集合像hashset中插入元素时，会导致：逻辑上相等的两个对象（属性值相等），但由于hashcode值不一样，则会插入到两个位置，这是我们需要避免的问题。因此重写hashcode后，只要两个对象逻辑上相等（属性值一致），就能产生相等的hashcode值。

![image-20221208115629863](C:\Users\pc\AppData\Roaming\Typora\typora-user-images\image-20221208115629863.png)

**如果两个对象相等 (`A.equals(B) == true`)，那么它们的 `hashCode()` 必须相等**。

**如果两个对象的 `hashCode()` 相等，并不意味着它们相等 (`A.equals(B)` 不一定为 `true`)**，这种情况叫做哈希冲突（hash collision），即不同的对象可能有相同的哈希值。

## 3. 那重写hashCode后还需重写equals吗？

​	hashcode相等，两个对象不一定相等，所以判断发现两个对象的hashcode相等后还需判断是否每个属性值都相等。

## 4. java中是值传递还是引用传递？

值传递：函数调用时，将实参的拷贝传递给形参，形参的变化不影响实参。

引用传递：函数调用时，将实参的地址传递给形参，形参的变化会直接影响实参。

​	java中只有值传递，如果实参是基本数据类型，那么我们对形参的变化是不影响实参的。如果是引用数据类型，由于栈中实参和拷贝(形参)指向堆中同一个对象，所以对形参所指向对象的改变也会影响实参所指向的对象。但对形参的指向的改变不会影响形参，就是说形参无法改变实参的指向。

## 5. 多态的好处？

​	首先多态有三个特点，子类继承父类，重写父类的方法，最后最重要的是父类引用指向子类对象。这样我们不需要在编译期就确定调用哪一个实例对象的方法，而是在运行期根据传入的实例对象来调用对应的方法。好处：当我们要调用多个对象的方法时，如果他们有共性的话，我们可以给他设计一个基类，子类重写基类的方法；然后在一个统一的方法里，形参用父类引用，通过传入不同的子类实例来调用对应的方法。更加灵活；简化代码。

![image-20221208161434546](C:\Users\pc\AppData\Roaming\Typora\typora-user-images\image-20221208161434546.png)

![image-20221208161445823](C:\Users\pc\AppData\Roaming\Typora\typora-user-images\image-20221208161445823.png)

## 6. public protected private？

private: 在同一类内可见；不能修饰类

default:在同一包内可见

protected:对同一包内的类、不同包的子类可见；不能修饰类

public:对所有类可见

![img](https://img-blog.csdnimg.cn/20190225130919201.jpg)

## 7. String 为什么不可变？

String类由final修饰，不可以被继承

String底层是private final char[] value

* 该数组是final修饰的，所以不能修改value的指向（不能改变引用），即不能指向新数组
* 该数组又是private修饰，没有对外暴露公共的修改数组值的方法，所以一旦初始化后外界无法修改值

所以我们平时改变string，用replace等方法，都只是产生了新的string对象而已

但实际上我们可以通过反射的方式获取string的私有属性value[]，然后修改值，只是不推荐这么做。

java 9之后底层是 private final byte[] value

* 一个字符使用一个字节，减少内存占用
* 计算机处理字节数组的效率更高，提高字符串处理的性能



## 8. HashMap的实现原理？

​	Hashmap有着1.7，1.8两个版本，1.8中Hashmap数据结构是数组加链表加红黑树的形式。

如何put元素时（采用尾插法，避免环形链表）：

* 首先计算key的hash值，计算方式为调用hashcode方法再进行扰动后

  （h = key.hashCode()) ^ (h >>> 16），>>>16是为了获取低位信息，与高位的hash^是为了让高位也参与运算，获取更加散列的值，减少哈希碰撞）得到hash值

* 再用hash&length-1得到最终的位置，效果等同hash%length，但能增加运算速度

得到位置之后，要判断该位置上是否有元素。

* 若无元素则直接添加
* 若有元素,则先比较hash值
  * 若hash不同,则直接添加key-value对
  * 若hash相同,则调用equals方法
    * 若返回false,则直接添加
    * 若返回true,则用新value替换旧value

在不断的添加过程中,会设计到扩容问题。默认扩容为0.75,而数组的默认长度为16，那么当添加元素个数达到12时(阈值)，触发扩容，数组长度扩大为原来的两倍，新位置为原来的位置或者原位置+旧数组长度

当链表长度大于8且数组长度大于64时，链表会变为红黑树结构，node变为treenode并且在下一次resize时，若链表长度少于6时，再转变为链表。

## 9. 为什么hashmap数组长度为2的n次方？

如果length=2^n，则length-1用二进制表示则低位都是1，与hash值取与后能更好的保留hash值低位的信息；反之，二进制上出现0，任何数和0取与都是0，这样不能充分利用数组能保存的信息（即一个bit上只能存0，存的信息变少），会增加哈希碰撞的可能性。

## 10. HashMap 为什么选用红黑树这种数据结构优化链表？

## 11. 为什么头插法会导致循环链表？

hashmap在扩容时使用头插法可能出现循环链表, 后果就是调用get()方法时可能陷入死循环. 为什么会出现循环链表呢?

[链接](https://blog.csdn.net/littlehaes/article/details/105241194?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522167860630316800182119767%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=167860630316800182119767&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-1-105241194-null-null.142^v73^pc_search_v2,201^v4^add_ask,239^v2^insert_chatgpt&utm_term=%E4%B8%BA%E4%BB%80%E4%B9%88%E5%A4%B4%E6%8F%92%E6%B3%95%E4%BC%9A%E5%AF%BC%E8%87%B4%E5%BE%AA%E7%8E%AF%E9%93%BE%E8%A1%A8%EF%BC%9F&spm=1018.2226.3001.4187)

## 12. HashMap和HashTable的区别？

相同点：都实现map,cloneable(可克隆）,serializable（可序列化）三个接口

不同点：

* HashTable内部方法用syncronized修饰，是线程同步的，也就是线程安全的，HashMap线程不安全，但性能高于HashTable。
* HashTable不允许键or值为null，而hashmap允许一个键为null，允许多个值为null
* 计算hash值方法不同，hashtable直接用hashcode方法，而hashmap用的是自定义的哈希算法（加了两次扰动，增加散列性）
* 初始化容量不同，hashtable为11，hashmap为16
* 扩容机制不同，hashtable为当前容量翻倍+1(保证为奇数），hashmap只是翻倍

## 13. 实现一个保证迭代顺序的HashMap？

使用LinkedHashMap,他继承于HashMap,并维护了一个双向链表，用头节点记录插入的第一个元素，然后每个元素都会用before和after两个指针指向前后元素，以此保证迭代顺序。



## 14. String、StringBuffer和StringBuilder的异同？

相同点：底层都是通过char数组实现的
不同点：

* String对象一旦创建，其值是不能修改的，如果要修改，会重新开辟内存空间来存储修改之后的对象；而StringBuffer和StringBuilder对象的值是可以被修改的；

* StringBuffer几乎所有的方法都使用synchronized实现了同步，线程比较安全，在多线程系统中可以保证数据同步，但是效率比较低；而StringBuilder 没有实现同步，线程不安全，在多线程系统中不能使用 StringBuilder，但是效率比较高。

* 如果我们在实际开发过程中需要对字符串进行频繁的修改，不要使用String，否则会频繁创建新对象，造成内存空间的浪费；当需要考虑线程安全的场景下使用 StringBuffer，如果不需要考虑线程安全，追求效率的场景下可以使用 StringBuilder。

StringBuilder和StringBuilder的默认长度为16，在添加元素时如果大于16，则将数组长度扩容为原来的2n+2倍(由于创建对象时可以传入参数，如传入0，这样的话+2就避免0左移1位还是0这样的问题)

## 15. New String（）会创建几个对象？

1. String str = new String("ab")

* new 在堆中创建的对象
* 字符串常量池中的“ab"

字符串常量池中可能已有ab，所以最多2个

2. String str = "a"+"b"

编译器会将其优化为String str = "ab"，所以最多1个

3. ```java
   String s1 = "a";
   String s2 = s1+"b" //new StringBuilder().append(a).append(b).toString()
   ```

   第二行代码创建了几个对象？

   * new StringBuilder() //+号两边有变量，则会创建StringBuilder对象（jdk1.8）
   * 字符串常量池中的“b"
   * new String("ab")（调用了StringBuilder的toString方法，toString内部就是：new String(value, 0, count)，value就是append（a),append(b)的结果"ab",可通过字节码看出）

   这里要注意的是：并没有在字符串常量池中创建“ab"，所以会创建最多三个对象

   ![image-20221212130340797](C:\Users\pc\AppData\Roaming\Typora\typora-user-images\image-20221212130340797.png)

4. String str = new String("a") + new String("b")

* new StringBuilder()
* new String("a")
* 字符串常量池中的"a"
* new String("b")
* 字符串常量池中的"b"
* new String("ab") //调用了StringBuilder的toString()方法

因此最多产生6个对象

![image-20221212131443987](C:\Users\pc\AppData\Roaming\Typora\typora-user-images\image-20221212131443987.png)

5. intern方法相关（jdk1.8)

```java
String s1 = new String("a")+new String("b");
String s2 = s1.intern();将这个字符串对象尝试放入串池，有就不放，没有就放入，并把传池中的对象返回
System.out.println(s1=="ab");true
System.out.println(s2=="ab");true

```

```java
String x = "ab"
String s1 = new String("a")+new String("b");
String s2 = s1.intern();将这个字符串对象尝试放入串池，有就不放，没有就放入，并把传池中的对象返回
System.out.println(s1==x);false（因为串池中已经有ab了，s1就保持在堆中，不放入串池）
System.out.println(s2==x);true
```

## 16. 了解哪些设计模式

设计模式分类：

* 创建型模式：用于创建对象的设计模式。可以简化用户创建对象的过程，降低耦合度，使用户不用关心对象具体的创建过程。
  * 单例模式
  * [工厂模式](https://blog.csdn.net/a745233700/article/details/120253639)
  * 建造者模式

* 结构型模式：组织对象之间的结构，使其易于扩展
  * 代理模式
  * 装饰者模式
  * 享元模式：共享相同内容，实现对象的复用，如字符串常量池。
* 行为型模式：主要用于决定对象做出何种行为
  * 观察者模式：又被称为发布-订阅模式，它定义了一种一对多的依赖关系，让多个观察者对象监听某一个主题对象。当这个主题对象状态变化时，会通知所有的观察者对象及时更新自己。zookeeper就是实现了观察者设计模式的分布式服务管理框架。

## 17. 单例模式详解？

1. 定义

​	单例模式：在内存中只会创建一个对象的设计模式，所有需要用到的地方都共享这一个对象实例

2. 类型

* 饿汉式：在类加载时已经创建好该单例对象，等待被程序使用
* 懒汉式：在真正需要使用对象时才去创建该单例对象

使用情形：

如果对内存要求不高，建议使用饿汉式，因为代码简单还没有线程安全问题

如果在开发中对内存要求非常高，内存有限，建议使用懒汉式，在用的时候再创建

3. 基本实现

饿汉式：

```java
public class Singleton(){
    private Singleton(){}
    private static final Singleton singleton = new Singleton();//final:防止子类重写，破坏单例模式，即避免意外的修改
    public static Singleton getInstance(){
        return singleton;
    }
}
```

懒汉式：

```java
public class Singleton(){
    private Singleton(){}
    private static Singleton singleton;
    public static Sington getInstance(){
        if(sinleton == null){
            singleton = new Singleton();
        }
        return singleton;
    }
}
```

4. 优化

上述懒汉式的实现方式只适用于单线程，多线程环境下可能导致创建多个对象。

因此采用双重检查+锁的方式实现多线程安全（直接只用锁会导致性能低下）

```java
public class Singleton(){
    private Singleton(){}
    private static Singleton singleton;
    public static Sington getInstance(){
        if(singleton == null){
            syncronized(Singleton.class){
                if(singleton == null){//防止别的线程已经创建好对象
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

上述写法还会导致多线程下指令重排的问题：在不破坏数据依赖性的前提下，代码执行顺序可以不按照代码编写顺序来，以提高单线程下cpu性能。

创建一个对象，在jvm中分为三个步骤：

* 为该对象分配内存空间
* 初始化
* 将对象引用指向分配好的内存空间

其中第二步和第三步可以颠倒顺序，也就是说线程1创建对象的时，还未初始化就将对象引用指向分配好的空间的话，线程2判断对象就不为空了，就会直接返回这个对象，由于该对象还未初始化，所以会报错

解决方案：用volatile修饰变量，所以完整版本：

```java
public class Singleton(){
    private Singleton(){}
    private static volatile Singleton singleton;
    public static Sington getInstance(){
        if(singleton == null){
            syncronized(Singleton.class){
                if(singleton == null){//防止别的线程已经创建好对象
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

5. 单例模式的破坏

* 反射：强制访问类的私有构造器，创建另一个对象
* 序列化与反序列化

可以用枚举类避免上述问题



## 代理模式详解

https://blog.csdn.net/qq_45034708/article/details/115030032

代理模式的主要作用是**控制对目标对象的访问**，并在不修改目标对象的前提下，扩展其功能。常见的使用场景包括：

- **延迟加载**：在目标对象实际需要使用时才创建或加载它（如虚拟代理）。
- **控制访问权限**：可以通过代理限制某些操作，控制对目标对象的访问。
- **记录日志或监控**：通过代理可以记录对目标对象的所有访问和操作，方便日志记录或监控。
- **性能优化**：代理可以在调用目标对象前后加入缓存或优化逻辑，提升性能。
- **远程代理**：允许在不同地址空间中访问目标对象（如远程方法调用，RMI）。

## 工厂模式

**工厂模式**是一种用于创建对象的设计模式，其目的是将对象的创建过程与使用过程分离，以提高代码的灵活性、可复用性和可维护性

1. 简单工厂模式（Simple Factory）

**简单工厂模式**是最基础的一种工厂模式，通过一个工厂类根据输入的条件来决定创建哪一个具体的产品对象。简单工厂模式通常用一个静态方法来创建不同类型的实例。

**优点**：

- 通过使用工厂类来创建对象，客户端无需关心对象的具体实现过程。
- 更容易添加新产品。

**缺点**：

- 违反了**开闭原则**，每次新增产品都需要修改工厂类。
- 工厂类容易变得复杂，难以维护。

```java
// 产品接口
public interface Car {
    void drive();
}

// 具体产品
public class SedanCar implements Car {
    @Override
    public void drive() {
        System.out.println("Driving a sedan car");
    }
}

public class SUVCar implements Car {
    @Override
    public void drive() {
        System.out.println("Driving an SUV car");
    }
}

// 简单工厂类
public class CarFactory {
    public static Car createCar(String type) {
        if ("sedan".equalsIgnoreCase(type)) {
            return new SedanCar();
        } else if ("suv".equalsIgnoreCase(type)) {
            return new SUVCar();
        } else {
            throw new IllegalArgumentException("Unknown car type");
        }
    }
}

// 客户端代码
public class Main {
    public static void main(String[] args) {
        Car sedan = CarFactory.createCar("sedan");
        sedan.drive();
        
        Car suv = CarFactory.createCar("suv");
        suv.drive();
    }
}

```



2. 工厂方法模式（Factory Method）

**工厂方法模式**通过**定义一个抽象的工厂接口**，并由实现类来决定具体创建哪个产品。这种方式符合**开闭原则**，更有利于扩展。

**优点**：

- 遵循**开闭原则**，通过增加新的工厂类来创建新产品，而不需要修改现有代码。
- 易于扩展，新增产品类型时，只需要创建一个新的工厂类。

**缺点**：

- 增加了系统的复杂性，因为每个产品都需要对应一个工厂类

```java
// 产品接口
public interface Car {
    void drive();
}

// 具体产品
public class SedanCar implements Car {
    @Override
    public void drive() {
        System.out.println("Driving a sedan car");
    }
}

public class SUVCar implements Car {
    @Override
    public void drive() {
        System.out.println("Driving an SUV car");
    }
}

// 抽象工厂
public interface CarFactory {
    Car createCar();
}

// 具体工厂
public class SedanCarFactory implements CarFactory {
    @Override
    public Car createCar() {
        return new SedanCar();
    }
}

public class SUVCarFactory implements CarFactory {
    @Override
    public Car createCar() {
        return new SUVCar();
    }
}

// 客户端代码
public class Main {
    public static void main(String[] args) {
        CarFactory sedanFactory = new SedanCarFactory();
        Car sedan = sedanFactory.createCar();
        sedan.drive();
        
        CarFactory suvFactory = new SUVCarFactory();
        Car suv = suvFactory.createCar();
        suv.drive();
    }
}

```



3. 抽象工厂模式

**抽象工厂模式**用于创建一系列相关的或互相依赖的对象，而无需指定具体的类。这种模式提供了一个接口来创建多个产品对象。

**优点**：

- 提供了一种**创建一族相关产品**的方式，避免了客户代码与具体类之间的紧耦合。
- **符合开闭原则**，便于扩展。

**缺点**：

- 难以支持新的产品类型，新增产品系列需要修改所有的工厂接口和工厂实现类。

```java
// 产品族接口
public interface Engine {
    void start();
}

public interface Tire {
    void produce();
}

// 具体产品
public class SedanEngine implements Engine {
    @Override
    public void start() {
        System.out.println("Starting sedan engine");
    }
}

public class SedanTire implements Tire {
    @Override
    public void produce() {
        System.out.println("Producing sedan tire");
    }
}

public class SUVEngine implements Engine {
    @Override
    public void start() {
        System.out.println("Starting SUV engine");
    }
}

public class SUVTire implements Tire {
    @Override
    public void produce() {
        System.out.println("Producing SUV tire");
    }
}

// 抽象工厂
public interface CarPartsFactory {
    Engine createEngine();
    Tire createTire();
}

// 具体工厂
public class SedanPartsFactory implements CarPartsFactory {
    @Override
    public Engine createEngine() {
        return new SedanEngine();
    }

    @Override
    public Tire createTire() {
        return new SedanTire();
    }
}

public class SUVPartsFactory implements CarPartsFactory {
    @Override
    public Engine createEngine() {
        return new SUVEngine();
    }

    @Override
    public Tire createTire() {
        return new SUVTire();
    }
}

// 客户端代码
public class Main {
    public static void main(String[] args) {
        CarPartsFactory sedanFactory = new SedanPartsFactory();
        Engine sedanEngine = sedanFactory.createEngine();
        Tire sedanTire = sedanFactory.createTire();
        sedanEngine.start();
        sedanTire.produce();

        CarPartsFactory suvFactory = new SUVPartsFactory();
        Engine suvEngine = suvFactory.createEngine();
        Tire suvTire = suvFactory.createTire();
        suvEngine.start();
        suvTire.produce();
    }
}

```

##  抽象类和接口的区别

* 抽象类
  * 单继承
  * 构造方法：有构造方法，用于子类实例化使用。
  * 成员变量：可以是变量，也可以是常量。
  * 成员方法：可以是抽象的，也可以是非抽象的。
* 接口
  * 多实现
  * 构造方法：没有构造方法
  * 成员变量：只能是常量。默认修饰符：public static final
  * 成员方法：jdk1.7只能是抽象的。jdk1.8可以写以default和static开头的具体方法

> 抽象类是对本质属性的抽象，接口是对动作的抽象，
>
> 抽象类表示的是，这个对象是什么。接口表示的是，这个对象能做什么。比如，男人，女人，这两个类（如果是类的话……），他们的抽象类是人。说明，他们都是人。
>
> 人可以吃东西，狗也可以吃东西，你可以把“吃东西”定义成一个接口，然后让这些类去实现它.
>
> 所以，在高级语言上，一个类只能继承一个类（抽象类）(正如人不可能同时是生物和非生物)，但是可以实现多个接口(吃饭接口、走路接口)。
>
> 当你关注一个事物的本质的时候，用抽象类；当你关注一个操作的时候，用接口。
> 

## HashMap什么时候会线程不安全？

​	如果线程A和线程B同时进行put操作，刚好这两条不同的数据hash值一样，并且该位置数据为null，所以这线程A、B都会进入第6行代码中。假设一种情况，线程A进入后还未进行数据插入时挂起，而线程B正常执行，从而正常插入数据，然后线程A获取CPU时间片，此时线程A不用再进行hash判断了，问题出现：线程A会把线程B插入的数据给**覆盖**，发生线程不安全。

```java
 final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
         Node<K,V>[] tab; Node<K,V> p; int n, i;
         if ((tab = table) == null || (n = tab.length) == 0)
             n = (tab = resize()).length;
         if ((p = tab[i = (n - 1) & hash]) == null) //如果没有hash碰撞则直接插入元素
             tab[i] = newNode(hash, key, value, null);
         else {
```



## Java对象的生命周期

1. 创建阶段：为对象分配内存空间，调用父类和子类的构造函数等等
2. 应用阶段：对象至少被一个强引用持有，除非显示的使用软引用、弱引用、虚引用
3. 不可见阶段：程序的运行已经超出了该对象的作用域了
4. 不可达阶段：对象不再被不论什么强引用所持有
5. 收集阶段：GC发现对象处于不可达阶段并且GC已经对该对象的内存空间重新分配做好准备，对象进程收集阶段。如果，该对象的finalize()函数被重写，则执行该函数。
6. 终结阶段：对象的finalize()函数执行完成后，对象仍处于不可达状态，该对象进程终结阶段
7. 对象内存空间重新分配阶段：GC对该对象占用的内存空间进行回收或者再分配，该对象彻底消失

## Java类的生命周期

1. 加载：找到需要加载的类并把类的信息加载到jvm的方法区中，然后在堆区中实例化一个java.lang.Class对象，作为方法区中这个类的信息的入口。 类的加载方式比较灵活，我们最常用的加载方式有两种，一种是根据类的全路径名找到相应的class文件，然后从class文件中读取文件内容；另一种是从jar文件中读取。（类的加载和连接可能会交替进行，但加载一定在连接之前开始，连接一定在加载之后结束）

2. 连接：做一些加载后的验证工作以及一些初始化前的准备工作（与加载和初始化阶段交替进行）

   * 验证：当一个类被加载之后，必须要验证一下这个类是否合法，比如这个类是不是符合字节码的格式、变量与方法是不是有重复、数据类型是不是有效、继承与实现是否合乎标准等等。总之，这个阶段的目的就是保证加载的类是能够被jvm所运行。

   * 准备：为类中的静态变量分配内存并设置默认值

   * 解析：把常量池中的符号引用转换为直接引用

   > 比如我们要在内存中找一个类里面的一个叫做show的方法，显然是找不到。但是在解析阶段，jvm就会把show这个名字转换为指向方法区的的一块内存地址，比如c17164，通过c17164就可以找到show这个方法具体分配在内存的哪一个区域了。这里show就是符号引用，而c17164就是直接引用。在解析阶段，jvm会将所有的类或接口名、字段名、方法名转换为具体的内存地址。

3. 初始化：为静态变量和静态代码块执行初始化工作

> 一个类的初始化顺序：父类的静态成员变量 > 父类静态代码块 > 子类静态成员变量 > 子类静态代码块 > 父类成员变量 > 父类构造代码块 > 父类构造器 > 子类成员变量 > 子类构造代码块 > 子类构造器
>
> 这里要注意：在类的初始化阶段，只会初始化与类相关的静态赋值语句和静态语句，也就是有static关键字修饰的信息，而没有static修饰的赋值语句和执行语句在实例化对象的时候才会运行。

4. 使用：分为主动引用和被动引用，主动饮用会引起类的初始化，而被动引用不会引起类的初始化
   * 主动引用
     * 通过new关键字实例化对象、读取或设置类的静态变量、调用类的静态方法。
     * 通过反射方式执行以上三种行为。
     * 初始化子类的时候，会触发父类的初始化。
     * 作为程序入口直接运行时（也就是直接调用main方法）
   * 被动引用
     * 引用父类的静态字段，只会引起父类的初始化，而不会引起子类的初始化。
     * 定义类数组，不会引起类的初始化。
     * 引用类的常量，不会引起类的初始化。

5. 卸载：在类使用完之后，如果满足下面的情况，类就会被卸载：
   * 该类所有的实例都已经被回收，也就是java堆中不存在该类的任何实例。
   * 加载该类的ClassLoader已经被回收。
   * 该类对应的java.lang.Class对象没有任何地方被引用，无法在任何地方通过反射访问该类的方法。

​	 如果以上三个条件全部满足，jvm就会在方法区垃圾回收的时候对类进行卸载，类的卸载过程其实就是在方法区中清空类信息，java类的整个生命周期就结束了。

## ArrayList、LinkedList、Vector三者的异同

1. ArrayList

* 数据结构：动态数组（静态数组+扩容机制）
* 适用场景
  * 随机访问get和set，时间复杂度为O(1)
  * 插入和删除开销较大，因为要移动数据，并可能触发扩容
* 线程不安全

2. LinkedList

* 数据结构：双向链表

* 适用场景

  * 随机访问get和set，时间复杂度为O(n),因为要遍历链表
  * 插入和删除不需要移动数据，开销小。但是如果靠近末尾，ArrayList时间复杂度为O(1)，而LinkedList反而要遍历链表，时间复杂度为O(n)。所以只能说通常情况下，LinkedList更适合插入和删除，ArrayList更适合访问和更新。

* 线程不安全


3. Vector：与ArrayList相似，只不过是线程安全的，但开销跟大

## Java的IO模型？BIO和NIO的区别？

* BIO：同步阻塞IO，在此种方式下，用户进程在发起一个IO操作以后，必须等待IO操作的完成，只有当真正完成了IO操作以后，用户进程才能运行
* NIO：同步非阻塞IO，在此种方式下，用户进程发起一个IO操作以后边可返回做其它事情，但是用户进程需要时不时的询问IO操作是否就绪，这就要求用户进程不停的去询问，从而引入不必要的CPU资源浪费。
* AIO：异步非阻塞IO，此种方式下是指应用发起一个IO操作以后，不等待内核IO操作的完成，等内核完成IO操作以后会通知应用程序，这其实就是同步和异步最关键的区别，同步必须等待或者主动的去询问IO是否完成

[参考](https://blog.csdn.net/m0_38109046/article/details/89449305)

> 我认为：同步和异步讲究的是被调用方是不是直接返回，如异步就是直接返回调用，而同步就是要等被调用方处理完才返回，那么怎么算处理完，就得调用方一直等（阻塞）或者去主动轮询（非阻塞）。而阻塞和非阻塞讲究的是调用方主动不主动去等待（调用方线程挂起），等着就是阻塞，不等而去做别的事就是非阻塞

## 红黑树详解

一种二叉查找树，但在每个节点增加一个存储位表示节点的颜色，可以是红或黑（非红即黑）。通过对任何一条从根到叶子的路径上各个节点着色的方式的限制，红黑树确保没有一条路径会比其它路径长出两倍，因此，红黑树是一种弱平衡二叉树（由于是弱平衡，可以看到，在相同的节点情况下，AVL树的高度低于红黑树），相对于要求严格的AVL树来说，它的旋转次数少，所以对于搜索，插入，删除操作较多的情况下，我们就用红黑树。

1. 为什么要用红黑树：

我们知道平衡二叉查找树的查找效率非常稳定，为O(logn)，但由于它要遵循左右子树高度差不超过1的规则，所以在插入和删除操作时要进行左旋和右旋来维持平衡。因此当有大量的删除和操作的情况时，AVL树就不适合了，而红黑树就应运而生了。

2. 红黑树的特性：

* 每个节点或红或黑
* 根节点是黑色
* 每个叶子节点都是黑色的空节点
* 任何相邻的节点都不能同时为红色
* 从任意一个结点出发到空的叶子结点经过的黑结点个数相同

3. 红黑树与AVL树的比较:

* 红黑树对平衡要求更低，更擅长插入删除

* AVL树更擅长查找

  

## RPC协议

RPC协议是HDFS中节点之间的通信协议，简单来讲就说在一台机器上面调用另机器上的程序，它主要是通过Java的序列化和反序列化来实现的，因为不同节点的信息主要是保存在java对象里面的嘛，序列化就是把java文件转化成字节的形式通过保存下来，然后通过网络发送到另一台机器上面。

## final是什么？可以被继承吗？

final关键字代表最终的、不可改变的

* 修饰类，表示不能被继承
* 修饰方法，表示不能被重写
* 修饰局部变量
  * 对于基本类型来说，不可变说的是变量当中的数据不可改变
  * 对于引用类型来说，不可变说的是变量当中的地址值不可改变

* 修饰成员变量：
  * 不变（同局部变量）
  * 对于final的成员变量，要么使用直接赋值，要么通过构造方法赋值。二者选其一。

String类就是用final修饰的。

## java集合的继承关系

Java 集合可分为Collection和Map两种体系

- Collection接口：单列数据，定义了存取一组对象的方法的集合
  - List：元素有序、可重复的集合
  - Set：元素无序、不可重复的集合
- Map接口：双列数据，保存具有映射关系“key-value对”的集合

![image-20240319130208875](C:\Users\pc\AppData\Roaming\Typora\typora-user-images\image-20240319130208875.png)



## java的基本数据类型有哪些，和对应的引用数据类型有什么区别？有了Integer等引用数据类型为什么还需要int等基本数据类型？

整型：byte、short、int、long

字符型：char

浮点型：float、double

| 基本数据类型 | 包装类    |
| ------------ | --------- |
| byte         | Byte      |
| short        | Short     |
| int          | Integer   |
| long         | Long      |
| float        | Float     |
| double       | Double    |
| char         | Boolean   |
| boolean      | Character |

1. 有了Int，为什么还要用Integer?

​	主要是因为面向对象的思想，因为Java语言是面向对象的，这也是它只所以流行的原因之一，对象封装有很多好处，可以把属性也就是数据跟处理这些数据的方法结合在一起，比如Integer就有parseInt()等方法来专门处理int型相关的数据，另一个非常重要的原因就是在Java中绝大部分方法或类都是用来处理类类型对象的，如ArrayList集合类就只能以类作为他的存储对象，而这时如果想把一个int型的数据存入list是不可能的，必须把它包装成类，也就是Integer才能被List所接受。所以Integer的存在是很必要的.
​	int默认值为0，Integer默认为null。即Integer可以区分出未赋值和值为0的区别，int则无法表达出未赋值的情况，例如，要想表达出没有参加考试和考试成绩为0的区别，则只能使用Integer

2. 有了Integer，为什么还要用Int?

​	基本类型都是直接存储在栈内存中，不需要动态分配内存。栈的存取速度比堆快，因此基本类型变量的操作比引用类型变量更快。

## HashMap底层通过什么操作进行去重？

当我们添加元素时，HashMap会自动处理哈希冲突和键的比较，以保证不会出现重复的键。详情见8（HashMap的实现原理）



## Java多线程了解吗？写个生产者消费者模型吧（wtf？上来就搞这个）

## 遍历集合的方式;for和迭代器遍历哪个更快

1. 遍历集合的方式

* 迭代器

* 普通for循环

* 增强for循环

```java
List<String> list = new ArrayList<>();
list.add("a");
list.add("b");
list.add("c");

//迭代器
Iterator<String> iterator = list.iterator();
while(iterator.hasNext()) {
    System.out.println(iterator.next());
}
//普通for循环
int size = list.size();
for (int i = 0; i < size; i++) {
    System.out.println(list.get(i));
}

//增强for循环（foreach）
for (String s : list) {
    System.out.println(s);
}
```

2. 遍历速度

* 遍历顺序存储集合，如ArrayList，for≈foreach>iterator：for直接通过索引获取数据
* 遍历链式存储集合，如LinkedList，iterator>for≈foreach：for每次遍历需要重新获取位置，而iterator直接将指针指向下一个节点就行

增强for循环是语法糖，底层就是iterator

## 两个double相加，比如0.1+0.1=0.200001这类问题如何解决

* 使用精度更高的数据类型

​	可以使用BigDecimal或其他高精度数值类型来避免浮点数精度问题。使用BigDecimal时需要注意，需要使用BigDecimal的构造方法来创建BigDecimal对象，而不能直接使用浮点数作为参数

```java
BigDecimal a = new BigDecimal("0.1");//传入string类型的才可避免精度丢失问题
BigDecimal b = new BigDecimal("0.1");
BigDecimal c = a.add(b);
System.out.println(c); // 输出0.2
```



## HashMap和TreeMap的区别

* 数据结构：HashMap是数组+链表+红黑树；TreeMap是红黑树
* 排序区别：HashMap不排序，TreeMap按照key排序
* null值区别
  * HashMap：key和value都可以为null
  * TreeMap：key不可以为null,value可以为null
* 性能（使用场景）
  * HashMap：插入、删除和定位元素效率高
  * TreeMap：适用于按自然顺序或自定义顺序遍历键
* 线程安全：两者都不是线程安全的

## String是线程安全的？

String是不可变的（参考String为什么是不可变的），所以是线程安全的。

## JVM熟悉吗？设置栈大小的参数是啥？

| jvm内存参数 | 含义                                        |
| ----------- | ------------------------------------------- |
| -xms        | 初始化时，最小堆内存                        |
| -xmx        | 初始化时，最大堆内存                        |
| -xmn        | Java Heap Young区大小，不熟悉最好保留默认值 |
| -xss        | 设置每个线程的栈大小                        |

## ThreadLocal原理

原理：ThreadLocal存入值时使用当前ThreadLocal实例作为key，线程的变量副本作为value，存入当前线程对象中的Map中去

* 每个Thread线程内部都有一个Map（ThreadLocalMap::threadlocals）;
* Map里面存储ThreadLocal对象（key）和线程的变量副本（value）;
* 对于不同的线程，每次获取副本值时，别的线程不能获取当前线程的副本值，就形成了数据之间的隔离。

作用：实现线程范围内的局部变量，即ThreadLocal在一个线程中是共享的，在不同线程之间是隔离的，这样就可以做到线程之间对于共享变量的隔离问题

与锁的使用场景区别：

|        | syncronized(锁)                                              | ThreadLocal                                                  |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 原理   | 同步机制采用了时间换空间的方式，只提供一份变量，让不同线程排队访问（临界区排队） | 采用空间换时间的方式，为每一个线程都提供一份变量的副本，从而实现同时访问而不互相干扰 |
| 侧重点 | 多个线程之间访问资源的同步                                   | 多线程中让每个线程之间的数据相互隔离                         |



## Java为什么不用多继承

* 避免命名冲突：假如C继承A和B,而A和B有相同的方法或属性，此时C就不知道调用谁的方法或属性，产生歧义性

## 不可变类和可变类

- 什么是java的不可变类Immutable和可变类Mutable
  举个例子，String和StringBuilder，两个类功能上很多相似点。String是一个不可变对象，对于String的每一次修改都会产生一个新的String对象（不严谨）。StringBuilder是一个可变类，对象的每一次修改是对其本身的一个修改，不产生新的对象。
- 那么如何设计一个不可变类？对象一旦创建，便不可被修改。
  - 属性设为private final。类也是final的，不可被继承，被重载，
  - 只保留类似getter的访问方法，无setter。并且，如果成员变量是一个引用类型，那么getter返回的是该引用对象的一个拷贝（这个才是最关键的地方）。

## 

## 什么是socket？

​	在网络编程中，网络上的两个程序通过一个双向的通信连接实现数据的交换，这个连接的一端称为一个socket(网络套接字，端口号与IP地址的组合)

```java
public class test   {
    public static void main(String[] args) {
    }
    //客户端
    @Test
    public void client()  {
        Socket socket = null;
        OutputStream os = null;
        try {
            //1.创建Socket对象，指明服务器端的ip和端口号
            InetAddress inet = InetAddress.getByName("192.168.85.1");
            socket = new Socket(inet,8899);
            //2.获取一个输出流，用于输出数据
            os = socket.getOutputStream();
            //3.写出数据的操作
            os.write("你好，我是客户端HH".getBytes());
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4.资源的关闭
            if(os != null){
                try {
                    os.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(socket != null){
                try {
                    socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        }
    }
    //服务端
    @Test
    public void server()  {

        ServerSocket ss = null;
        Socket socket = null;
        InputStream is = null;
        ByteArrayOutputStream baos = null;
        try {
            //1.创建服务器端的ServerSocket，指明自己的端口号
            ss = new ServerSocket(8899);
            //2.调用accept()表示接收来自于客户端的socket
            socket = ss.accept();
            //3.获取输入流
            is = socket.getInputStream();

            //不建议这样写，可能会有乱码
//        byte[] buffer = new byte[1024];
//        int len;
//        while((len = is.read(buffer)) != -1){
//            String str = new String(buffer,0,len);
//            System.out.print(str);
//        }
            //4.读取输入流中的数据
            baos = new ByteArrayOutputStream();
            byte[] buffer = new byte[5];
            int len;
            while((len = is.read(buffer)) != -1){
                baos.write(buffer,0,len);
            }

            System.out.println(baos.toString());

            System.out.println("收到了来自于：" + socket.getInetAddress().getHostAddress() + "的数据");

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(baos != null){
                //5.关闭资源
                try {
                    baos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(is != null){
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(socket != null){
                try {
                    socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(ss != null){
                try {
                    ss.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```



## java中object的方法有哪些？

![image-20230312113311051](C:\Users\pc\AppData\Roaming\Typora\typora-user-images\image-20230312113311051.png)

* equals：等价于==；String等类对其重写了。
* getclass：返回Class类型的对象
* hashCode：返回该对象的哈希码值
* toString：
* wait：多线程时用到的方法，作用是让当前线程进入等待状态，同时也会让当前线程释放它所持有的锁。直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，当前线程被唤醒

## Java1.8的新特性你了解么？stream流了解吗？



## java基础类型，以及int和long的大小

| 数据类型  | 字节大小 | 位宽  | 默认值   | 取值范围                                         |
| --------- | -------- | ----- | -------- | ------------------------------------------------ |
| `byte`    | 1 字节   | 8 位  | 0        | -128 到 127                                      |
| `short`   | 2 字节   | 16 位 | 0        | -32,768 到 32,767                                |
| `int`     | 4 字节   | 32 位 | 0        | -2^31 到 2^31-1(-2,147,483,648 到 2,147,483,647) |
| `long`    | 8 字节   | 64 位 | 0L       | -2^63 到 2^63-1                                  |
| `float`   | 4 字节   | 32 位 | 0.0f     | ±1.4E-45 到 ±3.4028235E38                        |
| `double`  | 8 字节   | 64 位 | 0.0d     | ±4.9E-324 到 ±1.7976931348623157E308             |
| `char`    | 2 字节   | 16 位 | '\u0000' | 0 到 65,535                                      |
| `boolean` | 不定     | 不定  | `false`  | `true` 或 `false`                                |



## 装箱和拆箱

 **自动装箱**：Java 自动将基本数据类型转换为对应的包装类型对象

```java
int a = 5;
Integer boxedA = a; // 自动装箱，等价于 Integer boxedA = Integer.valueOf(a);

```

 **自动拆箱**：Java 自动将包装类型转换为基本数据类型

```java
Integer boxedA = 10;
int a = boxedA; // 自动拆箱，等价于 int a = boxedA.intValue();
```

Integer等包装类的拆箱与装箱过程中会有什么问题

1. 频繁的拆装箱会有性能开销
2. 空指针异常

```java
Integer intWrapper = null;
int i = intWrapper;  // 会抛出 NullPointerException
因此，在拆箱前，必须确保对象不为 null。
```

3. 对象比较问题：包装类对象是引用类型，不同于基本类型的值比较。如果直接使用 `==` 比较 `Integer` 对象，比较的是引用地址而不是值。这可能会导致意外的结果：

   ```java
   java复制代码Integer a = 1000;
   Integer b = 1000;
   System.out.println(a == b);  // 输出 false，因为它们是两个不同的对象
   ```

   对于小范围值（-128 到 127），`Integer` 对象会被缓存并复用，因此：

   ```java
   java复制代码Integer a = 100;
   Integer b = 100;
   System.out.println(a == b);  // 输出 true，可能因为它们指向同一个缓存对象
   ```

   如果希望比较包装类的值，应使用 `.equals()` 方法：

   ```java
   System.out.println(a.equals(b));  // 输出 true
   ```



# 二、jvm

## 1. jvm,jre,jdk的区别和联系？

​	jvm∈jre∈jdk

* jvm用于将字节码文件编译成具体平台的机器码（准确来说是解释执行字节码，并用jit编译热点字节码为机器码），这就是java能做到“一次编译，到处运行”的原因
* jre=jvm+lib，提供了jvm运行字节码文件用到的java核心类库（运行时类库）
* jdk=jre+编译工具，他是java的开发工具包，有了jdk便可以编译和运行java程序

## 2. jvm的主要组成部分及其作用？

jvm由四个部分组成：类加载器、运行时数据区（虚拟机栈、本地方法栈、堆、方法区、程序计数器）、执行引擎（解释器，jit即时编译器、垃圾回收器）和本地库接口组成。

jvm运行流程：首先通过编译器将java程序编译成字节码，类加载器再将字节码文件加载到内存中，放在运行时数据区的方法区内，由于字节码文件只是jvm的一套指令集，不能直接交给底层操作系统去执行，所以由执行引擎将其解释为机器码，交由cpu执行，而整个过程中会需要调用其他语言的方法，所以需要用到本地库接口。

![img](https://img-blog.csdnimg.cn/20211008091315501.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5byg57u06bmP,size_20,color_FFFFFF,t_70,g_se,x_16)

## 3. 运行时数据区（jvm内存结构）？

* 虚拟机栈：由栈帧组成（栈帧代表一个被调用的方法），包含了局部变量表，操作数栈，方法返回地址、指向运行时常量的引用、附加信息

* 本地方法栈：java调用本地的c/c++的方法，用于和底层操作系统交互

* 堆：存放对象实例和数组（jdk1.6)

* 方法区：类信息、常量、静态变量、即时编译后的代码等数据

* 程序计数器：用于保存jvm下一条所要执行的指令地址，用于多线程切换时，让线程知道自己执行到哪了

  方法区和堆是线程共享的，其他三个是线程私有的。
  
  虚拟机栈和本地方法栈为什么是私有的？这是为了保证线程中的局部变量不被其他线程访问到

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200329234127908.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc2NzAxNQ==,size_16,color_FFFFFF,t_70)

## 4. 为什么用元空间取代永久代？

	* 永久代使用jvm内存，容易发生内存溢出的问题，而元空间放在本地内存中，内存上限高，不容易出现oom的问题
	* 减少gc扫描的时间，提升性能

为什么字符串常量池放在堆里？

元空间gc频率低，放在堆里可以及时的进行gc，腾出空间

## 5. 浅拷贝与深拷贝的区别？

* 浅拷贝复制的是引用，即新旧引用指向堆中的同一个对象
* 深拷贝复制的是堆中完整的对象，然后让新引用指向这个新对象，新旧对象互不干扰。

## 6. JVM如何加载一个类的过程，什么是双亲委派模型？

jvm的类加载分为五个阶段（类的生命周期的前三个阶段）：

* 加载：将类的字节码载入方法区，根据class文件的描述创建class对象
* 验证：验证class字节流的数据是否符合jvm规范
* 准备：为类中的静态变量分配内存并设置默认值
* 解析：将常量池中的符号引用解析为直接引用
* 初始化：为静态变量和静态代码块执行初始化工作

双亲委派模型：如果一个类加载器收到了类加载的请求，它首先不会自己去加载这个类，而是让父类加载器去加载，一直传递到最顶层的启动类加载器。只有当父类加载器无法加载时，才会反向传递给子类加载器加载。

Java中提供如下四种类型的加载器：

* 启动类加载器：加载JAVA_HOME\lib下的核心类库

* 扩展类加载器：加载JAVA_HOME\lib\ext下的所有类库

* 应用类加载器：加载用户类路径的所有类库，这是默认的类加载器

* 自定义加载器：加载用户自定义路径下的类包

  ![img](https://img-blog.csdnimg.cn/20201217213314510.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NvZGV5YW5iYW8=,size_16,color_FFFFFF,t_70)

 双亲委派机制有什么优点：

* 避免了类的重复加载：双亲委派模型要求一个类加载器在尝试加载一个类时，首先委托其父类加载器进行加载。这个过程一直向上递归，直到顶层的启动类加载器。这样做的好处是，当一个类已经被一个在委派链更高位置的类加载器加载时，下面的类加载器就不会再次加载该类，从而避免了类的重复加载。这种机制确保了Java类（尤其是java.*包中的类）的唯一性，因为这些核心类库由根类加载器加载，而根类加载器对于JVM来说是唯一的
* 保护了程序的安全性，防止核心的API被修改：通过双亲委派模型，Java环境的核心类库，如Java API的类，都是由根类加载器来加载的。这意味着用户定义的类加载器无法替代这些核心类库中的类。这样就防止了恶意代码替换标准的Java API类库，从而提高了Java平台的安全性

破坏双亲加载模型

* 为什么要破坏

  * 因为在某些情况下父类加载器需要委托子类加载器去加载class文件。受到加载范围的限制，父类加载器无法加载到需要的文件。以JDBC为例，DriverManager是被启动类加载器加载的，而Driver的实现类基本都是第三方提供的，第三方的类不能被启动类加载器加载。

  * 无法加载相同的类的不同版本：不同的应用程序可能会依赖同一个第三方类库的不同版本，但是不同版本的类库中某一个类的全路径名可能是一样的。



* 如何破坏

  * 自定义类加载器加载一个类需要：继承ClassLoader，重写findClass，如果不想打破双亲委派模型，那么只需要重写findClass；如果想打破双亲委派模型，那么就重写整个loadClass方法，设定自己的类加载逻辑。想要打破即重写的时候让自己去加载不让父加载器去加载。
  * 使用线程上下文类加载器:如果创建线程时还未设置，他将从父线程中继承一个，如何在应用程序的全局范围内都没有设置的话，那这个类加载器默认就是应用程序类加载器

  ```java
  public class Main{
  	public static void main(String[]args){
          ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
      }
  }
  ```

  

## 7. 讲一讲GC？

垃圾回收主要在堆中进行，所以垃圾一般指的是在堆中存在的，可以被销毁的对象。

① 如何判断是垃圾

* 引用计数法：堆中每个对象都有一个引用计数器，对象刚创建时计为1，当有一个地方引用它时，计数器+1，当这个引用被释放时（超过生命周期），计数器-1，值为0时，就成为垃圾
  * 优点：实现简单
  * 缺点：难以检测循环引用（两个对象互相引用->计数器不能归0->无法回收->内存泄露）

* 可达性分析法：如何判断一个对象是否可达，就需满足以下两个条件中的一项：

  * 对象属于根对象
  * 对象被一个可达的对象引用

  简单来说，就是看该对象是否有一条路径可以到达根对象

  根对象：

  * 系统类

  * 虚拟机栈引用的对象
  * 本地方法栈应用的对象
  * 活动线程使用的对象
  * 正在加锁的对象
  * 方法区中常量引用的对象
  * 方法区中类静态属性引用的对象
  * ...

② 垃圾回收算法

* 标记清除算法：该算法分为标记和清除两个阶段，首先标记出所有要被回收的对象，再统一回收掉被标记的对象，标记的过程就是可达性分析判定为垃圾对象的标记过程
  * 优点：不需要进行对象的移动，当存活的对象较多时，性能高
  * 缺点：标记和清除的效率都不高，当要回收的对象很多时，效率低；会产生大量不连续的内存碎片，导致之后要为大对象分配内存时找不到连续的内存空间而触发另一次垃圾回收。
  
* 标记复制算法：将内存区域分成大小相等的两块，当第一块内存用完时，将存活的对象移动至第二块内存，再把已使用的内存清理掉。
  * 优点：每次都是针对整个半区进行回收，分配内存时不用考虑内存碎片问题，按顺序分配即可，即运行简单高效。解决内存碎片问题
  * 缺点：当存活对象很多时，复制的效率会很低；每次使用一半的区域，空间浪费大；

* 标记整理算法：标记所有需要被回收的对象->将所有存活的对象移动至内存的一段->将边界外的内存空间全都回收掉
  * 优点：解决内存碎片问题；解决复制效率低和空间利用率低的问题
  * 缺点：会暂停所有应用程序来垃圾回收(因为移动需要更新对象引用），所以得综合考虑利弊
  
* 分代回收算法：收集器根据对象存活的不同生命周期将堆分为了新生代和老年代两个区域。新生代存活生命周期短的对象（如刚生成的对象），老年代存放生命周期长的对象。空间占比，默认为新生代：老年代=1：2。

  ​	一般来说是将新生代划分为一块较大的Eden空间和两块较小的Survivor空间，每次使用Eden空间和其中的一块Survivor空间，当进行回收时，将Eden和Survivor中还存活的对象复制到另一块Survivor空间中，然后清理掉Eden和刚才使用过的Survivor空间。

  * 新生代：区域分为eden、survivor from和survivor to三个部分，默认占比是8：1：1，采用的是标记复制算法。
    * 首先新生成的对象放在eden区
    * 当eden区满了后，触发minor GC，根据可达性分析算法将存活下来的对象复制到to区域中，然后清理掉eden区，同时to中对象的寿命+1
    * from和to交换位置，保证垃圾回收前to为空
    * 第二次垃圾回收时，将eden区和from中存活的对象复制到to区域中，寿命+1,再交换from和to区域，如此往复
    * 每经历一轮垃圾回收寿命加1，寿命超过15时，进入老年代。
  * 老年代：寿命超过15or大对象，都会放入老年代（大对象复制的开销大，并且会导致新生代空间不够进而频繁minor gc),老年代可采用标记清除or整理算法。

③ 垃圾回收器：

* Serial：新生代收集器，单线程、简单高效，采用的是标记复制算法，会stop the world

* ParNew：Serial的多线程版本

* CMS(concurrent mark sweep)：分为四个阶段

  * 初始标记：标记出GC roots能直接关联到的对象，速度很快，会stw
  * 并发标记：继续GC roots 追溯的过程，与用户线程并发执行
  * 重新标记：修正在并发标记期间，因用户线程运行而导致标记发生变动的那一部分，需要stw
  * 并发清除：清除GC roots不可达的对象，与用户线程一起工作

  虽然初始标记和重新标记会stw，但最耗时的部分是并发标记和清除，这两部分是与用户线程并发执行的，所以总体而言达到了近似并发的目的；适用于停顿时间短，响应时间快的场景。

  ![image-20221210151728211](C:\Users\pc\AppData\Roaming\Typora\typora-user-images\image-20221210151728211.png)

缺点：

* 内存碎片
* 浮动垃圾（并发清理时用户线程也在运行，则会产生新垃圾，这些垃圾没有被标记，则只能等到下一次垃圾回收时再清理）
* 对处理器资源敏感，并发阶段，虽然不会暂停用户线程，但会占用处理器资源，导致应用程序变慢，降低吞吐量。





G1：将内存分为多个大小相等的region,每个region又标记为eden,survivor,老年代和H(用来大对象)。G1垃圾回收阶段大致可分为：

* 初始标记
* 并发标记
* 最终标记
* 筛选回收

前三个阶段与cms大致类似，最后一个阶段会根据回收价值、成本和用户期待的停顿时间来指定回收计划,并且会stw（因此不会产生浮动垃圾）。

优点：

* region之间采用标记复制算法，整体采用标记整理算法，所以不会产生内存碎片问题
* 使用region,不会存在新生代or老年代分配过大造成浪费
* 垃圾回收时是回收垃圾最多的region（垃圾回收价值最大的region),而不是回收整个堆，大大减少stw的时间；此外，g1的每个region维护了一个记忆集，用来记录region之间的依赖关系，这样只需要扫描关联的region而不是整个堆。
* 可以指定最大停顿时间



## 8. CMS和G1区别？

* CMS只能用于老年代垃圾回收，G1都行

* CMS采用标记清除，G1region之间采用标记复制/标记整理，所以前者会有内存碎片问题。
* CMS会产生浮动垃圾，因为CMS在并发清除阶段没有stw，而G1有stw
* G1以region为单位进行垃圾回收（回收垃圾最多的region)，大大减少stw时间，并且可以指定最大停顿时间
* G1由于每个region都要维护记忆集，所以内存占用高。因此小内存应用中cms优于g1

记忆集的作用：用来解决跨代引用问题（minor gc时，如果gc roots位于老年代，那就要扫描整个老年代区域，效率低，因此在新生代维护一个记忆集，记忆集以卡表的形式实现，卡表就是一个字节数组，数组的每一个元素（卡页）都指代了一块内存区域，当这个区域存在跨代引用时，就把值置为1，称为脏页。因此，在垃圾回收时，只需要将脏页区域加入gc roots扫描。(G1的卡表结构要比这个一般结构更复杂）（卡表状态用写屏障来维护）

## 9. Minor GC、Major GC、Full GC？

minor gc：新生代eden区空间满时发生，只清理新生代

major gc：老年代空间满时发生，只清理老年代

full gc：清理新生代、老年代和方法区

触发条件：

* 调用system.gc
* 老年代空间不足
* 方法区空间不足
* 通过minor gc进入老年代的对象大小大于老年代可用空间大小
* 空间分配担保：minor gc中，eden和survivor from中存活对象向to区域复制时，如果to 区域空间不足，就会把to区域无法容纳的对象送入老年代，但如果老年代空间不足以容纳（如何判断无法容纳：因为在这次回收中活下来的对象大小在完成回收前是无法得知的，所以就取每一次晋升至老年代的对象大小的平均值作为经验值，来和老年代剩余空间比），则会触发full gc。

解决：

* **调整 JVM 参数**：根据应用程序的内存需求合理设置 JVM 参数，增加堆内存大小
* **监控和诊断**：使用 JVM 监控工具，如 VisualVM 或 JConsole，来监控内存使用情况和 GC 事件
* **选择合适的垃圾收集器**：根据应用程序的特点选择合适的垃圾收集器，以减少 Full GC 的发生

## 内存泄漏和内存溢出

内存泄漏：是指程序在申请内存后，无法释放已申请的内存空间，导致系统无法及时回收内存并且分配给其他进程使用。通常少次数的内存无法及时回收并不会到程序造成什么影响，但是如果在内存本身就比较少获取多次导致内存无法正常回收时，就会导致内存不够用，最终导致内存溢出。

内存溢出：指程序申请内存时，没有足够的内存供申请者使用，导致数据无法正常存储到内存中。

## 内存发生泄漏，怎么查找问题出现的地方？

1. 导致内存泄漏的原因？

* 静态属性导致内存泄露：会导致内存泄露的一种情况就是大量使用static静态变量。在Java中，静态属性的生命周期通常伴随着应用整个生命周期

```java
public class StaticTest {
    public static List<Double> list = new ArrayList<>();
 
    public void populateList() {
        for (int i = 0; i < 10000000; i++) {
            list.add(Math.random());
        }
        Log.info("Debug Point 2");
    }
 
    public static void main(String[] args) {
        Log.info("Debug Point 1");
        1
        Log.info("Debug Point 3");
    }
}
解决方案：第一，尽量减少静态变量；第二，如果使用单例，尽量采用懒加载
```

* 未关闭的资源：无论什么时候当我们创建一个连接或打开一个流，JVM都会分配内存给这些资源。比如，数据库链接、输入流和session对象。忘记关闭这些资源，会阻塞内存，从而导致GC无法进行清理。
* 不当的equals方法和hashCode方法实现

```java
public class Person {
    public String name;
    
    public Person(String name) {
        this.name = name;
    }
}
@Test
public void givenMap_whenEqualsAndHashCodeNotOverridden_thenMemoryLeak() {
    Map<Person, Integer> map = new HashMap<>();
    for(int i=0; i<100; i++) {
        map.put(new Person("jon"), 1);
    }
    Assert.assertFalse(map.size() == 1);
}

//上述代码中将Person对象作为key，存入Map当中。理论上当重复的key存入Map时，会进行对象的覆盖，不会导致内存的增长。但由于上述代码的Person类并没有重写equals方法，因此在执行put操作时，Map会认为每次创建的对象都是新的对象，从而导致内存不断的增长。
```



## 再说几个java的知识吧，线程创建方式有哪几种？线程池有哪几种？看过源码没？都是用哪些BlockingQueue实现的？线程池满了会怎样

## 如果要设计一个线程池, 需要考虑哪些要素. Executors工厂类能创建哪些线程池, 用过哪些

## 一个Java应用上线后, 关注哪些性能指标. 如果响应时间过长或者CPU占用过高, 如何排查, 用哪些工具或命令

## System.gc()的条件，调用System.gc()一定会触发垃圾回收嘛

​	提醒可以进行垃圾回收：在调用System.gc()时，JVM会根据当前的内存使用情况和垃圾回收策略等因素来判断是否需要执行垃圾回收。如果JVM认为当前内存不足，或者执行垃圾回收可以显著减少内存占用，那么就会立即执行垃圾回收操作。否则，JVM也有可能不立即执行垃圾回收，而是等待更合适的时机再执行。

## 谈谈GC的发展过程

**Serial GC**、**Parallel GC**、**CMS**、**G1垃圾回收器**、**低延迟垃圾回收器（如ZGC和Shenandoah）**

## 类的初始化顺序

* 父类静态变量、父类静态代码块（执行顺序由出现顺序决定）

* 子类静态变量、子类静态代码块（执行顺序由出现顺序决定）
* 父类非静态变量、父类非静态代码块（执行顺序由出现顺序决定）
* 父类构造方法
* 子类非静态变量、子类非静态代码块（执行顺序由出现顺序决定）
* 子类构造方法

## 讲一下CMS，说一下它的过程。为什么用要用标记清除，而不是用其他的算法？

1. **并发性**：CMS设计的主要目标是减少应用程序停顿时间，特别是针对老年代的垃圾回收。标记-清除算法允许垃圾回收过程中的大部分工作与应用线程并发执行。这样，它可以在应用程序运行时并行地进行垃圾标记和清除，从而减少停顿时间。
2. **空间效率**：与标记-复制算法相比，标记-清除算法不需要额外的空间来存放复制过程中的存活对象。在处理老年代这样的大内存区域时，这一点尤其重要，因为老年代通常包含大量存活对象，且对象的生命周期较长。标记-清除算法可以直接在原地清理死亡对象，避免了复制存活对象所需的大量额外空间

## volatile关键字，出题两线程同时对volatile变量执行自增操作，分析可能出现的情况



# 三、juc

## 1. 进程和线程？

程序是静态文件，把程序编译成二进制文件加载到内存里，并为其分配资源，开始执行之后，就成为了进程（动态的，程序的一次执行过程）。线程属于进程，它是进程的一条执行路径。举个通俗的例子，正在运行的软件迅雷就是一个进程，而下载的多个任务就是多个线程。

具体展开，进程和线程的区别：

* 根本区别：进程是操作系统资源分配的基本单位，线程是cpu任务调度和执行的基本单位
* 包含关系：进程包含多个线程，线程属于进程
* 拥有资源：进程拥有自己独立的代码和数据空间（程序上下文）；线程之间共享代码段和数据空间
* 内存分配：同一进程的线程共享地址空间；进程之间有着独立的内存地址
* 开销：进程的创建、切换和销毁的开销都要大于线程

那么进程切换开销为什么比线程大?

​	切换指的是上下文切换，上下文是进程运行所保存的状态，保存了上下文后，进程下次运行时才知道从哪里开始重新运行。上下文包含：用户栈、内核栈、寄存器的内容等等。而进程的上下文切换比线程多一个虚拟内存的切换（最主要的原因）

​	虚拟内存：

* 虚拟内存是计算机系统内存管理的一种技术，它允许程序在逻辑上使用更多的内存空间，而不受物理内存大小的限制。通过虚拟内存技术，操作系统为每个程序提供了一个看似连续的地址空间，这个地址空间被称为虚拟地址空间。然而，这些虚拟地址并不直接对应物理内存中的实际位置。相反，它们通过内存管理单元（MMU）的页表映射转换成物理地址，
* 当内存不够用时，会匀出一部分磁盘作为虚拟的内存来使用，也就是把不常用的程序先放入磁盘（外存）中，等之后要用时再加载入物理内存。
* 还有一个很重要的作用就是内存保护，不同的进程拥有各自的虚拟内存，虚拟内存通过页表映射为不同的物理内存地址，这样就不会出现进程1占用进程2的物理内存而导致进程崩溃

​	由于不同的进程有着各自的虚拟内存空间，也就维护了各自的页表（将虚拟内存映射为物理内存），然而页表查询是一个缓慢的过程，因此就需要用到cache(TLB)，chache记录了常用的地址映射）。因此进程切换会导致页表也要切换，也就意味着cache失效，这样的话查询效率就会大幅降低，表现出来就是应用程序变慢了。而线程之间是共享进程的虚拟内存的，也就不存在切换页表，缓存失效等问题。

线程的五个状态

* 新建
* 就绪
* 运行
* 阻塞
* 终止

![image-20230126101429135](C:\Users\pc\AppData\Roaming\Typora\typora-user-images\image-20230126101429135.png)

java线程的状态：

- 线程**初始**状态：`NEW`
- 线程**运行**状态：`RUNNABLE`
- 线程**阻塞**状态：`BLOCKED`
- 线程**等待**状态：`WAITING`
- **超时等待**状态：`TIMED_WAITING`
- 线程**终止**状态：`TERMINATED`

![Java 线程状态变迁图](https://oss.javaguide.cn/github/javaguide/java/concurrent/640.png)

### blocked和waiting有啥区别

1. BLOCKED是锁竞争失败后被被动触发的状态，WAITING是人为的主动触发的状态；
2. BLCKED的唤醒时自动触发的，而WAITING状态是必须要通过特定的方法来主动唤醒。

## 2. 什么线程安全与线程不安全？

​	多个线程访问共享变量时得到的运行结果与预期一致时就是线程安全的（或者说多线程并行运行的结果和单线程串行运行的结果能一致就是线程安全的）。通俗来说，加锁的就是线程安全的，不加锁就是线程不安全的。（要保证原子性、可见性和有序性）

如何保证线程安全？



## 3. 介绍一下java内存模型？

1. 定义：JMM：描述了多线程访问变量的一种规范

​	JMM规定了所有的变量都存储在主内存中，主内存是多线程共享的，线程不能直接读写主内存中的数据，而是将其拷贝到自己的工作内存中再运算。线程不能直接访问其他线程的工作内存，线程间变量值的传递要通过主内存来，也就是说线程1在自己的工作内存中更新值后刷新回主内存，然后再被其他线程拷贝回自己的工作内存。

2. 为什么要有JMM：为了屏蔽各种硬件和操作系统的内存访问差异，使得java程序在各种平台下都能有一致的内存访问效果。

3. JMM的三个特征：

* 原子性：一个or多个操作，要么都被执行，要么都不执行，不能只执行一半，也不能被外界因素（别的线程）干扰。这里要注意的是：单个操作是原子的，但组合到一起就不是原子的。

  * 举例：i++分为三个步骤：线程从主内存种读取i的值到工作内存，+1，再把值更新到主内存。但这样会被别的线程干扰，如线程1读取i的值0后，线程2将其更新为1，这样线程1读取到的值就是旧值了，就不能保证原子性了。（volatile只能保证你读取到当前最新的值0，但如果线程2再将其更新为1后再进行后续的操作的话，就是操作旧值了）
  * 可用syncronized保证原子性
  * CAS

* 可见性：当一个线程对共享变量进行修改后，其他线程能立即看到该变量修改后的值

  * 可用volatile实现

  volatile如何实现可见性：每次在工作内存中修改值是都会立即同步到主内存，使用一个值时都会从主内存中刷新

  * 可用syncronized实现：保证同一时间只能有一个线程获得锁，然后执行同步方法，并在释放锁前将变量的修改刷新至主内存
  * CAS

* 有序性：在多线程环境下，指一组操作之间遵守一定的顺序规则，确保程序行为的一致性和可预测性

  * 可用volatile、syncronized、Lock相关工具包
  * happens-before实现自然的有序性，即不用额外的代码控制

4. happens-before：用于定义多线程程序中内存操作的顺序规则，保证可见性和有序性，如果操作X happens-before Y，那么X的结果一定对于Y是可见的，且X执行顺序在Y之前。这其实只是对程序员的保障，是程序员需要遵循的，实际上，只要重排序后不影响结果，编译器可以随意优化。也就是说编译器要做到优化后的结果要与按照Happens-Before原则执行的结果一致，happens-before原则是对编译器优化的一种约束。

具体的规则：

- 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。
- 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。
- volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
- 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。
- start()规则：如果线程A执行操作ThreadB.start()（启动线程B），那么A线程的ThreadB.start()操作happens-before于线程B中的任意操作。
- join()规则：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回。
- 程序中断规则：对线程interrupted()方法的调用先行于被中断线程的代码检测到中断时间的发生。
  对象finalize规则：一个对象的初始化完成（构造函数执行结束）先行于发生它的finalize()方法的开始。
- ...



## 4. volatile如何保证有序性？

打破有序性的罪魁祸首就是指令重排序，可用volatile来避免指令重排序

指令重排序：在保证数据依赖性的情况下，代码执行顺序和编写顺序可以不一致

- 好处：指令2的执行无需等待指令1的执行完成，这样可以大大提升cpu处理性能，串行语义下没有问题，但在多线程下可能导致问题，如单例模式下：

  ```java
  public class Singleton {
      private Singleton (){}
  
      private static boolean isInit = false;
      private static Singleton instance;
  
      public static Singleton getInstance() {
          if (!isInit) {//判断是否初始化过
              instance = new Singleton();//初始化
              isInit = true;//初始化标识赋值为true
          }
          return instance;
      }
  }
   
  ```

面试时通俗点的描述：单例模式下有对象就返回该对象，没有则创建。线程1在创建对象前先将标记置为true，这样线程2通过标记判断后以为有对象了，就将对象返回了，而实际上对象还未创建，所以就会报错。

volatile避免指令重排序的原理：

通过内存屏障的方法保证：

* 当执行到volatile读or写操作时，其前面的操作已经全部完成，其后面的操作还没执行；
* 不能将volatile变量前的语句放到其之后，不能将后面的语句放到volatile之前

## 5. 乐观锁和悲观锁？

1. 定义：

* 乐观锁：总是假设最好的情况，每次去拿数据时都认为别人不会修改，所以不会上锁，但是在更新的时候会判断别人是否修改过数据，如果修改过，则重新读取数据和更新，可以使用CAS算法实现。适用于多读情形。

* 悲观锁：总是假设最坏的情况，每次去拿数据时都认为别人会修改，所以会上锁，只有当自己修改完后才会释放锁，让给别的线程。常见的有syncronized，reentrantlock,数据库中的行锁，读写锁。适用于多写的情形。

2. 实现

* 乐观锁的实现：CAS（Compare and Swap)

  CAS：使用了3个操作数：内存地址V、旧的预期值A、要修改的新值B

  只有在当前内存地址V中的值等于A时，才会将V中的值改为B。（通俗来说，就是先将内存中的值取出来（A），再进行更新操作之前,将A值与当前内存中的实际值比较，如果一样就更新，不一样就说明被别的线程修改过，那么就要重新读取和更新，也就是CAS自旋）

  CAS的缺点：

  * 只能保证单个共享变量的原子性操作

  * ABA：线程1获取内存值A后阻塞，线程2将A改为B，线程3将B改回A，此时线程1察觉不到值被修改过，比较了下，发现当前内存值和刚刚获取的内存值都是A，于是执行更新操作

    ABA会带来的影响：普通场景下ABA可能并不会造成不良影响，某些业务场景下可能会导致问题，例如：我银行账户有100块，需要取出来50块，但由于提款机硬件出问题导致提交了2次，理想情况肯定是只有一个提交成功，只扣50块。实际可能发生：线程1和线程2都读取了值100，线程1阻塞，线程2成功更新为50，此时线程3再将值从50改到100（汇款操作），线程1恢复运行时发现旧值与当前内存值都是100，则进行扣除操作。

    如何解决：引入版本号机制，一旦别的线程修改了内存值，就将版本号+1，这样就算经历了ABA，也能察觉到修改。

  * CPU开销过大：高并发下，多线程写的情况下，可能导致更新一直失败，然后不断自旋，增加cpu压力。

    CAS只能保证原子性（只能保证单个共享变量），因此可以配合volatile保证并发三特性，即实现线程安全

* 悲观锁的实现：

  * syncronized
  * reentrantlock

## 6. Syncronized详解？

1. 定义：Synchronized是Java中解决并发问题的一种最常用的方法，也是最简单的一种方法，它是可重入、不公平的重量级锁
2. 用法：

* 修饰实例方法，给当前实例加锁
* 修饰静态方法，给当前类对象加锁
* 修饰代码块，指定加锁对象，对给定对象加锁

3. 实现原理

* 当我们执行Syncronized(Obj)时，就会将Obj中对象头里的MarkWord指向一个Monitor，这个Monitor就是我们所谓的锁，Monitor由三部分组成：Owner、Entrylist和WaitSet。
* Owner所关联的线程，就是当前获取了锁，也就是执行了Syncronized(Obj)的线程。
* 假如线程1是Owner，而现在又来了一个线程2执行Syncronized（Obj)，那线程2就会陷入阻塞状态，进入EntryList。如果此时线程1释放了锁，位于阻塞队列（EntryList)中的线程2和线程3就可以去争夺锁，谁抢到谁就是Owner，不是按照谁先来谁就能获取锁的顺序，因此是Syncronize的是非公平的。
* Owner线程发现条件不满足时，调用wait方法，即可进入WaitSet，调用notify/notifyall时唤醒，进入EntryList，参与锁的竞争

<img src="https://seazean.oss-cn-beijing.aliyuncs.com/img/Java/JUC-Monitor工作原理1.png" style="zoom:67%;" />

4. 锁升级：JDK1.6版本为了弥补Synchronized的性能缺陷，设计了Synchronized锁的膨胀升级。

* 无锁：当对象锁被创建出来时，在线程获得该对象锁之前，对象处于无锁状态
* 偏向锁：当一个线程访问同步块并获取锁时，锁就进入偏向模式。当这个线程再次请求锁时，无需任何同步操作，这减少了不必要的同步开销（如锁状态检查和转换、线程挂起和唤醒等）。只有当另一个线程尝试获取这个锁时，偏向模式才会被取消，并且锁会升级到轻量级锁
* 轻量级锁：如果有多个线程要加锁，但加锁时间是错开的，不存在竞争，则获取的是轻量级锁
* 重量级锁：多个线程要加锁，cas自旋获取锁失败，即多个线程存在竞争关系，则升级为重量级锁

Syncronized为什么被称为重量级锁：早期，Synchronized属于重量级锁，效率低下，当一个线程尝试获取一个已被其他线程持有的锁时，该线程会阻塞，在操作系统层面上，这会导致线程状态的转换（从运行状态到阻塞状态），伴随着上下文切换的开销。上下文切换涉及保存和加载不同线程的上下文信息，这是一个成本相对较高的操作，特别是在高度争用的环境下，频繁的锁竞争会导致大量的上下文切换。



## 7. ReentrantLock详解？

ReentrantLock的基本实现可以概括为：先通过CAS尝试获取锁。如果此时已经有线程占据了锁，那就加入AQS队列并且被挂起。当锁被释放之后，排在CLH队列队首的线程会被唤醒，然后CAS再次尝试获取锁。在这个时候，如果：

非公平锁：如果同时还有另一个线程进来尝试获取，那么有可能会让这个线程抢先获取；

公平锁：如果同时还有另一个线程进来尝试获取，当它发现自己不是在队首的话，就会排到队尾，由队首的线程获取到锁。



AQS：AbstractQuenedSynchronizer抽象的队列式同步器。**AQS的核心思想**是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并将共享资源设置为锁定状态，如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。

用大白话来说，AQS就是基于CLH队列，用volatile修饰共享变量state，线程通过CAS去改变状态符，成功则获取锁成功，失败则进入等待队列，等待被唤醒。（**注意：AQS是自旋锁：**在等待唤醒的时候，经常会使用自旋（while(!cas())）的方式，不停地尝试获取锁，直到被其他线程获取成功）



gpt介绍：`ReentrantLock` 是 Java 中的一个高级同步机制，提供比 `synchronized` 关键字更灵活的锁定和解锁操作。它支持可重入性，允许线程重复获取已持有的锁，提供了公平和非公平锁选择，支持中断处理，并允许绑定多个条件对象以实现复杂的同步。`ReentrantLock` 主要用于那些需要高级锁定功能的场景，如实现公平性策略或响应中断等，但它也要求开发者更加注意锁的正确管理和释放。

## 8. ReentrantLock和Syncronized和区别

相同点：

* 都是可重入锁（同一线程可多次获得本对象上的锁），可重入锁意义：防止死锁
* 都是悲观锁

不同点

* syncronized依赖于JVM实现，ReentrantLock依赖于API
* syncronized由编译器来保证加锁释放锁，ReentrantLock需要手动加锁释放锁

* ReentrantLock实现等待可中断：持有锁的线程长期不释放的时候，正在等待的线程可以选择放弃等待，这相当于Synchronized来说可以避免出现死锁的情况
* ReentrantLock实现公平锁：多个线程等待同一个锁时，必须按照申请锁的时间顺序获得锁，Synchronized锁非公平锁，ReentrantLock默认的构造函数是创建的非公平锁，可以通过参数true设为公平锁，但公平锁表现的性能不是很好
* ReentrantLock锁绑定多个条件，一个ReentrantLock对象可以同时绑定对个对象。ReenTrantLock提供了一个Condition（条件）类，用来实现分组唤醒需要唤醒的线程们，而不是像synchronized要么随机唤醒一个线程要么唤醒全部线程。
* 性能区别：高并发线程竞争激烈时，ReentrantLock性能更好，多线程竞争不激烈时Syncronized性能好。原因：ReentrantLock有利用CAS自旋操作来实现锁，在大并发的情况下synchronized线程间切换会有很大的资源消耗。

## 9. 说一说ConcurrentHashMap

1. ConcurrentHashMap的实现原理

 jdk1.7下：

 ConcurrentHashMap由Segment数组和HashEntry数组构成。即ConcurrentHashMap将哈希桶数组切分成一段一段的小数组，即Segment，而Segment又由n个HashEntry组成。每一段segment实现了reentrantlock锁，保证在同一时间内，一段segment只能被一个线程访问，并且当这段segment被锁住时，别的segment依旧能被其他线程访问，实现了并发访问。

jdk1.8：

ConcurrentHashMap与HashMap的数据结构一样，由Node数组、链表和红黑树组成。在锁的实现上，抛弃了原有的Segment分段锁，而是通过CAS+Syncronized实现了更细粒度的锁，只会锁住单个node节点，这样就不会影响别的数组元素的读写，大大提高并发度。

JDK1.7 与 JDK1.8 中ConcurrentHashMap 的区别？

* 数据结构：取消了 Segment 分段锁的数据结构，取而代之的是数组+链表+红黑树的结构。
* 保证线程安全机制：JDK1.7 采用 Segment 的分段锁机制实现线程安全，其中 Segment 继承自 ReentrantLock 。JDK1.8 采用CAS+synchronized保证线程安全。
* 锁的粒度：JDK1.7 是对需要进行数据操作的 Segment 加锁，JDK1.8 调整为对每个数组元素加锁（Node）。

2. JDK1.8 中为什么使用内置锁 synchronized替换可重入锁 ReentrantLock？

3. ConcurrentHashMap 和 Hashtable 的效率哪个更高？为什么？

   ConcurrentHashmap效率跟高，因为它实现了更细粒度的锁，提高了并发度。

4. 具体说一下Hashtable的锁机制

​	Hashtable是使用syncronized来实现线程安全的，但它会给整个哈表表上一个大锁，这样在多线程下，只要有一个线程获取锁，其他线程就只能等待锁释放，在高并发场景下性能很差

## 11.start()方法和run()方法的区别？

start方法：

* 启动当前线程，当前线程进入就绪态
* 调用run方法

start方法会执行线程前的相应准备工作，然后在执行run方法运行线程体，这才是真正的多线程工作。只有调用了start方法才能启动新的线程，而直接运行run方法只会在主线程里执行run这个普通方法。

## 12.创建多线程的方式

1. 概述

* 继承Thead类的方式
  * 创建一个继承于Thread类的子类
  * 重写run方法
  * 创建Thread类的子对象
  * 调用run方法
  
* 实现Runnable接口的方式
  * 创建一个实现Runnable接口的类	
  * 重写run方法
  * 创建一个该实现类的对象
  * 将该对象作为参数传递给Thread类的构造器，创建Thread类的对象
  * 通过Thread类的对象调用start()

* 实现Callable接口的方式

  * 创建一个实现Callable接口的类

  * 重写call方法

  * 创建一个该实现类的对象

  * 将该对象作为参数传给FutureTask的构造器中，创建一个FutureTask队形

  * 将该FutureTask对象作为参数传给Thread类的构造器中，创建Thread类对象

    （FutureTask同时实现了Future和Runnable接口，也就是说FutureTask可以作为Runnable任务提交给线程池）

  * 调用start方法

  * 获取call方法的返回值

* 使用`ExecutorService`线程池

* 使用`CompletableFuture`类

2. 继承和实现方式的区别

开发中优先选择：实现Runnable接口的方式

原因：

* 实现的方式没有类的单继承性的局限性
* 实现的方式更适合用来处理多个线程有共享数据的情况（共用同一个实现类对象）

3. Runnable和Callable区别

callable有返回值，能对外抛出异常而runnable不行，所以当我们需要知道任务执行的结果时，callable是个更好的选择

4. 代码实现展示

方式1：

```java
/**
 * 多线程的创建，方式一：继承于Thread类
 * 1.创建一个继承于Thread类的子类
 * 2.重写Thread的run()方法 ---> 将此线程的方法声明在run()中
 * 3.创建Thread类的子对象
 * 4.通过此对象调用start()
 *
 * 例子:遍历100以内的所有的偶数
 */

//1.创建一个继承于Thread类的子类
class MyThread extends Thread{
    //重写Thread类的run()
    @Override
    public void run() {
        for(int i = 1;i < 100;i++){
            if(i % 2 == 0){
                System.out.println(i);
            }
        }
    }
}

public class ThreadTest {
    public static void main(String[] args) {
        //3.创建Thread类的子对象
        MyThread t1 = new MyThread();

        //4.通过此对象调用start():①启动当前线程 ②调用当前线程的run()
        t1.start();

        //如下操作仍在main线程中执行的
        for(int i = 1;i < 100;i++){
            if(i % 2 == 0){
                System.out.println(i + "***main()***");
            }
        }
    }
}

```

方式2：

```java
/**
 * 创建多线程的方式二：实现Runnable接口
 * 1.创建一个实现了Runnable接口的类
 * 2.实现类去实现Runnable中的抽象方法:run()
 * 3.创建实现类的对象
 * 4.将此对象作为参数传递到Thread类的构造器中，创建Thread类的对象
 * 5.通过Thread类的对象调用start()
 */

/**
 *  比较创建线程的两种方式。
 *  开发中：优先选择：实现Runnable接口的方式
 *  原因：1. 实现的方式没有类的单继承性的局限性
 *      2. 实现的方式更适合来处理多个线程有共享数据的情况。
 *  
 *  联系：public class Thread implements Runnable
 *  相同点：两种方式都需要重写run(),将线程要执行的逻辑声明在run()中。
 */

//1.创建一个实现了Runnable接口得类
class MThread implements Runnable{

    //2.实现类去实现Runnable中的抽象方法:run()
    @Override
    public void run() {
        for(int i = 0;i < 100;i++){
            if(i % 2 == 0){
                System.out.println(Thread.currentThread().getName() + ":" + i);
            }
        }
    }
}

public class ThreadTest1 {
    public static void main(String[] args) {
        //3.创建实现类的对象
        MThread m1 = new MThread();
        //4.将此对象作为参数传递到Thread类的构造器中，创建Thread类的对象
        Thread t1 = new Thread(m1);
        //5.通过Thread类的对象调用start():①启动线程 ②调用当前线程的run() --> 调用了Runnable类型的target的run()
        t1.start();

        //再启动一个线程，遍历100以内的偶数
        Thread t2 = new Thread(m1);
        t2.setName("线程2");
        t2.start();
    }
}

```

方式3：

```java
import java.util.concurrent.Callable;
import java.util.concurrent.FutureTask;

class CallableExample implements Callable<String> {
    private final int num;

    public CallableExample(int num) {
        this.num = num;
    }

    @Override
    public String call() throws Exception {
        Thread.sleep(1000); // 模拟计算
        return "线程执行结果: " + num * num;
    }
}

public class Main {
    public static void main(String[] args) throws Exception {
        CallableExample callable1 = new CallableExample(10);
        FutureTask<String> futureTask1 = new FutureTask<>(callable1);

        Thread thread1 = new Thread(futureTask1);
        thread1.start();

        // 获取计算结果
        String result = futureTask1.get();
        System.out.println(result);
    }
}

```



方式4：

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadPoolExample {

    public static void main(String[] args) {
        // 创建一个固定大小的线程池
        ExecutorService executor = Executors.newFixedThreadPool(3);

        // 提交多个任务到线程池执行
        executor.execute(new Task("Task 1"));
        executor.execute(new Task("Task 2"));
        executor.execute(new Task("Task 3"));

        // 关闭线程池，不再接受新任务，已提交的任务继续执行
        executor.shutdown();
    }

    static class Task implements Runnable {
        private final String name;

        Task(String name) {
            this.name = name;
        }

        @Override
        public void run() {
            System.out.println("Executing : " + name + " inside : " + Thread.currentThread().getName());
            try {
                Thread.sleep(1000); // 模拟任务执行耗时
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                System.out.println("Task " + name + " was interrupted.");
            }
            System.out.println("Task " + name + " completed.");
        }
    }
}

```

方式5：

```java
public class UseCompletableFuture {
    public static void main(String[] args) throws InterruptedException {
        CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> {
            System.out.println("5......");
            return "zhuZi";
        });
        // 需要阻塞，否则看不到结果
        Thread.sleep(1000);
    }
}
```



## 13. 线程池详解

1. 什么是线程池？线程池有什么好处？

通俗来说，就是管理线程的池子。它可以容纳多个线程，其中的线程可以反复利用，省去频繁创建销毁的开销

好处：

* 降低资源开销：线程复用，减少线程创建和销毁的消耗
* 提高响应速度：当有任务到达时，可以直接拿池子里的线程，省去创建的过程
* 提高线程的可管理性：避免无限制的创建，通过线程池进行统一的分配、调优和监控

2. 有几种常见的线程池（必知必会）?

* 定长线程池（FixedThreadPool）
* 定时线程池（ScheduledThreadPool）
* 可缓存线程池（CachedThreadPool）
* 单线程化线程池（SingleThreadExecutor）

核心概念：这四个线程池的本质都是ThreadPoolExecutor对象

不同点在于：

* FixedThreadPool：只有核心线程，线程数量固定，执行完立即回收，任务队列为链表结构的有界队列。
* ScheduledThreadPool：核心线程数量固定，非核心线程数量无限，执行完闲置 10ms 后回收，任务队列为延时阻塞队列。
* CachedThreadPool：无核心线程，非核心线程数量无限，执行完闲置 60s 后回收，任务队列为不存储元素的阻塞队列。
* SingleThreadExecutor：只有 1 个核心线程，无非核心线程，执行完立即回收，任务队列为链表结构的有界队列

## 线程池哪几种，参数有哪些，拒绝策略，线程池状态工作原理


Java 线程池主要通过 `java.util.concurrent.ExecutorService` 接口实现，具体的线程池实现类主要在 `java.util.concurrent.Executors` 和 `java.util.concurrent.ThreadPoolExecutor` 类中。下面是几种常见的线程池类型及其用途：

1. **FixedThreadPool**：固定数量的线程池。创建一个固定线程数的线程池，任何时候最多有N个线程处于活跃状态，并且如果一个线程因为执行异常而结束，那么线程池会补充一个新线程。
2. **CachedThreadPool**：缓存线程池。创建一个可缓存线程的线程池，如果线程池的当前规模超出了处理需求，将回收空闲（60秒不执行任务）线程，当需求增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制。
3. **SingleThreadExecutor**：单线程化的线程池。创建一个单线程的Executor，确保所有的任务都在同一个线程中按顺序执行。
4. **ScheduledThreadPool**：计划任务线程池。创建一个线程池，它可以安排在给定延迟后运行命令或者定期地执行。

**线程池的关键参数**：

- corePoolSize：核心池的大小。
- maximumPoolSize：最大线程数。
- keepAliveTime：当线程数大于核心时，这是多余空闲线程在终止前等待新任务的最长时间。
- TimeUnit：keepAliveTime 的时间单位。
- workQueue：用来存储等待执行的任务的阻塞队列。
- threadFactory：线程工厂，用来创建线程。
- handler：拒绝策略，当任务太多来不及处理，用来拒绝任务。

**拒绝策略(RejectedExecutionHandler)**：

1. **AbortPolicy**：直接抛出异常。
2. **CallerRunsPolicy**：只用调用者所在线程来运行任务。
3. **DiscardPolicy**：不处理，直接丢弃掉。
4. **DiscardOldestPolicy**：丢弃队列中等待最久的任务，然后把当前任务加入到队列中尝试再次提交。

**线程池的状态**：

- RUNNING：接受新任务并处理排队任务。
- SHUTDOWN：不接受新任务，但处理排队任务。
- STOP：不接受新任务，不处理排队任务，并中断正在进行的任务。
- TIDYING：所有任务都已终止，workerCount为零，线程转换到状态TIDYING将运行terminated()钩子方法。
- TERMINATED：terminated()已完成。

**工作原理**：

1. 如果当前运行的线程数小于核心线程数，那么就会新建一个线程来执行任务。

2. 如果当前运行的线程数等于或大于核心线程数，但是小于最大线程数，那么就把该任务放入到任务队列里等待执行。

3. 如果向任务队列投放任务失败（任务队列已经满了），但是当前运行的线程数是小于最大线程数的，就新建一个线程来执行任务。

4. 如果当前运行的线程数已经等同于最大线程数了，新建线程将会使当前运行的线程超出最大线程数，那么当前任务会被拒绝，拒绝策略会调用`RejectedExecutionHandler.rejectedExecution()`方法。

![图解线程池实现原理](https://oss.javaguide.cn/github/javaguide/java/concurrent/thread-pool-principle.png)

![线程池各个参数的关系](https://oss.javaguide.cn/github/javaguide/java/concurrent/relationship-between-thread-pool-parameters.png)

## 14. wait()和sleep()的区别?

共同点：都会暂停当前线程并让出cpu

不同点：wait()会释放锁，需要notify()唤醒，sleep不会释放锁

## 15. 死锁是什么？如何检测死锁？怎么预防死锁？

1. 死锁：两个或以上的线程互相持有对方所需要的资源而陷入等待

2. 死锁产生四大必要条件：

* 资源互斥：资源在同一时刻只能被一个线程使用

* 占有且请求：已经得到资源的进程或线程，继续请求新的资源，并持续占有旧的资源。

* 资源不可剥夺：资源已经分配进程或线程后，不能被其它进程或线程强制性获取，除非资源的占有者主动释放。

* 循环等待：死锁发生时，系统中必定有两个或两个以上的进程或线程组成一条等待环路

3. 产生死锁的原因

* 竞争资源
  * 竞争不可剥夺资源（如打印机资源，一个线程抢占后，别的线程只能等）
  * 竞争临时性资源
* 进程间（线程间）推进顺序不合理（6中死锁代码演示便是）

4. 如何检测

然后建立资源分配表和进程等待表

最简单的便是使用死锁检测工具

* Jconsole：对java应用程序的资源消耗和性能进行监控，提供可视化界面。你可以直接点击要检测的线程，选择死锁检测就ok

5. 怎么预防

破坏四大必要条件

* 破坏资源不可剥夺：设置超时时间，一定时间内未获取到想要的锁，就释放当前占有的锁（释放已占有的资源）
* 破坏占用且请求条件：一次性分配所有资源
  缺点：系统资源会被严重浪费，因为有些资源可能开始时使用，而有些资源结束时才使用。
* 破坏循环等待条件：给每个资源进行编号，进程或线程按照顺序请求资源，只有拿到前一个资源，才能继续请求下一个资源。（或者设置线程的优先级）
  缺点：资源的编号必须相对稳定，资源添加或销毁时会受到影响。

6. 死锁代码演示

```java
/**
 * 演示线程的死锁
 *
 * 1.死锁的理解：不同的线程分别占用对方需要的同步资源不放弃，
 *       都在等待对方放弃自己需要的同步资源，就形成了线程的死锁
 * 2.说明:
 *      》出现死锁后，不会出现异常，不会出现提示，只是所有的线程都处于阻塞状态，无法继续
 *      》我们使用同步时，要避免出现死锁。
 */
public class ThreadTest {
    public static void main(String[] args) {

        StringBuffer s1 = new StringBuffer();
        StringBuffer s2 = new StringBuffer();

        new Thread(){
            @Override
            public void run() {

                synchronized (s1){
                    s1.append("a");
                    s2.append("1");

                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    synchronized (s2){
                        s1.append("b");
                        s2.append("2");

                        System.out.println(s1);
                        System.out.println(s2);
                    }
                }
            }
        }.start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (s2){
                    s1.append("c");
                    s2.append("3");

                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    synchronized (s1){
                        s1.append("d");
                        s2.append("4");

                        System.out.println(s1);
                        System.out.println(s2);
                    }
                }
            }
        }).start();
    }
}

```



## 16. 同步和异步？线程同步和线程互斥？

**同步**：发出一个调用之后，在没有得到结果之前， 该调用就不可以返回，一直等待。

**异步**：调用在发出之后，不用等待返回结果，该调用直接返回

线程互斥：同一时刻，某一共享资源（代码块）只能被一个线程访问，具有唯一性和排他性

线程同步：在线程互斥的基础之上，实现线程访问的有序性。可用syncronized等保证

（同步是一种更为复杂的互斥，而互斥是一种特殊的同步）

## 多线程和异步的关系（多线程编程角度）

1. **多线程用于实现异步**：
   - 在多线程编程中，可以通过创建多个线程来执行不同的任务（如I/O任务和计算任务）。如果这些线程之间没有相互阻塞，那么这些任务就可以被认为是异步执行的。
   - 例如，一个线程处理I/O操作，另一个线程执行计算任务。它们在各自的线程中并行执行，而不会阻塞对方，从而实现异步操作。
2. **异步的实现方式**：
   - 异步可以通过多线程来实现。例如，Java中的`CompletableFuture`和线程池。
   - 异步也可以在单线程环境中实现。例如，JavaScript中的事件循环机制，Node.js中的异步I/O操作。
3. **多线程不等于异步**：
   - 多线程编程并不总是异步的。多个线程可以并行执行任务，但如果它们之间存在同步机制（如锁、信号量等），则这些任务实际上是同步执行的。
   - 例如，两个线程在同一时刻需要访问同一个资源，但通过同步机制确保一个线程完成任务后，另一个线程才能开始。这种情况下，尽管使用了多线程，任务之间是同步的。

> 异步是任务之间不互相阻塞，但是任务本身可能是阻塞的，所以异步不等于非阻塞

## join和sleep的区别

### join()

1. **用途**：`join()`方法用于让当前执行线程等待`join`线程执行结束。其主要用途是确保线程按照一定的顺序执行。例如，如果线程`B`需要在线程`A`执行完后再执行，那么在线程`B`中调用`A.join()`会让线程`B`处于等待状态直到线程`A`完成。
2. **锁释放**：调用`join()`方法时，当前线程不会释放已经持有的任何锁资源。
3. **异常**：如果在等待期间线程被中断，`join()`会抛出一个`InterruptedException`。

### sleep()

1. **用途**：`sleep()`方法用于让当前执行的线程暂停执行指定的时间（使当前线程进入阻塞状态），让出CPU给其他线程，但是不会释放对象锁或监视器。它主要用于给其他线程运行的机会或者在特定的时间内让当前线程暂停操作。
2. **锁释放**：调用`sleep()`方法时，如果当前线程持有某个对象锁，即使线程睡眠，也不会释放锁。
3. **异常**：`sleep()`方法也可能在睡眠期间被中断，这种情况下它会抛出`InterruptedException`。

## 如何让A、B、C三个线程按C、B、A的方式执行

```java
public class ThreadOrderExecution {
    public static void main(String[] args) throws InterruptedException {
        Thread threadC = new Thread(() -> System.out.println("Thread C is running."));
        Thread threadB = new Thread(() -> {
            try {
                threadC.join();  // 等待线程C执行完成
                System.out.println("Thread B is running.");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        Thread threadA = new Thread(() -> {
            try {
                threadB.join();  // 等待线程B执行完成
                System.out.println("Thread A is running.");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        // 启动顺序不影响实际执行依赖逻辑，但这里我们按照A、B、C的顺序启动
        threadA.start();
        threadB.start();
        threadC.start();
    }
}

```





## 说一说多线程下进行 i++，有哪些方法保证安全



## threadlocal，三种原型

1. threadlocal定义：ThreadLocal叫做线程变量，意思是ThreadLocal中填充的变量属于当前线程，该变量对其他线程而言是隔离的，也就是说该变量是当前线程独有的变量。ThreadLocal为变量在每个线程中都创建了一个副本，那么每个线程可以访问自己内部的副本变量

![ThreadLocal 数据结构](https://oss.javaguide.cn/github/javaguide/java/concurrent/threadlocal-data-structure.png)

2. threadlocal和syncronized区别![image-20241027113133917](C:\Users\pc\AppData\Roaming\Typora\typora-user-images\image-20241027113133917.png)

3. 三种原型

| 类                         | 是否支持子线程继承 | 是否支持线程池传递 | 适用场景                                                     |
| -------------------------- | ------------------ | ------------------ | ------------------------------------------------------------ |
| `ThreadLocal`              | 否                 | 否                 | 线程局部变量，不需跨线程传递的场景                           |
| `InheritableThreadLocal`   | 是                 | 否                 | 父子线程继承的场景（子线程需为新建线程）                     |
| `TransmittableThreadLocal` | 是                 | 是                 | 专门为了解决 **在线程池** 或 **线程复用** 场景下传递父线程变量的问题 |






## 线程间通信的方式

1. 共享变量：

   线程可以通过共享变量进行通信，但是需要注意同步问题，通常会结合使用锁、条件变量等机制。

2. 互斥锁：

   互斥锁用于保护共享资源的访问。只有获得互斥锁的线程才能访问受保护的资源，其他线程会被阻塞直到锁被释放。

3. 条件变量：

   条件变量允许线程以无竞争的方式等待特定的条件满足。条件变量通常与互斥锁结合使用，以防止竞争条件。

4. 信号量：

   信号量是一种计数器，用于控制对共享资源的访问。信号量可以用于线程同步，防止多个线程同时访问共享资源。

5. 消息队列



# Spring 

## IOC

控制反转，简单点说，就是创建对象的控制权，被反转到了Spring框架上。通常，我们实例化一个对象时，都是使用类的构造方法来new一个对象，这个过程是由我们自己来控制的，而控制反转就把new对象的工交给了Spring容器。

IOC的主要实现方式：

* 依赖注入：由IoC容器动态地将某个对象所需要的外部资源（包括对象、资源、常量数据）注入到组件(Controller, Service等）之中。简单点说，就是IoC容器会把当前对象所需要的外部资源动态的注入给我们

## AOP

 1. 定义

    ​	面向切面编程，也是面向特定方法的编程。就是对于某一类方法，将与业务逻辑无关但共性的功能抽取出来集中到一个地方进行管理，达到解耦的目的。AOP 常用于对核心业务逻辑进行增强，而不改变其原始代码。Spring 框架通过动态代理和字节码操作实现了AOP。

 2. 如何实现

    AOP 的常见实现方式有动态代理、字节码操作等方式。

    Spring的AOP底层是基于动态代理技术来实现的，也就是说在程序运行的时候，会自动的基于动态代理
    技术为目标对象生成一个对应的代理对象。如果要代理的对象，实现了某个接口，那么 Spring AOP 会使用 **JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候 Spring AOP 会使用 **Cglib** 生成一个被代理对象的子类来作为代理

![SpringAOPProcess](https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/230ae587a322d6e4d09510161987d346.jpeg)



>在**静态代理**中，必须手动为每个目标类编写一个对应的代理类，这意味着如果有多个类需要增强，则需要为每一个类编写一个代理类。这样的代码是**重复且冗余**的，不便于维护。而**动态代理**通过反射机制，可以在运行时动态生成代理对象，不需要手动编写代理类，大大减少了代码的重复性



## 讲讲Spring依赖注入的方式

1. 字段注入(属性注入)

```java
@Component
public class UserService {

    @Autowired  // 自动注入依赖
    private UserRepository userRepository;
}

```

**优点**：简洁，代码量最少，适合简单的依赖注入场景。

**缺点**：不符合面向对象的最佳实践，因为对象的依赖关系隐藏在类内部，不易测试，特别是在单元测试时，无法直接在构造函数中传入依赖;更容易违背单一职责原则；不能注入final修饰的属性.

2. setter方法注入

```java
@Component
public class UserService {
    private UserRepository userRepository;

    @Autowired  // 自动注入依赖
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

```

**优点**：setter注入满足单一设计/职责原则(因为setter方法的特性,一个setter方法只对应一个对象,不会有注入多个对象的可能性,所以满足单一设计/职责原则)

**缺点**：不能注入final修饰的对象;由于setter方法是可以被多次调用的,有修改的风险,所以注入的对象就可能被修改.

3. 构造函数注入

```java
@Component
public class UserService {
    private final UserRepository userRepository;

    @Autowired  // 自动注入依赖
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

```

**优点**：

* 通用性更好：构造方法注入因为基于java的,JDK是最底层框架,所以无论在哪一个容器/框架都可以适用
* 注入的对象不会被修改：构造方法注入的对象不会被修改,因为构造方法只会执行一次.

**缺点**：构造方法可以注入多个对象,也就违背了单一设计原则;写法不简便





# SpringMVC

# SpringBoot

## 一、起步依赖

​	起步依赖就是引入一堆starter依赖，它们集成了一些常用依赖，我们只需要引入起步依赖，其他的依赖都会自动的通过Maven的依赖传递进来。比如：springboot-starter-web，这是web开发的起步依赖，在web开发的起步依赖当 中，就集成了web开发中常见的依赖：json、web、webmvc、tomcat等

## 二、自动配置

​	SpringBoot的自动配置就是当Spring容器启动后，一些配置类、bean对象就自动存入到了IOC容器中，不需要我们手动去声明，从而简化了开发，省去了繁琐的配置操作。

​	分析自动配置原理就是来解析在SpringBoot项目中，在引入依赖之后是如何将依赖jar包当中所定义的配置类以及bean加载到SpringIOC容器中的

**原理分析：**

​	自动配置原理源码入口就是@SpringBootApplication注解，在这个注解中封装了3个注解，分别

是：

* @SpringBootConfiguration：声明当前类是一个配置类

* @ComponentScan：进行组件扫描（SpringBoot中默认扫描的是启动类所在的当前包及其子包）

* @EnableAutoConfiguration
  * 封装了@Import注解（Import注解中指定了一个ImportSelector接口的实现类）
    * 在实现类重写的selectImports()方法，读取当前项目下所有依赖jar包中META-INF/spring.factories和META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports

​	当SpringBoot程序启动时，就会加载配置文件当中所定义的配置类，并将这些配置类信息(类的全限定

名)封装到String类型的数组中，最终通过@Import注解将这些配置类全部加载到Spring的IOC容器

中，交给IOC容器管理。

**简化来说**，就是@EnableAutoConfiguration会加载spring.factories和AutoConfiguration.imports中定义的配置类和其定义的bean，当然了，如果有**@Conditional**注解的话，那么就会按照一定的条件进行判断，在满足给定条件后才会注册对应的bean对象到Spring的IOC容器中

