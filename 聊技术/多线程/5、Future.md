## 《多线程系列二》不理解future怎么能有future？

今天说下future，Future是一个interface，可以方便的用于异步结果的获取。 

每个游戏都有世界BOSS的功能，因为世界boss 是涉及所有的玩家的，为了控制多线程的复杂度，会启动一个单线程的线程池控制整个流程，但是玩家的消息线程又需要获得当前世界boss 的信息，所以就有需求从单线程中获取boss信息。怎么做呐？future就可以解决这个问题。下面带你一起理解future和使用future。

1、Future的类图结构

首先看下future接口的函数。

get（） 获取执行的结果，另外一个重载是有时间限制的get ，如果超时会有异常

isDone()  判断future 结果是否处理完成

![image-20210311213026594](../../\img\20210311\2.png)

![img](../../\img\20210311\3.png)

2、future的使用

```
package thread;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

/**
 * @author 香菜
 */
public class FutureTest {
    private ExecutorService executor = Executors.newSingleThreadExecutor();

    public Future<Integer> calculate(Integer input) {
        return executor.submit(() -> {
            System.out.println("Calculating..." + input);
            Thread.sleep(1000);
            return input * input;
        });
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Future<Integer> calculate = new FutureTest().calculate(100);
        System.out.println(calculate.get());
        System.out.println("Done");
    }
}

```

3、通俗理解

 future 就像是去买手抓饼，你把钱给老板之后，老板对你我做好了之后会放在旁边的盘子里，而这个盘子就是future，你用isDone  判断盘子里是不是有你要的手抓饼。当然你可以一直在那等着 get(),或者去做其他的事情，等会再来拿。

4、原理

看下

```
public V get() throws InterruptedException, ExecutionException {
        int s = state;
        if (s <= COMPLETING)
            s = awaitDone(false, 0L);
        return report(s);
    }
    private int awaitDone(boolean timed, long nanos)
        throws InterruptedException {
        final long deadline = timed ? System.nanoTime() + nanos : 0L;
        WaitNode q = null;
        boolean queued = false;
        for (;;) {
            if (Thread.interrupted()) {
                removeWaiter(q);
                throw new InterruptedException();
            }

            int s = state;
            if (s > COMPLETING) {
                if (q != null)
                    q.thread = null;
                return s;
            }
            else if (s == COMPLETING) // cannot time out yet
                Thread.yield();
            else if (q == null)
                q = new WaitNode();
            else if (!queued)
                queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                     q.next = waiters, q);
            else if (timed) {
                nanos = deadline - System.nanoTime();
                if (nanos <= 0L) {
                    removeWaiter(q);
                    return state;
                }
                LockSupport.parkNanos(this, nanos);
            }
            else
                LockSupport.park(this);
        }
    }
```

看下上面的代码就是在获取结果的时候，会先判断状态是否完成，如果完成了就正常返回结果，如果没完成就会调用awaitDone，看名字也能看出来就是等待直到完成，进入代码可以看到就是将进入死循环检查状态，线程阻塞等待，直到完成。

6、总结

future理解起来很简单，就是给你个号，等下好了，我会把数据放进去，如果你愿意等就等着，不愿等就等会来拿，就是一种延时获取结果的方式。理解了很简单。下次写什么还没定，总之最近会一直写多线程东西，做个总结。

打字不容易，点赞，转发，关注三连，谢谢大家，对了，关注我公众号：【香菜聊游戏】有更多福利哦。常规福利双手送上。