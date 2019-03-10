### 78. 同步对共享可变数据的访问 ###
`synchronized`关键字确保一个方法或代码块同一时间只能被一个线程执行。

同步不仅可以阻止线程观察处于非持续状态的对象，也确保了每一个线程在进入被同步的方法或代码块的时候会看到之前在同一个所控制下的更改。

Java语言规范规定除`long`和`double`以外类型的变量的读和写都是原子性的。换句话说，读取一个除`long`和`double`以外类型的变量的值得返回结果一定是先前某个线程存放进去的，即使正有多个线程在同时修改该变量且并没有使用同步。

同步起到2个重要的作用：

- 线程对公共资源访问的互斥
- 线程之间以共享资源为媒介的通信。

一个线程不要使用`Thread.stop`方法来让其他线程停止执行。更好的方法是让需要停止的线程B查看一个初始化值为`false`，但可以被提出停止要求的线程A修改为`true`的布尔型变量，从而使线程B可以对线程A的要求作出响应。

如果没有同步，则下列代码中的第一个while循环会被虚拟机转变为第二个循环的形式，此时就可能导致死循环，因为系统不知道后台监督该标志的线程什么时候能看到对该标志作出的修改。此时该标志产生了隐身错误。

		//第一个while循环
		while (!stopRequest)
			i++;

		第二个循环
		if (!stopRequest)
			while (true)
				i++;

一个解决方法是同步对`stopRequest`的访问

		public class StopThread {
			private static boolean stopRequested;

			private static synchronized void requestStop() {
				stopRequest = true;
			}

			private static synchronized boolean stopRequested() {
				return stopRequested;
			}

			public static void main(String[] args) {
				Thread backgroundThread = new Thread(() -> {
					int i = 0;
					while (!stopRequested)
						i++;
				});
				backgroundThread.start();

				TimeUnit.SECOND.sleep(1);
				requestStop();
			}
		}

正确有效的同步应该是对变量的读，写操作都同步。只同步一个操作是无法起到完全的同步作用的。

使用`volatile`修饰变量虽然无法起到让访问它的方法互斥的作用，但是可以保证所有读取它的操作都能看到最后一次修改后写入的值。

`++`运算符的操作不是原子性的，它分为两步，第一步读取数据，第二步修改并写入数据。所以如果有一个线程在这两步操作之间访问了它所操作的变量，就会导致安全错误。

一个解决方法是使用`synchronized`关键字来修饰更改该变量值的方法的声明，并取消使用`volatile`关键字修饰该变量。
第二种方法是使用`java.util.concurrent.atomic`包中的`AtomicLong`类。这个包提供了一些基本数据类型用于单个变量的无锁，线程安全的编程。该包不仅像`volatile`一样提供了同步的通信功能，也提供了同步所需的原子性操作。

最佳的避免同步可能出现的问题的方法是根本不要共享可变数据。换句话说，就是可变数据应该只被单线程使用。如果采用了这个策略，则应该在文档中说明，作为将来的参考。

如果一个线程在一段时间内修改了数据对象，然后再将其共享给其他线程，那么哪怕只同步了这个共享步骤，也是可以接受的。而且如果该数据对象再不会被更改，那么其他线程就可以不需要同步对他的访问。这个对象也被认为是可以当做不可变对象使用。把这种对象从一个线程传递到另一个线程的过程称之为对该变量的安全发布。安全发布有很多种形式：

- 在类的初始化阶段使用静态成员变量存储该对象
- 使用`volatile`成员变量，最终成员变量，以及需要通过正常锁才能访问的成员变量来存储该对象
- 将该对象存储在一个并发集合中

当有多个线程分享可变数据时，每个线程对该数据的读和写都应该要实现同步。否则就会导致该数据产生隐身错误和安全错误。如果仅需要使用该数据作为线程之间通信的标志，而不需要操作互斥，则可以使用`volatile`关键字来实现片面的同步。

### 79. 避免过度的同步 ###
永远不要在同步方法或代码块中把控制权交给用户。换句话说，就是不能在同步的方法或代码块中调用被设计为用来重写的方法，或者由用户来提供的功能对象。

当系统已有的功能性接口的名称或者其方法名不能准确地表达我们所需要的功能时，可以考虑自定义一个功能接口，该接口可以包装一个标准的功能性接口，如将其作为它的方法的参数，来实现方法的回调。

如果有一个线程正在遍历某个列表，那么另外一个线程就不能对该列表做出更改，包括增加，删除和修改元素的值。

Java7以后，可以在`catch`语句中捕捉多个异常，以`|`隔开

把对界外方法（alien methods）的调用放到同步代码块之外。比如在同步代码块中为共享数据建立一个快照，然后就可以在代码块的外部使用这个快照来进行相关操作而不需要同步该数据了。除此之外，一个更好的解决方法是使用并发集合，如`CopyOnwriteArrayList`类，该类是`ArrayList`类的变体，该类所有涉及更改操作的方法都是在内部创建一个底层数组的新鲜复本并在此复本基础上进行操作的。由于这个内部数组从来都不会改变，因此遍历它不需要加锁，速度也很快。虽然这个类的性能有些低，但非常适合用作观察者列表，因为该列表很少会被改动但却需要频繁地遍历。在同步代码块外部被调用的界外方法被称为开放调用（open call）。

应该在同步域内安排尽量少的工作。如果某个工作比较耗费时间，就应该找一个方法将其移出同步域内。

在编写可变类时有2个选择：

- 完全忽略同步，让用户根据需要从外部控制同步
- 在内部实现同步，从而让该类做到线程安全

通常只有在内部实现同步会比从外部通过锁住整个对象来实现同步带来非常大的并发性能的提升时，才考虑第二种方案。如果不确定，就采取第一种方案，并在文档中注明该类不是线程安全的。

如果一个方法修改了某个静态成员变量，并且该方法还有可能被多个线程同时调用，那么就必须同步对该成员变量的访问。

### 80. 倾向于用executors， tasks， streams来替代threads ###
`java.util.concurrent`包中包含了一个`Executor Framework`，这是一个灵活的基于接口的任务执行机制。使用以下代码来使用该机制运行和结束任务：

		//创建一个执行任务的服务
		ExecutorService exec = Executors.newSingleThreadExecutor();
		//使用服务运行一个任务
		exec.execute(runnable);
		//关闭服务
		exec.shutdown();

这个服务机制还有很多其他功能，比如：

