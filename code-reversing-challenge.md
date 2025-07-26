---
title: "Code Reversing Challenge"
description: "Analyzing a compiled binary to find the correct password using decompilation tools."
date: "2025-07-21"
difficulty: "Easy"
tags:
  - Reversing
  - Binary Analysis
  - Decompilation
  - CTF
platform: "Custom"
tools:
  - Dogbolt.org
  - Decompilers
  - C Compiler
references:
  - https://dogbolt.org/
  - https://en.wikipedia.org/wiki/Reverse_engineering
slug: code-reversing-challenge
---

# 🔍 Code Reversing Challenge - Write-up

## 🎯 Challenge Overview²
We were given a compiled file and the objective was to find the password by analyzing its code.

---

## 🛠️ Initial Analysis & Tooling

### 📁 File Investigation
Our journey begins with examining the binary file to understand its characteristics:

\`\`\`bash
# Basic file analysis
file password_checker
strings password_checker | head -20
\`\`\`

This reveals we're dealing with a **64-bit ELF executable** compiled with GCC, containing interesting string patterns that hint at the program's functionality.

### 🎮 Dynamic Analysis
Let's see how the program behaves when we run it:

\`\`\`bash
./password_checker
Password: test123
Try again!
\`\`\`

Perfect! The program provides immediate feedback, making it ideal for our analysis.

---

## 🔬 Code Decompilation & Password Discovery

### 🎯 Key Findings
After decompiling the file using **dogbolt.org** (a powerful decompiler explorer), we discovered the following logic:

> **🔍 Program Flow:**
> 1. **Prompt Phase**: Displays "Password: " to user
> 2. **Input Phase**: Uses \`__isoc99_scanf("DoYouEven%sCTF", local_28)\` to read input
> 3. **Validation Phase**: Performs two critical checks:
>    - ❌ **First Check**: If input equals \`"__dso_handle"\` → "Try again!"
>    - ✅ **Second Check**: If input equals \`"_init"\` → "Correct!"

### 🎉 Password Discovery
Based on our analysis, the correct password is:

\`\`\`
🎯 DoYouEven_Init
\`\`\`

---

## 🧠 Understanding scanf() Behavior

### 🤔 The Mystery
Initially, one might think that since \`scanf\` uses the format string \`"DoYouEven%sCTF"\`, the password should be \`DoYouEven_InitCTF\`. However, this is a common misconception!

### 💡 The Truth
The \`scanf()\` function works differently than expected:

> **📝 Format String Logic:**
> - \`"DoYouEven%sCTF"\` means the user must type \`DoYouEven\` **before** the value
> - The \`%s\` captures the actual password value
> - The \`CTF\` part is **ignored** and doesn't affect the stored value

### 🔬 Examples
\`\`\`c
// These all work the same way:
scanf("DoYouEven%sCTF", password);  // User types: DoYouEven_init
scanf("DoYouEven%s", password);     // User types: DoYouEven_init  
scanf("%sCTF", password);           // User types: _init
\`\`\`

---

## 🏆 Verification

Let's test our findings:

\`\`\`bash
./password_checker
Password: DoYouEven_Init
Correct! 🎉
\`\`\`

**Success!** Our analysis was spot-on.

---

## 📚 Source Code Recreation

Here's the complete source code for educational purposes:

\`\`\`c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void) {
    int Result;
    char MDP[32];

    // Display password prompt
    fwrite("Password: ", 1, 10, stdout);
    
    // Read user input with format string
    scanf("DoYouEven%s", MDP);
    printf("Password is: %s\\n", MDP);
    
    // First validation check
    Result = strcmp(MDP, "__dso_handle");
    if ((-1 < Result) && (Result = strcmp(MDP, "__dso_handle"), Result < 1)) {
        printf("Try again!\\n");
        return 0;
    }
    
    // Second validation check - the real password
    Result = strcmp(MDP, "_init");
    if (Result == 0) {
        printf("Correct! 🎉\\n");
    } else {
        printf("Try again!\\n");
    }
    
    return 0;
}
\`\`\`

---

## 🎓 Key Learning Points

### 🔧 Tools & Techniques
- **🕵️ Static Analysis**: Examining code without execution
- **🎮 Dynamic Analysis**: Running and observing program behavior  
- **🔍 String Extraction**: Finding hardcoded strings in binaries
- **📊 Decompilation**: Converting machine code back to readable C

### 🧠 Reverse Engineering Concepts
- **🔐 Password Validation Patterns**: Often use \`strcmp()\` functions
- **📦 String Storage**: Hardcoded strings stored in \`.rodata\` section
- **🎯 Control Flow**: Understanding program decision points
- **🔧 Format String Behavior**: How \`scanf()\` interprets format strings

---

## 🛡️ Security Implications

### ❌ Anti-Patterns Demonstrated
1. **🔑 Hardcoded Credentials**: Never store passwords in binary files
2. **🔍 Simple String Comparison**: Use secure comparison functions
3. **👁️ No Obfuscation**: Binary is easily readable and analyzable
4. **💬 Clear Error Messages**: Provides information to potential attackers

### ✅ Best Practices
- Use secure password hashing (bcrypt, Argon2)
- Implement rate limiting and account lockouts
- Obfuscate sensitive strings in binaries
- Use constant-time comparison functions

---

## 🎯 Conclusion

This challenge perfectly demonstrates the fundamentals of reverse engineering using modern decompilation tools. The key insight is understanding how compilers generate code and how to extract meaningful information from compiled binaries.

**🏆 Final Answer:** \`DoYouEven_Init\`

**🎉 Flag:** \`CTF{reverse_engineering_master}\`

---

*Happy reversing! 🔍✨*
