﻿## Java 知识点
### Jvm内存模型
- #### 方法区（线程共享）
存储已被虚拟机加载的类信息、常量、静态变量、即时编译后代码等数据。常量池位于方法区，并使用永久代来实现方法区
- #### 堆（线程共享）
所有实例域、静态域和数组元素存储在堆内存中，堆内存在线程之间共享
- #### 虚拟机栈（线程独享）
局部变量，方法定义参数和异常处理器参数存储在栈中（不只）
栈中含有栈帧，一次方法调用产生一个栈帧，存储有函数入参、出参、返回地址和上一个栈帧的栈底指针等信息。栈帧深度受内存大小影响，超出深度报StackOverflowError异常，用-xss调整参数调整，代表每个栈最大大小
- #### 本地方法栈（线程独享）
用于存放执行native方法的运行数据
- #### 程序计数器（线程独享）
当前线程所执行的字节码的指示器，通过改变计数器来选取下一条需要执行的字节码指令
- ### 相关调节参数
- > ##### Out of Memory 时打印：HeapDump-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath= 
- > ##### Heap 相关：-Xms512M(初始值) -Xmx1024M(最大值)
- > ##### -Xmn128M 新生代内存初始值 -XX:MaxNewSize=256M 新生代内存最大值 

- > ##### 永久代相关：-XX:PermSize=64M 永久代内存初始值 -XX:MaxPermSize=128M 永久代内存最大值
- > ##### -XX:MaxDirectMemorySize=128M 堆外内存最大值
- > ##### GC 优化: XX:+PrintGCDetails 打印 GC 日志 XX:+PrintGCApplicationStoppedTime 打印 GC 的停顿时间



### Java内存模型（JMM）
- #### 本地内存（线程独享）
 存储该线程以读/写共享变量的副本
- #### 主内存（线程共享）
存储线程之间的共享变量
- #### [交换流程](https://res.infoq.com/articles/java-memory-model-1/zh/resources/11.png)
- #### 重排序
包含：编译器优化、指令级并行的重排序、内存系统的重排序。后两者属于处理器重排序，执行顺序从前到后，关键字volatile作用于处理器重排序
### 线程安全
线程安全指多个线程对同一共享数据操作出现的非预期问题。既然是对数据操作，就必然有：获取的数据是否是预期的、获取到数据后对数据的操作流程执行是否是预期的、在操作过程中数据是否保持预期，对应的三要素就是：数据可见性、操作有序性、操作原子性。前两者在JMM处理得当的情况下可以保证，剩下的是操作原子性。
#### 解决方向
既然要操作原子性，那么每个线程把对共享数据的一系列操作打包成‘一个’操作统一提交，当其执行完了才执行下一个操作就可以了，其实就是将对共享数据的操作线性化，那么如果线程A在打包，线程B同时也在打包，就算线性执行包，还是有问题，不能保证打包的线性化，java中将对共享资源的线性处理转化为对另外一个资源的线性获取，称为锁，包含synchronized(lock)(基于JVM层面实现)、ReentrantReadWriteLock、ReentrantLock等。
上面都是不论如何，对对应的操作都无条件加锁，也会有一些问题：加锁、释放锁会导致线程上下文切换（受处理器核数影响），等待锁的线程也会一直等待直到拿到锁，在一些情况下优先级高的线程会因为等待优先级低的线程释放锁导致优先级倒置，因此一种想法是预期共享数据没被其他线程改动，只有当比较了预期和共享数据不同时才做另外处理，叫做乐观锁。
##### CAS
乐观锁一种实现方法是CAS（compare and swap ），主要靠三个操作数完成的： 内存位置、预期原值和新值，Java中依靠Unsafe类执行。仅靠比较预期值和共享值这种其实也有问题，如果线程A修改了值做了一些操作有改回去了，这是线程B是察觉不到的，叫做ABA问题，一种解决方法是再加一个比较值，版本号，每次修改值时版本号更新。
##### 数据库存放本版号实现
##### Java中
Java中采用的CAS:Atomic系列、concurrentHashmap、ConcurrentLinkedDeque等
### 流
按操作数据单位分为字节流和字符流，前者继承InputStream/OutputStream抽象类，操作数据单位是字节，后者继承自Reader/Writer，操作数据单位是字符
#### 缓冲流
> 缓冲流要“套接”在相应的节点流之上，对读写的数据提供了缓冲的功能，提高了读写效率，同时增加了一些新的方法。
> 
> 字节缓冲流有BufferedInputStream/BufferedOutputStream，字符缓冲流有BufferedReader/BufferedWriter，字符缓冲流分别提供了读取和写入一行的方法ReadLine和NewLine方法。
> 
> 对于输出地缓冲流，写出的数据，会先写入到内存中，再使用flush方法将内存中的数据刷到硬盘。所以，在使用字符缓冲流的时候，一定要先flush，然后再close，避免数据丢失。
#### 转换流
> InputStreamReader/OutputStreamWriter。其中，InputStreamReader需要与InputStream“套接”，OutputStreamWriter需要与OutputStream“套接”。

#### 数据流
> 提供了读写Java中的基本数据类型的功能
> DataInputStream和DataOutputStream分别继承自InputStream和OutputStream，需要“套接”在InputStream和OutputStream类型的节点流之上
#### 对象流
> 用于直接将对象写入写出。
> ObjectInputStream和ObjectOutputStream,这两个流对操作的对象有要求，需要实现Serialized接口，声明可以被序列化，这里有个关键字transient，被其声明的> 字段在序列化时被忽略

