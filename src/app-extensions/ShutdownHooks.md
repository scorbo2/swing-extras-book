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

## Failing to register a shutdown hook

If you have not registered any shutdown hooks, you will notice a warning issued in the
log during an application restart:

```
[WARNING] No shutdown hooks are registered! Application may not terminate cleanly.
```

This is a hint that you should register at least one shutdown hook to provide a clean
termination of your application during a restart.
