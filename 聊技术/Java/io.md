https://zhuanlan.zhihu.com/p/355582245

前言：

​	群里有大佬说想让我写一篇NIO，一直也没写，但是和同事聊天也说对Java的IO不是很清晰，因此今天就写下Java的IO，先打个基础，下次写NIO，我们开始吧

### 1、IO 到底是咋回事

​	操作系统就是管家，电脑的设备就是资源，如果进程先要操作资源，必须要进行系统调用，有操作系统去处理，然后再返回给进程，这样的代理模式是不是很常见？因此app 就是你写的程序，资源就是硬盘或者其他的设备，io就是进行的系统调用。

![image-20210520221313172](D:\wechat\gameWathcer\img\20210519\1.png)

​	为了保证操作系统的稳定性和安全性，一个进程的地址空间划分为 **用户空间（User space）** 和 **内核空间（Kernel space ）** 。
像我们平常运行的应用程序都是运行在用户空间，只有内核空间才能进行系统态级别的资源有关的操作，比如如文件管理、进程通信、内存管理等等。也就是说，我们想要进行 IO 操作，一定是要依赖内核空间的能力。
并且，用户空间的程序不能直接访问内核空间。
当想要执行 IO 操作时，由于没有执行这些操作的权限，只能发起系统调用请求操作系统帮忙完成。
因此，用户进程想要执行 IO 操作的话，必须通过 **系统调用** 来间接访问内核空间

### 2、IO的类结构

​	java的io 实在太复杂了，往往新手很难掌握，因为只缘身在此山中，新手往往很难从全体去看到问题的本质，我和打铁的朋友的聊天截图能帮你解答一些。



![image-20210520223947161](D:\wechat\gameWathcer\img\20210519\2.png)



类结构如下



![image-20210520225851996](D:\wechat\gameWathcer\img\20210519\4.png)

在平常的读写文件的时候可以先用基本流，然后看是否需要字符流，最后在用上带buffer 的流。

IO流的设计思想就是装饰器模式，一层一层的进行升级功能。

### 3、罗列一下IO流的类型

![img](D:\wechat\gameWathcer\img\20210519\5.jpg)



### 4、来波实例

**1、访问操作文件（FileInputStream/FileReader ，FileOutputStream/FileWriter）**

```java

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;

/**
 * 拷贝文件
 * @author 香菜
 */
public class CopyFileWithStream {
    public static void main(String[] args) {
        int b = 0;
        String inFilePath = "D:\\wechat\\A.txt";
        String outFilePath = "D:\\wechat\\B.txt";
        try (FileInputStream in = new FileInputStream(inFilePath); FileOutputStream out = new FileOutputStream(outFilePath)) {
            while ((b = in.read()) != -1) {
                out.write(b);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        System.out.println("文件复制完成");
    }
}

```

**2、缓存流的使用（BufferedInputStream/BufferedOutputStream，BufferedReader/BufferedWriter）**

```java
package org.pdool.iodoc;

import java.io.*;

/**
 * 拷贝文件
 *
 * @author 香菜
 */
public class CopyFileWithBuffer {
    public static void main(String[] args) throws Exception {
        String inFilePath = "D:\\wechat\\A.txt";
        String outFilePath = "D:\\wechat\\B.txt";
        try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(inFilePath));
             BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(outFilePath))) {
            byte[] b = new byte[1024];
            int off = 0;
            while ((off = bis.read(b)) > 0) {
                bos.write(b, 0, off);
            }
        }
    }
}

```

**3、获取键盘输入**

```java

import java.util.Scanner;

public class TestScanner {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNextLine()){
            System.out.println(scanner.nextLine());
        }
    }
}
```

让我们看下源码是啥情况：

![image-20210520233139428](D:\wechat\gameWathcer\img\20210519\6.png)



### 总结：

- 而Reader/Writer则是用于操作字符，增加了字符编解码等功能，适用于类似从文件中读取或者写入文本信息。本质上计算机操作的都是字节，不管是网络通信还是文件读取，Reader/Writer相当于构建了应用逻辑和原始数据之间的桥梁。
- Buffered等带缓冲区的实现，**可以避免频繁的磁盘读写**，进而提高IO处理效率。
- 记住IO流的设计模式是装饰器模式，对流进行功能升级。
- stream，reader ,buffered 三个关键词记住







1、类结构

2、nio

3、scanner



BufferedReader，BufferedWriter 要比 FileReader 和 FileWriter高效

**减少 IO 次数。**
**IO 访问是个慢操作**
**减少 IO 次数就是提高效率**

BufferedReader，BufferedWriter 要比 FileReader 和 FileWriter高效

































