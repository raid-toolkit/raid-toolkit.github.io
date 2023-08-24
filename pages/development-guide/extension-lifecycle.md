---
parent: Development Guide
title: Extension Lifecycle
---

# Extension Lifecycle
{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

# Handling Activation

When RTK starts up, it will first enumerate all of its extensions and their codegen requirements listed in their manifests. 
Before activating any extensions, it will ensure the current game model that has been built includes those requirements (building a new one if necessary).
After generating and loading the codegen module, each extension will be activated by invoking its `OnActivate` method.

{: .important }
> Extension packages don't *need* to implement all of the lifecycle methods if they aren't required. 
> It is recommended to inherit from `ExtensionPackage` in this
case which will provide reasonable default implementations for those lifecycle methods that you can override as needed.

When shutting down, RTK will also give extensions an opportunity to shut down gracefully. 
The `OnDeactivate` method will be called, which may do any cleanup required by your extension. This cleanup must be performed <ins>synchronously</ins>; RTK will exit its process as soon as all extensions have returned from their `OnDeactivate` methods.

Also note that all extensions share a **5 second time limit** during shut down, after which the process will be terminated before they can complete their work.

## OnActivate


When activated, your extension can use this hook to expose its own functionality to the user. 
The interfaces required to do this are exposed on the `IExtensionHost` interface.
For example, you might register a window and menu item to show that window:

```cs
private IExtensionHost Host;
public override void OnActivate(IExtensionHost host)
{
    Host = host;
    MenuEntry menuEntry = new() { DisplayName = "Artifact Sorter", IsEnabled = true, IsVisible = true };
    menuEntry.Activate += OnShowUI;
    Disposables.Add(host.RegisterMenuEntry(menuEntry));
}

private void OnShowUI(object sender, EventArgs eventArgs)
{
    ArtifactSorterDialog dialog = Host.CreateWindow<ArtifactSorterDialog>();
    dialog.Show();
}
```

## OnDeactivate

When deactivating your extension, you are expected to dispose all API objects that were created, especially menu items and any UI elements.

```cs
private IntPtr HSomeNativeHandle;
public override void OnDeactivate(IExtensionHost host)
{
    base.OnDeactivate(host); // if you override `ExtensionPackage` don't forget to call base implementation!
    NativeMethods.Release(HSomeNativeHandle);
    HSomeNativeHandle = IntPtr.Zero;
}
```

{: .tip }
> The `ExtensionPackage` object provides a `Disposables` member which can collect all `IDisposable` objects
> that you might create and automatically disposes them when the extension is disposed. Using this can save you a lot of tedious work!

# Handling installation

When installed, an extension is given the opportunity to do some initialization work that might be required, such as setting up an 
initial database or interacting with other software on the computer.
Similarly, an extension is expected to clean up its data when it is uninstalled, an can use the obvious hook for that.

```cs
void OnInstall(IExtensionHost host);
void OnUninstall(IExtensionHost host);
```

# Handling activation

A slightly different activation path may come from the shell. If you wish to support things like web links or shortcuts that can invoke 
your extension in some way, you can do so by generating a link starting with `rtk://extension/<your-package-id>/`. Any protocol activations
of RTK with this URL will be routed to the `HandleRequest` method, which should return `true` if it handled the requested URL or not.

```cs
bool HandleRequest(Uri requestUri)
{
    if (requestUri.Query === "?action=ShowChampionBuilds")
    {
        this.ShowChampionBuilds(requestUri);
        return true;
    }
    return false;
}
```