- 使用`get`方法来等待特定任务完成；
- 使用`invokeAny`或`invokeAll`方法来等待一个集合内的任意一个或全部方法都结束；
- 使用`awaitTermination`方法来等待关闭一个执行服务；
- 使用`ExecutorCompletionService`来在任务一个一个结束后获取任务的执行结果；
- 使用`ScheduledThreadPoolExecutor`来安排任务在特定时间或固定间隔之间运行。

`java.util.concurrent.Executors`类提供了不同的静态工厂方法来创建不同的执行服务。如果需要多个线程来处理请求队列中的请求时，可以使用其中的一个静态工厂方法来创建一个称之为线程池（thread pool）的执行服务。如果需要定制自己的线程池对象，可以直接使用`ThreadPoolExecutor`类来创建线程池对象。

对于小程序或者轻负载服务器，可以`Executors.newCachedThreadPool`方法来创建一个简单的带缓冲功能的线程池对象。但这个对象并不适用于高负载服务器，因为该线程池内，提交的任务不会被列队，而是立刻被一个线程执行。如果没有可用的线程，则会创建一个新的线程。高负载服务器最好使用`Executors.newFixedThreadPool`类来创建一个容量固定的线程池，或者直接使用`ThreadPoolExecutor`类来实现对该线程池的控制最大化。

不仅应该避免编写自己的工作队列，通常也应该避免直接操作线程。当直接操作线程时，一个`Thread`将作为一个工作单位和执行该工作单位的机制来使用。

而在执行者框架中，工作单位和执行机制是分开的。

- 工作单位（a unit of work）的最关键的抽象是任务（task）。有两种类型的任务：`Runnable`和`Callable`（和`Runnable`类似，但它有返回值并可能抛出异常）。
- 执行任务的执行机制通常是执行者服务（executor service）。

分支-合并任务（fork-join taskhs）可以被分解成更小的子任务，由`ForkJoinTask`类的对象代表并由被称为分支-合并池（fork-join pool）的服务来执行。该服务所使用的线程会在完成自己的分配任务后，从其他线程（同一服务控制的线程）“窃取”任务，让所有的线程都处于忙碌状态，从而实现高CPU使用率，低延迟。并发流就是基于分支-合并池编写的。

### 81. 倾向使用高级的并发工具来代替`wait`和`notify`方法 ###
在`java.util.concurrent`包中的高级并发工具分为三大类：

- 执行者框架
- 并发集合
- 同步器

并发集合是普通集合接口的高并发性能实现，因为自带内部并发管理，所以不可能从并发集合中排除并发活动，对它们加锁只会减慢程序。

因为不能排除并发集合上的并发活动，所以也不能以原子方式组合对它们的方法调用。也就是说，每个方法的调用都可能引发该集合的并发活动，所以将多个基于该集合的方法调用通过原子方式组合是无法达到排除并发活动而实现线程安全的目的的。因此，并发集合接口配备了依赖于状态的修改操作，这些操作将几个原语组合成单个原子操作。

		// Concurrent canonicalizing map atop ConcurrentMap - not optimal
		private static final ConcurrentMap<String, String> map = new ConcurrentHashMap<>();

		public static String intern(String s) {
			String previousValue = map.putIfAbsent(s, s);
			return previousValue == null ? s : previousValue;
		}

		//ConcurrentHashMap针对检索操作进行了优化，例如get。 因此，如果get表明有必要，最初只需调用get并调用putIfAbsent
		// Concurrent canonicalizing map atop ConcurrentMap - faster!
		public static String intern(String s) {
			String result = map.get(s);
			if (result == null) {
				result = map.putIfAbsent(s, s);
				if (result == null)
					result = s;
			} 
			return result;
		}
	
使用`ConcurrentHashMap`优先于`Collections.synchronizedMap`。

一些集合接口使用阻塞操作进行扩展，这些操作等待（或阻塞）直到可以成功执行。例如适用于工作队列（也被称为生产者-消费者队列）并被大多数`ExecutorService`实现所使用的`BlockingQueue`接口。该接口中的`take`默认方法从队列中删除并返回head元素，如果队列为空则等待。

同步器是使线程能够彼此等待的对象，允许它们协调它们的活动。 最常用的同步器是`CountDownLatch`和`Semaphore`。 不太常用的是`CyclicBarrier` 和`Exchanger`。 最强大的同步器是`Phaser`。

`CountDownLatch`是一次性使用的屏障，允许一个或多个线程等待一个或多个其他线程执行某些操作。`CountDownLatch`的唯一构造函数接受一个`int`，它是在允许所有等待的线程继续之前必须在`CountDownLatch`对象上调用`countDown`方法的次数。

		// Simple framework for timing concurrent execution
		public static long time(Executor executor, int concurrency,
									Runnable action) throws InterruptedException {
			CountDownLatch ready = new CountDownLatch(concurrency);
			CountDownLatch start = new CountDownLatch(1);
			CountDownLatch done = new CountDownLatch(concurrency);
			
			for (int i = 0; i < concurrency; i++) {
				executor.execute(() -> {
					ready.countDown(); // 告诉计数器完成准备
					try {
						start.await(); // 等待所有的工作准备就绪
						action.run();
					} catch (InterruptedException e) {
						Thread.currentThread().interrupt();
					} finally {
						done.countDown(); // 告诉计数器工作完成
					} 
				});
			}
			
			ready.await(); // 等待所有的工作准备就绪
			long startNanos = System.nanoTime();
			start.countDown(); // 开始工作
			done.await(); // 等待所有工作完成
			return System.nanoTime() - startNanos;
		}

注意传递给`time`方法的执行者必须允许创建至少与给定并发级别一样多的线程，否则测试将永远不会完成。

对于间隔计时，请始终使用`System.nanoTime`而不是`System.currentTimeMillis`。`System.nanoTime`更准确，更精确，不受系统实时时钟调整的影响。

始终使用wait循环模式来调用wait方法; 永远不要在循环之外调用它。 循环用于测试等待前后的状况。

		// 通常使用wait方法的模板
		synchronized (obj) {
			while (<condition does not hold>)
				obj.wait(); // 释放锁并在被唤醒时再次获取他
			... // 根据条件执行相应的操作
		}

在等待之前测试条件并在条件满足的情况下跳过等待。 如果条件已经存在并且在线程等待之前已经调用了`notify`（或`notifyAll`）方法，则无法保证线程将从等待中唤醒。

在等待后再次测试条件并在条件不满足的时候再次等待是确保安全必须要做的事情。

当条件不成立时，线程可能会被唤醒的原因有多种：

