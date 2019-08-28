# Java并发

摘自[HowToDoInJava](https://howtodoinjava.com/java-concurrency-tutorial/)

# 并发（Concurrency）vs并行（Parallelism）

## 并发（Concurrency）

当我们至少谈论2个任务或更多的时候，实际上指的是并发。当一个应用能够看上去像同一时间内执行多个任务时，我们称它为可并发的应用。尽管这里任务看来像是同时运行，但基本上它们不是同时运行的。它们只是有效利用了操作系统的**时间切片**特性。时间切片是每个任务都会运行一部分，然后进入等待状态。当第一个任务进入等待状态，CPU就会被分配给第二个任务完成它的部分任务。

操作系统会根据任务的优先级轮流给所有任务分配CPU和其他计算资源例如内存并且给它们时间去执行完成。对于最终用户来说，所有任务似乎都是并行运行的。这叫做并发。

## 并行（Parallelism）

并行实际上使用CPU的多核基础结构，通过为每个任务或子任务分配一个核心CPU，达到同时运行多个任务。

并行要求硬件具有多个处理单元。在单个CPU操作系统上，都是并发而不是并行。

## 并行与并发之间的区别

并发是指两个任务在交叉时间段内开启，运行和完成。并行是指在多核处理器上多个任务在同一时间段内完成。

Concurrency is the composition of independently executing processes, while parallelism is the simultaneous execution of (possibly related) computations.

并行是同时**处理很多事情**，然而并发是同一时刻**做很多事情**。


# Java Compare and Swap Example – CAS Algorithm

Java5中最棒的新增特性之一是在类中支持了原子操作，例如`AtomicInteger`，`AtomicLong `等等。在多线程编程中，我们可以使用这些类来简化基础操作（增加，减少等）的代码。这些类都是在内部实现了**CAS（compare and swap）**算法。

## 乐观锁和悲观锁

传统的锁机制据说是**悲观锁技术**，例如**在Java中的synchronized关键字**。悲观锁机制首先要求保证，没有其他线程来干扰你这次操作，并且只允许你有权限访问实例或方法。

		It’s much like saying “please close the door first; otherwise some other crook will come in and rearrange your stuff”.

尽管上述的方式是安全且有效地，但是它会**对你的应用耗费大量的性能**。原因很简单，等待线程将不会做任何事情除非也获得机会执行保护操作。

乐观锁是另一种锁机制，但是它在性能方面优于悲观锁。乐观锁是当你在执行更新操作时，**希望能够在没有干扰的情况下完成操作**。这种方式依赖于你在更新操作时需要去检测是否有其他线程执行了更新操作，如果有，那么就失败并进行重试。

		The optimistic approach is like the old saying, “It is easier to obtain forgiveness than permission”, where “easier” here means “more efficient”.

**CAS（compare and swap）**就是一个乐观锁机制的例子。

## CAS算法

该算法主要将给定值与内存中已经存在的值进行比较，当且仅当两个值相等时，则将内存中的值修改为新值。这样做就相当于一次简单的原子操作。原子操作保证了新的值是基于此操作之前最新的值计算得来的。如果同一时间，该值被其他线程修改了，这更新操作就会失败。这个操作的结果必须表明，操作的值是否已经被修改；这可以通过简单的Boolean响应来完成，或者返回内存中的值（不是写入的值）。

CAS操作有3个要素：

1. 在内存中有个需要被替换的值V
2. 线程上次读取的旧值A
3. 需要写入内存的新值B


		CAS says “I think V should have the value A; if it does, put B there, otherwise don’t change it but tell me I was wrong.” CAS is an optimistic technique—it proceeds with the update in the hope of success, and can detect failure if another thread has updated the variable since it was last examined.

## 一个CAS的例子

让我们通过一个例子来理解CAS的整个过程。假设内存中存的值V为10，现在有多个线程想要增加V的值并用增加后的值进行其他操作，一个非常实际的场景。现在将CAS操作进行拆解：

1. 线程1和线程2想要增加V值，它们都读取V值并将它修改为11
V = 10，A = 0，B = 0

2. 现在线程1分配到资源，首先执行，先将最后读取到的值与现在内存中的V值进行比较
V = 10，A = 10，B = 11
```Java
if A = V
	V = B
else
	operation failed
	return V
```

很显然V的值将会被覆盖更新为11，操作成功

3. 现在线程2分配到资源并开始执行与线程1相同的操作
V = 11，A = 10，B = 11
```Java
if A = V
	V = B
else
	operation failed
	return V
```

4. 在这种情况下，V与A不相等，所以操作不成功，V的值没有被更新，并且返回当前V的值。现在线程2用以下的值进行重试：
V = 11，A = 11，B = 12
这时，条件满足，V被重写成12并返回给线程2。

总之，当多个线程同时使用CAS尝试去修改共享参数的值时，只有一个会赢并进行更新操作，剩下的都会失败。但是剩下的线程不会暂停。它们可以进行重试操作或者不做任何事情。

# synchronized关键字

**synchronized关键字**会对一个代码块或者一个方法进行重要标记。被标记的代码块或方法一次只允许一个线程可以执行，并且这个线程会持有该标记内容的锁。

**synchronized关键字**在编写应用程序的并发部分有很大的帮助。它能对块中的共享资源起到保护的作用。

`synchronized`关键字的适用范围如下：

* 代码块
* 方法

## 1.Java同步代码块

**1.1.语法**

编写一个同步块的通用语法可以参见下面的代码。这里**lockObject**是一个对象的引用，该对象的锁与synchronized语句所代表的监视器相关联。

```Java
synchronized( lockObject )
{
	// synchronized statements
}
```

**1.2.内部工作流程**

当一个线程想要在同步代码块中执行同步语句时，它必须向`lockObject`的监视器获取对象锁。一次只能有一个线程获取到对象锁。所有其他线程必须等到当前获取到锁的线程执行完它的操作。

通过这种方式，synchronized关键字保证了同一时间，只有一个线程可以执行同步代码块中的代码并且防止其他线程污染在代码块中的共享数据。

需要记住，如果一个线程执行`sleep()`方法，它不会释放它所持有的锁。在该线程的睡眠期间，没有其他线程能够执行同步代码块中的代码。

如果使用`synchronized(lock)`的对象锁为`null`，那么Java的同步时会抛出**NullPointerException**异常。

**1.3.Java同步代码块例子**

在给出的例子中，我们有一个拥有`printNumbers()`方法的`MathClass`类。该方法的将会一次打印从数字1到数字N的值。

注意，printNumbers()方法的代码是在同步代码块中的。

```Java
public class MathClass
{
	void printNumbers(int n) throws InterruptedException
	{
		synchronized (this)
		{
			for (int i = 1; i <= n; i++)
			{
				System.out.println(Thread.currentThread().getName() + " :: " + i);
				Thread.sleep(500);
			}
		}
	}
}
```

这里创建两个线程使其同一时间执行`printNumbers()`方法。由于这里使用了同步块，所以只有一个线程允许执行，而另一个线程必须等到该线程执行完毕才能执行。

```Java
public class Main
{
	public static void main(String args[])
	{
		final MathClass mathClass = new MathClass();
		
		//first thread
		Runnable r = new Runnable()
		{
			public void run()
			{
				try{
					mathClass.printNumbers(3);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		};
		
		new Thread(r, "ONE").start();
		new Thread(r, "TWO").start();
	}
}
```

## 2.Java同步方法

**2.1.语法**

编写一个同步方法的一般语法如下所示。

```Java
<access modifier> synchronized method( parameters )
{
	// synchronized code
}
```

**2.2.内部工作流程**

与同步代码块类似，一个线程必须获取与同步方法相关联对象监视器的锁。同步方法的情况下，锁对象为——

* 如果同步方法是静态方法，锁对象即为**类对象**。
* 如果同步方法时非静态方法，锁对象即为**实例对象**。

synchronized关键字是**重入锁**。意思是如果在一个同步方法中调用另外一个需要获取相同锁的同步方法，那么当前持有锁的线程就可以不需要获取锁而直接进入。

**2.3.Java同步方法例子**

和同步代码块类似，在`printNumber()`方法上使用synchronized关键字使得它成为同步方法。

```Java
public class MathClass
{
	synchronized void printNumbers(int n) throws InterruptedException
	{
		for (int i = 0; i < n; i++)
		{
			System.out.println(Thread.currentThread().getName() + " :: " + i);
			Thread.sleep(500);
		}
	}
}
```