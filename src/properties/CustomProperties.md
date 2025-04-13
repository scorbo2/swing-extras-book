# Custom properties

Like most things in `swing-extras`, the properties handling is designed with extensibility in mind.
It's fairly straightforward to implement a new `AbstractProperty` to store whatever custom data
your application needs to worry about. *Usually* this will also mean developing a new `FormField`
implementation to represent that data to the user and allow viewing/editing of it, but not 
necessarily. For example, in the previous section, we saw how `EnumProperty` was able to back
itself onto the existing `ComboField` by just presenting a friendlier interface to it. Your custom
property might be able to make use of an existing `FormField` in the same way. 

In order to implement the `saveToProps()` and `loadFromProps()` successfully, though, it's 
vitally important to understand how and why properties are given a `fullyQualifiedName`.

And that is an excellent segue to talk about the most powerful feature of the
`ca.corbett.extras.properties` package... 