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

