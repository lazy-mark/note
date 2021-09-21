# 同步工具tools

## Semaphore—信号量

在计算机中被解释为“信号量”，Semaphore是一种计数信号量，用于管理一组资源，给资源的使用者规定一个量，从而控制同一时刻的使用者数量。



Semaphore经常用于限制同一时刻获取某种资源的线程数量，最典型的就是做流量控制。举个生活的例子，公交车抢座位，车上有2个座位，此时上车40人，从上车到下车的时间段表示业务处理的时间，业务处理完用户就下车。

```java
public class SemaphoreTest {

    private static final Semaphore semaphore = new Semaphore(2);

    public static void main(String[] args) {
        for (int i = 0; i < 40; i++) {
            new Thread(new User(semaphore, i),"线程" + i).start();
        }
    }

    private static class User implements Runnable {
        private Semaphore semaphore;
        private String name;
        public User(Semaphore semaphore, int id) {
            this.semaphore = semaphore;
            this.name = "用户" + id;
        }

        @Override
        public void run() {
            try {
              	semaphore.acquire();
                System.out.println(Thread.currentThread().getName() + this.name + "抢到座位了");
              	// 座位上坐了2s
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                System.out.println(this.name + "已经下车了");
                semaphore.release();
            }
        }
    }
}
```



```reStructuredText
线程0用户0抢到座位了
线程1用户1抢到座位了
用户0已经下车了
用户1已经下车了
线程2用户2抢到座位了
线程3用户3抢到座位了
用户2已经下车了
用户3已经下车了
线程4用户4抢到座位了
线程5用户5抢到座位了
用户4已经下车了
用户5已经下车了
线程6用户6抢到座位了
线程7用户7抢到座位了
用户6已经下车了
用户7已经下车了
线程8用户8抢到座位了
线程9用户9抢到座位了
用户8已经下车了
用户9已经下车了
线程10用户10抢到座位了
线程11用户11抢到座位了
用户10已经下车了
用户11已经下车了
线程12用户12抢到座位了
线程13用户13抢到座位了
用户12已经下车了
用户13已经下车了
线程14用户14抢到座位了
线程15用户15抢到座位了
用户15已经下车了
用户14已经下车了
线程16用户16抢到座位了
线程17用户17抢到座位了
用户16已经下车了
用户17已经下车了
线程18用户18抢到座位了
线程19用户19抢到座位了
用户19已经下车了
用户18已经下车了
线程20用户20抢到座位了
线程21用户21抢到座位了
用户20已经下车了
用户21已经下车了
线程22用户22抢到座位了
线程23用户23抢到座位了
用户23已经下车了
用户22已经下车了
线程24用户24抢到座位了
线程25用户25抢到座位了
用户24已经下车了
用户25已经下车了
线程26用户26抢到座位了
线程27用户27抢到座位了
用户27已经下车了
用户26已经下车了
线程28用户28抢到座位了
线程29用户29抢到座位了
用户28已经下车了
用户29已经下车了
线程30用户30抢到座位了
线程31用户31抢到座位了
用户30已经下车了
用户31已经下车了
线程32用户32抢到座位了
线程33用户33抢到座位了
用户33已经下车了
用户32已经下车了
线程34用户34抢到座位了
线程35用户35抢到座位了
用户35已经下车了
用户34已经下车了
线程36用户36抢到座位了
线程37用户37抢到座位了
用户36已经下车了
用户37已经下车了
线程38用户38抢到座位了
线程39用户39抢到座位了
用户38已经下车了
用户39已经下车了
```

观察，同一时刻最多只能有2个用户占座，其他用户需要等待用户让出座位才能占座，因为acquire方法是阻塞式的。



Semaphore底层本质上使用的是AbstractQueuedSynchronizer实现，默认是非公平竞争，通过构造方法传参可改变公平性；



1. availablePermits：返回当前信号量还可用的许可数，即还允许多少个线程进行使用资源，上面的例子也就是返回当前时刻还有多少个座位空余。
2. hasQueuedThreads：返回是否有线程正在等待获取资源；
3. acquire(int permits)：申请指定数目的信号量许可，在获取不到指定数目的许可时将一直阻塞，也就是一个人比较霸道，需要占用permits个座位；
4. tryAcquire()：尝试申请信号量许可，无论是否申请成功都返回申请结果。成功时返回true，否则返回false，程序根据返回结果补充逻辑。和acquire主要区别在于，不会阻塞；



Semaphore和sync的比较：

- sync真正用于并发控制，确保对某一个资源的串行访问；
- Semaphore限制访问资源的线程数，并没有实现同步，当Semaphore限制的资源同时只允许一个线程访问时，两者等价；



Semaphore注意点：当acquire申请数permits大于release数时，程序将会被阻塞。



总结：sync只能控制一个并发，而Semaphore能控制多个并发；底层通过AQS实现；



## CountDownLatch—同步计数器

CountDownLatch内部使用了一个计数器进行实现，计数器初始值为线程的数量，当每一个线程完成自己的任务后，计数器的值就会减1，当计数器的值为0时，表示所有的线程都已经完成了任务，在CountDownLatch上等待的线程就可以恢复继续执行后续的任务。



