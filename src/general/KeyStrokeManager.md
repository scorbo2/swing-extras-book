# KeyStrokeManager

The `KeyStrokeManager` class is a utility for managing keyboard shortcuts (keystrokes) in Swing applications.
This class has a very capable parser that allows you to specify keystrokes using a human-readable string format,
instead of having to manually create `KeyStroke` instances. For example:

```java
keyStrokeManager.isKeyStrokeValid("Ctrl+P"); // true
keyStrokeManager.isKeyStrokeValid("Ctrl+Shift+S"); // true
keyStrokeManager.isKeyStrokeValid("Alt+F4"); // true
keyStrokeManager.isKeyStrokeValid("Meta+DEL"); // true
```

Available modifiers (case-insensitive) are:

```
ctrl/control
alt
shift
meta/cmd/command
```

You can combine multiple modifiers with a main key using the `+` character. Example: `ctrl+alt+shift+X`.

Many "special" keys are available by name, including:

```
enter
escape/esc
space
tab
backspace
delete/del
insert/ins
home
end
pageup
pagedown
up
down
left
right
f1, f2, ..., f12
```

Any other input is treated as a single case-insensitive character key.

The `KeyStrokeManager` class provides handy methods for converting a KeyStroke instance into a user-friendly
String, and vice versa:

```java
KeyStroke ks = KeyStrokeManager.parseKeyStroke("Ctrl+S");
String ksString = KeyStrokeManager.keyStrokeToString(ks); // "Ctrl+S"
```

## Okay, how do I use it?

It's very easy to get up and running with `KeyStrokeManager`. Here's a simple example:

```java
// In your main window setup code:
KeyStrokeManager keyManager = new KeyStrokeManager(mainWindow);

// Register as many handlers as you need!
keyManager.registerHandler("Ctrl+A", aboutAction);
keyManager.registerHandler("Ctrl+S", saveAction);
keyManager.registerHandler("Ctrl+Q", exitAction);
```

As long as your `mainWindow` is the active window, pressing the specified keystrokes will 
automatically trigger the associated actions. You can temporarily suspend keyboard handling:

```java
keyManager.suspend(); // temporarily disable all keystroke handling
keyManager.resume(); // re-enable keystroke handling
```

Or you can reassign handlers at any time:

```java
// Remap aboutAction to Ctrl+Shift+A instead of Ctrl+A:
keyManager.registerHandler("Ctrl+Shift+A", aboutAction);
```

## User customization and persistence

The new `KeyStrokeField` field in swing-forms allows you to expose keystroke settings to users
on any FormPanel. For example:

```java
KeyStroke keyStroke = KeyStrokeManager.parseKeyStroke("Ctrl+A");
KeyStrokeField keyField = new KeyStrokeField("About shortcut:", keyStroke);
formPanel.add(keyField);
```

And of course, there is a companion `KeyStrokeProperty` class, so that you can easily wire up
keyboard shortcuts into your application properties, for persistence:

```java
// In your application props setup:
props.add(new KeyStrokeProperty("UI.Shortcuts.about",
          "About dialog shortcut:",
          KeyStrokeManager.parseKeyStroke("Ctrl+A"))); // default value
```

That way, the property will automatically show up in your application settings dialog, and the user's
preferred shortcut will be saved and restored between application runs. Then, you just need to expose
the current value in a getter in your AppProperties implementing class:

```java
public KeyStroke getAboutShortcut() {
    return getKeyStrokeProperty("UI.Shortcuts.about").getKeyStroke();
}
```

And then the setup code in your window would look like this:

```java
// In your main window setup code:
KeyStrokeManager keyManager = new KeyStrokeManager(mainWindow);

// Register as many handlers as you need!
keyManager.registerHandler(myAppConfig.getAboutShortcut(), aboutAction);
keyManager.registerHandler(myAppConfig.getSaveShortcut(), saveAction);
keyManager.registerHandler(myAppConfig.getExitShortcut(), exitAction);
// And so on for your other shortcuts
```

## Application extensions

You can allow your application extensions to supply their own KeyStroke handlers, to allow easy
activation of extension features with the same KeyStrokeManager instance! Extensions can use
the `isAvailable` method in KeyStrokeManager to check for conflicts before registering their handlers:

```java
if (keyStrokeManager.isAvailable("Ctrl+E")) {
    keyStrokeManager.registerHandler("Ctrl+E", myExtensionAction);
} else {
    // Handle conflict (e.g. choose a different shortcut, or notify the user)
}
```

And of course, if your application extensions return `KeyStrokeProperty` instances in their list
of exposed properties, those will also be automatically added to the application settings dialog,
allowing users to customize extension shortcuts as well!
