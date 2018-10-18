---
title: PackageReference with packages lock file
description: Details on how to use packages.lock.json file with PackageReference by NuGet 4.9+ and VS2017 and .NET Core 2.1.5xx
author: jainaashish
ms.author: jainaashish
ms.date: 10/18/2018
ms.topic: conceptual
---

# Package references (PackageReference) with packages lock file (packages.lock.json)

Package references, using the `PackageReference` node, manage NuGet dependencies directly within project files (as opposed to a separate `packages.config` file) [PackageReference in Project file](../../consume-packages/Package-References-in-Project-Files.md). 

With PackageReference, Developers do not have confidence that NuGet will restore to the same full closure of package dependencies when they build it on Dev machine vs. CI/CD machines. They also doesn't know anything about their transitive dependencies coming through direct dependencies and when they get changed? (See for more details [Enable repeatable package restore using lock file](https://github.com/NuGet/Home/wiki/Enable-repeatable-package-restore-using-lock-file).)

So with this packages lock file (`packages.lock.json`) feature, developers will be able to persist their restore resolution graph which should be added into their source control. Later this file will become the source of truth on CI/CD or any new machine to resolve the restore graph.

## Opt-In to packages lock file

Set a MSBuild property `RestorePackagesWithLockFile` in your project file using the following syntax:

```xml
<PropertyGroup>
    <!--- ... -->
    <RestorePackagesWithLockFile>true</RestorePackagesWithLockFile>
    <!--- ... -->
</PropertyGroup>    
```

After that either run a restore or install/update a NuGet dependency, which will generate a `packages.lock.json` file at the project root level.

You can also create blank `packages.lock.json` or `packages.<project_name>.lock.json` (replace spaces with `_` in project name) file at project root directory and run restore. This will also allow you to opt-in to this feature.

## Overwriting default packages lock file path

To overwrite default packages lock file name and path, Set MSBuild property `NuGetLockFilePath` in your project file using the following syntax:

```xml
<PropertyGroup>
    <!--- ... -->
    <NuGetLockFilePath>custom/<custom>.lock.json</NuGetLockFilePath>
    <!--- ... -->
</PropertyGroup>    
```

## Controlling restore locked mode

By default restore will always update this packages lock file (if needed) but developer can control this by setting another MSBuild property `RestoreLockedMode` in the project file like below:

```xml
<PropertyGroup>
    <!--- ... -->
    <RestoreLockedMode>true</RestoreLockedMode>
    <!--- ... -->
</PropertyGroup>    
```

This property must be set to true as part of build scripts for CI/CD machines where restore should never update an existing lock file instead should fail so that developer can go back to their dev box, run restore which should update the lock file, test, and then check-in to the source control.

## Forcing restore re-evaluation

With this packages lock file, each subsequent restore will skip resovling floating ranges or version ranges to get the latest version from the NuGet feed, instead it will always use the resovled version from the packages lock file. But at times, we may want to re-evaluate existing resolved dependencies and pull the newer versions. For now, there is only `dotnet.exe restore` option to force restore to re-evaluate NuGet dependencies even if `packages.lock.json` file exists to get the newer version.

`dotnet restore --force-evaluate`

There are other equavalent dotnet.exe options to do most of the other stuff as well. See [Repeatable build restore commandline options](https://github.com/NuGet/Home/wiki/Enable-repeatable-package-restore-using-lock-file#restore-command-line-options).