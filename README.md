# macOS App structure
Transitioning to macOS from Linux or Windows can feel like walking in a strange new land.
Since Linux is open-source and Windows is well-documented and very popular, macOS can be challenging at times.
In this blogpost, I intend to discuss some of the first things you might notice on macOS - Apps, Apps everywhere!

## Apps vs. processes (tasks?)
Coming from a Windows or Linux background, the concept of Apps might seem weird.
We all know threads are "units of execution" and processes are containers for threads with their own address space -- what more is there to it?
Well, processes are rarely deployed in single files. On both Windows and Linux there are many things code might need to function, some of them are:
- Loadable modules (`.dll`, `.so`). For example, the C runtime library (`msvcr<version>.dll` on Windows, `libc-<version>.so`) as well as other depndencies.
- Resources. For example, on Windows, executable files come in a format called `PE`, which has directories - one of them is the resource directory (even kind of documented [here](https://referencesource.microsoft.com/#System.Deployment/System/Deployment/Application/PEStream.cs,b01be218023fc607,references) that might contain resources (images, strings and others). Resources could also be loaded from disk dynamically, of course.
- Digital signatures. Those are less common on Linux (although they do exist in some form - for example, in [Debian Packages](https://www.debian.org/doc/manuals/securing-debian-manual/deb-pack-sign.en.html)) but are important. On Windows they might exist in the `PE` file itself (read [here](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/authenticode)) or in catalogue files (i.e., externally).
- Configuration. On Linux, those are files (like your trustworthy [.bashrc files](https://linux.die.net/man/1/bash)), and on Windows they split between files (e.g., `xml`, `ini`, `json`) and the [Windows Registry](https://en.wikipedia.org/wiki/Windows_Registry).
- Other executables.

Well, macOS puts heavy emphasis on [Application Bundles](https://en.wikipedia.org/wiki/Bundle_(macOS)). The idea is to package (almost) everything required for the program to run in a directory structure - including resources, localization information, etc.. Of course, not evertything could be nicely packaged (like the C runtime library, for instance) - but it still means that things are *bundled* together nicely - no need to navigate a huge Registry or to read manual pages for obscure configuration file locations. Application bundles are just directories ending with `.app` - even though the UI hides the `.app` extension (and the fact it's a directory).
From an attacker's perspective this is interesting - since an `Application Bundle` can have arbitrary icons and hides the `.app` extension - delivering malware could be achieved by fooling an unsuspecting user to click such an app. For example, think about a `Resume.app` file with a PDF icon.

The directory structure for an `Application Bundle` can be easily examined, obviously by the builtin `Calculator` App:
```shell
jbo@McJbo ~ % cd /System/Applications/Calculator.app
jbo@McJbo Calculator.app % ll
total 0
drwxr-xr-x   3 root  wheel    96 Mar 17 21:34 .
drwxr-xr-x  43 root  wheel  1376 Mar 17 21:34 ..
drwxr-xr-x   9 root  wheel   288 Mar 17 21:34 Contents
jbo@McJbo Calculator.app % cd Contents
jbo@McJbo Contents % ll
total 16
drwxr-xr-x    9 root  wheel   288 Mar 17 21:34 .
drwxr-xr-x    3 root  wheel    96 Mar 17 21:34 ..
-rw-r--r--    1 root  wheel  2147 Mar 17 21:34 Info.plist
drwxr-xr-x    3 root  wheel    96 Mar 17 21:34 MacOS
-rw-r--r--  204 root  wheel     8 Mar 17 21:34 PkgInfo
drwxr-xr-x    4 root  wheel   128 Mar 17 21:34 PlugIns
drwxr-xr-x   54 root  wheel  1728 Mar 17 21:34 Resources
drwxr-xr-x    3 root  wheel    96 Mar 17 21:34 _CodeSignature
-rw-r--r--    1 root  wheel   461 Mar 17 21:34 version.plist
jbo@McJbo Contents % cd MacOS
jbo@McJbo MacOS % ll
total 344
drwxr-xr-x  3 root  wheel      96 Mar 17 21:34 .
drwxr-xr-x  9 root  wheel     288 Mar 17 21:34 ..
-rwxr-xr-x  1 root  wheel  540912 Mar 17 21:34 Calculator
jbo@McJbo MacOS %
```

As you can see, `Calculator.app` is a directory. Under it there is a single item - another directory called `Contents`.
Under `Contents` there are multiple items:
- `Info.plist` - contains metadata on the App. More on that later.
- `MacOS` - contains the main executable of the App (as can be seen in the 3rd directory listing).
- `PkgInfo` - non-mandatory. A binary file that contains package information.
- `PlugIns` - non-mandatory. A directory that might contain plugins for the App. Calculator has two - one for "Basic and Scientific" and one for "Hexadecimal" (I really don't know why they made that separation, nor do I care).
- `Resources` - non-mandatory. As the name suggests, contains resources. You might find several items there, including `.icns` file with Icons, as well as directories with the `.lprroj` suffix that are related to localization.
- `_CodeSignature` - non mandatory. As the name suggests - contains code signing information.
- `version.plist` - non-mandatory, contains verison information.

Note there are very few items that are *officially* required. In fact, we can create our own first App without even compiling anything!
But first we must discuss that `Info.plist` file.

## Property list files
The more you look at macOS, the more you'll find those strange files. Those are nothing more than glorified configuration files.
They will always have a `.plist` extension, which is just a short way of calling their formal name: `Property list` files.
Unfortunately, there are 3 different `plist` formats maintained by Appled:
- An `xml` format, readable by humans.
- A `json` format, not widely used.
- A binary format, will usually be visible by the text `bplist` as magic.

Luckily, there is a utility called `plutil` that supports all formats. To output a `plist` file, simply use `plutil -p`. For example:

```bash
jbo@McJbo Contents % plutil -p Info.plist | head -n 20
{
  "BuildMachineOSBuild" => "22A380007"
  "CFBundleDevelopmentRegion" => "English"
  "CFBundleExecutable" => "Calculator"
  "CFBundleGetInfoString" => "10.14, Copyright © 2000-2018, Apple Inc."
  "CFBundleHelpBookFolder" => "Calculator.help"
  "CFBundleHelpBookName" => "com.apple.Calculator.help"
  "CFBundleIconFile" => "AppIcon"
  "CFBundleIconName" => "AppIcon"
  "CFBundleIdentifier" => "com.apple.calculator"
  "CFBundleInfoDictionaryVersion" => "6.0"
  "CFBundleName" => "Calculator"
  "CFBundlePackageType" => "APPL"
  "CFBundleShortVersionString" => "10.16"
  "CFBundleSignature" => "????"
  "CFBundleSupportedPlatforms" => [
    0 => "MacOSX"
  ]
  "CFBundleVersion" => "223"
  "CTIgnoreUserFonts" => 1
jbo@McJbo Contents %
```

There are also conversion functionalities built into `plutil` - we won't demonstrate those right now.
Apple [documents](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html) several requirements in an App's `Info.plist`, but there are very little fields that are actually mandatory. Here are some interesting fields:
- `CFBundleExecutable` - the name of the main executable, expected to be under the `MacOS` directory.
- `CFBundleIconFile` - the name of the icon file. Non-mandatory.
- `CFBundleIdentifier` - an identifier for the App Bundle. Apple recommends using a reverse DNS notation (e.g. `com.apple.calculator`).
- `CFBundleName` - the bundle name.

With that in mind, we can create our own first awesome app, *without even coding*!
Take a look:
```shell
#!/bin/zsh

# Create Bundle structure
mkdir -p ./MyApp.app/Contents/MacOS

# Create main executable file - a shell script in our case
cat <<EOF > ./MyApp.app/Contents/MacOS/MyApp
#!/bin/zsh
osascript -e 'tell app "Finder" to display dialog "Hello from MyApp!"'
EOF
chmod +x ./MyApp.app/Contents/MacOS/MyApp

# Create the Info.plist file
cat <<EOF > ./MyApp.app/Contents/Info.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>CFBundleExecutable</key>
	<string>MyApp</string>
	<key>CFBundleIdentifier</key>
	<string>com.myapp</string>
	<key>CFBundleName</key>
	<string>MyApp</string>
	<key>CFBundlePackageType</key>
	<string>APPL</string>
</dict>
</plist>
EOF
```

This will create a new app called `MyApp` - clicking it will simply run the `zsh` script (called `MyApp`).
Note it uses `osascript`, which is an [AppleScript](https://developer.apple.com/library/archive/documentation/AppleScript/Conceptual/AppleScriptLangGuide/introduction/ASLR_intro.html) interpreter and a whole can of worms, but it will just show a dialog that says `Hello from MyApp!`. You could write arbitrary code in that `zsh` shell file, obviously. Newer macOS versions might display a prompt asking to allow `zsh` to call `osascript` - we will discuss why it happens in a future post, but note it won't ask after a first approval.

# Launching an App
Launching an App means that a process is still created - obviously the one pointed by `CFBundleExecutable`. What process does it run under? Let us see:
```shell
jbo@McJbo ~ % open -a Calculator
jbo@McJbo ~ % ps -A -j | grep Calculator | grep -v grep
jbo              12067     1 12067      0    1 S      ??    0:00.41 /System/Applications/Calculator.app/Contents/MacOS/Calculator
jbo@McJbo ~ %
```

The `open` command is equivalent to double-clicking the `Calculator` App - it is quite fascinating and we will discuss it soon.
Now that `Calculator` is running we use `ps` to show the running processes. As expected, `/System/Applications/Calculator.app/Contents/MacOS/Calculator` is the process that runsm and it has a PID of `12067`. However, its parent process ID is `1`!

Process ID `1` is macOS is `/sbin/launchd`. It is the "system wide and per-user daemon/agent manager". You can imagine it as `services.exe` (if you come from a Windows background) or `systemd` (if you're familiar with Linux). In addition to managing services (which are called `Launch Agents` and `Launch Daemons` on macOS), it is also the parent of all Applications, which is one of the reasons why getting a meaningful process tree on macOS is challenging.

Interestingly, you could still run the `Calculator` App just as a process by invoking `/System/Applications/Calculator.app/Contents/MacOS/Calculator` directly, but it will not be the child of `launchd`:

```shell
jbo@McJbo ~ % /System/Applications/Calculator.app/Contents/MacOS/Calculator &
[1] 12502
jbo@McJbo ~ % 2023-04-04 15:54:22.987 Calculator[12502:1297087] XType: XTFontStaticRegistry is enabled by Info.plist.

jbo@McJbo ~ % ps -A -j | grep Calculator | grep -v grep
jbo              12502   951 12502      0    1 SN   s000    0:00.31 /System/Applications/Calculator.app/Contents/MacOS/Calculator
jbo@McJbo ~ % echo $$
951
jbo@McJbo ~ %
```

Indeed, `Calculator` runs smoothly, but is now the child process of our terminal.
One thing to note in the behavior we've seen is that *attackers might use it for different purposes*. For example, attackers can easily breaking out of the process tree to evade security tools, as well as abusing logic vulnerabilities (read my [macOS Sandbox escape vulnerability writeup](https://www.microsoft.com/en-us/security/blog/2022/07/13/uncovering-a-macos-app-sandbox-escape-vulnerability-a-deep-dive-into-cve-2022-26706/) if you have time).

There are other interesting responsibilities to `launchd` (read about [LaunchAgents and LaunchDaemons](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html)) but we will not be discussing those for now.

## 
