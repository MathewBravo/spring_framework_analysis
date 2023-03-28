# Analyzing the Observer Pattern in Spring Framework

Within the Spring Frameworks objects can inherit the ApplicationEvent which allows them to create events, give those events timestamps, and allow other objects to listen to those events. Application events are a class extended by ALL application events, and is made abstract because generic events should not be published directly.

Essentially event handling in the `ApplicationContext` is provided through the `ApplicationEvent`
class and the `ApplicationListener` interface. If a bean that implements the
`ApplicationListener` interface is deployed into the context, every time an
`ApplicationEvent` gets published to the `ApplicationContext`, that bean is notified.
Essentially, this is the standard Observer design pattern.

# Example in Action

Let's look at an example of the Observer pattern within the Spring Framework.

### Concrete Subject

```java
public class NewUserEvent extends ApplicationEvent {

    private User user;

    /**
     * Create a new {@code ApplicationEvent}.
     *
     * @param source the object on which the event initially occurred or with
     *               which the event is associated (never {@code null})
     */
    public NewUserEvent(final Object source, final User user) {
        super(source);
        this.user = user;
    }

    public User getUser() {
        return this.user;
    }
}
```

The NewUserEvent extends ApplicationEvent. This is the Concrete Subject. It creates and manages events that are sent to the observers anytime a new user is registered.

It overrides the constructor of the ApplicationEvent (our subject) which also overrides the EventObject constructor to create a new event object and set it's parameter as the source of the event.

### Subject

```java
public ApplicationEvent(Object source) {
		super(source);
		this.timestamp = System.currentTimeMillis();
}
```

### Concrete Observer's

The SendUserRegisteredEmailHandler,UserExternalSynchronizationHandler are our concrete observers and implements the interface for responding to change using information about the change obtained from the concrete subject. Here anytime we detect a newUserEvent via an event Listener we will call the handleNewUserEvent method.

```java
@Component
public class SendUserRegisteredEmailHandler {

    @Autowired
    private INotificationService notificationService;

    @EventListener
    public void handleNewUserEvent(final NewUserEvent newUserEvent) {

        notificationService.sendUserRegisteredNotification(newUserEvent.getUser());

    }
}
```

```java
@Component
public class UserExternalSynchronizationHandler {

    @Autowired
    private IUserSynchronizationService userSynchronizationService;

    @EventListener
    public void handleNewUser(final NewUserEvent newUserEvent) {
        userSynchronizationService.synchronizeWithExternalServices(newUserEvent.getUser());
    }
}
```

### Observer

The built in EventListener util of Java acts as our observer. It is a built in annotation that allows us to listen for events. It comes with a single method called handleEvent() which fires whenever an event occurs of the type for which the EventListener interface was registered.

```java
    @EventListener // <- here
    public void handleNewUserEvent(final NewUserEvent newUserEvent) {
```

# Diagrams

