# Sharing extension config

Generally, extensions that expose config properties will expose properties that are specific to that extension.
This is enforced by specifying a unique fully-qualified property name, as described in the [properties](../properties/PropertiesDialog.md) section.
For example:

```java
// In our custom extension class:
@Override
protected List<AbstractProperty> createConfigProperties() {
    return List.of(new BooleanProperty("CustomExtension.CustomExtension.fieldName", "My checkbox"));
}
```

This will create a new tab on the properties dialog called `CustomExtension` with a checkbox labeled "My checkbox".
Not only is the name unique, but this property will be physically separated by the others by placing it into
a new tab just for this extension. 

But, sometimes, we have a set of related extensions that might share some common configuration properties.
For example, there might be multiple extensions for a document-processing application. These related extensions
might each add a different kind of annotation or footnote to the current document. The font size, face, and style
of these annotations can be controlled with config properties. But, we don't want to clutter the properties dialog
with multiple properties for font selection. Can we set it up so that all of our related annotation extensions
use the same font property? Yes we can!

The solution is for each extension to specify a property with the same fully qualified name. `ExtensionManager` is
smart enough to filter out duplicates from the combined list. This means that only one property will be created
for each unique name, regardless of how many extensions expose that property. To illustrate the above scenario,
each of our annotation extensions could have the same implementation of `createConfigProperties`:

```java
@Override
protected List<AbstractProperty> createConfigProperties() {
    return List.of(new FontProperty("Annotations.Font.fontSelector", "Annotation font:"));
}
```

Now only one font control will be added to the properties dialog, and all extensions that registered a config
property with that name will be able to query the value of it. This allows the user to set the annotation font
settings just once, and all annotation extensions (regardless of how many there are!) will use those settings.
Even if a new annotation extension is added later, if it also registers a property with the same name, then
it will also be able to immediately make use of the user's preferred annotation font setting.

It's just that easy!