- 另一个线程可以获得锁并在线程调用`notify`和等待线程醒来之间改变了保护状态。也就是说通知线程没有改变条件的状态，但是在它调用`notify`方法前，另一个线程取得锁并改变了条件，再将锁还给通知线程让他继续执行`notify`方法来唤醒等待线程。
- 当条件不成立时，另一个线程可能会意外或恶意地调用`notify`。在公共可访问对象的同步方法中中的任何`wait`方法都容易受到此问题的影响。
- 通知线程在唤醒等待线程时可能过于“慷慨”。 例如，即使只有一些等待的线程满足条件，通知线程也可以调用`notifyAll`。
- 等待线程可以（很少）在没有通知的情况下唤醒。 这被称为虚假唤醒。

总是使用`notifyAll`方法来代替`notify`方法。在新代码中尽量不要使用`wait`和`notify`方法。

### 82. 在文档中注明线程安全性 ###
方法声明中的`synchronized`关键字是实现细节，而不是API的一部分。

要启用安全的并发使用，类必须清楚地记录它支持的线程安全级别。常见的等级划分为：

- Immutable 不变的：此类的实例显示为常量。无需外部同步。包括`String`，`Long`和`BigInteger`（第17项）。
- Unconditionally thread-safe 无条件线程安全：此类的实例是可变的，但该类具有足够的内部同步，以便可以同时使用其实例而无需任何外部同步。包括`AtomicLong`和`ConcurrentHashMap`。
- Conditionally thread-safe 有条件线程安全：像无条件线程安全一样，除了某些方法需要外部同步以便安全地并发使用。包括`Collections.synchronized`包装器返回的集合，其迭代器需要外部同步。
- Not thread-safe 不是线程安全：这个类的实例是可变的。要同时使用它们，客户端必须使用客户端选择的外部同步来包围每个方法调用（或调用序列）。包括通用集合实现，例如`ArrayList`和`HashMap`。
- Thread-hostile 线程敌对：即使每个方法调用都被外部同步包围，此类对于并发使用也是不安全的。线程敌意通常是在没有同步的情况下修改静态数据。

记录有条件的线程安全类需要小心。必须指明哪些调用序列需要外部同步，以及必须获取哪个锁（或在极少数情况下，多个锁）才能执行这些序列。通常它是实例本身的锁，但也有例外。例如，`Collections.synchronizedMap`的文档说明了这一点：

		当迭代任何返回的map对象的集合视图时，用户必须手动同步该map对象。
		Map<K, V> m = Collections.synchronizedMap(new HashMap<>());
		Set<K> s = m.keySet(); // 不需要放在同步代码块中
		...
		synchronized(m) { // 同步m，而不是s
			for (K key : s)
				key.f();
		}

类的线程安全性的描述通常属于类的doc注释，但具有特殊线程安全属性的方法应在其自己的文档注释中描述这些属性。没有必要记录枚举类型的不变性。除非从返回类型中显而易见，否则静态工厂必须记录返回对象的线程安全性，如`Collections.synchronizedMap`（上文）所示。

当类承诺使用可公开访问的锁时，它允许客户端以原子方式执行一系列方法调用，但它与并发集合（如`ConcurrentHashMap`）使用的高性能内部并发控制不兼容。此外，客户端可以通过长时间保持可公开访问的锁来发起拒绝服务攻击。

要防止此拒绝服务攻击，可以使用私有锁对象而不是使用`synchronized`方法（这意味着可公开访问的锁）：

		// Private lock object idiom - thwarts denial-of-service attack
		private final Object lock = new Object();

		public void foo() {
			synchronized(lock) {
				...
			}
		}

锁应始终声明为final。无论使用的是普通的监视器锁（如上所示）还是使用`java.util.concurrent.locks`包中的锁，都是如此。

私有锁对象类的这个习惯用法只能用于无条件的线程安全类。 有条件的线程安全类不能使用这个习惯用法，因为它们必须在文档中说明在执行某些方法调用序列时客户端要获取的锁，从而暴露了实现细节。

使用私有锁对象特别适合用于被设计用来继承的类。如果这样的类要使用其实例作为锁，则其子类很容易在无意中干扰父类的操作，反之亦然。通过为不同的目的使用相同的锁，子类和父类可能最终互相干扰到彼此。

小结：每一个类应该清楚地以措辞谨慎的文字描述或线程安全有关的注解说明它的线程安全性。有条件的线程安全类必须记录哪些方法调用序列需要外部同步，在执行这些序列时需要获取哪些锁。如果编写无条件的线程安全类，请考虑使用私有锁对象代替同步方法。这可以保护您免受客户端和子类的同步干扰。

### 83. 谨慎使用延迟初始化 ###

在存在多个线程的情况下，延迟初始化会很麻烦。如果两个或多个线程共享一个延迟初始化的字段，则必须采用某种形式的同步，否则可能导致严重的错误（第78项）。

在大多数情况下，常规初始化优于延迟初始化。 以下是常规初始化的实例字段的典型声明。 注意使用final修饰符（Item 17）：

		// 常规初始化
		private final FieldType field = computeFieldValue();

如果使用延迟初始化是为了破坏初始化环路（类A的构造器中创建了类B的实例，类B的构造器中创建了类A的实例），则应该使用同步的访问器访问该变量，因为它是最简单，最清晰的替代方法：

		// Lazy initialization of instance field - synchronized accessor
		private FieldType field;

		private synchronized FieldType getField() {
			if (field == null)
				field = computeFieldValue();
			return field;
		}

如果需要在静态成员变量上使用延迟初始化来提高性能，请使用以下这个惯用方法-延迟初始化持有者类（lazy initialization holder class）：

		// Lazy initialization holder class idiom for static fields
		private static class FieldHolder {
			static final FieldType field = computeFieldValue();
		}

		private static FieldType getField() { return FieldHolder.field; }

当第一次调用getField时，它首次读取`FieldHolder.field`时导致`FieldHolder`类的初始化。 这个用法的优点在于`getField`方法不是同步的，只执行字段访问，因此延迟初始化几乎不会增加访问成本。

