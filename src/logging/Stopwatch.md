# Stopwatch

Another handy utility class in `swing-extras` is the `Stopwatch` class. 

When performance testing your code, it's often the case that you want to know how long a particular
section of code takes to execute, so you can measure the success (or failure) of your optimization
efforts. A fairly standard approach is to do something like this:

```java
long startTimeMs = System.currentTimeMillis();

// Do something that might eat a few CPU cycles...

long elapsedTimeMs = System.currentTimeMillis() - startTimeMs;
logger.info("That took "+elapsedTime+"ms.");
```

This approach is usually "good enough" but isn't very pretty. We can make it somewhat easier to work
with by using the `Stopwatch` class:

```java
Stopwatch.start("myTimer");

// Do something that might eat a few CPU cycles...

Stopwatch.stop("myTimer");
logger.info("That took "+Stopwatch.reportFormatted("myTimer"));
```

This code is functionally equivalent to what we had before, with a couple of important distinctions:
- We can have multiple timers running concurrently by giving them unique names.
- We can invoke `report()` to get the raw millisecond count, or `reportFormatted()` to get a human-readable version.

## A more complex example:

```java
Stopwatch.start("totalProcess");
for (int i = 0; i < someHighNumber; i++) {
        doSomething();
        doSomethingElse();
        
        Stopwatch.start("criticalInnerSection");
        doSomethingCritical();
        Stopwatch.stop("criticalInnerSection");
        logger.fine("Critical inner section took "+Stopwatch.report("criticalInnerSection")+"ms.");
}
Stopwatch.stop("totalProcess");
logger.info("Total process took "+Stopwatch.reportFormatted());
```

We can start and stop as many timers as we need and they can run concurrently without interfering
with one another. 