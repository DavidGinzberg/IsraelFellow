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
- Thread
  - Has attributes
  - Entity that is scheduled by the kernel to execute code
- A thread contains:
  -The state of the CPU registers
  - Current access mode (user mode or kernel mode)
  - Two stacks, one in user mode and one in kernel mode
  - A private storage area, called Thread Local Storage (TLS)
    - Used, for example, to store `errno` global variable in C - Stores error code of last I/O operation
      - Due to multithreading, this could be the wrong value if another thread also called an I/O op
      - `errno` now has become a macro that is implemented using TLS. Each thread gets it own `errno` in its TLS and accesses it with a known index
      - Maintains source compatibility while only requiring compiler changes for newer, multi-threaded, systems
  - Optional Security token
  - Optional message queue and Windows the thread creates
    - By default windows assumes a newly created thread will be a worker thread.
    - If the thread creates a window, it becomes a UI thread and is assigned a message queue
    - Thread now has the responsibility to do "message pumping" -- reading the messages in the queue and processing them
    - Trade-off: more threads can mean more robustness to recover when one thread crashes, but this requires more resources  
    (Firefox vs chrome example: Firefox uses one thread and if any tab locks up, the browser dies. Chrome uses a thread for each tab and can recover when just one tab dies... in theory)
  - A priority, used in thread scheduling
    - Helps OS to decide which threads will run at what time when there are more threads than cores
    - Priority ranges from 1 to 31 -- 31 is the highest
    - If #threads is less than #cores, priority doesn't matter
  - A state: running, ready, waiting
    - Running: The thread is currently executing code on one of the processors
    - Ready: The thread wants to run but all cores are currently occupied executing other threads
    - Waiting: Thread does not need to run right now because it is waiting for something (a message, operation, or something else)
    
###User Mode vs. Kernel Mode
- Thread Access Modes
- User Mode
  - Allow access to non-operating system code & Data only
  - No access to the hardware
  - Protects user applications from crashing the system
    - User-mode threads can request services such as files or devices using windows API calls - Kernel checks whether it can execute the request
    - This is done in the same thread, which switches to kernel mode, attempts to execute the API call, and then reverts to user mode
- Kernel Mode
  - Privileged mode for use by the kernel and device drivers only
    - A device driver is 'the ultimate virus' -- if you can get a custom one onto a system, you can do anything
  - Allows access to all system resources
  - Can potentially crash the system
    - Exceptions in kernel mode cause BSOD

## Virtual Memory
- Each process "sees" a flat, linear memory
  - By default on 32-bit systems this is 2GB of memory
- Internally, virtual memory may be mapped to physical memory, but may also be stored on disk
  - There is a mapping between the virtual memory and the physical memory chunk that it represents
  - Pagefile - used to allocate more memory than is available on the system.
    - Virtual memory that hasn't been accessed for a while is written (swapped) to the pagefile to free up that block of physical memory for other processes.
    - By default pages are 4Kb each
  - Processes can access memory regardless of where it actually resides
    - The memory manager handles mapping of virtual to physical pages
    - This is a normally transparent process
    - Processes cannot (and need not) know the actual physical address of a given address in virtual memory - and they can't access the memory of other processes
    
### Virtual Memory Layout

x86 (32-bit)
- Each process sees 2Gb of user process space
- 2^32 bits = 4Gb. Why only half of it? This is because there is 2Gb System space
  - This address space (system) is inaccessible to user-mode threads -- attempting to address it raises an exception
x64 (64-bit)
- 8192Gb (8TB) user process space
- wow64 - "Windows on Windows" -- allows 32-bit applications to run on 64-bit systems
  - These processes still get the 2Gb they are used to - allows the program to behave as it normally would 
- System space is also 8TB "No kernel in the world that uses that much space. That wouldn't be a kernel; it would be a monster!"
- Actual address space is 2^64 = 16EB
  - Current bottleneck is the only 48-bit-wide address bus on current processors -- allows 2^48 addresses at most

