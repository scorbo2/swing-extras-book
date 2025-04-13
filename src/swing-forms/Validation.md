# Form validation

Form validation is optional, but is very easy to do with `swing-forms`. Often you want
to restrict certain fields so that they only allow certain values, or so that the
values in one field are only valid if some value in some other field is within a
certain range, etc. These rules are very easy to apply in swing-forms.

We've already seen in the code example above an example of a possible built-in
validation, and that is the last parameter to the TextField constructor. If you
tell the TextField not to allow blank values, then it will automatically add
a FieldValidator with that rule to itself. But you are not limited to built-in
validation capabilities with these fields! You can add new validation rules
very easily. First, let's use the built-in validation rule to tell the TextField
to not allow blank values:

```java
TextField textField = new TextField("Can't be blank:", 15, 1, false);
```

Note that we have set the final constructor parameter to false, meaning that we
don't want the field to accept blank values. What happens now when we try to
validate the form with a blank value in that field?

![TextField validation](swing_forms_validation1.png "Validation in action")

We see that the field fails validation, and we get a helpful tooltip message
from the red validation marker. The TextField itself added that FieldValidator
on our behalf because of the way we instantiated it.

But what if we want to add custom validation? This is quite easy!

```java
textField.addFieldValidator(new FieldValidator<>(textField) {
    @Override
    public ValidationResult validate() {
        ValidationResult theResult = new ValidationResult();
        TextField theField = (TextField)super.field;
        if (theField.getText().length() < 3) {
            theResult.setResult(false, "Text must be at least three characters!");
        }
        return theResult;
    }
});
```

![TextField custom validation](swing_forms_validation2.png "Custom validation")

Now we see our custom validation message is triggered if our own validation logic
determines that the field is invalid.

FormFields can have multiple FieldValidators attached to them. In this case, all
FieldValidators must report that the field is valid, otherwise the field will be
marked as invalid. If more than one FieldValidator reports a validation failure,
the validation messages will be concatenated together, like this:

![Multiple validation rules](swing_forms_validation3.png "Multiple validation rules")

Here we see both the built-in FieldValidator and our custom FieldValidator have both
reported a validation failure for the field in question. In this particular case,
the two messages are redundant. But, you can see how easy it is to apply multiple
validation rules to a FormField and have the FormPanel itself manage validating each
field and displaying messages as appropriate.

### Validating a form

FormPanel offers two equivalent methods for form validation:

- `isFormValid()` will validate the form and return a boolean indicating validation success.
- `validateForm()` will simply validate the form and return nothing.

Both of these methods will cause the validation success (green checkmark) or
failure (X marker) to appear beside each validatable field. Note that some
fields, like LabelField or CheckBoxField, do not subject themselves to
validation by default, as their contents are quite simple.

### Manually enabling or disabling validation on a field

All `FormField` implementations have a `showValidationLabel` property which (usually)
defaults to `true`. Some fields, for example `LabelField`, set this property to `false`
automatically, to prevent validation checkmarks showing up pointlessly beside the label.
We can override that behaviour, either by explicitly calling `setShowValidationLabel(true)`
on the field in question, or implicitly, by adding a `FieldValidator` to the field.
Let's try that with a `LabelField` and see what happens:

```java
LabelField labelField = new LabelField("Labels usually don't validate...");
labelField.addFieldValidator(new FieldValidator<>(labelField) {
    @Override
    public ValidationResult validate() {
        ValidationResult theResult = new ValidationResult();
        LabelField theField = (LabelField)super.field;
        if (!theField.getText().isBlank()) {
            theResult.setResult(false, "How dare you have a label with text!");
        }
        return theResult;
    }
});
```

When we validate the form, we see that the label has allowed itself to be validated:

![Validating a LabelField](swing_forms_validation4.png "Validating a LabelField")

Later, when we look at [custom FormField implementations](CustomFields.md), we can think
about whether it makes sense for our new custom field to respond to validation or not.
It's easy to enable/disable this behaviour either per field instance or per field type
in the case of custom fields.
