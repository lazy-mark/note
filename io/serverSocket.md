# serverSocket

ServerSoket的构造方法：

- ServerSocket() throws IOException
- ServerSocket(int port) throws IOException
- ServerSocket(int port,int backlog) throws IOException
- ServerSocket(int port,int backlog, InetAddress bindAddr) throws IOException

参数port指定服务器要监听的端口，参数backlog指定客户连接请求队列的长度，参数bindAddr指定服务器要绑定的IP地址。如果端口为0，那么将由系统为服务器分配一个匿名端口，程序调用getLocalPort方法就能获知这个端口号。

匿名端口一般适用于服务器与客户端之间的临时通信，通信结束，就断开连接，并且ServerSocket占用的临时端口也被释放。FTP协议就使用了匿名端口。



默认构造方法的作用

```java
public ServerSocket() throws IOException {
	setImpl();
}
```

通过该方法创建的ServerSocket不与任何端口绑定，接下来还需要通过bind方法与特定端口绑定。这个默认构造方法的用途是，允许服务器在绑定到特定的端口之前，先设置ServerSocket的一些选项，因为一旦服务器与特定端口绑定，有些选项就不能再改变了。

```java
private void setImpl() {
  if (factory != null) {
      impl = factory.createSocketImpl();
      checkOldImpl();
  } else {
      // SocketImpl!
      impl = new SocksSocketImpl();
  }
  if (impl != null)
      impl.setServerSocket(this);
}
```



设定客户端连接请求队列的长度

当服务器进程运行时，可能会同时监听到多个客户端的连接请求，管理客户端连接请求的任务是由操作系统来完成的。操作系统把这些连接请求存储在一个先进先出的队列中。许多操作系统限定了队列的最大长度，一般为50。当队列中的连接请求达到了队列的最大容量时，服务器进程所在的主机会拒绝新的连接请求。只有当服务器进程通过ServerSocket的accept方法从队列中取出连接请求，使队列腾出空位时，队列才能继续加入新的连接请求。



ServerSocket构造方法的backlog参数用来显式设置连接请求队列的长度，它将覆盖操作系统限定的队列最大长度。在以下情况，仍然会采用操作系统限定的队列的最大长度：

- backlog参数的值大于操作系统限定的队列的最大长度
- backlog参数的值 <= 0
- 没有设置backlog参数



对于客户端进程，如果它发出的连接请求被加入到服务器的队列中，就意味着客户端与服务器的连接建立成功，客户进程从Socket构造方法中正常返回。如果客户端进程发出的连接请求被服务器拒绝，Socket构造方法就会抛出ConnectionException。



接收和关闭与客户端的连接

ServerSocket的accept方法从连接请求队列中取出一个客户端的连接请求，然后创建并返回与客户端连接的Socket对象。如果队列中没有连接请求，accept方法就会一直等待，直到接收到了连接请求才返回。接下来，服务器从Socket对象中获得输入流和输出流，就能与客户端交换数据。

当服务器正在进行发送数据的操作时，如果客户端断开了连接，那么服务器端就会抛出SocketException异常。这只是服务器与单个客户端通信中出现的异常，这种异常应该被捕获，使得服务器能继续与其他客户端通信。



关闭ServerSocket

ServerSocket的close方法使服务器释放占有的端口，并且断开与所有客户端的连接。当一个服务器程序运行结束时，即使没有执行close方法，操作系统也会释放这个服务器占有的端口。因此，服务器程序并不一定要在结束之前执行ServerSocket的close方法。



```java
public boolean isClosed() {
  synchronized(closeLock) {
      return closed;
  }
}
```

ServerSocket的isClosed方法判断ServerSocket是否关闭，只有执行了ServerSocket的close方法，isClosed方法才返回true。否则，即使ServerSocket还没有和特定端口绑定，isClosed方法也会返回false。



```java
public boolean isBound() {
    return bound || oldImpl;
}
```

ServerSocket的isBound方法判断ServerSocket是否已经与一个端口绑定，只要ServerSocket已经与一个端口绑定，即使它已经被关闭，isBound方法也会返回true。



如果需要确定一个ServerSocket已经与特定端口绑定，并且还没有被关闭，则可以采用以下方式：

```java
serverSocket.isBound() && !serverSocket.isClosed();
```



ServerSocket选项

SO_TIMEOUT：表示等待客户连接的超时时间，以毫秒为单位。如果SO_TIMEOUT的值为0，表示永远不会超时。

当服务器执行accept方法时，如果连接请求队列为空，服务器就会一直等待，直到接收到了客户连接才从accept方法返回。如果设定了超时时间，那么当服务器等待的时间超过了超时时间，就会抛出SocketTimeoutException，它是InterruptException的子类。



