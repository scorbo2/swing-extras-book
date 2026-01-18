# ResourceLoader

Your application likely has packaged resources such as images, audio files, configuration files, etc.
Normally, this means you'll end up with code for loading these resources, and handling any errors
that may arise during the load:

```java
String path = "com/example/myapp/resources/image.png";
URL url = MyClass.class.getClassLoader().getResource(path);
if (url == null) {
    throw new IOException("Resource not found: " + path);
}
try {
    Image image = ImageIO.read(url);
    return image;
} catch (IOException e) {
    throw new IOException("Error loading resource: " + path, e);
}
```

Then, repeat that code for every resource you need to load! This quickly becomes tedious and error-prone.
Wouldn't it be nice if there were an easier way to handle this common task?

The `ResourceLoader` class in `swing-extras` provides a simple way to load resources from your application's
classpath, with built-in error handling and support for multiple resource types. 

## Usage option 1: Use ResourceLoader directly

The `ResourceLoader` class exposes static convenience methods for loading common resource types.
An example of using these methods is shown below:

```java
// Set our "prefix", so we don't have to repeat it for every resource load:
ResourceLoader.setPrefix("com/mycompany/myapp/");

// Retrieve a named image as a BufferedImage:
BufferedImage logo = ResourceLoader.getImage("images/logo.png");

// Retrieve a text resource as a String:
// Will preserve line endings as in the original file.
String releaseNotes = ResourceLoader.getText("ReleaseNotes.txt");

// Retrieve a text resource as a List of lines:
List<String> configLines = ResourceLoader.getLines("config/settings.conf");

// Retrieve an ImageIcon, scaled to the given square size:
ImageIcon icon = ResourceLoader.getScaledIcon("icons/appIcon.png", 64);

// Extract a resource to a temporary file:
ResourceLoader.extractResourceToFile("data/template.docx",
                      new File("output/template.docx"));
```

This is a modest code saving, but wouldn't it be even nicer if we could customize this by adding
specific getter methods for our expected resources? Well, we can do that too!

## Usage option 2: Subclass ResourceLoader for your application

The `ResourceLoader` class is extensible, so that you can create your own convenience methods
to supply your application's specific resources. There is, in fact, an example of this within
`swing-extras` itself: the `SwingFormsResources` class. Let's take a quick look at it:

```java
public final class SwingFormsResources extends ResourceLoader {
    // ...

    /**
     * All swing-forms icon resources share this resource prefix.
     */
    private static final String PREFIX = "ca/corbett/swing-forms/images/";

    // Icons intended for use with FormFields:
    static final String VALID = "formfield-valid.png";
    static final String INVALID = "formfield-invalid.png";
    static final String HELP = "formfield-help.png";
    static final String BLANK = "formfield-blank.png";
    static final String LOCKED = "formfield-locked.png";
    static final String UNLOCKED = "formfield-unlocked.png";
    static final String COPY = "formfield-copy.png";
    static final String HIDDEN = "formfield-hidden.png";
    static final String REVEALED = "formfield-revealed.png";

    // ...

    /**
     * Returns an ImageIcon to represent a "valid" FormField - that is, one that
     * has no validation errors.
     */
    public static ImageIcon getValidIcon(int size) {
        return internalLoad(VALID, size);
    }

    /**
     * Returns an ImageIcon to represent an "invalid" FormField - that is, one that
     * has at least one validation error.
     */
    public static ImageIcon getInvalidIcon(int size) {
        return internalLoad(INVALID, size);
    }

    /**
     * Returns an ImageIcon to represent a FormField that has some help or informational
     * text associated with it.
     */
    public static ImageIcon getHelpIcon(int size) {
        return internalLoad(HELP, size);
    }

    /**
     * Returns an ImageIcon to represent a lock.
     */
    public static ImageIcon getLockedIcon(int size) {
        return internalLoad(LOCKED, size);
    }
    
    // ... and etc for other icons ...

    /**
     * All icons are all stored at 48x48 internally, but can be requested at any size.
     * This method will load one and cache it at native size, then return a resized
     * version as requested.
     *
     * @param resourceName Any of the icon name constants.
     * @param size The requested size. Use NO_RESIZE or a negative value to get the native size of 48x48.
     * @return An ImageIcon instance.
     */
    static ImageIcon internalLoad(String resourceName, int size) {

        // Pseudocode for internalLoad method:
        //
        // See if the image is cached, and return it if so
        // Load the resource image using ResourceLoader static methods
        // Add to cache at native size
        // Perform resize if requested
        // return the scaled image as an ImageIcon!
    }
```

The advantage of this approach is that our application code can now make extremely simple
and self-documenting calls to retrieve resources:

```java
final int ICON_SIZE = 16;
ImageIcon lockedIcon = SwingFormsResources.getLockedIcon(ICON_SIZE);
ImageIcon helpIcon = SwingFormsResources.getHelpIcon(ICON_SIZE);
```

## Summary

No matter which usage option you choose, the `ResourceLoader` class makes it easy to load
resources from your application's classpath, with built-in error handling and support for
multiple resource types. By subclassing `ResourceLoader`, you can create your own convenience
methods tailored to your application's specific resources, making your code cleaner and
more maintainable.

## A note about application extensions

Application extensions can make use of `ResourceLoader` to load resources from the application's
classpath/jar file, but they cannot use it to load resources from their extension jar files, as they
have their own class loaders. 

The [Extension resources](../app-extensions/ExtensionResources.md) section of the book provides
important information for application extension developers on how to properly load resources from extension
jar files.