- **区分同步或异步（synchronous/asynchronous**）。简单来说，同步是一种可靠的有序运行机制，当我们进行同步操作时，后续的任务是等待当前调用返回，才会进行下一步；而异步则相反，其他任务不需要等待当前调用返回，通常依靠事件、回调等机制来实现任务间次序关系。
- **区分阻塞与非阻塞（blocking/non-blocking）**。在进行阻塞操作时，当前线程会处于阻塞状态，无法从事其他任务，只有当条件就绪才能继续，比如ServerSocket新连接建立完毕，或数据读取、写入操作完成；而非阻塞则是不管IO操作是否结束，直接返回，相应操作在后台继续处理。



- 而Reader/Writer则是用于操作字符，增加了字符编解码等功能，适用于类似从文件中读取或者写入文本信息。本质上计算机操作的都是字节，不管是网络通信还是文件读取，Reader/Writer相当于构建了应用逻辑和原始数据之间的桥梁。
- BufferedOutputStream等带缓冲区的实现，**可以避免频繁的磁盘读写**，进而提高IO处理效率。这种设计利用了缓冲区，将批量数据进行一次操作，但在使用中千万别忘了flush。





## **内核**

内核是硬件和软件之间的一道桥梁、内核是个大管家、管理各种IO设备，进行程序调度、保存现场

内核保护模式：不能直接访问内核，但是提供了系统调用供应用程序使用，系统调用会从用户态切换到内核态，也会引起性能损耗



- 而NIO是面向缓冲区(它由ByteBuffer抽象类中的byte[] hb实现)的，也就是说数据先被读取到缓冲区，当缓冲区的数据量达到一定的大小后(比如BufferReader.readLine() )，一次写入虚拟机中，减少了CPU等待磁盘读写所浪费的时间。并且NIO的基本数据类型是基于块的，并且块与块之间是没有顺序的，这就保证了NIO的读写操作的可以同时进行。缺点就是数据处理的过程变得复杂，不容易理解。
- 注意，在Windows系统中换行是\r\n，在Linux系统中是\n





**从计算机结构的视角来看的话， I/O 描述了计算机系统与外部设备之间通信的过程。**
**我们再先从应用程序的角度来解读一下 I/O。**
根据大学里学到的操作系统相关的知识：为了保证操作系统的稳定性和安全性，一个进程的地址空间划分为 **用户空间（User space）** 和 **内核空间（Kernel space ）** 。
像我们平常运行的应用程序都是运行在用户空间，只有内核空间才能进行系统态级别的资源有关的操作，比如如文件管理、进程通信、内存管理等等。也就是说，我们想要进行 IO 操作，一定是要依赖内核空间的能力。
并且，用户空间的程序不能直接访问内核空间。
当想要执行 IO 操作时，由于没有执行这些操作的权限，只能发起系统调用请求操作系统帮忙完成。
因此，用户进程想要执行 IO 操作的话，必须通过 **系统调用** 来间接访问内核空间
我们在平常开发过程中接触最多的就是 **磁盘 IO（读写文件）** 和 **网络 IO（网络请求和相应）**。
**从应用程序的视角来看的话，我们的应用程序对操作系统的内核发起 IO 调用（系统调用），操作系统负责的内核执行具体的 IO 操作。也就是说，我们的应用程序实际上只是发起了 IO 操作的调用而已，具体 IO 的执行是由操作系统的内核来完成的。**
当应用程序发起 I/O 调用后，会经历两个步骤：

1. 内核等待 I/O 设备准备好数据
2. 内核将数据从内核空间拷贝到用户空间。





以前的流总是堵塞的，一个线程只要对它进行操作，其它操作就会被堵塞，也就相当于水管没有阀门，你伸手接水的时候，不管水到了没有，你就都只能耗在接水（流）上。

nio的Channel的加入，相当于增加了水龙头（有阀门），虽然一个时刻也只能接一个水管的水，但依赖轮换策略，在水量不大的时候，各个水管里流出来的水，都可以得到妥

善接纳，这个关键之处就是增加了一个接水工，也就是Selector，他负责协调，也就是看哪根水管有水了的话，在当前水管的水接到一定程度的时候，就切换一下：临时关上当前水龙头，试着打开另一个水龙头（看看有没有水）。

当其他人需要用水的时候，不是直接去接水，而是事前提了一个水桶给接水工，这个水桶就是Buffer。也就是，其他人虽然也可能要等，但不会在现场等，而是回家等，可以做其它事去，水接满了，接水工会通知他们。

这其实也是非常接近当前社会分工细化的现实，也是统分利用现有资源达到并发效果的一种很经济的手段，而不是动不动就来个并行处理，虽然那样是最简单的，但也是最浪费资源的方式。





