这个资料挺好的：[java_interview](https://codejuan.gitbooks.io/java_interview/content/io/syn-ays-blocked/index.html)
# 什么是JIT
JIT (Just In Time Compile)即时编译器
将*.java编译成.class文件的是前端编译器，而相对应的还有后端编译器，它在程序运行期间将字节码转变为机器码(现在的Java程序在运行时基本都是解释执行加编译执行)，即时编译器就是后端编译器。

Java程序最初是仅仅通过解释器解释执行的，即对字节码逐条解释执行，这种方式的执行速度相对会比较慢，尤其当某个方法或代码块运行的特别频繁时，这种方式的执行效率就显得很低。于是后来在虚拟机中引入了JIT编译器（即时编译器），当虚拟机发现某个方法或代码块运行特别频繁时，就会把这些代码认定为“Hot Spot Code”（热点代码），为了提高热点代码的执行效率，在运行时，虚拟机将会把这些代码编译成与本地平台相关的机器码，并进行各层次的优化，完成这项任务的正是JIT编译器。

二者各有优势：当程序需要迅速启动和执行时，解释器可以首先发挥作用，省去编译的时间，立即执行；当程序运行后，随着时间的推移，编译器逐渐会发挥作用，把越来越多的代码编译成本地代码后，可以获取更高的执行效率。解释执行可以节约内存，而编译执行可以提升效率。

热点代码有两类：被多次调用的方法、被多次调用的循环体。

要知道一段代码是不是热点代码，需要进行热点探测(Hot Spot Detection),目前主要有两种热点判定方式：
* 基于采样的热点探测：采用这种方法的虚拟机会周期性地检查各个线程的栈顶，如果发现某些方法经常出现在栈顶，那这段方法代码就是“热点代码”。这种探测方法的好处是实现简单高效，还可以很容易地获取方法调用关系，缺点是很难精确地确认一个方法的热度，容易因为受到线程阻塞或别的外界因素的影响而扰乱热点探测。
* 基于计数器的热点探测：采用这种方法的虚拟机会为每个方法，甚至是代码块建立计数器，统计方法的执行次数，如果执行次数超过一定的阀值，就认为它是“热点方法”。这种统计方法实现复杂一些，需要为每个方法建立并维护计数器，而且不能直接获取到方法的调用关系，但是它的统计结果相对更加精确严谨。
HotSpot使用的是第二种，内部有两种计数器，方法调用计数器和回边计数器，回边计数器用于统计一个方法中循环体代码执行的次数。

参考:[Javac编译与JIT编译](http://blog.csdn.net/ns_code/article/details/18009455)

# 堆的结构
堆结构一般分为新生代和老年代，和永久代。
新生代又分为Eden、From Survivor, To Survivor，三者的比例默认是8:1:1,新生成的对象主要存储在新生代中,大部分对象在Eden空间中。

老年代：在年轻代中经历了N次垃圾回收后仍然存活的对象，就会被放到年老代中。因此，可以认为年老代中存放的都是一些生命周期较长的对象。

永久代就是方法区，但是不同的虚拟机方法区所处的位置也不同，有的是在堆中，有的可能不在堆中,永久代主要存储静态文件，如Java类，静态方法，字符串常量池的字符串等，但是到jdk1.8后就废弃了永久代，转到了元空间中。

为什么辞永久代，迎元空间？
1. 字符串在永久代中，容易出现性能问题和内存溢出
2. 类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致老年代溢出。
3. 永久代会为 GC 带来不必要的复杂度，并且回收效率偏低。

# 垃圾回收算法
标记-清除：标记阶段的任务是标记出所有需要被回收的对象，清除阶段就是回收被标记的对象所占用的空间，缺点：容易产生内存碎片
标记-整理：标记后将存活对象向一端移动。缺点:移动的代价较高
复制：它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用的内存空间一次清理掉。缺点：占用空间
分代：将内存划分为若干个区域，对不同的区域采用不同的垃圾回收算法。如JVM的新生代采用复制算法，老年代采用标记-整理算法。

# 垃圾收集器
垃圾收集器分为以下几种：串行收集器，并行收集器，并行合并收集器，并发标记清除收集器。除了CMS收集器采用标记清除算法，以及G1全部都是标记整理，其他都是新生代复制，老年代标记整理。
## 串行收集器
Serial/Serial Old:最基本最古老的收集器，单线程收集器，所以名字为Serial(串行),垃圾回收的时候会STW(stop the world)暂停其他所有线程。Serial针对新生代，采用复制算法，Serial Old针对老年代，采用标记整理。缺点是STW
## 并行收集器
ParNew:Serial的多线程版本，新生代收集器，复制算法。
## 并行合并收集器
Parallel/Parallel Old：Parallel Scavenge新生代多线程收集器，关注吞吐量，采用复制算法。Parallel Old老年代多线程收集器，采用标记整理算法。
所谓吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值，即吞吐量 = 运行用户代码时间 /（运行用户代码时间 + 垃圾收集时间），虚拟机总共运行了100分钟，其中垃圾收集花掉1分钟，那吞吐量就是99%。
## 并发标记清除收集器(CMS)
CMS(Concurrent Mark Sweep)，名字就是并发标记清除收集器，分四个阶段：初始标记，并发标记(前两个阶段STW)， 重新标记，并发清除。其采用的是并发清除算法，老年代收集器。
## G1
G1：标记整理算法。G1收集器是当今收集器技术发展最前沿的成果，使用G1收集器时，将整个Java堆划分为多个大小相等的独立区域（Region）。

# JVM可视化工具
JConsole:JConsole工具在JDK/bin目录下，启动JConsole后，将自动搜索本机运行的jvm进程，不需要jps命令来查询指定。双击其中一个jvm进程即可开始监控，也可使用“远程进程”来连接远程服务器。
VisualVM:VisualVM是一个集成多个JDK命令行工具的可视化工具
## JVM调优相关命令
* jps: linux的ps，列举当前所有的java虚拟机进程pid，`–l`输出主类或者jar的完全路径名
* jstat：用于收集HotSpot虚拟机各方面的运行数据．它可以显示本地或者远程虚拟机进程中的类装载，内存，垃圾收集，JIT编译等运行数据.如：`jstat -gc 8888`显示pid为8888的jvm虚拟机进程的相关数据。
* jinfo ：显示虚拟机配置信息，如：`jinfo 8888`
* jmap：生成虚拟机的内存转储快照，还可以查询finalize执行队列，Java堆和永久代的详细信息，如空间利用率，当前用的是哪种收集器等．

# Java内存模型
多线程通信方式一般有两种，一种是共享内存，一种是消息传递，Java中采用的共享内存的方式。在Java中所有的变量或对象都存储在主存中，每个线程又有自己独立的工作内存，将主存类比为CPU，那么工作内存就相当于告诉缓存。线程在操作某一对象时候，要经三个步骤：
1. 从主存中复制变量到工作内存中
2. 执行代码，改变工作内存中变量的值
3. 将工作内存中的值刷新到主存中去。

什么是工作内存？其实是对CPU寄存器和告诉缓存的抽象描述.CPU对数据的读取顺序一般为:CPU寄存器-高速缓存-主存

对于第三步，何时同步回去完全是由JVM决定.
JVM规范定义了线程对主存的操作指令：read,load,use,assign,store, write。
read-load是读取主存变量到工作内存的过程，store-write是刷新工作内存变量到主存的过程，而use就是将变量放到执行引擎，assign对变量进行赋值。

为什么synchronized是安全的，因为其每次获取锁后都要把变量从主存复制到工作内存，在释放锁之前，将工作内存变量刷新到主存中。synchronized保证了代码的可见性和有序性。
但是volatile只能保证代码的可见性，不能保证有序性。
参考:[java线程安全总结](http://blog.csdn.net/haolongabc/article/details/7249098)
# 线程join
线程join和线程yeild一起记忆就比较好记了，首先是yeild:
```
public static native void yield();
```
该方法是静态原生方法，该方法将让出当前CPU，将当前线程的状态从运行态转换为可运行状态。
线程join：
```
public final void join() throws InterruptedException
```
该方法表示等到当前线程运行结束,如`a.join()`表示a线程运行结束后，才会执行后续代码。

# 线程的状态
在不同书籍中对线程的描述不同，有说6种，也有说7种的

6种的版本：

* NEW:创建
* RUNNABLE:运行
* WATING:等待
* TIME-WAITING:限时等待
* BLOCKED:阻塞
* TEMINAL:终止

7种的版本：

* RUNNABLE:创建(可运行)
* RUNNING:运行
* SLEEPING:睡眠
* WAITING:等待
* BLOCKED ON IO:IO阻塞
* BLOCKED ON SYNCHRONIZED:同步阻塞
* DEAD：终结

# 线程池
通过Executors可以创建以下几种线程池
* newCachedThreadPool:可缓冲线程池，灵活回收线程，如果没有可回收的线程，则创建新的线程
* newFixedThreadPool:定长线程池，超出的线程会等待
* newScheduledThreadPool:定时或周期执行的线程池
* newSingleThreadExecutor:单线程化线程池，只有一个线程来工作，保证所有工作按序执行。
* newSingleThreadScheduledExcutor：single和scheduled合并的线程池，创建单一线程并定时或周期执行
* newWorkStealingPool：创建持有足够线程的线程池来支持给定的并行级别，并通过使用多个队列，减少竞争，它需要穿一个并行级别的参数，并行级别决定了同一时刻最多有多少个线程在执，如果不传，则被设定为默认的CPU数量.

不通过Executors可以创建线程池是ForkJoinPool,该线程池支持大任务分解成小任务的线程池，这是Java8新增线程池，通常配合ForkJoinTask接口的子类RecursiveAction或RecursiveTask使用。

# CountDownLatch
闭锁，主要使用场景是，当前任务想要执行下去，必须要等到其他任务执行完毕后才能继续执行。主要方法有两个:`  countDown()`,`await()`.比如示例代码：

```
CountDownLatch countDownLatch = new CountDownLatch(2);
new Thread(()->{
    System.out.println("嘿嘿");
    countDownLatch.countDown();
}).start();
new Thread(()->{
    System.out.println("啦啦");
    countDownLatch.countDown();
} ).start();

countDownLatch.await();
System.out.println("哈哈哈");
```

其中创建CountDownLatch的时候，传入了参数2表示，表示初始计数器是2，构造器中的计数值（count）实际上就是闭锁需要等待的线程数量，如果想要执行await的时候不阻塞，计数器必须减到0才可以，而调用countDown就是将计数器的值减1。这样代码中，就能保证"哈哈哈"一定在"嘿嘿"和"啦啦"输出后才输出。
# java信号量(Semaphore)
操作系统的信号量主要用于进程控制，可以控制某个资源被同时访问的个数等。java并发库的Semaphore可以轻松完成信号量的控制。
构造方法确定同时访问进程的个数，在代码中通过`acquire`获取许可，`release`释放许可，示例代码如下：
```
Semaphore semaphore = new Semaphore(5);
for ( int i = 0; i < 20; i++) {
    final int t = i;
    new Thread(()->{
        try {
            semaphore.acquire();   //获取许可
            System.out.println(""+t+"在执行");
            Thread.sleep(1000);
            semaphore.release();   //释放许可
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();
}
```
运行结果：
```
1在执行
6在执行
2在执行
10在执行
14在执行
18在执行
9在执行
3在执行
17在执行
13在执行
7在执行
15在执行
19在执行
11在执行
5在执行
0在执行
4在执行
8在执行
12在执行
16在执行
```
如果运行观察的话，运行结果是5个一组的执行。注意结果不是从0打印到4的原因是，在执行acquire是需要耗费时间的，执行到这个地方的时候，可能就切换回主线程循环了，这样就导致，各个线程执行acquire的时间消耗不一样。

# 读写锁的实现
## 读写锁状态的设计
在一个整形变量上维护多种状态，需要“按位切割使用”，读写锁将变量切分成了两个部分，高16位表示读，低16位表示写。
通过位运算可以算出当前状态是读还是写，如果是读，则写状态值为0，即S&(0x0000FFFF)为0，这个时候，读状态(S>>>16)值大于0，即读锁被获取。

## 写锁的获取与释放
写锁支持重入，所以在获取写锁时，如果当前线程获取了写锁，则增加写状态，如果当前线程获取了读锁，则进行等待
## 读锁的获取与释放
读锁支持重入，并能够被多个线程同时获取，在没有写线程访问的时候(S&(0x0000FFFF) == 0) ，读锁总会被获取成功。如果读锁获取成功，则增加读状态。

## 锁降级
指写锁直接降级成为读锁，目的为的是数据的可见性，如果没有锁降级，那么写锁释放的时候，其他写锁修改掉了数据，那么在去读数据的时候，发现数据已经改变了，这样保证不了数据的可见性。所以锁降级，就是直接将本来的写锁降级成为读锁并能直接使用。操作过程为 持有写锁的同时再获取读锁，随后释放写锁。

# 输出文件夹下的所有文件
通过File的的isDirectory判断是否是文件夹，如果是就调用.list()方法，取得所有文件的名字，然后通过file.getAbsolutePath() +  File.separator + name可以获得该文件的绝对路径。

# List和Set的区别
List和Set都是接口，继承与Collection.
List可重复，Set不可重复
# HashSet如何实现
通过HashMap实现，在内部定义了一个HashMap对象，其中存储的元素都在map的key上，其中value为Object类型，并且该值为一个固定值`private static final Object PRESENT = new Object()`

# for遍历集合能进行删除操作吗、
如果是用Iterator进行遍历是可以删除的。
但是如果使用下标的话，对于List而言在代码中写删除虽然可以，但是如果是从前往后遍历，运行的时候会出现异常，而从后往前遍历则不会。
而Set删除一般是指定Object，不会指定下标
# Map EntrySet和KeySet哪个效率更高
entrySet效率高，因为使用KeySet只是返回一个Key的set集合，如果要获取所有的value，就需要重新遍历一遍Key的Set集合，然后对每个元素进行查找取得value值
。但是entrySet中存储的是Map.Entry，这个类存储了节点的key和value,也就是说一次entrySet可以将所有的key-value取出来。
# 集合元素排序
数组排序，可以通过使用Arrays.sort()方法进行排序。
集合可以通过Collections.sort()方法进行排序，该方法可以传入一个Comparator。

# 进制转换
A、十进制转换其他
　　十进制转成二进制
　　Integer.toBinaryString(int i)
　　十进制转成八进制
　　Integer.toOctalString(int i)
　　十进制转成十六进制：
　　Integer.toHexString(int i)

B、其他转换十进制
　　二进制转十进制
　　Integer.valueOf("1010",2).toString()
　　八进制转成十进制
　　Integer.valueOf("125",8).toString()
　　十六进制转成十进制
　　Integer.valueOf("ABCDEF",16).toString()

# 各种排序
参考：[八大排序](http://blog.csdn.net/hguisu/article/details/7776068)
稳定性是指序列中如果存在相等的数，在排序后，如果相等的数前后顺序比边就是稳定的。
不稳定的排序算法有 快排，堆排，希尔，选择
## 插入排序
类比扑克牌，每次从后边的牌中抽取一张，将该牌插入到前边已经排好的序列中。
不论二分插入排序，还是普通的直接插入排序，每次插入后，都要移动后边的数组，所以复杂度总是O(n2)
```
private void insertSort(int a[], int length){
    for(int i=0; i<length; i++){
        //选择当前的数字，和前边的数字依次比较
        for(int j=i-1; j>=0; j--){
            if(a[j+1] < a[j]){
                int t = a[j+1];
                a[j+1] = a[j];
                a[j] = t;
            }else break;
        }
    }
}
```

## 希尔排序
希尔排序实际是插入排序，需要选择增量因子如d={n/2,n/4,n/8...1}对每个因子进行插入排序
```
private void insertSort(int a[], int length,int up){
    for(int i=0; i<length; i+=up){
        //选择当前的数字，和前边的数字依次比较
        for(int j=i-up; j>=0; j-=up){
            if(a[j+up] <= a[j]){
                int t = a[j+up];
                a[j+up] = a[j];
                a[j] = t;
            }else break;
        }
    }
}

private void shellSort(int a[],int length){
    int up = length/2;
    while(up>=1){
        insertSort(a,length,up);
        up /= 2;
    }
    System.out.println(Arrays.stream(a).mapToObj(String::valueOf).collect(Collectors.joining(",")));
}
```
## 选择排序
在数组中选择最小的数和第一个数交换，依次类推
```
private void selectSort(int a[],int start, int length){
    //选择start开始的数到最后的数的最小值和start位置的数交换
    if(start == length)return;
    int min = Integer.MAX_VALUE;
    int index = 0;
    for(int i=start; i<length; i++){
        if(a[i] < min){
            index = i;
            min = a[i];
        }
    }
    min = a[start];
    a[start] = a[index];
    a[index] = min;
    //递归
    selectSort(a,start+1,length);
}
```
## 堆排序
借助小顶堆或大顶堆的排序，O(nlogn)
```
private void heapSort(int a[]){
    PriorityQueue<Integer> priorityQueue = new PriorityQueue<>();
    for(int t:a){
        priorityQueue.add(t);
    }

    for(int i=0; i<a.length; i++){
        a[i] = priorityQueue.poll();
    }
}
```
## 块排
```
private void quickSort(int a[],int low,int high){
    if(low>=high)return;
    int start = low;
    int end = high;
    int mid = (low + high)/2;
    int tar = a[start];
    while(start < end){
        //找后边比基准小的数，并交换
        while(start<end && a[end]>tar)end--;
        a[start] = a[end];
        while(start < end && a[start]<tar)start++;
        a[end] = a[start];
    }
    a[start] = tar;
    quickSort(a,low,mid);
    quickSort(a,mid+1,high);
}
```

## 堆排
```
    private void merge(int a[],int low,int mid, int high){
        //将来a[low,mid] 和 a[mid+1,high]合并
        int length = (int) (high - low + 1);
        int[] b = new int[a.length];
        int ls = low;
        int rs = mid+1;
        int index = 0;
        while(ls<=mid && rs<=high){
            if(a[ls] <= a[rs]) b[index++] = a[ls++];
            else b[index++] = a[rs++];
        }
        while(ls <= mid)b[index++]=a[ls++];
        while(rs <= high)b[index++] = a[rs++];
        for(int i=0; i<length; i++,low++)a[low] = b[i];
    }
    private void mergeSort(int a[],int low,int high){
        if(low >= high) return;

        int mid = (low + high) >> 1; //注意这里一定要用 >> 1表示除以2，效率高 
        mergeSort(a,low,mid);
        mergeSort(a,mid+1,high);
        merge(a,low,mid,high);
    }
```
# Long、AtomicLong、LongAdder
LongAdder是jdk1.8新增的原子操作的类，且在高并发的时候比AtomicLong更加高效，AtomicLong的原子操作是采用Unsafe中的一系列CAS方法，这些方法都是调用原生方法，以及机器的原子指令，所以能够保证原子性，但是LongAdder是保存了Cell，Cell中有一些列的段，这些段的和就是LongAdder将增加的值，所以可以类比分段锁，高并发的时候，增加的值可能会分配到不同的段中，所以效率上有提升。
* 首先和AtomicLong一样，都会先采用cas方式更新值
* 在初次cas方式失败的情况下(通常证明多个线程同时想更新这个值)，尝试将这个值分隔成多个cell（sum的时候求和就好），让这些竞争的线程只管更新自己所属的cell（因为在rehash之前，每个线程中存储的hashcode不会变，所以每次都应该会找到同一个cell），这样就将竞争压力分散了
# simpledateformat安全问题
SimpleDateFormat是线程不安全的，原因在于其内部实现是有状态的，一个simpledateformat实例中，会通过Calendar的setTime/setTimeZone来设置一些状态，假设线程一调用了setTime/setTimeZone设置了一个状态，此时线程二开始使用该状态进行format操作，这样肯定会出现问题。
解决该问题的方法有：
* 每个线程创建一个实例
* 使用ThreadLocal
* 使用的时候添加synchronized关键字
* 使用DateTimeFormatter
java8中的java.time对象都是不可变的，即无状态，是线程安全的。

# clone、深拷贝和浅拷贝，高效的clone方法
浅拷贝是拷贝引用，深拷贝会拷贝对象。

# 设计模式

## 设计模式六大原则
* 开闭原则：对扩展开放，对修改关闭
* 里氏替换：有基类的地方，替换为子类的时候，照常运行
* 依赖倒置：高层模块不依赖低层模块，抽象不依赖细节，核心思想是面向接口编程
* 接口隔离：客户端不应该依赖它不需要的接口；一个类对另一个类的依赖应该建立在最小(方法最少)的接口上。
* 迪米特法则：一个对象应该对其他对象保持最少的了解，降低耦合
* 合成复用：尽量使用合成/聚合模式方式，而不是继承

## 常用设计模式

### 普通工厂
需要创建工厂实例:
```
public interface CarFactory{
  Car createCar();
}

public interface Car{

}
```
### 静态工厂
不用创建实例:
```
public class CarFactory{
  public static Car createCar(){
    //...
  }
}
```
### 抽象工厂类
面向接口编写工厂，其实就是上面普通工厂所举的例子，只是没写完整。
### 单例模式
单例模式在面试过程中，经常出现，但是在Java中的单例模式的方式又有多少种呢？

#### 单线程懒汉式
顾名思义，就是这个单例类很懒，在用到的时候才会去创建，所以写法如下：
```
public class Singleton{
  private Singleton();
  private static Singleton INSTANCE = null;
  public static Singleton getInstance(){
    if(INSTANCE == null)
      INSTANCE = new Singleton();
    return INSTANCE;
  }
}
```
但是多线程的时候，这种单例就不可用了，因为可能会在某个时刻，有两个线程都执行到了if(INSTANCE == null)的内部去，这样就导致两个线程都创建了一个单例，那么这个时候就可以使用下列这种方式

#### 多线程懒汉式(double check)
在多线程创建单例的时候，会涉及到锁的概念，但是加锁会消耗资源，如果每次获取单例都加速，性能当然会有影响，所以使用if判断需不需要加锁，写法如下：
```
public class Singleton{
  private Singleton();
  private static Singleton INSTANCE = null;
  public static Singleton getInstance(){
    if(INSTANCE == null){
      sychronized(this){
        if(INSTANCE == null)
          INSTANCE = new Singleton();
      }
    }

    return INSTANCE;
  }
}
```
可以看到其中使用了双层判断。虽然这种方式加了锁，但是在某些情况下还是会出现问题，对于`INSTANCE = new Singleton()`这行代码而言，其操作过程并不是原子性的，一般创建对象的操作步骤如下：
1. 创建对象内存
2. 初始化对象
3. 将对象引用指向内存
但如果出现了指令重排序，可能导致对象创建过程为下：
1. 创建对象内存
2. 将对象引用指向内存
3. 初始化对象
那么如果出现这种情况，在操作第二部的时候就可以将该对象返回，这个时候使用该对象的时候，因为没有初始化，那么肯定会出现错误。所以可以在某些库中的单例模式是这样写的：
```
public class Singleton{
  private Singleton();
  private static volitale Singleton INSTANCE = null;
  public static Singleton getInstance(){
    if(INSTANCE == null){
      sychronized(this){
        if(INSTANCE == null)
          INSTANCE = new Singleton();
      }
    }

    return INSTANCE;
  }
}
```
即给变量加一个volitale，这个修饰符的含义是，禁止指令重排序。

#### 饿汉式
顾名思义，即时单例类很饿，就需要抓紧时间吃东西，所以一开始的时候就去创建，写法如下：
```
public class Singleton{
    private Singleton(){}
    private static Singleton INSTANCE = new Singleton()；  //建立对象
    public static Singleton getInstance(){
        return INSTANCE ；//直接返回单例对象    
    }
}
```
这种写法，在多线程中，是可以运行的，因为static静态变量，只会运行一次，但是如果一直用不到这个单例，那么就白白的浪费空间。

#### 通过枚举创建单例
枚举其实就是静态类，这个毫无疑问，并且枚举创建的单例类，不涉及到序列化反序列化的问题，写法：
```
public Enum Singleton{
  INSTANCE
  public Type someMethod(){
    //...
  }
}
```

#### 通过内部类创建单例
如果使用饿汉式创建单例，其必定会在不使用的时候耗费资源，如何让其在使用的时候创建，方法就是通过嵌套的内部类来实现：
```
public class Singleton{
  private Singleton();
  private static Singleton INSTANCE = null;
  public static Singleton getInstance(){
    return Holder.INSTANCE;
  }

  class Holder{
    public static final Singleton INSTANCE = new Singleton();
  }
}
```
### Builder模式
在构造函数中的参数相对比较多的时候，可以使用Builder模式，
### 适配器模式
将类的接口转换成客户端期望的另一个接口，消除接口不匹配造成的类的兼容性问题，主要分为三类：类的适配器模式，对象的适配器模式，接口的适配器模式。

#### 类的适配器
```
public class Source{
  public void method(){

  }
}

public interface Target{
  void method();
}

//类适配
public Adapter extends Source implements Target{

}
```
#### 对象适配
```
public Wrapper implements Target{
  private Source source;

  public Wrapper(Source source){
    this.source = source;
  }

  public void method(){
    source.method();
  }
}
```

#### 接口适配器
最好理解的适配器，如果想要接口中某些方法不去实现，那么就定义一个接口适配器，将所有方法实现为空方法，则其他类继承的时候可以选择性的实现某些方法。

### 代理模式
持有某个对象的引用，使用代理对象到时候，可以选择性的调用自定义方法后，再调用对象的目标方法。
```
public class Proxy implements SomeInteface{
  private Source source;
  public Proxy(Source source){
    this.source = source;
  }

  public void method(){
    before();
    source.method();
    after();
  }
}
```
### 组合模式
将对象组合成树形结构，表示"部分-整体"的层次结构，组合模式使得用户对单个对象和组合对象的使用具有唯一性。
参考[组合模式](http://www.cnblogs.com/jingmoxukong/p/4221087.html)

### 其他
责任链，观察者，生产者/消费者等。

# 一个数中二进制数中一的个数
```
private void solve(int input){
    int cnt = 0;
    while(input != 0){
        if((input&1) == 1)cnt++;
        input >>= 1;
    }
    System.out.println(cnt);
}
```
该方法出现的问题是如果有负数，则无限循环，原因是负数最前边的数字是1。
改进的方法：
```
private void solve(int input){
    int t = 1;   //以这个为基准，依次向左进行移位，移位到超过范围的时候，值为0.
    int cnt = 0;
    while(t != 0){
        if((t & input) != 0)cnt++;
        t<<=1;
    }
    System.out.println(cnt);
}
```
高效方法：
```
private void solve2(int input){
    int cnt = 0;
    while(input!=0){
        cnt++;
        input = input&(input-1);
    }
    System.out.println(cnt);
}
```
使用位操作高效，并且每次都能命中一个1，所以这个方法是最高效的。
# ThreadLocal
# spring类加载方式
一种是扫描加载，通过配置`<context:component-scan base-package="com.qxg" />`会自动扫描com.qxg包下的有注解的类，一般注解有:`@Repository`、`@Service`、`@Controller`、`@Component`.
一种是xml配置文件加载，通过配置bean进行加载
# 实例保存在哪
IOC容器中
# AOP IOC
AOP实现方式是通过动态代理实现的，一种是Java自带的动态代理Proxy。使用其中的Proxy.newInstance()即可创建一个代理对象，这种实现方式是给类添加接口。
还有一种是CGLib实现的，这种实现方式是通过继承类，给子类添加方法实现的。
Sprign中的AOP是使用的Proxy实现的。

AOP是面向切面编程，一般使用来说会使用aspectj框架，两种实现方式，一种是注解，一种是xml配置。
xml举例：
```
<aop:config>
     <aop:aspect id="time" ref="timeHandler">
         <aop:pointcut id="addAllMethod" expression="execution(* com.xrq.aop.HelloWorld.*(..))" />
         <aop:before method="printTime" pointcut-ref="addAllMethod" />
     </aop:aspect>
 </aop:config>
```

注解实现举例:
```
public class MyInterceptor {  
    @Pointcut("execution(* com.bird.service.impl.PersonServiceBean.*(..))")  
    private void anyMethod(){}//定义一个切入点  

    @Before("anyMethod() && args(name)")  
    public void doAccessCheck(String name){  
        System.out.println(name);  
        System.out.println("前置通知");  
    }  

    @AfterReturning("anyMethod()")  
    public void doAfter(){  
        System.out.println("后置通知");  
    }  

    @After("anyMethod()")  
    public void after(){  
        System.out.println("最终通知");  
    }  

    @AfterThrowing("anyMethod()")  
    public void doAfterThrow(){  
        System.out.println("例外通知");  
    }  

    @Around("anyMethod()")  
    public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable{  
        System.out.println("进入环绕通知");  
        Object object = pjp.proceed();//执行该方法  
        System.out.println("退出方法");  
        return object;  
    }  
}  
```
IOC即控制反转，又名DI 依赖注入，所谓控制反转，即本来对象是在自己的类中创建的，现在将对象的控制权交给spring进行管理，这就是控制反转。

# 反射机制
自行查看博客

# 类加载器
jvm将类加载过程分三个步骤：加载，链接，初始化，其中链接又分为 验证，准备，解析。
装载：查找并加载类的二进制数据
验证：确保加载的类的正确性
准备：为类的静态变量分配内存，并设置默认值
解析：把类中的符号引用转换为直接引用。
初始化：为类的的静态变量赋予正确初始值。
参考: [Java类加载器总结](http://blog.csdn.net/gjanyanlig/article/details/6818655/)

java类加载器通过ClassLoader及其子类完成的:Bootstrap ClassLoader,Extension Classloader, App ClassLoader, Custom ClassLoader
Bootstrap ClassLoader 负责加载JAVA_HOME%中jre/lib/rt.jar里所有的class,由C++实现，不是ClassLoader子类
Extension ClassLoader:加载%JAVA_HOME%中jre/lib/ext下的jar包
App ClassLoader:加载classpath中的class文件

# 双亲委派模型
一个类加载器收到加载类的请求的时候，不会自己先加载，而是将该请求委派给父类加载器去完成，如果父类无法加载，则子类尝试去加载，如果都没有加载成功，则抛出ClassNotFound异常
双亲委派 模式的类加载机制的优点是java类它的类加载器一起具备了一种带优先级的层次关系，越是基础的类，越是被上层的类加载器进行加载，保证了java程序的稳定运行。

# 内存结构
程序计数器，虚拟机栈，堆，本地方法栈，方法区
参考[java 内存结构](http://blog.csdn.net/bluetjs/article/details/52874852)

# tcp/ip,OSI七层模型
OSI七层模型:物理层，数据链路层(arp),网络层(ip)，传输层(tcp,udp协议)，会话层，表示层，应用层(http,stmp,ftp)

TCP/IP四层协议：链路层，网络层，传输层，应用层

TCP三次握手四次挥手
SYN_SEND SYN_RCVD ESTABLISHED 
FIN_WAIT1 CLOSED_WAIT FIN_WAIT2 LAST_ACK TIME_WAIT

参考:[TCP协议，我想你应该懂了](http://www.qxgzone.com/2017/01/07/%E8%BD%AC-%E5%85%B3%E4%BA%8ETCP%E5%8D%8F%E8%AE%AE%EF%BC%8C%E6%88%91%E6%83%B3%E4%BD%A0%E5%BA%94%E8%AF%A5%E6%87%82%E4%BA%86%EF%BC%81/)
# get和post区别
* get会将参数放到URL上，POST是放在Request的请求体中
* 长度方面，get会将参数放到URL中，因为一般浏览器都会限制URL的长度，比如IE限制为2048个字节等等，但是POST是没有长度限制的。
* 安全方面，get会把参数放到URL中，如果是英文数字则原文输出，如果是空格，则用`+`代替，而如果是中文，则通过BASE64进行编码后发送，形式为%XX%XX,POST是将数据放入REQUEST请求中，相对于GET来说是安全的，如果要追求更加安全，则可以使用HTTPS，HTTPS会将POST的数据进行加密，使用的是非对称加密方式。
REST 中 GET，POST，PUT,DELETE分别是查,改，增，删4个操作。

# tcp ip的arp协议
ARP(Address Resolution Protocal)是地址解析协议，获取IP地址的MAC地址,该协议在OSI模型中的数据链路层，在TCP/IP协议中的网络层。在cmd中使用方式为
```
arp -a 192.168.1.1
```
RARP反向地址转换协议，即知道对方的MAC地址，不知道对方IP

# 负载均衡
将请求[均匀]分布到多个系统上执行

# Mysql两种引擎及区别
Innodb,MyISAM
1. M非事务安全，I事务安全
2. M是表锁，I是行锁
3. M支持全文类型索引，I不支持全文索引

# 容器和虚拟机的区别

# fail-fast和fail-safe
fail-fast是集合操作的一种错误机制，当你在迭代一个集合的时候，如果有另一个线程正在修改你正在访问的那个集合时，就会抛出一个ConcurrentModification异常。在java.util包下的都是快速失败。
fail-safe:你在迭代的时候会去底层集合做一个拷贝，所以你在修改上层集合的时候是不会受影响的，不会抛出ConcurrentModification异常。在java.util.concurrent包下的全是安全失败的。

# 数据库索引原理
数据库中索引是使用B+树实现的
# 聚簇索引和非聚簇索引的区别
聚集索引确定表中数据的物理顺序，由于聚集索引规定数据在表中的物理存储顺序，因此一个表只能包含一个聚集索引。但该索引可以包含多个列（组合索引），聚簇索引一般用于InnoDB中
我们可以这么理解聚簇索引：索引的叶节点就是数据节点。而非聚簇索引的叶节点仍然是索引节点，只不过有一个指针指向对应的数据块。
非聚簇索引相对聚簇索引多了一次磁盘IO所以查找性能上较差。

参考:[聚集索引和非聚集索引](http://www.cnblogs.com/aspnethot/articles/1504082.html)
# 索引分类和区别

# B+树和B树区别
首先什么是B树？
B树是一种多路搜索树，其需要满足以下性质：
1. 根节点至少有两个节点
2. 一个K阶B树的非根节点的关键字个数位(k/2-1,k-1)
3. 所有叶子节点位于同一层

B+树和B树的区别：
1.树的非叶子节点不存储数据，所以相对于B树，能存储更多关键字信息，所以B+树比B树更矮，磁盘IO次数也更少。B树的非叶子节点存储数据。
3. 树的叶子节点之间通过指针相连，所以在范围查询的时候效率更高。而如果是B树范围查询，需要在叶子节点和非叶子节点间来回移动。
3. K阶B+树的关键字个树是K个，而不是K-1个
4. B+树每个关键字都是指向叶子节点中的最大值

# tire树
tire树即字典树，包含主要的3个性质：
1. 根节点不包含字符，其他节点，每个节点包含一个字符
2. 从根节点到某一节点的路径连起来的字符，为该节点对应的字符
3. 每个节点对应的字符都不相同
4. 如果某节点对应的字符存在，则该节点标记为红色。
# 海量数据面试问题
[海量数据面试集锦](http://blog.csdn.net/ts173383201/article/details/7858598)
# 二叉搜索树
略
# 海量数据中查找一个单词
Tire树
分布式ForkJoin,或Map/Reduce
Hash等
# 抽象类和接口区别
1、抽象类和接口都不能直接实例化，如果要实例化，抽象类变量必须指向实现所有抽象方法的子类对象，接口变量必须指向实现所有接口方法的类对象。

2、抽象类要被子类继承，接口要被类实现。

3、接口只能做方法申明，抽象类中可以做方法申明，也可以做方法实现

4、接口里定义的变量只能是公共的静态的常量，抽象类中的变量是普通变量。

5、抽象类里的抽象方法必须全部被子类所实现，如果子类不能全部实现父类抽象方法，那么该子类只能是抽象类。同样，一个实现接口的时候，如不能全部实现接口方法，那么该类也只能为抽象类。

6、抽象方法只能申明，不能实现。abstract void abc();不能写成abstract void abc(){}。

7、抽象类里可以没有抽象方法

8、如果一个类里有抽象方法，那么这个类只能是抽象类

9、抽象方法要被实现，所以不能是静态的，也不能是私有的。

10、接口可继承接口，并可多继承接口，但类只能单根继承。
# HashMap实现原理
拉链表，就是链表的数组实现的。
首先HashMap里面实现一个静态内部类Entry，其重要的属性有 key , value, next，从属性key,value我们就能很明显的看出来Entry就是HashMap键值对实现的一个基础bean，我们上面说到HashMap的基础就是一个线性数组，这个数组就是Entry[]，Map里面的内容都保存在Entry[]里面。
```
static class Node<K,V> implements Map.Entry<K,V>

transient Node<K,V>[] table;   //存储节点的数组
```
在存储的时候，会计算key的hash值，其计算过程是通过自定义的hash方法计算的：
```
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```
由上可知，如果key为null则hash值为0，所以HashMap中是可以存储null值的。

# HashMap中jdk1.8和之前的有什么区别

## table初始化大小
在创建的时候，并没有创建Node数组，而是在put的时候，开始创建，这样设计类似延迟加载，如果创建一个HashMap,但不使用的话不会浪费空间：
```
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
```
put中的这行代码表示，如果table是空，也就是没创建的时候，会执行resize()方法，在resize方法中有这么一行代码:
```
     newCap = DEFAULT_INITIAL_CAPACITY;
```
这就是默认初始化容量：
```
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```
即16.

内部还有两个变量，分别是threshold和loadFactory,其中threshold = capacity * loadFactory，threshold表示当HashMap的size大于threshold时会执行resize操作。 那么扩容原理就是当table中的kv个数大于threshold时，会进行扩容。
## hashmap的扩容
首先说为什么要扩容？当hashmap中的元素越来越多的时候，碰撞的几率也就越来越高（因为数组的长度是固定的），所以为了提高查询的效率，就要对hashmap的数组进行扩容。当kv键值对的元素个数大于capacity*loadFactory时，就会进行扩容，容量为原来的两倍。
因为扩容会重新创建数组，重新赋值，所以性能方面会有影响，所以如果能初始化一个初始值，避免扩容，是最好不过了。假设想要存储1000个元素，一般来说1000<threshold即可，即1000<capacity*loadfactory,loadfactory默认是7.5，所以capacity可以设置为1340，但是一般hashMap初始化大小为刚好比capacity大的2的倍数，所以可以设置为2048。

## 解决hash冲突
* 开放地址法
* 再hash
* 链地址法
* 创建溢出区

HashMap用的第三种。
# java中实例，常量都放在哪里
实例存储在堆中，常量存储在方法区中的常量池中，1.8后存储在元空间。

# 基本数据类型和字节个数
从小到大分别为boolean(1位)、byte(1)、char(2)、short(2)、int(4)、long(8)、float(4)、double(8)
# sleep和wait的区别，notify的作用
* sleep是Thread的方法，wait是Object的方法
* sleep是占用cpu去睡觉，wait是让出cpu，即sleep不会释放资源，wait会释放资源。
* sleep是静态方法，在哪个线程中调用，哪个线程去睡觉
* sleep指定时间，时间不到，只能使用interrupt方法强制中断，wait可以不指定时间，等待调用对象的notify唤醒。
* sleep不会释放同步锁，wait会释放同步锁
# 手写生产者消费者模式
要点：生产者和消费者是个Thread，有wait就有notify，wait再循环内，锁的是buf。
```
    class Storage{
        private LinkedList<String> list;
        private int max;
        public Storage(int max){
            this.max = max;
            list = new LinkedList<>();
        }

        public void produce(String input){
            synchronized (list){
                while(list.size() == max){
                    System.out.println("当前队列满");
                    try {
                        list.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                list.add(input);
                System.out.println("生产了，当前队列数量为"+list.size());
                list.notifyAll();
            }
        }

        public void consume(){
            synchronized (list){
                while(list.isEmpty()){
                    System.out.println("当前队列为空");
                    try {
                        list.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                list.poll();
                System.out.println("消费了，当前队列数量为"+list.size());
                list.notifyAll();
            }
        }
    }
```
其他方式比如BlockingQueue.参考:[生产者消费者模式多种实现方法](http://blog.csdn.net/monkey_d_meng/article/details/6251879/)
# 子网掩码的作用
划分ip地址的网络位(网络地址)和主机位(主机地址)。

# 数据库连接池的作用，配置文件
资源重用，加快响应速度，统一资源管理。
一般数据库连接池框架有druid c3p0 dpcp，以druid举例来说，配置是在spring配置文件中完成一般来说要配置基础，driver,url,username,password等，后边的配置就是连接池的大小，超时时间等等各种配置。
# 为什么要数据库连接池，有什么好处
在不使用数据库连接池的时候，每次访问数据库都要创建连接，访问然后断开连接，这样会给数据库增大压力。而使用数据库连接池，可以类比线程池，他对连接进行统一管理，在高并发的场景下，会使用连接池现存的连接，不需要频繁创建，这样资源重用，会提高效率。所以总结就是下面一句话：
资源重用，加快响应速度，统一资源管理。
# java栈的作用
java栈由一个个栈帧组成，每次调用一个方法，实际上都是创建一个栈帧放入栈顶，栈帧中存储的有该方法中使用的局部变量表，操作数栈，常量池指针等内容。
# java堆存储什么
存放所有new的实例。
# 方法区存什么
常量池位于方法区，方法区存储静态变量和类信息
# tomcat配置，堆的初始化大小是多少
不会
# java三大特性
多态，继承，封装
# 项目中遇到的问题
略。。。能说什么问题，爬虫问题？还是啥。。遇到的最大的问题就是没遇到问题。。
# 手写二分查找代码
```
    private int binSearch(int[] arr, int target){
        //从arr查找target返回坐标
        if(arr == null || arr.length == 0)return -1;
        int start = 0;
        int end = arr.length-1;
        int mid = (start + end) >> 1;
        while(start <= end ){
            if(arr[mid] == target) return mid;
            if(arr[mid] < target)
                start = mid + 1;
            else
                end = mid-1;
            mid = (start + end) >> 1;
        }
        return -1;
    }
```
# 前后端数据交互
1. 通过json进行交互，
2. 通过springMVC框架中的jstl
3. 通过WebSocket
# 项目开发流程
1. 分析需求
2. 创建表
3. 创建项目，导入包，导入web.xml
4. 配置spring,springMVC,db.properties等基础信息
5. 使用MybatisGenerator导入mapper等
# http包格式
request:
请求行(method,url,协议版本)，请求头(各种参数，refer,user-agent,connection,host)，请求体(get或者post的参数等)
response:
返回行(协议版本，状态码),返回头(cookie),返回体.
# tcp包含ip么(看TCP报文，是包含的)

# mysql数据库连接池的驱动参数
driverClassName:com.mysql.jdbc.Driver
url:jdbc:mysql://localhost:3306/xxxdb
username:root
password:root
# 数据库连接池如何防止失效
1. 创建一个线程每隔一段时间就测试一下，缺点浪费资源
2. 每次线程池取连接的时候，判断该连接是否有效，如果无效，就创建一个新的连接
# 部署项目时tomcat 的参数
1. 将项目放在webapp中。
2. 配置conf/server.xml中文件内容，修改Context对应的节点，输入url和目录的映射关系。
# 热加载的原理
自定义ClassLoader实现，可以重写loadClass,再每次加载类的时候，检查是否是自己想要实现热加载的类，如果是，就自己加载，否则就调用父类的加载器进行加载.
```
    protected Class loadClass(String name, boolean resolve) 
            throws ClassNotFoundException { 
        Class cls = null; 
        cls = findLoadedClass(name); 
        if(!this.dynaclazns.contains(name) && cls == null) 
            cls = getSystemClassLoader().loadClass(name); 
        if (cls == null) 
            throw new ClassNotFoundException(name); 
        if (resolve) 
            resolveClass(cls); 
        return cls; 
    } 
```
参考：[Java 类的热替换 —— 概念、设计与实现](https://www.ibm.com/developerworks/cn/java/j-lo-hotswapcls/index.html)
# 类加载的原理
类加载步骤， 双亲委派，loadClass,findClass,defineClass.
首先三种类加载器：BootstrapClassloader,ExtClassloader,AppClassloader.
BoootstrapClassloader负责加载%JAVA_HOME%/lib下的rt.jar。
ExtClassloader负责加载%JAVA_HOME%/lib/ext下的的jar包
AppClassloader负责加载Classpath下的class文件。

双亲委派：就是某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父类加载器，依次递归，如果父类加载器可以完成类加载任务，就成功返回；只有父类加载器无法完成此加载任务时，才自己去加载。

再自定义类加载器的时候，如果不指定父类加载器，那么默认的是AppClassloader.如果强制将父类加载器设置为null,则默认父类加载器将变为BootstrapClassloader.

类加载的步骤分为：加载，链接，初始化。其中链接又分为验证，准备，解析。
加载：通过类全名获取二进制字节流。
验证：检验文件格式是否正确
准备：分配变量内存并赋默认值。
解析：符号引用转为直接引用
初始化：对类中的变量等的初始化，比如构造器中的初始化。

loadClass: 该方法实现了双亲委派模型，一般再自定义ClassLoader的时候，不会重写该方法，防止破坏双亲委派模型。该方法内部通过findClass()来查找Class文件。
findClass:通过类名查找Class文件，并通过defineClass()返回一个Class对象。
defineClass:将传入的字节数组转换为Class对象，该方法为native方法。
所以一般再自定义ClassLoader只需要重写findClass方法，并在内部调用defineClass方法来返回一个Class对象。

参考[ 深入理解Java类加载器](http://blog.csdn.net/zhoudaxia/article/details/35824249)
[深入探讨 Java 类加载器【推荐】](https://www.ibm.com/developerworks/cn/java/j-lo-classloader/index.html)
# mybatis中#和$的区别
1. #是将传入的值当做字符串的形式，eg:`select id,name,age from student where id =#{id}`,当前端把id值1，传入到后台的时候，就相当于 `select id,name,age from student where id ='1'`.
2. $是将传入的数据直接显示生成sql语句，eg:`select id,name,age from student where id =${id}`,当前端把id值1，传入到后台的时候，就相当于 `select id,name,age from student where id = 1`.
3. 使用#可以很大程度上防止sql注入。(语句的拼接)
4. 但是如果使用在order by 中就需要使用 $.
5. 在大多数情况下还是经常使用#，但在不同情况下必须使用$. 
# hashmap读取的时候是否会读取到另一个线程put的数据
会，但hashMap线程不安全，所以多线程的情况不建议使用hashmap.
# linux显示文件夹大小
-l 列表样式显示文件
-h 显示文件大小，并且显示以kb,mb等结尾的易读的样式，所以显示文件大小是通过ls -lh
-S 排序
-r 反转排序
-a 显示隐藏文件
-R 递归列出子目录
-t 时间排序

# linux查看端口状态,natstat 加参数
首先通过ps查找程序对应的进程id即pid,如找tomcat的进程
ps -aux | grep tomcat
然后通过pid查找到pid对应的端口
netstat -anp | grep pid
# linux查看进程的启动时间
ps -eo lstart 启动时间 
ps -eo etime 运行使劲啊

查看某进程对应的时间
ps -eo pid,lstart,etime |grep PID
# 进程间通信方式
管道：半双工通信，数据单向流动，只有亲缘关系间使用，一般是父子进程
流管道：可以双向传输
命名管道：有名，且允许无亲缘关系的进程间传递数据
信号量(Semophore)：一种计数器，控制进程共同某资源的手段。
消息队列：消息的链表，有足够权限的进程可以向队列中传递消息，被赋予读权限的进程可以从中读消息。
共享内存：多个进程访问同一个块内存，最快的IPC方式。往往与信号量等一同使用，达到同步的效果
套接字(Socket):可用于不同机器间的进程通信。

# java socket用法
客户端传出数据：
```
        Socket socket = new Socket("192.168.1.115", 5209);
        OutputStream outputStream = socket.getOutputStream();
        outputStream.write("something".getBytes());

```
服务端接收数据：
```
 		ServerSocket serverSocket = new ServerSocket(5509);
        Socket socket = serverSocket.accept();
        socket.getInputStream().read();
```
# 内存泄漏发生在哪
什么是内存泄漏？
这个问题很普遍，但是千万不要回答成OOM,因为OOM是内存溢出，而内存溢出就是由内存泄漏造成的。
内存泄漏是一个对象在该被清除的时候没有被清除。
什么时候会导致内存泄漏？
当长生命周期有短声明周期的引用时。
举个例子？
单例模式因为是静态的，所以生命周期是整个APP的生命周期，如果有一个外部对象的引用，且这个对象是个短生命周期的，这个时候就会导致内存泄漏。
内部类默认有一个外部类的引用，所以使用内部类的时候，也要注意内存泄漏，比如安卓中的handler，一般创建Handler为了方便都是使用匿名内部类，或内部类的形式，因为handler有一个外部类的引用，如果handler再发送消息message后，message迟迟没被消耗，因为message又有handler引用，这样导致handler不会被消除，但是如果此时Activity已经被销毁，但是因为handler有该activity的引用，导致其并没有被成功销毁，这就造成内存泄漏。
内存泄漏发生再哪？
堆内存中。
# 线程安全实现方式
1. synchronized
2. volitale(不绝对)
3. Lock
4. Condition
5. 不可变对象(举个例子，java.time下的对象都是不可变的，所以相对于Date更安全)
6. ThreadLocal
7. concurrent包下的类
# ThreadLocal
每个线程都有一个ThreadLocalMap,在ThreadLocal中，每次取数据，都会通过getMap方法来取得对应线程的ThreadLocalMap对象，查看TreadLocalMap，可以看到这个是自定义的静态内部类。得到ThreadLocalMap对象后，通过get()来取出线程对应的值，注意get中传入的是ThreadLocal对象，如果想要维护两个线程安全有关的变量，就可以创建两个ThreadLocal，因为再map取数据的时候传入的就是ThreadLocal,不同的ThreadLocal取出的值必然是不同的。
所以整体看下来，其实ThreadLocal存储线程对应的对象使用的是弱引用。

那么回答以下问题：1、每个线程的变量副本是存储在哪里的？
线程变量副本存储位置是ThreadLocalMap中，取数据的时候，通过Thread获取对应的ThreadLocalMap,在通过Map的get方法取出对应的参数，其中get传入的参数是ThreadLocal对象。再细一点，存储位置其实是ThreadLocalMap内部维护的Entry对象，该对象继承WeakRefrence，Entry内部维护的变量就是我们想要取得的变量。
# synchronized reentrantlock区别
synchronized是jvm自带。
synchronized非公平锁，reentrantlock可以公平也可以不公平
reentrantlock比synchronzied多了很多功能，比如创建多个条件，定时获取锁等。
在资源竞争激烈的时候，Reentrantlock要优于synchroinzed的。
# 静态内部类和内部类的区别
静态内部类不能访问外部非静态变量和方法。内部类可以访问。
内部类默认有外部类的引用。
静态内部类可以有静态方法，也可以有非静态方法。
内部类不能有静态方法或静态变量(原因是内部类不是静态的，相当于一个变量，导致类加载的时候，不会扫描这个类，但是内部类如果有静态变量，但是jvm并不会加载，这就导致了矛盾)
静态内部类可以直接new，内部类需要先new外部类，才能new内部类。
# HashMap再jdk1.8中有什么改变
在1.8之前，hashmap使用链地址法解决冲突，但是有一个问题是，如果所有的key都对应同一个下标，就会导致hashmap成为一个链表，这样效率不高，在1.8中，链表的长度大于8时，就会转换为红黑树。
[美团：Java 8系列之重新认识HashMap【荐】](https://tech.meituan.com/java-hashmap.html)
# 红黑树是什么
基于二叉搜索树实现的，5个性质:
1. 每个结点要么是红的要么是黑的。  
2. 根结点是黑的。  
3. 每个叶结点（叶结点即指树尾端NIL指针或NULL结点）都是黑的。  
4. 如果一个结点是红的，那么它的两个儿子都是黑的。  
5. 对于任意结点而言，其到叶结点树尾端NIL指针的每条路径都包含相同数目的黑结点

可以这么记忆：根黑叶黑根叶同 红下两黑仅黑红
根黑，就是根节点是黑的，叶黑，就是叶子节点是黑的，根叶同，是根节点到达叶子节点的路径上黑色节点的数目是相同的。
红下两黑是红节点的两个子节点一定是黑色的，仅黑红表示一个节点要么黑要么红。
[教你初步了解红黑树](http://blog.csdn.net/v_july_v/article/details/6105630)
# GC7种收集器
serial,serial Old,ParNew,Parallel，Parallel Old, CMS, G1
# 数据库的存储数据的数据结构
mysql索引用B+树，其他数据结构并不清楚= =
# 熟悉的nosql，redis?
redis.
# 操作系统文件管理
不会
# 操作系统的内存管理
不会
# 虚拟内存的作用
物理内存不够的时候，可以将内存不用的东西放到虚拟内存中。
# C/S请求数据，如何实现断点续传
再http头部加上相应的字段，来实现，比如我当前所接收到的总字节数为2000，那么客户端在请求的时候，可以向某个参数中加上2001这个值，表示要重新传送2001处字节数据，服务端在获取的时候判断是否有这个参数，如果有说明是续传，取到相应文件后，将文件中的第2000到最后的字节发送即可。
# topK问题
利用Hash分文件/外排序/堆/分治等各种方法实现，前边有个网址有总结。
# 快排优化
快排的优化就是针对基准的优化，再选择基准的时候，有三种方式
1. 固定基准，选择开头第一个数
2. 随机基准，每次随机
3. 开头，尾巴和中间三个数中选择中间值。

一般优化而言是使用第三种方式。
其他优化，比如再数组中的数目小于某个值的时候，会使用插入排序等，再java中的排序默认采用这种规则。
java中的排序针对对象类型和非对象采用不同的排序方法，非对象默认使用快排，对象则使用归并。
# 线程和进程的区别
线程是调度的基本单位，进程是资源分配的基本单位
线程又名轻进程
一个进程可以有多个线程，但至少有一个线程
同一个进程间的线程资源共享，但是进程间的资源不共享
线程切换快，进程切换需要消耗资源。

# http状态码
状态码分类：

|分类|分类描述|
|-|-|
|1xx|服务器收到请求，需要请求者继续执行操作|
|2xx|请求被成功接收并处理|
|3xx|重定向，需要进一步操作以完成请求|
|4xx|客户端错误，请求包含语法错误或无法完成请求(服务端已收到请求，但是拒绝服务)|
|5xx|服务器错误，服务器再请求处理的时候发生错误|

常见状态码
200 OK：客户端请求成功。
400 Bad Request：客户端请求有语法错误，不能被服务器所理解。
401 Unauthorized：请求未经授权，这个状态代码必须和WWW-Authenticate报头域一起使用。
403 Forbidden：服务器收到请求，但是拒绝提供服务。
404 Not Found：请求资源不存在，举个例子：输入了错误的URL。
500 Internal Server Error：服务器发生不可预期的错误。
503 Server Unavailable：服务器当前不能处理客户端的请求，一段时间后可能恢复正常，举个例子：HTTP/1.1 200 OK（CRLF）。
[HTTP常见的状态码](http://www.cnblogs.com/hjxcode/p/5663830.html)
# 手写单链表有没有环
```
    public boolean haveCicle(Node n){
        Node quick = n;
        Node slow = n;
        while(quick!=null){
            if(quick.next!=null) quick = quick.next.next;
            else quick = null;
            //slow绝对不会为null的
            slow = slow.next;
            if(quick!=null && quick == slow)return true;
        }
        return false;
    }
```
# 当内存放不下大规模的数据，需要按照什么方法去取（并发？）
可以将数据分隔文件，然后通过排序将数据排序后在分割文件，每次取其中数据的一部分。
# 数据库引擎类型与差别
MyISAM和InnoDB的区别，MyISAM不支持事务，表锁，关注的是性能，InnoDB支持事务，行锁，关注事务。
# 数据库索引类型
普通索引，唯一索引，组合索引，主键索引
普通索引是最基本的索引，没有任何限制，创建的方式`create index idx_user_name on user(name)`
唯一索引表示索引列的值必须唯一，但允许为空`create unique index idx_user_name on user(name)`
主键索引是特殊的唯一索引，不允许有空值
组合索引创建一个索引，该索引包含多列，比如有一个索引包含ABC三列，那么他就是组合索引，什么时候会用到组合索引，再符合最佳左前缀的时候会用到，比如A，AB,ABC都会用到ABC这个组合索引。
# 唯一索引和普通索引的差别
略
# HashMap可以存储null么，HashTable呢？两者的区别
可以存储，HashTable不能存储，hashMap是不安全的，HashTable是安全的。
# 装饰模式和代理模式的区别
为什么会有这个问题，原因就在于连者的类图长得比较相像。
区别就在于：装饰器模式应当为所装饰的对象提供增强功能，而代理模式对所代理对象的使用施加控制，并不提供对象本身的增强功能。
# 装饰模式和适配器模式的区别
装饰模式和适配器模式两者都有一个别名叫做包装模式，但是两者包装形式是不一样的。
装饰模式中，装饰者和被装饰者都实现了同一个接口。而适配器模式是将一个被适配者转换为另一种接口。
# Redis的数据类型有哪些
string,hash,list,set,zset(有序set，注意不是排序)
# JUC是什么，AtomicInteger是怎么实现的
JUC 是java.util.concurent.
AtomicInteger是通过内部的Unsafe对应的cas方法实现的，cas是系统中的原子命令，其中unsafe的这些方法最终都是通过native实现。
# 新生代的垃圾收集器有哪些，老年代的垃圾收集器有哪些
新生代有serial,ParNew,parellel,G1.
老年代有serial old,Parellel Old,CMS,G1
G1是将整个堆划分为一个个区域，然后对这些区域采用标记整理的方法进行垃圾收集。
# notify工作原理
唤醒队列中的第一个对象，具体实现再cpp代码中，没看过。
# linux管道= =原来是`cat a.txt | grep "abc"`
啦啦啦
# 路由器和交换机有什么区别
路由器工作在网络层，网络层主要就是负责路由
交换机工作在数据链路层
在数据链路层只能识别物理地址，因此当交换机的某个端口收到一个数据帧时，交换机会读取数据帧中相应的目标地址的MAC地址，然后在自己的MAC地址表中查找是否有目标MAC地址的端口信息，如果有，则把数据帧转发到相应的端口；如果没有，则向除源端口外的所有端口进行转发。
# 数据库的三范式
不会。。
# JDK的代理和cglib代理的区别
JDK动态代理只能对实现了接口的类生成代理。
CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法 。
因为是继承，所以该类或方法最好不要声明成final ，final可以阻止继承和多态。
# 数据库的四大特性ACID
原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability）
原子性：事务操作要么完成要么都不完成
一致性：数据保持一致，并能读到更新后的值
持久性：在事务完成以后，该事务对数据库所作的更改便持久的保存在数据库之中，并不会被回滚
隔离性：隔离状态执行事务，使它们好像是系统在给定时间内执行的唯一操作。如果有两个事务，运行在相同的时间内，执行相同的功能，事务的隔离性将确保每一事务在系统中认为只有该事务在使用系统。这种属性有时称为串行化，为了防止事务操作间的混淆，必须串行化或序列化请求，使得在同一时间仅有一个请求用于同一数据。

# 排序算法哪个是最快的，哪个不需要临时空间
没有最快的，针对不同情况有不同答案。
再数组有序的情况下，冒泡和插入排序都可以做到O(n)。
所有简单排序和堆排序的临时空间都是O(1).快排为O(logn),为栈所需临时空间，归并排序需要O(n).
# 什么是二叉平衡树?什么是二叉搜索树
AVL是平衡二叉树，是二叉搜索树的升级版本，符合左右子树的深度差<=1.
二叉搜索树节点大于左子节点，小于右子节点，中序遍历结果就是排序的。
# concurrent包下的类
不知道
# springMVC流程，事务怎么配置
1. 用户向服务器发送请求，请求被Spring 前端控制Servelt DispatcherServlet捕获；
2. DispatcherServlet对请求URL进行解析，得到请求资源标识符（URI）。然后根据该URI，调用HandlerMapping获得该Handler配置的所有相关的对象（包括Handler对象以及Handler对象对应的拦截器），最后以HandlerExecutionChain对象的形式返回；
3. DispatcherServlet 根据获得的Handler，选择一个合适的HandlerAdapter。（附注：如果成功获得HandlerAdapter后，此时将开始执行拦截器的preHandler(...)方法）
4.  提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller)。 在填充Handler的入参过程中，根据你的配置，Spring将帮你做一些额外的工作：
HttpMessageConveter： 将请求消息（如Json、xml等数据）转换成一个对象，将对象转换为指定的响应信息
数据转换：对请求消息进行数据转换。如String转换成Integer、Double等
数据根式化：对请求消息进行数据格式化。 如将字符串转换成格式化数字或格式化日期等
数据验证： 验证数据的有效性（长度、格式等），验证结果存储到BindingResult或Error中
5.  Handler执行完成后，向DispatcherServlet 返回一个ModelAndView对象；
6.  根据返回的ModelAndView，选择一个适合的ViewResolver（必须是已经注册到Spring容器中的ViewResolver)返回给DispatcherServlet ；
7. ViewResolver 结合Model和View，来渲染视图
8. 将渲染结果返回给客户端。

参考:[Spring MVC 流程图](http://blog.csdn.net/zuoluoboy/article/details/19766131/)
事务配置有两种方式，一种是xml，一种是注解方式。
xml方式：
```
<!-- 配置事务管理器 这里拿HibernateT举例 -->
<bean id="transactionManager"
    class="org.springframework.orm.hibernate4.HibernateTransactionManager">
  <property name="sessionFactory" ref="sessionFactory" />
</bean>

<!-- 配置事务 -->
<!--  声明式容器事务管理 ,transaction-manager指定事务管理器为transactionManager -->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="add*" propagation="REQUIRED" />
        <tx:method name="get*" propagation="REQUIRED" />
        <tx:method name="*" read-only="true" />
    </tx:attributes>
</tx:advice>

<!-- aop配置 tx增强-->
<aop:config expose-proxy="true">
    <!-- 只对业务逻辑层实施事务 -->
    <aop:pointcut id="txPointcut" expression="execution(* com.lei.demo.service..*.*(..))" />
    <!-- Advisor定义，切入点和通知分别为txPointcut、txAdvice -->
    <aop:advisor pointcut-ref="txPointcut" advice-ref="txAdvice"/>
</aop:config>
```
注解需要配置扫描，并使用:`@Transactional`，再加这么一行配置文件：
```
<!-- 基于注释的事务，当注释中发现@Transactional时，使用id为“transactionManager”的事务管理器  -->
<!-- 如果没有设置transaction-manager的值，则spring以缺省默认的事务管理器来处理事务，默认事务管理器为第一个加载的事务管理器 -->
<tx:annotation-driven transaction-manager="transactionManager"/>
```

# 多列索引及最左前缀原则和其他使用场景
即最佳左前缀法则。 create index idx_user_name_age_id on user(name,age,id);
# bean的生命周期
通过xml中init-method和destroy-method来配置。
上述是简单的。
参考下面两个连接，背去吧，其中一个评论是：草。面试必问。还是硬记吧，
[一分钟掌握Spring中bean的生命周期！](http://developer.51cto.com/art/201104/255961.htm)
[Spring MVC事务配置](http://www.cnblogs.com/leiOOlei/p/3725911.html)
# 线程池工作原理
关键字:任务队列
[线程池的原理及实现](http://blog.csdn.net/hsuxu/article/details/8985931)
# NIO和BIO的区别
BIO 是普通的IO，指的是同步阻塞IO
NIO 是NEW IO/non-blocking io,是同步非阻塞IO
AIO 是异步非阻塞IO
# 常见命令ls,top,lsof,ps，grep等，vim快捷键
ls:显示文件列表
lsof:列出当前系统打开的文件
ps:显示进程
grep:通过正则进行搜索

vim常用命令参考:[vim常用命令](http://www.cnblogs.com/Nice-Boy/p/6124177.html)
# 什么是内存屏障，以及内存屏障的几种方式
内存屏障主要就是防止内存屏障前的指令重排序到内存屏障后，或者反过来。
目前有四种屏：
LoadLoad屏障：对于这样的语句Load1; LoadLoad; Load2，在Load2及后续读取操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕。
StoreStore屏障：对于这样的语句Store1; StoreStore; Store2，在Store2及后续写入操作执行前，保证Store1的写入操作对其它处理器可见。
LoadStore屏障：对于这样的语句Load1; LoadStore; Store2，在Store2及后续写入操作被刷出前，保证Load1要读取的数据被读取完毕。
StoreLoad屏障：对于这样的语句Store1; StoreLoad; Load2，在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见。它的开销是四种屏障中最大的。在大多数处理器的实现中，这个屏障是个万能屏障，兼具其它三种内存屏障的功能。

Java中对内存屏障的使用在一般的代码中不太容易见到.常见的有两种.
通过 Synchronized关键字包住的代码区域,当线程进入到该区域读取变量信息时,保证读到的是最新的值.这是因为在同步区内对变量的写入操作,在离开同步区时就将当前线程内的数据刷新到内存中,而对数据的读取也不能从缓存读取,只能从内存中读取,保证了数据的读有效性.这就是插入了StoreStore屏障
使用了volatile修饰变量,则对变量的写操作,会插入StoreLoad屏障.
其余的操作,则需要通过Unsafe这个类来执行.
# volatile的实现方式
Lock前缀指令会引起处理器缓存回写到内存
一个处理器的缓存回写到内存会导致其他处理器的缓存无效

# 什么是happens-before
一种用于重排序的规则：
1. 程序次序规则：在一个线程中，程序流运行顺序不会被重排序。
2. 管程锁定规则：对于同一个锁，unlock一定发生于下一次lock前。
3. volatile规则：对volatile的读一定先行发生于后面对该变量的写之前
4. 线程的一系列规则->start:线程的start语句一定happens-before该线程内部操作前。终止：线程的所有操作happens-before对该线程的终止检测。比如Thread.join一定发生于线程中所有操作之后。中断：啦啦啦
5. 对象终止规则：一个对象的初始化完成（构造函数执行结束）先行发生于它的finalize（）方法的开始。

常用前三条，牢记。
# 数据库的各种JOIN
sql一共有7种join:[戳一戳](http://www.qxgzone.com/2017/06/23/mysql%E7%B4%A2%E5%BC%95%E4%BC%98%E5%8C%96/)
# http状态码3xx 4xx 5xx分别是啥
3xx是重定向
4xx是客户端错误
5xx是服务端错误
# 如果我们一个项目，理论上需要1.5G的内存就足够，但是项目上线后发现隔了几个星期，占用内存到了2.5G，这时候你会考虑是什么问题？怎么解决？
1. 内存泄漏
2. 资源未关闭，也是内存泄漏的一种

内存泄漏解决方式是通过jvm调优工具，一个是jconsole，一个visualVm,这些都是界面工具，但是如果项目发布在只有命令行的服务器上怎么办？首先通过heap dump，将堆内存的dump文件导出。
平常可以使用jmap进行heap dump，但是因为内存溢出是瞬间发生，时间不能把握准确，这个命令就来不及。所以可以通过命令行参数来进行，-XX:+HeapDumpOnOutOfMemoryError，这个就表示发生内存溢出的时候进行heapdump，-XX:HeapDumpPath=/path/file.hprof 而这个命令指定dump的目录。将这个dump文件取出，使用visualVm或eclipse的MemoryAnalyzer进行分析。这些工具可以看到对象的大小和个数等信息。
# 如果想实现一个线程安全的队列，可以怎么实现？ ArrayBlockingQueue?
可以通过BlockingQueue进行实现，或者自定义实现。

参考:[各种阻塞队列](http://blog.csdn.net/suifeng3051/article/details/48807423)
# 说说http报文的header里面有什么
accept,cookie,host,referer,user-agent
set-cookie,cache-control
http://kb.cnblogs.com/page/92320/
# 一个大文件中数据排序，内存一次装不下，怎么实现？哈希+排序+归并+最小堆?
首先hash分文件，使用hash分文件，可以将相同数字分在一个文件中。
然后对每个小文件进行快排序。
再然后使用归并+最小堆的形式对所有文件排序，方式就是每次从两个文件中读数，放到堆中，如果堆满了，就装到一个最终文件中。
# 判断一个32位整数是不是4的幂？
首先2的幂的性质是整数中1的个数为1，那么4的幂的性质必然如此，所以通过n&(n-1)查看结果是否为0，如果不是，则为false，如果为真，就看这个1在什么位置，如果处于偶数位上，则不是，位于奇数位上是。
# 是否了解动态规划
了解一点。动态规划的思想和分治类似，重要是状态转移，每个状态的解都依赖前的状态。

# 求两个字符串的最长公共自序列
# 职责链模式
责任链模式：将能够处理同一类请求的对象连成一条链，使这些对象都有机会处理请求，所提交的请求沿着链传递。从而避免请求的发送者和接受者之间的耦合关系。链上的对象逐个判断是否有能力处理该请求，如果能则就处理，如果不能，则传给链上的下一个对象。直到有一个对象处理它为止。
[职责链模式（Chain of Responsibility）](http://www.cnblogs.com/cxxjohnson/p/6403849.html)
# 实践中如何优化MySQL
explain sql ,show profile,开启慢查询日志。
# 什么情况下设置了索引但无法使用(SQL语句的优化)

* 全值匹配
* 最佳左前缀
* 索引列上做任何操作(计算,函数,类型转换等)
* 范围之后全失效
* 尽量少用*,而使用覆盖索引(只访问索引的查询)
* 在使用!=或者<>时,无法使用索引会导致全表扫描.
* is null,is not null无法使用索引
* like以通配符开头(‘%’)索引会失效.比如like(‘%lisi’)索引失效
* 字符串不加单引号导致索引失效
* 少用or,用or的时候也会导致索引失效.
# HTTP和HTTPS的主要区别
https相比于http来说，再tcp方面会多握几次手，主要是传输证书。
https传输的数据是加密的。
https使用的端口是443，http是80
https加密工作再传输层。(尽量不要回答这个，否则问你传输层主要干什么你就蒙蔽去吧)
# Cookie和Session的区别
Cookie保存在客户端，Session保存在服务端
session保存在服务端的一个文件里。
session 的运行依赖 session id，而 session id 是存在 cookie 中的，也就是说，如果浏览器禁用了 cookie ，同时 session 也会失效（但是可以通过其它方式实现，比如在 url 中传递 session_id）
session 可以放在 文件、数据库、或内存中都可以。

# 如何设计一个高并发的系统
负载均衡，redis缓存，数据库分表分库(主库，读库)，集群等。
# linux中如何查看进程等命令
ps -aux。。。
pstree。。。
# 二叉树的遍历方式，前序、中序、后序和层序
略。。。
# volatile关键字
给大家推荐两本书：《Java多线程实战》和《Java并发编程的艺术》，这会儿已经三点了，脑子有点乱书名可能未必无误。对Java实现多线程描述的非常详细。现场跟面试官老师扯了很多，我在这里挑主要的说 volatile关键字是Java并发的最轻量级实现，本质上有两个功能，在生成的汇编语句中加入LOCK关键字和
内存屏障 
作用就是保证每一次线程load和write两个操作，都会直接从主内存中进行读取和覆盖，而非普通变量从线程内的工作空（默认各位已经熟悉Java多线程内存模型） 
但它有一个很致命的缺点，导致它的使用范围不多，就是他只保证在读取和写入这两个过程是线程安全的。如果我们对一个volatile修饰的变量进行多线程下的自增操作，还是会出现线程安全问题。根本原因在于volatile关键字无法对自增进行安全性修饰，因为自增分为三步，读取-》+1-》写入。中间多个线程同时执行+1操作，还是会出现线程安全性问题
# 为什么有多余的load和write操作？
简单说你可以这么看

* read是把变量从shared memory读入CPU local memory，或者说从内存读入CPU cache，write反之
* load是把变量从CPU local memory读入JVM stack，你可以认为它是把数据从CPU cache读入到“JVM寄存器”，store反之
之所以会这么麻烦，是因为现代电脑都有不止一个CPU，每个CPU都有自己的1级2级甚至3级缓存，CPU之间共享主存，一个CPU对主存所做的改动并不会自动被其它CPU发现，必须有某种机制让其它CPU知道这一点，当然最简单的思路是让cache和主存永远同步，但cache的速度远高于主存，强制同步其实相当于把cache强制降速，这对于程序执行效率是不利的。
现在的做法是选择性的同步，当你不需要同步时，只需要一次load，然后就可以多次read/write，避免和主存的同步，这样可以让这个CPU保持最高的效率运转；当你需要同步时，用store将变更写回主存，JVM/CPU/MMU会协调将这个变更通知到其它CPU以保证程序的正确性。
use是用来配合上述过程的，只有use了特定变量的CPU才会收到针对这个变量做store时发出的通知，这样就避免了无谓的CPU cache flush操作。

lock/unlock是传统方式，用来限制CPU对共享区域操作的，如果一个变量被lock了，那么其它所有CPU针对这个变量做出的lock操作都回阻塞直到拥有者释放这个锁。

以上是一个极为简化的memory model说明，里面还有很多细节需要长篇大论才说的清。
# synchronized

## 锁的优化,偏向锁，轻量锁，自旋锁，重量锁
偏向锁是jdk1.6引入的，其适用于无资源竞争的环境下，再有资源竞争的时候，会通过cas获取锁，如果没获取到就升级为轻量级锁。
轻量锁，适用各种资源交替执行的环境，

戳这:[详解锁锁锁锁锁。。。](http://blog.csdn.net/zqz_zqz/article/details/70233767)
## 乐观锁和悲观锁
乐观锁

乐观锁是一种乐观思想，即认为读多写少，遇到并发写的可能性低，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，采取在写时先读出当前版本号，然后加锁操作（比较跟上一次的版本号，如果一样则更新），如果失败则要重复读-比较-写的操作。

java中的乐观锁基本都是通过CAS操作实现的，CAS是一种更新的原子操作，比较当前值跟传入值是否一样，一样则更新，否则失败。

悲观锁

悲观锁是就是悲观思想，即认为写多，遇到并发写的可能性高，每次去拿数据的时候都认为别人会修改，所以每次在读写数据的时候都会上锁，这样别人想读写这个数据就会block直到拿到锁。java中的悲观锁就是Synchronized,AQS框架下的锁则是先尝试cas乐观锁去获取锁，获取不到，才会转换为悲观锁，如RetreenLock。
## 锁如何膨胀
偏向锁->轻量锁->重量锁
偏向锁在有竞争的时候，会通过cas获取锁，如果获取失败，就会膨胀为轻量锁，膨胀的方式是将MarkWord的最后两位改为轻量锁的标志，然后复制markword到栈中，并指针互指，如果通过cas设置指针成功，则就当前线程获取锁，如果cas失败，就会膨胀为重量级锁。

## 与Lock的区别和联系
# concurrentHashMap
线程安全的hashMap.内部维护了Segment，继承于ReentrantLock,即段,默认段的数量是16个。该类中使用的是分段锁.
再jdk1.8中放弃了分段锁的概念，而是使用cas进行线程安全的操作。
参考:[ConcurrentHashMap](http://www.importnew.com/22007.html)
# 锁的优化策略
① 读写分离
② 分段加锁
③ 减少锁持有的时间
④ 多个线程尽量以相同的顺序去获取资源
参考:[锁的优化和注意事项](https://my.oschina.net/hosee/blog/615865?fromerr=VVN0C2h0#OSC_h2_12)
# 三次握手为什么要三次而不是两次
防止失效数据重发建立连接消耗资源。
再传递过程，假设某段时间因为网络滞留导致数据停滞在网络传输过程中，在过了一段时间后，两者链接已经关闭后，这个数据突然就传递到B端了，此时B端认为A端要和B建立连接，所以就开启连接，然后继续等待A向B发送数据，但是此时A已经无数据可发，这样B只能无限等待下去，这样就导致资源的浪费。
采用三次握手就不会出现这种问题，因为B端在接受到数据后，会向A端返回一个确认码，并等待A回复ACK，三次握手A并不会回复ACK，导致B端不会一直等待下去。
# 在内存中怎么存储一张图
使用矩阵保存。
序列化。
# 谈一谈你做过的给你印象深刻的事情吧，我们从你讲的事情展开
自己写的登录注册功能。。。。。。。。。。。
自己创建的论坛。。。。。。。。。。。。。。
这个问题就不要想，碰到的几率太小。
# 多线程应用题
五个运动员（相当于五个线程）一个裁判（相当于主线程），满足一下3个条件，如何实现： 
  1.要同时起跑 
  2.要所有运动员都到达终点以后才能进行下一个环节 
  3.如果有一个运动员摔跤了（异常处理），就终止这次比赛，让所有运动员都到终点进行下一个环节。
实现同时的功能，可以通过volatile来控制，第二种就是通过闭锁控制。其他方式不知道= =
# Jvm中的常用的参数有哪些
-xms:初始化堆大小
-xmx:最大堆大小
-XX:+PrintGC 打印GC日志
-XX:HeapDumpOnOutOfMemeoryError jvm内存溢出自动生成堆转储
-XX:HeapDumpPath 生成的路径放哪
# 链表使用的哪种链表，循环链表还是双向链表
双向链表
# java线程池达到提交上限的具体情况
* newCachedThreadPool:可缓冲线程池，灵活回收线程，如果没有可回收的线程，则创建新的线程
* newFixedThreadPool:定长线程池，超出的线程会等待
* newScheduledThreadPool:定时或周期执行的线程池
* newSingleThreadExecutor:单线程化线程池，只有一个线程来工作，保证所有工作按序执行。
# java无锁原理
cas，CAS的CPU指令是cmpxchg
# rehash过程
当HashMap的元素数量达到了threshold的阀值就开始扩容，容量为原来的两倍。其中threshold为loadFactory*length,
# java如何定位内存泄漏
通过jstat来打印当前堆内存中各种GC的情况，如果发现FULL GC的频率高，大概就有内存泄漏了。
然后就可以通过heapdump 比如jmap或jvm参数获得dump文件，然后通过mat进行分析。
# 数组中Arrays.sort()的排序方式是什么
在数组个数少于某个临界值的时候(286)，使用插入排序，大于的时候，会使用快速排序或归并排序，对于基本类型使用快速排序，对于对象类型使用归并排序。
# 快速排序和堆排序的优缺点
堆排序平均时间比快排慢一点，但最差时间也是O(nlogn).
两者排序都不稳定。
快排最差时间复杂度为O(n2)
快排通常使用递归来进行的，可能导致栈溢出。
# 当一个tcp监听了80端口后，Udp还能否监听80端口
两者可以共用
# PING位于哪一层
ping相当于一个应用，所以属于应用层
# 二叉树中找出从根到叶子节点中和最大的那条路径。
深搜就OK
# 进程调度算法
* 先来先服务调度算法
* 短作业(进程)优先调度算法
* 高优先权优先调度算法
* 时间片轮转法
# 深度优先遍历，广度优先遍历算法，在什么地方可以应用。
宽搜找最短路径，深搜找最深路径。(= =什么地方能用到什么地方用，谁问这么弱智问题)
# 提升访问网页效率的方法(缓存:客户端缓存，cdn缓存，服务器缓存，多线程，负载均衡之类)
缓存
# list,map,set之间的区别
http://wiki.jikexueyuan.com/project/java-interview-bible/collection.html
# Servlet的生命周期
初始化：init()
响应客户端请求：service()
终止:destroy()
http://www.cnblogs.com/cuiliang/archive/2011/10/21/2220671.html
# 杨辉三角形的算法，第N行的数的计算
最简单的算法是暴力，复杂一点就是二项式转化成的排列组合算法
# 给定两个全都是大写的字符串a,b，a的长度大于b的长度，问如何判断b中的所有字符都在a中
（首先a,b排序，然后再两列比较）
# 线程池里面有Queue,你知道它的作用吗
这个队列是阻塞队列BlockingQueue,主要用于存放Worker，Worker是静态内部类，实现了Runnable接口。如果任务达到极限，就会出现阻塞等情况。
# 说一下Stack和ArrayList的区别
http://blog.csdn.net/a19881029/article/details/45533733
# 说一下HashMap和TreeMap的区别
http://blog.csdn.net/fg2006/article/details/6411200
# Servlet是线程安全的吗
不是，为什么这样设计？
如果是线程安全的，并发量就会降低。
# 如何在生产线Dump堆分析程序是否有内存及性能问题
jmap -dump转储文件，注意的是，stw。
或者通过jvm命令参数在内存泄漏的时候转储，前边有讲。
再Tomcat中修改jvm参数为：http://www.cnblogs.com/bluestorm/archive/2013/04/23/3037392.html
http://hellojava.info/?p=328&utm_source=tuicool
# 如何确保分布式环境下异步消息处理的顺序性
给每个任务分配一个全局的跟时间有关的序列号。
其他方式。。。。不会
# List,List<?>,List<? extends XX>的区别
考察泛型，泛型擦除，泛型通配符，PECS：http://ifeve.com/difference-between-super-t-and-extends-t-in-java/
# sql+
|-|-|-|
|姓名|分数|课程|
|name|score|course|
统计出每个学生有多少门课分数大于80分.
select count(*) from table group by name having score > 80
# 怎么检测死锁
通过编程的方式，使用ThreadMXBean来检测，其有个方法是findDeadlockedThreads。
通过jconsole检测死锁，jconsole可以输出很多信息，其中一个就是死锁，再线程的选项卡中有个选项是检测死锁，点击即可。
# 说一说项目中Spring的IOC和AOP具体怎么使用的
spring IOC就是配置bean，配置bean的内容，有property标签，对应set方法，以及constructor-arg等。
以及注解自动注入，比如<context-scan>扫描对应目录自动注入，扫描的内容有`@Service`,`@Repository`,`@Controller`,`@Component`等注解的类。

AOP也是有两种实现方式，一个是注解，一个是xml配置。
# 实习没学到什么东西，只能聊聊GIT了。。。
学会了git。。。。
再看看书吧
# Spring的annotation如何实现
反射
# Redis如何解决key冲突
不会
# 数据库事务的四个隔离级别，MySql在哪一个级别
http://blog.csdn.net/fg2006/article/details/6937413
| |脏读|不可重复读|幻读|
|-|-|-|-|
|Read Uncommited|Y|Y|Y|
|Read Commited|N|Y|Y|
|Repeatable read|N|N|Y|
|Serializable|N|N|N|

脏读：读到未提交的数据
不可重复读：同一个事务前后读取的数据不一致，注意是读取，事务1读->事务2修改->事务1在读
幻读：同一个事务前后操作后的结果不一致，注意操作是修改插入或删除。事务1操作->事务2操作->事务1查看.
幻读就是你操作了数据，数据应该已经改变的了，但是第二次再取查看，发现还有一些数据没有被改变。举个例子，事务1update了所有表，事务2此时insert一个数据，事务1再查看，便发现有一个记录没有被update，这就是幻读。
# JDK中哪些体现了命令模式
Thread，Runnable
[细数JDK状态](http://blog.jobbole.com/62314/)
# 线程池使用了什么设计模式
静态工厂
# 一致性Hash原理
不会
# super()和this能不能同时使用
不能，但是可以通过一些技巧，同时执行这俩，比如在this()对应的方法中，调用super()。
# java四种引用
强软弱虚
强不会被回收
软是在内存空间不够的时候被回收
弱是无论内存空间是否够，发生GC的时候都会被回收
虚不影响对象的生命周期，只是在回收的时候将其添加到对应的引用队列中
# Linux(查看指定进程)
ps -aux|grep pid
# String，是否可以继承，“+”怎样实现，与StringBuffer区别
不能，final修饰的类不能有子类。
StringBuffer是可变对象，String不可变，StringBuffer是线程安全，StringBuilder是线程不安全。
+号在编译后会生成StringBuilder,并使用StringBuilder的append方法进行操作，最后使用toString()进行转换。

# servlet流程
http://www.cnblogs.com/Wonghy/p/5542277.html
# 序列化
Serialize。。。
# tomcat均衡方式
https://www.ibm.com/developerworks/cn/opensource/os-lo-apache-tomcat/
别看了，看了也不会
# Linux命令常用的有哪些
ls,ps,top,grep,find,netstat。。
# TCP/IP 有几层，每层有何含义
链路层，网络层，传输层，应用层
链路层：负责接收物理帧，并抽取出ip数据报或封装ip数据包并发送出去。
网络层：主要定义IP格式，并负责路由等功能,IP协议
传输层：主要负责应用间通信，端到端的传输协议等，TCP/UDP
应用层：和应用程序打交道，并提供常用的网络应用服务。
# DNS的解析流程
[一张图看懂DNS域名解析全过程](http://www.maixj.net/ict/dns-chaxun-9208)
# mysql数据库的锁有多少种，怎么编写加锁的sql语句
三种，表锁，行锁，页锁。
比如行锁，在sql语句后边加上 for update等等各种方式。
回去看视频。
# mysql什么情况下会触发表锁
如果引擎使用的是MyISAM的时候，如果有多个事务进行写，或读写冲突就会触发表锁。
如果引擎使用的是InnoDB,再索引失效的时候，会触发表锁。
# 页锁、乐观锁、悲观锁
悲观锁是每次操作都认为有并发现象出现，所以每次都先获得锁。
乐观锁是先进行操作，不得已才会拿锁。
# redis的操作是不是原子操作
单个操作是原子性的。
多个操作可以通过事务实现原子性。
# ConcurrentHashMap中的seg是不是越过越好
不是，太多会导致性能降低，至于原因呢，不知道，个人认为太多会降低空间利用率
# TCP和UDP的区别
TCP 建立连接，UDP面向无连接
TCP 提供可靠服务，UDP尽量最大交付，不保证可靠传输
TCP面向字节流，UDP面向报文

# session在服务器上以怎样的形式存在
文件，内存，缓存，都可以。
# 怎么设置session和cookie的有效时间
cookie通过expires属性设置，session可以通过spring中的某个配置设置。
# springMVC和spring是什么关系
spring是一个框架，springMVC是一种类似structs的MVC框架，springMVC的运行依赖于spring。
# hash冲突的四种办法
* 开放地址法
* 再hash
* 链地址法
* 创建溢出区

# 给你一个表只有一列name~~有重复的name, 然后求出前十个name数最大的： 
  select name,count(name) from table group by name order by count(name) desc limit 10
# 内存溢出了怎么办
一般是内存泄漏导致，dump并分析。
# 反射机制中可以获取private成员的值吗
可以
getDeclaredMethod,getDeclareFiled可以获取私有方法，但是只能获取跟本类有关的方法，而getMethod可以获取父类中和该类中的所有方法，但不能获取私有方法。
# 红黑树与二叉树有什么区别、红黑树用途
1. 如果插入一个node引起了树的不平衡，AVL和RB-Tree都是最多只需要2次旋转操作，即两者都是O(1)；但是在删除node引起树的不平衡时，最坏情况下，AVL需要维护从被删node到root这条路径上所有node的平衡性，因此需要旋转的量级O(logN)，而RB-Tree最多只需3次旋转，只需要O(1)的复杂度。
2. 其次，AVL的结构相较RB-Tree来说更为平衡，在插入和删除node更容易引起Tree的unbalance，因此在大量数据需要插入或者删除时，AVL需要rebalance的频率会更高。因此，RB-Tree在需要大量插入和删除node的场景下，效率更高。自然，由于AVL高度平衡，因此AVL的search效率更高。
3. map的实现只是折衷了两者在search、insert以及delete下的效率。总体来说，RB-tree的统计性能是高于AVL的。
# Servlet的Filter用的什么设计模式
职责链模式
# 利用Bean的初始化可以做什么事情
不知道。。。
# 有一条SQL语句很慢，如何优化
explain +sql进行优化，查看是否是索引失效等问题。
show profile 查看具体问题在哪一层。
通过`show open tables`查看是否是锁问题。
# mysql varchar与text区别
char 定长0~255
varchar 非定长65535个字节
text不能有默认值，最大2^16 - 1.
# Redis持久化机制
aof:append of file
rdb:..
回去看视频
# 基于快照的方式在持久化的过程中有其他更新怎么办
丢失。。
# 500万数字排序，内存只能容纳5万个，如何排序，如何优化
hash分文件 + 小文件快排 + 归并和堆排
# 有人建议给每张表都建一个自增主键，这样做有什么优点跟缺点
缺点不连续，在分库分表可能会导致主键重复的情况，手动设置id比较麻烦。
优点方便，数据库自动编号，速度快，占用空间小
# mysql 范式，第三范式
。。。。
# 120道java面试常考题目（附答案）
https://www.nowcoder.com/discuss/19479?type=2&order=3&pos=27&page=1
# java后端总结资料
https://www.nowcoder.com/profile/518327/myDiscussPost
- - -
算法
# 手写算法，找出增序排序中一个数字第一次和最后一次出现的数组下标
# 手写算法，数据去重，海量数据去重
# 手写算法，给定区间，输出区间结果，如[1,3],[2,5],[5,8],[12,55],[3,6],[7,8]，输出[1,8],[12,15],
# 给n个数，按字典序排序后，求第m个数 
# K个超长有序数组，求中位数,注意是K个
# 给定一个整数数组，数组中元素无重复。和一个整数limit，求数组元素全排列，要求相邻两个数字和小于limit
- - -
美团
# 数组先升序再降序，找出最大数
二分，如果mid大于mid+1,则结果在left，如果mid大于mid-1，则结果在右
# jvm内存模型(这个是最最重要的)
前边有
# java线程和进程的差别
即线程和进程的区别。。。有种再讲个协程。
# 正整数数组，找出最大的一个正数
循环啊。。。
# java nio(美团重视技术，所以新技术要看，javanio,java8等)
//TODO
# 数据库隔离级别
read uncommited,read commited, repeatable read,序列化。
# mysql存储引擎及差别
myisam和innodb的差别
# 运行时异常及处理方法
运行时异常不需要捕获，系统直接崩还处理什么。。
# 抽象类和接口的区别
前边写的有
# HashMap的put方法源码

```
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

# ArrayList,LinkedList的实现以及插入，查找，删除的过程
ArrayList:
```
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

	//扩容
	    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
get方法就不贴了吧。。。
remove也不贴了吧。。。

linkedList唉，算了，也不贴了。，
# TCP和UDP区别
前边写的有
# 三次握手，TIME_WAIT状态
三次握手:
SYN_SENT,SYN_RCVD,ESTABLISHED
TIME_WAIT四次挥手的时候计时关闭的状态
[戳这](http://www.qxgzone.com/2017/01/07/%E8%BD%AC-%E5%85%B3%E4%BA%8ETCP%E5%8D%8F%E8%AE%AE%EF%BC%8C%E6%88%91%E6%83%B3%E4%BD%A0%E5%BA%94%E8%AF%A5%E6%87%82%E4%BA%86%EF%BC%81/)
# jvm内存区域和GC
前边有
# finalize方法
在垃圾被回收前会调用一次finalize方法，由java中的一个专门的Finalize线程进行调用。
一般不建议在finalize进行资源关闭等操作，建议的是进行资源检查并重新关闭。
jvm不能保证该方法能准时执行，所以一般都是废弃，不使用。
# 哪些对象可以作为GC Root
虚拟机栈（栈帧中的本地变量表）中引用的对象。
方法区中类静态属性引用的对象。
方法区中常量引用的对象。
本地方法栈中JNI（即一般说的Native方法）引用的对象。
# NIO的DirectByteBuffer和HeapByteBuffer，同步和异步，阻塞和非阻塞
深入理解，//TODO
# RestFul API的理解
//TODO
# Spring IOC,AOP，项目中怎么体现的
看前边的
# 框架代码中用到的设计模式
//TODO
# 写一个SQL语句，索引的最左前缀原则
= =
# 两种存储引擎在索引以及锁机制上的实现方式的区别
= =
# limit offset优化
//TODO
# jvm调优
//TODO
# 如果自定义一个String类会怎样(双亲委派机制)
没用，因为类加载的时候，首先会让父加载器加载，双亲委派机制，就不会让AppClassLoader加载，写了也用不了，还可能导致程序跑不了，不信用IDEA去试试。
# ReentrantLock源码，AQS，synchronized实现，乐观锁和悲观锁，volatile，ThreadLocal原理，线程池工作流程
我擦。。。炮轰么
重点看AQS
# HashMap和ConcurrentHashMap基本原理，扩容机制

# 装饰者模式和代理模式的区别
前边写的有
# 海量URL数据，低内存情况下找重复次数最高的那一个
hash分文件，每个文件中通过hash找出次数最高的。
# 线程状态切换
http://blog.csdn.net/sinat_36042530/article/details/52565296
# InterruptException什么时候抛
http://blog.csdn.net/derekjiang/article/details/4845757
# finalize会不会立即出发GC，finalize对象复活
对象复活，就是给对象多加个引用就OK了，反正GC无法回收就行。
# Linux中如何查看cpu的使用率
top
# 有两个单链表（不存在环），不借用任何其他数据结构，怎么遍历一次就判断是否相交
查看尾节点是否相同
# TCP是如何来保证可靠的传输的
超时重发，收到后确认，校验和，丢失顺序的数据包重新排序并重发，

http://blog.csdn.net/jhh_move_on/article/details/45770087
# 如何你和你的同事同时在开发项目，但是你们的代码冲突了，并且生成了日志信息，那么请问你怎么进行处理
如果有冲突是提交不了的，所以首先要做的是拉代码，将同时的代码拉到自己这来，但是因为冲突，又导致拉取失败，所以方法是使用`git stash`来暂时保存自己的文件，然后再拉代码:`git pull --rebase`，拉完代码后，通过`git stash pop`来将自己代码抽取出来，这个时候因为冲突，导致对应的文件部分信息变为冲突日志，导致编译不通过，所以要到对应的文件中修改冲突的代码，修改完成后，再进行`git push`提交即可
#  Linux中查看服务的命令
chkconfig -l
# 你这个命令会出现很多服务，那么怎么找到我要搜索的服务名称（管道）
chkconfig -l|grep xx
#  写一下tcp主动关闭的一方的几个状态，并且解释一下这些状态
FIN_WAIT1,FIN_WAIT2,TIME_WAIT
tcp主动关闭的时候，会发送FIN码，发送后的状态为FIN_WAIT1，等待另一方回复ACK，
在接受到ACK后，就变为FIN_WAIT2，等待另一方主动关闭。
另一方再数据发送完毕后，开始主动关闭后，该方收到FIN码后，然后返回ACK码，该方就变成TIME_WAIT状态，该状态是等待一段时间后，再关闭。
TIME_WAIT状态作用是，防止A向B端发送ACK失效，如果失效，B重新向A发送FIN，A再TIME_WAIT状态，就会接受到FIN，并重发ACK。
# http的常用的状态有哪些，301和302的区别是什么，503是什么意思
301是永久性转移，302表示暂时性转移
503服务不可用
# 求解一颗二叉树的深度，并分析
深搜
# 求解一个旋转数组中出现的最小的数字，要求效率高，并分析
二分
# java实现线程的方式；哪种好；为什么好
线程池方式好，对资源进行管理，并可重复使用，减少创建线程的开销等，能够避免过度创建线程导致的灾难。
# java中hashMap结构，处理冲突方法
链地址法
# 红黑树
啦啦啦
# 进程调度
啦啦啦
# 页面置换
//TODO
# 手写栈实现队列
啦啦啦
# 自己实现IOC
//TODO
# 分布式锁
//TODO
# 谈谈对分布式的理解
//TODO
# static修饰的变量，在类加载的时候，什么时候初始化
类加载的阶段有 加载 链接 初始化
链接又分为验证，准备，解析。
在初始化阶段开始初始化
# 一个无序数组，找第K大的元素
小顶堆实现
# 输入一个网址的过程
http://www.cnblogs.com/wenanry/archive/2010/02/25/1673368.html

# 客观
如果出现了不一致的意见，你们是怎么解决的
平时是如何学习的，通过哪些方式，学到了什么 
Ø 你本科硕士并不是计算机专业，为什么想从事互联网方向 
Ø 你看过哪些书，详细的说说 
Ø 在项目里面你是如何和你的同学进行分工协调，高效工作的 
Ø 如果出现了不一致的意见，你们是怎么解决的 
Ø 你对我们新美大的产品有过哪些接触，感觉如何 
Ø 我们新美大工作地点有北京和上海，你会选择哪一个城市，为什么 
Ø 你还有没有收到其他公司的offer，那你会在这些里面如何的选择 
Ø 有没有什么问题需要问我的

- - -
以下是客观问题

# 遇到过什么难题
在编程的时候，遇到第一个难题就是爬虫碰到验证码如何解决的问题，刚开始能力有限，略懂一些cookie的知识，当时还不怎么懂session，也不知道这个名词，所以如果没有验证码的时候，只是获取登陆成功后的cookie就OK了。但是后来爬一个项目，登陆的时候要让用户去输验证码，这个问题在当时确实难到我了。不过我比较不甘心，开始上网搜有关登陆的后台方面技术文章，当时对后台的理解还不太深入。经过两天左右的学习后，了解到了session和cookie的一般机制，知道了cookie其实是和后台的session对应的，http的无状态等各种特性，然后就开始想办法爬验证码这个东西，首先按照浏览器理解的思路是，先去访问登陆界面，然后再访问验证图片，取得验证码图片后让用户去填写数据，之后填写用户名密码登陆就好了。但是实际操作的时候，还是遇到了问题，问题就是验证码错误。当时百思不得解，感觉自己的思路是对的。所以我就开始在网页来回的登陆退出，看看cookie有什么变化。代码也是各种尝试，后来偶然的访问验证码的时候，发现也可以获取cookie，所以就猜测，是否应该是先访问验证码，然后再访问登陆页面呢,于是开始尝试这一个解决思路，最终成功登陆。当时把我高兴坏了，真心的为自己的 成长感到高兴，为自己的独立解决问题感到高兴。后来又想了一个问题，就是如果经常让用户去填写验证码，用户这一繁琐的操作也会影响用户体验。索性上网搜索自动识别验证码的问题，当时觉得使用算法识别应该比较难，所以想看看有没有成型的api可以调用，最后发现基本上验证码识别的api都是收费的，于是开始自己研究，找了一些ocr方面的算法去测试，没有成功，偶然看到一篇技术文章，讲解的是简单验证码识别思路，对比了下我想要识别的验证码，发现十分对应，于是研究了一下午，最后自己编码验证，发现识别率挺高的，有95%,其识别的思路是建立模版对象，然后收集目标对象，将目标对象和模板对象对比，算出其相似率，如果相似率高，就是该模板对应的数字没跑了。

安卓系统的密码问题。
# 遇到问题的解决办法
办法有很多，查阅技术博客，百度，谷歌，查阅官方文档等
# 抗压能力
！！！抗压能力最牛了
# 优缺点
踏实，爱学
稍微有点内向。
# 是否接受加班
是
# 遇到问题，同时不配合怎么办
谁敢不配合。。。
# 平时学习方法
看书，看博客，看微信公众号
# 职业规划
深入后台，学会分布式系统
# 你是怎么学习的，说完会让举个例子

# 实习投了哪几个公司？为什么，原因

# 最得意的项目是什么？为什么？(回答因为项目对实际作用大，并得到认可)
# 最得意的项目内容，讲了会
# 你简历上写的是最想去的部门不是我们部门，来我们部门的话对你有影响麽？
# 你觉得你的优势在哪里？在技术上你觉得比别人哪些地方更优秀？
踏实的做项目，做项目效率比较高
# 计算机网络
https://www.nowcoder.com/discuss/1937




# hashMap的扩容原理，初始有16个，要怎么new？(达到了负载因子，直接手动>>1)
初始化有16个，再达到threshold的时候，就会调用rehash来进行扩容，直接扩容为原来的2倍大小。
new的时候，传入一个数字，在内部会自动增长2^n次方倍，并刚好比该数字大。
为什么这样设计，就是因为在取模的时候运算方便。
# Integer的常量缓存池的问题(-127~128范围有个cache)
```
    private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```
# ConcurrentHashMap的size()怎么做的
再1.8之前，因为有段，做法是前后计算两次断中总和大小，如果相同，说明中间没有插入删除元素如果不同，就开始对段上锁并计算。
1.8之后，是计算cells的相加的大小，每个cell对应一个table的大小。

http://www.jianshu.com/p/c0642afe03e0
# Spring的AOP关于拦截private方法一些问题.(细节忘记了，当时答得也不好)
拦截不了。。。。
jdk代理是通过实现接口的方式实现代理，但是接口中是没有private方法的。
cglib是通过继承，但是子类是无法继承父类private，所以也无法实现。。
# 说说你了解的反爬虫措施，和针对异常的处理。
user-agent
ip限制
访问次数限制
以后爬虫可以考虑使用PhantomJs来试下，内置浏览器。。。
# 那你觉的你来做一个网站要从哪些方面考虑反爬虫。
数据是否是核心数据，如果不是，反爬虫可以相对宽容一点，如果是，就严格限制爬虫
# spring层面做事务和数据库层面做的区别，各自实现方式。
spring事务是操纵mysql事务，如果mysql引擎不支持事务，比如myisam，那么spring也无法完成事务操作。
# 聊了事务的传播性和隔离级别，问了mysql的默认隔离级别（可重复读）
PROPAGATION_REQUIRED 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。
PROPAGATION_SUPPORTS 支持当前事务，如果当前没有事务，就以非事务方式执行。
PROPAGATION_MANDATORY 使用当前的事务，如果当前没有事务，就抛出异常。
PROPAGATION_REQUIRES_NEW 新建事务，如果当前存在事务，把当前事务挂起。
PROPAGATION_NOT_SUPPORTED 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
PROPAGATION_NEVER 以非事务方式执行，如果当前存在事务，则抛出异常。
PROPAGATION_NESTED 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与 PROPAGATION_REQUIRED 类似的操作。

spring 默认的事务传播行为是 PROPAGATION_REQUIRED，它适合于绝大多数的情况。
https://www.ibm.com/developerworks/cn/java/j-lo-spring-ts1/index.html
# spring中事务传播性怎么配置（xml方式和注解方式，还有关于savepoint的使用）
http://blog.csdn.net/qh_java/article/details/51811533
# 算法：O(1)删除执行链表结点，做分析（其实是要指出剑指offer中那个直接copy值的方法的缺陷和隐患）
如果有其他地方引用的被删除的节点，会导致数据差错
# 详细聊了聊spring的IOC和AOP思想
前边有
# 关于AOP在spring的应用（比如事务，通知，aspectJ，slf4j的原理,和log4j的对比）
日志就是基于aop来搞定的，否则日志怎么给你打印日志。。= =

# 关于jdk代理和cglib第三方代理（说出对接口代理和子类继承的区别）
接口代理，需要有对应的方法，方法不能是private,protected等
继承方面，方法不能是final，private，static等。

static方法不能被重写，只能是子类多了一个静态方法而已。

http://blog.csdn.net/u013126379/article/details/52121096
# 最大的数据量多大，用了索引没有，怎么用的（聊了前缀索引，对于varchar类型的值，又聊了聊char，varchar，text，blob的关系和区别）。
前缀索引，对字符串的前几个字符创建索引。。。。具体用法,别深究了...
# 为什么索引不能随便用，什么时候用（什么时候失效，什么时候最高效）
因为索引占用空间,一般来说,经常查询的记录,不经常修改的记录可以使用索引.
高效的话，通过explain可以看出
# 数据库用了缓存没有，讲讲redis的理解（用作缓存，队列，也可做存储）
redis是一种noSql,可以用作数据库，缓存等。
# redis是单线程还是多线程的，举个例子（做计数器，rank排行榜）
单线程
为什么单线程也很快，因为它基于内存，多路IO复用模型
# 讲了讲项目的设计，包括异常的处理，数据库设计，通信模型的设计
异常处理。。。http://blog.csdn.net/ufo2910628/article/details/40399539
# 讲讲你理解的JVM吧（从内存划分说到了GC算法、分代思想，CMS和G1 collector，到类加载模型，tomcat的非双亲委派、线程上下文加载器，到JVM调优的策略，gc参数设置策略，如何找死锁，读快照，发现内存泄漏等等吧
靠。。让自己去讲JVM= =
# 给200个200个数的数组，找到最大的200个
建小顶堆。
# git 常用的操作，git rebase和git merge区别
git merge合并两个版本
git rebase是将另一个修改的版本打成补丁后，再在该补丁后生成一个新的节点。
http://blog.csdn.net/wh_19910525/article/details/7554489
# linux常用命令，查看内存
top
# tomcat类加载有什么不同，说加载顺序并不是双亲模型，具体顺序说一下
http://www.cnblogs.com/xing901022/p/4574961.html
# 悲观锁乐观锁，底层怎么实现的，越详细越好
乐观锁:cas,偏向锁都是乐观，重量锁是悲观锁。
# 内存泄漏

# jvm调优如何检查内存泄露，如何优化gc参数
http://844604778.iteye.com/blog/1960793
-XX:use...
-xmx,-xms
# 写sql 查询带日期多次考试成绩表中，每个学生的每门课最高成绩，日期要准确
select name,max(score),date?? from xx group by course 
# 写代码 旋转数组中查找某一个值
二分
# hashmap底层结构画一下，手写代码做一个url解析器，用正则方式和hashMap的数据结构
scheme://address:port/path/parameters
# 识别2的n次方，写个函数。（最快的是用位操作，大家应该都知道n&(n-1)可以去掉二进制最右的1，那2的n次幂&之后便为0）
n&(n-1)
# 自己实现http response响应头的结构及解析，用buffer（写个伪代码）
第一行是response line,包括状态码 + 协议
第二行才开始是response header
如果发现了空行，就是第三部分和第二部分的分割处。

# 海量数据找到出现次数最多的100个（内存不足的时候可以先做hash分片，最后多路merge，每次操作可以用hashMap计数，也可以自己做hash函数计数）

# redis底层实现，zset数据结构（问到了SkipList跳表这种结构）
zset的每个数据都和一个double形的分数相关联
# jvm内存模型，分代，cpu100% 怎么排查（我以为问Jconsole的使用，其实是想问linux性能监测和调优）
top
# 用int值表示ip如何做（刚好32位bit一对一映射），写个伪代码做transfer
不告诉你
# nio模型说一下
不说
# 怎么看待java跟c++（说下区别和自己的感受）
用眼看

# 100亿个数找最大1000个
遍历 + 小顶堆
调整堆结构需要O(logk)
有没有其他思路（用hash散列，计数排序）
mapReduce 	
那又给你一个数，你怎么快速告诉我是不是在这100亿个数中？

布隆过滤器或bitmap，100亿大概占用1g内存。
但是有一点要注意，bitmap是根据最大数来建立的
	
# 怎么保证进程间数据的安全？线程呢？
加密？
# 登录验证怎么做的，为什么用md5，有没有改进（+salt使md5库难解出），微信用的什么方式你知道吗？你想想应
使用rsa算法，非对称加密，一个公匙，一个私匙
# 那说到通信安全，怎么保证http的安全性，幂等性，回调同一个会话怎么标识不同请求，不同会话怎么区分（这个每个问题都画图叙述了下）
幂等不会，安全，加密
加一个token
	
# TCP 3次握手和timewait讲一下原理
前边有
# 讲一下滑动窗口，饱和了怎么处理
就你腾讯爱问这问题。。。
http://www.cnblogs.com/woaiyy/p/3554182.html	
	
http安全吗?https说一下？
get和post请求
linux怎么查看网络状态（vmstat）
vmstat的缩写是virtual memory statistics的缩写，它能实时输出系统的各种资源的使用情况，比如进程信息，内存使用情况，CPU使用率以及I/O使用情况
	
查看udp的性能，udp端口多少，什么时候用udp？
65535个端口。
为什么tcp不行？
视频，语音使用udp，tcp需要建立连接，udp要的是实时，即便是丢包也要即时传送。
qq里哪些用的tcp哪些用udp？分别针对每种情况说一下为什么
不知道

一面 5.12 

说说你对现有Web开发框架的理解
	
要你设计的话，如何实现一个线程池(就讲线程池的原理，从初始线程数，核心线程数，然后到任务队列，满了继续到最大线程数，再满了到饱和策略handler，饱和策略一般有哪几种，基本上要理解ThreadPoolExcuter的构造方法那几个参数)
	
synchronized关键字，实现原理（和Lock对比着说，说到各自的优缺点，synchronized从最初性能差到jdk高版本后的锁膨胀机制，大大提高性能，再说底层实现，Lock的乐观锁机制，通过AQS队列同步器，调用了unsafe的CAS操作，CAS函数的参数及意义；同时可以说说synchronized底层原理，jvm层的moniter监视器，对于方法级和代码块级，互斥原理的不同，+1-1可重入的原理等）
	
算法：手写一个ArrayList类，实现add，remove，等基本的方法（主要考扩容的原理和实现，重点写出扩容机制以及扩容时的copy过程）
	
然后让对这个ArrayList进行改进，使之可以应对并发的场景
	
算法：手写字符串的正则匹配，实现*和.的功能，用的递归（写了一半他说时间差不多了，思想大概了解了）。

二面 5.12 

举例说说在什么情况下会出现性能瓶颈，如何优化（答了用NIO的方式）
	
NIO的原理，jdk中有哪些工具和类去实现，如何实现（selector和channel的用法）,真的好用吗？还可以用什么？（面试官应该是想问netty，因为没有实际用过，只能给他讲了netty的原理）
	
那来说说AIO吧，和NIO什么区别（对异步的理解）,AIO在工程中如何实现的？（大概说了下ajax的回调函数），又问回调函数具体是怎么实现的（传递函数指针）。
	
然后借着异步IO想问消息队列，讲了一下几种模型和原理。（面试中没有用过没关系，只要你懂原理还是可以跟面试官讲，起码可以证明你是爱学习的）
	
项目中非技术上的困难（和甲方沟通需求，没有规范化的项目设计，需求变更太频繁等），问了我解决的方法还有以后希望怎么改进。（变相问互联网公司里面各个team以及需求方是如何合作和分工的）
	
	
讲讲Spring中怎么对初始化的bean做其他操作。（这里有三种方式，@PostConstruct注解方式，init-method的XML配置方式，InitializingBean接口方式）
	
	
三种实现上有什么区别（还好看过点源码，其实前两种是一个意思，都是通过反射的方式用aop思想实现，可以消除对spring的依赖；接口方式是直接调用afterPropertiesSet方法，效率更高点。spring加载bean时先判断接口方式，再执行配置注解方式）
	
	
算法题，一个先减后增的数组，查找目标值。（这里并不是查找最值，也不是剑指offer上的旋转数组，但是思想上也可以用二分的方式）
	
算法题，两个大数求和，要按高到低位的输入，实时输出结果的对应位，空间O，时间O(n)，不借助工具类。（要考虑实时的进位标识，以及多个9之后的连续进位标识）
	
两面完了电话让去参加新锐的现场终面，很有诚意地报销了所有的花销。新锐的三面还是有难度，基本上围着算法在问。

三面 5.12 


算法：int范围的随机数的阶乘编码实现。
（这个题如果直接按最简单的算法题肯定是不行的）
		
1.首先考虑要用字符串做运算(因为中间数太大了，只有String能保存，当然你可以借助BigInteger或BigDecimal类去辅助实现)。
2.阶乘直接计算代价太大，循环太多，考虑设计中间缓存。（正常算复杂度太高，本身就是阶乘级的，所以正常想到用时间换空间）
3.只用空间换时间的话缓存也不能覆盖全部，如果把所有的中间值保存，空间是eb级别，不现实。（这里就要达到一个空间和时间复杂度的平衡点）
4.存部分中间值用部分空间换取时间，达到空间复杂度和时间复杂度的最优平衡。（开始说的二分做分割存储之后改为等间隔做分隔存储，间隔选取多长为好？我觉得要首先确定空间复杂度的接受极限，然后尽可能减小时间复杂度，因为空间复杂度是可以有预估值的，而时间复杂度当然我们是希望约小越好的）
			
（这里说一下，我并不是一开始都想到了，只是面试官一直在提示我思路，给我时间思考，没有否定过我）


因为头一次手写白板，返回类型有错误，面试官说你这个编译器会提示什么？
	
算法，最长递增子序列，一个dp数组一个max数组，最优情况
	
ps：这个面试官应该是面试过程中遇到最nice的一个，也是我现在的老大。其实面试除了自身的因素也有面试官的因素，一个好的面试官不会随便地否定和质疑你（当然有专门压力面的），而是可以让你在放松的环境下，挖掘你真正对于一个方面的深度和理解。最后的十几分钟他并没有问我问题，只是在跟我聊天，他跟我说不管是哪个公司，真正的发展还是跟部门的方向和氛围有关系，选择的时候不要只看公司，做的业务部门方向和leader才是该去了解和考虑的。作为应届生很多时候不那么了解，这就要靠我们（指面试官）多去了解你想发展的方向。然后聊了很多成长路径和规划的事。
	
真正实习到现在一个多月，深深觉得面试就是面试，很多知识和题目都是可以准备的，而工作中面对各种情况解决问题的能力和方式才是更重要的。为了面试准备了很多，工作了发现要学的东西更多，我们真的还有很长的路要走。
	
美团（123面）

1面 1hour  5.26 

java基础，从头到尾问了个遍，都是大家准备的，但是也挺深的，包括：
	
hashMap，红黑树处理冲突，jdk7和jdk8有什么区别
	
JUC相关的集合，ConcurrentHashMap jdk7和jdk8的区别，Collections.sort函数jdk7 和 jdk8 分别怎么实现的。（总感觉这个面试官在某段时间肯定纠结过两个版本）
	
CopyOnWriteList底层是什么，适用的情况，vector的特点，实现的是List接口吗。
	
并发的问题，线程间通信三种方式
	
锁的膨胀过程，Synchronized和Lock的区别，底层的monitor实现和unsafe类的CAS函数，参数表示什么，寄存器cpu如何做）
	
volatile cpu和寄存器层面是怎么实现的。
	
线程池构造函数参数，各种类型的预设池各自的特点，ForkJoinPool是怎么实现的，多线程等等问了一个遍。
	
为什么匿名内部类的变量必须用final修饰，编译器为什么要这么做，否则会出现什么问题

数据库： 
	
索引的分类。
	
主键索引和普通索引的区别，组合索引怎么用会失效。
	
索引的前缀匹配的原理，从B树的结构上具体分析一下。
	
聚集索引在底层怎么实现的，数据和关键字是怎么存的。
	
组合索引和唯一性索引在底层实现上的区别（这个是整个一面感觉答得不好的一个问题，不太明白面试官想问啥）
	
sql的优化策略，慢查询日志怎么操作，参数含义。
	
explain 每个列代表什么含义（关于优化级别 ref 和 all，什么时候应该用到index却没用到，关于extra列出现了usetempory 和 filesort分别的原因和如何着手优化等）
	
show profile 怎么使用。
	
2面 1hour 5.27 （因为这一面问得很深，所以到现在都记得很清楚）

一个url到页面全过程（让我能说多详细说多详细，最好从OSI七层的每一层去扩展）
	
http的请求头格式（这个真的记不太清了，只说了几个有印象的标志位）
	
getpost区别，post可不可以用url的方式传参。
	
说到了url有最大长度，就问长度有限制是get的原因还是url的原因，为什么长度会有限制，是http数据包的头的字段原因还是内容字段的原因，详细说明。（在他一步步追问下答了个差不多）
	

后台服务器对于一个请求是如何做负载均衡的，有哪些策略，会出现什么样的问题，怎么解决。（说了一致性hash算法，分布式hash的特性,具体的应用场景，又非要问我知不知道这个最早在哪个公司使用的...我说这个真不知道。好像是amazon?）
	
说说http的缺点，无状态，明文传输。
		
那https是怎么做的，如何实现的？ca认证机构。
	
然后问我https ssl tcp三者关系，其中哪些用到了对称加密，哪些用到了非对称加密，非对称加密密钥是如何实现的。（还好我项目中涉及到了一些加密）
	
关于加密的私钥和公钥各自如何分配（客户端拿公钥，服务器拿私钥）
	
那客户端是如何认证服务器的真实身份，详细说明一下过程，包括公钥如何申请，哪一层加密哪一层解密。
	
java的优先级队列，如果让你设计一个数据结构实现优先级队列如何做？
		
用TreeMap复杂度太高，有没有更好的方法。	
	
hash方法，但是队列不是定长的，如果改变了大小要rehash代价太大，还有什么方法？	
	
用堆实现，那每次get put复杂度是多少（lgN）
（思想就是并不一定要按优先级排队列的所有对象，复杂度太高，但每次保证能取最大的就行，剩下的顺序不用保证，用堆调整最为合适）
	
在线编程题：敲一个字串匹配问题，写了常规代码。问kmp的代码思想，最后问了下正则中用的改进后的BM算法。（还有个比较新奇的Sunday算法，有兴趣的同学也可以看一下）
	
3面 hr 

其实写了3面，感觉根本不算面试了，就是随便介绍了下部门，然后商量实习时间(大概补招都这样吧)。因为已经决定去滴滴新锐了，就跟她说可能暑期不能实习，然后说可以秋招再联系。
	
另外美团这家要跟师弟师妹们说一声，投简历一定还是要选事业群的，千万不要选都喜欢，否则就算过了笔试，也会像我这样等两个月大概是补招才会联系到你。
	
- - -


# 基础知识
		
## 算法和数据结构
									
### 数组、链表、二叉树、队列、栈的各种操作（性能，场景）
										
### 二分查找和各种变种的二分查找
										
### 各类排序算法以及复杂度分析（快排、归并、堆）
					
### 各类算法题（手写）					
					
### 理解并可以分析时间和空间复杂度。
										
### 动态规划（笔试回回有。。）、贪心。
										
### 红黑树、AVL树、Hash树、Tire树、B树、B+树。					
					
### 图算法（比较少，也就两个最短路径算法理解吧）
					
	
			
# 计算机网络
				
					
## OSI7层模型（TCP4层）
													
### 每层的协议
													
## url到页面的过程
																							
## HTTP						
							
### http/https 1.0、1.1、2.0							
							
### get/post 以及幂等性
												
### http 协议头相关
														
### 网络攻击（CSRF、XSS）
																		
					
## TCP/IP
													
### 三次握手、四次挥手							
							
### 拥塞控制（过程、阈值）
														
### 流量控制与滑动窗口
														
### TCP与UDP比较							
							
### 子网划分（一般只有笔试有）
														
### DDos攻击
																		
					
#(B)IO/NIO/AIO						
							
## 三者原理，各个语言是怎么实现的
													
# Netty
														
# Linux内核select poll epoll
																									
# 数据库（最多的还是mysql，Nosql有redis）				
					
## 索引（包括分类及优化方式，失效条件，底层结构）
										
## sql语法（join，union，子查询，having，group by）					
					
## 引擎对比（InnoDB，MyISAM）
										
## 数据库的锁（行锁，表锁，页级锁，意向锁，读锁，写锁，悲观锁，乐观锁，以及加锁的select sql方式）
										
## 隔离级别，依次解决的问题（脏读、不可重复读、幻读）
										
## 事务的ACID
										
## B树、B+树
										
## 优化（explain，慢查询，show profile）				
					
## 数据库的范式。
										
## 分库分表，主从复制，读写分离。					
					
## Nosql相关（redis和memcached区别之类的，如果你熟悉redis，redis还有一堆要问的）
								
# 操作系统：
									
## 进程通信IPC（几种方式），与线程区别					
					
## OS的几种策略（页面置换，进程调度等，每个里面有几种算法）
										
## 互斥与死锁相关的					
					
## linux常用命令（问的时候都会给具体某一个场景）					
					
## Linux内核相关（select、poll、epoll）
															
# 编程语言（这里只说Java）：
				
把我之后的面经过一遍，Java感觉覆盖的就差不多了，不过下面还是分个类。
									
## Java基础（面向对象、四个特性、重载重写、static和final等等很多东西）
										
## 集合（HashMap、ConcurrentHashMap、各种List，最好结合源码看）
										
## 并发和多线程（线程池、SYNC和Lock锁机制、线程通信、volatile、ThreadLocal、CyclicBarrier、Atom包、CountDownLatch、AQS、CAS原理等等）					
					
## JVM（内存模型、GC垃圾回收，包括分代，GC算法，收集器、类加载和双亲委派、JVM调优，内存泄漏和内存溢出）
										
## IO/NIO相关
										
## 反射和代理、异常、Java8相关、序列化
										
## 设计模式（常用的，jdk中有的）
										
## Web相关（servlet、cookie/session、Spring<AOP、IOC、MVC、事务、动态代理>、Mybatis、Tomcat、Hibernate等）					
					
## 看jdk源码
					 
						
	
	
# 项目经历
		
这个每个人的项目不同，覆盖的技术也不一样，所以不能统一去说。这里的技巧呢，在下面也会详细说明。无非是找到自己项目中的亮点，简历上叙述的简练并且吸引眼球，同时自己要很熟悉这个点（毕竟可以提前准备）最好自己多练，就像有个剧本或者稿子一样，保证面试中可以很熟练通俗地讲出，并且让人听着很舒服。
			
		
# 实习经历
		
			
这个很抱歉，因为我是找实习的经历，所以也没有实习经历的讲述经验。
				
但我想如果你有实习经历，那面试过程的重点也会在实习做了什么上面，所以大家最好对实习所做的工作做一个总结，并且同样抓出亮点，搞懂内部原理，提前锻炼讲述的过程。
			
		
# 其他扩展技能（这个方方面面太多了，全部掌握基本上不可能，只是作为大家其他时间扩充技能的参考）
		
Nosql与KV存储（redis，hbase，mongodb，memcached等）
负载均衡（原理、cdn、一致性hash）					
消息队列（原理、kafka，activeMQ，rocketMQ）


http://blog.csdn.net/csuliyajin2012/article/details/49444867
