---
layout: notes
date: 2014-11-24
---

##Function Call Flow
- Call fread -- application  
  :down_arrow:
- call ReadFile -- Msvcrt.dll  
  :down_arrow:
- call NtReadFile -- Kernel32.DLL  
  :down_arrow:
-sysenter/syscall -- NtDll.DLL  
  :down_arrow:
**User Mode**
---
**Kernel Mode**
  :down_arrow:
- call NtReadFile -- NtOskrnl.EXE  
  :down_arrow:
- NtReadFile: call driver -- NtOskrnl.EXE  
  :down_arrow:
- initiate I/O return to caller -- driver.sys


Kernel name explained:  
"6009 x64 Multiprocessor Free"  
- x64 -- 64-bit kernel
- Multiprocessor -- Kernel includes code to synchronize processors. These can be optimized out for (old) single-processor systems
- Free -- optimized kernel (aka: `release`); alternative is "checked" or `debug'

###System Service Table(s)
-2 built-in, up to 4 supported
  - bits 12-13 select the table
  - Lower...
  ...
  
### Executive
- Upper layer of NtOsKrnl.exe
- functions
  - System svc functions callalble from user mode
    -Accessed through NtDll.dll
    - Most can be called from Windows API
  - Callable by dev drvrs and doc'd in the WDK
  - Exported buta undocumented for use by NtOsKrnl.exe
  - Internal functions
####Executive components
- Config mgr
  -Manages system registry
-Object manager
  - Mg obj created by kernel or user code
- Process &thread manager
  - Creates and terminates processes and threads
  -underlying support provided by lower-layer kernel
-Security reference monitor (SRM)
-I/O mgr
-Plug & Play manager
 - Handles hardware ID, Enumeration, resource allocation ;and loading of appropriate drivers
- Power manager
  -Coordinates power events and generates I/O requests to drivers based on state changes
- Cache manager
  - Manages caching for file based I/O
- Memory Manager
  - Implements virtual memory handling for use by other system components
  
###Kernel
- Lower layer of kernel
- Implements all low level activity such as interrupt dispatching thread scheduling and processor synchronization
- Implements a set of kernel objects
  - Control objects
    - Controlling various os functions eg: APC, DPC
  - Dispatcher objects
    - Have synchronization capabilities eg: mutex, event
    - Executive adds management over these primitive objects Eg: Security handle management
###Hardware Abstraction Layer HAL
- Isolates kernel and device drivers from underlying hardware platform
  - eg: interrupt controller
- Allows easy access to hardware by device drivers
- Various HALs are on the windows install media
  - appropriate one is copied to System 32 directory and always named Hal.Dll
  MS offers a 'HAL development kit' that you can purchase from them -- with certain agreements (NDA?)
### Device drivers
- Loadable kernel modules (usually with SYS extension) 
- The only documented way to add code that runs in the kernel
- Types (partial list):
  - Hardware device drivers - manage a physical device
  - File system device drivers - Translate file I/O to device specific requests
  - Protocol drivers - Implement a specific network protocol (TCP/IP, IPX/SPX, etc.)
  - Software only driver (non-Plug & Play driver), simply for the ability to execute code in kernel mode
- potentially the ultimate virus, but they must be signed by a well-known authority (or forged?)
### System Processes
- idle process -- 
<enum headers>

#### Idle Process
- visible in task manger
- id is 0
  - Not really a process
- accounts for idle time on the CPU
- always has N threads where N= number of cores
#### System process
- No exe associated
- also in TM processes list
- always has PID of 4
- Represents the kernel address space and everything happening there
  - memory allocated
  - kernel threads
  - resources used by kernel
  - Upper 4TB/2GB (64/32-bit) in memory
  - most of the threads seen are created by the kernel; some may be from device drivers
- Hosts sytem threads
  - execute code in kernel space only
  - created using the `PsCreateSystemThread` kernel API (doc'd in WDK)
#### Session Manager
- Running the image `\windows\system32\smss.exe`
- first user mode process created by the system
- Main tasks:
  - Create sys env vars
  - launch subsys procs (normally just csrss.exe;)
  - Launches itself in other sessions
    - instance loads WINLOGON and CSRSS in that session
    - then terminates
- Finally
  - Waits forever for csrss.exe instances to terminate
  - killing csrss or smss causes BSOD
#### Winlogon
- Running the image `\windows\system32\winlogon.exe`
- handles interactive logons and logoffs
- if terminated, logs off user session
- notified of user request by Secure Attention Sequence (SAS), typically `Ctrl`+`Alt`+`Del`
- creates two desktops:
  - default desktop (what you see)
  - winlogon desktop -- the screen you see when you hit `Ctrl`+`Alt`+`Del`
  - There is also the screensaver desktop (maybe)
  - sysinternls has desktops.exe program to create virtual desktops
- Desktops are part of a "Window Station" object, which belongs to a session object and also contains "global" -- the clipboard
  - Important that these are different sessions so that users don't end up sharing a clipboard during concurrent use.
- authenticates user by presenting username/password dialog (through LogonUI.exe)
  - can be replaced by other authentication providers such as fingerprint ID
#### LSASS
- Running the image `\windows\system32\Lsass.exe`
- Calls the appropriate authentication package
  - local authentication
  - domain authentication
- Upon successful authentication, creates a token representing the user's security profile
- returns information to Winlogon
- info is encrypted in transit
#### Service Control Manager (SCM)
- runs image `\windows\system32\services.exe`
- Responsible for communicating (start/stop/interact) with windows services -- often early before the user has logged in
- runs in special accounts
- Services
  - Similar to UNIX "daemon processes"
  - ...
###Summary
- User mode code has limited access to the system
- kernel mode has full and total access to all system resources
- applications run under a specific subsystem
- the primary subsystem is the Windows subsystem


----

#Processes and Threads

**Module 2**

##Process
- Management or containment object
  - owns
    - Private virtual address space (2GB/3GB on 32-bit; 8TB on 64-bit)
    - working set (Physical memory owned by process)
    - private handle table to kernel objects
      - handles are the way to get to a particular object
    - Access token
      - object indicating security context of the process
      - threads run with this particular security context by default
  - Has a priority class (from Win32)
    - Affects all threads running in that process
      - threads can only be within a certain offset of the base priority set here
  - Basic creation functions: `CreateProcess`, `CreateProcessAsUser`S
  - Terminated when any of the following occurs:
    - All threads in the process terminate
      - Kernel is responsible for freeing all memory; nothing remains or leaks
    - One of the threads calls `ExitProcess` (Win32)
    - Killed with `TerminateProcess` (Win32) -- called by TM when you end a process
### Process creation
- Flow of process creation  
  - Open image file
  - Create kernel Executive Process object
  - create initial thread
  - create kernel Executive Thread object
  - notify CSRSS of new process and thread
  - Complete process and thread initialization
    - load required DLLs and Initialize
    - `DllMain` function called with `DLL_PROCESS_ATTACH` reason
  - Start execution of main entry point (main / WinMain)
 **note:** any process that takes a string is actually a macro with two versions for unicode and ascii. always use the unicode version (prefix string literals with `L`)
#### Default DLL Search Paths
- Same logic as used by `LoadLibrary`
  - Directory of executable
  - System directory (`GetSystemDirectory`, eg: `c:\Windows\System32`)
  - The 16-bit `System` directory (eg: `c:\Windows\System`)
  - The `Windows` directory (`GetWindowsDirectory`, eg: `c:\Windows`)
  - The current directory of the process
  - Directories specified in the `PATH` environment variable
- If safe DLL search mode is not active (XP SP2+) or Windows version is XP SP1 or lower, the current directory is the second item in the above list
  
  
#### Creating Process Example

```C
STARTUPINFO si = { sizeof(si) };
PROCESS_INFORMATION pi;
TCHAR name[] = _T("Notepad.exe");
BOOL created = CreateProcess(0, name, 0, 0, FALSE, 0, 0, 0, &si, &pi);
if(created) {
    printf("Process started... pid=%d tid=%d\n",
    pi.dwProcessId, pi.dwThreadId);
    DWORD rv = WaitForSingleObject(pi.hProcess, 10000);
    if(rv == WAIT_TIMEOUT)
        printf("Process still running...\n");
    else
        printf("Process terminated.\n");
    CloseHandle(pi.hProcess);
    CloseHandle(pi.hThread);
}
else
printf("Failed to start process (error=%d)\n", GetLastError());
```

#### Access Tokens
- Access token
  - Security context of a process or thread
  - A process runs under an access token with the security settings of the specific user
    - `CreateProcess` (win32) uses the logged on user
    - `CreateProcessAsUser` (Win32) uses whatever token is specified in the call (assuming a token for that user can be obtained)
  - A thread by default runs under the access token of its process
    - Can assume another token temporarily by using _impersonation_

###Thread
- Instance of a function executing code
  - Owns
    - Context (registers, etc.), 2 stacks (user mode and kernel mode)
      -Why: A thread running in kernel mode needs to read its arguments from the stack  
      If the arguments are on the user stack, another thread can garble or free the stack frame needed  
      This could cause a BSOD, which should never be possible for a user thread to cause  
      instead, parameters are copied to the kernel mode stack when the syscall occurs, and this prevents other user-mode code from corrupting the data passed to the kernel  
      **Note:** Heap space is also different between kernel- and user-modes
    - Optionally, message queue and Windows
    - Optional security token
  - Scheduling state
    - Priority (0-31)
      - technically only 1-31
      - 0 is saved for a special "zero-paged" thread
    - State (Ready, Wait, Running)
      - Ready : thread wants to run but all processors are currently executing other threads
      - Running: currently running on a processor
      - waiting: does not want to run because it is currently waiting on something
    - Current access mode (user or kernel)
  - Basic creation function: `CreateThread` (Win32)
    - Creates a separate thread of execution in the same process
    - will run concurrently with other threads and fight for cpu resources
  - Destroyed when:
    - Thread function returns (Win32)
    - The thread calls `ExitThread` (Win32)
    - Terminated with `TerminateThread` (Win32)
    
####Creating Thread Example
```C
DWORD WINAPI MyThread(PVOID param);
void _tmain(int argc, _TCHAR* argv[]) {
    DWORD id;
    HANDLE hThread = CreateThread(0, 0, MyThread, NULL, 0, &id);
    printf("Thread %d free as a thread...\n", GetCurrentThreadId());
    WaitForSingleObject(hThread, INFINITE);
    DWORD code;
    GetExitCodeThread(hThread, &code);
    printf("Thread finished. result=%d\n", code);
    CloseHandle(hThread);
}
DWORD WINAPI MyThread(PVOID param) {
    printf("Thread started id=%d\n", GetCurrentThreadId());
    for(int i = 1; i <= 10; i++) {
        printf("%d\n", i);
        Sleep(::rand() % 2000);
    }
    return 10;
}
```
    
#### Thread Stacks
- Every user mode thread has two stacks
  - In kernel space (12K (x86), 24K (x64))
    - Resides in physical memory (most of the time)
  - In user space (may be large)
    - By default 1MB is reserved, 64KB committed
    - A guard page is placed just below the last committed page so that the stack can grow
    - Can change the initial size
      - Using linker settings as new defaults
      - On a thread by thread basis in the call to `CreateThread` / `CreateRemoteThread(Ex)`
      - Can specify a new committed or reserved size, but not both
      - Committed is assumed, unless the flag `STACK_SIZE_PARAM_IS_A_RESERVATION` is used

#### Thread Priorities
- Thread priorities are between 1 and 31 (31 being the highest)
- Priority 0 is reserved for the zero page thread
- The Windows API mandates thread priority be based on a process priority class (base priority)
  - Base priority for processes can be shown in Task Manager
- A thread's priority can be changed around the base priority
- APIs (Win32)
  - `SetPriorityClass` - Changing process base priority
  - `SetThreadPriority` - Change the thread priority offset from the parent's base priority
    - Default value is 8
- API (kernel)
  - `KeSetPriorityThread` - Change thread priority to some absolute value
- Win32 view
  - Process has a priority class: Idle(4), Below Normal(6), Normal(8), Above Normal (10), High(13), Realtime(24)
  - Thread priority is an offset from that base priority (-2, -1, 0, 1, 2 plus 2 special levels)
    - This is a win32 constraint
    - More special levels for Real Time range
- Kernel View
  - Thread priority is an absolute value (0-31)
    - the scheduler does not care about process
    
####Thread Scheduling (Single Processor)
- Priority based, preemptive, time-sliced
  - Highest priority thread runs first
  - If time slice (quantum) elapses, and there is another thread with the same priority in the Ready state, it runs
    - Otherwise the same thread runs again
  - If thread A runs and thread B (with a higher priority) receives something it waited upon (message, kernel object signaling, etc.), thread A is preempted and thread B becomes the Running thread
- Voluntary switch
  - A thread entering a wait state is dropped from the scheduler's Ready list
- Typical time slice is 30 ms on client, 180 ms on server
- On a MP system with n logical processors, n concurrent threads may be running
**Note:** priorities 16-32 are realtime, and windows will not change them.  
Priorities 1-15 are dynamic. If a thread is in Waiting status for enough time (~4s) windows will boost its priority to give it a better chance to run (client only)  
UI threads automatically get a priority boost (when in foreground or always?)