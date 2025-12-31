# File utilities

There are a few utility classes available in `swing-extras` to simplify common file-related tasks.

## FileSystemUtil

The `FileSystemUtil` class provides several useful static utility methods:

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