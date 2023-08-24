---
layout: default
parent: Development Guide
title: Anatomy of an Extension
---

# Anatomy of an Extension
{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>


An extension consists of essentially two critical parts: a **manifest** and an **extension assembly** (dll). These provide RTK enough information to find your code and load it in its own context.

# Installable package

The package format for `.rtkx` files is simple. It uses a ZIP format and is expected to contain a valid manifest and required files/directory structure around it to support loading and executing your extension.

![](/img/rtkx-contents.png)

# The Manifest

The manifest must be a file named `.rtk.extension.json` and be in the root directory of your extension. This file includes some display information that allows RTK to tell the user a little about the extension before they install it as well as the entrypoint class and your extension dll.

Let's go through it bit by bit:

Firstly, your extension must have a unique id. This matches the directory name that will be created to contain your extension when it is installed on an end-user machine.
```jsonc
{
    "id": "DungeonMonkey",
```

Next, some basic display information for your extension:
```jsonc
    "displayName": "Dungeon Monkey"
    "description": "Monkey click replay. Good monkey."
```

After this is the information that is used to load and prepare for your extension. RTK needs to know what the DLL filename is for your extension as well as the .Net type name for your class that implements `ExtensionPackage` (more on this later).
```jsonc
    "type": "DungeonMonkey.Extension"
    "asm": "DungeonMonkey.dll"
```

Finally, a list of regular expressions will be used to match which class names you need RTK to generate code for. This will control what parts of the game you have access to, and will be automatically kept up to date as you change and build your extension.
```jsonc
    "codegen": {
        "types": [
            "^Client\\.Model\\.AppModel$",
            "^Client\\.RaidApp\\.RaidViewMaster$",
            "^Client\\.MVVM\\.Base\\.ViewModel\\.Contexts\\.",
            "^Client\\.ViewModel\\.Contextes\\.",
            "^Client\\.View\\.Views\\.",
            "^Entitas\\."
        ]
    }
}
```

# Extension assembly

In the sample manifest above, it refers to an assembly `"DungeonMonkey.dll"`. This DLL should be a C# class library output, and it should contain a class within the `DungeonMonkey` namespace named `Extension`.

When RTK loads your extension, it will load this assembly and call the various lifecycle methods on it to allow you to add your own functionality to the RTK application:

```cs
// IExtensionPackage interface reference
public interface IExtensionPackage : IDisposable
{
    void OnActivate(IExtensionHost host);
    void OnDeactivate(IExtensionHost host);
    void OnInstall(IExtensionHost host);
    void OnUninstall(IExtensionHost host);
    bool HandleRequest(Uri requestUri);
}
```

Read [Extension Lifecycle](./extension-lifecycle.md) next to understand how an extension is loaded and executed.
