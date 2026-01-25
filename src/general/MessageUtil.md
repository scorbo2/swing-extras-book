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

The interface to `MessageUtil` for info, warning, or error dialogs is pretty straightforward:

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

## Interactivity

Starting in the 2.7 release of swing-extras, the `MessageUtil` also adds a few options for interactivity:

- `askYesNo()` - Displays a Yes/No dialog to the user.
- `askYesNoCancel()` - Displays a Yes/No/Cancel dialog to the user.
- `askText()` - Prompts the user for simple single-line text input.
- `askSelect()` - Prompts the user to select one option from a list of options.

These methods all wrap similar methods in the underlying `JOptionPane` class, but simplify the API,
thereby saving a modest amount of code. Some simple examples:

### Yes/No question

```java
// The JOptionPane way:
if (JOptionPane.showConfirmDialog(
                null,
                "Are you sure about that?",
                "Confirm",
                JOptionPane.YES_NO_OPTION
        ) == JOptionPane.YES_OPTION) {
    // User clicked Yes
}

// The MessageUtil way:
if (messageUtil.askYesNo("Confirm", "Are you sure about that?") == MessageUtil.YES) {
    // User clicked Yes
}
```

### Text input

```java
// The JOptionPane way:
String input = (String)JOptionPane.showInputDialog(
                parent,
                message,
                title,
                JOptionPane.QUESTION_MESSAGE,
                null, // icon
                null, // selectionValues - not relevant here
                initialValue);
if (input != null) {
    // User entered some text
}

// The MessageUtil way:
String input = messageUtil.askText("Enter value:", initialValue);
if (input != null) {
    // User entered some text
}
```

### Multi-selection

```java
// The JOptionPane way:
String[] options = {"Option 1", "Option 2", "Option 3"};
Object result = JOptionPane.showInputDialog(
        parent,
        "Choose an option:",
        "Select",
        JOptionPane.QUESTION_MESSAGE,
        null,    // icon
        options,
        "Option 1"); // initial value
if (result != null) {
    // User made a selection
}

// The MessageUtil way:
String[] options = {"Option 1", "Option 2", "Option 3"};
String selection = messageUtil.askSelect("Choose an option:", options, "Option 1");
if (selection != null) {
    // User made a selection
}
```

### Summary

We see that `MessageUtil` saves a modest amount of code for such requests.
It also centralizes the logging of messages, which is a nice bonus.
