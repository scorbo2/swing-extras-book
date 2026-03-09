# Events

ActionPanel has a number of discrete, functional listener interfaces that callers can use to respond to events:

- *ExpandListener*: notifies you when a group is expanded or collapsed.
- *GroupRemovedListener*: notifies you when a group is removed.
- *GroupRenamedListener*: notifies you when a group is renamed.
- *GroupReorderedListener*: notifies you when a group is reordered.
- *OptionsListener*: notifies you when any ActionPanel option is modified.
- *MarginsListener*: notifies you when the margins of the ActionPanel are changed.

## "But wait, where is the 'item has been added' listener?"

Adding new items to an ActionPanel is done through a caller-supplied item supplier. It is therefore left as an 
exercise for the caller to respond to items being added. Your supplier provides the new item, therefore it makes
sense for your supplier to notify your application code about the new item!

## Some examples of listening for events

```java
// Listen for any group expand/collapse event:
actionPanel.addExpandListener((g,e) -> {
    // Example: "Group FirstGroup expanded: true"
    // Example: "Group SecondGroup expanded: false" 
    log.info("Group " + g.getName() + " expanded: " + e);
});

// Listen for changes to header margins:
actionPanel.getHeaderMargins().addListener(m -> {
    log.info("Header margins changed! New margins: " + m);
});

// Listen for changes to our border options (via OptionsListener):
actionPanel.getBorderOptions().addListener(() -> {
    // Do something in response to border option changes
});
```

You can see that there are quite a large number of possible events that you can listen for and respond to!
`OptionsListener` in particular is quite powerful. The general pattern is that ActionPanel exposes a number
of `get...Options()` that return some subclass of `ActionPanelOptions`. Each of these classes offer a
`addListener()` method to allow you to listen for changes to only that set of options, independently of 
options changes happening elsewhere in the ActionPanel. And because every Listener interface used by
ActionPanel is a `FunctionalInterface`, you can use lambda expressions to keep your code concise and readable!
