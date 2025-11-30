---
title: "XorXorXor"
description: "Breaking a simple XOR encryption scheme by exploiting known plaintext and key reuse to recover the encryption key and decrypt the flag."
date: "2025-11-29"
difficulty: "Easy"
tags:
  - Cryptography
  - XOR Cipher
  - Known Plaintext Attack
  - Repeating Key
  - Python
  - CTF
platform: "HackTheBox"
tools:
  - Python3
  - Hex Editor
references:
  - https://en.wikipedia.org/wiki/XOR_cipher
  - https://crypto.stackexchange.com/questions/59/taking-advantage-of-one-time-pad-key-reuse
slug: xorxorxor-crypto-challenge
---

# XorXorXor - Cryptography Challenge

### Challenge Description

We are provided with a Python script that implements a simple XOR encryption scheme and an encrypted flag output. The challenge demonstrates a fundamental cryptographic weakness: using a short repeating key with XOR encryption, especially when portions of the plaintext are known.

---

## Source Code Analysis

### Provided Encryption Script

```python
#!/usr/bin/python3

import os

flag = open('flag.txt', 'r').read().strip().encode()

class XOR:
    def __init__(self):
        self.key = os.urandom(4)
    
    def encrypt(self, data: bytes) -> bytes:
        xored = b''
        for i in range(len(data)):
            xored += bytes([data[i] ^ self.key[i % len(self.key)]])
        return xored
    
    def decrypt(self, data: bytes) -> bytes:
        return self.encrypt(data)

def main():
    global flag
    crypto = XOR()
    print('Flag:', crypto.encrypt(flag).hex())

if __name__ == '__main__':
    main()
```

### Vulnerability Analysis

The encryption implementation contains several critical weaknesses that make it vulnerable to cryptanalysis:

1. **Short Key Length:** The encryption key is only 4 bytes long, generated using `os.urandom(4)`

2. **Key Reuse:** The key repeats cyclically throughout the entire plaintext using the modulo operator (`i % len(self.key)`). This means:
   - Bytes 0-3 are XORed with the key
   - Bytes 4-7 are XORed with the same key again
   - This pattern continues for the entire message

3. **Known Plaintext:** We know the flag format begins with `HTB{`, providing us with known plaintext for the first 4 bytes

4. **XOR Properties:** XOR encryption has a fundamental property: if `C = P ⊕ K`, then `K = C ⊕ P`. This allows us to recover the key when we know both ciphertext and plaintext.

---

## Exploitation Strategy

### Known Plaintext Attack

Since we know the flag format starts with `HTB{`, we can exploit this known plaintext to recover the 4-byte encryption key. The attack follows this logical process:

1. **Extract the first 4 bytes** of the encrypted output (ciphertext)
2. **XOR these bytes with the known plaintext** `HTB{` to recover the key
3. **Use the recovered key** to decrypt the entire encrypted flag

### Mathematical Foundation

The XOR operation has the following properties:
- `A ⊕ B = C`
- `C ⊕ B = A`
- `C ⊕ A = B`

In our case:
- `Ciphertext = Plaintext ⊕ Key`
- `Key = Ciphertext ⊕ Plaintext`

---

## Solution Implementation

### Step 1: Key Recovery

Let's assume the encrypted output begins with the hex bytes: `13 4a f6 e1`

We know the plaintext is `HTB{`, which in ASCII hex is: `48 54 42 7b`

We can now recover the key by XORing the ciphertext with the known plaintext:

```
Ciphertext byte 0: 0x13 ⊕ Plaintext byte 0: 0x48 = 0x5b
Ciphertext byte 1: 0x4a ⊕ Plaintext byte 1: 0x54 = 0x1e
Ciphertext byte 2: 0xf6 ⊕ Plaintext byte 2: 0x42 = 0xb4
Ciphertext byte 3: 0xe1 ⊕ Plaintext byte 3: 0x7b = 0x9a
```

**Key Calculation:**
```
13 ^ 48 = 5b
4a ^ 54 = 1e
f6 ^ 42 = b4
e1 ^ 7b = 9a
```

**Recovered Key:** `5b 1e b4 9a`

### Step 2: Decryption Script

With the recovered key, we can now decrypt the entire flag. Here's the complete decryption script:

```python
#!/usr/bin/python3

# Read the encrypted output
with open('output.txt', 'r') as f:
    hex_string = f.read().strip()

# Convert hex string to bytes
encrypted_bytes = bytes.fromhex(hex_string)

# Recovered 4-byte key
key = bytes([0x5b, 0x1e, 0xb4, 0x9a])

# Decrypt using XOR with repeating key
decrypted = bytes([encrypted_bytes[i] ^ key[i % 4] for i in range(len(encrypted_bytes))])

# Display the decrypted flag
print('Flag:', decrypted.decode())
```

### Script Breakdown

1. **File Reading:** Opens `output.txt` containing the hexadecimal encrypted flag
2. **Hex Conversion:** Converts the hex string to raw bytes using `bytes.fromhex()`
3. **Key Definition:** Defines the recovered 4-byte key as a bytes object
4. **XOR Decryption:** Applies XOR operation with the repeating key pattern:
   - Iterates through each byte of the encrypted data
   - XORs each byte with the corresponding key byte (using modulo 4 for repetition)
   - Constructs the decrypted bytes
5. **Output:** Decodes the bytes to UTF-8 and displays the flag

---

## Alternative Approach: Brute Force

While we used a known plaintext attack, an alternative approach would be brute-forcing the 4-byte key space:

```python
#!/usr/bin/python3

with open('output.txt', 'r') as f:
    encrypted = bytes.fromhex(f.read().strip())

# Known plaintext prefix
known_prefix = b'HTB{'

# Brute force 4-byte key (2^32 possibilities)
for k1 in range(256):
    for k2 in range(256):
        for k3 in range(256):
            for k4 in range(256):
                key = bytes([k1, k2, k3, k4])
                
                # Test if first 4 bytes decrypt correctly
                test = bytes([encrypted[i] ^ key[i % 4] for i in range(4)])
                
                if test == known_prefix:
                    # Found the key, decrypt entire message
                    decrypted = bytes([encrypted[i] ^ key[i % 4] for i in range(len(encrypted))])
                    print(f'Key found: {key.hex()}')
                    print(f'Flag: {decrypted.decode()}')
                    exit()
```

However, this approach is significantly slower (approximately 4.3 billion iterations) compared to the direct key recovery method, which requires only 4 XOR operations.

---