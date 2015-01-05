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
- Basic concept demonstrated on [Wikipedai](http://en.wikipedia.org/wiki/Stack_buffer_overflow#Exploiting_stack_buffer_overflows):
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
