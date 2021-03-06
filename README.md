# MSBuildStructuredLog
A logger for MSBuild that records a structured representation of executed targets, tasks, property and item values.

[![Build status](https://ci.appveyor.com/api/projects/status/v7vwgphs239i14ya?svg=true)](https://ci.appveyor.com/project/KirillOsenkov/msbuildstructuredlog)
[![NuGet package](https://img.shields.io/nuget/v/Microsoft.Build.Logging.StructuredLogger.svg)](https://nuget.org/packages/Microsoft.Build.Logging.StructuredLogger)

## Install:
https://github.com/KirillOsenkov/MSBuildStructuredLog/releases/download/v1.1.138/MSBuildStructuredLogSetup.exe

The app updates automatically via [Squirrel](https://github.com/Squirrel/Squirrel.Windows) (after launch it checks for updates in background), next launch starts the newly downloaded latest version.

![Screenshot1](/docs/Screenshot1.png)

## Requirements:
 * .NET Framework 4.6
 * MSBuild 14.0 or 15.0
 * Visual Studio 2017 (only needed for development)

## Usage:

The logger is in a single file: `StructuredLogger.dll`. It is available in a NuGet package:
https://www.nuget.org/packages/Microsoft.Build.Logging.StructuredLogger

You can either build your solution yourself and pass the logger:

```
msbuild solution.sln /t:Rebuild /v:diag /noconlog /logger:StructuredLogger,%localappdata%\MSBuildStructuredLogViewer\app-1.1.138\StructuredLogger.dll;1.buildlog
```

or you can build the solution or open an existing log file through the viewer app:

![Screenshot2](/docs/Screenshot2.png)

To use a portable version of the logger (e.g. with the `dotnet msbuild` command) you need a .NET Standard version of `StructuredLogger.dll`, not the .NET Framework (Desktop) version.

Download this NuGet package: https://www.nuget.org/packages/Microsoft.Build.Logging.StructuredLogger/1.1.138
and inside it there's the `lib\netstandard1.5\StructuredLogger.dll`. Try passing that to `dotnet build` like this:
```
dotnet msbuild Some.sln /v:diag /nologo /logger:StructuredLogger,"packages\Microsoft.Build.Logging.StructuredLogger.1.1.138\lib\netstandard1.5\StructuredLogger.dll";"C:\Users\SomeUser\Desktop\structuredlog.buildlog"
```

The logger supports three file formats:

 1. `*.binlog` (official MSBuild binary log format, same as `msbuild.exe /bl`)
 2. `*.xml` (for large human-readable XML logs)
 3. `*.buildlog` (compact binary logs, may run out of memory on very large builds)
 
Depending on which file extension you pass to the logger it will either write XML or binary.
The viewer supports all formats and defaults to binary when saving. The binary log can be up to 200x smaller and faster.

Read more about the log formats here:
https://github.com/KirillOsenkov/MSBuildStructuredLog/wiki/Log-Format

## Features:

 * Displays [double-writes](https://github.com/KirillOsenkov/MSBuildStructuredLog/wiki/Double%20write%20detection) (when files from different sources are written to the same destination during a build, thus causing non-determinism)
 * Displays target dependencies for each target
 * Text search through the entire log
 * Narrow down the search results using the under() clause to only display results under a certain parent.
 * Each node in the tree has a context menu. Ctrl+C to copy an item and the entire subtree to Clipboard as text.
 * Delete to hide nodes from the tree (to get uninteresting stuff out of the way).
 * Open and save log files (ask a friend to record and send you the log which you can then investigate on your machine)
 * Logs can include the source code project files and all imported files used during the build.
 * If a log has embedded files, you can view the list of files, full-text search in all files, and use the Space key (or double-click) on most nodes to view the source code.
 * If MSBuild 15.3 or later was used during the build the log can also display a preprocessed view for each project, with all imported projects inlined.
 * The viewer associates with `*.binlog` and `*.buildlog` so you can double-click a log file in Explorer to open it.

## Differences between StructuredLogger and the new BinaryLogger that is shipped with MSBuild as of 15.3

Starting with Visual Studio 2017 Update 3 MSBuild supports a new `BinaryLogger` documented here:
https://github.com/Microsoft/MSBuild/wiki/Binary-Log

The Structured Log Viewer is able to open the `.binlog` format created by `BinaryLogger` (as well as its own `.buildlog` format). Here are some differences between the `BinaryLogger` and the `StructuredLogger`:

 * `BinaryLogger` (`/bl`) will consume less memory and hopefully not OOM on huge builds, whereas the `StructuredLogger` `.buildlog` format may OOM on very large builds (by design). The /bl `.binlog` format is streaming and should be constant in terms of memory.

 * `StructuredLogger` on the other hand captures the target graph so it is able to show DependsOn list for each target and also order targets slightly better in some cases.

 * `StructuredLogger` is much more compact since it is not streaming and thus able to use string deduplication, reducing the log file up to 10x smaller as a `.binlog` for a Binary Logger.

 * `BinaryLogger` is replayable and you're able to reconstruct text logs of any verbosity out of it. It is hard to reconstruct text logs of conventional format from a StructuredLogger log.

Other than that they're pretty much equivalent and both can be opened in the viewer.

## Investigating problems with MSBuildStructuredLog

Open an issue if you're running into something weird and I can take a look into it. If MSBuildStructuredLog crashes during the build, it will attempt to write the exception call stack to:

```
%localappdata%\Microsoft\MSBuildStructuredLog\LoggerExceptions.txt
```

## MSBuild Resources
 * [https://github.com/Microsoft/msbuild/wiki/MSBuild-Resources](https://github.com/Microsoft/msbuild/wiki/MSBuild-Resources)
 * [https://github.com/Microsoft/msbuild/wiki/MSBuild-Tips-&-Tricks](https://github.com/Microsoft/msbuild/wiki/MSBuild-Tips-&-Tricks)
