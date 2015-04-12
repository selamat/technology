### 1. 采用java实现单例模式
#### （1）最初级写法，看过面试宝典的人估计都这么写。
```java
public class Single {	private static Single single = null;		public static Single getInstance() {		if(single == null) {			single = new Single();		}		return single;	}}
```
#### (2) 稍微好点的会考虑到线程同步，但是线程开销比较大，仍然不够
```java
public class Single {	private static Single single = null;		public synchronized static Single getInstance() {		if(single == null) {			single = new Single();		}		return single;	}}
```
#### （3）双重检查锁定，避免在并发环境下被重复实例化，并且充分考虑到并发时对于性能的要求
```java
public static Single getInstance() {		if(single == null) {			synchronized(Single.class) {				if(single == null) {					single = new Single();				}			}				}		return single;
}
```


#### （4）上述写法考虑仍然不够充分，因为类仍然可以在外部被实例化，所以需要在类的代码中添加下述代码避免类在外部被实例化，这个是最完整的写法
```java
class Single {	
	private static Single single = null;
	
	private Single() {}
	
	public static Single getInstance() {		if(single == null) {			synchronized(Single.class) {				if(single == null) {					single = new Single();				}			}				}		return single;
	}
}
``` 
### 2. 有如下代码，如果运行正常请写出输出结果，如果异常则尝试在不删除现有代码的同时让程序运行正确，并解释原因
```java
class A {
	public int age;
	public String say;
	
	public A() {
		this.age = 12;
		say = "hello";
	}
	
	public static void main(String[] args) {
		A a = new A();
		A b = new A();
		HashMap<A, A> map = new HashMap<A, A>();
		map.put(a, a);
		
		System.out.println(map.get(b).age);
	}
}
```
#### 简述：输出NullPointerException，原因，hashmap存储对象会计算对象hashcode，a b作为两个不同的对象，hashcode不相等，所以将a放入map后，用b get无法获取需要的值。更正办法，在Class A中重载equals与hashcode方法：
```java
public int hashCode() {  
	return age;  
}  
 
public boolean equals(Object o) {  
	if ((o != null) && (o instanceof A))  
		return age == ((A) o).age;  
   else  
       return false;  
}  
```
#### new一个object作为key去拿value是永远得不到结果的，因为每次new一个object，这个object的hashcode在理论上是不同的，所以我们要重写hashcode，你可以令你的hashcode是object中的一个恒量，这样永远可以通过你的object的hashcode来找到key的地址，然后你要重写你的equals方法，使内存中的内容也相等

### 3. 下述代码是否运行正常，如果正常，请写出运行结果，如果不正常则解释原因并写出正确代码
```java
public static void iteratorTest() {        List<String> a1 = new ArrayList<String>();                a1.add("List01");        a1.add("List02");        a1.add("List04");                 Iterator<String> i1 = a1.iterator();        while (i1.hasNext()) {            Object obj = i1.next();            if (obj.equals("List02"))                a1.add("List03");        }
        System.out.print("集合：" + a1.size());
}
```
#### 答案：会抛出java.util.ConcurrentModificationException异常，在创建迭代器之后，除非通过迭代器自身的 remove 或 add 方法从结构上对列表进行修改，否则在任何时间以任何方式对列表进行修改，迭代器都会抛出ConcurrentModificationException，正确写法：

```java
public static void iteratorTest() {        List<String> a1 = new ArrayList<String>();              a1.add("List01");        a1.add("List02");        a1.add("List04");                 ListIterator<String> i1 = a1.listIterator();        while (i1.hasNext()) {            Object obj = i1.next();            if (obj.equals("List02")) {            	i1.remove();            	i1.add("List03");            }                   }        System.out.println(a1.size());}
```

### 4. 简述jvm中内存模型与常用的垃圾回收机制，并尝试分析各种垃圾回收算法在JAVA Heap中体现

