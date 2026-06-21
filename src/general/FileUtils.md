# File utilities

There are a few utility classes available in `swing-extras` to simplify common file-related tasks.

## File and Directory scanning

The `3.0` release introduces two classes to greatly simplify scanning for files and directories in the filesystem:

- `FileScannerThread` - a worker thread that can scan for files with a particular extension, or files from within a list of extensions.
- `DirectoryScannerThread` - a worker thread that can scan for directories, with optional recursion.

These classes both implement `SimpleProgressWorker` from the [progress package](Progress.md), so they can 
easily be used with the `MultiProgressDialog` class to provide a progress bar and cancel button to the user
while the search is in progress. Cancellation is handled by the worker threads, so your code simply needs to
implement a `CompletionListener` and a `CancelListener` to handle the results:

```java
// Find all text and Markdown files in a directory (recursion is true by default):
FileScannerThread scanner = new FileScannerThread(new File("/path/to/search"))
    .addExtensionsToMatch(List.of(".txt", ".md"))
    .addCompletionListener(results -> handleResults(results))
    .addCancelListener(() -> handleCancellation());
MultiProgressDialog progressDialog = new MultiProgressDialog(parent, "Scanning...");
progressDialog.runWorker(scanner, true); // run the scan and dispose when finished.
```

Note that your handlers are invoked on the worker thread! If you need to update the UI from these
handlers, you must remember to marshal back to the EDT using `SwingUtilities.invokeLater()`.

```java
public void handleResults(List<File> results) {
    // It's fine to do heavy IO here, 
    // as we're still in the worker thread:
    // Load a file, for example...
    
    // Then, update the UI safely:
    SwingUtilities.invokeLater(() -> {
        // Update the UI with the results here.
    });
}
```

## FileSystemUtil

The `FileSystemUtil` class also provides several useful lower-level static utility methods,
if you wish to handle the file/directory scanning yourself:

- findFiles/findFilesExcluding/findSubdirectories - perform a search with optional recursion looking for files or directories matching certain criteria
- extractTextFileFromJar - extract a text file from within a JAR file and return its contents as a String
- readFileToString - read the contents of a text file into a String with optional Charset support
- writeStringToFile - write a String to a text file with optional Charset support
- readFileLines - read the lines of a text file into a List of Strings
- writeLinesToFile - write a List of Strings to a text file, one line per entry
- readStreamToString - read the contents of an InputStream into a String with optional Charset support
- sanitizeFilename - makes any String safe to use as a filename by removing/replacing invalid characters
- getPrintableSize - converts a file size in bytes to a human-readable String (e.g. "1.5 MB")

## DownloadManager

The `DownloadManager` class provides a simple way to download files from the internet with support for 
progress monitoring and cancellation. This is basically a convenient wrapper around the `java.net.HttpClient`
class, with some convenience methods added on top.

Downloading a file with DownloadManager is as easy as specifying the URL and a progress listener.
The file is downloaded asynchronously on a background thread and saved in the system temp directory.
Upon completion, your code can inspect the file, or move it to a permanent location if desired.
Here's a simple example of how to use DownloadManager:

```java
DownloadManager downloadManager = new DownloadManager();
String fileUrl = "https://example.com/somefile.zip";
downloadManager.downloadFile(fileUrl, new MyDownloadListener());
```

The `MyDownloadListener` class would implement the `DownloadListener` interface to receive progress updates:

```java
public class MyDownloadListener implements DownloadListener {
    @Override
    public void downloadBegins(DownloadThread thread, URL url) {
        // We are notified that the download has begun
    }

    @Override
    public void downloadProgress(DownloadThread thread, URL url, long bytesDownloaded, long totalBytesIfKnown) {
        // Here, we receive progress updates at regular intervals as the download proceeds
        // (Note that very small files may complete too quickly to receive progress updates,
        //  so this method is not guaranteed to fire)
        
        // If the download is taking too long, we can offer the user a "cancel" button,
        // and we can signal that the download should be aborted by using the "kill" method:
        if (shouldCancelDownload()) {
            thread.kill(); // This will abort the download and trigger a downloadFailed() callback
        }
    }

    @Override
    public void downloadFailed(DownloadThread thread, URL url, String errorMsg) {
        // If something goes wrong, we are notified here.
    }

    @Override
    public void downloadComplete(DownloadThread thread, URL url, File result) {
        // When the download finishes successfully, we receive the resulting File here.
    }
}
```

