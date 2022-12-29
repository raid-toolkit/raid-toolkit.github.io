---
title: Create an extension (VS Code)
parent: Get Started
nav_order: 2
---

# Creating a new extension using `dotnet` cli

> Before getting started, consider following the Visual Studio project template directions [here](./quickstart-vs22.md).

## Create a new classlib project

To start, we will base our extension off of a basic classlib project. We need to **be sure to specify `net6.0-windows10.0.19041` as our target framework**:

```ps
> dotnet new classlib --name SampleExtension --framework net6.0 --target-framework-override net6.0-windows10.0.19041
```

Next, edit your new `.csproj` file and add the following to the top `<PropertyGroup>`. Note that both singular and plural forms of these variables must be set. This will ensure the extension builds specifically for win10 x64 platforms.

```xml
  <Platform>x64</Platform>
  <Platforms>x64</Platforms>
  <RuntimeIdentifier>win10-x64</RuntimeIdentifier>
  <RuntimeIdentifiers>win10-x64</RuntimeIdentifiers>
  <!-- https://github.com/dotnet/maui/issues/5886#issuecomment-1123106200 -->
  <EnablePreviewMsixTooling>true</EnablePreviewMsixTooling>
```

And add the required nuget packages:
```ps
> dotnet add package Raid.Toolkit.Extensibility --prerelease
> dotnet add package Raid.Toolkit.Extensibility.Tasks --prerelease
```

## Create a solution

Create a new solution for your project by running:

```ps
> dotnet new sln
> dotnet sln add <path_to_your.csproj>
```

Finally, edit the sln file manually and change all occurrences of `Any CPU` to `x64`.

It should look something like this:

```go
Microsoft Visual Studio Solution File, Format Version 12.00
# Visual Studio Version 17
VisualStudioVersion = 17.0.31903.59
MinimumVisualStudioVersion = 10.0.40219.1
Project("{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}") = "SampleExtension", "SampleExtension.csproj", "{6D6BB049-00BC-4A0F-8C4A-BC89C4016957}"
EndProject
Global
	GlobalSection(SolutionConfigurationPlatforms) = preSolution
		Debug|x64 = Debug|x64
		Release|x64 = Release|x64
	EndGlobalSection
	GlobalSection(SolutionProperties) = preSolution
		HideSolutionNode = FALSE
	EndGlobalSection
	GlobalSection(ProjectConfigurationPlatforms) = postSolution
		{6D6BB049-00BC-4A0F-8C4A-BC89C4016957}.Debug|x64.ActiveCfg = Debug|x64
		{6D6BB049-00BC-4A0F-8C4A-BC89C4016957}.Debug|x64.Build.0 = Debug|x64
		{6D6BB049-00BC-4A0F-8C4A-BC89C4016957}.Release|x64.ActiveCfg = Release|x64
		{6D6BB049-00BC-4A0F-8C4A-BC89C4016957}.Release|x64.Build.0 = Release|x64
	EndGlobalSection
EndGlobal
```

## Add an extension package class

Replace the default `Class1.cs` file with one that defines the following:

```cs
using Raid.Toolkit.Extensibility;

namespace SampleExtension01
{
	public class Extension : ExtensionPackage
	{
		public override void OnActivate(IExtensionHost host)
		{
			// TODO: Initialize your extension here
		}
	}
}
```

## Add a manifest

Create a new file `manifest.json` in your project root directory with the following contents. See [Manifest Schema](../development-guide/manifest-schema.md) for more details on the contents of this file.

```json
{
    "id": "SampleExtension01",
    "type": "SampleExtension01.Extension",
    "asm": "SampleExtension01.dll",
    "displayName": "SampleExtension01",
    "description": "Extracts account data",
    "codegen": {
        "types": [
            "^Client\\.Model\\.AppModel$"
        ]
    }
}
```

Lastly, add the new manifest file to the bottom of your csproj as an `RTKExtension` target:

```xml
<ItemGroup>
    <RTKExtensionManifest Include="manifest.json" />
</ItemGroup>
```

## Build your (empty) extension

Run `dotnet build` in your terminal and you should see a successful build. If you hit any errors, check back through these steps and make sure you didn't miss or mistype anything.

Now that your extension has built, you'll notice a little `_rtkCache` folder in the root of your extension project with `Raid.Interop.dll` in it. This dll is generated with each build and is automatically linked to your project as a dependency so that you get debug and intellisense functionality in visual studio. When your extension is installed it will instead get a version of this DLL directly from RTK at runtime.

If you don't see this directory but have a successful build, you probably are missing a manifest.json file or the `RTKExtensionManifest` element in your csproj. This file (specifically it's `codegen.types` property is what controls the classes that RTK will generate code for).

Once this is available, you can start referencing APIs from the game to access various objects, state and data.

# Debugging your extension

To debug your extension in VS Code, use the following launch configuration:

```jsonc
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug Extension",
            "type": "coreclr",
            "request": "launch",
            "program": "${env:userprofile}\\AppData\\Local\\RaidToolkit\\bin\\Raid.Toolkit.exe",
            "cwd": "${env:userprofile}\\AppData\\Local\\RaidToolkit",
            "env": {
                // update this path to the output directory of your extension build
                "DEBUG_PACKAGE_DIR": "${workspaceFolder}/SampleExtension/bin/x64/Debug/net6.0-windows10.0.19041"
            },
            "args": [
                "--standalone",
                "--no-default-packages",
                "--debug-package",
                "."
            ]
        }
    ]
}
```