#### 简述jvm ：jvm大致上有如下区域
>* program counter : 每一个线程都必须用一个独立的程序计数器，用于记录下一条要运行的指令
>* stack : 虚拟机栈用于存放函数调用堆栈信息。Java 虚拟机栈也是线程私有的内存空间，它和 Java 线程在同一时间创建，它保存方法的局部变量、部分结果，并参与方法的调用和返回。
>* heap : 堆用于存放 Java 程序运行时所需的对象等数据。几乎所有的对象和数组都是在堆中分配空间的。
>* 方法区（或持久代） : 主要存储乐类的类型信息、常量池、域信息、方法信息。

#### heap中又被划分成四个区代，eden(新生代)、survivor0(from区)  survivor1(to区) tenured(老年代)，当一个对象被创建后，存在于eden，经历过一次gc后仍然存活会被移入survivor0，之后再经过垃圾回收会在survivor0和1之间复制，当到达一定阀值之后会被移入tenured。

#### 简述垃圾回收 
>* 标记-清除 ：首先遍历对象图并标记可到达的对象，然后扫描堆栈以寻找未标记对象并释放它们的内存。但是这种方式很容易造成内存碎片。
>* 标记-压缩 ：第一步与标记-清除相同，但在第二步中，会将剩余的对象进行压缩，释放连续的内存空间。
>* 复制 ：这种收集器将堆栈分为两个域，常称为半空间。每次仅使用一半的空间，JVM生成的新对象则放在另一半空间中。
>* 增量收集 ：增量收集器把堆栈分为多个域，每次仅从一个域收集垃圾，也可理解为把堆栈分成一小块一小块，每次仅对某一个块进行垃圾收集。这会造成较小的应用程序中断时间，使得用户一般不能觉察到垃圾收集器正在工作。
>* 分代收集 ：把堆栈分为两个或多个域，用以存放不同寿命的对象，较好的体现在jvm heap内存模型中。

#### 简述各个代适用的算法
>* 新生代通常存活时间较短，因此基于Copying算法来进行回收，所谓Copying算法就是扫描出存活的对象，并复制到一块新的完全未使用的空间中。
>* 旧生代（from区和to区）与新生代不同，对象存活的时间比较长，比较稳定，因此采用标记(Mark)算法来进行回收，所谓标记就是扫描出存活的对象，然后再进行回收未被标记的对象，回收后对用空出的空间要么进行合并，要么标记出来便于下次进行分配，总之就是要减少内存碎片带来的效率损耗。

### 5. java中synchronized与lock机制有何异同

#### 简述：相同的地方都是锁，浅显的不同的地方用法不同，不过都是语法上的问题，更深层次来说，synchronized内部采用悲观锁机制，一条线程获得锁之后，其他线程会被阻塞等待直到锁被释放；但是lock内部基于乐观锁实现，所以在使用lock时，没有明显的加锁过程，如果对比发生了变化，也就意味着其他线程做了修改，则这个操作失败，做失败后的处理，可以考虑循环尝试。！！！显然考虑到系统底层的阻塞和唤醒的成本考虑，乐观锁通常会比悲观锁效率更好一些。

### 6. 现有一台机器A，外部应用程序向A的某一个端口持续不断发送数据，请设计一个程序，处理从该端口接收到的内容，概述设计思路即可。

#### 简述 ：这个其实向考一下JAVA NIO理论的活学活用，NIO中添加了channel，用来做读写分离，避免阻塞，该程序也可以这样设计，相似的设计思路产生的框架如flume。

### 7. 在1~100亿中随机选出10亿个数存入N个文件中（N>3），采用mapreduce进行排序，请设计方案

#### 简述：启动N个map从文件中读取数据并按照指定规则分发到指定的reduce上，规则是1~33亿分发至reduce1，33~66亿分发到reduce2，66~100亿分发到reduce3，第一轮reduce结束后，按照上述规则（规则更加细致）再次进行mapreduce迭代，依次类推最终得出结果

