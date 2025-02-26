Java泛型是java1.5中引入的一个新特性，其本质是参数化类型，也就是说所操作的数据类型被指定为一个参数。这种参数类型可以用在类、接口和方法的创建中，分别称为泛型类、泛型接口、泛型方法。

泛型的泛指的是方法或类适用场景的广泛，实现有以下三种方式:

- 泛型类，是在实例化类的时候指明泛型的具体类型。带有泛型的类中可以有一套方法、一组成员变量适用于所有类型的处理。

- 泛型方法，是在调用方法的时候指明泛型的具体类型 。带有泛型的方法则可以独立于类而产生变化，达到同样的效果，带来更细粒度地的“泛”。
- 泛型接口，是在对接口进行实现的时候指明泛型的具体类型，带有泛型的接口可以有多种实现类，同时还允许实现类声明具体类型，或继续持有泛型。

```java
/**
 * 泛型类：用于承载任意类型的消息
 */
public class HybridMessage<T> {
  private T message;
  
  public HybridMessge(T message) {
    this.message = message;
  }
  
  public T getMessage() {
    return this.message
  }
 
  /**
   * 泛型方法：用于打印任意类型的数组，且泛型上界为String
   */
	public static <E extends String> void print(E[] contentArray ) {
		for (E element : contentArray) {
			System.out.println(element.getBytes());
		}
	}
}

/**
* 泛型接口：存在一个快速发送器作为其实现类，且快速发送器适用任意类型
*/
public class quickSender<T> implements Sender<T> {
  
  private ArrayList<T> email = initEmial<T>();
  
  @Overrride
  void sendMessage(T message){
    System.out.println("zoom,zoom,send " + message + "success")
  }

  @Overrride
  T getFirst() { 
    System.out.println("zoom,zoom,first message")
    return email.get(0);
  }
} 
```

使用泛型的本质目的是：便利&安全

- 使得这些类或方法中的代码组织和编写工作可以被复用，即我们编写一次模板代码，便可以适应所有类型。

- 在使用含有泛型的类时，编译器会自动进行推断与检查，避免了我们人为的强制类型转换，减少了错误发生。



**以使用ArrayList为例：**

1、生产队的驴的做法

为了借助集合的功能来处理字符串，我们就按照源码的方式自己写一个集合类Arraylist4String，专门用于存储并操作String类型的元素。紧接着随着需求的扩大，如果我们想使用ArrayList处理Integer、Double、Date.....等等都需要按照此方式。要写无数的、重复的类程序Arraylist4Int，Arraylist4Date.......，太累了！

2、java引入泛型前的做法

要么，为了适应所有类型，Arraylist中的元素被定义成了Object类型，无论我们存储进集合的是什么类型，最终处理的时候都是按Object进行处理的，如果想要使用某些类的特性就必须我们人为手动强制向下转型，这引入了极大的不安全性，谁又能准确记得当初放的具体是什么类型呢？

3、java引入泛型后的做法

为了适应所有类型并保证便捷安全，ArrayList中的元素类型被声明为泛型<T>，每当我们对其进行实例化时，我们都可以对其中元素声明一个具体类型，例如ArrayList<String>，它被专门用于存储并操作String类型。在编译期，我们的操作会受编译器检查（不能违反类型的约定），随后编译器放心地进行了泛型擦除（让泛型回归为Object），由它在编译期自动完成类型的转换。



注意事项：

- 一旦确定了其泛型的具体类型，T就不再具有泛化能力了。一切都变得具体了起来（确定的字段类型，确定的方法参数，确定的方法返回值）。我们在对这个单位后续的使用中，也必须严格遵循这种具体的要求。

- 泛型继承关系 ArrayList<String> 可以转换为List<String>, 但是ArrayList<IntegerNumber> 不能转换为 ArrayList<Number>

- 泛型类中的静态方法和静态变量不可以使用泛型类所声明的泛型类型参数，但是可以使用泛型方法自身声明的泛型参数。
- if( arrayList instanceof ArrayList<String>) 以及 if (arrayList<String>.getClass() == arrayList<Ineger>.getClass() )）都是true



**使用场景**

A模块调用B模块

A模块的所提供的数据类型不唯一，但是都想借助B模块的能力进行处理，那么B模块就声明泛型。

B模块可以是类，也可以是方法。同时B也可以是接口。这就好比Map<K, V> 下面有hashMap，LinkedhashMap，这二者有不同的功能，但均支持所有类型。



在使用泛型时，想利用继承关系（一定程度上的可变性），那么一定要明确使用”有限定通配符“。声明函数参数的泛型。

返回值不支持<？extends T>,因为函数的返回值需要是”确定的“，如果想可变，我们可以自行用不同的类型来接收。

PECS原则

```java
// iterable 是一个生产者，生产的内容需要适配当前的使用，因此需要是当前泛型的子类
public void pushAll(Iterable<? extends T> iterable) {

}

// iterable 是一个消费者，消费的内容需要兼容当前的使用，因此需要是当前泛型的父类
public void popAll(Collection<? super T> collection) {

}
```

如果使用得当，通配符类型对于类的用户来说几乎是无形的 。 它们使方法能够接受它们应该接受的参数，并拒绝那些应该拒绝的参数 



一个复杂的例子

```java
public <T extends Comparable<T>> T max(Collection<T> collection) {
    
}

public <T extends Comparable<? super T>> T max(Collection<? extends T> collection) {
    
}
```

collection 提供对象是生产者

返回值是T始终是为消费者（Comparable<T> 也始终是消费者）

我们应该始终使用Comparable<? super T> 来代替 Comparable<T> 保证T是可与T的父类进行比较的





**不允许使用带有泛型的类型声明数组**

```java
Map<Integer, String>[] mapArray = new Map<Integer, String>[20];
```

Java数组是类型协变的，而集合不是，并且Java泛型会在编译之后被擦除。

上述代码在被编译之后，这个数组从外面看就是Map[]，因此可以转换为Object[]，那么随后我们便可以将Map<Stirng, Object>类型的对象放入该数组，此时类型已经不再安全。因此带有泛型的数组是不允许使用的。

```java
List<Map<String, String>> list = new ArrayList<>();
```

而使用泛型的集合，一旦声明了泛型，集合行为就与该类型所绑定，即List的元素类型不能再改变。所以在之后的使用中，能够保证存入的元素都是Map<String,String>类型，保证了类型安全。





**允许使用带有泛型的可变参数列表，当且仅当 可变参数列表只用来将参数传递到方法中**

1、没有在泛型可变参数数组中保存任何值

2、它没有对外部不信任的程序开放该数组。

泛型可变参数列表是建立在泛型数组之上的，泛型数组是在编译方法的时候创建的，用来保存可变参数。

其类型是Object[]，而若将此泛型数组外透出去后，可能会变为String[]，如此一来就产生了堆污染：堆中产生了坏数据，参数化类型（泛型）变量 指向了不是那种参数化类型的对象，这容易导致异常。



在Java7中，增加了SafeVarargs注解，它让带泛型vararg参数的方法的设计者能够自动禁止客户端的警告。

@SafeVarargs 注解 意味着设计者做出了承诺：我保证，这个带有泛型的可变参数列表是安全的.



