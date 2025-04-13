# Properties handling

For the purposes of this documentation "properties" refers to any kind of name/value pair of data
that you wish to persist in your application. This could be application settings, user preferences,
application state (window size and position, etc) that you wish to automatically load and use on
the next startup, and so on. Java provides some built-in ways of solving this problem, but as we'll
see, the built-in Java approach is limiting and sometimes frustrating.

## java.util.Properties

The built-in `java.util.Properties` class is great at dealing with simple String-based name/value
pairs of data. It even comes with `store()` and `load()` methods that allow you to write data out
to a file and read it back in later.

Advantages of `java.util.Properties`:
- extremely simple API
- handles i/o for you

However, there are some drawbacks:
- values are `String` only
  - That means, if you want to store some custom type, you have to handle conversion to/from `String`
- this means that the simple API actually works against you (you have to write custom code here)

## java.prefs.Preferences

The built-in `java.util.Preferences` seems at first glance as though it solves the problems
presented by the `java.util.Properties` class:
- primitive types, such as `boolean`, `int`, `float`, and `double` are wrapped
- can write to a "user node" (negates the need to manually pick a file save location)

For example, to use `java.util.Preferences`:

```java
Preferences prefs = Preferences.userNodeForPackage(ca.corbett.Example.class);
prefs.put("someImportantPref", "somevalue");
```

On my system (linux-based), this automatically creates a file: 
`/home/scorbett/.java/.userPrefs/ca/corbett/prefs.xml` with the following content:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE map SYSTEM "http://java.sun.com/dtd/preferences.dtd">
<map MAP_XML_VERSION="1.0">
  <entry key="someImportantPref" value="someValue"/>
</map>
```

The nice part about this is that it was very easy for us to use, and also that we didn't have to
worry about selecting a save location - this is done automatically for us by `java.util.Preferences`.
The not-so-nice part about this is that the exact location of the output varies from system to system,
and on some systems, may not even be stored in a file at all, but rather in some other OS-supplied
storage mechanism, such as a system registry. This can make troubleshooting difficult, and also makes
it difficult to transport application settings easily from one machine to another.

Another drawback of this approach is that, if each of your classes uses its own class as the parameter
for `userNodeForPackage`, then your application preferences may end up spread across multiple files
in multiple directories, making it difficult or impossible to view them all at once (again, in a 
troubleshooting type of scenario where you're trying to figure out why things aren't saving/loading
correctly).

Wouldn't it be nice if there was a portable way to consistently handle properties, resulting in all
application properties going to the same location, which can then be transported as needed to another
machine, or even checked into source control? Well, there is!

## The ca.corbett.extras.properties package

First, let's say hello to the `ca.corbett.extras.properties.Properties` class. It has the following benefits:

- wraps not just primitives, but also `Color` and `Font` objects
- the `FileBasedProperties` subclass can handle reading/writing to a file
- keeps all related properties in the same place.
- tightly integrates with `swing-forms` for generating form fields

If it ended there, this would be a pretty underwhelming offering. But wait, there's more!

### PropertiesManager and PropertiesDialog

The `PropertiesManager` and `PropertiesDialog` classes help your application manage settings, not just
in a "please persist these properties" kind of way, but also in a "hey, while you're at it, could you
please generate a nice UI for my users to view and edit those properties" kind of way.

The power of these classes is considerable in saving you from having to write code for managing
and displaying properties to the user. Let's take a closer look at these features!
