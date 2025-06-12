# Extension resources

It's a fairly common use case for extensions to package some resource that needs to be loaded. For example,
an icon to display on a button, or an audio file to play when certain actions happen. Of course, these resources
can be packaged into the jar file and then loaded via `getClass().getResourceAsStream()`. 

However! There's an odd bug that requires this to be done **only in the constructor of the extension**. 
Attempting to invoke `getResourceAsStream()` from any method other than the extension constructor will
simply return null. See the comments on [swing-extras issue 34](https://github.com/scorbo2/swing-extras/issues/34)
for a little more details.

This also applies to loading the `extInfo.json` file as a resource in your extension. For example:

```java
public class MyExtension extends MyAppExtension {
    private final AppExtensionInfo extInfo;
    private final BufferedImage someIcon;
    
    public MyExtension() {
        extInfo = AppExtensionInfo.fromExtensionJar(getClass(), "/path/to/extInfo.json");
        if (extInfo == null) {
            throw new RuntimeException("MyExtension: can't load extInfo.json!");
        }
        
        try {
            someIcon = ImageUtil.loadFromResource(getClass(), "/path/to/icon.png", 32, 32);
        }
        catch (IOException ioe) {
            throw new RuntimeException("MyExtension: can't load icon resource!", ioe);
        }
    }
}
```

As long as all resources are loaded in the extension constructor, as depicted above, they will be loaded as normal.
Attempting to lazy-load any resources after construction will result in those resources failing to load.

The cause of this is currently unknown. Pull requests welcome! :) 