如果需要在实例变量上使用延迟初始化来提高性能，请使用双重检查法。 这个方法避免了初始化后访问变量加锁的成本（第79项）。该方法检查变量的值两次（因此称为双重检查）：第一次仅仅是查看变量是否被初始化，不会改变变量的状态，所以没有加锁。如果变量没有被初始化，则在第二次访问该变量前加锁，然后检查该字段此时是否被初始化，如果没有，则开始初始化该变量。因为变量初始化后就释放了锁，所以必须将该变量声明为`volatile`

		// Double-check idiom for lazy initialization of instance fields
		private volatile FieldType field;

		private FieldType getField() {
			FieldType result = field;
			if (result == null) { // First check (no locking)
				synchronized(this) {
					if (field == null) // Second check (with locking)
						field = result = computeFieldValue();
				}
			}
			return result;
		}

需要反复初始化的实例变量可以省去第二次检查。

		// Single-check idiom - can cause repeated initialization!
		private volatile FieldType field;

		private FieldType getField() {
			FieldType result = field;
			if (result == null)
				field = result = computeFieldValue();
			return result;
		}

基本数据类型中数字类型的变量可以使用`0`作为未初始化的判断依据（需要注意故意初始化为0的情况）。

小结：优先使用常规化初始化方法。如果必须延迟初始化以满足性能需要或者打破初始化环路，则应该：

- 对于静态变量，使用初始化持有者类；
- 对于实例变量，使用双重检查方法；可以重复初始化的变量，可以考虑单一检查方法。

### 84. 不要依赖线程调度器 ###

任何依赖于线程调度程序以获得正确性或性能的程序都可能是不可移植的。

编写健壮、响应迅速的可移植程序的最佳方法是确保可运行线程的平均数量不会明显大于处理器数量。注意可运行线程的数量与线程总数不同，后者可能要高得多，因为包含了等待的线程。

保持可运行线程数量较少的主要方法是让每个线程做一些有用的工作，然后等待更多的工作。就Executor Framework而言（第80项），这意味着适当地调整线程池的大小并保持任务简短但不会太短。

线程不应该忙于等待，即反复检查共享对象的状态是否改变。当程序因为有些线程无法获得相对于其他线程足够的CPU时间而几乎无法工作时，要抵制想调用`Thread.yield`来“修复”程序的诱惑。

通过设置线程的优先级来调度线程也是不可取的。因为也会导致移植的问题。

### 85. 优先使用Java序列化的替代方案 ###

在反序列化期间调用的执行潜在危险活动的方法称为小工具。可以一起使用多个小工具来形成小工具链。一个足够强大的小工具链允许攻击者在底层硬件上执行任意本机代码，只要有机会提交精心设计的字节流进行反序列化。

在不使用任何小工具的情况下，您可以通过导致需要很长时间反序列化的短流的反序列化来轻松地发起拒绝服务攻击。这种流被称为反序列化炸弹。下面这个例子只使用了`Set`和字符串：

		// Deserialization bomb - deserializing this stream takes forever
		static byte[] bomb() {
			Set<Object> root = new HashSet<>();
			Set<Object> s1 = root;
			Set<Object> s2 = new HashSet<>();
			for (int i = 0; i < 100; i++) {
				Set<Object> t1 = new HashSet<>();
				Set<Object> t2 = new HashSet<>();
				t1.add("foo"); // Make t1 unequal to t2
				s1.add(t1); s1.add(t2);
				s2.add(t1); s2.add(t2);
				s1 = t1;
				s2 = t2;
			} 
			return serialize(root); // Method omitted for brevity
		}

对象图由201个`HashSet`实例组成，每个实例包含3个或更少的对象引用。整个流的长度只有5,744字节，但问题是反序列化`HashSet`实例需要计算其元素的哈希码。根哈希集的2个元素本身是包含2个哈希集元素的哈希集，每个哈希集元素包含2个哈希集元素，依此类推，深度为100个级别。因此，反序列化集会导致`hashCode`方法被调用超过2100次。

避免序列化漏洞利用的最佳方法是永远不要反序列化任何东西。没有理由在你编写的任何新系统中使用Java序列化。应该使用其他的跨平台结构化数据表示。这些表示的共同之处在于它们比Java序列化简单得多。它们不支持任意对象图的自动序列化和反序列化。相反，它们支持由一组属性 - 值对组成的简单结构化数据对象。仅支持少数基本数据类型和数组数据类型。

领先的跨平台结构化数据表示是JSON和Protocol Buffers，也称为protobuf。 JSON由Douglas Crockford设计用于浏览器 - 服务器通信，并且协议缓冲器由Google设计用于在其服务器之间存储和交换结构化数据。即使这些表示有时被称为语言中性，但JSON最初是为JavaScript开发的，而protobuf是为C++开发的。

JSON和protobuf之间最显着的区别是JSON是基于文本的，人类可读的，而protobuf是二元的，效率更高; 并且JSON完全是数据表示，而protobuf提供模式schema（类型type）来记录和加强适当的用法。尽管protobuf比JSON更有效，但JSON对于基于文本的表示非常有效。虽然protobuf是二进制表示，但它提供了一种替代文本表示，用于需要人类可读性的用途（pbtxt）。

如果无法完全避免Java序列化，那么下一个最佳选择是永远不会反序列化不受信任的数据。 特别是永远不应接受来自不受信任来源的RMI流。

如果无法避免序列化，并且不确定需要反序列化的数据的安全性，则应该使用Java 9中添加的对象反序列化过滤并向后移植到早期版本（java.io.ObjectInputFilter）。此工具允许指定在反序列化之前应用于数据流的过滤器。它以类为基本单位来进行控制，允许接受或拒绝某些类。默认接受类并拒绝潜在危险类列表称为黑名单;默认情况下拒绝类并接受假定安全的列表称为白名单。首选使用白名单而不是黑名单。`Serial Whitelist Application Trainer（SWAT）`的工具可用于为应用程序自动准备白名单。

小结： 如果是从头开始设计系统，使用跨平台的结构化数据表示，例如JSON或protobuf。 不要反序列化不受信任的数据。 如果必须这样做，使用对象反序列化过滤，但这也不能保证阻止所有攻击 避免编写可序列化的类。 如果你必须这样做，请谨慎行事。

### 86. 实现`Serializable`接口时加倍小心 ###

实现`Serializable`的一个主要成本是它降低了发布后再更改类的实现的灵活性。当类实现`Serializable`时，其字节流编码（或序列化格式）将成为其导出API的一部分。在广泛分发类之后，通常需要永久支持序列化格式，就像需要支持导出的API的所有其他部分一样。 如果没有努力设计自定义序列化格式而接受了默认格式，则该类的序列化格式将永远与其原始内部表示相关联。换句话说，如果接受了默认的序列化格式，则该类的私有和包私有实例变量将成为其导出API的一部分，并且最小化对变量的访问（第15项）的做法将失去其作为信息隐藏工具的有效性。

