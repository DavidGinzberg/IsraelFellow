---
---

#Vulnerabilities  
David Oren


##Agenda
- What is a vulnerability?
- What is an exploit
- Common vulnerability categories
  - Design
  - Implementation
  - Operation & management
- Famous vulnerability case studies
  - Memory corruption: Win svr svc buffer overflow
  - Cryptography: MD5 Hash function collision
  - CPL link files
- Philosophy

## What is a vulnerability
- [Wikipedia Definition](http://en.wikipedia.org/wiki/Vulnerability_(computing))
- "(I) A flaw or weakness in a system's design, implementation, or
      operation and management that could be exploited to violate the
      system's security policy." - [IETF RFC-2828](https://www.ietf.org/rfc/rfc2828.txt)
      
## What is an exploit?
- [Wikipedia Definition](http://en.wikipedia.org/wiki/Exploit_(computer_security))
- A collection of bits causing a vulnerable system to act differently than was planned.

## Common Vulnerability categories
- Implementation
  - Memory Corruption
    - Buffer overflow
    - Use after free
      - Access of a pointer after it has been freed
      - If an attacker can replace a freed function with arbitrary code, this attack is successful
    - [Double free](http://cwe.mitre.org/data/definitions/415.html)
      - After allocating memory, free it, then free it again.
  - Cryptography
    - Using easily factored number for RSA encryption (Implementation vulnerability)
    - Using 24-bit [IV](http://en.wikipedia.org/wiki/Initialization_vector) to initialize [RC4](http://en.wikipedia.org/wiki/RC4) (famous WEP flaw)
    - Reusing the same keys for encrypting different data (in certain cases)
    
    
**Aside:**
###Cryptography Algorithms
- [Stream ciphers](http://en.wikipedia.org/wiki/Stream_cipher)
  - [RC4](http://en.wikipedia.org/wiki/RC4)
    - Invented by Ron Rivest (see below)
- [Block Ciphers](http://en.wikipedia.org/wiki/Block_cipher)
  - [DES](http://en.wikipedia.org/wiki/Data_Encryption_Standard)/ [3DES](http://en.wikipedia.org/wiki/Triple_DES)
  - AES
- Asymmetric ciphers
  - RSA (named for its creators: Rivest, Shamir, Adelman)

  ####Stream Ciphers
  - Given a plaintext...
    - Generate a key as long as the plaintext
      - This is difficult; helps to have a standard
      - For the standard, you take a n-bit key and generate a one-time-key specific to the plaintext
        - Given the same key and same key generator, you will get the same cipher
        - Add an Initialization Vector (IV)

#### Block Ciphers
1. Takes a key and a plaintext
2. Encrypt a block of plaintext using the key
3. Encrypt the next block with a key determined based on the result of the previous block.


## Common Vulnerability Categories (continued)
- Design
  - Autorun mechanism on USB connection
  - Choosing a weak [hash function](http://en.wikipedia.org/wiki/Hash_function)
- Operation & Management
  - Not changing your home router default password
    - worse: leaving the control panel open to connections on the Internet

##Implementation Vulnerability: Buffer Overflow Intro
- Basic concept demonstrated on [Wikipedia](http://en.wikipedia.org/wiki/Stack_buffer_overflow#Exploiting_stack_buffer_overflows):
```C
#include <string.h>
 
void foo (char *bar)
{
   char  c[12];
 
   strcpy(c, bar);  // no bounds checking
}
 
int main (int argc, char **argv)
{
   foo(argv[1]);
}
```


##Buffer overflow case study:  
Conficker & MS08-067 AKA CVE-2008-4250  
Microsoft Server Service remote code execution vulnerability

###Conficker & MS08-067
- Server Service - Networking feature in Windows
- Supports file, print, an dnamed-pipe sharing
- the latter is used for [Remote Procedure Calls(RPC)](http://en.wikipedia.org/wiki/Remote_procedure_call) (http://msdn.microsoft.com/en-us/library/windows/desktop/aa378651%28v=vs.85%29.aspx)
- Usually client-server over USB (SRVSVC)
- [NetprPathCanonicalize](http://msdn.microsoft.com/en-us/library/cc247258.aspx) - canonicalization RPC


**Aside:**
#### RPC - NetprPathCanonicalize -ation
`C:\Windows` :arrow_right: `C/Windows`  
`"C:\Windows\../Program Files"` :arrow_right: `"C/Program Files"`

### Conficker & MS08-067 (cont.)
- NetprPathCanonicalize: PathName (String) argument
- The vuln exists in PathName processing
  - The traversal passes the root directory
- Normal case:
  - `C:\program files\..\Windows` :arrow_right: `c/windows`
- Vulnerable:
  - `/<name>/../../<rest of path>` :arrow_right: `/../<rest of path>`
  - Processes the extra `/..`, and searches the stack for the previous slash and copies the rest of the path to be right after the slash found.
  - Buffer Overflow

## MD5 Collision attack case study:  
Flame & Windows Update MitM 
- Windows updates are almost always enabled
- Flame contained no new/ zero-day vulnerabilities, just previous exploits from Conficker and Stuxnet
- In June 2012 Flame was discovered to use Windows upate mechanism to spread
- How?
  - Usually we get updates from official MS domains
  - Connection is HTTPS (HTTP over SSL)
  - The update file is signed by Microsoft CA
- Step-by-step
  - MS wants to make WU mechanism as generic as possible
    - Whether client is connected directly to the gateway
    - if there is a proxy
    - whether using DNS or [NBNS](http://wiki.wireshark.org/NetBIOS/NBNS)
  - If proxy settings are set to `auto detect`, it uses [WPAD](http://en.wikipedia.org/wiki/Web_Proxy_Autodiscovery_Protocol) (Web Proxy Auto Discovery) protocol, and sends an NBNS query for WPAD
- Flame listens and stores NBNS queries
- if it's WPAD, spoof the response
- Waits for HTTP connections to WU domains (only their hashes are stored) --why?
- if User-agent and URI are one of the preset patterns, responds with corresponding payload
- But update should be signed by MS
- Cryptography flaws:
  - [TSL](http://technet.microsoft.com/en-us/library/cc770371%28v=ws.10%29.aspx) (terminal services licensing) - Licensed Certificate generation for terminal servers
    - Starting with Vista - this certificate is unable to sign code
  -Flame authors managed to
    - obtain one of these certificates
    - Add code signing privileges
    - Extra manipulations so that MD5 is still the same as the original

- What did we have
  - Flame sniffing NBNS to hijack WU requests
  - Update signed by MS that installs Flame by manipulatin TSL using MD5 collision attack
  - Have a cert with chain originating at MS, with code signing privileges. If this isn't a 0-day, what is?
  - Alex Gostev (Kaspersky) said: "It actually looks more like a 'god mode' cheat code" - Mentioned in an interview with [Alex Gostev](http://blog.kaspersky.com/a-social-media-interview-with-alex-gostev/)
  - (Two URLS for analysis of the exploit will be provided in the deck)

## Case Study:  
Stuxnet LNK vulnerability

- Windows contains Control Panel applications
  - Enable you to change settings in your computer - netwrokd, display, date, time etc
- Their extension is CPL
  - ncpa.cpl (network)
  - Appwiz.cpl (program installation)
- Located in `C:\Windows\System32`
- Windows enables creating shortcuts to applications: LNK files
- LNK files can also link to CPL applications
- When you open a folder with Windows Explorer, it tries to display the icon for each of the files and folders in that folder
  - This also applies to LNK files
    - if a LNK file points to a CPL file, windows actually calls the DLL for that CPL in order to call the LoadImage method which displays the icon (also LoadLibraryW)
    - This allows code execution
-Stuxnet created a CPL-like LNK file, only it referred to the ~WRT4141.tmp - a DLL that loads the main Stuxnet DLL named ~WTR4132.tmp (on the same USB flash drive)

- What's left is just to:
  - Hide the evidence
  - Drop Stuxnet on the PC that the USB storage device is connected to
  - Continue to chase after the Iranian Uranium enrichment control systems and save the world....

  
- what do we have?
  - No memory corruption, buffer overflows, etc
  - No screenshots from IDA/Immunity/WinDBG
  -Just not-security-oriented programmer, who thought that loading a DLL without any user interaction except viewing a folder is a good idea

## A Little Philosophy
- Why are there always more and more vulnerabilities?
- Defender-Attacker axiom
- What are the disadvantages of the presented case studies?
- Vulnerabilities are just everywhere
  - Mobile
  - Cars
  - Smart TV
  - Microwave (Really? Really.)
- Systems are not vulnerable, humans are.