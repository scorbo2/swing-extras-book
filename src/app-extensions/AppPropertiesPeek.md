# AppProperties.peek

It's worth a quick sidetrack at this point to talk about `AppProperties.peek()`. This method allows you to
"peek" at the currently-saved value of a property in the properties file, without actually loading that 
properties file. Why would you need to do this?

## Setting initial property states

Occasionally, you may wish to set the initial state of some property based on the value of some other
property. This creates a chicken-and-egg type of situation, where you can't properly write your
`createInternalProperties()` method without that value, but you can't get that value until after
the properties have been created. We can get around that by "peeking" at the value in the 
properties file, if it exists. 

Let's look at a hypothetical situation where I have some property,
let's call it "X", whose visibility is controlled by a checkbox labelled "Enable X". When the checkbox
is checked, the X property becomes visible, and when the checkbox is unchecked, then the X property
should be hidden. This is reasonably straightforward to implement, but we have a slight problem when
setting the *initial* visibility of property X. Let's see how `peek()` can help us here:

```java
@Override
protected List<AbstractProperty> createInternalProperties() {
    List<AbstractProperty> props = new ArrayList<>();
    
    // ... creating all of our properties ...
    
    // We'll have a checkbox to enable property "X" (whatever it is), and a property for X itself:
    BooleanProperty enableXProperty = new BooleanProperty("enableXProperty", "Enable X property", true);
    ShortTextProperty someXProperty = new ShortTextProperty("someXProperty", "Enter X:", "");
    
    // Now wire them up so that the checkbox shows or hides the X property at runtime:
    enableXProperty.addFormFieldChangeListener(event -> {
        boolean isEnabled = ((CheckBoxField)event.formField()).isChecked();
        FormField field = event.formPanel().getFormField("someXProperty");
        field.setVisible(isEnabled);
    });
    
    // That's great, but I also want to set the *initial* state of the X property.
    // It seems like I can't do that here, because the properties are not yet loaded!
    // i.e. I can't ask the checkbox for its current value because it's not yet properly initialized.
    // But we can "peek" at its value currently in the file, even before we load it:
    String peekedOption = peek(myConfigFile, "enableXProperty");
    if (peekedOption != null && ! peekedOption.isBlank()) {
        someXProperty.setInitiallyVisible(Boolean.parseBoolean(peekedOption));
    }
}
```

Now, when the properties dialog is created and shown, the *initial* state of property X is guaranteed
to be properly set. After that initial setup, the `FormFieldChangeListener` handles toggling visibility
as the checkbox is selected or unselected.

If the config file is empty because this is the first run and no properties have been
saved, then `peek()` will simply return an empty string.

This scenario is something that probably won't come up very often, but the `peek()` method provides a handy way 
to preview the config properties before they are loaded, in the rare cases where that is needed!