```java
public class CountDownLatchTests {

    final static int threadCount = 5;
    static CountDownLatch downLatch = new CountDownLatch(threadCount);

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < threadCount; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(new Random().nextInt(5000));
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + "已经处理完毕");
                    downLatch.countDown();
                }
            }, "线程" + i).start();
        }
        // 主线程等待所有子线程运行完
        downLatch.await();
        System.out.println("计算完毕...");
    }

}
```

CountDownLatch主要体现的是多个线程共同完成一件事，await方法也是阻塞式的。



1. getCount()：获取当前计数器的数值，了解处理进度；



## CyclicBarrier—循环栅栏

CyclicBarrier 工具类允许一组线程相互等待，直到所有线程都到达一个公共的屏障点，然后这些线程一起继续执行后继逻辑。之所以称之为 “循环”，是因为在所有线程都释放了对这个屏障的使用后，这个屏障还可以重新使用。



CyclicBarrier适合一个由多个线程共同协作完成任务的场合；来看个例子，学校组织春游，准备回家吃饭，班级租了一辆车，老师规定所有同学到齐了车才后出发；

```java
public class CyclicBarrierTests {

    private static Runnable runnable = new Runnable() {
        @Override
        public void run() {
            System.out.println("集合完毕，开启启动车辆进行出发");
        }
    };

    private static final int threadCount = 50;
    private static final CyclicBarrier CYCLIC_BARRIER = new CyclicBarrier(threadCount, runnable);
    public static void main(String[] args) {
        for (int i = 0; i < threadCount; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread().getName() + "开始回集合地点...");
                    try {
                        Thread.sleep(1000);
                        System.out.println(Thread.currentThread().getName() + "已经到达集合地点");
                        CYCLIC_BARRIER.await();
                    } catch (InterruptedException | BrokenBarrierException e) {
                        e.printStackTrace();
                    }
                }
            }, i + "号同学").start();
        }
    }

}
```



```reStructuredText
0号同学已经开始出门...
3号同学已经开始出门...
2号同学已经开始出门...
1号同学已经开始出门...
6号同学已经开始出门...
5号同学已经开始出门...
7号同学已经开始出门...
4号同学已经开始出门...
8号同学已经开始出门...
9号同学已经开始出门...
10号同学已经开始出门...
11号同学已经开始出门...
12号同学已经开始出门...
13号同学已经开始出门...
14号同学已经开始出门...
15号同学已经开始出门...
16号同学已经开始出门...
17号同学已经开始出门...
18号同学已经开始出门...
19号同学已经开始出门...
20号同学已经开始出门...
21号同学已经开始出门...
22号同学已经开始出门...
23号同学已经开始出门...
24号同学已经开始出门...
25号同学已经开始出门...
26号同学已经开始出门...
27号同学已经开始出门...
28号同学已经开始出门...
29号同学已经开始出门...
30号同学已经开始出门...
31号同学已经开始出门...
32号同学已经开始出门...
33号同学已经开始出门...
34号同学已经开始出门...
35号同学已经开始出门...
36号同学已经开始出门...
37号同学已经开始出门...
38号同学已经开始出门...
39号同学已经开始出门...
40号同学已经开始出门...
41号同学已经开始出门...
42号同学已经开始出门...
43号同学已经开始出门...
44号同学已经开始出门...
45号同学已经开始出门...
46号同学已经开始出门...
47号同学已经开始出门...
48号同学已经开始出门...
49号同学已经开始出门...
2号同学已经打达集合地点
48号同学已经打达集合地点
17号同学已经打达集合地点
42号同学已经打达集合地点
4号同学已经打达集合地点
22号同学已经打达集合地点
7号同学已经打达集合地点
44号同学已经打达集合地点
38号同学已经打达集合地点
16号同学已经打达集合地点
41号同学已经打达集合地点
1号同学已经打达集合地点
3号同学已经打达集合地点
40号同学已经打达集合地点
20号同学已经打达集合地点
34号同学已经打达集合地点
25号同学已经打达集合地点
33号同学已经打达集合地点
36号同学已经打达集合地点
15号同学已经打达集合地点
49号同学已经打达集合地点
45号同学已经打达集合地点
46号同学已经打达集合地点
11号同学已经打达集合地点
28号同学已经打达集合地点
9号同学已经打达集合地点
6号同学已经打达集合地点
26号同学已经打达集合地点
30号同学已经打达集合地点
10号同学已经打达集合地点
12号同学已经打达集合地点
21号同学已经打达集合地点
32号同学已经打达集合地点
24号同学已经打达集合地点
19号同学已经打达集合地点
35号同学已经打达集合地点
18号同学已经打达集合地点
13号同学已经打达集合地点
39号同学已经打达集合地点
5号同学已经打达集合地点
23号同学已经打达集合地点
14号同学已经打达集合地点
0号同学已经打达集合地点
47号同学已经打达集合地点
31号同学已经打达集合地点
43号同学已经打达集合地点
8号同学已经打达集合地点
37号同学已经打达集合地点
27号同学已经打达集合地点
29号同学已经打达集合地点
集合完毕，开启启动车辆进行出发
```

例子可能也存在问题，因为某个同学可能早就打到集合地点，而有些同学还在集合地点的途中。CyclicBarrier只能保证所有同学到达了集合地点才能通知老师。

