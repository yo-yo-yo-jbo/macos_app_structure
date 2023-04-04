# macOS App structure
Transitioning to macOS from Linux or Windows can feel like walking in a strange new land.
Since Linux is open-source and Windows is well-documented and very popular, macOS can be challenging at times.
In this blogpost, I intend to discuss some of the first things you might notice on macOS - Apps, Apps everywhere!

## Apps vs. processes (tasks?)
Coming from a Windows or Linux background, the concept of Apps might seem weird.
We all know threads are "units of execution" and processes are containers for threads with their own address space -- what more is there to it?
Well, processes are rarely deployed in single files. On both Windows and Linux there are many things code might need to function, some of them are:
- Loadable modules (*.dll, *.so). For example, the C runtime library (`msvcr<version>.dll` on Windows, `libc-<version>.so`) as well as other depndencies.
- Resources. For example, on Windows, executable files come in a format called `PE`, which has directories - one of them is the resource directory (even kind of documented [here](https://referencesource.microsoft.com/#System.Deployment/System/Deployment/Application/PEStream.cs,b01be218023fc607,references) that might contain resources (images, strings and others). Resources could also be loaded from disk dynamically, of course.
- Digital signatures. Those are less common on Linux (although they do exist in some form - for example, in [Debian Packages](https://www.debian.org/doc/manuals/securing-debian-manual/deb-pack-sign.en.html)) but are important. On Windows they might exist in the `PE` file itself (read [here](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/authenticode)) or in catalogue files (i.e., externally).
- Configuration. On Linux, those are files (like your trustworthy [.bashrc files](https://linux.die.net/man/1/bash)), and on Windows they split between files (e.g., `XML`, `INI`, `JSON`) and the [Windows Registry](https://en.wikipedia.org/wiki/Windows_Registry).
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


