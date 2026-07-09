---
layout: post
title: dev_setup_windows
categories: dev
tags:
- dev
- windows
- setup
author: nicoliboli
description: environment setup for windows dev
---

visual studio is the best way to go with windows dev. this is the version i'm working with

![visual_studio_version](/assets/img/visual_studio_version.png)

helpful shortcuts
- CTRL+SHIFT+B: build a solution
- CTL+B: build a project (helpful if your solution has more than one project)
- F5: start debugging
- CTRL+F5 run w/o debugging
- F9: set breakpoint on current line
- ALT+F9 (wait a sec) C: set conditional breakpoint on current line
- F10: step over
- F11: step into
- F12: go to definition
- ALT+F12: peek definition 
- CTRL+F12: go to declaration
- F1: open MSDN


# breakpoints

- set in the code
- [DebugBreak()](https://learn.microsoft.com/en-us/windows/win32/api/debugapi/nf-debugapi-debugbreak) from the header file `debugapi.h`
- `__debugbreak` that you can [use if symbols aren't available](https://learn.microsoft.com/en-us/visualstudio/debugger/debugbreak-and-debugbreak?view=vs-2022) b/c its intrinsic to the compiler (i.e. not a function from a library)
- `__asm int 3` if you want the straight assembly that will trigger an interrupt as the software breakpoint

some [good info](https://anti-debug.checkpoint.com/techniques/assembly.html) about how breakpoints generally work and techniques that malware uses to determine if the program is being debugged

# dependencies

a couple things to check if a solution has multiple projects and one of them is a library that the others may want access to. for this example i will have a solution called `HelloWorld` with two projects: `HelloFriend` and `GoodbyeFriend`. the `HelloFriend` functions as the startup project and will compile the `exe`, while the `GoodbyeFriend` compiles to a library that `HelloFriend` uses functions from.

1. right click on `HelloFriend` project and select properties. c/c++ --> general --> additional include directories. put the path to the project you want to include in there. this way it knows where to look for the header and source files. for this example, you can use the macro `$(SolutionDir)GoodbyeFriend`
2. in that same properties windows. linker --> input --> additional dependencies. put the path to the library that the project you're pulling from produces. for this example you can use macro `$(OutputPath)GoodbyeFriend.lib`
3. right click on the solution and select properties. common properties --> project dependencies. for this example, select `HelloFriend` in the projects dropdown and check the box for `GoodbyeFriend` in the depends on list
4. check that `GoodbyeFriend` is configured as a library and that `HelloFriend` is set as the startup project before you build

# cmd line args

right click project --> properties. debugging --> command arguments