---
title: "CyberHeroes"
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

## Overview

This CTF challenge is classified as an easy-level room on TryHackMe. The main purpose of this challenge is to identify the correct username and password credentials that will grant access to the elite club of CyberHeroes. The challenge focuses on client-side authentication analysis and JavaScript code review.

## Initial Analysis

Upon accessing the target website, we are presented with a login page interface. When attempting to authenticate with incorrect credentials, the system displays a specific type of alert notification to the user.

![Login Error Alert](Capture/CyberHeroes/CH1.png)

## Network Traffic Investigation

To better understand the authentication mechanism, I examined the network traffic during the login attempt. The network recording reveals a crucial observation: there is no HTTP request made to any backend endpoint to verify the credentials against a database or external authentication service.

![Network Traffic Analysis](Capture/CyberHeroes/CH2.png)

This finding suggests that the credential validation logic is implemented entirely on the client-side rather than through a server-side API call. This is a significant security vulnerability as it exposes the authentication mechanism to client-side analysis.

## Client-Side Code Analysis

Further investigation of the client-side code reveals the presence of an `authenticate()` function that is executed when the submit button is clicked on the login form.

![Authentication Function Location](Capture/CyberHeroes/CH3.png)

## Credential Discovery

After locating the authenticate function in the JavaScript code, I was able to examine its implementation and extract the hardcoded credentials. The analysis reveals the following:

- **Variable 'a' (Username):** `h3ck3rBoi`
- **Variable 'b' (Password):** This value is stored as a reversed string `54321@terceSrepuS`, which when reversed becomes `SuperSecret@12345`

![Authenticate Function Code](Capture/CyberHeroes/CH4.png)

## Solution and Flag Retrieval

With the discovered credentials in hand, I proceeded to input them into the sign-in form:
- **Username:** `h3ck3rBoi`
- **Password:** `SuperSecret@12345`

Successfully authenticating with these credentials grants access to the system and reveals the flag.

![Successful Authentication and Flag](Capture/CyberHeroes/CH5.png)

## Conclusion

This challenge demonstrates a common security vulnerability where authentication logic is implemented on the client-side, making credentials easily discoverable through code analysis. The challenge was straightforward and quick to solve, serving as an excellent introduction to client-side security analysis techniques.


The challenge reinforces the importance of implementing proper server-side authentication and never storing credentials in client-accessible code.