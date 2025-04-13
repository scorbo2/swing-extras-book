# Working with enums

It's worth taking a moment to look more closely at a very common use case with properties in general:
I want my application to have a property whose available options are determined with an enum. How can I
do that? You could of course use the `swing-extras` class `ComboProperty`, which specifically lets
you store a multi-choice option as a property. It looks like this:

```java
List<String> options = new ArrayList<>();
for (MyEnum value : MyEnum.values()) {
    options.add(value.toString());
}
comboProperty = new ComboProperty("fieldname", "Choose:", options, 0, false);
```

There are several problems with this approach:
- I have to write boilerplate code to get all the values out of the enum
- By using `toString()` to load the value into the combo, I have to parse the String value that comes out of the combo box.
- If the `toString()` changes over time (or gets localized to another language), my properties file breaks.

Alternatively, I could use `value.name()` instead of `value.toString()` to populate the combo box, but 
then the user has to look at the internal names of my enum values, which are often written in ALL_CAPS.

Wouldn't it be nice if I could just say "this property should use values from this enum"?

## Using EnumProperty

This is the problem that `EnumProperty` was designed to solve. Let's look at a better way of doing the above code:

```java
EnumProperty<MyEnum> enumProp = new EnumProperty<>("fieldname", "Choose:" MyEnum.VALUE1);
```

That's it! The `EnumProperty` is smart enough to interrogate the given enum and extract its values, using
`toString()` to populate the combo box, but using `name()` to store items into the `Properties` object.
This gives you the best of both worlds.

You also have the choice of using the `useNamesInsteadOfLabels` option with `EnumProperty` if you actually
want to display the enum `name()` instead of its `toString()` in the combo box. The default value is
to use `toString()` as it is usually the user-friendlier option.
