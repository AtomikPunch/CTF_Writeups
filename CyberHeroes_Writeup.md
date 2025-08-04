---
title: "CyberHeroes — TryHackMe Walkthrough"
description: "Client-side JavaScript analysis to uncover login credentials and retrieve the flag in this easy CTF challenge."
date: "2025-08-04"
difficulty: "Easy"
tags:
  - Web Exploitation
  - JavaScript Analysis
  - Client-side Security
  - CTF
platform: "TryHackMe"
tools:
  - Browser DevTools
  - JavaScript Debugging
references:
  - https://tryhackme.com/room/cyberheroes
slug: cyberheroes-walkthrough
---

# CyberHeroes — TryHackMe Walkthrough

[TryHackMe Room — CyberHeroes](https://tryhackme.com/room/cyberheroes)

This CTF is classified as **easy**. The primary objective is to retrieve the **username** and **password** in order to join the elite club of **CyberHeroes**.

---

## Initial Reconnaissance

Upon visiting the website, we are presented with a **login page**.

![Login Page](Capture/CyberHeroes/CH1.png)

When incorrect credentials are entered, we receive a generic alert message.

![Alert on Wrong Credentials](Capture/CyberHeroes/CH2.png)

To investigate how the login is handled, we open the **network tab** in the developer tools and attempt a login.

![Network Recording](Capture/CyberHeroes/CH3.png)

Interestingly, we observe that **no network requests** are sent to any backend endpoint for authentication. This strongly indicates that the **credential validation is handled on the client side**, likely via JavaScript code.

---

## Code Analysis

We examine the site’s JavaScript and identify a function named `authenticate()` that is triggered upon clicking the submit button.

![Authenticate Function Trigger](Capture/CyberHeroes/CH4.png)

After locating the `authenticate()` function in the source code, we discover that:

- Variable `a` (the username) is set to:  
  ```
  h3ck3rBoi
  ```

- Variable `b` (the password) is set to the **reverse** of the string:  
  ```
  54321@terceSrepuS
  ```

Reversing this gives us the password:  
```
SuperSecret@12345
```

---

## Final Step — Login Success

Using the discovered credentials:

- **Username:** `h3ck3rBoi`  
- **Password:** `SuperSecret@12345`

We successfully log in and retrieve the **flag**.

![Flag Retrieved](/Capture/CyberHeroes/CH5.png)

---

## Conclusion

This was a quick and straightforward challenge that emphasizes the importance of **not placing authentication logic in client-side code**. A great reminder for developers and an easy win for aspiring CyberHeroes.