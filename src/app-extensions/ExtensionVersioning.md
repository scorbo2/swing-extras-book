# Extension versioning

As your application goes through normal development and extension over time, you may find that it becomes
a bit cumbersome to match which versions of your extensions were written for which version of your application.
Of course, each extension publishes an `extInfo.json` as described in the section [Registering extensions](Registering.md).
This json file explicitly says which version of the app that extension targets. But, this is not immediately obvious
from looking at the extension jar directory. For example:

```shell
app-extension-name-1.0.jar
app-extension-name-1.1.jar
app-extension-name-1.2.jar
app-extension-name-1.3.jar
```

We see we've gone through a few versions of the extension, and there's no way to tell at a glance what
app version each of these extensions intended to target.

## A versioning convention

For this reason, the following convention is *suggested* (nothing in the code enforces this - you can version
your extensions and your application however you like... but this might avoid headaches):

- Extensions should use the major.minor version of the application
- Extensions can add a patch version for multiple releases for a specific app version

This convention allows us to easily determine compatibility just by looking at the jar files:

```text
app-extension-name-2.1.0.jar (targets app version 2.1)
app-extension-name-2.1.1.jar (targets app version 2.1)
app-extension-name-2.2.0.jar (targets app version 2.2)
app-extension-name-3.0.0.jar (targets app version 3.0)
```

And so on. Now the guesswork is removed and we know which jar files apply to the version of the application we're using.

