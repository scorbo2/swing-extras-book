# General design

## Designing an application with extensions in mind

The first and most important step is to identify possible extension points in your code.
Let's start with the simpler of our two examples: we want our application to be
able to accept file input in other file formats. Okay, let's start by extending the
`AppExtension` abstract class for our fictional application:

```java
public abstract class MyExtension extends AppExtension {
    
    public abstract boolean isFormatSupported(File inputFile);
}
```

We've created an abstract class for our application and extended the base `AppExtension`
abstract class. We then add a new abstract method called `isFormatSupported` that accepts
a candidate file and returns either true or false based on that file's format. But
where is the logic for this method? Why not just implement right here?

**Because we don't want the application to know what additional formats are supported**.

The whole point of offloading this logic into an extension class is that the extension class
might not exist yet. In fact, the file format that we eventually want to support might
also not exist yet. We're coding for a future that hasn't happened yet. Putting the logic
into the application would make the application inflexible about what formats it supports,
and would require a new version of the application code down the road when we want to add
support for a new file format later. The whole point of this library is to avoid that.

We will also need a way for the extension to load data from the input file and return it,
presumably in the form of some model object that our application deals with. Let's do that next:

```java
public abstract class MyExtension extends AppExtension {
    
    public abstract boolean isFormatSupported(File inputFile);
    
    public abstract MyModelObject loadFromFile(File inputFile) 
            throws InputFormatNotSupportedException, InputProcessingException;
}
```

Okay, now we have an abstract method that our application code can invoke when loading data
from input files. But the logic for this method does not exist yet and in fact won't ever
exist within our application's codebase. In theory, our application can support
any format we want, via extensions.

But we're not done yet. We have to write our application code to consider the existence
and the capabilities of any extensions that may be present at runtime. Let's extend the
`ExtensionManager` class and add some logic to it:

```java
public class MyExtensionManager extends ExtensionManager<MyExtension> {
    // ...
    // skipping some boilerplate for now
    // ...
    
    public boolean isFormatSupported(File inputFile) {
        for (MyExtension extension : getAllLoadedExtensions()) {
            if (extension.isFormatSupported(inputFile)) {
                return true;
            }
        }
        return false;
    }
    
    public MyModelObject loadFromFile(File inputFile) 
        throws InputFormatNotSupportedException, InputProcessingException {
        for (MyExtension extension : getAllLoadedExtensions()) {
            if (extension.isFormatSupported(inputFile)) {
                return extension.loadFromFile(inputFile);
            }
        }
        return null;
    }
}
```

The methods we add here wrap the functionality offered by our currently loaded
extensions (if any). Now we in theory have what we need to modify our application's
loading code to take extensions into consideration:

```java
// ... somewhere in our application code ...
public MyModelObject loadInputFile(File inputFile) 
    throws InputFormatNotSupportedException, InputProcessingException {
    // If it's an xml file, we can handle it natively:
    if (isXmlFile(inputFile)) {
        return loadXmlInputFile(inputFile);
    }
    
    // Otherwise, see if any of our extensions know how to handle this file type:
    if (MyExtensionManager.getInstance().isFormatSupported(inputFile)) {
        return MyExtensionManager.getInstance().loadFromFile(inputFile);
    }
    
    throw InputFormatNotSupportedException("Input file is not in a supported format.");
}
```

First, we check to see if the input file is in a format recognized by our application code
itself. If so, we can handle it natively, the same way we would have before adding extension
support to our application. But, if the format is unknown to us, instead of simply giving up,
we can ask our extension manager if there are any registered extensions that know how to
process that file type. If so, we can get that extension to handle the loading of the input
file, even without our application knowing or caring about the particulars of what format it's in.

Using this same basic pattern, we can augment our application in any number of ways,
allowing extensions to either enhance existing processing, or even to add new processing
options that the original application codebase didn't even think of.
