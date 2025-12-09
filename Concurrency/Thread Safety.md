
A class is thread-safe if it behaves correctly when accessed from multiple threads, regardless of the scheduling or intereleaving of the execution of those threads by the runtime environmentm and with no additional synchronization or other coordination on the part of the calling code

Stateless servlet:
```java
@ThreadSafe
public class StatelessFactorizer implements Servlet {
	public void service(ServletRequest req, ServletResponse resp) {
	BigInteger i = extractFromRequest(req);
	BigInteger[] factors = factor(i);
	encodeIntoResponse(resp, factors);
	}
}
```

StatelessFactorizer is, like most servlets, stateless: it has no fields and references no fields from other classes. The transient state for a particular computation exists solely in local variables that are stored on the *thread's stack* and are accessible only to the executing thread

```java
@NotThreadSafe
public class UnsafeCountingFactorizer implements Servlet {
	private long count = 0;
	public long getCount() { return count; }
	public void service(ServletRequest req, ServletResponse resp) {
	BigInteger i = extractFromRequest(req);
	BigInteger[] factors = factor(i);
	++count;
	encodeIntoResponse(resp, factors);
	}
}
```

If the counter is initially 9, with some unlucky timing each thread could read the value, see that it is 9, add one to it, and each set the counter to 10. This is clearly not what is supposed to happen; an increment got lost along the way, and the hit counter is now permanently off by one.

### Race Conditions

A race condition occurs when the correctness of a computation depends on the relative timing or interleaving of multiple threads by the runtime

**Race conditions in Lazy initialization**
```java
@NotThreadSafe
public class LazyInitRace {
	private ExpensiveObject instance = null;
	public ExpensiveObject getInstance() {
	if (instance == null)
		instance = new ExpensiveObject();
	return instance;
	}
}
```

Say that threads A and B execute getInstance at the same time. A sees that instance is null, and instantiates a new ExpensiveObject. B also checks if instance is null. Whether instance is null at this point depends unpredictably on timing, including the vagaries of scheduling and how long A takes to instantiate the ExpensiveObject and set the instance field.

**Compound Actions**

Operations A and B are atomic with respect to each other if, from the perspective of a thread executing A, when another thread executes B, either all of B has executed or none of it has. An atomic operation is one that is atomic with respect to all operations, including itself, that operate on the same state.

```java
@ThreadSafe
public class CountingFactorizer implements Servlet {
	private final AtomicLong count = new AtomicLong(0);
	public long getCount() { return count.get(); }
	public void service(ServletRequest req, ServletResponse resp) {
	BigInteger i = extractFromRequest(req);
	BigInteger[] factors = factor(i);
	count.incrementAndGet();
	encodeIntoResponse(resp, factors);
	}
}
```

### Locking

```java
@NotThreadSafe
public class UnsafeCachingFactorizer implements Servlet {
	private final AtomicReference<BigInteger> lastNumber
	= new AtomicReference<BigInteger>();
	private final AtomicReference<BigInteger[]> lastFactors
	= new AtomicReference<BigInteger[]>();
	public void service(ServletRequest req, ServletResponse resp) {
		BigInteger i = extractFromRequest(req);
		if (i.equals(lastNumber.get()))
		encodeIntoResponse(resp, lastFactors.get() );
		else {
			BigInteger[] factors = factor(i);
			lastNumber.set(i);
			lastFactors.set(factors);
			encodeIntoResponse(resp, factors);
		}
	}
}
```

*one thread's write of a reference is visible as a whole to other threads*

### Intrinsic Locks

```java
@ThreadSafe
public class SynchronizedFactorizer implements Servlet {
	@GuardedBy("this") private BigInteger lastNumber;
	@GuardedBy("this") private BigInteger[] lastFactors;
	public synchronized void service(ServletRequest req,
	ServletResponse resp) {
		BigInteger i = extractFromRequest(req);
		if (i.equals(lastNumber))
		encodeIntoResponse(resp, lastFactors);
		else {
		BigInteger[] factors = factor(i);
		lastNumber = i;
		lastFactors = factors;
		encodeIntoResponse(resp, factors);
		}
	}
}
```

**Reentrancy**

Reentrancy means that locks are acquired on a per‐thread rather than per‐invocation basis. Reentrancy is implemented by associating with each lock an acquisition count and an owning thread. When the count is zero, the lock is considered unheld. When a thread acquires a previously unheld lock, the JVM records the owner and sets the acquisition count to one.

*Code that would Deadlock if Intrinsic Locks were Not Reentrant.*

```java
public class Widget {
	public synchronized void doSomething() {
	
	...
	
	}
}

public class LoggingWidget extends Widget {

	public synchronized void doSomething() {
	
	System.out.println(toString() + ": calling doSomething");
	
	super.doSomething();
	
	}

}
```

## Guarding state with locks

