# app-extensions

The `ca.corbett.extensions` package contains some powerful classes that you can use to dynamically
change the functionality of your application, without changing the code of your application itself.

Because `app-extensions` builds upon both `swing-forms` and the properties handling classes contained
within `swing-extras`, it is **strongly recommended** that you read the sections on
swing-forms and properties handling first.

## How does it all work?

When designing your application, you identify certain "extension points" - that is, parts
of the code where an extension could either change the way that something is done, or add
additional functionality that the application does not do out of the box.

For example, let's say we're writing an application that accepts input in the form
of Xml files, extracts information from those files, and writes it to a database.
The loading of the input files is an obvious extension point. Could we write an extension
that would allow the application to accept input in other formats? Json or CSV, for example?

For a more complex example, let's say we are writing an image editing application that
supports certain image edit operations out of the box: rotate, flip, resize, and so on.
Could we write an extension that dynamically adds a completely new image editing operation
to our application, without changing the code of our application itself?

The `app-extensions` code provides a way to handle both of the example scenarios,
and many more. In the next sections, we'll look at this in detail.
