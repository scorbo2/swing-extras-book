# Relation to swing-forms

If we look into `ca.corbett.extras.properties` package, we'll find many different
Property implementations... but implementations of what? They all extend a class
called `AbstractProperty`, which is well worth a closer look. Let's zoom in and
take a look at four of the abstract methods in this class:

```java
public abstract void saveToProps(Properties props);
public abstract void loadFromProps(Properties props);

public abstract FormField generateFormField();
public abstract void loadFromFormField(FormField field);
```

(Javadoc omitted for brevity). We can see that all implementations of `AbstractProperty`
have to do at least two basic tasks:
- load and save themselves from and to a `ca.corbett.extras.properties.Properties` instance
- generate a `FormField` from themselves and then later load the value from that `FormField`

This, in theory, allows us to support data in **any** custom format that we want. 

## A simple example: BooleanProperty

Let's start by looking at the easiest example: `BooleanProperty` extends `AbstractProperty`
to allow handling of boolean properties:

```java
@Override
public void saveToProps(Properties props) {
    props.setBoolean(fullyQualifiedName, value);
}

@Override
public void loadFromProps(Properties props) {
    value = props.getBoolean(fullyQualifiedName, value);
}
```

The `saveToProps()` and `loadFromProps()` methods are pretty much exactly what we might expect.
We save or load a single boolean value, and we're done. But we must also implement
`generateFormField()` and `loadFromFormField()` as well. What kind of form field would be
best for representing a boolean value? Why, a `CheckBoxField` of course!

```java
@Override
public FormField generateFormField() {
    CheckBoxField field = new CheckBoxField(propertyLabel, value);
    field.setIdentifier(fullyQualifiedName);
    field.setEnabled(!isReadOnly);
    field.setHelpText(helpText);
    return field;
}

@Override
public void loadFromFormField(FormField field) {
    if (field.getIdentifier() == null
            || !field.getIdentifier().equals(fullyQualifiedName)
            || !(field instanceof CheckBoxField)) {
        logger.log(Level.SEVERE, "BooleanProperty.loadFromFormField: received the wrong field \"{0}\"",
                   field.getIdentifier());
        return;
    }

    value = ((CheckBoxField)field).isChecked();
}
```

We see that the `generateFormField()` method creates a simple checkbox and assigns it a unique 
identifier (we'll discuss property identifiers in much more detail later). The `loadFromFormField()`
method does some basic error handling to make sure the field was wired up correctly, and then 
just reads the value from the checkbox. 

So far, so good. But this is barely scratching the surface of what we can do here.

## A complex example: FontProperty

Let's look at a property that isn't quite so straightforward. How do we store a Font? Well,
a Font can have a name (`SansSerif`, `Monospaced`, `Serif`, etc), but also style information,
such as bold or italics. In `swing-extras`, fonts can also optionally have color information
for their foreground color and background color. How can we store so much data in one single
property? Don't we have to map everything to a single name/value pair on the back end?

No, we don't. We can split it into multiple properties that we will manage behind the scenes.

```java
@Override
public void saveToProps(Properties props) {
    props.setString(fullyQualifiedName + ".name", font.getFamily());
    props.setBoolean(fullyQualifiedName + ".isBold", font.isBold());
    props.setBoolean(fullyQualifiedName + ".isItalic", font.isItalic());
    props.setInteger(fullyQualifiedName + ".pointSize", font.getSize());
    if (textColor != null) {
        props.setColor(fullyQualifiedName + ".textColor", textColor);
    }
    else {
        props.remove(fullyQualifiedName + ".textColor");
    }
    if (bgColor != null) {
        props.setColor(fullyQualifiedName + ".bgColor", bgColor);
    }
    else {
        props.remove(fullyQualifiedName + ".bgColor");
    }
    props.setBoolean(fullyQualifiedName + ".allowSizeSelection", allowSizeSelection);
}

@Override
public void loadFromProps(Properties props) {
    String fontName = props.getString(fullyQualifiedName + ".name", font.getFamily());
    boolean isBold = props.getBoolean(fullyQualifiedName + ".isBold", font.isBold());
    boolean isItalic = props.getBoolean(fullyQualifiedName + ".isItalic", font.isItalic());
    int pointSize = props.getInteger(fullyQualifiedName + ".pointSize", font.getSize());
    font = Properties.createFontFromAttributes(fontName, isBold, isItalic, pointSize);
    textColor = props.getColor(fullyQualifiedName + ".textColor", textColor);
    bgColor = props.getColor(fullyQualifiedName + ".bgColor", bgColor);
    allowSizeSelection = props.getBoolean(fullyQualifiedName + ".allowSizeSelection", allowSizeSelection);
}
```

Whoah, there's a lot going on here! Saving a single `FontProperty` to a `Properties` object
actually ends up creating a bunch of property entries! But how will we keep them all grouped
together? We use the `fullyQualifiedName` of the property and then append sub-names for all
of our individual attributes. So, a single `FontProperty` with a `fullyQualifiedName` of
`my.amazing.font` will end up creating the following entries:

```properties
my.amazing.font.allowSizeSelection=true
my.amazing.font.isBold=false
my.amazing.font.isItalic=false
my.amazing.font.name=SansSerif
my.amazing.font.pointSize=22
```

This is transparent to callers of this class - they don't need to know or care about the storage details
of the property in question. It also means there's effectively no limit to how complicated our `AbstractProperty`
implementations can get. Because we're not limited to a single name/value pair for our properties, we can
design property wrappers around even very complex data types, and still save them with one line of code!

Let's see now what happens when we ask a `FontProperty` to generate a form field, and to load itself
from a form field:

```java
@Override
public FormField generateFormField() {
    FontField field = new FontField(propertyLabel, getFont(), textColor, bgColor);
    field.setIdentifier(fullyQualifiedName);
    field.setEnabled(!isReadOnly);
    field.setHelpText(helpText);
    field.setShowSizeField(allowSizeSelection);
    return field;
}

@Override
public void loadFromFormField(FormField field) {
    if (field.getIdentifier() == null
            || !field.getIdentifier().equals(fullyQualifiedName)
            || !(field instanceof FontField)) {
        logger.log(Level.SEVERE, "FontProperty.loadFromFormField: received the wrong field \"{0}\"",
                   field.getIdentifier());
        return;
    }

    FontField fontField = (FontField)field;
    font = ((FontField)field).getSelectedFont();
    textColor = fontField.getTextColor();
    bgColor = fontField.getBgColor();
}
```

We see that there's a very close relationship between `FontProperty` and the matching `swing-forms` class `FontField`.
This is because the intention with *most* properties is that you're eventually going to want to expose them
to the user for viewing and/or editing. This integration between the `ca.corbett.extras.properties` package
and the `swing-forms` classes is what not only enables this, but also makes things MUCH easier when it comes
time to generating and showing that UI. As it turns out, our calling code won't have to worry about 
generating `FormPanel` instances at all...
