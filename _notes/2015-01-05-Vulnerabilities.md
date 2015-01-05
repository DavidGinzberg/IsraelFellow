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
  - If proxy settings are set to `auto detect`, it uses WPAD (Web Proxy Auto Discovery) protocol, and sends an NBNS query for WPAD



