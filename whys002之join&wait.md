# whys002之join&wait

Thread.join的代码实现非常简单，如下：
```java
/**
 * Waits for this thread to die.
 *
 * <p> An invocation of this method behaves in exactly the same
 * way as the invocation
 *
 * <blockquote>
 * {@linkplain #join(long) join}{@code (0)}
 * </blockquote>
 *
 * @throws  InterruptedException
 *          if any thread has interrupted the current thread. The
 *          <i>interrupted status</i> of the current thread is
 *          cleared when this exception is thrown.
 */
public final void join() throws InterruptedException {
	join(0);
}

/**
 * Waits at most {@code millis} milliseconds for this thread to
 * die. A timeout of {@code 0} means to wait forever.
 *
 * <p> This implementation uses a loop of {@code this.wait} calls
 * conditioned on {@code this.isAlive}. As a thread terminates the
 * {@code this.notifyAll} method is invoked. It is recommended that
 * applications not use {@code wait}, {@code notify}, or
 * {@code notifyAll} on {@code Thread} instances.
 *
 * @param  millis
 *         the time to wait in milliseconds
 *
 * @throws  IllegalArgumentException
 *          if the value of {@code millis} is negative
 *
 * @throws  InterruptedException
 *          if any thread has interrupted the current thread. The
 *          <i>interrupted status</i> of the current thread is
 *          cleared when this exception is thrown.
 */
public final synchronized void join(long millis)
throws InterruptedException {
	long base = System.currentTimeMillis();
	long now = 0;

	if (millis < 0) {
		throw new IllegalArgumentException("timeout value is negative");
	}

	if (millis == 0) {
		while (isAlive()) {
			wait(0);
		}
	} else {
		while (isAlive()) {
			long delay = millis - now;
			if (delay <= 0) {
				break;
			}
			wait(delay);
			now = System.currentTimeMillis() - base;
		}
	}
}

/**
 * Tests if this thread is alive. A thread is alive if it has
 * been started and has not yet died.
 *
 * @return  <code>true</code> if this thread is alive;
 *          <code>false</code> otherwise.
 */
public final native boolean isAlive();

//...Object....
/**
 * Causes the current thread to wait until either another thread invokes the
 * {@link java.lang.Object#notify()} method or the
 * {@link java.lang.Object#notifyAll()} method for this object, or a
 * specified amount of time has elapsed.
 * <p>
 * The current thread must own this object's monitor.
 * <p>
 * This method causes the current thread (call it <var>T</var>) to
 * place itself in the wait set for this object and then to relinquish
 * any and all synchronization claims on this object. Thread <var>T</var>
 * becomes disabled for thread scheduling purposes and lies dormant
 * until one of four things happens:
 * <ul>
 * <li>Some other thread invokes the {@code notify} method for this
 * object and thread <var>T</var> happens to be arbitrarily chosen as
 * the thread to be awakened.
 * <li>Some other thread invokes the {@code notifyAll} method for this
 * object.
 * <li>Some other thread {@linkplain Thread#interrupt() interrupts}
 * thread <var>T</var>.
 * <li>The specified amount of real time has elapsed, more or less.  If
 * {@code timeout} is zero, however, then real time is not taken into
 * consideration and the thread simply waits until notified.
 * </ul>
 * The thread <var>T</var> is then removed from the wait set for this
 * object and re-enabled for thread scheduling. It then competes in the
 * usual manner with other threads for the right to synchronize on the
 * object; once it has gained control of the object, all its
 * synchronization claims on the object are restored to the status quo
 * ante - that is, to the situation as of the time that the {@code wait}
 * method was invoked. Thread <var>T</var> then returns from the
 * invocation of the {@code wait} method. Thus, on return from the
 * {@code wait} method, the synchronization state of the object and of
 * thread {@code T} is exactly as it was when the {@code wait} method
 * was invoked.
 * <p>
 * A thread can also wake up without being notified, interrupted, or
 * timing out, a so-called <i>spurious wakeup</i>.  While this will rarely
 * occur in practice, applications must guard against it by testing for
 * the condition that should have caused the thread to be awakened, and
 * continuing to wait if the condition is not satisfied.  In other words,
 * waits should always occur in loops, like this one:
 * <pre>
 *     synchronized (obj) {
 *         while (&lt;condition does not hold&gt;)
 *             obj.wait(timeout);
 *         ... // Perform action appropriate to condition
 *     }
 * </pre>
 * (For more information on this topic, see Section 3.2.3 in Doug Lea's
 * "Concurrent Programming in Java (Second Edition)" (Addison-Wesley,
 * 2000), or Item 50 in Joshua Bloch's "Effective Java Programming
 * Language Guide" (Addison-Wesley, 2001).
 *
 * <p>If the current thread is {@linkplain java.lang.Thread#interrupt()
 * interrupted} by any thread before or while it is waiting, then an
 * {@code InterruptedException} is thrown.  This exception is not
 * thrown until the lock status of this object has been restored as
 * described above.
 *
 * <p>
 * Note that the {@code wait} method, as it places the current thread
 * into the wait set for this object, unlocks only this object; any
 * other objects on which the current thread may be synchronized remain
 * locked while the thread waits.
 * <p>
 * This method should only be called by a thread that is the owner
 * of this object's monitor. See the {@code notify} method for a
 * description of the ways in which a thread can become the owner of
 * a monitor.
 *
 * @param      timeout   the maximum time to wait in milliseconds.
 * @throws  IllegalArgumentException      if the value of timeout is
 *               negative.
 * @throws  IllegalMonitorStateException  if the current thread is not
 *               the owner of the object's monitor.
 * @throws  InterruptedException if any thread interrupted the
 *             current thread before or while the current thread
 *             was waiting for a notification.  The <i>interrupted
 *             status</i> of the current thread is cleared when
 *             this exception is thrown.
 * @see        java.lang.Object#notify()
 * @see        java.lang.Object#notifyAll()
 */
public final native void wait(long timeout) throws InterruptedException;

/**
 * Causes the current thread to wait until another thread invokes the
 * {@link java.lang.Object#notify()} method or the
 * {@link java.lang.Object#notifyAll()} method for this object, or
 * some other thread interrupts the current thread, or a certain
 * amount of real time has elapsed.
 * <p>
 * This method is similar to the {@code wait} method of one
 * argument, but it allows finer control over the amount of time to
 * wait for a notification before giving up. The amount of real time,
 * measured in nanoseconds, is given by:
 * <blockquote>
 * <pre>
 * 1000000*timeout+nanos</pre></blockquote>
 * <p>
 * In all other respects, this method does the same thing as the
 * method {@link #wait(long)} of one argument. In particular,
 * {@code wait(0, 0)} means the same thing as {@code wait(0)}.
 * <p>
 * The current thread must own this object's monitor. The thread
 * releases ownership of this monitor and waits until either of the
 * following two conditions has occurred:
 * <ul>
 * <li>Another thread notifies threads waiting on this object's monitor
 *     to wake up either through a call to the {@code notify} method
 *     or the {@code notifyAll} method.
 * <li>The timeout period, specified by {@code timeout}
 *     milliseconds plus {@code nanos} nanoseconds arguments, has
 *     elapsed.
 * </ul>
 * <p>
 * The thread then waits until it can re-obtain ownership of the
 * monitor and resumes execution.
 * <p>
 * As in the one argument version, interrupts and spurious wakeups are
 * possible, and this method should always be used in a loop:
 * <pre>
 *     synchronized (obj) {
 *         while (&lt;condition does not hold&gt;)
 *             obj.wait(timeout, nanos);
 *         ... // Perform action appropriate to condition
 *     }
 * </pre>
 * This method should only be called by a thread that is the owner
 * of this object's monitor. See the {@code notify} method for a
 * description of the ways in which a thread can become the owner of
 * a monitor.
 *
 * @param      timeout   the maximum time to wait in milliseconds.
 * @param      nanos      additional time, in nanoseconds range
 *                       0-999999.
 * @throws  IllegalArgumentException      if the value of timeout is
 *                      negative or the value of nanos is
 *                      not in the range 0-999999.
 * @throws  IllegalMonitorStateException  if the current thread is not
 *               the owner of this object's monitor.
 * @throws  InterruptedException if any thread interrupted the
 *             current thread before or while the current thread
 *             was waiting for a notification.  The <i>interrupted
 *             status</i> of the current thread is cleared when
 *             this exception is thrown.
 */
public final void wait(long timeout, int nanos) throws InterruptedException {
	if (timeout < 0) {
		throw new IllegalArgumentException("timeout value is negative");
	}

	if (nanos < 0 || nanos > 999999) {
		throw new IllegalArgumentException(
							"nanosecond timeout value out of range");
	}

	if (nanos > 0) {
		timeout++;
	}

	wait(timeout);
}

/**
 * Causes the current thread to wait until another thread invokes the
 * {@link java.lang.Object#notify()} method or the
 * {@link java.lang.Object#notifyAll()} method for this object.
 * In other words, this method behaves exactly as if it simply
 * performs the call {@code wait(0)}.
 * <p>
 * The current thread must own this object's monitor. The thread
 * releases ownership of this monitor and waits until another thread
 * notifies threads waiting on this object's monitor to wake up
 * either through a call to the {@code notify} method or the
 * {@code notifyAll} method. The thread then waits until it can
 * re-obtain ownership of the monitor and resumes execution.
 * <p>
 * As in the one argument version, interrupts and spurious wakeups are
 * possible, and this method should always be used in a loop:
 * <pre>
 *     synchronized (obj) {
 *         while (&lt;condition does not hold&gt;)
 *             obj.wait();
 *         ... // Perform action appropriate to condition
 *     }
 * </pre>
 * This method should only be called by a thread that is the owner
 * of this object's monitor. See the {@code notify} method for a
 * description of the ways in which a thread can become the owner of
 * a monitor.
 *
 * @throws  IllegalMonitorStateException  if the current thread is not
 *               the owner of the object's monitor.
 * @throws  InterruptedException if any thread interrupted the
 *             current thread before or while the current thread
 *             was waiting for a notification.  The <i>interrupted
 *             status</i> of the current thread is cleared when
 *             this exception is thrown.
 * @see        java.lang.Object#notify()
 * @see        java.lang.Object#notifyAll()
 */
public final void wait() throws InterruptedException {
	wait(0);
}
```
## join解惑
join: 线程合并, 把指定的线程加入到当前线程, 可以将两个交并行执行的线程合并为串行执行的线程.

#### 参考: [whys327之运算符短路](https://github.com/knowlifedeath/100000Whys/blob/master/whys001%E4%B9%8B%E6%8E%A7%E5%88%B6%E7%BA%BF%E7%A8%8B%E6%89%A7%E8%A1%8C%E9%A1%BA%E5%BA%8F.md)

在方法一中，t2 run方法体中调用t1.join, 则t1加到当前线程t2前，t2在t1执行完才会被执行, 达到顺序控制的目的

在方法二中, t1.join, 则t1加到当前线程 main线程前, main线程在t1执行完才会被执行, 这样下面的t2 t3就不会被start, 到达控制顺序的目的

#### 参照源码
可知, 当在线程A中调用线程B份join方法时，会使线程A调用自己的wait方法从而进行阻塞线程A，以达到让线程B执行的目的

main线程中调用t1.join, 则main线程调用自己的wait方法导致阻塞让出cpu, 然后t1先执行完.




