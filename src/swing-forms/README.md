# swing-forms

The `swing-forms` library was initially developed and maintained as a separate library, but
has been absorbed into `swing-extras` as of the `2.0.0` release, to make maintenance and
extension much easier.

`swing-forms` is greatly useful in saving you from having to write a lot of manual layout
code for input forms, particularly with `GridBagLayout`, which can be complicated and
tedious to use. With `swing-forms`, you can very quickly stand up an input form with
optional validation rules and with optional actions attached to each form field. You can
also easily extend `swing-forms` by writing your own custom form field implementations.

## What can I do with swing-forms?

The swing-forms library wraps most of the common input components into easy-to-use
wrapper classes and hides much of the complexity of using them. Included example
form field implementations are:

- Checkbox
- Color picker (with support for solid colors and for color gradients)
- Combo box (editable and non-editable variants)
- File chooser
- Directory chooser
- Static label fields
- Number pickers (spinners)
- Text input fields (single-line and multiline supported)
- Multi-select list fields
- Panel fields (for rendering custom stuff)

In the next few sections, we'll take a tour of the features provided by `swing-forms`, and we
will be making use of these features in most of the rest of this documentation, because many
of the other features included in `swing-extras` build on top of `swing-forms`.
