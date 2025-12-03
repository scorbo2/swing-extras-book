# Extension resources

It's a fairly common use case for extensions to package some resource that needs to be loaded. For example,
an icon to display on a button, or an audio file to play when certain actions happen. Of course, these resources
can be packaged into the jar file and then loaded via `getClass().getResourceAsStream()`. 

However! Due to the way extension jars are dynamically loaded, you can **only load resources in your constructor,
or in the loadJarResources() method**. The class loader that loads the jar is **closed** after the extension is
loaded, so any attempt to do `getClass().getResourceAsStream()` will return `null` after your extension is
instantiated.

Here's the load order:

1. ExtensionManager opens the extension jar file
2. Extension main class is instantiated (your constructor fires)
3. ExtensionManager invokes `createConfigProperties()` exactly once
4. ExtensionManager invokes `loadJarResources()` exactly once
5. ExtensionManager closes the jar file (class loader is gone)

So, any resources that you need to load should be loaded either in your constructor or in the
`loadJarResources()` method. This also applies to loading the `extInfo.json` file as a resource 
in your extension. For example:

```java
public class MyExtension extends MyAppExtension {
    private final AppExtensionInfo extInfo;
    private BufferedImage someIcon;
    
    public MyExtension() {
        // We can safely load resources here in the constructor:
        extInfo = AppExtensionInfo.fromExtensionJar(getClass(), "/path/to/extInfo.json");
        if (extInfo == null) {
            throw new RuntimeException("MyExtension: can't load extInfo.json!");
        }
    }
     
    @Override
    public void loadJarResources() {
        try {
            // It's also safe to load jar resources here in the loadJarResources method:
            someIcon = ImageUtil.loadFromResource(getClass(), "/path/to/icon.png", 32, 32);
        }
        catch (IOException ioe) {
            throw new RuntimeException("MyExtension: can't load icon resource!", ioe);
        }
    }
}
```

Attempting to lazy-load any resource outside of your constructor or the loadJarResources() method will fail!

