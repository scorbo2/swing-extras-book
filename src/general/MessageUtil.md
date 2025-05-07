# MessageUtil

Often, your application will need to show an informational, warning, or error message to the user. Usually,
you will want to log that message also. This can of course be done in two steps, but wouldn't it be nice
if there were a handy utility to bring those two things together? Meet `MessageUtil`!

`MessageUtil` needs a parent component (for displaying informational dialogs) and a Logger instance (for
writing log messages). For example:

```java
MessageUtil messageUtil = new MessageUtil(MainWindow.getInstance(), Logger.getLogger(getClass().getName()));
```

Now you can easily display informational, warning, or error messages, and have them simultaneously go
out to the log file:

```java
messageUtil.error("Load error", "Error loading data!", exception);
```

If an exception is supplied, it will be included in the log output (But not in the displayed message).

The interface to `MessageUtil` is pretty straightforward:

```java
public void error(String message) { ... }
public void error(String title, String message) { ... }
public void error(String message, Throwable ex) { ... }
public void error(String title, String message, Throwable ex) { ... }

public void info(String message) { ... }
public void info(String title, String message) { ... }

public void warning(String message) { ... }
public void warning(String title, String message) { ... }
```

This utility saves a small amount of code every time you need to display a message to the user
and log it at the same time.
