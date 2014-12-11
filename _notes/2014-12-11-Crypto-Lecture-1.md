---
---

#Cryptography
A lecture by Ben Herzog

##What's Cryptography?
A story about WWII German practice of harassing various ships except American ships:

Germany sent a communication to Mexico asking them to declare war on the US if it got involved in the war.  
Because the undersea cable had been cut by the British, the German message was relayed through Europe and the UK and was successfully analyzed.

###Cryptography Definition:

[See wikipedia](http://en.wikipedia.org/wiki/Cryptography).

###Fundamental Aspects of Infosec

- Confidentiality
  - Protect info from disclosure
- Integrity
  - Protect info from tampering
- Availability
  - Ensure authorized parties can access info when needed
- Authentication and Nonrepudiation
  - Ensure the generation or modification is bound to the responsible party

###The Role of Cryptographic Tools

- Confidentiality
  - Encryption (symmetric and asymmetric key)
  - Secret sharing schemes
- Integrity
  - Message authentication codes
  - Hash functions (data digests)
- Authentication & Nonrepudiation
  - Digital Signatures
  - SSL
  - Commitment Schemes
  - Zero-knowledge Proofs
  
  
### A brief history of cryptography

- Crypto is at least 3500 years old
  - encrypted clay tablet found near mesopot datesd near 1500 bce
- History of crypto roughtly divided into classical and modern eras




####The classical era of crypto
- Crypto was synonymous with symmetric encription - resot fo field did not exist yet
- Encryptin relied almost excluseively on two principles: Transposition and Substitution
- Cryptanalysis was primitive; Best known method was Frequency analysis. Cyphers resistant to it were considered unbreakable
- Notable cyphers:
  - Atbash
  - CCaesar
  - Keyword
  - Vigenere
  - Autokey
  - Pig Latin
- These methods were naive compared to what we have today.

## 2. Symmetric and Asymmetric encryption

#### Symmetric Encryption
- Alice and bob hold the same key k
- Look it up


#### One-Time Pad
- Symmetric encryption scheme, invented in 1882 by American Cryptographer Frank Miller
- Key *K* is a string of bits with the same length as the plaintext
- To encrypt a message, it is XOR-ed with the key
- To Decrypt a message, it is also XOR-ed with the key
- Not feasible for use in practice due to key length
  - Key must be as long as the plaintext
  - The key must never be re-used or it will become easily breakable.
    - [The soviets made this mistake](http://en.wikipedia.org/wiki/Venona_project)
  - Security at the expense of usability comes at the expense of security (???)
- How can we "fix" this?

#### Stream Cipher
- Approximate the function of the one-time pad
- Generate a keystream based on a secret key using an algorithm
- Sacrifice security for convenience and practicality
- The most popular stream cipher is RC4 - It is used in skype, MS Office, Acrobate Reader and many other applications; And is often implemented from scratch in malware
- RC4's strength is its huge internal state, but improper use of it can lead to catastrophic weakness (See: [WEP](http://en.wikipedia.org/wiki/Wired_Equivalent_Privacy))

#### Block Ciphers
- Operate on blocks of bits (typically 128-bit blocks)
- Effectively a permutation in the space of blocks
- Example: Rijndael (a.k.a. AES)
- Naive approach: Iterate over the blocks, encrypting each one to create the ciphertext
  - Called ECB mode (Electronic Code Book)
    - Key flaw: [The ECB Penguin (Tux)](https://filippo.io/the-ecb-penguin/)
- [CBC mode](http://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher-block_chaining_.28CBC.29) can be better.

####Asymmetric Encryption
- Problems with symmetric encrypint
  -Keys have to be distributed in advance
  - Every pair of parties needs its own key (Requiring ~N^2 keys)
-Solution: *Asymmetric Encryption*



#####The RSA Algorithm
- RSA:inventors Ron Rivest, Adi Shamir, and Leonard Adelman
- Based on a problem cluster believed to be computationally difficult (Finding k-th roots modulo semiprimes; computing Euliers totient function efficiently; prime decomposition of integers)
- Messages are treated as numbers modulo n = pq where p,q are primes
- A public key e and private key d are picked such that for all messages m, (m^e)^d = m^ed = m
- To encrypt m, compute m^e; to decrypt c, compute c^d
- obtaining d from e is easy when prime decomposition of the group modulus n is known, and (apparently) difficult otherwise

##### Diffie-Hellman Key exchange
- An asymmetric encryption technique where tow parties establish shared key by revealing public keys to each other
- Calculations are performd modulo a prime number
- Both parties agree on prime number p and a base g to perform all calculations
- Alice picks her private key k(alice) and computes the public key, g^k(alice). Bob does the same
- Alice and bob exchange their public keys
- Alice computes (g^k(bob))^k(alice) = g^k(bob)k(alice) and bob computes (g^k(alice))^k(bob) = g^k(bob)k(alice). Alice and Bob now use g^k(bob)k(alice) as a symmetric key.

##3. Hashes, Signatures & Certificates

### Hash Functions
- Hash function is a function that maps data of arbitrary size to data of a fixed size, with small variations in input causing large variations in output
- Commonly used hash functions
  - SHA1 - output size 20bytes
  - MD5 - Output size 16 bytes
    - Deprecated but still used in practice currently
- A good hash function is **easy to calculate** and **difficult to invert**
- What are hash functions useful for?
  - Keeping passwords in plaintext on a server is risky; instead we can keep the password hashes (still a bad idea, but a step in the right direction)
  - Commitment schemes (publish the hash of some data, then publish the data at a later date)
  - Shortening digital signatures (next slide)
- MD5sum of previous text in this slide: c353739daf6f4045a004b9316b4087f2 (from original slide... recompute for these notes :smile:)

###Digital Signatures
- A Digital signature is a mathematical scheme for demonstrating the authenticity of a digital message or document
- A good digital signature scheme is:
  - Easy to calcualte for the person authorized to sign
  - difficult to forge for unauthorized parties
- Proof of concept - Bob can create a signature by encrypting the original data with his own RSA private key; signature is verified by decrypting using the public key
  - Vulnerable to **Existential Forgery** - Try various signatures and decrypt them using the public key; stop when you get a message you like
  - The **signature length** is the same length as the message
- Solution: Sign the hash of the message instead of the message itself

###Problem: Man in the Middle (MITM) Attack on Diffie-Hellman

Diagram (upload photo?)

###Solution: Digital Certificates
- The underlying issue - noconnection between **public key** and **identity**
- I can talk to someone and say "Hi, I'm google and this is my public key", and they have no way of verifying my identity
- A **digital certificate** is a document tying an entity's public key to that entity's identity. The certificate is signed by a trusted certificate authority
- In the example above, when Mallory (Mallory in the Middle:laugh:)intercepts Alice's connection and attempts to impersonate Bob, she will not be able to present a valid certificate tying her public key to Bob's identity.

- Suppose I want to get a digital certificate for myself. I have to contact a valid Certificate authority (Verisign, Thawte, Globalsign, Comodogroup...)
- I request a class 1 certificate.... <missed it>



##4. Modern applications
###Zero-Knowledge Proofs
- Introduced in a 1985 paper by Goldwasser, Micali, and Rackoff
- The idea: Alice can prove to Bob that she has some informaition without divulging any of the actual information
- Modern cryptography is full of constructions of this type that look like dark foodoo magic
- But it turns out that there is a real-life intuition behind why this is possible
- Sometimes, you can show a result that you **must** have known X to have obtained, which provides no information about X itself
- In [this example](http://en.wikipedia.org/wiki/Zero-knowledge_proof#Abstract_example), X is the magic word to open a door in the middle of a cave.

###Commitment Schemes
- Allows a person to commit to a certain value or statement, and reveal the result later (non-repudiation). The idea is similar to giving away a message in a locked box
- First formalized by Brassard, Chaum and Crepeau in 1988
- Example: predicting lottery results
  - Alice wants to prove she can predict the lottery results, without giving away the winning numbers
  - She can post a commitment to the value
  
### Secret Sharing Schemes
- Highly classified and sensitive information - Such as missle launch codes and encryption keys - cannot be trusted to a single person; that person could turn coat or be compromised
- **Secret Sharing Schemes**, discovered indepentently by Shamir and Blakely in 1979, allow "cutting up a secret into parts", such that all the parts are required to recover the secret. The partsa can then be distributed to several people
- Primitive proof of concept for sharing secret <missed it...>


###Bitcoin !!!
- Suggested in essay by "Satoshi Nakamoto" in 2008
- A decentralized coin without a central managing agent
- Problem: How do we stop people from creating fake coins or copies of coins?
- Solution: **Everyone** keeps track of a ledger with a transaction history
- For doing the paperwork and contributing to the system's stability, you are awarded newly minted Bitcoins
  - This is the only way to get new Bitcoins
- Work requires proof of work - Breaking a hash (sort of)

