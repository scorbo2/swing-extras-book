# Extension versioning

As your application goes through normal development and extension over time, you may find that it becomes
a bit cumbersome to match which versions of your extensions were written for which version of your application.
Of course, each extension publishes an `extInfo.json` as described in the section [Registering extensions](Registering.md).
This json file explicitly says which version of the app that extension targets. But, this is not immediately obvious
from looking at the extension jar directory. For example:

```shell
app-extension-name-1.0.jar # This one was written for application version 1.0
app-extension-name-1.1.jar # This one was also written for application 1.0
app-extension-name-1.2.jar # This one was released for application 1.1
app-extension-name-1.3.jar # This one was released for application 2.0... wait, what?
```

We see we've gone through a few versions of both the application and extension, and there's no way to tell at a glance what
app version each of these extension versions intended to target.

## A versioning convention

For this reason, the following convention is *suggested* - nothing in the code enforces this! Of course,
you can ignore this suggestion and version your extensions and your application however you like... but 
the below suggestion is what I do, and I find it avoids headaches:

- Applications use a major.minor version scheme
- Extensions use a major.minor.patch version scheme
- The extension major.minor matches the application version that it targets
- The extension patch number can be used for subsequent versions of the same extension targeting the same application version

This convention allows us to easily determine compatibility just by looking at the jar files:

```text
app-extension-name-2.1.0.jar (targets app version 2.1)
app-extension-name-2.1.1.jar (targets app version 2.1)
app-extension-name-2.2.0.jar (targets app version 2.2)
app-extension-name-3.0.0.jar (targets app version 3.0)
```

And so on. Now the guesswork is removed, and we know which jar files apply to the version of the application we're using.

## Backward and forward compatibility

Starting in `swing-extras` 2.6, the extension loading system has been enhanced to allow for
some backward and forward compatibility when loading extensions. The old approach in ExtensionManager was very strict:

**Determining compatibility** (the old, strict approach, pre-2.6):
- An extension is compatible with the application only if its target application version (as specified in `extInfo.json`) exactly matches the current application version.
- If an extension targets version 2.1 of the application, then that extension jar may ONLY be used with application version 2.1.
- This means that every minor application release necessitates a new release of every extension that targets that application! Even if no breaking changes were made in the extension API.

**Determining compatibility** (the new, more flexible approach, 2.6 and later):
- An extension is compatible with the application if its target application version (as specified in `extInfo.json`) is less than or equal to the current application version, but with the same major version.
- If an extension targets version 2.1 of the application, then that extension jar may be used with application versions 2.1, 2.2, 2.3, etc., but NOT with version 3.0 or later.
- This allows extensions to be forward-compatible with future minor versions of the application, assuming no breaking changes were made in the extension API.
- So, applications can release new minor versions, and all existing extensions will still work with the new minor version.

The new approach is less strict, but it comes with an important caveat: it assumes that no breaking changes were made in the extension API
between minor versions of the application. If breaking changes were made, then the extension may not function correctly, even if it is considered "compatible" by the above rules. This means that applications must exercise discipline when releasing new versions of themselves:

- A new minor version of the application (e.g., 2.1 to 2.2) should not introduce breaking changes to the extension API!
- Breaking changes should only be introduced in new major versions of the application (e.g., 2.x to 3.0).

ExtensionManager may fail to load an extension if there have been breaking changes in the application since
the extension was written. This may result in a `NoSuchMethodError`, `ClassNotFoundException`, or similar error when the extension is loaded.
Applications should document any breaking changes in their extension API in their release notes, so that extension developers are aware of them.
