## CountDownLatch和AtomicInteger示例

[Java并发_CAS原理详细分析 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/35158220)

```java
import java.util.Collections;
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicInteger;

class Data {
    AtomicInteger num;

    public Data(int num) {
        this.num = new AtomicInteger(num);
    }

    public int getAndDecrement() {
        return num.getAndDecrement();
    }
}

class MyRun implements Runnable {

    private Data data;
    // 用来记录所有卖出票的编号
    private List<Integer> list;
    private CountDownLatch latch;

    public MyRun(Data data, List<Integer> list, CountDownLatch latch) {
        this.data = data;
        this.list = list;
        this.latch = latch;
    }

    @Override
    public void run() {
        try {
            action();
        }  finally {
            // 释放latch共享锁
            latch.countDown();
        }
    }

    // 进行买票操作，注意这里没有使用data.num>0作为判断条件，直到卖完线程退出。
    // 那么做会导致这两处使用了共享变量data.num，那么做多线程同步时，就要考虑更多条件。
    // 这里只for循环了5次，表示每个线程只卖5张票，并将所有卖出去编号存入list集合中。
    public void action() {
        for (int i = 0; i < 5; i++) {
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            int newNum = data.getAndDecrement();

            System.out.println("线程"+Thread.currentThread().getName()+"  num=="+newNum);
            list.add(newNum);
        }
    }
}

public class ThreadTest {


    public static void startThread(Data data, String name, List<Integer> list,CountDownLatch latch) {
        Thread t = new Thread(new MyRun(data, list, latch), name);
        t.start();
    }

    public static void main(String[] args) {
        // 使用CountDownLatch来让主线程等待子线程都执行完毕时，才结束
        CountDownLatch latch = new CountDownLatch(6);

        long start = System.currentTimeMillis();
        // 这里用并发list集合
        List<Integer> list = new CopyOnWriteArrayList();
        Data data = new Data(30);
        startThread(data, "t1", list, latch);
        startThread(data, "t2", list, latch);
        startThread(data, "t3", list, latch);
        startThread(data, "t4", list, latch);
        startThread(data, "t5", list, latch);
        startThread(data, "t6", list, latch);


        try {
            latch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // 处理一下list集合，进行排序和翻转
        Collections.sort(list);
        Collections.reverse(list);
        System.out.println(list);

        long time = System.currentTimeMillis() - start;
        // 输出一共花费的时间
        System.out.println("\n主线程结束 time=="+time);
    }
}
```