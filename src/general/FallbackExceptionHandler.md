# FallbackExceptionHandler

The `FallbackExceptionHandler` is a utility class designed to provide a default mechanism for
handling uncaught exceptions in Java applications. It implements the `Thread.UncaughtExceptionHandler` interface,
allowing it to be set as the default handler for uncaught exceptions in any thread.

## How does it work?

By default, if your application code encounters an exception but does not explicitly handle it,
the Java runtime will print the exception stack trace to the console and terminate the thread.
This is a serious problem if your application is not launched from the console, but rather
through a desktop shortcut. In that case, the exception is effectively invisible to the user.
Worse, even if you are using the [LogConsole](../logging/LogConsole.md) utility, nothing will
appear there, because the exception is uncaught and never makes it to your logging framework.

Fortunately, Java has a mechanism in place for this specific scenario: the `Thread.UncaughtExceptionHandler`
interface. You can implement this interface and register a handler that will be invoked whenever
an uncaught exception occurs in any thread. The `swing-extras` library provides a very convenient
implementation of this interface in the form of the `FallbackExceptionHandler` class.
This handler ensures that uncaught exceptions are logged to your logging framework.

## How do I use it?

`FallbackExceptionHandler` comes with a static method called `register()` that you can call at the
start of your application:

```java
public static void main(String[] args) {
    // Register the fallback exception handler to catch uncaught exceptions and log them
    FallbackExceptionHandler.register();

    // ... rest of your application startup ...
}
```

That's it! You should invoke `register()` very early during your application startup, to ensure that
it's in place, and that exceptions will be caught and logged properly. There's no need to unregister
the handler, as it will remain in place for the lifetime of your application.
