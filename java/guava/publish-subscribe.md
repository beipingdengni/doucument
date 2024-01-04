

```java
com.google.common.eventbus.EventBus

// 注册订阅、取消订阅和发布消息
// 注册
public void register(Object object);
// 取消注册
public void unregister(Object object);
// 发布
public void post(Object event);
```

## 同步使用(调用还是当前线程，方便代码解藕)

> EventBus eventBus = new EventBus();

> 默认的构造方法创建 EventBus 的时候，其中 executor 为 MoreExecutors.directExecutor()，其具体实现中直接调用的 Runnable#run 方法，使其仍然在同一个线程中执行，所以默认操作仍然是同步的，这种处理方法也有适用的地方，这样既可以解耦也可以让方法在同一个线程中执行获取同线程中的便利，比如事务的处理

```java
class EventListener {
  /**
  * 监听 Integer 类型的消息
  */
  @Subscribe
  @AllowConcurrentEvents //  执行Handler逻辑的时候有synchronized锁
  public void listenInteger(Integer param) {
	  System.out.println("EventListener#listenInteger ->" + param);
  }

  /**
  * 监听 String 类型的消息
  */
  @Subscribe
  public void listenString(String param) {
  	System.out.println("EventListener#listenString ->" + param);
  }
}

// 创建监听
EventBus eventBus = new EventBus();
eventBus.register(new EventListener());

// 按照发布顺序执行
eventBus.post(1);
eventBus.post("hello world");
```

## 异步使用

```java
EventBus eventBus = new AsyncEventBus(Executors.newCachedThreadPool());
```

## 异常处理

> 无论是 EventBus 还是 AsyncEventBus 都可传入自定义的 SubscriberExceptionHandler 该 handler 当出现异常时会被调用，我可可以从参数 exception 获取异常信息，从 context 中获取消息信息进行特定的处理

```java
public interface SubscriberExceptionHandler {
	/** 
	* 异常处理
	*/
	void handleException(Throwable exception, SubscriberExceptionContext context);
}
```

