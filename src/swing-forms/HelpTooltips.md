# Help tooltips

Most `FormField` implementations (including your own custom ones, if you want) support
the concept of a helpful informational tooltip that can be displayed next to the field.
It looks like this:

![Tooltips1](swing_forms_tooltips1.png "Tooltips1")

To enable this, you can invoke `setHelpText()` on the field in question and supply
some non-blank value (setting `null` or an empty string will disable the tooltip and
prevent the information icon from appearing).

```java
ShortTextField textField = new ShortTextField("Text:", 12);
textField.setHelpText("Help icons show up whenever a field has help text");
panel.add(textField);
```

When the user hovers the mouse over the informational icon, the tooltip appears:

![Tooltips2](swing_forms_tooltips2.png "Tooltips2")

To support this in custom form field implementations, you don't actually have to do
anything at all, as the code for this is handled by the abstract `FormField` class
and by the `FormPanel` class (which handles the rendering of the icon). 

You can prevent the help icon from showing up in your custom FormField implementation
by overriding the `hasHelpLabel()` method in your implementing class to return `false`.
Let's look at the default implementation of this method in the parent `FormField` class:

```java
public boolean hasHelpLabel() {
    return helpLabel.getToolTipText() != null && !helpLabel.getToolTipText().isBlank();
}
```

This default implementation is generally what you want: the help label will automatically
show up if your form field has help text, and will hide itself if not. If you're writing
a custom `FormField` implementation that, for whatever reason, never requires a help
label, you could override that method like this:

```java
@Override
public boolean hasHelpLabel() {
    return false; // help label will not show up even if help text is set
}
```
