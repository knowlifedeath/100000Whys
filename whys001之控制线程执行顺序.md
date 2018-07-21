# whys001之控制线程执行顺序

## 问题：现在有T1、T2、T3三个线程，你怎样保证T2在T1执行完后执行，T3在T2执行完后执行
### 这个线程问题通常会在第一轮或电话面试阶段被问到，目的是检测你对”join”方法是否熟悉。这个多线程问题比较简单，可以用join方法实现

## 方法一
```java
package insight.whys.common.whys000001;

/**
 * Created by xdp on 2018/4/26.
 */
public class TestJoin2
{
    public static void main(String[] args)
    {
        final Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("t1");
            }            
        });
        
        final Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    //引用t1线程，等待t1线程执行完
                    t1.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("t2");
            }
        });
        
        Thread t3 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    //引用t2线程，等待t2线程执行完
                    t2.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("t3");
            }
        });
        
        t3.start();
        t2.start();
        t1.start();
    }
}
/*
Output:
t1
t2
t3
*/
```

## 方法二
```java
class MyThread extends Thread{
    public MyThread(String name){
        setName(name);
    }

    @Override
    public void run()
    {
        for (int i = 0; i < 5; i++)
        {
            System.out.println(Thread.currentThread().getName()+": "+i);
            try
            {
                Thread.sleep(100);
            }
            catch (InterruptedException e)
            {
                e.printStackTrace();
            }
        }
    }
}

public class TestJoin
{
    public static void main(String[] args)
    {
        Thread t1 = new MyThread("线程1");
        Thread t2 = new MyThread("线程2");
        Thread t3 = new MyThread("线程3");

        try
        {
            //t1先启动
            t1.start();
            t1.join();
            //t2
            t2.start();
            t2.join();
            //t3
            t3.start();
            t3.join();
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
    }
}

/*
Output:
线程1: 0
线程1: 1
线程1: 2
线程1: 3
线程1: 4
线程2: 0
线程2: 1
线程2: 2
线程2: 3
线程2: 4
线程3: 0
线程3: 1
线程3: 2
线程3: 3
线程3: 4
*/
```
#### 线程状态转换图
![](https://github.com/knowlifedeath/100000Whys/blob/master/resources/whys001_0.png)