## FileWatcher

The `FileWatcher` utility provides a way to monitor a particular file for changes.  It wraps Java's `WatchService` API 
to provide a simple interface for watching a single file. You can specify a callback to be invoked whenever the file 
is modified. This is useful if your application is presenting a file for viewing or editing, and you want to be
aware of external changes to the file. `FileWatcher` is very easy to set up:

```java
FileWatcher watcher = new FileWatcher(someFile, this::onChange);
watcher.start(); // starts a worker thread to monitor the file
watcher.isRunning(); // reports true if the watcher is active
watcher.stop(); // stops the watcher thread and cleans up.
```

Your `onChange` handler is any Runnable that will be invoked when the file is changed.
Note that the handler is invoked on the worker thread, so you should marshal back to the EDT if you need to update the UI:

```java
public void onChange() {
    SwingUtilities.invokeLater(() -> {
        // Update the UI to reflect the file change here.
    });
}
```

If your application wants to save changes to the file, you can temporarily suspend the watcher
to avoid triggering a change report from your own change:

```java
watcher.ignoreSelfTriggeredChanges();

// We now have a few milliseconds to save our changes.
```

You can optionally specify the time for event suspension (the default is 1 second):

```java
watcher.ignoreSelfTriggeredChanges(2000); // ignore changes for 2 seconds
```

File watching automatically resumes after the suspension period.

## TextFileDetector

Sometimes, it's handy to have a way of knowing if a given file is a plain text file or not.
For example, we want to load it and show it to the user in a text edit dialog, but only if it's a text file.
The `TextFileDetector` class provides a simple way to determine if a file is likely to be a text file.
You can use it like this:

```java
File file = new File("path/to/somefile.txt");
boolean isTextFile = TextFileDetector.isTextFile(file);
if (isTextFile) {
    // Load and display the file contents
} else {
    // Show an error message or handle accordingly
}
```

The `isTextFile()` method performs a simple heuristic check by reading the first few bytes of the file
and looking for non-text characters. While not foolproof, it works well for most common cases.
The default settings are sufficient for most purposes, but the `isTextFile()` method offers an overload
that allows you to customize the number of bytes to check and the threshold for non-text characters.

```java
// Check only the first 512 bytes, and allow up to 10% non-text chars
boolean isTextFile = TextFileDetector.isTextFile(file, 512, 0.1); 
```

## HyperlinkUtil

The `HyperlinkUtil` class provides an easy way to open URLs in the user's default web browser,
if the current JRE allows this operation. You can use it like this:

```java
String url = "https://example.com";
HyperlinkUtil.openHyperlink(url);
```

This will attempt to open the specified URL in the default web browser. If it fails,
an error is logged, but no exception is thrown. You can optionally specify an owner Component when
invoking this method. If specified, a popup dialog will be shown with the error message if the operation fails.

```java
// Show a popup error if hyperlink browsing fails:
HyperlinkUtil.openHyperlink(url, ownerComponent);
```

### BrowseAction

The `HyperlinkUtil` class also provides a convenient `BrowseAction` class that can be used
to create Actions that open hyperlinks when triggered. This is useful for adding hyperlink
functionality to buttons or menu items in your Swing application. In particular, this integrates
very well with the hyperlink capabilities of the `LabelField` class in `swing-forms`:

```java
FormPanel formPanel = new FormPanel();

// Create a label field with hyperlink text:
final String url = "https://github.com/scorbo2/swing-extras"; 
LabelField labelField = new LabelField("Project page:", url);

// If hyperlink browsing is available, we can make it clickable:
if (HyperlinkUtil.isBrowsingSupported() && HyperlinkUtil.isValidUrl(url)) {
    labelField.setHyperlink(HyperlinkUtil.BrowseAction.of(url, introPanel));
}

formPanel.add(labelField);
```

If browsing is supported, the hyperlink text in the LabelField will be made clickable, and clicking it
will open the URL in the default web browser. If browsing is not supported, the text will be displayed
without hyperlink styling added to it, which is a graceful fallback.

The BrowseAction class can also be used with JButtons or JMenuItems directly:

```java
final String url = "https://example.com";
HyperlinkUtil.BrowseAction browseAction = HyperlinkUtil.BrowseAction.of(url, ownerComponent);

// We can optionally set the name of the action (used as button/menu text):
browseAction.setName("Visit example.com");

JButton visitButton = new JButton(browseAction);
``` 