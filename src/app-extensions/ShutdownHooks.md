# Shutdown hooks

If your application supports dynamic extension discovery and download as discussed in
sections 3.7 and 3.8, then you should know that your application will need to restart
in order to install, uninstall, or update any extension jar. This application restart
is handled by ExtensionManager and you generally don't need to worry about it.

However! Your application may have some kind of cleanup method that gets invoked
when shutting down, to handle cleaning up resources, closing database connections
cleanly, and so on. Here's a hypothetical example:

```java
/**
 * Perform normal shutdown tasks before the application exits.
 */
private void cleanupAndShutDown() {
    // Save the size and position of open windows:
    saveUIState();
    
    // Tell all of our extensions that we are shutting down:
    myExtensionManager.deactivateAll();
    
    // Close any open database connections:
    DatabaseManager.getInstance().close();
    
    // Save anything that may not have been persisted:
    myDocumentManager.saveAll();
}
```

This is all well and good, but the application restarts fired by ExtensionManagerDialog
will use System.exit(), which will cut your cleanup method out of the loop. How do we fix this?

## Registering a shutdown hook

The UpdateManager class provides an easy mechanism for this:

```java
// Somewhere in your startup code...
myUpdateManager.registerShutdownHook(MainWindow::cleanupAndShutDown);
```

This registers your `cleanupAndShutDown()` method with the UpdateManager class, so that
the UpdateManager knows to invoke your cleanup method before restarting the application.
You can register as many shutdown hooks as you need.

Note that this is not a replacement for your normal cleanup scheme! The shutdown hook
is only invoked when UpdateManager restarts your application, not when your application
exits normally.

## IMPORTANT NOTE ABOUT THREADING

Shutdown hooks are executed on a worker thread. Most of the time, this is not a problem.
However, sometimes your cleanup code may need to interact with the UI. For example, perhaps
you wish to show a dialog to ask the user about committing unsaved changes, or perhaps you
need to manually close some open windows. In that case, it is **very important** that you
properly marshal these UI interactions to the Swing Event Dispatch Thread:

```java
/**
 * Perform normal shutdown tasks before the application exits.
 */
private void cleanupAndShutDown() {
    try {
        // Close our floating tool window:
        SwingUtilities.invokeAndWait(() -> {
            myToolWindow.close();
        });
    }
    catch (Exception e) {
        logger.error("Caught exception while shutting down: " 
                + e.getMessage(), e);
    }

    // ... the rest of your shutdown code
}
```

Note also that we use `invokeAndWait()` here instead of `invokeLater()` - the reason for this
is that the `UpdateManager` will terminate the application as soon as the last shutdown hook
returns. If you use `invokeLater()`, it is highly likely that your shutdown code will not
get a chance to execute before the application is terminated.

Failure to properly marshal UI interactions to the Event Dispatch Thread may cause unpredictable behavior, including
deadlocks. In the worst case, the UI may become completely unresponsive until `UpdateManager` hits its
timeout (described below).

`UpdateManager` will only wait for 30 seconds before force-terminating
all shutdown hook threads. This value is not currently configurable.
Relying on a popup to ask the user a question may cause your shutdown hook
to exceed that time limit. Consider forcing a silent save without user interaction, or at least
providing a visible timer on your popup to let the user know that they must respond quickly.
If your hook has not completed within the time limit, those unsaved changes may be lost!
Application restart cannot be prevented or extended by any shutdown hook.

## Failing to register a shutdown hook

If you have not registered any shutdown hooks, you will notice a warning issued in the
log during an application restart:

```
[WARNING] No shutdown hooks are registered! Application may not terminate cleanly.
```

This is a hint that you should register at least one shutdown hook to provide a clean
termination of your application during a restart.
