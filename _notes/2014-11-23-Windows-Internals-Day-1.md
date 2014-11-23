---
layout: notes
date: 2014-11-23
---

Windows Internals
=================
### Presented by Pavel Yosifovich

####Recommended Resources
##### Books
-Windows internals
-Others(missed)
#####Web
- www.sysinternals.com
- www.osr.com
- www.osronline.com
- www.microsoft.com/whdc -- specifications, drivers, and APIs

##Course Contents
1. System Architecture - How we access files and devices. How windows works inside
2. Processes & Threads - One of the most important things in windows. Access to data, scheduling, security
3. Memory Management - every OS needs to manage memory; how is this odne in windows and how can we leverage these capabilities?
4. I/O System - How are I/O devices and other devices exposed through windows to applications. How to install them, how drivers work, and how we can call them.
5. Appendix A: Introduction to WinDbg - Debugger used to investigate things in windows. -- Not very friendly -- We will learn the basics throughout class.
6. Appendix B: Kernel Mechanisms - Lots of content we may not have time to cover in class. Many of these topics covered in modules 1, 2, 3

#System Architecture

Module 1

##Windows NT History
- Windows NT 3.1 (Jul 1993)
- Windows NT 3.5 (Sep 1994)
- Windows NT 3.51 (May 1995)
- Windows NT 4.0 (Jul 1996)
- Windows 2000 (December 1999)
- Windows XP (August 2001)
- Windows Server 2003(March 2003)
- Vista (Jan 2007)
- Windows Server 2008 (feb 2008)
- win 7 & 2008 R2 (Oct 2009)
- Windows 8 & Svr 2012 (Oct 2012)
- 8.1 & Server 2012 R2 (oct 2013)
- Windows 10 (Expected mid-2015)
 - Evaluation version available now

Trivia: Why no Windows 9? Nobody knows

Despite this long history, the basic structure of the OS hasn't changed very much.  
Threads, memory management, etc, still all pretty much the same.  
Windows NT design wasn't haphazard; It was intentionally designed the way it is when the project started.  
Consumer versions in 32- and 64-bit versions.  
Starting with Server 2008 R2, Server versions only available in 64 bit  

##Tools
- Windows
  - Task :Manager, Perf Monitor
  - Others
- [SysInternals](www.sysinternals.com)
  - Process explorer, Process monitor, Debug view
  - Many others
  - Still free and actively maintained now that MS acquired SysInternals
- Debugging Tools for Windows - Also free. available in the windows SDK
  -WinDbg, cdb, ntsd, kd
    -MS Uses WinDbg for its own debugging of windows
  -others

##Processes
*What is an OS*
- A program which runs on a computer and runs other programs
- Provides an abstraction of the computer hardware for other programs to use
  - All hardware, logical, and physical devices are abstracted
- Should be generally transparent to the user

-Process
  - A set of resources used to execute a program
- A process consists of:
  - A private virtual address space in which memory can be allocated and used
  - An executable program, referring to an image file on disk which contains the initial code and data to be executed
  - A table of handles to various objects such as files, events, threads, and others
  - A security context, called an access token, used for security checks when accessing shared resources
  - One or more threads that execute code
    - Every process has threads
    - Scheduled to processors
    - Processes are always created with one thread 
      - executes the `main` function
      - Can create new threads if needed
    - A process without any threads is considered dead/useless and the process is killed.
  
A process is a manager -- it doesn't really run; it provides resources for code to execute.  
An executable (.exe) is usually associated with a process. **BUT** it is not what uniquely identifies the process (eg: you can open three instances of notepad.exe, but they don't conflict with each other.)  
Processes are isolated -- if something goes wrong in a process, it may crash but it shouldn't impact other processes running on the same OS.

**Brief demo of Task manager**
Task manager can be shown through `ctrl`+`alt`+`del` menu, or by pressing `ctrl`+`alt`+`esc`  
Shows details of processes, applications, and various other information -- go explore  
**Status** - Running doesn't mean an application is doing anything; just that it is responsive. Without UI activity the thread just waits and doesn't use processor time.  
Status changes to `Not Responding` after 5 seconds of blocking activity on an application.  
Windows Store Applications will go to `suspended` status when the focus moves away from them. This is similar to how mobile applications behave. This model is new in Windows 8; previous versions only had standard Desktop applications which could use resources at any time.  
**handles** - a number which represents some type of object (thread, file, mutex, etc.)

### Sysinternals tools 
-- already on the computers. Also downloadable from [here](http://technet.microsoft.com/en-us/sysinternals/bb545021.aspx)

Start with Process Explorer (procexp.exe)  
Run as administrator - right click or change compatibility mode (in properties) to run as administrator.
-Colors have meaning:
  - Pink: Windows Services
    - Typically running without user intervention, and with high privileges.
    - Examining these processes shows a 'services' tab that displays hosted services
  - Yellow: Hosting .Net CLR (Common Language Runtime - comparable to JVM)
  - Green/Red: Processes being created or killed
    - By default this only shows for 1 second. Can be reconfigured
  - Other colors in Options-> Configure Colors...
- Views:
  - Tree view - Shows which processes spawned other processes
    - Processes in windows do not die when their parent process does
    - Notice some processes (Eg: Explorer) are aligned all the way left.
      - This just means the creating (parent) process is no longer alive -- this is normal;  
      userinit.exe loads explorer and other similar programs and then exits
  - Lower pane (`ctrl`+`L`)
    - Shows either handles or dlls - change with the button up top
  - Find handle or dll
    - find option that will show all the processes holding a file with the user-provided string in its name.
    - When a process is killed all handles and dlls it has open will be freed automatically.
    
_10-minute break_

##Threads



