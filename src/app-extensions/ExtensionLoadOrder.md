# Extension load order

Most of the time, the order in which extensions are loaded from disk doesn't matter, and so the default
load order (alphabetically by jar file name) is fine. However, there are some circumstances where you
want to ensure that extension A loads before extension B. Consider the following hypothetical method in
our application's ExtensionManager implementation:

```java
public boolean handleKeyEvent(KeyEvent keyEvent) {
    for (MyAppExtension extension : getEnabledLoadedExtensions()) {
        if (extension.handleKeyEvent(keyEvent)) {
            return true;
        }
    }
    return false;
} 
```

When a `KeyEvent` is received, our extension manager will iterate through all of our enabled and loaded
extensions in the order in which they were loaded, asking each one if it wants to handle that particular
key event. The first extension that handles the event terminates the search. So what happens if there
are two extensions that can both handle the same KeyEvent? Well, whichever one was loaded first will 
be able to process it, and the other one will not be called upon.

One possible approach to solve this is to continue making the `KeyEvent` available to all other extensions,
even after an extension has handled it. But perhaps we only want the KeyEvent to be handled by one single
extension. This same general pattern can appear whenever there is some kind of event within the application
that might be handled by an extension - sometimes, we only want ONE extension to handle it, and no more.
But how can we control which one?

## The default sort order

By default, all extension jars within the extension jar directory are loaded in alphabetical order. So, we
can control load order by naming our jars carefully. For example:

```shell
0001_important_extension.jar
0002_less_important_extension.jar
9999_really_unimportant_extension.jar
```

This will ensure that the `0001_important_extension.jar` is loaded before the others. The ExtensionManager will
always query extensions in the order they were loaded, so in this case, the `0001` extension will always get
first kick at the cat.

But this approach requires manual filename manipulation, and is therefore a little clunky and fragile. Is there
a better way?

## A better way: ext-load-order.txt

`ExtensionManager` will automatically look for an optional file called `ext-load-order.txt` in the extension
jar directory. If present, this file can specify the order of extensions by naming one jar file per line
within the file. For example:

```shell
# ext-load-order.txt
#
# Blank lines and lines starting with # are ignored
important_extension.jar
less_important_extension.jar
```

Any jars named in this file will be loaded in the order specified BEFORE the rest of the jars in the extension
jar directory are loaded. So, if a jar file is not explicitly mentioned in this text file, it will be loaded
after any jar that is mentioned.

If the file is not present, load order will revert to the default load order described above.

## Verifying extension load order

`ExtensionManager` will output log information as extensions are loaded. This can be used to easily verify
that extensions are being loaded in the expected order. For example:

```text
2025-06-11 09:04:54 P.M. [INFO] Extension loaded internally: Important Extension
2025-06-11 09:04:54 P.M. [INFO] Extension loaded externally: Less important extension
2025-06-11 09:04:54 P.M. [INFO] Extension loaded externally: Really unimportant extension
```

Here we see the expected results. You can increase the log level for the `ca.corbett.extensions` package to `FINE`
if you wish to see more information about the extensions that are being scanned and loaded, but the default
`INFO` log level will show you basic information that is sufficient for verifying load order.