## Objects and Handles
- Objects are runtime instances of static structures (object type)
  - Examples: Processes, mutex, event, desktop, file
- Reside in system memory space
- Kernel code can obtain direct pointer to an object
- User mode can only obtain a handle to an object
  - Shields user code from directly accessing an object
  - Handles are process-relative
- Objects are reference counted
  - Object may have other handles pointing to it and will not close/be destroyed until all of them are released
- The Object Manager is the entity responsible for creating, obtaining, and otherwise manipulating objects
  - Has lists of objects, security descriptors, handles to objects, etc.

## The Registry
- Global hierarchical repository of data
- Machine wide data as well as user specific data
- Persistant as well as volatile data
- Also "contains" performance data
  - Not really in the registry
  - But the registry API is used to query this data.
- Main hives:
  - HKEY_LOCAL_MACHINE (HKLM) - Machine-wide settings
    - Only the system account can see the SECURITY sub-key -- unless you change the permissions
  - HKEY_CURRENT_USER (HKCU)- User-specific settings
    - effectively just a clone (symlink) of one of the entries under HKEY_USERS
  - HKEY_CURRENT_CONFIG - symlink to HKLM/HARDWARE/CONFIG
  - HKEY_CLASSES_ROOT - symlink to two other places
    - contains file type info for windows explorer ("open with" options)
    - Also contains COM (component Object Model) GUID info for objects

## Windows NT Design Goals
- Separate address space per process
  - One proc can't (easily) corrupt another's memory
- Protected Kernel
  - User mode applications can't crash kernel
- Preemptive multitasking and multithreading
- Multiprocessing support
- Internationalization support using Unicode
  - Set of standards for character encoding
  - Windows NT designed to work with UTF-16 -- 16-bits per character ~65k characters
  - not UTF-8 because each character may be a different size and accessing characters by index requires string traversal
- Security throughout the system
  - Can provide a security descriptor to every object in windows
- Integrated networking
- Powerful File system (NTFS)
  - Supports protection, compression, and encryption
  - does not require code to be written differently for encrypting or compressing files
- Run most 16-bit Windows and DOS apps
  - On 32-bit systems
  - Some programs won't work -- very low-level access was allowed to programs in the early days of Windows, but not later