SO_REUSEADDR：表示是否允许重用服务器所绑定的地址。用于决定如果网络上仍然有数据向旧的ServerSocket传输数据，是否允许新的ServerSocket绑定到与旧的ServerSocket同样的端口上。该选项的默认值与操作系统有关，在某些操作系统中，允许重用端口，而在某些操作系统中不允许重用端口。

当ServerSocket关闭时，如果网络上还有发送到这个ServerSocket的数据，这个ServerSocket不会立刻释放本地端口，而是会等待一段时间，确保接收到了网络上发送过来的延迟数据，然后再释放端口。

许多服务器程序都使用固定的端口，当服务器程序关闭后，有可能它的端口还会被占用一段时间，如果此时立刻在同一个主机上重启服务程序，由于端口已经被占用，使得服务器程序无法绑定到该端口，服务器启动失败，并抛出BindException ：Address already in use。为了确保一个进程关闭了ServerSocket后，即使操作系统还没释放端口，同一个主机上的其他进程还可以立刻重用该端口，可以调用ServerSocket的setResuseAddress(true)方法。需要注意的是，调用该方法必须在还没有绑定到一个端口之前调用，否则无效。此外，两个共用同一个端口的进程必须都调用ServerSocket的setResuseAddress(true)方法，才能使得一个进程关闭ServerSocket后，另外一个进程的ServerSocket还能够立刻重用相同端口。



SO_RCVBUF：表示接收数据的缓冲区的大小，以字节为单位。一般来说，传输大的连续的数据块（HTTP或FTP数据传输）可以使用较大的缓冲区，还可以减少传输数据的次数，从而提高传输数据的效率。而对于交互式的通信（Telnet和网络游戏），则应该采用小的缓冲区，确保能及时把小批量的数据发送给对方。



```java
public void setPerformancePreferences(int connectionTime,
                                      int latency,
                                      int bandwidth)
{
    /* Not implemented yet */
}
```

设定连接时间、延迟和带宽的相对重要性。



多线程的服务器

许多实际应用要求服务器具有同时为多个客户端提供服务的能力，HTTP服务器就是最明细的例子。任何时刻，HTTP服务器都可能接受到大量的客户请求，每个客户都希望能快速得到HTTP服务器的响应。可以用并发性能来衡量一个服务器同时响应多个客户的能力。一个具有好的并发性能的服务器，必须符合两个条件：

1. 能够同时接收多个客户端连接
2. 对于每个客户端的连接，都会迅速给予响应



为每一个客户端分配一个工作线程，服务器的主线程负责接收客户的连接，每次接收一个客户连接，就会创建一个工作线程，由它负责与客户端的通信。

```java
public void service() throw Exception {
  while(true) {
    Socket socket = null;
    Thread worker = new Thread(new Handler(socket));
    worker.start();
  }
}
```

不足之处：

1. 服务器创建和销毁工作线程的开销很大。如果服务器需要与许多的客户端通信，并且与每个客户的通信的时间都很短，那么有可能服务器为客户端创建新线程的开销与实际与客户端通信的开销还要大。
2. 活动的线程也消耗系统资源。每个线程本身都会占用一定的内存（每个线程需要大约1M），如果同时有大量客户连接服务器，就必须创建大量工作线程，它们消耗了大量内存，可能会导致系统的内存空间不足。
3. 如果线程数目固定，并且每个线程都有很长的生命周期，那么线程切换也是固定的。不同操作系统有不同的切换周期，一般在20毫秒。这里说的线程切换是指在Java虚拟机，以及底层操作系统的调度下，线程之间转让CPU的使用权。如果频繁创建和销毁线程，那么将导致频繁地切换线程，因为一个线程被销毁后，必然要把CPU转让给另外一个已经就绪的线程，使该线程获得运行机会。在这种情况下，线程之间的切换不再遵循系统的固定切换周期，切换线程的开销甚至比创建及销毁线程的开销还大。



比较好的方案可采用线程池。



使用线程池的注意事项
虽然线程池能大大提高服务器的并发性能，但使用它也会存在一定风险。与所有多线程应用程序一样，用线程池构建的应用程序容易产生各种并发问题，如对共享资源的竞争和死锁。此外，如果线程池本身的实现不健壮，或者没有合理地使用线程池，还容易导致与线程池有关的死锁、系统资源不足和线程泄漏等问题。