如果接受了默认的序列化格式并在稍后更改了类的内部表示，则将导致序列化格式的不兼容。尝试使用旧版本的类序列化实例并使用新版本对其进行反序列化（反之亦然）的客户将遇到程序失败。可以在保持原始序列化形式（使用`ObjectOutputStream.putFields`和`ObjectInputStream.readFields`）的同时更改内部表示，但这可能很困难并且在源代码中留下可见的瑕疵。如果选择将类序列化，应该仔细设计一个能长期使用的高质量序列化格式（第87,90项）。

在类的进化过程中，一个由序列化带来的限制是关于流的唯一标识符（stream unique identifiers），通常称为系列版本UID（serial version UID）。每个可序列化的类都有一个与之关联的唯一标识号。如果未通过声明名为`serialVersionUID`的`long`类型的静态最终变量来指定此数字，则系统会在运行时通过将加密哈希函数（SHA-1）应用于类的结构来自动生成它。此值受类的名称，其所实现的接口及其大多数成员（包括编译器生成的合成成员）的影响。如果更改任何这些内容，例如，添加便捷方法，自动生成的系列版本UID会更改。如果声明串行版本UID失败，则兼容性将被破坏，从而导致运行时出现`InvalidClassException`。

实现`Serializable`的第二个成本是它增加了错误和安全漏洞的可能性（第85项）。通常，对象是通过构造器来创建；但序列化也是一种创建对象的语言额外机制。无论是接受默认行为还是覆盖默认行为，反序列化都是一个“隐藏的构造函数”。 因为没有与反序列化相关联的显式构造函数，所以很容易忘记必须确保它保证构造函数建立的所有不变量，并且它不允许攻击者访问构造中的对象的内部。

实现`Serializable`的第三个成本是它增加了与发布新版本类相关的测试负担。修改可序列化类时，重要的是检查是否可以序列化新版本中的实例并在旧版本中反序列化，反之亦然。必须确保序列化 - 反序列化过程成功并且它会产生一个原始对象的忠实副本。

为继承而设计的类（第19项）应该很少实现`Serializable`，接口也应该很少扩展它。

专为实现`Serializable`的继承而设计的类包括`Throwable`和`Component`。`Throwable`实现`Serializable`，因此RMI可以从服务器向客户端发送异常。`Component`实现`Serializable`，因此可以发送，保存和恢复GUI。

如果在实现一个可同时序列化及扩展具有实例变量的类，则需要注意：如果有任何实例变量存储的是不变量，则必须防止子类覆盖`finalize`方法，这可以通过在该类中重写`finalize`并将其声明为`final`来完成。如果类的实例变量被初始化为默认值（整数类型为零，布尔值为`false`，引用类型为`null`），则它的不变量可能会需要重新赋值（延迟初始化），此时必须添加此`readObjectNoData`方法：

		// readObjectNoData for stateful extendable serializable classes
		private void readObjectNoData() throws InvalidObjectException {
			throw new InvalidObjectException("Stream data required");
		}

如果为继承而设计的类不可序列化，则要求该类具有可访问的无参数构造器从而实现它的子类的正常反序列化。如果没有提供这样的构造器，则子类必须使用序列化代理模式（Item 90）。

内部类（第24项）不应实现`Serializable`。但是静态内部类可以实现`Serializable`。

### 87. 考虑使用自定义的序列化格式 ###

不要在没有事先考虑是否合适的情况下使用默认的序列化格式。

对象的默认序列化格式是对以该对象为起点的对象图的物理表示进行合理有效编码。它描述了此对象中以及可从此对象访问的每个对象中包含的数据。它还描述了所有这些对象相互链接的拓扑。对象的理想序列化格式仅应该包含对象表示的逻辑数据，独立于物理表示。

如果对象的物理表示与其逻辑内容相同，则默认的序列化格式可能是合适的。 例如，对于以下类，默认的序列化形式是合理的，这简单地表示一个人的姓名：

		// Good candidate for default serialized form
		public class Name implements Serializable {
		/**
		* Last name. Must be non-null.
		* @serial
		*/
		private final String lastName;
		/**
		* First name. Must be non-null.
		* @serial
		*/
		private final String firstName;
		/**
		* Middle name, or null if there is none.
		* @serial
		*/
		private final String middleName;
			... // Remainder omitted
		}

即使确定默认的序列化格式是合适的，通常也必须提供`readObject`方法以确保不变量和安全性。对于`Name`类，`readObject`方法必须确保字段`lastName`和`firstName`为非空值。

注意，对`lastName`，`firstName`和`middleName`变量有文档注释，因此即使这些变量是私有的，但它们被用于定义一个公共API，即该类的序列化格式，而公共API必须被文档说明。`@serial`注解的存在告诉Javadoc将此文档放在一个记录序列化格式的特殊页面上。这条规则也适用于私有方法。

当对象的物理表示与其逻辑数据内容明显不同时，使用默认的序列化格式有四个缺点：

- 它将导出的API永久绑定到当前内部表示。 
- 它会消耗过多的空间。
- 它可能会消耗过多的时间。
- 它可能导致堆栈溢出。

`transient`修饰符指示要从类的默认序列化格式中省略该实例变量：

`writeObject`方法首先应该调用`defaultWriteObject`方法，而`readObject`做的第一件事就是调用`defaultReadObject`。如果所有类的实例变量都是`transient`，可以不用调用`defaultWriteObject`和`defaultReadObject`，但通常都会调用它们，因为可以方便在以后的版本中添加非`transient`实例变量，并同时保持向后和向前兼容性。如果实例在更高版本中序列化并在早期版本中反序列化，则添加的变量将被忽略。如果早期版本的`readobject`方法无法调用`defaultReadObject`，则反序列化将因`StreamCorruptedException`而失败。

如果某个类的不变量与具体的实现相关联，则其不变量值可能在序列化-反序列化过程中被破坏。例如，哈希表的物理表示是包含键值条目的一系列桶。条目所在的桶是基于其密钥的哈希码的函数。通常，它不保证不同实现所产生的值是相同的。实际上，相同实现，不同的运行都不能保证相同。因此，接受哈希表的默认序列化格式将构成严重错误。序列化和反序列化哈希表可能会产生一个不变量严重损坏的对象。

