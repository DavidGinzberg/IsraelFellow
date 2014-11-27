---
---

*beginning notes midway through*

##Virtual Address Space Layout
- Each process sees its own private address space
- System space is part of the tentire address space that is visible 
  - But not accessible by user mode code
- The layout depends on the "bitness" (Endianness?) of the OS and the specific process

**Note:** If the operating system is configured with a certain flag, Kernel memory space can be halved (down to 1GB) allowing 3GB addr space per process (in 32-bit systems)  
This only applies to processes which were liked using the flag `/LARGEADDRESSAWARE`  
On 64-bit systems, 32-bit processes (running on WoW64) with this flag can get up to 4GB  of virtual memory  
This is because some psychotic programmers use the extra 1 bit in 31-bit addresses (for 2GB) to store some 1-bit information, and then mask it out any time they try to access addresses.

##Virtual Address Translation
- Hardware translates each virtual address to a physical address

##Page File(s)
- Backup storage for writable, non-shareable committed memory
  - Up to 16 page files supported
  - On different partitions
  - initial size and maximum size can be set
  - named `PageFile.Sys` on disk (root partition)
  - Created contiguous on boot
  - Initial value should be maximum of normal usage
  - If you have more than one physical disk (not just partitions) having multiple pagefiles can be beneficial because it allows parallel pagefile accesses.
  
  
## Working set
- The amount of memory currently used (in physical memory)
- Process Working Set
  - The subset of the process's committed memory that resides in physical memory
- System working set
  - The subset of system memory that resides in physical memory
- Systems with terminal services
  - Some kernel memory is on a per-session basis and as such working set is as well
- Demand Paging
  - When a page is needed from disk, more than one is read at a time to reduce I/O
  
  
###Page frame number database
- PFN database describes the state of all physical pages
- Valid PTEs point to entries in the PFN database