# Listening for changes in a ListField

Like all `FormField` implementations, the `ListField` class allows you to add a `ValueChangedListener`
to listen for changes in the field's value:

```java
myListField.addValueChangedListener(new ValueChangedListener() {
    @Override
    public void formFieldValueChanged(FormField field) {
        log.info("The ListField value changed!");
    }
});
```

Or, more simply, with a lambda expression:

```java
myListField.addValueChangedListener(field -> log.info("The ListField value changed!"));
```

However, with `ListField`, there's a slightly unexpected complication. 

## What do we mean by "value"?

The "value" of a FormField usually refers to the data that the user has entered or selected in the field.
For a `ListField`, the "value" is the list of selected items. That means that the `ValueChangedListener`
will only be invoked when the selection changes - that is, when the user selects or deselects items
in the list. But what if the *contents* of the list change? For example, what if items are added to 
or removed from the list?

For this, we use `ListDataListener` instead!

```java
myListField.addListDataListener(new ListDataListener() {
    @Override
    public void intervalAdded(ListDataEvent e) {
        log.info("Items were added to the ListField!");
    }

    @Override
    public void intervalRemoved(ListDataEvent e) {
        log.info("Items were removed from the ListField!");
    }

    @Override
    public void contentsChanged(ListDataEvent e) {
        log.info("The contents of the ListField changed!");
    }
});
```

## Choose either, or both

It's of course possible to have both a `ValueChangedListener` and a `ListDataListener` on the same `ListField`,
so that you are notified of both selection changes and content changes. But it's important to understand the difference
between the two types of listeners, so that you can choose the right one (or both) for your use case!
