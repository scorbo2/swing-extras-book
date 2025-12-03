# Customizing the form field

Generally, we see a pattern emerges:

```
AbstractProperty  --wraps-->  FormField
----------------              -------------
BooleanProperty               CheckBoxField
ComboProperty                 ComboField
ShortTextProperty             ShortTextField
         (and so on, and so on...)
```

Each `AbstractProperty` implementation wraps a `FormField` implementation, and exposes some of 
the configuration of that field. So, when writing an application with swing-extras, you actually
don't generally need to create your form fields directly, because whatever properties
you're working with will do that for you. Great! This provides an abstraction that lets you
focus more on what kind of data your application needs to work with, and less on how exactly
that data should be represented to the user.

But what if we want to customize the form field that our properties generate for us? What if
our wrapping AbstractProperty implementation doesn't expose every possible configuration option
in the wrapped FormField? Wouldn't it be nice if we had some kind of hook we could use to
inspect and modify the FormField that our property generates for us?

## Hooking into FormField generation

It turns out that there's a fairly easy way to do this! For example, let's say that I want
to generate a checkbox, but I want to override the default font color and style for the 
checkbox label, and supply my own. Here's how I can hook into the FormField generation
process to supply my own custom styling:

```java
BooleanProperty myBoolean = new BooleanProperty("myBoolean", "Some checkbox label");
myBoolean.addFormFieldGenerationListener(new FormFieldGenerationListener() {
    @Override
    public void formFieldGenerated(AbstractProperty property, FormField formField) {
        CheckBoxField checkbox = (CheckBoxField)formField;
        checkbox.getFieldComponent().setForeground(Color.RED);
        checkbox.getFieldComponent().setFont(new Font(Font.MONOSPACED, Font.BOLD, 12));
    }
});
```

The `FormFieldGenerationListener` does exactly what it sounds like - it lets you listen for
the creation of FormField instances from a given AbstractProperty, not just to be notified as
to when it happens, but to allow your code to inspect and modify the given FormField to do
things that you otherwise wouldn't be able to do through the wrapping AbstractProperty alone.

Here's how our custom BooleanProperty will render with our listener in place:

![FormFieldGenerationListener](formfieldgenerationlistener.png "FormFieldGenerationListener")