Behind the scenes diagram
![Observer Pattern](https://github.com/MathewBravo/spring_framework_analysis/blob/main/ObserverPattern.png)

How Autowiring works
![PubSub](https://github.com/MathewBravo/spring_framework_analysis/blob/main/Autowired.png)


## Source Code Behind It.

```java
public abstract class ApplicationEvent extends EventObject {

	/** use serialVersionUID from Spring 1.2 for interoperability. */
	private static final long serialVersionUID = 7099057708183571937L;

	/** System time when the event happened. */
	private final long timestamp;


	/**
	 * Create a new {@code ApplicationEvent} with its {@link #getTimestamp() timestamp}
	 * set to {@link System#currentTimeMillis()}.
	 * @param source the object on which the event initially occurred or with
	 * which the event is associated (never {@code null})
	 * @see #ApplicationEvent(Object, Clock)
	 */
	public ApplicationEvent(Object source) {
		super(source);
		this.timestamp = System.currentTimeMillis();
	}

	/**
	 * Create a new {@code ApplicationEvent} with its {@link #getTimestamp() timestamp}
	 * set to the value returned by {@link Clock#millis()} in the provided {@link Clock}.
	 * <p>This constructor is typically used in testing scenarios.
	 * @param source the object on which the event initially occurred or with
	 * which the event is associated (never {@code null})
	 * @param clock a clock which will provide the timestamp
	 * @since 5.3.8
	 * @see #ApplicationEvent(Object)
	 */
	public ApplicationEvent(Object source, Clock clock) {
		super(source);
		this.timestamp = clock.millis();
	}


	/**
	 * Return the time in milliseconds when the event occurred.
	 * @see #ApplicationEvent(Object)
	 * @see #ApplicationEvent(Object, Clock)
	 */
	public final long getTimestamp() {
		return this.timestamp;
	}

}
```

This spring abstract class is an implementation of the basic java Util, EventObject. EventObject is the root class from which all state objects are derived.

```java
public class EventObject implements java.io.Serializable {

    @java.io.Serial
    private static final long serialVersionUID = 5516075349620653480L;

    /**
     * The object on which the Event initially occurred.
     */
    protected transient Object source;

    /**
     * Constructs a prototypical Event.
     *
     * @param source the object on which the Event initially occurred
     * @throws IllegalArgumentException if source is null
     */
    public EventObject(Object source) {
        if (source == null)
            throw new IllegalArgumentException("null source");

        this.source = source;
    }

    /**
     * The object on which the Event initially occurred.
     *
     * @return the object on which the Event initially occurred
     */
    public Object getSource() {
        return source;
    }

    /**
     * Returns a String representation of this EventObject.
     *
     * @return a String representation of this EventObject
     */
    public String toString() {
        return getClass().getName() + "[source=" + source + "]";
    }
}
```

Events are published for the purpose of being listened to by the ApplicationEventPublisher.

```java
@FunctionalInterface
public interface ApplicationEventPublisher {

	/**
	 * Notify all <strong>matching</strong> listeners registered with this
	 * application of an application event. Events may be framework events
	 * (such as ContextRefreshedEvent) or application-specific events.
	 * <p>Such an event publication step is effectively a hand-off to the
	 * multicaster and does not imply synchronous/asynchronous execution
	 * or even immediate execution at all. Event listeners are encouraged
	 * to be as efficient as possible, individually using asynchronous
	 * execution for longer-running and potentially blocking operations.
	 * @param event the event to publish
	 * @see #publishEvent(Object)
	 * @see org.springframework.context.event.ContextRefreshedEvent
	 * @see org.springframework.context.event.ContextClosedEvent
	 */
	default void publishEvent(ApplicationEvent event) {
		publishEvent((Object) event);
	}

	/**
	 * Notify all <strong>matching</strong> listeners registered with this
	 * application of an event.
	 * <p>If the specified {@code event} is not an {@link ApplicationEvent},
	 * it is wrapped in a {@link PayloadApplicationEvent}.
	 * <p>Such an event publication step is effectively a hand-off to the
	 * multicaster and does not imply synchronous/asynchronous execution
	 * or even immediate execution at all. Event listeners are encouraged
	 * to be as efficient as possible, individually using asynchronous
	 * execution for longer-running and potentially blocking operations.
	 * @param event the event to publish
	 * @since 4.2
	 * @see #publishEvent(ApplicationEvent)
	 * @see PayloadApplicationEvent
	 */
	void publishEvent(Object event);

}

```

The EventListener is the class that handles the tracking of all observers.
Classes are tracked within an array of classes that require a listener in order to process published events.

This class is used via the @EventListener annotation.

```java
public @interface EventListener {

	/**
	 * Alias for {@link #classes}.
	 */
	@AliasFor("classes")
	Class<?>[] value() default {};

	/**
	 * The event classes that this listener handles.
	 * <p>If this attribute is specified with a single value, the
	 * annotated method may optionally accept a single parameter.
	 * However, if this attribute is specified with multiple values,
	 * the annotated method must <em>not</em> declare any parameters.
	 */
	@AliasFor("value")
	Class<?>[] classes() default {};

	/**
	 * Spring Expression Language (SpEL) expression used for making the event
	 * handling conditional.
	 * <p>The event will be handled if the expression evaluates to boolean
	 * {@code true} or one of the following strings: {@code "true"}, {@code "on"},
	 * {@code "yes"}, or {@code "1"}.
	 * <p>The default expression is {@code ""}, meaning the event is always handled.
	 * <p>The SpEL expression will be evaluated against a dedicated context that
	 * provides the following metadata:
	 * <ul>
	 * <li>{@code #root.event} or {@code event} for references to the
	 * {@link ApplicationEvent}</li>
	 * <li>{@code #root.args} or {@code args} for references to the method
	 * arguments array</li>
	 * <li>Method arguments can be accessed by index. For example, the first
	 * argument can be accessed via {@code #root.args[0]}, {@code args[0]},
	 * {@code #a0}, or {@code #p0}.</li>
	 * <li>Method arguments can be accessed by name (with a preceding hash tag)
	 * if parameter names are available in the compiled byte code.</li>
	 * </ul>
	 */
	String condition() default "";

	/**
	 * An optional identifier for the listener, defaulting to the fully-qualified
	 * signature of the declaring method (e.g. "mypackage.MyClass.myMethod()").
	 * @since 5.3.5
	 * @see SmartApplicationListener#getListenerId()
	 * @see ApplicationEventMulticaster#removeApplicationListeners(Predicate)
	 */
	String id() default "";

}
```

An example of the EventListener annotation is shown below. This is Springs built in test event listener. EventListeners must be attached to the event object in order to be able to process events.

```java
@Component
	static class TestEventListener extends AbstractTestEventListener {

		@EventListener
		public void handle(TestEvent event) {
			collectEvent(event);
		}

		@EventListener
		public void handleString(String content) {
			collectEvent(content);
		}
	}
```