无论是否接受默认的序列化格式，当调用`defaultWriteObject`方法时，每个未标记为`transient`的实例变量都将被序列化。因此，每个可以声明为`transient`的实例变量都应该声明（包括派生字段，其值可以从主数据字段计算，例如缓存的哈希值）。要确认每一个非transient变量的值都必须是对象逻辑状态的一部分。

如果使用了默认序列化格式并且有一个或多个变量标记为`transient`，那么在反序列化时，这些变量都会被初始化为其默认值：对象引用字段为null，数字基本字段为零，布尔字段为false。如果需要在反序列化时为他们提供非默认值，则必须提供一个`readObject`方法，该方法调用`defaultReadObject`方法，然后将transient变量设定为想要的值（第88项）。或者，这些变量可以在第一次使用时进行延迟初始化（第83项）。

无论是否使用默认的序列化格式，都必须像同步其他读取对象完整状态的方法一样强制同步对象的序列化过程。因此，如果有一个线程安全的对象（Item 82）通过同步每个方法来实现其线程安全，并且选择了使用默认序列化格式，应该使用以下`writeObject`方法：

		// writeObject for synchronized class with default serialized form
		private synchronized void writeObject(ObjectOutputStream s)
						throws IOException {
			s.defaultWriteObject();
		}

如果在`writeObject`方法中运用了同步，则必须确保它与其他活动使用了相同的锁定顺序约束，否则将面临资源排序死锁的风险。

无论选择哪种序列化格式，在编写的每个可序列化类中都应该显式声明系列版本UID，从而消除版本UID不兼容引发的问题（第86项）。而且还可以避免系统自动生成版本UID带来的性能损耗。使用以下代码声明UID：

		private static final long serialVersionUID = randomLongValue;

如果是编写一个新类，则为`randomLongValue`选择的值无关紧要。可以通过在类上运行`serialver`工具方法来生成值，但也可以凭空挑选任一数字。版本UID可以不是唯一的。如果修改缺少自定义版本UID的现有类，并且希望新版本和现有的序列化实例兼容，则必须使用旧版本自动生成的UID。可以通过在类的旧版本上运行`serialver`工具方法来获取旧版本类的UID编号。

如果在新版本类中更改了版本UID的值，那么在尝试反序列化旧版本的实例时会抛出`InvalidClassException`。

### 88. 防御性编写`readObject`方法 ###

`readObject`方法实际上相当于另一个公共构造函数，因此需要与任何其他构造函数一样小心。正如构造函数必须检查其参数的有效性（第49项）并在适当的地方制作参数的防御性复本（第50项），`readObject`方法也必须如此做。

笼统地讲，`readObject`是一个使用字节流作为唯一参数的构造函数。在正常使用中，字节流是通过序列化正常构造的实例生成的。但如果该字节流被人工构造以生成破坏该类不变量的对象时，就可能创建一个正常情况下不可能生成的对象。

要解决此问题，在`readObject`方法中调用`defaultReadObject`，然后检查反序列化对象的有效性。如果有效性检查失败，则`readObject`方法将抛出`InvalidObjectException`，从而阻止反序列化完成：

		// readObject method with validity checking - insufficient!
		private void readObject(ObjectInputStream s)
					throws IOException, ClassNotFoundException {
			s.defaultReadObject();
			// Check that our invariants are satisfied
			if (start.compareTo(end) > 0)
				throw new InvalidObjectException(start +" after "+ end);
		}

虽然这可以防止攻击者创建无效的实例，但仍然存在潜在的问题。例如，可以通过构造以有效实例开头的字节流来创建可变实例，然后在这个可变实例中提供和所包含有效实例中的私有变量相同类型的公共变量，再通过修改字节流中这些公共变量对应的字节码，使它们指向私有变量的引用，从而将这些引用指向的对象通过这些额外的公共变量暴露出来。如下所示：

		public class MutablePeriod {
			// A period instance
			public final Period period;
			// period's start field, to which we shouldn't have access
			public final Date start;
			// period's end field, to which we shouldn't have access
			public final Date end;
			
			public MutablePeriod() {
				try {
					ByteArrayOutputStream bos = new ByteArrayOutputStream();
					ObjectOutputStream out = new ObjectOutputStream(bos);
					// Serialize a valid Period instance
					out.writeObject(new Period(new Date(), new Date()));
					/*
					* Append rogue "previous object refs" for internal
					* Date fields in Period. For details, see "Java
					* Object Serialization Specification," Section 6.4.
					*/
					byte[] ref = { 0x71, 0, 0x7e, 0, 5 }; // Ref #5
					bos.write(ref); // The start field
					ref[4] = 4; // Ref # 4
					bos.write(ref); // The end field
					// Deserialize Period and "stolen" Date references
					ObjectInputStream in = new ObjectInputStream( 
									new ByteArrayInputStream(bos.toByteArray()));
					period = (Period) in.readObject();
					start = (Date) in.readObject();
					end = (Date) in.readObject();
				} catch (IOException | ClassNotFoundException e) {
					throw new AssertionError(e);
				}
			}
		}

运行以下程序发动攻击：

		public static void main(String[] args) {
			MutablePeriod mp = new MutablePeriod();
			Period p = mp.period;
			Date pEnd = mp.end;
			// Let's turn back the clock
			pEnd.setYear(78);
			System.out.println(p);
			// Bring back the 60s!
			pEnd.setYear(69);
			System.out.println(p);
		}

问题的根源是`Period`的`readObject`方法没有做足够的防御性复制。对象反序列化时，必须防御性地复制客户端不得拥有的引用类型的变量。因此，每个包含私有可变组件的可序列化不可变类必须在其`readObject`方法中防御性地复制这些组件。 以下`readObject`方法足以确保`Period`的不变量并保持其不变性：

		// readObject method with defensive copying and validity checking
		private void readObject(ObjectInputStream s)
						throws IOException, ClassNotFoundException {
			s.defaultReadObject();
			// Defensively copy our mutable components
			start = new Date(start.getTime());
			end = new Date(end.getTime());
			// Check that our invariants are satisfied
			if (start.compareTo(end) > 0)
				throw new InvalidObjectException(start +" after "+ end);
		}

注意复制防御性复本应在有效性检查之前执行，另外最终变量无法进行防御性复制。因此，在上例中，要使用`readObject`方法，必须使`start`和`end`变量为非最终变量

一个简单的用于判断默认的`readObject`方法是否适用于某个类的方法：是否愿意添加一个公共构造函数，该构造函数将对象中每个非transient变量的值作为参数，然后将值存储在变量中而不进行任何验证。如果没有，则必须提供`readObject`方法，并且必须执行构造函数所需的所有有效性检查和防御性复制。或者，可以使用序列化代理模式（项目90）。

