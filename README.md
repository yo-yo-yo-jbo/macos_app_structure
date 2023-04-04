# macOS App structure
Transitioning to macOS from Linux or Windows can feel like walking in a strange new land.
Since Linux is open-source and Windows is well-documented and very popular, macOS can be challenging at times.
In this blogpost, I intend to discuss some of the first things you might notice on macOS - Apps, Apps everywhere!

## Apps vs. processes (tasks?)
Coming from a Windows or Linux background, the concept of Apps might seem weird.
We all know threads are "units of execution" and processes are containers for threads with their own address space -- what more is there to it?
Well, processes are rarely deployed in single files. On both Windows and Linux there are many things code might need to function, some of them are:
- Loadable modules (*.dll, *.so). For example, the C runtime library (`msvcr<version>.dll` on Windows, `libc-<version>.so`) as well as other depndencies.
- Resources. For example, on Windows, executable files come in a format called `PE`, which has directories - one of them is the resource directory that might contain resources (images, strings and others).