1. 死锁
   任何多线程应用程序都有死锁风险。造成死锁的最简单的情形是，线程A持有对象X的锁，并且在等待对象Y的锁，而线程B持有对象Y的锁，并且在等待对象X的锁。线程A与线程B都不释放自己持有的锁，并且等待对方的锁，这就导致两个线程永远等待下去，死锁就这样产生了。
   虽然任何多线程程序都有死锁的风险，但线程池还会导致另外一种死锁。在这种情形下，假定线程池中的所有工作线程都在执行各自任务时被阻塞，它们都在等待某个任务A的执行结果。而任务A依然在工作队列中，由于没有空闲线程，使得任务A一直不能被执行。这使得线程池中的所有工作线程都永远阻塞下去，死锁就这样产生了。

2. 系统资源不足
   如果线程池中的线程数目非常多,这些线程会消耗包括内存和其他系统资源在内的大量资源，从而严重影响系统性能。

3. 并发错误
   线程池的工作队列依靠wait()和notify()方法来使工作线程及时取得任务，但这两个方法都难于使用。如果编码不正确，可能会丢失通知，导致工作线程一直保持空闲状态，无视工作队列中需要处理的任务。因此使用这些方法时，必须格外小心，即便是专家也可能在这方面出错。最好使用现有的、比较成熟的线程池。
   例如，直接使用java.util.concurrent包中的线程池类。 

4. 线程泄漏
   使用线程池的一个严重风险是线程泄漏。对于工作线程数目固定的线程池，如果工作线程在执行任务时抛出 RuntimeException 或Error，并且这些异常或错误没有被捕获，那么这个工作线程就会异常终止，使得线程池永久失去了一个工作线程。如果所有的工作线程都异常终止，线程池就最终变为空，没有任何可用的工作线程来处理任务。
   导致线程泄漏的另一种情形是，工作线程在执行一个任务时被阻塞，如等待用户的输入数据，但是由于用户一直不输入数据（可能是因为用户走开了），导致这个工作线程一直被阻塞。这样的工作线程名存实亡，它实际上不执行任何任务了。假如线程池中所有的工作线程都处于这样的阻塞状态，那么线程池就无法处理新加入的任务了。

5. 任务过载
   当工作队列中有大量排队等候执行的任务时，这些任务本身可能会消耗太多的系统资源而引起系统资源缺乏。
   综上所述，线程池可能会带来种种风险，为了尽可能避免它们，使用线程池时需要遵循以下原则。

（1）如果任务A在执行过程中需要同步等待任务B的执行结果，那么任务A不适合加入到线程池的工作队列中。如果把像任务A一样的需要等待其他任务执行结果的任务加入到工作队列中，可能会导致线程池的死锁。
（2）如果执行某个任务时可能会阻塞，并且是长时间的阻塞，则应该设定超时时间，避免工作线程永久的阻塞下去而导致线程泄漏。在服务器程序中，当线程等待客户连接，或者等待客户发送的数据时，都可能会阻塞。可以通过以下方式设定超时时间：
调用ServerSocket的setSoTimeout(int timeout)方法，设定等待客户连接的超时时间
对于每个与客户连接的Socket，调用该Socket的setSoTimeout(int timeout)方法，设定等待客户发送数据的超时时间
（3）了解任务的特点，分析任务是执行经常会阻塞的I/O操作，还是执行一直不会阻塞的运算操作。前者时断时续地占用CPU，而后者对CPU具有更高的利用率。预计完成任务大概需要多长时间？是短时间任务还是长时间任务？
根据任务的特点，对任务进行分类，然后把不同类型的任务分别加入到不同线程池的工作队列中，这样可以根据任务的特点，分别调整每个线程池。
（4）调整线程池的大小。线程池的最佳大小主要取决于系统的可用CPU的数目，以及工作队列中任务的特点。假如在一个具有 N 个CPU的系统上只有一个工作队列，并且其中全部是运算性质（不会阻塞）的任务，那么当线程池具有 N 或 N+1 个工作线程时，一般会获得最大的 CPU 利用率。   
（5）避免任务过载。服务器应根据系统的承载能力，限制客户并发连接的数目。当客户并发连接的数目超过了限制值，服务器可以拒绝连接请求，并友好地告知客户：服务器正忙，请稍后再试。

6. 关闭服务器
   强行终止服务器程序会导致服务器中正在执行的任务被突然中断,服务器除了在8000端口监听普通客户程序EchoClient的连接外，还会在8001端口监听管理程序AdminClient的连接。当服务器在8001端口接收到了AdminClient发送的"shutdown"命令时，EchoServer就会开始关闭服务器，它不会再接收任何新的 EchoClient进程的连接请求，对于那些已经接收但是还没有处理的客户连接，则会丢弃与该客户的通信任务，而不会把通信任务加入到线程池的工作队列中。另外，EchoServer会等到线程池把当前工作队列中的所有任务执行完，才结束程序。



http://www.xiegw.cn/14787.html