`readObject`方法和构造函数都适用于非最终可序列化类。与构造函数一样，`readObject`方法中不能直接或间接调用可覆盖的方法（第19项）。如果违反此规则，重写了某个方法，则重写的方法会在子类的状态被反序列化之前运行，进而导致程序失败。

小结：编写readObject方法时要采取编写公共构造函数时一样的思路：不管给出了什么字节流，它都必须生成有效的实例。同时不要假设所给字节流一定是合法的序列化实例。以下是编写`readObject`方法的一些指南：

- 对于具有必须保持私有的引用类型变量的类，防御性地复制此变量中的每个对象。 如不可变类的可变组件。
- 检查所有不变量，并在检查失败时抛出`InvalidObjectException`。 检查应紧跟防御性复制。
- 如果在反序列化后必须验证整个对象图，请使用`ObjectInputValidation`接口（本书未讨论）。
- 不要直接或间接调用类中的任何可被覆盖的方法。

### 89. 使用枚举类型代替`readResolve`实现对生成实例数量的控制 ###

任何`readObject`方法，无论是显式方法还是默认方法，都会返回一个新创建的实例，该实例与在类初始化时创建的实例不同，因此在序列化单例类时要小心。

`readResolve`功能允许将另一个实例替换为`readObject`创建的实例。如果被反序列化的对象的类使用适当的声明定义了`readResolve`方法，则反序列化后新创建的对象会调用此方法。然后用此方法返回的对象来代替这个新创建的对象。在大多数情况中，不会保留对新创建的对象的引用，因此它立即可被垃圾回收。

如果Elvis类用于实现Serializable，则以下read-Resolve方法足以保证singleton属性：

// readResolve for instance control - you can do better!
private Object readResolve() {
    // Return the one true Elvis and let the garbage collector
    // take care of the Elvis impersonator.
    return INSTANCE;
}

此方法忽略了反序列化时生成的对象，直接返回在初始化类时创建的唯一Elvis实例。因此，Elvis类的序列化格式不需要包含任何实际数据，可以把所有实例字段声明为`transient`。实际上，如果依赖`readResolve`进行实例控制，则所有引用类型的实例变量都应该声明为`transient`。 否则，攻击者有可能在反序列化对象运行`readResolve`方法之前使用类似于第88项中的`MutablePeriod`的攻击技术。

以下是它详细的作案流程：首先，编写一个stealer类，它同时具有`readResolve`方法和一个实例变量，该变量指向窃取类所寄生的序列化的单例类。然后修改要反序列化的字节流，将单例类的非transient变量替换为窃取类的实例。现在出现了一个环路：单例类包含窃取类，窃取类中也有变量指向该单例类。

因为单例类包含窃取类，所以当单例类被反序列化时，窃取类的readResolve方法首先运行。而当窃取类的`readResolve`方法运行时，其实例变量仍指向那个已完成部分反序列化但还没有换身的单例类实例。

窃取类的`readResolve`方法将引用从这个实例变量复制到一个静态变量，以在`readResolve`方法运行后能通过该静态变量访问该单例类实例。然后，该方法返回一个和在单例类中它所替换的那个非transient变量相同类型的值。如果不这样做，当序列化系统尝试将窃取者类的引用存储到该变量时，两者的类型不一样，VM将抛出`ClassCastException`。换句话说，通过修改字节码流将单例类中的非transient变量的值强制改为窃取类的实例，再通过窃取类将单例类反序列化生成的实例半成品提取出来做修改，然后使用该窃取类的`readResolve`方法返回正确的类型，从而造成运行正常的假象。最后，单例类会执行自己的`readResolve`方法，但此时窃取类已经对单例类的反序列化对象作出了修改。最终得到的对象与实际对象可能是完全不同的。

		// Broken singleton - has nontransient object reference field!
		public class Elvis implements Serializable {
			public static final Elvis INSTANCE = new Elvis();
			private Elvis() { }
			private String[] favoriteSongs =
						{ "Hound Dog", "Heartbreak Hotel" };
			public void printFavorites() {
				System.out.println(Arrays.toString(favoriteSongs));
			}
			private Object readResolve() {
				return INSTANCE;
			}
		}

下面是一个窃取类：

		public class ElvisStealer implements Serializable {
			static Elvis impersonator;
			private Elvis payload;
			
			private Object readResolve() {
				// Save a reference to the "unresolved" Elvis instance
				impersonator = payload;
				// Return object of correct type for favoriteSongs field
				return new String[] { "A Fool Such as I" };
			}
			
			private static final long serialVersionUID = 0;
		}