- 首先，通过Selector.open()创建一个Selector，作为类似调度员的角色。
- 然后，创建一个ServerSocketChannel，并且向Selector注册，通过指定SelectionKey.OP_ACCEPT，告诉调度员，它关注的是新的连接请求。
    **注意**，为什么我们要明确配置非阻塞模式呢？这是因为阻塞模式下，注册操作是不允许的，会抛出IllegalBlockingModeException异常。
- Selector阻塞在select操作，当有Channel发生接入请求，就会被唤醒。
- 在sayHelloWorld方法中，通过SocketChannel和Buffer进行数据操作，在本例中是发送了一段字符串。



```java
public class NIOServer extends Thread {
    public void run() {
        try (Selector selector = Selector.open();
             ServerSocketChannel serverSocket = ServerSocketChannel.open();) {// 创建Selector和Channel
            serverSocket.bind(new InetSocketAddress(InetAddress.getLocalHost(), 8888));
            serverSocket.configureBlocking(false);
            // 注册到Selector，并说明关注点
            serverSocket.register(selector, SelectionKey.OP_ACCEPT);
            while (true) {
                selector.select();// 阻塞等待就绪的Channel，这是关键点之一
                Set<SelectionKey> selectedKeys = selector.selectedKeys();
                Iterator<SelectionKey> iter = selectedKeys.iterator();
                while (iter.hasNext()) {
                    SelectionKey key = iter.next();
                   // 生产系统中一般会额外进行就绪状态检查
                    sayHelloWorld((ServerSocketChannel) key.channel());
                    iter.remove();
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    private void sayHelloWorld(ServerSocketChannel server) throws IOException {
        try (SocketChannel client = server.accept();) {          
        	client.write(Charset.defaultCharset().encode("Hello world!"));
        }
    }
   // 省略了与前面类似的main
}
```







`DirectByteBuffer` 会分配 `Jvm 堆外`，不受 JVM 堆大小的限制，创建速度慢，读写快。`DirectByteBuffer` 内存在 Linux 中，属于进程的堆内。`DirectByteBuffer` 受 jvm 参数 `MaxDirectMemorySize` 的影响





所谓多路复用器，**本质上是将轮询从用户态转移到了内核态**

标准的IO基于字节流和字符流进行操作的，而NIO是基于通道（Channel）和缓冲区（Buffer）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中??



```java
 //输出流
    public static void outputStream() throws IOException{
        //创建一个File实例
        File file = new File("/home/wenhaibo/IOTest.txt");
        //FileOutputStream为文件输出流
        FileOutputStream out = new FileOutputStream(file);
        //将内容转换为字节码输出
        out.write("This is IOTest".getBytes());
        //强制输出内存中所有内容
        out.flush();
        //关闭输出流
        out.close();
    }
```



```java
    //输入流
    public static void inputStream() throws IOException{
        //创建一个File实例
        File file = new File("/home/wenhaibo/IOTest.txt");
        //FileInputStream为文件输入流
        FileInputStream in = new FileInputStream(file);
        byte[] b = new byte[1024];
        //将 byte.length 个字节的数据读入一个 byte 数组中
        int len =in.read(b);
        //将字节码转为字符串打印输出
        System.out.println(new String(b, 0, len));
        //关闭输入流
        in.close();
    }
```



```java
//字符流
    public static void outputStreamWriter() throws IOException{
        //创建一个File实例
        File file = new File("/home/wenhaibo/IOTest.txt");
        //FileWriter为文件输出流
        Writer out = new FileWriter(file);
        //直接输出字符
        out.write("This is IOTest");
        //强制输出内存中所有内容
        out.flush();
        //关闭输出流
        out.close();
    }
```

\##Java 中的 mmap

看源码我们发现 `open.map` 返回的也是 `DirectByteBuffer`，只是这个方法返回的 `DirectByteBuffer` 使用了不同的构造方法，它绑定了一个 `fd` 。当我们读写数据的时候是不会触发系统调用 read 和 write 的，也就是内存映射的好处。

```java
public class MMapDemo {
    public static void main(String[] args) throws URISyntaxException, IOException, InterruptedException {
        final URL resource = MMapDemo.class.getClassLoader().getResource("demo.txt");
        final Path path = Paths.get(resource.toURI());
        final FileChannel open = FileChannel.open(path, StandardOpenOption.READ);
        // 发起系统调用 mmap
        final MappedByteBuffer map = open.map(FileChannel.MapMode.READ_ONLY, 0, open.size());
        // 读取数据时，不会再出发调用 read,直接从自己的虚拟内存中即可拿数据
        final CharBuffer decode = StandardCharsets.UTF_8.decode(map);
        System.out.println(decode.toString());
        open.close();
        Thread.sleep(100000);
    }
}
```



