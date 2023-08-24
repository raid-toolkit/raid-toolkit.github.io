---
parent: Reference
title: Extension Manifest Schema
---

# Manifest.json

The `manifest.json` (or `.rtk.extension.json` in the produced build output) file describes your extension and everything RTK needs to know about it before it loads any of your code.

| property | usage |
| -------- | ----- |
| id       | unique id for your extension |
| type     | Namespace-prefixed class name for your extension entrypoint. This must reference a class that extends from `Raid.Toolkit.Extensibility.ExtensionPackage` |
| asm      | Output assembly .dll filename |
| displayName | Display name for your extension. Will be displayed when the user is prompted to install or manage your extension.
| description | Description for your extension.  Will be displayed when the user is prompted to install or manage your extension.
| codegen.types | This string-array is a list of regular expressions to match classes that should have code generated for them. All dependencies of the initial class list will automatically be added, so generally only entrypoint classes are needed. |