最后这个程序将手工更改需要反序列化的流，以生成有缺陷的单例的两个不同实例。

		public class ElvisImpersonator {
			// Byte stream couldn't have come from a real Elvis instance!
			private static final byte[] serializedForm = {
			(byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x05,
			0x45, 0x6c, 0x76, 0x69, 0x73, (byte)0x84, (byte)0xe6,
			(byte)0x93, 0x33, (byte)0xc3, (byte)0xf4, (byte)0x8b,
			0x32, 0x02, 0x00, 0x01, 0x4c, 0x00, 0x0d, 0x66, 0x61, 0x76,
			0x6f, 0x72, 0x69, 0x74, 0x65, 0x53, 0x6f, 0x6e, 0x67, 0x73,
			0x74, 0x00, 0x12, 0x4c, 0x6a, 0x61, 0x76, 0x61, 0x2f, 0x6c,
			0x61, 0x6e, 0x67, 0x2f, 0x4f, 0x62, 0x6a, 0x65, 0x63, 0x74,
			0x3b, 0x78, 0x70, 0x73, 0x72, 0x00, 0x0c, 0x45, 0x6c, 0x76,
			0x69, 0x73, 0x53, 0x74, 0x65, 0x61, 0x6c, 0x65, 0x72, 0x00,
			0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x02, 0x00, 0x01,
			0x4c, 0x00, 0x07, 0x70, 0x61, 0x79, 0x6c, 0x6f, 0x61, 0x64,
			0x74, 0x00, 0x07, 0x4c, 0x45, 0x6c, 0x76, 0x69, 0x73, 0x3b,
			0x78, 0x70, 0x71, 0x00, 0x7e, 0x00, 0x02
			};
			public static void main(String[] args) {
				// Initializes ElvisStealer.impersonator and returns
				// the real Elvis (which is Elvis.INSTANCE)
				Elvis elvis = (Elvis) deserialize(serializedForm);
				Elvis impersonator = ElvisStealer.impersonator;
				elvis.printFavorites();
				impersonator.printFavorites();
			}
		}

最终输出结果如下，证明了可以创建两个不同的`Elvis`实例

		[Hound Dog, Heartbreak Hotel] [A Fool Such as I]

可以通过声明`favoriteSongs`变量为transient来解决这个问题，但最好是让Elvis成为只有一个值的枚举类型来修复它（第3项）。

如果将需要控制实例数量的可序列化类编写为枚举类型，Java会保证除了声明的枚举值之外不会有其他实例。以下是`Elvis`类作为枚举类型的示例： 

		// Enum singleton - the preferred approach
		public enum Elvis {
			INSTANCE;
			private String[] favoriteSongs =
					{ "Hound Dog", "Heartbreak Hotel" };
			
			public void printFavorites() {
				System.out.println(Arrays.toString(favoriteSongs));
			}
		}

如果一个需要控制实例数量的可序列化类的实例在编译时是未知的，那么就无法将该类表示为枚举类型，只能使用`readResolve`方法。

`readResolve`的可访问性非常重要。如果在最终类上放置readResolve方法，它应该是私有的。如果将`readResolve`方法放在非最终类上，则不同的访问性会产生不同效果：

- 如果它是私有的，则不适用于任何子类。
- 如果它是包私有的，它将仅适用于同一包中的子类。
- 如果它是受保护的或公开的，它将适用于所有不覆盖它的子类。但反序列化子类实例将生成一个父类实例，这可能会导致`ClassCastException`。

小结：尽量使用枚举类型来控制实例的生成。如果这个需要控制实例生成以及可序列化的类的实例不能在编译期间确定，则使用`readResolve`方法并确定类的所有实例变量都是基本数据类型或transient的。

### 90. 考虑使用序列化代理代替序列化的实例 ###

序列化代理模式：首先，设计一个私有静态内部类，它简洁地代表外部类实例的逻辑状态。这个内部类称为外部类的序列化代理。它应该只有一个构造函数，其参数类型是外部类。此构造函数仅仅复制其参数中的数据：它不需要执行任何一致性检查或防御性复制。序列化代理的默认序列化格式是外部类的完美序列化格式。外部类及其序列化代理都必须实现`Serializable`。

例如，在Item 50中编写的不可变`Period`类，在Item 88中进行了序列化。下面是该类的序列化代理。`Period`非常简单，其序列化代理与该字段具有完全相同的字段：

		// Serialization proxy for Period class
		private static class SerializationProxy implements Serializable {
			private final Date start;
			private final Date end;
			SerializationProxy(Period p) {
				this.start = p.start;
				this.end = p.end;
			}
			private static final long serialVersionUID =
						234098243823485285L; // Any number will do (Item 87)
		}

接下来，将以下`writeReplace`方法添加到外部类中。可以将此方法逐字复制到具有序列化代理的任何类中：

		// writeReplace method for the serialization proxy pattern
		private Object writeReplace() {
			return new SerializationProxy(this);
		}

外部类上存在此方法会导致序列化系统产生的是序列化代理类的实例而不是外部类的实例。换句话说，`writeReplace`方法在序列化之前将外部类的实例转换为其序列化代理类的实例。

使用此`writeReplace`方法，序列化系统将永远不会生成外部类的序列化实例，但攻击者可能会构造一个试图违反类不变量的实例。要确保此类攻击失败，只需将此`readObject`方法添加到外部类：

		// readObject method for the serialization proxy pattern
		private void readObject(ObjectInputStream stream)
					throws InvalidObjectException {
			throw new InvalidObjectException("Proxy required");
		}

最后，序列化代理类提供`readResolve`方法，该方法返回外部类的逻辑等效实例。此方法会在反序列化时将序列化代理类的实例转换回外部类的实例。

这个`readResolve`方法只使用它的外部类的公共API创建一个它的实例，这样做的好处是反序列化时生成的实例是使用常规的创建实例的方法，即通过构造器，静态工厂方法来生成。这样就不必需要单独确保反序列化生成的实例要满足类的不变量要保持一致的约束。如果类的静态工厂或构造函数负责生成这些不变量并且使用实例方法来维护它们，那么这些不变量也会在序列化过程中得到维护。

以下是`Period.SerializationProxy`的`readResolve`方法：

		// readResolve method for Period.SerializationProxy
		private Object readResolve() {
			return new Period(start, end); // Uses public constructor
		}

序列化代理方法允许`Period`的变量为`final`，从而让`Period`类成为真正的不可变类。序列化代理模式还允许反序列化生成的实例具有与序列化生成实例属于不同的类型。

比如`EnumSet`（第36项）。 这个类没有公共构造函数，只有静态工厂。从客户端的角度来看，它们返回的是`EnumSet`实例，但在当前的OpenJDK实现中，它们返回两个子类中的一个，具体取决于底层枚举类型的容量。如果底层枚举类型包含64个或更少的元素，则静态工厂返回`RegularEnumSet`; 否则，他们返回一个`JumboEnumSet`。

如果序列化时其底层枚举类型具有六十个元素，将生成`RegularEnumSet`实例，然后再向枚举类型添加五个元素，然后再反序列化先前生成的实例得到的结果不是`RegularEnumSet`实例，而是`JumboEnumSet`实例。

		// EnumSet's serialization proxy
		private static class SerializationProxy <E extends Enum<E>>
					implements Serializable {
			// The element type of this enum set.
			private final Class<E> elementType;
			// The elements contained in this enum set.
			private final Enum<?>[] elements;
			SerializationProxy(EnumSet<E> set) {
				elementType = set.elementType;
				elements = set.toArray(new Enum<?>[0]);
			}
			private Object readResolve() {
				EnumSet<E> result = EnumSet.noneOf(elementType); //noneof方法是返回不同类型EnumSet子类的关键
				for (Enum<?> e : elements)
					result.add((E)e);
				return result;
			}
			private static final long serialVersionUID = 362491234563181265L;
		}

序列化代理模式有两个限制。

- 与可扩展的类不兼容（第19项）。
- 与某些对象图中包含环路的类不兼容：如果尝试从其序列化代理类的`readResolve`方法中调用外部类对象上的方法，将抛出`ClassCastException`，因为该外部类的对象还不存在。

小结：如果必须在不需要由客户端扩展的类上编写`readObject`或`writeObject`方法，就考虑使用序列化代理模式。








































