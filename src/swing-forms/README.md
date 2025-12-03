# swing-forms

The `swing-forms` library allows developers to very quickly and easily lay out
forms without having to write manual layout code with `GridBagLayout` and
`GridBagConstraints`. Most commonly-used UI components are wrapped in 
`swing-forms`, allowing you to simply instantiate them and add them to
a `FormPanel`, with configurable validation and optional form actions.

Almost all `swing-forms` classes are extensible by design, allowing you to 
quickly build your own custom form fields if the built-in form fields don't
meet your needs.

## What can I do with swing-forms?

The `swing-forms` library wraps most of the common input components into easy-to-use
wrapper classes and hides much of the complexity of using them. Included example
form field implementations are:

- Checkbox
- Color picker (with support for solid colors and for color gradients)
- Combo box (editable and non-editable variants)
- File chooser (with optional image preview panel)
- Directory chooser
- Static label fields
- Number pickers (spinners)
- Text input fields (single-line and multiline supported)
- Password input fields
- Multi-select list fields
- ImageList fields, with drag-and-drop support for adding images
- Panel fields (for rendering custom stuff)

In the next few sections, we'll take a tour of the features provided by `swing-forms`, and we
will be making use of these features in most of the rest of this documentation, because many
of the other features included in `swing-extras` build on top of `swing-forms`.

## Brief history

`swing-forms` was originally developed and distributed as a separate library, but because of the
large overlap between `swing-form` and `swing-extras`, the decision was made to absorb all of
`swing-forms` into `swing-extras` as of the 2.0 release in April 2025.  