- Run POSIX 1003.1 and OS/2 applications
  - Originally conceived to run mostly OS/2 applications
    - OS/2 subsystem originally planned as the main subsystem in windows -- changed direction when the windows subsystem became popular
  - Can still run POSIX programs in windows under the POSIX view/subsystem (see [here](http://stackoverflow.com/questions/20339358/creating-posix-applications-in-windows-7))
    - This means some file naming conventions from POSIX (eg: case sensitivity) may work -- but this is not allowed by the Windows API
- Portable across processors and platforms
- Be a great client as well as server platform


---

##Windows Editions
- Windows XP Home
- Windows Professional (2000, XP), Vista, 7, 8.x
  - Main desktop (client) OS
- Windows Server Standard, Advanced, Datacenter editions ( Windows 2000, 2003, 2008, 2008 R2 etc)
- ...

Windows Numeric Versions
- Windows NT 4 (4.0)
- Windows 2000 (5.0)
- Windows XP (5.1)
- Windows Server 2003, 2003 R2 (5.2)
- Windows Vista, Server 2008 (6.0)
- Windows 7, Server 2008 R2 (6.1)
- Windows 8, Server 2012 (6.2)
- Windows 8.1, Server 2012 R2 (6.3)
- Windows 10 (6.4)
- *These versions obtained using `GetVersionEx`

*Reasons for Vista failure*
- Kernel was actually very good
- Mostly marketing failures
  - 6 year gap since XP - people were too familiar with XP and didn't want to switch
  - Resource hungry -- didn't run well on existing machines
  - Device support for some hardware was not up to par
  - the name 'Vista' set high expectations
  - UAC behavior was unexpected to many programs and caused false failures

###Professional Vs. Server
- Same core system files
- Differences
  -Number of processors supported
  - Maximum amount of RAM thant can be used
  - Max of concurrent network connections supported for file and print sharing (10 on professional)
  - Some services only appear in Server versions
  - Other system policies and default settings (eg: thread quantum -- number of ms a thread can wait on another thread of equal priority)
- To query OS type ...

##General Architecture overview

User Mode
---
Kernel mode

**User Mode:**  
- System Processes
- Services
- Environment Subsystem
- User Applications
- User applications
- Subsystem DLLs
- NTDLL.DLL -- largely undocumented lower-layer call. Almost any documentation on this is likely written by reverse-engineers and may not be consistent across versions
---
**Kernel Mode:**
- Executive dispatcher
- Kernel & Device Drivers (The only official way to add functionality to the kernel)
- Graphics (win32k)
- Hardware Abstraction Layer (HAL)(I'm sorry, I can't let you do that.)

###Kernel Mode Components
- Hardware abstraction Layer (HAL)
  - Isolates the kernel and device drivers from platform specific issues
- Kernel
  - Thread scheduling, interrupt & exception dispatching, multiprocessor support, synchronization primitives
- Device Drivers
  - Loadable kernel modules that handle I/O requests for hardware devices and buses
- Executive
  - Virtual memory manager, object manager, security, IPC, Plug & play, power manager, configuration manager
- Win32K.SYS
  - Windows subsystem kernel component
  - handles UI and graphics
  
### User Mode Components
- User apps
- System processes
- svcs
- Subsystem procs
- Subsystem DLLs
- NTDLL.DLL

### Core System Files
- Ntoskrnl.exe
  - Executive and kernel
  - Original is NtOsKrnl.Exe (single CPU) or NtKrnlMp.Exe (multi-CPU)
- NtKrnlPa.exe
  - Executive and kernel (32-bit) with support for Physical Address Extension (PAE)
  - Original is NtKrnlPa.Exe (single CPU) or NtKrPaMp.Exe (multi CPU)
- Hal.dll
  - Hardware Abstraction Layer
- Win32k.sys
  - Kernel component of the Win32/Win64 subsystem
- NtDll.dll
  - System support routines and Native API dispatcher to executive services
- Kernel32.dll, user32.dll, gdi32.dll, advapi32.dll
  - Core windows subsystem DLLs
-CSRSS.exe (Client Server Runtime SubSystem)
  - The Win32/Win64 subsystem process
  
  
**Note:** C:\Windows\system32 contains 64-bit components (on 64-bit systems) and C:\Windows\system64 contains 32-bit components  
32-bit processes running on 64-bit systems that request system32 will get the contents of system64 due to FS redirection  
On a 32-bit system system32 just contains the 32-bit components (essentially, system32 contains the native components on whatever system it resides)

###Symmetric Multiprocessing
- SMP
  - All CPUs are the same and share main memory and have equal access to peripheral devices (no master/slave)
- Basic architecture supports up to 32/64 CPUs
  - Win 7 64 bit & 2008 support for up to 256 cores
  - Windows 8 /2012 supports up to 640 cores
- Actual number of CPUs determined by licensing and product type
  - Multiple cores do not count toward this limit.
Symmetric -- All processors can execute any type of code -- no discrimination between kernel/user mode processors

###The Windows API
- Application Programming Interface for all Windows versions
- Documented in the Windows SDK (Formerly Platform SDK)
- Each version implements a different subset of the API
- Now collectively called the windows API
  - Previously referred to as the "Win32 API" (name still used today)
- Contains functions in the following areas
  - Base services
  - user interface services
  - component services
  - graphics and multimedia
  - messaging and collaboration
  - networking
- API Style
  - Flat C functions
  - COM (Component Object Model) -- object oriented framework to implement a set of interfaces; too big a subject to cover here
- The new Windows Runtime (WinRT)


###Subsystem DLLs
- Every image belongs to exactly one subsystem
  - Value stored in image PE header
    - Can view with [Dependency Walker (Depends.exe)](http://www.dependencywalker.com/)
    
### The native API
- Impllemented by NTDLL.DLL





